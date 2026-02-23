# RESULTS.md — Lottery Scheduling (xv6)

## Setup
I wrote a small user program testlottery that creates **3 child processes** using fork().

Ticket counts:
- Child A: 10 tickets
- Child B: 20 tickets
- Child C: 70 tickets
(Total tickets = 100)

Each child calls settickets(tickets[i]) right after it starts, so the ticket count applies to the calling process.

## Workload
Each child runs a CPU-bound loop.  
It measures a fixed runtime using uptime() and increments a counter as fast as possible:

- Run length: RUN_TICKS = 2000
- “Work unit”: work++ in a tight loop while uptime() - start < RUN_TICKS

At the end, each child prints:
tickets=<n> work=<count>

The work counter is used as an approximation of how much CPU time the process received.

## Observed Results (3 runs)

### Run 1
- tickets=10 work=4,145,174  
- tickets=20 work=7,269,568  
- tickets=70 work=9,540,573  
Total work = 20,955,315

Observed shares:
- 10 tickets: 4,145,174 / 20,955,315 = 19.78%
- 20 tickets: 7,269,568 / 20,955,315 = 34.69%
- 70 tickets: 9,540,573 / 20,955,315 = 45.53%

### Run 2
- tickets=10 work=4,143,333  
- tickets=20 work=7,266,262  
- tickets=70 work=9,541,848  
Total work = 20,951,443

Observed shares:
- 10 tickets: 19.78%
- 20 tickets: 34.68%
- 70 tickets: 45.54%

### Run 3
- tickets=10 work=4,154,375  
- tickets=20 work=7,284,821  
- tickets=70 work=9,564,764  
Total work = 21,003,960

Observed shares:
- 10 tickets: 19.78%
- 20 tickets: 34.68%
- 70 tickets: 45.54%

### Average over 3 runs
Average observed shares:
- 10 tickets: 19.78%
- 20 tickets: 34.69%
- 70 tickets: 45.54%

In all runs, the process with 70 tickets consistently received the most work, followed by 20, then 10, which matches the intended direction of lottery scheduling.

## Notes on variance and why longer runs converge
Lottery scheduling is probabilistic: every scheduling decision is like a random draw weighted by ticket counts. Because of randomness, short runs can be noisy, and the observed CPU shares will not perfectly match the expected ticket ratios.

Even though the results above are stable across runs, they still show variance compared to the ideal 10/20/70 split. In general, increasing the runtime (larger RUN_TICKS) and averaging more trials reduces randomness and makes the observed shares move closer to the expected proportions. This happens because as the number of lottery draws grows, the “luck” from random selection averages out over time.