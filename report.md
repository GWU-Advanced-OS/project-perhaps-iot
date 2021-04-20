## Intro  
ThreadX is a deterministic, embedded RTOS created by Express Logic (acquired by Microsoft) that provides "preemption-threshold, priority inheritance, efficient timer management, fast software timers, picokernel design, event-chaining, and small size: minimal size on an ARM architecture processor is about 2 KB." [referenced from Wikipedia]( https://en.wikipedia.org/wiki/ThreadX). ThreadX as described by [Microsoft](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#azure-rtos-threadx-services): Azure RTOS ThreadX provides advanced scheduling, communication, synchronization, timer, memory management, and interrupt management facilities. It is important to note that ThreadX is a component of the larger Azure RTOS. 

## Summary, Goals, and Purpose of ThreadX (Q1) 
Azure RTOS ThreadX is an RTOS designed specifically for multithreaded and safety-critical embedded applications.<sup>[[1a]](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter1#multitasking)</sup> It acheives this goal by providing conventional RTOS features such as priority inheritance and deterministic processing, as well as features unique to ThreadX such as premption-thresholds (the ability to determine the need for preemptive versus non-premptive scheduling)<sup>[2a](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#preemption-threshold)</sup> and memory protection provided by Modules (bundled application threads that can be dynamically loaded and run on the target),<sup>[[3a]](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1)</sup> which is discussed in more detail later on in this document.

## Target Domain, Value, and Trade-Offs (Q2)
ThreadX is most concerned with concurrency issues on real-time embedded systems. Support for this large amount of concurrency and priority-based solutions are evident when looking through the code base. Therefore, it is not optimal for non-embedded applications, nor for applications that don't require several tasks being done concurrently. Systems that could benefit from using ThreadX share a common need for real-time systems with a small memory footprint and multitasking abilities.<sup>[[4a]](https://militaryembedded.com/comms/rf-and-microwave/case-bullseye-million-miles-away)</sup>  

Priority of threads and also the threshold which is required for preemption is stored for each process currently running, which allows for the concurrency of the system.<sup>[[5a]](https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common/inc/tx_trace.h#L223-L227)</sup> 
```
[Thread Priority or         4           This 4-byte field contains the current thread pointer for interrupt
             Current Thread                         events or the thread preemption-threshold/priority for thread events.
             Preemption-Threshold/
             Priority]
            [Event ID]         
```
However, because the support for this is so vast, it is unlikely that this OS would be particularly useful for anyone who does not want to take advantage of this aspect of the OS.

## Modules and the Module Manager (Q3)
Isolation is provided through ThreadX Modules. The layout out how these modules interact with one another can be in this graphic provided by the ThreadX Documentation.<sup>[[6a]](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1))</sup>
![module layout](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/media/image2.png "threadx modules")  
These modules communicate via a ThreadX Module API.<sup>[[7a]](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter4)</sup>
ThreadX modules provide dynamic memory management so that several large modules can be running on little memory by only allocating memory to the specific thread running, not the whole module. As illustrated in the image above, ThreadX also includes a module manager within the kernel to keep track of  indivudal modules using a list.<sup>[[8a]](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter3)</sup>
```
/* Load the module that is already there,
        in this example it is placed at 0x00800000. */
    txm_module_manager_in_place_load(&my_module1, "my module1", (VOID *) 0x00800000);

    /* Load a second instance of the module. */
    txm_module_manager_in_place_load(&my_module2, "my module2", (VOID *) 0x00800000);

    /* Enable shared memory region for module2. */
    txm_module_manager_external_memory_enable(&my_module2, (void*)0x20600000, 0x010000, 0x3F);

    /* Start the modules. */
    txm_module_manager_start(&my_module1);
    txm_module_manager_start(&my_module2);
```
Above is a sample of how the module manager loading two mondules and altering information about these modules, such as allowing the second module to access shared memory.<sup>[[9a]](https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter3#module-manager-example)</sup> You can also see in the code below an example of how the data structure within the module manager may add modules to the list that tracks exisiting modules.<sup>[[10a]](https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common_modules/module_manager/src/txm_module_manager_absolute_load.c#L403-L414)</sup> 
```
/* Load the module control block with port-specific information. */
    TXM_MODULE_MANAGER_MODULE_SETUP(module_instance);

    /* Now add the module to the linked list of created modules.  */
    if (_txm_module_manger_loaded_count++ == 0)
    {

        /* The loaded module list is empty.  Add module to empty list.  */
        _txm_module_manager_loaded_list_ptr =                     module_instance;
        module_instance -> txm_module_instance_loaded_next =      module_instance;
        module_instance -> txm_module_instance_loaded_previous =  module_instance;
    }
```

## Q4  

## Q5  

## Q6  

## Q7 
When searching Azure RTOS ThreadX documentation for security features, I found that it has several security certifications such as EAL4+ Common Criteria. The Target of Evaluation (TOE) covers Azure RTOS ThreadX, Azure RTOS NetX-Duo, Azure RTOS NetX Secure TLS, and Azure RTOS NetX MQTT. This represents the most typical IoT protocols required by deeply embedded sensors, devices, edge routers, and gateways.The IT Security Evaluation Facility used for the Azure RTOS security certification is Brightsight BV and the Certification Authority is SERTIT. <sup>[[1c]](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#pre-certified-by-tuv-and-ul-to-many-safety-standards)</sup>. The documentation also mentions that ThreadX supports IP layer security (IPsec) and socket layer security (TLS and DTLS) protocols, is tested and certified to meet international security assurance requirements, and integrated with Azure Defender to detect threats and remediate issues. The ThreadX repository on Github states that Azure RTOS provides OEMs with components to secure communication and to create code and data isolation using underlying MCU/MPU hardware protection mechanisms <sup>[[2c]](https://github.com/azure-rtos/threadx#security)</sup>. In addition to the security provided by Azure RTOS ThreadX, there are additional compatible services available, termed Azure Sphere, to add layers of security to the system such as Azure Sphere–certified chips (HW), Azure Sphere OS, and Azure Sphere Security Service. ThreadX itself doesn’t seem to provide many security features unique to the system, only properties that are inherently included. ThreadX does have a semaphore implementation <sup>[[3c]](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_semaphore_initialize.c)</sup>, mutex implementation <sup>[[4c]](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_mutex_initialize.c)</sup>, and preemption-threshold scheduling implementation<sup>[[5c]](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_thread_relinquish.c)</sup>, which is impressive for a RTOS. However, there is a large issue with the memory management for threads as it is easily possible to overflow the stack. More worrisome than the system simply crashing or exiting due to this overflow, is the fact that the results are unpredictable, typically resulting in an unnatural change in the program counter. This is often called "jumping into the weeds." The only way to prevent this is to ensure all thread stacks are large enough <sup>[[6c]](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls)</sup>. Similarly, there is an issue with thread priority, as ThreadX provides a priority-based, preemptive scheduling algorithm, which could lead to starvation. Misuse of thread priorities can starve other threads, create priority inversion, reduce processing bandwidth, and make the application's run-time behavior difficult to understand. Another possible issue is priority inversion. Priority inversion takes place when a higher priority thread is suspended because a lower priority thread has a needed resource <sup>[[7c]](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-priority-pitfalls)</sup>. If threads of intermediate priority become active during this priority inversion condition, the priority inversion time is no longer deterministic and could cause an application failure. To some extent these issues appear with any system containing multi threading, however, I think that ThreadX should have better security for these cases by having safe exit/end behavior whenever possible.

## Q8  
Azure RTOS ThreadX has several optimizations to speed up execution times. The first is sub-microsecond context switch on most popular processors and is significantly faster overall than other commercial RTOSes. In addition to being fast, Azure RTOS ThreadX is also highly deterministic. It achieves the same fast performance whether there are 200 threads ready, or just one. Additionally, Azure RTOS ThreadX boots in less than 120 cycles. The design called a Picokernel™ Design that makes services are not layered on each other, thus eliminating unnecessary function call overhead.As well as, optimized interrupt processing where only scratch registers are saved/restored upon ISR entry/exit, unless preemption is necessary <sup>[[8c]](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution)</sup>. Additionally, the TLS Implementation contains an internal certificate buffer optimization <sup>[[9c]](https://docs.microsoft.com/en-us/azure/rtos/netx-duo/netx-secure-tls/chapter3)</sup>. There is also an optimization in LevelX NOR flash memory as the 32-bit Minimum Mapped Sector and Maximum Mapped Sector fields are useful for optimization of the sector read operation <sup>[[10c]](https://docs.microsoft.com/en-us/azure/rtos/levelx/chapter5)</sup>.

## Q9  
Azure RTOS ThreadX writes themselves that the use case for this OS is very specific and narrow in scope, namely deeply embedded applications, and I agree with that assessment. I think that the system could better address security concerns (mentioned in question 7) than simply saying avoid doing this. It’s difficult to tell what amount of the information in the docs is marketing vs what’s true but based on the performance metrics, the certifications, and the fact that it can easily connect to be part of a larger ecosystem, ThreadX is a very good OS for its specific use case.

## Conclusion  
Azure RTOS ThreadX is a highly specialized RTOS that is optimized for the specific use case of a deeply embedded system. One of biggest challenges faced by IoT systems, that we have not addressed through our responses, is system updates; Azure RTOS ThreadX handles this through Azure IoT Hub. 

## Sources
### Sutton
1a: https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter1#multitasking  
2a: https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#preemption-threshold  
3a: https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1  
4a: https://militaryembedded.com/comms/rf-and-microwave/case-bullseye-million-miles-away  
5a: https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common/inc/tx_trace.h#L223-L227  
6a: https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1)  
7a: https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter4  
8a: https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter3  
9a: https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter3#module-manager-example  
10a: https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common_modules/module_manager/src/txm_module_manager_absolute_load.c#L403-L414  
### Becky
1b
2b
3b
### Tuhina
1c: https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#pre-certified-by-tuv-and-ul-to-many-safety-standards  
2c: https://github.com/azure-rtos/threadx#security  
3c: https://github.com/azure-rtos/threadx/blob/master/common/src/tx_semaphore_initialize.c  
4c: https://github.com/azure-rtos/threadx/blob/master/common/src/tx_mutex_initialize.c  
5c: https://github.com/azure-rtos/threadx/blob/master/common/src/tx_thread_relinquish.c  
6c: https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls  
7c: https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-priority-pitfalls  
8c: https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution  
9c: https://docs.microsoft.com/en-us/azure/rtos/netx-duo/netx-secure-tls/chapter3  
10c: https://docs.microsoft.com/en-us/azure/rtos/levelx/chapter5  

## Work Breakdown
Joint: Intro, Conclusion   
Sutton: Report Q 1-3, initial research for Q 7-9  
Becky: Report Q 4-6, initial research for Q 1-3  
Tuhina: Report Q 7-9, initial research for Q 4-6

