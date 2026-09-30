# Network Troubleshooting Lab: Intermittent Packet Loss Between Two Sites

> **Status: In progress.** I'm documenting each step as I go. The full investigation, including wrong turns, is in [log.md](log.md).

## The problem

Two offices are connected through a service provider network. All the links are up, but when Site B pings Site A, only some of the packets get through. The goal is to find out why, fix it, and verify the fix.

I don't come from a networking background. I chose this lab because the fault was built by someone else, so I didn't know the answer going in.

## Lab file

The broken network is a Cisco Packet Tracer file created by **SnehaSugilal**, recreated from a scenario by **PM Networking**. Download it from the original repository:

[SnehaSugilal/Troubleshooting_Scenario_60-Packet-Loss](https://github.com/SnehaSugilal/Troubleshooting_Scenario_60-Packet-Loss)

If you want to try it yourself, don't open the solution section in that README.

I haven't included the `.pkt` file here because the original repository has no license. All credit for the lab design goes to its authors.

## Topology

```
[Site A]  ---  R1  ========  MPLS  ========  R2  ---  [Site B]
10.1.1.0/24      1.1.1.1   1.1.1.2  2.2.2.2   2.2.2.1      20.1.1.0/24
```

- Three Cisco 2901 routers: **R1** (Site A), **MPLS** (the service provider), **R2** (Site B)
- R1 and R2 are not directly connected. All traffic passes through MPLS.

## How I'm approaching it

1. **Write a prediction before every test**, so I can compare what I expected with what actually happened.
2. **Reproduce the problem** before trying to explain it.
3. **Look at the raw data**, not just the summary number.
4. **Follow the packet hop by hop** (R2 → MPLS → R1 and back), checking each router's routing table.
5. **Change one thing at a time**, then test again.
6. **Verify the fix**, and check that nothing else broke.

## Progress so far

| Step | What I did | What I learned |
|---|---|---|
| 1 | Observed the topology | All links were green, so the problem is probably not physical |
| 2 | Pinged Site A from R2 four times | Success rates of 20%, 60%, 40%, 60% looked random, but the raw results formed one perfect alternating pattern: `!.!.!.!.!.` |
| 3 | Formed a hypothesis | Something in the network may be alternating between a correct and a wrong path |
| 4 | Checked R2's routing table | R2 has only one path to Site A (a default route to MPLS), so R2 is not the one alternating |
| 5 | Check MPLS's routing table | *Next* |

## Root cause, fix and verification

*To be added once I've found them.*

## What I learned

*To be added.*

## How I used AI

I used Claude as a tutor. It explained concepts in plain language (routing tables, ARP, how to read ping output) and suggested what to check next. I ran every command myself, wrote every prediction and hypothesis before seeing the result, and recorded what actually happened, including where my first reading was wrong.

I also chose this lab over earlier project ideas the AI suggested, because those ideas didn't have a real unknown to investigate.
