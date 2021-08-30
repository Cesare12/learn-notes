# Hardware Interrupts
## 1. Overview

                                    ____________             _____
               Timer ------------> |            |           |     |
               Keyboard ---------> | Interrupt  |---------> | CPU |
               Other Hardware ---> | Controller |           |_____|
               Etc --------------> |____________|


## 2. The 8259 PIC
### Implementation
## 3. Enable Timer Interrupts
### End of Interrupt
### Configuring the Timer
## 4. Deadlocks
### Provoking a Deadlock
### Fixing the Deadlock
## 5. Fixing a Race Conditon
## 6. The hlt Instruction
## 7. Keyboard Input
### Reading the Scancodes
### Interrupting the Scancodes
### Configuring the Keyboard
## 8. Summary
