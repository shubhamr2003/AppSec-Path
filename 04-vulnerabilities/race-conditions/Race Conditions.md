sources:
- [geek4geeks](https://www.geeksforgeeks.org/operating-systems/race-condition-vulnerability/)
- 
yt:
- [practical](https://www.youtube.com/watch?v=CcjwG5nDBx4)
- 

A race condition vulnerability occurs when two or more processes or threads access shared resources simultaneously without proper synchronization, causing the program's behavior to depend on the order of execution. Such vulnerabilities can lead to data corruption, privilege escalation, inconsistent program states, and security breaches.

- Race conditions occur due to uncontrolled concurrent access to shared resources.
- They can affect files, memory, threads, processes, and operating system resources.

Common manifestations include:
- **Concurrent Access**: Multiple threads modify shared data (e.g., bank balances, counters) without atomic operations, leading to incorrect states.
- **Business Logic Flaws**: Exploiting parallel requests to bypass rate limits, redeem single-use coupons multiple times, or access resources before permissions are enforced.
- **System-Level Risks**: In operating systems, unsynchronized access to files or memory can lead to crashes, denial of service, or unauthorized access to protected resources.

Bug Bounty Test Cases:

- Double Coupon Redemption: CAN THE SAME COUPON BE REDEEMED MULTIPLE TIMES SIMULTANEOUSLY
- Bypass Trail Feature: WHETHER SOME PAID FEATURE CAN BE USED BEYOND THEIR LIMITS
- Account Registration: SEND CONCURRENT REGISTRATION REQUESTS USING THE SAME EMAIL/USERNAME
- File upload: PERFORM CONCURRENT UPLOADS WHERE THE APPLICATION GENERATES A UNIQUE FILENAME OR CHECKS PERMISSIONS BEFORE WRITING.

