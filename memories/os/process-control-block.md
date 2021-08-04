## Process Control Block 

#### PCB 가 포함하는 항목은 다음과 같다. 

- Process scheduling state

  - The state of the process in terms of "ready", "suspended", etc., and other scheduling information as well, such as priority value, the amount of time elapsed since the process gained control of the CPU or since it was suspended. Also, in case of a suspended process, event identification data must be recorded for the event the process is waiting for.

- Process structuring information

  - the process's children id's, or the id's of other processes related to the current one in some functional way, which may be represented as a queue, a ring or other data structures

- Interprocess communication information

  - flags, signals and messages associated with the communication among independent processes

- Process Privileges

  - allowed/disallowed access to system resources

- Process State
  - new, ready, running, waiting, dead

- Process Number (PID)

  - unique identification number for each process (also known as Process ID)

- Program Counter (PC)
  
  - A pointer to the address of the next instruction to be executed for this process

- CPU Registers
  
  - Register set where process needs to be stored for execution for running state

- CPU Scheduling Information
  
  - information scheduling CPU time

- Memory Management Information

  - page table, memory limits, segment table

- Accounting Information
 
  - amount of CPU used for process execution, time limits, execution ID etc.

- I/O Status Information

  - list of I/O devices allocated to the process.