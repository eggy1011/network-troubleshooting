# Network Troubleshooting Lab: Two Routing Faults, Two Learning Stages

I used Cisco Packet Tracer to investigate two routing faults between simulated sites: an incorrect static route that caused intermittent packet loss, and a missing return route that prevented connectivity when I changed the ping source.

This project also records how I learned. **V1 was a difficult first attempt with substantial AI support. V2 was an independent repeat investigation, using the OSI model to structure my reasoning.**

## V1 — Learning the foundations with AI support

When I started, I did not have the networking foundations needed to troubleshoot this lab independently. IP addressing, subnet masks, next hops, routing tables, ARP and even the meaning of ping output were unfamiliar or difficult for me to connect together.

That made the first investigation particularly challenging. I was learning the basic concepts at the same time as trying to understand the fault.

I relied heavily on Claude throughout V1. It helped me interpret command output, understand unfamiliar concepts, develop possible explanations and decide what to test next. I often needed explanations broken down into smaller steps before I could understand why a packet would succeed or fail.

I operated Packet Tracer and ran the commands myself, but the reasoning process was substantially AI-assisted. I do not present V1 as an independently solved troubleshooting exercise. Its value was building enough understanding to explain the results and revisit the problem myself.

The [V1 investigation log](log-v1.md) records the tests, reported results, interpretations and fixes.

## V2 — Repeating the investigation independently

After completing V1, I worked through the troubleshooting again on my own, without AI guiding the troubleshooting steps. This time, I used the OSI model to organise my investigation and explain why each check mattered.

I already knew the faults from the first attempt. The purpose of V2 was to check whether I could apply what I had learned independently and explain the packet's journey, including its return path.

| Focus | How it structured my thinking |
|---|---|
| **Layer 1 — Physical** | Start with the devices, links and interface state. An operational link is a baseline, not proof of end-to-end connectivity. |
| **Layer 2 — Data link** | Consider how a router reaches the selected next hop on an Ethernet link, including the role of ARP. Distinguish a possible neighbour-resolution issue from an incorrect route. |
| **Layer 3 — Network** | Identify the source and destination, check addresses and subnet masks, and follow route selection at each hop for both the request and the reply. |
| **Layers 4–7 — Transport through application** | Recognise the limits of the test. An ICMP ping does not establish that a TCP/UDP service or application works. |

Using OSI helped me narrow the investigation. Both confirmed faults were at Layer 3; I did not need to force an unrelated test into every layer.

The [V2 independent review log](log-v2.md) explains this reasoning in more detail. It distinguishes known V1 results from replay expectations and identifies second-run outputs that still need to be attached.

## Lab setup

| Device | Simulated internal endpoint | Provider-facing connection |
|---|---|---|
| R1 — Site A | `10.1.1.1`, representing `10.1.1.0/24` | R1 `1.1.1.1` connects to provider `1.1.1.2` |
| R2 — Site B | `20.1.1.1`, representing `20.1.1.0/24` | R2 `2.2.2.1` connects to provider `2.2.2.2` |

R1 and R2 communicate through the middle provider router, labelled `MPLS`. This lab investigates **IP routing**, not MPLS label switching, LDP or VRFs. The internal endpoints are represented by router interface addresses; they are not separate user workstations.

## Findings and fixes

### Fault 1 — Incorrect equal-cost route on R1

A standard ping from R2 to `10.1.1.1` showed alternating replies and timeouts, represented by `!.!.!` (`!` = reply received; `.` = timeout).

The source was R2's outgoing interface, `2.2.2.1`, so R1 needed a route back to that address. R1 had two equal-preference static routes to `2.2.2.0/24`:

| Next hop | Finding |
|---|---|
| `1.1.1.2` | Valid provider-router address |
| `1.1.1.100` | Incorrect next hop with no valid router at that address |

In this Packet Tracer scenario, traffic distribution across those next hops explained the alternating loss. This does not mean equal-cost routing always alternates individual packets on real equipment.

I removed the incorrect route in R1's global configuration mode:

```text
no ip route 2.2.2.0 255.255.255.0 1.1.1.100
```

The repeated standard ping then returned `!!!!!`.

### Fault 2 — Missing Site B return route on the provider

The successful standard ping did not test the return route to Site B's internal address. Using extended ping, I changed the source to `20.1.1.1` while keeping the destination as `10.1.1.1`. That test returned `.....` — 0% success.

The provider had no matching route to `20.1.1.1`, including no usable default route. It could reach R2's connected WAN network, but could not forward replies to the simulated internal network.

I added the following route in the provider's global configuration mode:

```text
ip route 20.1.1.0 255.255.255.0 2.2.2.1
```

The source-specific test succeeded after that correction.

## Verification and scope

| Test from the original investigation | Result after the relevant fix | What it demonstrated |
|---|---|---|
| Source `2.2.2.1`, destination `10.1.1.1` | Successful after removing the incorrect R1 route | ICMP reachability between R2's WAN address and the Site A endpoint |
| Source `20.1.1.1`, destination `10.1.1.1` | Successful after adding the provider route | ICMP reachability between the simulated internal endpoints |

These results support the two routing fixes. They do not validate actual workstation connectivity or application behaviour. The logs are retrospective records; representative output and expected results are labelled rather than presented as complete terminal captures.

## What I learned

- **Define the test before interpreting it.** A ping's source address determines where its reply must return.
- **Follow both directions.** Every router makes a routing decision for the current destination; a working forward path does not guarantee a working return path.
- **Use observations to test hypotheses.** Alternating loss suggested multiple forwarding paths, but the routing findings and retest were needed to support that explanation.
- **Use OSI to organise the questions.** Distinguishing link state, local delivery, routing and application behaviour made the investigation easier to explain.
- **Check understanding through independent practice.** V1 gave me supported exposure to unfamiliar concepts. V2 helped me apply those concepts myself and identify the limits of what my tests proved.

## Lab credit

The original broken Packet Tracer topology was created by **SnehaSugilal**, based on a troubleshooting scenario by **PM Networking**:

[SnehaSugilal/Troubleshooting_Scenario_60-Packet-Loss](https://github.com/SnehaSugilal/Troubleshooting_Scenario_60-Packet-Loss)

I used an existing broken lab so that I did not begin V1 knowing the fault. The `.pkt` file is not redistributed here. Credit for the original scenario and topology belongs to its creators; this repository documents my investigation and learning.
