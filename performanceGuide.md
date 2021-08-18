# Performance Metrics

## Monitored Values
Performance Metrics test the runtime of all components and the communication overhead between elements. In addition, counters are kept for all components run. There are several places in which components are run in SST, but the three main areas are in the clock handler, event handler, and synchronization when using MPI. The clock handler deals with processing activity on the clock edges, while the event handler deals with communication between components.
In terms of overhead, the event handler is typically extremely short, on the order of 10s of nanoseconds while the clock handler is on the order of 500ns to several microseconds. Finally, the synchronization overhead can dominate the runtime if the ranks are not partitioned well. An example of this I've found though small scale experimentation is that partitioning the cache on a different rank than a core can result in serious preformance penalities. Granted part of this can be from the rank synchronization which has 5 barriers versus the thread which only has 1 barrier. 

In short the following values are tracked:
1. Per Component Clock Handler runtimes
2. Per Component Event Handler runtimes
3. Per Component Clock Handler counter
4. Per Component Event Handler runtimes
5. Rank synchronization runtime
6. Rank synchronization counter
7. Thread synchronization runtime
8. Thread synchronization counter
9. Average Handler times
10. Per Component Event Sending frequency
11. Per Component Event Receiving frequency
12. Per Rank Communication latency?

### Configuration Options
There exist several configuration options for the performance metrics. However there is also a required configuration option that must be enabled. The event handler tracing requires that SST_DEBUG_EVENT_TRACKING has to be enabled. This is because the only way to determine which components are communicating through the event is using this optional debug option which marks each transaction and the data being communicated across it. 

To enable debug event tracking, you will have use the configure script in the SST_CORE_ROOT directory. The flag --enable-event-tracking will enable the features. Once debug event tracking is enabled, to enable the performance tracking use the --enable-perf-tracking flag in the configuration script. This will enable all of the performance options by default and use the nanosecond resolution clock. All configuration options are listed below and if you wish to disable them you can find the define statements in the sst config file.

#### PERFORMANCE INSTRUMENTING
This option enables a performance printout and is a requirement for any other performance option. The performance instrumenting will create a file for each thread within each rank and populate it with any profiling data selected below. 

#### RUNTIME PROFILING
Runtime profiling will measure the total time that the simulation has been running. The main execution loop within the simulation.cc file is profiled. This information will be printed out the file enabled by performance instrumenting. This form of profiling has very small overhead as it is outside of the main simulation loop and can add information to the file output.

#### CLOCK PROFILING
Clock profiling measures all clock handler performance. The clock handlers are registered to the cpu using a registerHandler call to the baseComponent which is then passed to the simulation implementation. Originally, there is no association between the component id and the clock handler except within the setup of the component itself. Therefore, a new function was added to the baseComponent and simulation impl classes called registerClockHandler. This new function takes as inputs the componentId and the address of the clock handler and creates a mapping between the two. This mapping is then used during the printing process to associate the name of the component with the handler. Without this handler mapping, the raw address of the handler is printed out and it is the job of the user to determine which handler is matched to each component. An example of the function call for the REV CPU, miranda CPU, and cache element are shown below.
Besides the handler mapping, two maps are added to each instance of the simulation to track the execution time and counter for each clock handler. The code to populate the handlers can be found in the src/sst/core/clock.cc file. As clock handlers are run within a loop, activating clock handler profiling will have a performance penalty akin to ~3% additional overhead.

#### EVENT PROFILING
Event profiling deals with communication betwene various components and subcomponents. Event handlers are run within the src/sst/core/event.cc file. Normally these handlers do not contain information about their sending and receiving components, but this can be corrected by the --enable-event-tracking on the configuration script. Once this is enabled, the sending and receiving components are attached to each transaction and can be recorded by event tracking. All the component specific information is recorded in src/sst/core/event.cc, but other profiling information exists on a rank basis. 
To determine the amount of data sent and received per rank is recorded in the syncQueue. Since this syncQueue is used to communicate between ranks, all data sent and recieved per rank passes through this structure. Thus the size of each communication is recorded on a per rank basis. On a getData call, all information is received from a sending rank to a receiving, so we can measure received data here. Similarly, the insert call puts data in and the amount of data transmitted can be recorded here.
To determine latency for each rank, we need to measure the amount of time the getData call is run. This call transfers the data to a local buffer and thus needs to be profiled. This information is measured in the src/sst/core/sync/rankSyncSerialSkip.cc and src/sst/core/sync/rankSyncParallelSkip.cc.

#### SYNC PROFILING
One of the largest areas of time spent is each rank waiting within the synchronization calls for openMPI. This synchronization call is within src/sst/core/sync/syncManager.cc. Here profiling is done to measure the amount of time waiting on the barrier calls and then associating that information with a rank. Two variables are instantiated within each simulation implementation and are updated by the execute call within the syncManager file. 

#### HIGH RESOLUTION CLOCK
To allow flexibility in profiling data, the resolution of all time measurements can be in the nanosecond or microsecond resolution. The nanosecond resolution is implemented using the C++ std::chrono library while the microsecond resolution is implemented using gettimeofday. All profiling options listed above have an increased overhead using the nanosecond resolution, but this overhead is still below 10% when all profiling information is enabled. To enable the nanosecond resolution, ensure the define statement within sst config is enabled. If HIGH_RESOLUTION_CLOCK is not enabled, all timing collection will use gettimeofday and be of microsecond resolution. As some of the clock handlers and especially the event handlers are extremely short, the nanosecond resolution clock is reccomended.

#### PERIODIC PRINT
In order to periodically view performance of the simulator, the printPerformanceInfo function has been added to the src/sst/core/simulation.cc file and is called within the main loop in a periodic manner. After each execute call, a counter is updated and periodically the counter is reset and information is written to the file. However, the parameter is tunable. The parameter given in the sst config.h file prints about every 12 seconds, but this counter should be massively increased for longer simulations.

### Modifications to SST Elements
In order for clock handlers to work properly, you will need to slightly modify the way that clock handlers are set up. Shown below are three examples for the REV cpu, Miranda CPU, and caches.

#### REV
In the REV cpu, rather than declaring the clockHandler within the registerClock call, it is done so before hand and then since the handler is a pointer, the address is passed to the new registerClockHandler call.

-     timeConverter  = registerClock(cpuClock, new SST::Clock::Handler<RevCPU>(this,&RevCPU::clockTickPANTest));
+    auto clockHandler = new SST::Clock::Handler<RevCPU>(this,&RevCPU::clockTickPANTest);
+    timeConverter  = registerClock(cpuClock, clockHandler);
      testIters = params.find<unsigned>("testIters", 255);
+    registerClockHandler(id, (unsigned long long) clockHandler);

#### Miranda
In the miranda CPU, the clock handler is already declared outside of the registerClock call, so all that is necessary is an additional call after the registerClock to registerClockHandler to setup the mapping.

     clockHandler = new Clock::Handler<RequestGenCPU>(this, &RequestGenCPU::clockTick);
     timeConverter = registerClock(cpuClock, clockHandler );
     registerClockHandler(id, (unsigned long long) clockHandler);

#### Caches
Finally caches and other non-cpu clocked elements can be added to the mapping by adding a registerClockHandler exactly like the CPU versions.

    clockHandler       = new Clock::Handler<Cache>(this, &Cache::clockTick);
    defaultTimeBase    = registerClock(frequency, clockHandler);
    registerClockHandler(id, (unsigned long long) clockHandler);
