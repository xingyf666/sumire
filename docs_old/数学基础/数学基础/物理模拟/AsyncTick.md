# AsyncTick

## 循环调用流程

### FEngineLoop->Tick

首先在 FEngineLoop 中调用 Tick 方法

```cpp
void FEngineLoop::Tick()
{
//...

{
	TRACE_CPUPROFILER_EVENT_SCOPE(HeartBeat);
	// Send a heartbeat for the diagnostics thread
	FThreadHeartBeat::Get().HeartBeat(true);
}

FGameThreadHitchHeartBeat::Get().FrameStart();

// ...

// Must be called on the game thread outside of any frame breadcrumbs.
LatchRenderThreadConfiguration();

// ...

// flush debug output which has been buffered by other threads
{
	QUICK_SCOPE_CYCLE_COUNTER(STAT_FEngineLoop_FlushThreadedLogs); 
	GLog->FlushThreadedLogs(EOutputDeviceRedirectorFlushOptions::Async);
}

// ...

// main game engine tick (world, game objects, etc.)
GEngine->Tick(FApp::GetDeltaTime(), bIdleMode);
// 从这里进入

}
```



### UUnrealEdEngine->Tick

```cpp
void UUnrealEdEngine::Tick(float DeltaSeconds, bool bIdleMode)
{
	Super::Tick( DeltaSeconds, bIdleMode );
	// 从这里进入

}
```



### UEditorEngine->Tick

```cpp
void UEditorEngine::Tick( float DeltaSeconds, bool bIdleMode )
{
// ...

// tick the level
PieContext.World()->Tick( LEVELTICK_All, TickDeltaSeconds );
// 从这里进入
}

```



### UWorld->Tick

```cpp
/**
 * Update the level after a variable amount of time, DeltaSeconds, has passed.
 * All child actors are ticked after their owners have been ticked.
 */
void UWorld::Tick( ELevelTick TickType, float DeltaSeconds )
{
// ...


// If caller wants time update only, or we are paused, skip the rest.
if (bDoingActorTicks)
{
	// Actually tick actors now that context is set up
	SetupPhysicsTickFunctions(DeltaSeconds);
	TickGroup = TG_PrePhysics; // reset this to the start tick group
	FTickTaskManagerInterface::Get().StartFrame(this, DeltaSeconds, TickType, LevelsToTick);

	SCOPE_CYCLE_COUNTER(STAT_TickTime);
	CSV_SCOPED_TIMING_STAT_EXCLUSIVE(TickActors);
	{
		SCOPE_TIME_GUARD_MS(TEXT("UWorld::Tick - TG_PrePhysics"), 10);
		SCOPE_CYCLE_COUNTER(STAT_TG_PrePhysics);
		CSV_SCOPED_SET_WAIT_STAT(PrePhysics);
		RunTickGroup(TG_PrePhysics);
	}
	bInTick = false;
	EnsureCollisionTreeIsBuilt();
	bInTick = true;
	{
		SCOPE_CYCLE_COUNTER(STAT_TG_StartPhysics);
		SCOPE_TIME_GUARD_MS(TEXT("UWorld::Tick - TG_StartPhysics"), 10);
		CSV_SCOPED_SET_WAIT_STAT(StartPhysics);
		RunTickGroup(TG_StartPhysics);
		// 从这里进入

	}
	{
		SCOPE_CYCLE_COUNTER(STAT_TG_DuringPhysics);
		SCOPE_TIME_GUARD_MS(TEXT("UWorld::Tick - TG_DuringPhysics"), 10);
		CSV_SCOPED_SET_WAIT_STAT(DuringPhysics);
		RunTickGroup(TG_DuringPhysics, false); // No wait here, we should run until idle though. We don't care if all of the async ticks are done before we start running post-phys stuff
	}
	TickGroup = TG_EndPhysics; // set this here so the current tick group is correct during collision notifies, though I am not sure it matters. 'cause of the false up there^^^
	{
		SCOPE_CYCLE_COUNTER(STAT_TG_EndPhysics);
		SCOPE_TIME_GUARD_MS(TEXT("UWorld::Tick - TG_EndPhysics"), 10);
		CSV_SCOPED_SET_WAIT_STAT(EndPhysics);
		RunTickGroup(TG_EndPhysics);
	}
	{
		SCOPE_CYCLE_COUNTER(STAT_TG_PostPhysics);
		SCOPE_TIME_GUARD_MS(TEXT("UWorld::Tick - TG_PostPhysics"), 10);
		CSV_SCOPED_SET_WAIT_STAT(PostPhysics);
		RunTickGroup(TG_PostPhysics);
	}
	
}
else if( bIsPaused )
{
	FTickTaskManagerInterface::Get().RunPauseFrame(this, DeltaSeconds, LEVELTICK_PauseTick, LevelsToTick);
}

// ...
}

void UWorld::RunTickGroup(ETickingGroup Group, bool bBlockTillComplete = true)
{
	check(TickGroup == Group); // this should already be at the correct value, but we want to make sure things are happening in the right order
	FTickTaskManagerInterface::Get().RunTickGroup(Group, bBlockTillComplete);
	// 从这里进入

	// Flush any grouped updates (TODO replace this with a more generic end of tick group callback)
	ProcessPendingGroupMoves(true);

	TickGroup = ETickingGroup(TickGroup + 1); // new actors go into the next tick group because this one is already gone
}
```



### FTickTaskManager

TickTaskSequencer 调用 ReleaseTickGroup，后者调用 DispatchTickGroup 分发任务

```cpp
class FTickTaskManager : public FTickTaskManagerInterface
{
/**
	* Run a tick group, ticking all actors and components
	* @param Group - Ticking group to run
	* @param bBlockTillComplete - if true, do not return until all ticks are complete
*/
virtual void RunTickGroup(ETickingGroup Group, bool bBlockTillComplete ) override
{
	check(Context.TickGroup == Group); // this should already be at the correct value, but we want to make sure things are happening in the right order
	check(bTickNewlySpawned); // we should be in the middle of ticking

	TArray<FTickFunction*> TicksToManualDispatch;
	FTaskSyncManager* SyncManager = FTaskSyncManager::Get();

	if (SyncManager)
	{
		SyncManager->StartTickGroup(Context.World, Group, TicksToManualDispatch);
	}

	TickTaskSequencer.ReleaseTickGroup(Group, bBlockTillComplete, TicksToManualDispatch);
	// 从这里进入


	Context.TickGroup = ETickingGroup(Context.TickGroup + 1); // new actors go into the next tick group because this one is already gone
	if (bBlockTillComplete) // we don't deal with newly spawned ticks within the async tick group, they wait until after the async stuff
	{
		QUICK_SCOPE_CYCLE_COUNTER(STAT_TickTask_RunTickGroup_BlockTillComplete);

		bool bFinished = false;
		for (int32 Iterations = 0;Iterations < 101; Iterations++)
		{
			int32 Num = 0;
			for( int32 LevelIndex = 0; LevelIndex < LevelList.Num(); LevelIndex++ )
			{
				Num += LevelList[LevelIndex]->QueueNewlySpawned(Context.TickGroup);
			}
			if (Num && Context.TickGroup == TG_NewlySpawned)
			{
				SCOPE_CYCLE_COUNTER(STAT_TG_NewlySpawned);
				TickTaskSequencer.ReleaseTickGroup(TG_NewlySpawned, true, TicksToManualDispatch);
			}
			else
			{
				bFinished = true;
				break;
			}
		}
		if (!bFinished)
		{
			// this is runaway recursive spawning.
			for( int32 LevelIndex = 0; LevelIndex < LevelList.Num(); LevelIndex++ )
			{
				LevelList[LevelIndex]->LogAndDiscardRunawayNewlySpawned(Context.TickGroup);
			}
		}
	}

	if (SyncManager)
	{
		SyncManager->EndTickGroup(Context.World, Group);
	}
}

/**
 * Release the queued ticks for a given tick group and process them.
 * @param WorldTickGroup - tick group to release
 * @param bBlockTillComplete - if true, do not return until all ticks are complete
 * @param TickFunctionsToManualDispatch - dispatch any manual tick functions in this lister after the normal ones
**/
void ReleaseTickGroup(ETickingGroup WorldTickGroup, bool bBlockTillComplete, TArray<FTickFunction*>& TicksToManualDispatch)
{
if (bLogTicks)
{
	UE_LOG(LogTick, Log, TEXT("tick %6llu ---------------------------------------- Release tick group %d"),(uint64)GFrameCounter, (int32)WorldTickGroup);
}
checkSlow(WorldTickGroup >= 0 && WorldTickGroup < TG_MAX);

{
	SCOPE_CYCLE_COUNTER(STAT_ReleaseTickGroup);
	if (bSingleThreadMode || CVarAllowAsyncTickDispatch.GetValueOnGameThread() == 0)
	{
		DispatchTickGroup(ENamedThreads::GameThread, WorldTickGroup);
	}
	else
	{
		// dispatch the tick group on another thread, that way, the game thread can be processing ticks while ticks are being queued by another thread
		FTaskGraphInterface::Get().WaitUntilTaskCompletes(
			TGraphTask<FDipatchTickGroupTask>::CreateTask(nullptr, ENamedThreads::GameThread).ConstructAndDispatchWhenReady(*this, WorldTickGroup));
	}
}

for (FTickFunction* TickToManualDispatch : TicksToManualDispatch)
{
	// These must be safe to dispatch
	ensure(TickToManualDispatch->DispatchManually());
}
TicksToManualDispatch.Reset();

if (bBlockTillComplete || bSingleThreadMode)
{
	SCOPE_CYCLE_COUNTER(STAT_ReleaseTickGroup_Block);
	for (ETickingGroup Block = WaitForTickGroup; Block <= WorldTickGroup; Block = ETickingGroup(Block + 1))
	{
		CA_SUPPRESS(6385);
		if (TickCompletionEvents[Block].Num())
		{
			TRACE_CPUPROFILER_EVENT_SCOPE(TickCompletionEvents);

			if (GIdleTaskWorkMS >= 0.0)
			{
				bool bDoIdleWork = (GIdleTaskWorkMS > 0.0);
				bool bDoDeadlockCheck = ManualDispatchTicks[WorldTickGroup].Num() > 0;
				uint64 EndIdleWork = 0;
				int32 PreviousTasksRemaining = TickCompletionEvents[Block].Num();
				int32 IdleCount = 0;

				auto IdleWorkUpdate = [this, &bDoIdleWork, &bDoDeadlockCheck, &EndIdleWork, &PreviousTasksRemaining, &IdleCount, WorldTickGroup]
					(int32 TasksRemaining) -> FTaskGraphInterface::EProcessTasksOperation
				{
					if (bDoIdleWork)
					{
						// Compute cycles for ending idle work if necessary, this is called after the first pass of processing
						if (EndIdleWork == 0)
						{
							const double TaskWorkSeconds = (GIdleTaskWorkMS / 1000.0);
							EndIdleWork = FPlatformTime::Cycles64() + FMath::TruncToInt64(TaskWorkSeconds / FPlatformTime::GetSecondsPerCycle64());
						}
						else if (FPlatformTime::Cycles64() > EndIdleWork)
						{
							bDoIdleWork = false;
						}
					}

					if (bDoDeadlockCheck)
					{
						if (TasksRemaining == PreviousTasksRemaining)
						{
							// No tasks were completed during last attempt
							constexpr int32 IdleCountToBeDeadlocked = 10;
							if (++IdleCount > IdleCountToBeDeadlocked)
							{
								// Nothing is changing so this could be deadlocked, make sure our manual dispatches have happened
								this->VerifyManualDispatch(WorldTickGroup);

								bDoDeadlockCheck = false;
							}
						}
						else
						{
							PreviousTasksRemaining = TasksRemaining;
							IdleCount = 0;
						}
					}
					
					if (bDoIdleWork)
					{
						// Run a worker thread task, this could also run other game thread tasks if we add support for that
						return FTaskGraphInterface::EProcessTasksOperation::ProcessOneOtherTask;
					}
					else if (bDoDeadlockCheck)
					{
						// Still need to check for deadlock so process named thread tasks if there are any
						return FTaskGraphInterface::EProcessTasksOperation::ProcessNamedThreadTasks;
					}
					else
					{
						// We don't need to do the idle or deadlock checks so wait forever
						return FTaskGraphInterface::EProcessTasksOperation::WaitUntilComplete;
					}
				};

				FTaskGraphInterface::Get().ProcessUntilTasksComplete(TickCompletionEvents[Block], ENamedThreads::GameThread, IdleWorkUpdate);
// 从这里进入



			}
			else
			{
				// Old behavior of waiting for all of them
				FTaskGraphInterface::Get().WaitUntilTasksComplete(TickCompletionEvents[Block], ENamedThreads::GameThread);
			}

			if (bSingleThreadMode || Block == TG_NewlySpawned || CVarAllowAsyncTickCleanup.GetValueOnGameThread() == 0 || TickCompletionEvents[Block].Num() < 50)
			{
				ResetTickGroup(Block);
			}
			else
			{
				DECLARE_CYCLE_STAT(TEXT("FDelegateGraphTask.ResetTickGroup"), STAT_FDelegateGraphTask_ResetTickGroup, STATGROUP_TaskGraphTasks);
				CleanupTasks.Add(TGraphTask<FResetTickGroupTask>::CreateTask(nullptr, ENamedThreads::GameThread).ConstructAndDispatchWhenReady(*this, Block));
			}
		}
	}
	WaitForTickGroup = ETickingGroup(WorldTickGroup + (WorldTickGroup == TG_NewlySpawned ? 0 : 1)); // don't advance for newly spawned
}
else
{
	// since this is used to soak up some async time for another task (physics), we should process whatever we have now
	FTaskGraphInterface::Get().ProcessThreadUntilIdle(ENamedThreads::GameThread);
	check(WorldTickGroup + 1 < TG_MAX && WorldTickGroup != TG_NewlySpawned); // you must block on the last tick group! And we must block on newly spawned
}
}

};
```



### FTaskGraphCompatibilityImplementation

```cpp
/**
*	FTaskGraphCompatibilityImplementation
*	Implementation of the centralized part of the task graph system using the ne low level Backend.
*	These parts of the system have no knowledge of the dependency graph, they exclusively work on tasks.
**/
class FTaskGraphCompatibilityImplementation final : public FTaskGraphInterface
{
bool ProcessUntilTasksComplete(const FGraphEventArray& Tasks, ENamedThreads::Type CurrentThreadIfKnown, const FProcessTasksUpdateCallback& IdleWorkUpdate) final override
{
	TRACE_CPUPROFILER_EVENT_SCOPE(ProcessUntilTasksComplete);

	ENamedThreads::Type NamedThreadWithFlags = CurrentThreadIfKnown;
	if (ENamedThreads::GetThreadIndex(CurrentThreadIfKnown) == ENamedThreads::AnyThread)
	{
		bool bIsHiPri = !!ENamedThreads::GetTaskPriority(CurrentThreadIfKnown);
		int32 Priority = ENamedThreads::GetThreadPriorityIndex(CurrentThreadIfKnown);
		check(!ENamedThreads::GetQueueIndex(CurrentThreadIfKnown));
		CurrentThreadIfKnown = ENamedThreads::GetThreadIndex(GetCurrentThread());
		NamedThreadWithFlags = ENamedThreads::SetPriorities(CurrentThreadIfKnown, Priority, bIsHiPri);
	}
	else
	{
		CurrentThreadIfKnown = ENamedThreads::GetThreadIndex(CurrentThreadIfKnown);
		check(CurrentThreadIfKnown == ENamedThreads::GetThreadIndex(GetCurrentThread()));
	}

	// Copy into faster array to avoid ref count changes, these will all be in Tasks array
	TArray<FGraphEvent*> RemainingTasks;
	RemainingTasks.SetNumUninitialized(Tasks.Num());
	for (int32 TaskIndex = 0; TaskIndex < Tasks.Num(); TaskIndex++)
	{
		RemainingTasks[TaskIndex] = Tasks[TaskIndex].GetReference();
		if (!ensure(RemainingTasks[TaskIndex] != nullptr))
		{
			// Fail if one of the events is not initialized
			UE_LOG(LogTaskGraph, Error, TEXT("ProcessUntilTasksComplete was passed an invalid event!"));
			return false;
		}
	}

	// Always start by processing named thread tasks once
	EProcessTasksOperation CurrentOperation = EProcessTasksOperation::ProcessNamedThreadTasks;
	
	const UE::FTimeout NeverTimeout = UE::FTimeout::Never();		
	bool bHasOtherTasks = true;

	while (CurrentOperation != EProcessTasksOperation::StopProcessing)
	{
		if (CurrentOperation == EProcessTasksOperation::ProcessNamedThreadTasks)
		{
			// Process until this named thread (could be a local queue) is idle, which should complete most of the tasks
			ProcessThreadUntilIdle(NamedThreadWithFlags);
			// 从这里进入


		}
		// ...
	}

	// Stopped due to callback, some tasks may not be complete
	return false;
}

uint64 ProcessThreadUntilIdle(ENamedThreads::Type CurrentThread) final override
{
	int32 QueueIndex = ENamedThreads::GetQueueIndex(CurrentThread);
	CurrentThread = ENamedThreads::GetThreadIndex(CurrentThread);
	check(CurrentThread >= 0 && CurrentThread < NumNamedThreads);
	check(CurrentThread == GetCurrentThread());
	return Thread(CurrentThread).ProcessTasksUntilIdle(QueueIndex);
	// 从这里进入
}

virtual uint64 ProcessTasksUntilIdle(int32 QueueIndex) override
{
	check(Queue(QueueIndex).StallRestartEvent); // make sure we are started up

	Queue(QueueIndex).QuitForReturn = false;
	verify(++Queue(QueueIndex).RecursionGuard == 1);
	uint64 ProcessedTasks = ProcessTasksNamedThread(QueueIndex, false);
	// 从这里进入


	verify(!--Queue(QueueIndex).RecursionGuard);
	return ProcessedTasks;
}

uint64 ProcessTasksNamedThread(int32 QueueIndex, bool bAllowStall)
	{
		uint64 ProcessedTasks = 0;
#if UE_EXTERNAL_PROFILING_ENABLED
		static thread_local bool bOnce = false;
		if (!bOnce)
		{
			FExternalProfiler* Profiler = FActiveExternalProfilerBase::GetActiveProfiler();
			if (Profiler)
			{
				Profiler->SetThreadName(ThreadIdToName(ThreadId));
			}
			bOnce = true;
		}
#endif

		TStatId StallStatId;
		bool bCountAsStall = false;
#if STATS
		TStatId StatName;
		FCycleCounter ProcessingTasks;
		if (ThreadId == ENamedThreads::GameThread)
		{
			StatName = GET_STATID(STAT_TaskGraph_GameTasks);
			StallStatId = GET_STATID(STAT_TaskGraph_GameStalls);
			bCountAsStall = true;
		}
		else if (ThreadId == ENamedThreads::GetRenderThread())
		{
			if (QueueIndex > 0)
			{
				StallStatId = GET_STATID(STAT_TaskGraph_RenderStalls);
				bCountAsStall = true;
			}
			// else StatName = none, we need to let the scope empty so that the render thread submits tasks in a timely manner. 
		}
		else
		{
			StatName = GET_STATID(STAT_TaskGraph_OtherTasks);
			StallStatId = GET_STATID(STAT_TaskGraph_OtherStalls);

			// Don't count RHI thread waits as stalls.
			bCountAsStall = ThreadId != ENamedThreads::RHIThread;
		}
		bool bTasksOpen = false;
		if (FThreadStats::IsCollectingData(StatName))
		{
			bTasksOpen = true;
			ProcessingTasks.Start(StatName);
		}
#endif
		const bool bIsRenderThreadMainQueue = (ENamedThreads::GetThreadIndex(ThreadId) == ENamedThreads::ActualRenderingThread) && (QueueIndex == 0);
		while (!Queue(QueueIndex).QuitForReturn)
		{
			const bool bIsRenderThreadAndPolling = bIsRenderThreadMainQueue && (GRenderThreadPollPeriodMs >= 0);
			const bool bStallQueueAllowStall = bAllowStall && !bIsRenderThreadAndPolling;
			FBaseGraphTask* Task = Queue(QueueIndex).StallQueue.Pop(0, bStallQueueAllowStall);
			TestRandomizedThreads();
			if (!Task)
			{
#if STATS
				if (bTasksOpen)
				{
					ProcessingTasks.Stop();
					bTasksOpen = false;
				}
#endif
				if (bAllowStall)
				{
					TRACE_CPUPROFILER_EVENT_SCOPE(WaitForTasks);
					{
						FScopeCycleCounter Scope(StallStatId, EStatFlags::Verbose);
						Queue(QueueIndex).StallRestartEvent->Wait(bIsRenderThreadAndPolling ? GRenderThreadPollPeriodMs : MAX_uint32, bCountAsStall);
						if (Queue(QueueIndex).QuitForShutdown)
						{
							return ProcessedTasks;
						}
						TestRandomizedThreads();
					}
#if STATS
					if (!bTasksOpen && FThreadStats::IsCollectingData(StatName))
					{
						bTasksOpen = true;
						ProcessingTasks.Start(StatName);
					}
#endif
					continue;
				}
				else
				{
					break; // we were asked to quit
				}
			}
			else
			{
				Task->Execute(NewTasks, ENamedThreads::Type(ThreadId | (QueueIndex << ENamedThreads::QueueIndexShift)), true);
				// 从这里进入


				ProcessedTasks++;
				TestRandomizedThreads();
			}
		}
#if STATS
		if (bTasksOpen)
		{
			ProcessingTasks.Stop();
			bTasksOpen = false;
		}
#endif
		return ProcessedTasks;
	}
	
};
```



### FBaseGraphTask

```cpp
/** Base class for all graph tasks, used for both TGraphTask and simple graph events. This is a wrapper around the `Tasks::Private::FTaskBase` functionality */
class FBaseGraphTask : public UE::Tasks::Private::FTaskBase
{
	FORCEINLINE void Execute(TArray<FBaseGraphTask*>& NewTasks, ENamedThreads::Type CurrentThread, bool bDeleteOnCompletion)
	{	// only called for named thread tasks, normal tasks are executed using `FTaskBase` API directly (see `TGraphTask`)
		checkSlow(NewTasks.Num() == 0);
		checkSlow(bDeleteOnCompletion);
		checkSlow(IsNamedThreadTask());
		verify(TryExecuteTask());
		// 从这里进入 TryExecuteTask()


		ReleaseInternalReference(); // named tasks are executed by named threads, outside of the scheduler
	}
};
```




```cpp
// An abstract base class for task implementation. 
// Implements internal logic of task prerequisites, nested tasks and deep task retraction.
// Implements intrusive ref-counting and so can be used with TRefCountPtr.
// It doesn't store task body, instead it expects a derived class to provide a task body as a parameter to `TryExecute` method. @see TExecutableTask
class FTaskBase : private UE::FInheritedContextBase
{

// tries to get execution permission and if successful, executes given task body and completes the task if there're no pending nested tasks. 
// does all required accounting before/after task execution. the task can be deleted as a result of this call.
// @returns true if the task was executed by the current thread
bool TryExecuteTask()
{
	TASKGRAPH_VERBOSE_EVENT_SCOPE(FTaskBase::TryExecuteTask);

	if (!TrySetExecutionFlag())
	{
		return false;
	}

	AddRef(); // `LowLevelTask` will automatically release the internal reference after execution, but there can be pending nested tasks, so keep it alive
	// it's released either later here if the task is closed, or when the last nested task is completed and unlocks its parent (in `TryUnlock`)

	ReleasePrerequisites();

	FTaskBase* PrevTask = ExchangeCurrentTask(this);
	ExecutingThreadId.store(FPlatformTLS::GetCurrentThreadId(), std::memory_order_relaxed);

	if (GetPipe() != nullptr)
	{
		StartPipeExecution();
	}

	{
		UE::FInheritedContextScope InheritedContextScope = RestoreInheritedContext();
		TaskTrace::FTaskTimingEventScope TaskEventScope(GetTraceId());
		TASKGRAPH_VERBOSE_EVENT_SCOPE(FTaskBase::ExecuteTask);
		ExecuteTask();
		// 从这里进入
	}

	if (GetPipe() != nullptr)
	{
		FinishPipeExecution();
	}

	ExecutingThreadId.store(FThread::InvalidThreadId, std::memory_order_relaxed); // no need to sync with loads as they matter only if
	// executed by the same thread
	ExchangeCurrentTask(PrevTask);

	// close the task if there are no pending nested tasks
	uint32 LocalNumLocks = NumLocks.fetch_sub(1, std::memory_order_acq_rel) - 1; // "release" to make task execution "happen before" this, and "acquire" to 
	// "sync with" another thread that completed the last nested task
	if (LocalNumLocks == ExecutionFlag) // unlocked (no pending nested tasks)
	{
		Close();
		Release(); // the internal reference that kept the task alive for nested tasks
	} // else there're non completed nested tasks, the last one will unlock, close and release the parent (this task)

	return true;
}

};
```



### TGraphTask

```cpp
template<typename TTask>
class TGraphTask final : public TConcurrentLinearObject<TGraphTask<TTask>, FTaskGraphBlockAllocationTag>, public FBaseGraphTask
{
/**
 *	This is a helper class returned from the factory. It constructs the embeded task with a set of arguments and sets the task up and makes it ready to execute.
 *	The task may complete before these routines even return.
 **/
class FConstructor
{

virtual void ExecuteTask() override final
{
	FGraphEventRef GraphEventRef{ this };
	TTask* TaskObject = TaskStorage.GetTypedPtr();
	// Only use this scope when GetStatId() is actually implemented on TaskObject for backward compat, sometimes it's not.
	if constexpr (TModels_V<CGetStatIdProvider, TTask>)
	{
		FScopeCycleCounter Scope(TaskObject->GetStatId(), true);
		TaskObject->DoTask(TaskObject->GetDesiredThread(), GraphEventRef);
		// 从这里进入
	}
	else
	{
		TaskObject->DoTask(TaskObject->GetDesiredThread(), GraphEventRef);
	}
	DestructItem(TaskObject);
}

};

};
```



### FTickTaskManager


```cpp
/** Task for a single tick function */
class FTickFunctionTask
{
/**
	 * Actually execute the tick.
	 * @param	CurrentThread; the thread we are running on
	 * @param	MyCompletionGraphEvent; my completion event. Not always useful since at the end of DoWork, you can assume you are done and hence further tasks do not need you as a prerequisite.
	 * However, MyCompletionGraphEvent can be useful for passing to other routines or when it is handy to set up subsequents before you actually do work.
	 */
	void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent)
	{
		if (Context.bLogTick)
		{
			Target->LogTickFunction(CurrentThread, Context.bLogTicksShowPrerequistes);
		}
		if (Target->IsTickFunctionEnabled())
		{
#if DO_TIMEGUARD
			FTimerNameDelegate NameFunction = FTimerNameDelegate::CreateLambda( [&]{ return FString::Printf(TEXT("Slowtick %s "), *Target->DiagnosticMessage()); } );
			SCOPE_TIME_GUARD_DELEGATE_MS(NameFunction, 4);
#endif
			LIGHTWEIGHT_TIME_GUARD_BEGIN(FTickFunctionTask, GTimeguardThresholdMS);

#if UE_WITH_REMOTE_OBJECT_HANDLE
			auto ExecuteTickWork = [this, CurrentThread, &MyCompletionGraphEvent]()
			{
				// !IsCompletionHandleValid is an indication we had previously been ticked this frame and then migrated back
				if (Target->IsCompletionHandleValid() && Target->IsTickFunctionEnabled())
				{
#endif


// 注意前面 UE_WITH_REMOTE_OBJECT_HANDLE 为 0，因此不会包含在 lambda 表达式中
				Target->ExecuteTick(Target->CalculateDeltaTime(Context.DeltaSeconds, Context.World), Context.TickType, CurrentThread, MyCompletionGraphEvent);
				// 从这里进入


#if UE_WITH_REMOTE_OBJECT_HANDLE
				}
			};

			if (Target->bRunTransactionally)
			{
				static FName TransactionalWorkName(TEXT("FTickFunction"));
				UE::RemoteExecutor::ExecuteTransactional(TransactionalWorkName, ExecuteTickWork);
			}
			else
			{
				ExecuteTickWork();
			}
#endif

			LIGHTWEIGHT_TIME_GUARD_END(FTickFunctionTask, Target->DiagnosticMessage());
		}
		Target->ClearTaskInformation();  // This is stale and a good time to clear it for safety
	}
};
```



### FStartPhysicsTickFunction->ExecuteTick

```cpp
void FStartPhysicsTickFunction::ExecuteTick(float DeltaTime, enum ELevelTick TickType, ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent)
{
	QUICK_SCOPE_CYCLE_COUNTER(FStartPhysicsTickFunction_ExecuteTick);
	CSV_SCOPED_TIMING_STAT_EXCLUSIVE(Physics);
	check(Target);
	Target->StartPhysicsSim();
	// 从这里进入
}

```



### UWorld->StartPhysicsSim

```cpp
void UWorld::StartPhysicsSim()
{
	FPhysScene* PhysScene = GetPhysicsScene();
	if (PhysScene == NULL)
	{
		return;
	}

	PhysScene->StartFrame();
	// 从这里进入
}
```



### FChaosScene->StartFrame

```cpp
void FChaosScene::StartFrame()
{
	using namespace Chaos;

	SCOPE_CYCLE_COUNTER(STAT_Scene_StartFrame);

	if(CVar_ChaosSimulationEnable.GetValueOnGameThread() == 0)
	{
		return;
	}

	const float UseDeltaTime = OnStartFrame(MDeltaTime);

	TArray<FPhysicsSolverBase*> SolverList = GetPhysicsSolvers();
	for(FPhysicsSolverBase* Solver : SolverList)
	{
	// 进入 AdvanceAndDispatch_External
		if(FGraphEventRef SolverEvent = Solver->AdvanceAndDispatch_External(UseDeltaTime))
		{
			if(SolverEvent.IsValid())
			{
				CompletionEvents.Add(SolverEvent);
			}
		}
	}
}
```



### FPhysicsSolverBase

这个方法即使在不运行游戏时也会调用

```cpp
FGraphEventRef FPhysicsSolverBase::AdvanceAndDispatch_External(FReal InDt)
	{
		LLM_SCOPE(ELLMTag::ChaosScene);
		const bool bSubstepping = MMaxSubSteps > 1;
		SetSolverSubstep_External(bSubstepping);
		const FReal DtWithPause = bPaused_External ? 0.0f : InDt;
		FReal InternalDt = DtWithPause;
		int32 NumSteps = 1;

		if(IsUsingFixedDt())
		{
			AccumulatedTime += DtWithPause;
			if(InDt == 0)	//this is a special flush case
			{
				//just use any remaining time and sync up to latest no matter what
				InternalDt = AccumulatedTime;
				NumSteps = 1;
				AccumulatedTime = 0;
			}
			else
			{

				InternalDt = AsyncDt;
				NumSteps = FMath::FloorToInt32(AccumulatedTime / InternalDt);
				AccumulatedTime -= InternalDt * static_cast<FReal>(NumSteps);
			}
		}
		else if (bSubstepping && InDt > 0)
		{
			NumSteps = FMath::CeilToInt32(DtWithPause / MMaxDeltaTime);
			if (NumSteps > MMaxSubSteps)
			{
				// Hitting this case means we're losing time, given the constraints of MaxSteps and MaxDt we can't
				// fully handle the Dt requested, the simulation will appear to the viewer to run slower than realtime
				NumSteps = MMaxSubSteps;
				InternalDt = MMaxDeltaTime;
			}
			else
			{
				InternalDt = DtWithPause / static_cast<FReal>(NumSteps);
			}
		}

		if(InDt > 0)
		{
			ExternalSteps++;	//we use this to average forces. It assumes external dt is about the same. 0 dt should be ignored as it typically has nothing to do with force
		}

		// Eventually drop physics steps in mode 2
		if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
		{
			// Make sure not to accumulate too many physics solver tasks.
			constexpr int32 MaxPhysicsStepToKeep = 3;
			const int32 MaxNumSteps = MaxPhysicsStepToKeep - NumPendingSolverAdvanceTasks;
			if (NumSteps > MaxNumSteps)
			{
				CSV_CUSTOM_STAT(ChaosPhysicsSolver, PhysicsFrameDropped, NumSteps - MaxNumSteps, ECsvCustomStatOp::Accumulate);
				// NumSteps + NumPendingSolverAdvanceTasks shouldn't be bigger than MaxPhysicsStepToKeep
				NumSteps = FMath::Min<int32>(NumSteps, MaxNumSteps);
			}
		}
			
		if (NumSteps > 0)
		{
			//make sure any GT state is pushed into necessary buffer
			PushPhysicsState(InternalDt, NumSteps, FMath::Max(ExternalSteps, 1));
			ExternalSteps = 0;
		}

		// If standalone solver we are not responsible to spawn tasks
		if(IsStandaloneSolver())
		{
			return {};
		}

		// Ensures we block on any tasks generated from previous frames
		FGraphEventRef BlockingTasks = PendingTasks;

		while(FPushPhysicsData* PushData = MarshallingManager.StepInternalTime_External())
		{
			if(MRewindCallback && !bIsShuttingDown)
			{
				MRewindCallback->ProcessInputs_External(PushData->InternalStep, PushData->SimCallbackInputs);
			}

			if(ThreadingMode == EThreadingModeTemp::SingleThread)
			{
				ensure(!PendingTasks || PendingTasks->IsComplete());	//if mode changed we should have already blocked
				FAllSolverTasks ImmediateTask(*this, PushData);
#if !UE_BUILD_SHIPPING
				if(bStealAdvanceTasksForTesting)
				{
					StolenSolverAdvanceTasks.Emplace(MoveTemp(ImmediateTask));
				}
				else
				{
					ImmediateTask.AdvanceSolver();
				}
#else
				ImmediateTask.AdvanceSolver();
#endif
			}
			else
			{
				// If enabled, block on all but most recent physics task, even tasks generated this frame.
				if (AsyncBlockMode == EAsyncBlockMode::BlockForBestInterpolation)
				{
					BlockingTasks = PendingTasks;
				}

				FGraphEventArray Prereqs;
				if (PendingTasks && !PendingTasks->IsComplete())
				{
					Prereqs.Add(PendingTasks);
				}

				PendingTasks = TGraphTask<FPhysicsSolverProcessPushDataTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);
				Prereqs.Add(PendingTasks);

				if (bSolverHasFrozenGameThreadCallbacks)
				{
					PendingTasks = TGraphTask<FPhysicsSolverFrozenGTPreSimCallbacks>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this);
					Prereqs.Add(PendingTasks);
				}

				PendingTasks = TGraphTask<FPhysicsSolverAdvanceTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);

				if (IsUsingAsyncResults() == false)
				{
					BlockingTasks = PendingTasks;	//block right away
				}
			}

			// This break is mainly here to satisfy unit testing. The call to StepInternalTime_External will decrement the
			// delay in the marshaling manager and throw of tests that are explicitly testing for propagation delays
			if (IsUsingAsyncResults() == false && !bSubstepping)
			{
				break;
			}
		}
		if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
		{
			return {};
		}
		return BlockingTasks;
	}
```



### FChaosScene->EndFrame

```cpp

void FChaosScene::EndFrame()
{
	using namespace Chaos;
	using SpatialAccelerationCollection = ISpatialAccelerationCollection<FAccelerationStructureHandle,FReal,3>;

	SCOPE_CYCLE_COUNTER(STAT_Scene_EndFrame);

	if(CVar_ChaosSimulationEnable.GetValueOnGameThread() == 0 || GetSolver() == nullptr)
	{
		return;
	}

	CVD_TRACE_ACCELERATION_STRUCTURES(SolverAccelerationStructure, Chaos::FPhysicsSolver, *SceneSolver, CVDDC_AccelerationStructures);

#if !UE_BUILD_SHIPPING
	{
		Chaos::AABBTreeStatistics TreeStats;
		Chaos::AABBTreeExpensiveStatistics TreeExpensiveStats;
		GetAABBTreeStats(GetSpacialAcceleration()->AsChecked<SpatialAccelerationCollection>(), TreeStats, TreeExpensiveStats);

		CSV_CUSTOM_STAT(ChaosPhysics, AABBTreeDirtyElementCount, TreeStats.StatNumDirtyElements, ECsvCustomStatOp::Set);
		SET_DWORD_STAT(STAT_ChaosCounter_NumDirtyAABBTreeElements, TreeStats.StatNumDirtyElements);

		CSV_CUSTOM_STAT(ChaosPhysics, AABBTreeDirtyGridOverflowCount, TreeStats.StatNumGridOverflowElements, ECsvCustomStatOp::Set);
		SET_DWORD_STAT(STAT_ChaosCounter_NumDirtyGridOverflowElements, TreeStats.StatNumGridOverflowElements);

		CSV_CUSTOM_STAT(ChaosPhysics, AABBTreeDirtyElementTooLargeCount, TreeStats.StatNumElementsTooLargeForGrid, ECsvCustomStatOp::Set);
		SET_DWORD_STAT(STAT_ChaosCounter_NumDirtyElementsTooLargeForGrid, TreeStats.StatNumElementsTooLargeForGrid);

		CSV_CUSTOM_STAT(ChaosPhysics, AABBTreeDirtyElementNonEmptyCellCount, TreeStats.StatNumNonEmptyCellsInGrid, ECsvCustomStatOp::Set);
		SET_DWORD_STAT(STAT_ChaosCounter_NumDirtyNonEmptyCellsInGrid, TreeStats.StatNumNonEmptyCellsInGrid);

#if CSV_PROFILER_STATS
		if (FCsvProfiler::Get()->IsCapturing() && FCsvProfiler::Get()->IsCategoryEnabled(CSV_CATEGORY_INDEX(AABBTreeExpensiveStats)))
		{
			CSV_CUSTOM_STAT(AABBTreeExpensiveStats, AABBTreeMaxNumLeaves, TreeExpensiveStats.StatMaxNumLeaves, ECsvCustomStatOp::Set);
			CSV_CUSTOM_STAT(AABBTreeExpensiveStats, AABBTreeMaxDirtyElements, TreeExpensiveStats.StatMaxDirtyElements, ECsvCustomStatOp::Set);
			CSV_CUSTOM_STAT(AABBTreeExpensiveStats, AABBTreeMaxTreeDepth, TreeExpensiveStats.StatMaxTreeDepth, ECsvCustomStatOp::Set);
			CSV_CUSTOM_STAT(AABBTreeExpensiveStats, AABBTreeMaxLeafSize, TreeExpensiveStats.StatMaxLeafSize, ECsvCustomStatOp::Set);
			CSV_CUSTOM_STAT(AABBTreeExpensiveStats, AABBTreeGlobalPayloadsSize, TreeExpensiveStats.StatGlobalPayloadsSize, ECsvCustomStatOp::Set);
		}
#endif // CSV_PROFILER_STATS
	}
#endif // UE_BUILD_SHIPPING

	check(IsCompletionEventComplete())
	//check(PhysicsTickTask->IsComplete());
	CompletionEvents.Reset();

	// Make a list of solvers to process. This is a list of all solvers registered to our world
	// And our internal base scene solver.
	TArray<FPhysicsSolverBase*> SolverList = GetPhysicsSolvers();

	// Flip the buffers over to the game thread and sync
	{
		SCOPE_CYCLE_COUNTER(STAT_FlipResults);

		//update external SQ structure
		//for now just copy the whole thing, stomping any changes that came from GT
		CopySolverAccelerationStructure();

		for(FPhysicsSolverBase* Solver : SolverList)
		{
			Solver->CastHelper([&SolverList, Solver, this](auto& Concrete)
			{
				// 同步操作，从这里进入
				SyncBodies(&Concrete);


				Solver->FlipEventManagerBuffer();
				Concrete.SyncEvents_GameThread();
				{
					SCOPE_CYCLE_COUNTER(STAT_SqUpdateMaterials);
					Concrete.SyncQueryMaterials_External();
				}
			});
		}
	}

	OnPhysScenePostTick.Broadcast(this);
}
```



### FChaosScene->SyncBodies

```cpp
template <typename TSolver>
void FChaosScene::SyncBodies(TSolver* Solver)
{
	DECLARE_SCOPE_CYCLE_COUNTER(TEXT("SyncBodies"),STAT_SyncBodies,STATGROUP_Physics);
	CSV_SCOPED_TIMING_STAT_EXCLUSIVE(SyncBodies);
	OnSyncBodies(Solver);
}
```



### FPhysScene_Chaos->OnSyncBodies

其中调用 `PullPhysicsStateForEachDirtyProxy_External` 获取新的物理数据

```cpp

void FPhysScene_Chaos::OnSyncBodies(Chaos::FPhysicsSolverBase* Solver)
{
	using namespace Chaos;

	struct FDispatcher
	{
		FPhysScene_Chaos* Outer;
		FPhysScene* PhysScene;
		Chaos::FPhysicsSolverBase* Solver;
		TArray<FPhysScenePendingComponentTransform_Chaos> PendingTransforms;

		void operator()(FSingleParticlePhysicsProxy* Proxy)
		{
			SCOPE_CYCLE_COUNTER(STAT_SyncBodiesSingleParticlePhysicsProxy);
			FPBDRigidParticle* DirtyParticle = Proxy->GetRigidParticleUnsafe();

			if (FBodyInstance* BodyInstance = ChaosInterface::GetUserData(*(Proxy->GetParticle_LowLevel())))
			{
				SCOPE_CYCLE_UOBJECT(Component, GEmitSyncBodiesUObjectStats ? BodyInstance->OwnerComponent.Get() : nullptr);

				// Skip kinematics, unless they've been flagged to update following simulation
				if (!BodyInstance->bUpdateKinematicFromSimulation && BodyInstance->IsInstanceSimulatingPhysics() == false)
				{
					return;
				}

				if (UPrimitiveComponent* OwnerComponent = BodyInstance->OwnerComponent.Get())
				{
					bool bPendingMove = false;
					FRigidTransform3 NewTransform(DirtyParticle->X(), DirtyParticle->R());

					if (!NewTransform.EqualsNoScale(OwnerComponent->GetComponentTransform()))
					{
						if (BodyInstance->InstanceBodyIndex == INDEX_NONE)
						{

							bPendingMove = true;
							const FVector MoveBy = NewTransform.GetLocation() - OwnerComponent->GetComponentTransform().GetLocation();
							const FQuat NewRotation = NewTransform.GetRotation();
							PendingTransforms.Add(FPhysScenePendingComponentTransform_Chaos(OwnerComponent, MoveBy, NewRotation, Proxy->GetWakeEvent()));

						}

						PhysScene->UpdateActorInAccelerationStructure(BodyInstance->GetPhysicsActor());
					}

					if (Proxy->GetWakeEvent() != Chaos::EWakeEventEntry::None && !bPendingMove)
					{
						PendingTransforms.Add(FPhysScenePendingComponentTransform_Chaos(OwnerComponent, Proxy->GetWakeEvent()));
					}
					Proxy->ClearEvents();
				}
			}
		}

		void operator()(FJointConstraintPhysicsProxy* Proxy)
		{
			SCOPE_CYCLE_COUNTER(STAT_SyncBodiesJointConstraintPhysicsProxy);
			Chaos::FJointConstraint* Constraint = Proxy->GetConstraint();

			if (Constraint->GetOutputData().bIsBreaking)
			{
				if (FConstraintInstanceBase* ConstraintInstance = (Constraint) ? FPhysicsUserData_Chaos::Get<FConstraintInstanceBase>(Constraint->GetUserData()) : nullptr)
				{
					FConstraintBrokenDelegateWrapper CBD(ConstraintInstance);
					CBD.DispatchOnBroken();
				}

				Constraint->GetOutputData().bIsBreaking = false;
			}

			if (Constraint->GetOutputData().bIsViolating)
			{
				if (FConstraintInstanceBase* ConstraintInstance = (Constraint) ? FPhysicsUserData_Chaos::Get<FConstraintInstanceBase>(Constraint->GetUserData()) : nullptr)
				{
					FConstraintViolatedDelegateWrapper CVD(ConstraintInstance);
					CVD.DispatchOnViolated(Constraint->GetOutputData().LinearViolation, Constraint->GetOutputData().AngularViolation);
				}

				Constraint->GetOutputData().bIsViolating = false;
			}

			if (Constraint->GetOutputData().bDriveTargetChanged)
			{
				if (FConstraintInstanceBase* ConstraintInstance = (Constraint) ? FPhysicsUserData_Chaos::Get<FConstraintInstanceBase>(Constraint->GetUserData()) : nullptr)
				{
					FPlasticDeformationDelegateWrapper CPD(ConstraintInstance);
					CPD.DispatchPlasticDeformation();
				}

				Constraint->GetOutputData().bDriveTargetChanged = false;
			}
		}

		void operator()(FGeometryCollectionPhysicsProxy* Proxy)
		{
			if (Proxy && Outer->GetSpacialAcceleration())
			{
				UpdateAccelerationStructureFromGeometryCollectionProxy(*Proxy, *Outer->GetSpacialAcceleration());
			}
		}

		void operator()(FClusterUnionPhysicsProxy* Proxy)
		{
			SCOPE_CYCLE_COUNTER(STAT_SyncBodiesClusterUnionPhysicsProxy);
			FLockedWritePhysicsObjectExternalInterface Interface = FPhysicsObjectExternalInterface::LockWrite({});
			Chaos::FPhysicsObjectHandle Handle = Proxy->GetPhysicsObjectHandle();

			// Cluster unions should always have their owner be the cluster union component.
			FClusterUnionPhysicsProxy::FExternalParticle* DirtyParticle = Proxy->GetParticle_External();

			// Use the scene's GetOwningComponent rather than UserData since it's guaranteed to be safe and correspond to the GT state of the component.
			// i.e. if we destroy the component's physics state and then call sync bodies (not entirely impossible), we don't try to add the cluster union
			// back into the acceleration structure (since destroying the physics state already took it out). In that case, we could potentially end up with
			// a deleted cluster union particle in the SQ.
			if (UClusterUnionComponent* ParentComponent = Outer->GetOwningComponent<UClusterUnionComponent>(Proxy))
			{
				const FRigidTransform3 NewTransform(DirtyParticle->X(), DirtyParticle->R());

				bool bHasMoved = false;
				if (!NewTransform.EqualsNoScale(ParentComponent->GetComponentTransform()))
				{
					bHasMoved = true;
					const FVector MoveBy = NewTransform.GetLocation() - ParentComponent->GetComponentTransform().GetLocation();
					PendingTransforms.Add(FPhysScenePendingComponentTransform_Chaos(ParentComponent, MoveBy, NewTransform.GetRotation(), DirtyParticle->GetWakeEvent()));
				}

				if (DirtyParticle->GetWakeEvent() != Chaos::EWakeEventEntry::None && !bHasMoved)
				{
					PendingTransforms.Add(FPhysScenePendingComponentTransform_Chaos(ParentComponent, DirtyParticle->GetWakeEvent()));
				}

				if (bHasMoved || !bGClusterUnionSyncBodiesMoveNewComponents)
				{
					ParentComponent->SyncClusterUnionFromProxy(NewTransform, nullptr);
				}
				else
				{
					// We must to call MoveComponent on any newly added component. The Cluster Union will take care of
					// that but only if it moved. If it did not move we need to handle it manually
					TArray<TTuple<UPrimitiveComponent*, FTransform>> NewComponents;
					ParentComponent->SyncClusterUnionFromProxy(NewTransform, &NewComponents);

					for (TTuple<UPrimitiveComponent*, FTransform>& NewComponentTuple : NewComponents)
					{
						UPrimitiveComponent* NewComponent = NewComponentTuple.Get<0>();
						const FTransform& NewComponentTransform = NewComponentTuple.Get<1>();
						const FVector NewComponentCurrentLocation = NewComponent->GetComponentLocation();
						const FVector NewComponentDelta = NewComponentTransform.GetLocation() - NewComponentCurrentLocation;

						PendingTransforms.Add(FPhysScenePendingComponentTransform_Chaos(NewComponent, NewComponentDelta, NewComponentTransform.GetRotation(), DirtyParticle->GetWakeEvent()));
					}
				}

				// make sure we have at least a child to be added to the acceleration structure 
				// this avoid the invalid bounds to cause the particle to be added to the global acceleration structure array
				bool bShouldBeInSQ = false;

				if (FImplicitObjectRef GeometryRef = DirtyParticle->GetGeometry())
				{
					if (FImplicitObjectUnion* Union = GeometryRef->AsA<FImplicitObjectUnion>())
					{
						bShouldBeInSQ = (Union->GetNumRootObjects() > 0);
					}
				}

				if (bShouldBeInSQ)
				{
					Interface->AddToSpatialAcceleration({ &Handle, 1 }, Outer->GetSpacialAcceleration());
				}
				else
				{
					Interface->RemoveFromSpatialAcceleration({ &Handle, 1 }, Outer->GetSpacialAcceleration());
				}

				DirtyParticle->ClearEvents();
			}
		}
	};

	FDispatcher Dispatcher;
	FPhysicsCommand::ExecuteWrite(this, [this, Solver, &Dispatcher](FPhysScene* PhysScene)
	{
		Dispatcher.Outer = this;
		Dispatcher.PhysScene = PhysScene;
		Dispatcher.Solver = Solver;
		Solver->PullPhysicsStateForEachDirtyProxy_External(Dispatcher);
	});

	for (const FPhysScenePendingComponentTransform_Chaos& ComponentTransform : Dispatcher.PendingTransforms)
	{
		if (ComponentTransform.OwningComp != nullptr)
		{
			AActor* OwnerPtr = ComponentTransform.OwningComp->GetOwner();

			if (ComponentTransform.bHasValidTransform)
			{
				ComponentTransform.OwningComp->MoveComponent(ComponentTransform.NewTranslation, ComponentTransform.NewRotation, false, NULL, MOVECOMP_SkipPhysicsMove);
			}

			if (IsValid(OwnerPtr))
			{
				OwnerPtr->CheckStillInWorld();
			}
		}

		if (ComponentTransform.OwningComp != nullptr)
		{
			if (ComponentTransform.WakeEvent != Chaos::EWakeEventEntry::None)
			{
				ComponentTransform.OwningComp->DispatchWakeEvents(ComponentTransform.WakeEvent == EWakeEventEntry::Awake ? ESleepEvent::SET_Wakeup : ESleepEvent::SET_Sleep, NAME_None);
			}
		}
	}
}
```



## 相关类和函数

### 用到 bTickPhysicsAsync 的函数

#### UPhysicsSettings

控制是否开启 Tick Physics Async

```cpp
UCLASS(config=Engine, defaultconfig, meta=(DisplayName="Physics"), MinimalAPI)
class UPhysicsSettings : public UPhysicsSettingsCore
{

/** Whether to tick physics simulation on an async thread. This feature is still experimental. Certain functionality might not work correctly*/
UPROPERTY(config, EditAnywhere, Category = Framerate)
bool bTickPhysicsAsync;

};
```



#### UWorld

成员变量

```cpp
UPhysicsSettings* Settings = UPhysicsSettings::Get();
```

设置物理 Tick

```cpp
void UWorld::SetupPhysicsTickFunctions(float DeltaSeconds)
{
	// ...


	/* When using physics prediction, allow max delta time at least equal to max supported latency for physics prediction
	* NOTE: These values clamp how much game thread delta time that can accumulate towards ticking the fixed physics steps, 
	* if we clamp this accumulation too much we hinder time dilation from correcting desyncs and we also make the client (and server) more prone to desyncing physics due to dropping physics steps. */

	// 这里用到 bTickPhysicsAsync
	if (Settings->PhysicsPrediction.bEnablePhysicsPrediction && Settings->bTickPhysicsAsync)
	{
		MinPhysicsDeltaTime = 0.0f;
		MaxPhysicsDeltaTime = MaxPhysicsDeltaTime <= UE_SMALL_NUMBER ? 0.0f : FMath::Max((Settings->PhysicsPrediction.MaxSupportedLatencyPrediction / 1000.0f), MaxPhysicsDeltaTime);
		MaxSubstepDeltaTime = MaxSubstepDeltaTime <= UE_SMALL_NUMBER ? 0.0f : FMath::Max((Settings->PhysicsPrediction.MaxSupportedLatencyPrediction / 1000.0f) / Settings->MaxSubsteps, MaxSubstepDeltaTime);
	}

	PhysScene->SetUpForFrame(&DefaultGravity, DeltaSeconds, MinPhysicsDeltaTime, MaxPhysicsDeltaTime, MaxSubstepDeltaTime, Settings->MaxSubsteps, Settings->bSubstepping);
}
```



#### FPhysScene_Chaos

这里运行游戏时才会进入。`FPhysScene_Chaos` 派生自 `FChaosScene`，注意父类构造过程

```cpp
FPhysScene_Chaos::FPhysScene_Chaos(AActor* InSolverActor
#if CHAOS_DEBUG_NAME
	, const FName& DebugName
#endif
)
// 如果开启异步，第二个参数为 AsyncFixedTimeStepSize，否则为 -1
	: Super(InSolverActor ? InSolverActor->GetWorld() : nullptr
		, UPhysicsSettings::Get()->bTickPhysicsAsync ? UPhysicsSettings::Get()->AsyncFixedTimeStepSize : -1
#if CHAOS_DEBUG_NAME
		, DebugName
#endif
	)
	// ...
{
// ...
}
```



#### FChaosScene

在这里初始化 Chaos

- InAsyncDt 设置异步时间步长 AsyncFixedTimeStepSize，不开启时则为 -1

```cpp
FChaosScene::FChaosScene(
	UObject* OwnerPtr
	, Chaos::FReal InAsyncDt
	, const FName& DebugName
)
	: SolverAccelerationStructure(nullptr)
	, ChaosModule(nullptr)
	, SceneSolver(nullptr)
	, Owner(OwnerPtr)
{
// ...

// 创建求解器，用到异步步长
	SceneSolver = ChaosModule->CreateSolver(OwnerPtr, InAsyncDt, ThreadingMode, DebugName);
	check(SceneSolver);

#if WITH_CHAOS_VISUAL_DEBUGGER
	SceneSolver->GetChaosVDContextData().OwnerID = GetChaosVDContextData().Id;
	SceneSolver->GetChaosVDContextData().Id = FChaosVDRuntimeModule::Get().GenerateUniqueID();
	SceneSolver->GetChaosVDContextData().Type = static_cast<int32>(EChaosVDContextType::Solver);
#endif

	SceneSolver->PhysSceneHack = this;
	SimCallback = SceneSolver->CreateAndRegisterSimCallbackObject_External<FChaosSceneSimCallback>();

	// Apply project settings to the solver
	if (CVar_ApplyProjectSettings.GetValueOnAnyThread() != 0)
	{
		UPhysicsSettingsCore* Settings = UPhysicsSettingsCore::Get();
		SceneSolver->ApplyConfig(Settings->SolverOptions);
		SceneSolver->SetIsDeterministic(Settings->bEnableEnhancedDeterminism);
	}

	// Make sure we have initialized structure on game thread, evolution has already initialized structure, just need to copy.
	CopySolverAccelerationStructure();

}
```



#### FChaosSolversModule

CreateSolver 进入当前函数

```cpp

Chaos::FPBDRigidsSolver* FChaosSolversModule::CreateSolver(UObject* InOwner, Chaos::FReal InAsyncDt, Chaos::EThreadingMode InThreadingMode, const FName& DebugName)
{
	LLM_SCOPE(ELLMTag::Chaos);
	using namespace Chaos;

	FChaosScopeSolverLock SolverScopeLock;
	
	EMultiBufferMode SolverBufferMode = InThreadingMode == EThreadingMode::SingleThread ? EMultiBufferMode::Single : EMultiBufferMode::Double;

	// 从这里创建 Solver，并使用异步步长
	FPBDRigidsSolver* NewSolver = new FPBDRigidsSolver(SolverBufferMode, InOwner, InAsyncDt);
	AllSolvers.Add(NewSolver);

	// Add The solver to the owner list
	TArray<FPhysicsSolverBase*>& OwnerSolverList = SolverMap.FindOrAdd(InOwner);
	OwnerSolverList.Add(NewSolver);

#if CHAOS_DEBUG_NAME
    // Add solver number to solver name
	const FName NewDebugName = *FString::Printf(TEXT("%s (%d)"), DebugName == NAME_None ? TEXT("Solver") : *DebugName.ToString(), AllSolvers.Num() - 1);
	NewSolver->SetDebugName(NewDebugName);
#endif

	// Set up the material lists on the new solver, copying from the current primary list
	{
		FPhysicalMaterialManager& Manager =	Chaos::FPhysicalMaterialManager::Get();
		FPhysicsSceneGuardScopedWrite ScopedWrite(NewSolver->GetExternalDataLock_External());
		NewSolver->QueryMaterials_External = Manager.GetPrimaryMaterials_External();
		NewSolver->QueryMaterialMasks_External = Manager.GetPrimaryMaterialMasks_External();
		NewSolver->SimMaterials = Manager.GetPrimaryMaterials_External();
		NewSolver->SimMaterialMasks = Manager.GetPrimaryMaterialMasks_External();
	}

	return NewSolver;
}
```

调用构造函数

```cpp

FPBDRigidsSolver::FPBDRigidsSolver(const EMultiBufferMode BufferingModeIn, UObject* InOwner, FReal InAsyncDt)
	// 在这里使用异步步长
	: Super(BufferingModeIn, BufferingModeIn == EMultiBufferMode::Single ? EThreadingModeTemp::SingleThread : EThreadingModeTemp::TaskGraph, InOwner, InAsyncDt)
	// ...
{
	// ...
}

```

进入父类 FPhysicsSolverBase 构造函数。



#### FPhysicsSolverBase

注意 AsyncPhysicsBlockMode 设置；有一个 Log 可以找一下

```cpp
// 0 blocks on any physics steps generated from past GT Frames, and blocks on none of the tasks from current frame.
// 1 blocks on everything except the single most recent task (including tasks from current frame)
// 1 should guarantee we will always have a future output for interpolation from 2 frames in the past
// 2 doesn't block the game thread. Physics steps could be eventually be dropped if taking too much time.
int32 AsyncPhysicsBlockMode = 0;
FAutoConsoleVariableRef CVarAsyncPhysicsBlockMode(TEXT("p.AsyncPhysicsBlockMode"), AsyncPhysicsBlockMode, TEXT("Setting to 0 blocks on any physics steps generated from past GT Frames, and blocks on none of the tasks from current frame."
	" 1 blocks on everything except the single most recent task (including tasks from current frame). 1 should gurantee we will always have a future output for interpolation from 2 frames in the past."
	" 2 doesn't block the game thread, physics steps could be eventually be dropped if taking too much time."), LambdaAsyncMode);

FPhysicsSolverBase::FPhysicsSolverBase(const EMultiBufferMode BufferingModeIn,const EThreadingModeTemp InThreadingMode,UObject* InOwner, Chaos::FReal InAsyncDt)
		: BufferMode(BufferingModeIn)
		// ...
		// 保存异步步长
		, AsyncDt(InAsyncDt)
		// ...
	{
	// 这里可以看异步步长
		UE_LOG(LogChaos, Verbose, TEXT("FPhysicsSolverBase::AsyncDt:%f"), IsUsingAsyncResults() ? AsyncDt : -1);

		//If user is running with -PhysicsRunsOnGT override the cvar (doing it here to avoid parsing every time task is scheduled)
		if(FParse::Param(FCommandLine::Get(), TEXT("PhysicsRunsOnGT")))
		{
			PhysicsRunsOnGT = 1;
		}
	}
```



### 用到 AsyncDt 的函数

#### AdvanceAndDispatch_External



#### GetPhysicsResultsTime_External

FPhysicsSolverBase 中的获得物理耗时函数

```cpp
/** Returns the time used by physics results. If fixed dt is used this will be the interpolated time */
FReal GetPhysicsResultsTime_External() const
{
	const FReal ExternalTime = MarshallingManager.GetExternalTime_External() + AccumulatedTime;
	if (IsUsingFixedDt())
	{
		//fixed dt uses interpolation and looks into the past
		return ExternalTime - AsyncDt * AsyncMultiplier;
	}
	else
	{
		return ExternalTime;
	}
}
```



#### IsUsingAsyncResults

FPhysicsSolverBase 中的判断是否使用异步

```cpp
bool IsUsingAsyncResults() const
{
	// ForceDisableAsyncPhysics 一般为 0，因此只需要 AsyncDt 非负
	return !ForceDisableAsyncPhysics && AsyncDt >= 0;
}
```



#### IsUsingFixedDt

它调用了 IsUsingAsyncResults

```cpp
bool IsUsingFixedDt() const
{
	// UseAsyncInterpolation 恒为 1，因此它与 IsUsingAsyncResults 等价
	return IsUsingAsyncResults() && UseAsyncInterpolation;
}
```

> >注意这里用到 `UseAsyncInterpolation`，作为 FPhysicsSolverBase 的默认值为 `CHAOS_API int32 UseAsyncInterpolation = 1;`



#### GetAsyncDeltaTime

FPhysicsSolverBase 中的步长获取

```cpp
FReal GetAsyncDeltaTime() const { return AsyncDt; } 
```



#### Enable/Disable

```cpp
void FPhysicsSolverBase::EnableAsyncMode(FReal FixedDt)
{
	AsyncDt = FixedDt;
	UE_LOG(LogChaos, Verbose, TEXT("FPhysicsSolverBase::AsyncDt:%f"), IsUsingAsyncResults() ? AsyncDt : -1);
}

void FPhysicsSolverBase::DisableAsyncMode()
{
	AsyncDt = -1;
	UE_LOG(LogChaos, Verbose, TEXT("FPhysicsSolverBase::AsyncDt:%f"), AsyncDt);
}
```



### 用到 IsUsingAsyncResults 的函数

#### UChaosMoverBackendComponent

UChaosMoverBackendComponent 构造函数

```cpp

UChaosMoverBackendComponent::UChaosMoverBackendComponent()
	: PathTargetConstraintPhysicsUserData(&PathTargetConstraintInstance)
{
	PrimaryComponentTick.bCanEverTick = false;

	bWantsInitializeComponent = true;
	bAutoActivate = true;

	if (const Chaos::FPhysicsSolver* Solver = GetPhysicsSolver())
	{
		bIsUsingAsyncPhysics = Solver->IsUsingAsyncResults();
	}
}
```



#### FBodyInstance

更新时长

```cpp

void FBodyInstance::UpdateSolverAsyncDeltaTime()
{
	if (bOverrideSolverAsyncDeltaTime && !bEnableOverrideSolverDeltaTime)
	{
		UE_LOG(LogPhysics, Warning, TEXT("FBodyInstance::SolverAsyncDeltaTime : Ignoring parameter because overriden by p.EnableOverrideSolverDeltaTime"));
		return;
	}

	if (bOverrideSolverAsyncDeltaTime)
	{
		if (SolverAsyncDeltaTime < 0.f)
		{
			UE_LOG(LogPhysics, Error, TEXT("FBodyInstance::SolverAsyncDeltaTime : Value must be greater than 0"));			
			return;
		}

		Chaos::FPhysicsSolverBase* Solver = GetPhysicsActor()->GetSolverBase();

		// 如果使用异步，则在后面启用异步模式
		if (!Solver->IsUsingAsyncResults())
		{
			UE_LOG(LogPhysics, Warning, TEXT("FBodyInstance::SolverAsyncDeltaTime : Ignoring parameter because solver is not setup to use async results"));
			return;
		}
		else
		{
			float OldAsyncDeltaTime = Solver->GetAsyncDeltaTime();

			// set async delta time to minimum of delta time and what this body wants
			Solver->EnableAsyncMode(FMath::Min<float>(Solver->GetAsyncDeltaTime(), SolverAsyncDeltaTime));
		}
	}
}
```



#### AdvanceAndDispatch_External




### 用到 bIsUsingAsyncPhysics 的函数

创建 NetworkPhysicsComponent 启用异步物理

```cpp

void UChaosMoverBackendComponent::InitializeComponent()
{
	Super::InitializeComponent();

	if (UWorld* World = GetWorld(); World && World->IsGameWorld())
	{
		NullMovementMode = NewObject<UNullMovementMode>(&GetMoverComponent(), TEXT("NullMovementMode"));
		ImmediateModeTransition = NewObject<UImmediateMovementModeTransition>(&GetMoverComponent(), TEXT("ImmediateModeTransition"));

		// Create NetworkPhysicsComponent
		if ((World->GetNetMode() != ENetMode::NM_Standalone) && Chaos::FPhysicsSolverBase::IsNetworkPhysicsPredictionEnabled())
		{
			if (!bIsUsingAsyncPhysics)
			{
				// Verify that the Project Settings have bTickPhysicsAsync turned on.
				// It's easy to waste time forgetting that, since they are off by default.
				UE_LOG(LogChaosMover, Warning, TEXT("Chaos Mover Backend only supports networking with Physics Async. Networked Physics will not work well. Turn on 'Project Settings > Engine - Physics > Tick Physics Async', or play in Standalone Mode"));
				// This is important enough that we break for developers debugging in editor
				UE_DEBUG_BREAK();
			}
			else
			{
				NetworkPhysicsComponent = NewObject<UNetworkPhysicsComponent>(GetOwner(), TEXT("PhysMover_NetworkPhysicsComponent"));

				// This isn't technically a DSO component, but set it net addressable as though it is
				NetworkPhysicsComponent->SetNetAddressable();
				NetworkPhysicsComponent->SetIsReplicated(true);
				NetworkPhysicsComponent->RegisterComponent();
				if (!NetworkPhysicsComponent->HasBeenInitialized())
				{
					NetworkPhysicsComponent->InitializeComponent();
				}
				NetworkPhysicsComponent->Activate(true);

				// Register network data for recording and rewind/resim
				NetworkPhysicsComponent->CreateDataHistory<UE::ChaosMover::FNetworkDataTraits>(this);

				if (NetworkPhysicsComponent->HasServerWorld())
				{
					if (APawn* PawnOwner = Cast<APawn>(GetOwner()))
					{
						// When we're owned by a pawn, keep an eye on whether it's currently player-controlled or not
						PawnOwner->ReceiveControllerChangedDelegate.AddUniqueDynamic(this, &ThisClass::HandleOwningPawnControllerChanged_Server);
						HandleOwningPawnControllerChanged_Server(PawnOwner, nullptr, PawnOwner->Controller);
					}
					else
					{
						// If the owner isn't a pawn, there's no chance of player input happening, so inputs to the PT are always produced on the server
						NetworkPhysicsComponent->SetIsRelayingLocalInputs(true);
					}
				}
			}
		}
	}
}
```



### 用到 IsUsingFixedDt 的函数

#### AdvanceAndDispatch_External



#### FPhysicsReplicationAsync

这里对固定步长的判断似乎与网络有关，使用预计算物理

```cpp

void FPhysicsReplicationAsync::OnPreSimulate_Internal()
{
	if (FPhysicsReplication::ShouldSkipPhysicsReplication())
	{
		return;
	}

	Chaos::FPBDRigidsSolver* RigidsSolver = static_cast<Chaos::FPBDRigidsSolver*>(GetSolver());
	check(RigidsSolver);

	// Early out if this is a resim frame
	Chaos::FRewindData* RewindData = RigidsSolver->GetRewindData();
	const bool bRewindDataExist = RewindData != nullptr;
	if (bRewindDataExist && RewindData->IsResim())
	{
		// TODO, Handle the transition from post-resim to interpolation better (disabled by default, resim vs replication interaction is handled via FPhysicsReplicationAsync::CacheResimInteractions)
		if (SettingsCurrent.PredictiveInterpolationSettings.GetPostResimWaitForUpdate() && RewindData->IsFinalResim())
		{
			for (auto Itr = ObjectToTarget.CreateIterator(); Itr; ++Itr)
			{
				FReplicatedPhysicsTargetAsync& Target = Itr.Value();

				// If final resim frame, mark interpolated targets as waiting for up to date data from the server.
				if (Target.RepMode == EPhysicsReplicationMode::PredictiveInterpolation)
				{
					Target.SetWaiting(RigidsSolver->GetCurrentFrame() + Target.FrameOffset, Target.RepModeOverride);
				}
			}
		}
		return;
	}

	if (const FPhysicsReplicationAsyncInput* AsyncInput = GetConsumerInput_Internal())
	{
		// Update async targets with target input
		for (const FPhysicsRepAsyncInputData& Input : AsyncInput->InputData)
		{
			if (Input.TargetState.Flags == ERigidBodyFlags::None)
			{
				// Remove replication target
				RemoveObjectFromReplication(Input.PhysicsObject);
				continue;
			}

			if (!bRewindDataExist && Input.RepMode == EPhysicsReplicationMode::Resimulation)
			{
				// We don't have rewind data but an actor is set to replicate using resimulation; we need to enable rewind capture.



// 在这里判断是否固定步长
				if (ensure(Chaos::FPBDRigidsSolver::IsNetworkPhysicsPredictionEnabled() && RigidsSolver->IsUsingFixedDt()))
				{
					RigidsSolver->EnableRewindCapture();
				}
			}

			UpdateRewindDataTarget(Input);
			UpdateAsyncTarget(Input, RigidsSolver);
			
			DebugDrawReplicationMode(Input);

			// Deprecated, legacy BodyInstance flow for Default Replication
			if (Input.Proxy != nullptr)
			{
				Chaos::FSingleParticlePhysicsProxy* Proxy = Input.Proxy;
				Chaos::FRigidBodyHandle_Internal* Handle = Proxy->GetPhysicsThreadAPI();

				const FPhysicsRepErrorCorrectionData& UsedErrorCorrection = Input.ErrorCorrection.IsSet() ? Input.ErrorCorrection.GetValue() : AsyncInput->ErrorCorrection;
				DefaultReplication_DEPRECATED(Handle, Input, GetDeltaTime_Internal(), UsedErrorCorrection);
			}
		}
	}

	if (Chaos::FPBDRigidsSolver::IsNetworkPhysicsPredictionEnabled())
	{
		CacheResimInteractions();
	}

	ApplyTargetStatesAsync(GetDeltaTime_Internal());
}
```



```cpp

void FPhysicsReplicationAsync::UpdateAsyncTarget(const FPhysicsRepAsyncInputData& Input, Chaos::FPBDRigidsSolver* RigidsSolver)
{
	if (Input.PhysicsObject == nullptr)
	{
		return;
	}

	FReplicatedPhysicsTargetAsync* Target = ObjectToTarget.Find(Input.PhysicsObject);
	bool bFirstTarget = Target == nullptr;
	if (bFirstTarget)
	{
		// First time we add a target, set previous state to current input
		Target = AddObjectToReplication(Input.PhysicsObject);
		Target->PrevPos = Input.TargetState.Position;
		Target->PrevPosTarget = Input.TargetState.Position;
		Target->PrevRotTarget = Input.TargetState.Quaternion;
		Target->PrevLinVel = Input.TargetState.LinVel;
		Target->RepModeOverride = Input.RepMode;
	}
	check(Target);

	/** Target Update Description
	* @param Input = incoming state target for replication.
	* 
	* Input comes mainly from the server but can be a faked state produced by the client for example if the client object wakes up from sleeping.
	* Fake inputs should have a ServerFrame of -1 (bool bIsFake = Input.ServerFrame == -1)
	* Server inputs can have ServerFrame values of either 0 or an incrementing integer value.
	*	If the ServerFrame is 0 it should always be 0. If it's incrementing it will always increment.
	*
	* @local Target = The current state target used for replication, to be updated with data from Input.
	* Read about the different target properties in FReplicatedPhysicsTargetAsync
	* 
	* IMPORTANT:
	* Target.ServerFrame can be -1 if the target is newly created or if it has data from a fake input.
	* 
	* SendInterval is calculated by taking Input.ServerFrame - Target.ServerFrame
	*	Note, can only be calculated if the server is sending incrementing SendIntervals and if we have received a valid input previously so we have the previous ServerFrame cached in Target.
	* 
	* ReceiveInterval is calculated by taking RigidsSolver->GetCurrentFrame() - Target.ReceiveFrame
	*	Note that ReceiveInterval is only used if SendInterval is 0
	* 
	* Target.TickCount starts at 0 and is incremented each tick that the target is used for, TickCount is reset back to 0 each time Target is updated with new Input.
	* 
	* NOTE: With perfect network conditions SendInterval, ReceiveInterval and Target.TickCount will be the same value.
	*/

	// Update target from input if input is newer than target or this is the first input received (target is empty)
	if ((bFirstTarget || Input.ServerFrame == 0 || Input.ServerFrame > Target->ServerFrame))
	{
		// Get the current physics frame
		const int32 CurrentFrame = RigidsSolver->GetCurrentFrame();

		// Cache TickCount before updating it, force to 0 if ServerFrame is -1
		const int32 PrevTickCount = (Target->ServerFrame < 0) ? 0 : Target->TickCount;

		// Cache SendInterval, only calculate if we have a valid Target->ServerFrame, else leave at 0.
		const int32 SendInterval = (Target->ServerFrame <= 0) ? 0 : Input.ServerFrame - Target->ServerFrame;

		// Cache if this target was previously allowed to be altered, before this update
		const bool bPrevAllowTargetAltering = Target->bAllowTargetAltering;

		// Cache if the physics frame offset has changed since last target
		const bool bFrameOffsetCorrected = Target->FrameOffset != Input.FrameOffset;

		// Set if the target is allowed to be altered after this update
		Target->bAllowTargetAltering = !(Target->TargetState.Flags & ERigidBodyFlags::Sleeping) && !(Input.TargetState.Flags & ERigidBodyFlags::Sleeping);

		// Cache previous linear velocity
		const FVector PrevLinVel = Target->TargetState.LinVel;

		// Set Target->ReceiveInterval from either SendInterval or the number of physics ticks between receiving input states
		if (SendInterval > 0)
		{
			Target->ReceiveInterval = SendInterval;
		}
		else
		{
			const int32 PrevReceiveFrame = Target->ReceiveFrame < 0 ? (CurrentFrame - 1) : Target->ReceiveFrame;
			Target->ReceiveInterval = (CurrentFrame - PrevReceiveFrame);
		}

		// Update target from input and reset properties
		Target->ServerFrame = Input.ServerFrame;
		Target->ReceiveFrame = CurrentFrame;
		Target->TargetState = Input.TargetState;
		Target->RepMode = Input.RepMode;
		Target->FrameOffset = Input.FrameOffset.IsSet() ? *Input.FrameOffset : 0;
		Target->TickCount = 0;
		Target->AccumulatedSleepSeconds = 0.0f;

		// Update waiting state
		Target->UpdateWaiting(Input.ServerFrame);

		// Apply full Replication LOD on received target
		ApplyPhysicsReplicationLOD(Input.PhysicsObject, *Target, EPhysicsReplicationLODFlags::LODFlag_All);

		// Check if target is valid to use for resimulation and perform actions if not
		CheckTargetResimValidity(*Target);

		if (Target->RepMode == EPhysicsReplicationMode::PredictiveInterpolation)
		{
#if !(UE_BUILD_SHIPPING || UE_BUILD_TEST)
			if (PhysicsReplicationCVars::PredictiveInterpolationCVars::bDrawDebugTargets)
			{
				const FVector Offset = FVector(0.0f, 0.0f, PhysicsReplicationCVars::PredictiveInterpolationCVars::DrawDebugZOffset);
				Chaos::FDebugDrawQueue::GetInstance().DrawDebugBox(Input.TargetState.Position + Offset, FVector(15.0f, 15.0f, 15.0f), Input.TargetState.Quaternion, FColor::MakeRandomSeededColor(Input.ServerFrame), false, PhysicsReplicationCVars::DebugDrawLifeTime, 0, 1.0f);
			}
#endif

			// TickCount is 0 by default at this point and when LOD is used, TickCount will be 0 if no LOD alignment was performed, in this case perform the normal target alignment
			if (Target->TickCount == 0)
			{
				/** Target Alignment Feature
				* With variable network conditions state inputs from the server can arrive both later or earlier than expected.
				* Target Alignment can adjust for this to make replication act on a target in the timeline that the client is currently replicating in.
				* 
				* If SendInterval is 4 we expect TickCount to be 4. TickCount - SendInterval = 0, meaning the client and server has ticked physics the same amount between the target states.
				* 
				* If SendInterval is 4 and TickCount is 2 we have only simulated physics for 2 ticks with the previous target while the server had simulated physics 4 ticks between previous target and new target
				*	TickCount - SendInterval = -2
				*	To align this we need to adjust the new target by predicting backwards by 2 ticks, else the replication will start replicating towards a state that is 2 ticks further ahead than expected, making replication speed up.
				* 
				* Same goes for vice-versa:
				* If SendInterval is 4 and TickCount is 6 we have simulated physics for 6 ticks with the previous target while the server had simulated physics 4 ticks between previous target and new target
				*	TickCount - SendInterval = 2
				*	To align this we need to adjust the new target by predicting forwards by 2 ticks, else the replication will start replicating towards a state that is 2 ticks behind than expected, making replication slow down.
				* 
				* Note that state inputs from the server can arrive fluctuating between above examples, but over time the alignment is evened out to 0.
				* If the clients latency is raised or lowered since replication started there might be a consistent offset in the TickCount which is handled by TimeDilation of client physics through APlayerController::UpdateServerAsyncPhysicsTickOffset()
				*/

				// Run target alignment if we have been allowed to alter the target during the last two target updates
				if (!bFirstTarget && bPrevAllowTargetAltering && Target->bAllowTargetAltering && !bFrameOffsetCorrected)
				{
					const int32 AdjustedAverageReceiveInterval = FMath::CeilToInt(Target->AverageReceiveInterval) * PhysicsReplicationCVars::PredictiveInterpolationCVars::TargetTickAlignmentClampMultiplier;

					// Set the TickCount to the physics tick offset value from where we expected this target to arrive.
					// If the client has ticked 2 times ahead from the last target and this target is 3 ticks in front of the previous target then the TickOffset should be -1
					Target->TickCount = FMath::Clamp(PrevTickCount - Target->ReceiveInterval, -AdjustedAverageReceiveInterval, AdjustedAverageReceiveInterval);

					// Apply target alignment if we aren't waiting for a newer state from the server
					if (!Target->IsWaiting())
					{
						FPhysicsReplicationAsync::ExtrapolateTarget(*Target, Target->TickCount, GetDeltaTime_Internal());
					}
				}
			}

			// Teleport detection, we don't have specific data that tells us a teleport has happened on the server, so try to detect it by examining the previous and next state



// 在这里判断固定步长
			if (PhysicsReplicationCVars::PredictiveInterpolationCVars::TeleportDetectionEnabled == 1 && !bFirstTarget && SendInterval > 0 && RigidsSolver->IsUsingFixedDt())
			{
				const FVector PosOffset = (Input.TargetState.Position - Target->PrevPosTarget);
				if (PosOffset.SizeSquared() > (PhysicsReplicationCVars::PredictiveInterpolationCVars::TeleportDetectionMinDistance * PhysicsReplicationCVars::PredictiveInterpolationCVars::TeleportDetectionMinDistance))
				{
					const FVector Velocity = Input.TargetState.LinVel.SizeSquared() > PrevLinVel.SizeSquared() ? Input.TargetState.LinVel : PrevLinVel;
					const float DeltaSeconds = (SendInterval * RigidsSolver->GetAsyncDeltaTime());
					const float PossibleDistanceSquared = (Velocity * (DeltaSeconds * PhysicsReplicationCVars::PredictiveInterpolationCVars::TeleportDetectionVelocityMultiplier)).SizeSquared();

					if (PossibleDistanceSquared < PosOffset.SizeSquared())
					{
						// A teleport has most likely happened, set accumulated error seconds to above limit for hard snapping
						// TODO: Don't piggyback on AccumulatedErrorSeconds (potentially implement ERigidBodyFlags::Teleported)
						Target->AccumulatedErrorSeconds = PhysicsReplicationCVars::PredictiveInterpolationCVars::ErrorAccumulationSeconds + 1.0f;
					}
				}
			}

			// Cache the position we received this target at, Predictive Interpolation will alter the target state but use this as the source position for reconciliation.
			Target->PrevPosTarget = Input.TargetState.Position;
			Target->PrevRotTarget = Input.TargetState.Quaternion;
		}
	}

	/** Cache the latest ping time */
	LatencyOneWay = Input.LatencyOneWay;
}
```



#### PlayerController

控制器的 Tick 方法，这里对固定步长的判断似乎与网络有关，使用预计算

```cpp

void APlayerController::TickActor( float DeltaSeconds, ELevelTick TickType, FActorTickFunction& ThisTickFunction )
{
	CSV_SCOPED_TIMING_STAT_EXCLUSIVE(PlayerControllerTick);
	SCOPE_CYCLE_COUNTER(STAT_PlayerControllerTick);
	SCOPE_CYCLE_COUNTER(STAT_PC_TickActor);

	if (TickType == LEVELTICK_PauseTick && !ShouldPerformFullTickWhenPaused())
	{
		if (PlayerInput)
		{
			TickPlayerInput(DeltaSeconds, true);
		}

		// Clear axis inputs from previous frame.
		RotationInput = FRotator::ZeroRotator;

		if (IsValidChecked(this))
		{
			Tick(DeltaSeconds);	// perform any tick functions unique to an actor subclass
		}

		return; //root of tick hierarchy
	}

	//root of tick hierarchy

	const bool bIsClient = IsNetMode(NM_Client);
	const bool bIsLocallyControlled = IsLocalPlayerController();

	if ((GetRemoteRole() == ROLE_AutonomousProxy) && !bIsClient && !bIsLocallyControlled)
	{
		// force physics update for clients that aren't sending movement updates in a timely manner 
		// this prevents cheats associated with artificially induced ping spikes
		// skip updates if pawn lost autonomous proxy role (e.g. TurnOff() call)
		if (IsValid(GetPawn()) && GetPawn()->GetRemoteRole() == ROLE_AutonomousProxy && GetPawn()->IsReplicatingMovement())
		{
			UMovementComponent* PawnMovement = GetPawn()->GetMovementComponent();
			INetworkPredictionInterface* NetworkPredictionInterface = Cast<INetworkPredictionInterface>(PawnMovement);
			if (NetworkPredictionInterface && IsValid(PawnMovement->UpdatedComponent))
			{
				FNetworkPredictionData_Server* ServerData = NetworkPredictionInterface->HasPredictionData_Server() ? NetworkPredictionInterface->GetPredictionData_Server() : nullptr;
				if (ServerData)
				{
					UWorld* World = GetWorld();
					if (ServerData->ServerTimeStamp != 0.f)
					{
						const float WorldTimeStamp = World->GetTimeSeconds();
						const float TimeSinceUpdate = WorldTimeStamp - ServerData->ServerTimeStamp;
						const float PawnTimeSinceUpdate = TimeSinceUpdate * GetPawn()->CustomTimeDilation;
						// See how long we wait to force an update. Setting MAXCLIENTUPDATEINTERVAL to zero allows the server to disable this feature.
						const AGameNetworkManager* GameNetworkManager = (const AGameNetworkManager*)(AGameNetworkManager::StaticClass()->GetDefaultObject());
						const float ForcedUpdateInterval = GameNetworkManager->MAXCLIENTUPDATEINTERVAL;
						const float ForcedUpdateMaxDuration = FMath::Min(GameNetworkManager->MaxClientForcedUpdateDuration, 5.0f);

						// If currently resolving forced updates, and exceeded max duration, then wait for a valid update before enabling them again.
						ServerData->bForcedUpdateDurationExceeded = false;
						if (ServerData->bTriggeringForcedUpdates)
						{
							if (ServerData->ServerTimeStamp > ServerData->ServerTimeLastForcedUpdate)
							{
								// An update came in that was not a forced update (ie a real move), since ServerTimeStamp advanced outside this code.
								UE_LOG(LogNetPlayerMovement, Log, TEXT("Movement detected, resetting forced update state (ServerTimeStamp %.6f > ServerTimeLastForcedUpdate %.6f)"), ServerData->ServerTimeStamp, ServerData->ServerTimeLastForcedUpdate);
								ServerData->ResetForcedUpdateState();
							}
							else
							{
								const float PawnTimeSinceForcingUpdates = (WorldTimeStamp - ServerData->ServerTimeBeginningForcedUpdates) * GetPawn()->CustomTimeDilation;
								const float PawnTimeForcedUpdateMaxDuration = ForcedUpdateMaxDuration * GetPawn()->GetActorTimeDilation();

								if (PawnTimeSinceForcingUpdates > PawnTimeForcedUpdateMaxDuration)
								{
									// Waiting for ServerTimeStamp to advance from a client move.
									UE_LOG(LogNetPlayerMovement, Log, TEXT("Setting bForcedUpdateDurationExceeded=true (PawnTimeSinceForcingUpdates %.6f > PawnTimeForcedUpdateMaxDuration %.6f) (bLastRequestNeedsForcedUpdates:%d)"), PawnTimeSinceForcingUpdates, PawnTimeForcedUpdateMaxDuration, (int32)ServerData->bLastRequestNeedsForcedUpdates);
									ServerData->bForcedUpdateDurationExceeded = true;
								}
							}
						}
						
						const float CurrentRealTime = World->GetRealTimeSeconds();
						const bool bHitch = (CurrentRealTime - LastMovementUpdateTime) > GameNetworkManager->ServerForcedUpdateHitchThreshold && (LastMovementUpdateTime != 0);
						LastMovementHitch = bHitch ? CurrentRealTime : LastMovementHitch;
						const bool bRecentHitch = bHitch || (CurrentRealTime - LastMovementHitch < GameNetworkManager->ServerForcedUpdateHitchCooldown);
						LastMovementUpdateTime = CurrentRealTime;

						// Trigger forced update if allowed
						const float PawnTimeMinForcedUpdateInterval = (DeltaSeconds + 0.06f) * GetPawn()->CustomTimeDilation;
						const float PawnTimeForcedUpdateInterval = FMath::Max<float>(PawnTimeMinForcedUpdateInterval, ForcedUpdateInterval * GetPawn()->GetActorTimeDilation());

						if (!bRecentHitch && ForcedUpdateInterval > 0.f && (PawnTimeSinceUpdate > PawnTimeForcedUpdateInterval))
						{
							//UE_LOG(LogPlayerController, Warning, TEXT("ForcedMovementTick. PawnTimeSinceUpdate: %f, DeltaSeconds: %f, DeltaSeconds+: %f"), PawnTimeSinceUpdate, DeltaSeconds, DeltaSeconds+0.06f);
							const USkeletalMeshComponent* PawnMesh = GetPawn()->FindComponentByClass<USkeletalMeshComponent>();
							const bool bShouldForceUpdate = !ServerData->bForcedUpdateDurationExceeded || ServerData->bLastRequestNeedsForcedUpdates;
							if (bShouldForceUpdate && (!PawnMesh || !PawnMesh->IsSimulatingPhysics()))
							{
								const bool bDidUpdate = NetworkPredictionInterface->ForcePositionUpdate(PawnTimeSinceUpdate);

								// Refresh this pointer in case it has changed (which can happen if character is destroyed or repossessed).
								ServerData = NetworkPredictionInterface->HasPredictionData_Server() ? NetworkPredictionInterface->GetPredictionData_Server() : nullptr;

								if (bDidUpdate && ServerData)
								{
									ServerData->ServerTimeLastForcedUpdate = WorldTimeStamp;

									// Detect initial conditions triggering forced updates.
									if (!ServerData->bTriggeringForcedUpdates)
									{
										ServerData->ServerTimeBeginningForcedUpdates = ServerData->ServerTimeStamp;
										ServerData->bTriggeringForcedUpdates = true;
									}

									// Set server timestamp, if there was movement.
									ServerData->ServerTimeStamp = WorldTimeStamp;
								}
							}
						}
					}
					else
					{
						// If timestamp is zero, set to current time so we don't have a huge initial delta time for correction.
						ServerData->ServerTimeStamp = World->GetTimeSeconds();
						ServerData->ResetForcedUpdateState();
					}
				}
			}
		}

		// update viewtarget replicated info
		if (PlayerCameraManager != nullptr)
		{
			APawn* TargetPawn = PlayerCameraManager->GetViewTargetPawn();
			
			if ((TargetPawn != GetPawn()) && (TargetPawn != nullptr))
			{
				SetTargetViewRotation(TargetPawn->GetViewRotation());
			}
		}
	}
	else if (GetLocalRole() > ROLE_SimulatedProxy)
	{
		// Process PlayerTick with input.
		if (!PlayerInput && (Player == nullptr || Cast<ULocalPlayer>( Player ) != nullptr))
		{
			InitInputSystem();
		}

		if (PlayerInput)
		{
			QUICK_SCOPE_CYCLE_COUNTER(PlayerTick);
			PlayerTick(DeltaSeconds);
		}

		if (!IsValidChecked(this))
		{
			return;
		}

		// update viewtarget replicated info
		if (PlayerCameraManager != nullptr)
		{
			APawn* TargetPawn = PlayerCameraManager->GetViewTargetPawn();
			if ((TargetPawn != GetPawn()) && (TargetPawn != nullptr))
			{
				SmoothTargetViewRotation(TargetPawn, DeltaSeconds);
			}

			// Send a camera update if necessary.
			// That position will be used as the base for replication
			// (i.e., the origin that will be used when calculating NetCullDistance for other Actors / Objects).
			// We only do this when the Pawn will move, to prevent spamming RPCs.
			if (bIsClient && bIsLocallyControlled && GetPawn() && PlayerCameraManager->bUseClientSideCameraUpdates)
			{
				UPawnMovementComponent* PawnMovement = GetPawn()->GetMovementComponent();
				if (PawnMovement != nullptr &&
					!PawnMovement->IsMoveInputIgnored() &&
					(PawnMovement->GetLastInputVector() != FVector::ZeroVector || PawnMovement->Velocity != FVector::ZeroVector))
				{
					PlayerCameraManager->bShouldSendClientSideCameraUpdate = true;
				}
			}
		}
	}

	if (IsValidChecked(this))
	{
		QUICK_SCOPE_CYCLE_COUNTER(Tick);
		Tick(DeltaSeconds);	// perform any tick functions unique to an actor subclass
	}

	// Clear old axis inputs since we are done with them. 
	RotationInput = FRotator::ZeroRotator;

	if (bIsClient && UPhysicsSettings::Get()->PhysicsPrediction.bEnablePhysicsPrediction && GetLocalRole() == ROLE_AutonomousProxy)
	{
		if (UWorld* World = GetWorld())
		{
			if (FPhysScene* PhysScene = World->GetPhysicsScene())
			{
				if (Chaos::FPhysicsSolver* Solver = PhysScene->GetSolver())
				{

// 在这里判断固定步长
					if (Solver->IsUsingFixedDt())
					{
						TickOffsetSyncCountdown += DeltaSeconds;
						UpdateServerAsyncPhysicsTickOffset();
					}
				}
			}
		}
	}

#if !(UE_BUILD_SHIPPING || UE_BUILD_TEST)
	if (CheatManager != nullptr)
	{
		CheatManager->TickCollisionDebug();
	}
#endif // !(UE_BUILD_SHIPPING || UE_BUILD_TEST)
}
```



## AsyncPhysicsBlockMode

### 关键结构

这可能是关键，但是它默认值为 0，并且没有找到调用修改它的函数。因此

> >从上一帧 GT 生成的所有物理步都会阻塞，当前帧生成的任务则不阻塞。

FAutoConsoleVariableRef 类型的成员在 AsyncPhysicsBlockMode 改变时，调用 LambdaAsyncMode

```cpp
auto LambdaAsyncMode = FConsoleVariableDelegate::CreateLambda([](IConsoleVariable* InVariable)
{
	for (FPhysicsSolverBase* Solver : FChaosSolversModule::GetModule()->GetAllSolvers())
	{
		Solver->SetAsyncPhysicsBlockMode(EAsyncBlockMode(InVariable->GetInt()));
	}
});

// 0 blocks on any physics steps generated from past GT Frames, and blocks on none of the tasks from current frame.
// 1 blocks on everything except the single most recent task (including tasks from current frame)
// 1 should guarantee we will always have a future output for interpolation from 2 frames in the past
// 2 doesn't block the game thread. Physics steps could be eventually be dropped if taking too much time.
int32 AsyncPhysicsBlockMode = 0;
FAutoConsoleVariableRef CVarAsyncPhysicsBlockMode(TEXT("p.AsyncPhysicsBlockMode"), AsyncPhysicsBlockMode, TEXT("Setting to 0 blocks on any physics steps generated from past GT Frames, and blocks on none of the tasks from current frame."
	" 1 blocks on everything except the single most recent task (including tasks from current frame). 1 should gurantee we will always have a future output for interpolation from 2 frames in the past."
	" 2 doesn't block the game thread, physics steps could be eventually be dropped if taking too much time."), LambdaAsyncMode);

FPhysicsSolverBase::FPhysicsSolverBase(const EMultiBufferMode BufferingModeIn,const EThreadingModeTemp InThreadingMode,UObject* InOwner, Chaos::FReal InAsyncDt)
		: BufferMode(BufferingModeIn)
		// 注意是否并行——在 ChaosModule.h 中 GSingleThreadedPhysics == 1,
		// 在 Parallel.h 中 == 0
		// 不过 FPhysicsSolverBase.cpp 与 Parallel.cpp 在同一个 Framework 下，可能是 0
		, ThreadingMode(!!GSingleThreadedPhysics ? EThreadingModeTemp::SingleThread : InThreadingMode)
		// ...
		// 保存异步步长
		, AsyncDt(InAsyncDt)
		// ...
		// 使用成员变量
		, AsyncBlockMode(EAsyncBlockMode(AsyncPhysicsBlockMode))
	{
	// 这里可以看异步步长
		UE_LOG(LogChaos, Verbose, TEXT("FPhysicsSolverBase::AsyncDt:%f"), IsUsingAsyncResults() ? AsyncDt : -1);

		//If user is running with -PhysicsRunsOnGT override the cvar (doing it here to avoid parsing every time task is scheduled)
		if(FParse::Param(FCommandLine::Get(), TEXT("PhysicsRunsOnGT")))
		{
			PhysicsRunsOnGT = 1;
		}
	}
```

注意 AsyncBlockMode 变量类型

```cpp
enum EAsyncBlockMode
{
	BlockOnlyPastFrames = 0,
	BlockForBestInterpolation = 1,
	DoNoBlock = 2
};
```



### AdvanceAndDispatch_External

这里用到了 AsyncBlockMode 变量

```cpp
FGraphEventRef FPhysicsSolverBase::AdvanceAndDispatch_External(FReal InDt)
	{
		LLM_SCOPE(ELLMTag::ChaosScene);
		const bool bSubstepping = MMaxSubSteps > 1;
		SetSolverSubstep_External(bSubstepping);
		const FReal DtWithPause = bPaused_External ? 0.0f : InDt;
		FReal InternalDt = DtWithPause;
		int32 NumSteps = 1;

		// 开启异步后进入，没开则会跳过
		if(IsUsingFixedDt())
		{
			AccumulatedTime += DtWithPause;
			if(InDt == 0)	//this is a special flush case
			{
				//just use any remaining time and sync up to latest no matter what
				InternalDt = AccumulatedTime;
				NumSteps = 1;
				AccumulatedTime = 0;
			}
			else
			{
				InternalDt = AsyncDt;
				NumSteps = FMath::FloorToInt32(AccumulatedTime / InternalDt);
				AccumulatedTime -= InternalDt * static_cast<FReal>(NumSteps);
			}
		}
		else if (bSubstepping && InDt > 0)
		{
			// ...
		}

		if(InDt > 0)
		{
			ExternalSteps++;	//we use this to average forces. It assumes external dt is about the same. 0 dt should be ignored as it typically has nothing to do with force
		}

		// 理论上这才是异步，不过 AsyncBlockMode == BlockOnlyPastFrames，所以不进入
		// Eventually drop physics steps in mode 2
		if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
		{
			// Make sure not to accumulate too many physics solver tasks.
		   constexpr int32 MaxPhysicsStepToKeep = 3;
		   const int32 MaxNumSteps = MaxPhysicsStepToKeep - NumPendingSolverAdvanceTasks;
		   if (NumSteps > MaxNumSteps)
		   {
		   	CSV_CUSTOM_STAT(ChaosPhysicsSolver, PhysicsFrameDropped, NumSteps - MaxNumSteps, ECsvCustomStatOp::Accumulate);
		   	// NumSteps + NumPendingSolverAdvanceTasks shouldn't be bigger than MaxPhysicsStepToKeep
		   	NumSteps = FMath::Min<int32>(NumSteps, MaxNumSteps);
		   }
		}
			
		if (NumSteps > 0)
		{
			//make sure any GT state is pushed into necessary buffer
			PushPhysicsState(InternalDt, NumSteps, FMath::Max(ExternalSteps, 1));
			ExternalSteps = 0;
		}

		// If standalone solver we are not responsible to spawn tasks
		if(IsStandaloneSolver())
		{
			return {};
		}

		// Ensures we block on any tasks generated from previous frames
		FGraphEventRef BlockingTasks = PendingTasks;
		// 将等待任务作为阻塞任务，确保之前的任务都阻塞

		// 从 MarshallingManager 获得物理数据（Push Data）
		while(FPushPhysicsData* PushData = MarshallingManager.StepInternalTime_External())
		{
			if(MRewindCallback && !bIsShuttingDown)
			{
				MRewindCallback->ProcessInputs_External(PushData->InternalStep, PushData->SimCallbackInputs);
			}

			// 单线程进入（单线程需要等待之前的任务完成）
			if(ThreadingMode == EThreadingModeTemp::SingleThread)
			{
				ensure(!PendingTasks || PendingTasks->IsComplete());	//if mode changed we should have already blocked
				FAllSolverTasks ImmediateTask(*this, PushData);
#if !UE_BUILD_SHIPPING
				// 这个分支单元测试用
				if(bStealAdvanceTasksForTesting)
				{
					StolenSolverAdvanceTasks.Emplace(MoveTemp(ImmediateTask));
				}
				else
				{
					ImmediateTask.AdvanceSolver();
				}
#else
				ImmediateTask.AdvanceSolver();
#endif
			}
			// 多线程进入（默认）
			else
			{
				// AsyncBlockMode == BlockOnlyPastFrames，所以不进入
				// If enabled, block on all but most recent physics task, even tasks generated this frame.
				if (AsyncBlockMode == EAsyncBlockMode::BlockForBestInterpolation)
				{
					// 除了最近任务，其余全阻塞
					BlockingTasks = PendingTasks;
				}

				// 处理依赖事件，之前的任务未完成则添加依赖
				FGraphEventArray Prereqs;
				if (PendingTasks && !PendingTasks->IsComplete())
				{
					Prereqs.Add(PendingTasks);
				}

				// 分发当前数据对应的任务，应该是预处理
				PendingTasks = TGraphTask<FPhysicsSolverProcessPushDataTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);
/**
* Task responsible for processing the command buffer of a single solver and preparing data before solver task and callbacks are run
*/

				Prereqs.Add(PendingTasks);

				// 处理冻结的游戏线程回调
				if (bSolverHasFrozenGameThreadCallbacks)
				{
					PendingTasks = TGraphTask<FPhysicsSolverFrozenGTPreSimCallbacks>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this);
					Prereqs.Add(PendingTasks);
				}

				// 求解器步进任务
				PendingTasks = TGraphTask<FPhysicsSolverAdvanceTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);
/**
* Task responsible for advancing the solver once data has been prepared and GT callbacks have fired
*/

				// 启用异步则不阻塞步进任务
				if (IsUsingAsyncResults() == false)
				{
					BlockingTasks = PendingTasks;	//block right away
				}
			}

			// This break is mainly here to satisfy unit testing. The call to StepInternalTime_External will decrement the
			// delay in the marshaling manager and throw of tests that are explicitly testing for propagation delays
			if (IsUsingAsyncResults() == false && !bSubstepping)
			{
				break;
			}
		}
		if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
		{
			return {};
		}
		// 返回阻塞任务，但是几乎所有函数调用都不利用此返回值，除了 FChaosScene::StartFrame()
		return BlockingTasks;
	}
```

![](image-20250613131700131.png)



### 用到 AdvanceAndDispatch_External 的函数

#### FChaosScene

应该是场景刷新

```cpp

void FChaosScene::StartFrame()
{
	using namespace Chaos;

	SCOPE_CYCLE_COUNTER(STAT_Scene_StartFrame);

	if(CVar_ChaosSimulationEnable.GetValueOnGameThread() == 0)
	{
		return;
	}

	const float UseDeltaTime = OnStartFrame(MDeltaTime);

	TArray<FPhysicsSolverBase*> SolverList = GetPhysicsSolvers();
	for(FPhysicsSolverBase* Solver : SolverList)
	{
		if(FGraphEventRef SolverEvent = Solver->AdvanceAndDispatch_External(UseDeltaTime))
		{
			if(SolverEvent.IsValid())
			{
				CompletionEvents.Add(SolverEvent);
			}
		}
	}
}

void FChaosScene::Flush()
{
	check(IsInGameThread());

	Chaos::FPBDRigidsSolver* Solver = GetSolver();

	if(Solver)
	{
		//Make sure any dirty proxy data is pushed
		Solver->AdvanceAndDispatch_External(0);	//force commands through
		Solver->WaitOnPendingTasks_External();

		// Populate the spacial acceleration
		Chaos::FPBDRigidsSolver::FPBDRigidsEvolution* Evolution = Solver->GetEvolution();

		if(Evolution)
		{
			Evolution->FlushSpatialAcceleration();
		}
	}

	CopySolverAccelerationStructure();
}
```



#### AChaosSolverActor

应该是一个 Actor

```cpp
void AChaosSolverActor::WriteToSimulation(const float DeltaTime, const bool bAsyncTask)
{
	if (RigidSolverProxy.IsValid())
	{
		if(!bAsyncTask)
		{
			if(!Proxy && bHasFloor)
			{
				MakeFloor();
			} 
		
			// Update gravity in case it changed
			const FVector DefaultGravity( 0.f, 0.f, GetWorld()->GetGravityZ() );
		
			PhysScene->SetUpForFrame(&DefaultGravity, DeltaTime, UPhysicsSettings::Get()->MinPhysicsDeltaTime, UPhysicsSettings::Get()->MaxPhysicsDeltaTime,
				UPhysicsSettings::Get()->MaxSubstepDeltaTime, UPhysicsSettings::Get()->MaxSubsteps, UPhysicsSettings::Get()->bSubstepping);
		
			PhysScene->StartFrame();
		}
		else
		{
			GetSolver()->AdvanceAndDispatch_External(DeltaTime);
		}

		RigidSolverProxy.PushDatas.Reset();
		while(Chaos::FPushPhysicsData* PushData = RigidSolverProxy.Solver->GetMarshallingManager().StepInternalTime_External())
		{
			RigidSolverProxy.PushDatas.Add(PushData);
		}
	}
}
```



#### FPhysScene_Chaos

空函数，未完成

```cpp

void FPhysScene_Chaos::ResimNFrames(const int32 NumFramesRequested)
{
	//needs to run on physics thread from a special location
	//todo: flag solver
#if 0
	QUICK_SCOPE_CYCLE_COUNTER(ResimNFrames);
	using namespace Chaos;
	auto Solver = GetSolver();
	if(FRewindData* RewindData = Solver->GetRewindData())
	{
		const int32 FramesSaved = RewindData->GetFramesSaved() - 2;	//give 2 frames buffer because right at edge we have a hard time
		const int32 NumFrames = FMath::Min(NumFramesRequested,FramesSaved);
		if(NumFrames > 0)
		{
			const int32 LatestFrame = RewindData->CurrentFrame();
			const int32 FirstFrame = LatestFrame - NumFrames;
			if(ensure(Solver->GetRewindData()->RewindToFrame(FirstFrame)))
			{
				//resim as single threaded
				const auto PreThreading = Solver->GetThreadingMode();
				Solver->SetThreadingMode_External(EThreadingModeTemp::SingleThread);
				for(int Frame = FirstFrame; Frame < LatestFrame; ++Frame)
				{
					Solver->AdvanceAndDispatch_External(RewindData->GetDeltaTimeForFrame(Frame));
					Solver->UpdateGameThreadStructures();
				}

				Solver->SetThreadingMode_External(PreThreading);

#if !UE_BUILD_SHIPPING
				const TArray<FDesyncedParticleInfo> DesyncedParticles = Solver->GetRewindData()->ComputeDesyncInfo();
				if(DesyncedParticles.Num())
				{
					UE_LOG(LogChaos,Log,TEXT("Resim had %d desyncs"),DesyncedParticles.Num());
					for(const FDesyncedParticleInfo& Info : DesyncedParticles)
					{
						const FBodyInstance* BI = ChaosInterface::GetUserData(Info.Particle);
						const FBox Bounds = BI->GetBodyBounds();
						FVector Center,Extents;
						Bounds.GetCenterAndExtents(Center,Extents);
						DrawDebugBox(GetOwningWorld(),Center,Extents,FQuat::Identity, Info.MostDesynced == ESyncState::HardDesync ? FColor::Red : FColor::Yellow, /*bPersistentLines=*/ false, /*LifeTime=*/ 3);
					}
				}
#endif
			}
		}
	}
#endif
}
```



#### FPhysicsSolverBase

销毁求解器，注意其中改写线程模式

```cpp
void FPhysicsSolverBase::DestroySolver(FPhysicsSolverBase& InSolver)
{
	// Please read the comments this is a minefield.
			
	const bool bIsSingleThreadEnvironment = FPlatformProcess::SupportsMultithreading() == false;
	if (bIsSingleThreadEnvironment == false)
	{
		// In Multithreaded: DestroySolver should only be called if we are not waiting on async work.
		// This should be called when World/Scene are cleaning up, World implements IsReadyForFinishDestroy() and returns false when async work is still going.
		// This means that garbage collection should not cleanup world and this solver until this async work is complete.
		// We do it this way because it is unsafe for us to block on async task in this function, as it is unsafe to block on a task during GC, as this may schedule
		// another task that may be unsafe during GC, and cause crashes.
		ensure(InSolver.IsPendingTasksComplete());
	}
	else
	{
		// In Singlethreaded: We cannot wait for any tasks in IsReadyForFinishDestroy() (on World) so it always returns true in single threaded.
		// Task will never complete during GC in single theading, as there are no threads to do it.
		// so we have this wait below to allow single threaded to complete pending tasks before solver destroy.

		InSolver.WaitOnPendingTasks_External();
	}

	// GeometryCollection particles do not always remove collision constraints on unregister,
	// explicitly clear constraints so we will not crash when filling collision events in advance.
	// @todo(chaos): fix this and remove
	{
		auto* Evolution = static_cast<FPBDRigidsSolver&>(InSolver).GetEvolution();
		if (Evolution)
		{
			Evolution->ResetConstraints();
		}
	}

	// 这里改写为单线程
	// Advance in single threaded because we cannot block on an async task here if in multi threaded mode. see above comments.
	InSolver.SetThreadingMode_External(EThreadingModeTemp::SingleThread);
	InSolver.MarkShuttingDown();
	{
		InSolver.AdvanceAndDispatch_External(0);	//flush any pending commands are executed (for example unregister object)
	}

	// verify callbacks have been processed and we're not leaking.
	// TODO: why is this still firing in 14.30? (Seems we're still leaking)
	//ensure(InSolver.SimCallbacks.Num() == 0);

	delete &InSolver;
}
```



## 其它类——组件
### UChaosMoverBackendComponent

存在 `bIsUsingAsyncPhysics` 控制判断异步

```cpp
UChaosMoverBackendComponent::UChaosMoverBackendComponent()
	: PathTargetConstraintPhysicsUserData(&PathTargetConstraintInstance)
{
	PrimaryComponentTick.bCanEverTick = false;

	bWantsInitializeComponent = true;
	bAutoActivate = true;

	// 这里从求解器获得是否使用异步
	if (const Chaos::FPhysicsSolver* Solver = GetPhysicsSolver())
	{
		bIsUsingAsyncPhysics = Solver->IsUsingAsyncResults();
	}
	// ...
}

void UChaosMoverBackendComponent::InitializeComponent()
{
	Super::InitializeComponent();

	if (UWorld* World = GetWorld(); World && World->IsGameWorld())
	{
		NullMovementMode = NewObject<UNullMovementMode>(&GetMoverComponent(), TEXT("NullMovementMode"));
		ImmediateModeTransition = NewObject<UImmediateMovementModeTransition>(&GetMoverComponent(), TEXT("ImmediateModeTransition"));

		// Create NetworkPhysicsComponent
		if ((World->GetNetMode() != ENetMode::NM_Standalone) && Chaos::FPhysicsSolverBase::IsNetworkPhysicsPredictionEnabled())
		{
			if (!bIsUsingAsyncPhysics)
			{
				// Verify that the Project Settings have bTickPhysicsAsync turned on.
				// It's easy to waste time forgetting that, since they are off by default.
				UE_LOG(LogChaosMover, Warning, TEXT("Chaos Mover Backend only supports networking with Physics Async. Networked Physics will not work well. Turn on 'Project Settings > Engine - Physics > Tick Physics Async', or play in Standalone Mode"));
				// This is important enough that we break for developers debugging in editor
				UE_DEBUG_BREAK();
			}
			else
			{

// 开启异步的情况，似乎用于网络


				NetworkPhysicsComponent = NewObject<UNetworkPhysicsComponent>(GetOwner(), TEXT("PhysMover_NetworkPhysicsComponent"));

				// This isn't technically a DSO component, but set it net addressable as though it is
				NetworkPhysicsComponent->SetNetAddressable();
				NetworkPhysicsComponent->SetIsReplicated(true);
				NetworkPhysicsComponent->RegisterComponent();
				if (!NetworkPhysicsComponent->HasBeenInitialized())
				{
					NetworkPhysicsComponent->InitializeComponent();
				}
				NetworkPhysicsComponent->Activate(true);

				// Register network data for recording and rewind/resim
				NetworkPhysicsComponent->CreateDataHistory<UE::ChaosMover::FNetworkDataTraits>(this);

				if (NetworkPhysicsComponent->HasServerWorld())
				{
					if (APawn* PawnOwner = Cast<APawn>(GetOwner()))
					{
						// When we're owned by a pawn, keep an eye on whether it's currently player-controlled or not
						PawnOwner->ReceiveControllerChangedDelegate.AddUniqueDynamic(this, &ThisClass::HandleOwningPawnControllerChanged_Server);
						HandleOwningPawnControllerChanged_Server(PawnOwner, nullptr, PawnOwner->Controller);
					}
					else
					{
						// If the owner isn't a pawn, there's no chance of player input happening, so inputs to the PT are always produced on the server
						NetworkPhysicsComponent->SetIsRelayingLocalInputs(true);
					}
				}
			}
		}
	}
}
```



### UAsyncPhysicsInputComponent

在这里 SetDataClass 调用会启用 AsyncPhysicsTick

```cpp
void UAsyncPhysicsInputComponent::SetDataClass(TSubclassOf<UAsyncPhysicsData> InDataClass)
{
	ensureMsgf(DataClass == nullptr, TEXT("You can only set the data class once"));
	DataClass = InDataClass;

	DataToWrite = DuplicateObject((UAsyncPhysicsData*)DataClass->GetDefaultObject(), nullptr);
	
	//now that we have a class we're ready to async tick
	SetAsyncPhysicsTickEnabled(true);
	//...
}
```

然后可以回调函数 AsyncPhysicsTickComponent 。



### UAsyncPhysicsTickComponent

虚函数 AsyncPhysicsTickComponent



### AsyncPhysicsTickActor

虚函数 AsyncPhysicsTickComponent



### FAsyncPhysicsTickCallback

```cpp
class FAsyncPhysicsTickCallback : public Chaos::TSimCallbackObject<
	Chaos::FSimCallbackNoInput,
	Chaos::FSimCallbackNoOutput,
	Chaos::ESimCallbackOptions::Presimulate | Chaos::ESimCallbackOptions::RunOnFrozenGameThread>
{
public:

	TSet<UActorComponent*> AsyncPhysicsTickComponents;
	TSet<AActor*> AsyncPhysicsTickActors;
	TArray<FPendingAsyncPhysicsCommand> PendingCommands;

	virtual void OnPreSimulate_Internal() override
	 {
	 	using namespace Chaos;
	 
	 	/* #TODO implement and re-enable resim commands. This callback must run on the main thread and resim currently does
	 	 * not defer its callbacks to the main thread making its execution unsafe.
	 	const UPhysicsSettings* PhysicsSettings = UPhysicsSettings::Get();
	 	const bool bAllowResim = PhysicsSettings->PhysicsPrediction.bEnablePhysicsPrediction;
	 	const int32 NumFrames = PhysicsSettings->GetPhysicsHistoryCount();
	 	*/
	 
	 	TArray<int32> CommandIndicesToRemove;
	 	CommandIndicesToRemove.Reserve(PendingCommands.Num());
	 
	 	for (int32 Idx = 0; Idx < PendingCommands.Num(); ++Idx)
	 	{
	 		const int32 CurrentFrame = static_cast<FPBDRigidsSolver*>(GetSolver())->GetCurrentFrame();
	 
	 		const FPendingAsyncPhysicsCommand& PendingCommand = PendingCommands[Idx];
	 		bool bRemove = PendingCommand.OwningObject.IsStale() || !PendingCommand.Command;
	 
	 		/* #TODO implement and re-enable resim commands. This callback must run on the main thread and resim currently does
	 		 * not defer its callbacks to the main thread making its execution unsafe. 
	 
	 		if (!bRemove && bAllowResim && PendingCommand.bEnableResim && PendingCommand.PhysicsStep > (CurrentFrame - NumFrames))
	 		{
	 			if (PendingCommand.PhysicsStep < CurrentFrame)
	 			{
	 				if (Chaos::FRewindData* RewindData = GetSolver()->GetRewindData())
	 				{
	 					int32 ResimFrame = RewindData->GetResimFrame();
	 					ResimFrame = (ResimFrame == INDEX_NONE) ? PendingCommand.PhysicsStep :
	 						FMath::Min(ResimFrame, PendingCommand.PhysicsStep);
	 
	 					RewindData->RequestResimulation(ResimFrame);
	 				}
	 			}
	 			else if (PendingCommand.PhysicsStep == CurrentFrame)
	 			{
	 				PendingCommand.Command();
	 				bRemove = true;
	 			}
	 		}
	 		else
	 		*/
	 		{
	 			if (!bRemove && PendingCommand.PhysicsStep <= CurrentFrame)
	 			{
	 				PendingCommand.Command();
	 				bRemove = true;
	 			}
	 		}
	 
	 		if (bRemove)
	 		{
	 			CommandIndicesToRemove.Add(Idx);
	 		}
	 	}
	 
	 	RemoveArrayItemsAtSortedIndices(PendingCommands, CommandIndicesToRemove);
	 
	 	const FReal DeltaTime = GetDeltaTime_Internal();
	 	const FReal SimTime = GetSimTime_Internal();
	 	//TODO: handle case where callbacks modify AsyncPhysicsTickComponents or AsyncPhysicsTickActors
	 	{
	 		QUICK_SCOPE_CYCLE_COUNTER(STAT_AsyncPhys_TickComponents);
	 		for(UActorComponent* Component : AsyncPhysicsTickComponents)
	 		{
	 			check(Component && Component->IsActive());

				// 在这里调用虚函数 AsyncPhysicsTickComponent
	 			FScopeCycleCounterUObject ComponentScope(Component);
	 			Component->AsyncPhysicsTickComponent(DeltaTime, SimTime);
	 		}
	 
	 	{
	 		QUICK_SCOPE_CYCLE_COUNTER(STAT_AsyncPhys_TickActors);
	 		for(AActor* Actor : AsyncPhysicsTickActors)
	 		{
	 			check(Actor)
	 
	 			FScopeCycleCounterUObject ActorScope(Actor);
	 			Actor->AsyncPhysicsTickActor(DeltaTime, SimTime);
	 		}
	 	}
	 }
 };
```



```cpp
class ISimCallbackObject
{
void PreSimulate_Internal()
{
	OnPreSimulate_Internal();
}
};
```



### FPhysicsSolverFrozenGTPreSimCallbacks

在 FPhysicsSolverBase 中调用了 FPhysicsSolverFrozenGTPreSimCallbacks

```cpp

FGraphEventRef FPhysicsSolverBase::AdvanceAndDispatch_External(FReal InDt)
{
	LLM_SCOPE(ELLMTag::ChaosScene);
	const bool bSubstepping = MMaxSubSteps > 1;
	SetSolverSubstep_External(bSubstepping);
	const FReal DtWithPause = bPaused_External ? 0.0f : InDt;
	FReal InternalDt = DtWithPause;
	int32 NumSteps = 1;

	if(IsUsingFixedDt())
	{
		AccumulatedTime += DtWithPause;
		if(InDt == 0)	//this is a special flush case
		{
			//just use any remaining time and sync up to latest no matter what
			InternalDt = AccumulatedTime;
			NumSteps = 1;
			AccumulatedTime = 0;
		}
		else
		{

			InternalDt = AsyncDt;
			NumSteps = FMath::FloorToInt32(AccumulatedTime / InternalDt);
			AccumulatedTime -= InternalDt * static_cast<FReal>(NumSteps);
		}
	}
	else if (bSubstepping && InDt > 0)
	{
		NumSteps = FMath::CeilToInt32(DtWithPause / MMaxDeltaTime);
		if (NumSteps > MMaxSubSteps)
		{
			// Hitting this case means we're losing time, given the constraints of MaxSteps and MaxDt we can't
			// fully handle the Dt requested, the simulation will appear to the viewer to run slower than realtime
			NumSteps = MMaxSubSteps;
			InternalDt = MMaxDeltaTime;
		}
		else
		{
			InternalDt = DtWithPause / static_cast<FReal>(NumSteps);
		}
	}

	if(InDt > 0)
	{
		ExternalSteps++;	//we use this to average forces. It assumes external dt is about the same. 0 dt should be ignored as it typically has nothing to do with force
	}

	// Eventually drop physics steps in mode 2
	if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
	{
		// Make sure not to accumulate too many physics solver tasks.
		constexpr int32 MaxPhysicsStepToKeep = 3;
		const int32 MaxNumSteps = MaxPhysicsStepToKeep - NumPendingSolverAdvanceTasks;
		if (NumSteps > MaxNumSteps)
		{
			CSV_CUSTOM_STAT(ChaosPhysicsSolver, PhysicsFrameDropped, NumSteps - MaxNumSteps, ECsvCustomStatOp::Accumulate);
			// NumSteps + NumPendingSolverAdvanceTasks shouldn't be bigger than MaxPhysicsStepToKeep
			NumSteps = FMath::Min<int32>(NumSteps, MaxNumSteps);
		}
	}
		
	if (NumSteps > 0)
	{
		//make sure any GT state is pushed into necessary buffer
		PushPhysicsState(InternalDt, NumSteps, FMath::Max(ExternalSteps, 1));
		ExternalSteps = 0;
	}

	// If standalone solver we are not responsible to spawn tasks
	if(IsStandaloneSolver())
	{
		return {};
	}

	// Ensures we block on any tasks generated from previous frames
	FGraphEventRef BlockingTasks = PendingTasks;

	while(FPushPhysicsData* PushData = MarshallingManager.StepInternalTime_External())
	{
		if(MRewindCallback && !bIsShuttingDown)
		{
			MRewindCallback->ProcessInputs_External(PushData->InternalStep, PushData->SimCallbackInputs);
		}

		if(ThreadingMode == EThreadingModeTemp::SingleThread)
		{
			ensure(!PendingTasks || PendingTasks->IsComplete());	//if mode changed we should have already blocked
			FAllSolverTasks ImmediateTask(*this, PushData);
#if !UE_BUILD_SHIPPING
			if(bStealAdvanceTasksForTesting)
			{
				StolenSolverAdvanceTasks.Emplace(MoveTemp(ImmediateTask));
			}
			else
			{
				ImmediateTask.AdvanceSolver();
			}
#else
			ImmediateTask.AdvanceSolver();
#endif
		}
		else
		{
			// If enabled, block on all but most recent physics task, even tasks generated this frame.
			if (AsyncBlockMode == EAsyncBlockMode::BlockForBestInterpolation)
			{
				BlockingTasks = PendingTasks;
			}

			FGraphEventArray Prereqs;
			if (PendingTasks && !PendingTasks->IsComplete())
			{
				Prereqs.Add(PendingTasks);
			}

			PendingTasks = TGraphTask<FPhysicsSolverProcessPushDataTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);
			Prereqs.Add(PendingTasks);

			if (bSolverHasFrozenGameThreadCallbacks)
			{
				PendingTasks = TGraphTask<FPhysicsSolverFrozenGTPreSimCallbacks>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this);
				Prereqs.Add(PendingTasks);
			}

			PendingTasks = TGraphTask<FPhysicsSolverAdvanceTask>::CreateTask(&Prereqs).ConstructAndDispatchWhenReady(*this, PushData);

			if (IsUsingAsyncResults() == false)
			{
				BlockingTasks = PendingTasks;	//block right away
			}
		}

		// This break is mainly here to satisfy unit testing. The call to StepInternalTime_External will decrement the
		// delay in the marshaling manager and throw of tests that are explicitly testing for propagation delays
		if (IsUsingAsyncResults() == false && !bSubstepping)
		{
			break;
		}
	}
	if (AsyncBlockMode == EAsyncBlockMode::DoNoBlock)
	{
		return {};
	}
	return BlockingTasks;
}
```


## UE Chaos

在同步物理中，每一帧开始时，GameThread (GT) 首先将物理数据交给 Marshalling Manager，分发求解任务给 PhysicsThread，然后 GameThread 将会等待求解任务完成。PhysicsThread 完成计算任务后，将结果返回给 Marshalling Manager，后者将物理数据返回给 GameThread 

![](image-20250613131653232.png)

在异步物理中，GameThread 不再等待 PhysicsThread 完成。如果物理增量为 33，游戏增量为 16，则正常情况下 2 个游戏帧对应 1 个物理帧。但如果游戏线程卡顿 100，则下一帧会发出 3 个任务而非 1 个来赶上时间，不过游戏线程不会等待这 3 个任务完成后再移动到下一帧。所有内容最终通过时间戳同步，并在需要时插值。

![](image-20250613131700131.png)



