## Becky
Linked from hypothesis.md:
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter1#multitasking
* https://www.cs.utah.edu/~regehr/reading/open_papers/preempt_thresh.pdf
* https://www.embedded.com/bill-lamie-story-of-a-man-and-his-real-time-operating-systems/
* https://militaryembedded.com/comms/rf-and-microwave/case-bullseye-million-miles-away
* https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1
* https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter4
## Sutton
### Purpose within Systems
* ThreadX is most concerned with concurrency issues on real-time embedded systems. Support for this large amount of concurrency and priority-based solutions are evident when looking through the code base. Priority of threads and also the threshold to which is required for preemption is stored for each process currently running, which allows for the concurrency of the system. However, because the support for this is so vast, it is unlikely that this OS would be particularly useful for anyone who does not want to take advantage of this aspect of the OS. 
```
[Thread Priority or         4           This 4-byte field contains the current thread pointer for interrupt
             Current Thread                         events or the thread preemption-threshold/priority for thread events.
             Preemption-Threshold/
             Priority]
            [Event ID]         
```
<sup>https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common/inc/tx_trace.h#L223-L227</sup>

### Modules 
ThreadX is highly modularized and includes a memory manager within the kernel to keep track of these indivudal modules using a list 
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
<sup>[(src: txm_module_manager_absolute_load.c)](https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/common_modules/module_manager/src/txm_module_manager_absolute_load.c#L403-L414)</sup> 

Specific modules can run as memory protected by defining `TXM_MODULE_MEMORY_PROTECTION_ENABLED` without this definied, any modules that need memory protection to run will not run. <sup>[(src: txm_module_port.h)](https://github.com/azure-rtos/threadx/blob/d759e6bb9e040bc9f973ef706dc7b0a9c68be916/ports_module/cortex-a7/ac5/inc/txm_module_port.h#L89-L96)<sup>

### Security concerns
* https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#memory-protection-via-azure-rtos-threadx-modules
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-scheduling
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#time-slicing
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#deadly-embrace
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls

### System Optimizations
* https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#advanced-technology

## Tuhina
General Reading Sources:  
* https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx  
* https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution  
* https://github.com/azure-rtos/threadx#security  
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3  

7. When searching Azure RTOS ThreadX documentation for security features, I found that it has several security certifications such as EAL4+ Common Criteria. The Target of Evaluation (TOE) covers Azure RTOS ThreadX, Azure RTOS NetX-Duo, Azure RTOS NetX Secure TLS, and Azure RTOS NetX MQTT. This represents the most typical IoT protocols required by deeply embedded sensors, devices, edge routers, and gateways.The IT Security Evaluation Facility used for the Azure RTOS security certification is Brightsight BV and the Certification Authority is SERTIT. (https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#eal4-common-criteria-security-certification, https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#pre-certified-by-tuv-and-ul-to-many-safety-standards ). The documentation also mentions that ThreadX supports IP layer security (IPsec) and socket layer security (TLS and DTLS) protocols, is tested and certified to meet international security assurance requirements, and integrated with Azure Defender to detect threats and remediate issues.
The ThreadX repository on Github states that Azure RTOS provides OEMs with components to secure communication and to create code and data isolation using underlying MCU/MPU hardware protection mechanisms. (https://github.com/azure-rtos/threadx#security)
In addition to the security provided by Azure RTOS ThreadX, there are additional compatible services available, termed Azure Sphere, to add layers of security to the system such as Azure Sphere–certified chips (HW), Azure Sphere OS, and Azure Sphere Security Service. 
ThreadX itself doesn’t seem to provide many security features unique to the system, only properties that are inherently included. ThreadX does have a semaphore implementation(https://github.com/azure-rtos/threadx/blob/master/common/src/tx_semaphore_initialize.c), mutex implementation (https://github.com/azure-rtos/threadx/blob/master/common/src/tx_mutex_initialize.c), and preemption-threshold scheduling implementation(https://github.com/azure-rtos/threadx/blob/master/common/src/tx_thread_relinquish.c), which is impressive for a RTOS. 
However, there is a large issue with the memory management for threads as it is easily possible to overflow the stack. More worrisome than the system simply crashing or exiting due to this overflow, is the fact that the results are unpredictable, typically resulting  in an unnatural change in the program counter. This is often called "jumping into the weeds." The only way to prevent this is to ensure all thread stacks are large enough. (https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls ) Similarly, there is an issue with thread priority, as ThreadX provides a priority-based, preemptive scheduling algorithm, which could lead to starvation. Misuse of thread priorities can starve other threads, create priority inversion, reduce processing bandwidth, and make the application's run-time behavior difficult to understand. Another possible issue is priority inversion. Priority inversion takes place when a higher priority thread is suspended because a lower priority thread has a needed resource. (https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-priority-pitfalls) If threads of intermediate priority become active during this priority inversion condition, the priority inversion time is no longer deterministic and could cause an application failure. To some extent these issues appear with any system containing multi threading, however, I think that ThreadX should have better security for these cases by having safe exit/end behavior whenever possible.  
8. Azure RTOS ThreadX has several optimizations to speed up execution times. The first is sub-microsecond context switch on most popular processors and is significantly faster overall than other commercial RTOSes. In addition to being fast, Azure RTOS ThreadX is also highly deterministic. It achieves the same fast performance whether there are 200 threads ready, or just one. Additionally,  Azure RTOS ThreadX boots in less than 120 cycles. The design called a Picokernel™ Design that makes services are not layered on each other, thus eliminating unnecessary function call overhead.As well as, optimized interrupt processing where only scratch registers are saved/restored upon ISR entry/exit, unless preemption is necessary. (https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution). Additionally, the TLS Implementation contains an internal certificate buffer optimization. (https://docs.microsoft.com/en-us/azure/rtos/netx-duo/netx-secure-tls/chapter3) There is also an optimization in LevelX NOR flash memory as the 32-bit Minimum Mapped Sector and Maximum Mapped Sector fields are useful for optimization of the sector read operation. (https://docs.microsoft.com/en-us/azure/rtos/levelx/chapter5).  
9. Azure RTOS ThreadX writes themselves that the use case for this OS is very specific and narrow in scope, namely deeply embedded applications, and I agree with that assessment. I think that the system could better address security concerns (mentioned in question 7) than simply saying avoid doing this. It’s difficult to tell what amount of the information in the docs is marketing vs what’s true but based on the performance metrics, the certifications, and the fact that it can easily connect to be part of a larger ecosystem, ThreadX is a very good OS for its specific use case.  
