## Intro  
ThreadX is a deterministic, embedded RTOS created by Express Logic (acquired by Microsoft) that provides "preemption-threshold, priority inheritance, efficient timer management, fast software timers, picokernel design, event-chaining, and small size: minimal size on an ARM architecture processor is about 2 KB." [referenced from Wikipedia]( https://en.wikipedia.org/wiki/ThreadX). ThreadX as described by [Microsoft](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#azure-rtos-threadx-services): Azure RTOS ThreadX provides advanced scheduling, communication, synchronization, timer, memory management, and interrupt management facilities. It is important to note that ThreadX is a component of the larger Azure RTOS. 

## Q1  

## Q2 

## Q3  

## Q4  

## Q5  

## Q6  

## Q7 
When searching Azure RTOS ThreadX documentation for security features, I found that it has several security certifications such as EAL4+ Common Criteria. The Target of Evaluation (TOE) covers Azure RTOS ThreadX, Azure RTOS NetX-Duo, Azure RTOS NetX Secure TLS, and Azure RTOS NetX MQTT. This represents the most typical IoT protocols required by deeply embedded sensors, devices, edge routers, and gateways.The IT Security Evaluation Facility used for the Azure RTOS security certification is Brightsight BV and the Certification Authority is SERTIT. <sup>[(1c)](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#pre-certified-by-tuv-and-ul-to-many-safety-standards)</sup>. The documentation also mentions that ThreadX supports IP layer security (IPsec) and socket layer security (TLS and DTLS) protocols, is tested and certified to meet international security assurance requirements, and integrated with Azure Defender to detect threats and remediate issues. The ThreadX repository on Github states that Azure RTOS provides OEMs with components to secure communication and to create code and data isolation using underlying MCU/MPU hardware protection mechanisms <sup>[(2c)](https://github.com/azure-rtos/threadx#security)</sup>. In addition to the security provided by Azure RTOS ThreadX, there are additional compatible services available, termed Azure Sphere, to add layers of security to the system such as Azure Sphere–certified chips (HW), Azure Sphere OS, and Azure Sphere Security Service. ThreadX itself doesn’t seem to provide many security features unique to the system, only properties that are inherently included. ThreadX does have a semaphore implementation <sup>[(3c)](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_semaphore_initialize.c)</sup>, mutex implementation <sup>[(4c)](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_mutex_initialize.c)</sup>, and preemption-threshold scheduling implementation<sup>[(5c)](https://github.com/azure-rtos/threadx/blob/master/common/src/tx_thread_relinquish.c)</sup>, which is impressive for a RTOS. However, there is a large issue with the memory management for threads as it is easily possible to overflow the stack. More worrisome than the system simply crashing or exiting due to this overflow, is the fact that the results are unpredictable, typically resulting in an unnatural change in the program counter. This is often called "jumping into the weeds." The only way to prevent this is to ensure all thread stacks are large enough <sup>[(6c)](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls)</sup>. Similarly, there is an issue with thread priority, as ThreadX provides a priority-based, preemptive scheduling algorithm, which could lead to starvation. Misuse of thread priorities can starve other threads, create priority inversion, reduce processing bandwidth, and make the application's run-time behavior difficult to understand. Another possible issue is priority inversion. Priority inversion takes place when a higher priority thread is suspended because a lower priority thread has a needed resource <sup>[(7c)](https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-priority-pitfalls)</sup>. If threads of intermediate priority become active during this priority inversion condition, the priority inversion time is no longer deterministic and could cause an application failure. To some extent these issues appear with any system containing multi threading, however, I think that ThreadX should have better security for these cases by having safe exit/end behavior whenever possible.

## Q8  
Azure RTOS ThreadX has several optimizations to speed up execution times. The first is sub-microsecond context switch on most popular processors and is significantly faster overall than other commercial RTOSes. In addition to being fast, Azure RTOS ThreadX is also highly deterministic. It achieves the same fast performance whether there are 200 threads ready, or just one. Additionally, Azure RTOS ThreadX boots in less than 120 cycles. The design called a Picokernel™ Design that makes services are not layered on each other, thus eliminating unnecessary function call overhead.As well as, optimized interrupt processing where only scratch registers are saved/restored upon ISR entry/exit, unless preemption is necessary <sup>[(8c)](https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution)</sup>. Additionally, the TLS Implementation contains an internal certificate buffer optimization <sup>[(9c)](https://docs.microsoft.com/en-us/azure/rtos/netx-duo/netx-secure-tls/chapter3)</sup>. There is also an optimization in LevelX NOR flash memory as the 32-bit Minimum Mapped Sector and Maximum Mapped Sector fields are useful for optimization of the sector read operation <sup>[(10c)](https://docs.microsoft.com/en-us/azure/rtos/levelx/chapter5)</sup>.

## Q9  
Azure RTOS ThreadX writes themselves that the use case for this OS is very specific and narrow in scope, namely deeply embedded applications, and I agree with that assessment. I think that the system could better address security concerns (mentioned in question 7) than simply saying avoid doing this. It’s difficult to tell what amount of the information in the docs is marketing vs what’s true but based on the performance metrics, the certifications, and the fact that it can easily connect to be part of a larger ecosystem, ThreadX is a very good OS for its specific use case.

## Conclusion  
Azure RTOS ThreadX is a highly specialized RTOS that is optimized for the specific use case of a deeply embedded system. One of biggest challenges faced by IoT systems, that we have not addressed through our responses, is system updates; Azure RTOS ThreadX handles this through Azure IoT Hub. 

## Sources
### Sutton
1a
2a
3a
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
Sutton: Q 1-3  
Becky: Q 4 - 6  
Tuhina: Q 7- 9  
