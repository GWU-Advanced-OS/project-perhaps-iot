## Questions  

1. Summarize the project, what it is, what its goals are, and why it exists.
2. What is the target domain of the system? Where is it valuable, and where is it not a good fit? These are all implemented in an engineering domain, thus are the product of trade-offs. No system solves all problems (despite the claims of marketing material).
3. What are the "modules" of the system (see early lectures), and how do they relate? Where are isolation boundaries present? How do the modules communicate with each other? What performance implications does this structure have?
4. What are the core abstractions that the system aims to provide. How and why do they depart from other systems?
5. In what conditions is the performance of the system "good" and in which is it "bad"? How does its performance compare to a Linux baseline (this discussion can be quantitative or qualitative)?
6. What are the core technologies involved, and how are they composed?
7. What are the security properties of the system? How does it adhere to the principles for secure system design? What is the reference monitor in the system, and how does it provide complete mediation, tamperproof-ness, and how does it argue trustworthiness?
8. What optimizations exist in the system? What are the "key operations" that the system treats as a fast-path that deserve optimization? How does it go about optimizing them?
9. Subjective: What do you like, and what don't you like about the system? What could be done better were you to re-design it?


## Hypothesis
#### Our hypothesized answers to the above questions, with links to docs/code that gives us this impression.
1. Answer
2. Answer
3. Answer
4. ThreadX aims to provide scheduling, communication, synchronization, timer, memory management, and interrupt management facilities for embedded, real-time, and IoT applications. ThreadX is a component of the larger Azure RTOS (as seen in this dependency diagram: https://github.com/azure-rtos/threadx#understanding-inter-component-dependencies). This design differs from more traditional OS's like Linux and Plan9., which have a more monolithic architecture.  
5. ThreadX is a specialized OS (use case is embedded, real-time, IoT) and benchmarking places it to be faster than most other commercial RTOSes (benchmarking data: https://docs.microsoft.com/en-us/azure/rtos/threadx/overview-threadx#fast-execution ). Since it has a specific use case, if the conditions aren't in line with the use case, the perfance will likely be sub optimal since the system is so specialized. It's difficult to compare Linux and ThreadX since the use cases are very different, but I did find a paper (https://webthesis.biblio.polito.it/8505/1/tesi.pdf) that discusses benchmarking embedded Linux. Unfortunately, the same tests aren't conducted on both Linux and ThreadX so they can't directly be compared; however, the information found was still interesting.  
6. ThreadX consists of four types of program execution: Initialization, Thread Execution, Interrupt Service Routines (ISRs), and Application Timers. Each of these execution modes consists of different components, but generally each mode consists of memory usage (static, dynamic), interrupts, threads, application definition, preemption, scheduling algorithms (round robin, time slice), and message queue management.  
7. Answer
8. Answer
9. Answer
