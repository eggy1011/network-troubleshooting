# Troubleshooting Lab Log: broken_net.pkt

Source: Recreated by [SnehaSugilal](https://github.com/SnehaSugilal/Troubleshooting_Scenario_60-Packet-Loss) from a PM Networking scenario. I solved it without looking at the solution.

## Step 1 - What I see (before touching anything)

- **Devices and names:** 3 Cisco 2901 routers: R1 (Site A), MPLS (service provider in the middle), R2 (Site B).
- **How they are connected:** R1 -- MPLS -- R2
- **Link lights (green/red):** All 4 link lights are green, so the physical links are up.
- **My first impression:** The cables are fine (it's green), so if something is broken, it's probably not a physical problem.

## Step 2 - First test: ping from R2 to Site A (10.1.1.1)

**My prediction:** I think it might fail

- **Actual result (test 1):** `...!.` -> 20% (1/5)

**Next Step:** Repeat the ping test without changing the configuration to check whether the issue persists.

- **Actual result (test 2):** `!.!.!` -> 60% (3/5)
- **Actual result (test 3):** `.!.!.` -> 40% (2/5)
- **Actual result (test 4):** `!.!.!` -> 60% (3/5)

**Why test 1 was different:** TODO - write in my own words (ARP)

**What I first thought:** The problem was getting worse (60% -> 40%).

**What I found:** When I put test 2, test 3, and test 4 together, it's a single continuous alternating pattern. The success rate only changed because of where each test started in the pattern.

**My hypothesis:** The "courier" takes the correct path one time and a wrong path the next time, taking turns. That would explain the perfect alternating pattern.

## Step 3 - Checking routing tables hop by hop

**R2 (`show ip route`):**

- R2 has no specific route to 10.1.1.0, so it uses the default route: `0.0.0.0/0 via 2.2.2.2` (MPLS).
- Only ONE path. R2 is not the one alternating.
- Conclusion: R2 is fine. Next hop to check: MPLS.
