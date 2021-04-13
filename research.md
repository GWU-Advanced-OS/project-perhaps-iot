## Becky
Linked from hypothesis.md:
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter1#multitasking
* https://www.cs.utah.edu/~regehr/reading/open_papers/preempt_thresh.pdf
* https://www.embedded.com/bill-lamie-story-of-a-man-and-his-real-time-operating-systems/
* https://militaryembedded.com/comms/rf-and-microwave/case-bullseye-million-miles-away
* https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter1
* https://docs.microsoft.com/en-us/azure/rtos/threadx-modules/chapter4
## Sutton
Security concerns:
* https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#memory-protection-via-azure-rtos-threadx-modules
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#thread-scheduling
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#time-slicing
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#deadly-embrace
* https://docs.microsoft.com/en-us/azure/rtos/threadx/chapter3#memory-pitfalls

System Optimizations:
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
