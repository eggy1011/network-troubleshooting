# V1 — My First Troubleshooting Investigation

## Where I started

I found this lab difficult because I was learning the networking basics while trying to solve it. Routing tables, subnet masks, ARP and ping source addresses were not concepts I could confidently connect together yet.

I used Claude extensively as a tutor. It explained concepts, helped me interpret output and suggested where to look next. I ran the commands myself, wrote predictions before tests and came up with the idea that packets might be taking one correct path and one incorrect path. When something did not make sense, I asked more questions and tested again.

I wrote this log afterwards from my notes. The short outputs below show the patterns I recorded, rather than complete terminal sessions.

## My setup

| Device | Internal endpoint | Connection to the provider |
|---|---|---|
| R1 — Site A | `10.1.1.1` in `10.1.1.0/24` | R1 `1.1.1.1`; provider `1.1.1.2` |
| R2 — Site B | `20.1.1.1` in `20.1.1.0/24` | R2 `2.2.2.1`; provider `2.2.2.2` |

I used router interface addresses to represent the internal endpoints. The middle router was named `MPLS`, but I was troubleshooting its IP routes.

## 1. I checked the links and reproduced the problem

The links looked operational in Packet Tracer. I then ran:

```text
R2# ping 10.1.1.1
```

I got partial success. Across repeated runs, I recorded success rates including 20%, 60%, 40% and 60%.

At first, I did not understand why the result kept changing. Looking at the individual replies helped more than looking at the percentage:

```text
!.!.!.!.!.
```

I learned that `!` meant a reply arrived and `.` meant a timeout. The repeated alternation made me think of a delivery person taking the correct road once, then the wrong road next time.

**My hypothesis:** Somewhere in the network, traffic might be split between a working next hop and a failing next hop.

## 2. I checked whether R2 had two paths

```text
R2# show ip route
```

I found one relevant default route toward the provider at `2.2.2.2`.

That did not support my idea of two competing routes on R2. I kept the hypothesis, but needed to look elsewhere in the packet's journey.

## 3. I learned why the ping source mattered

This was one of the parts I could not work out alone. I thought that a ping from R2 automatically represented traffic from Site B's internal address.

With AI's help, I understood the actual addresses:

| Packet | Source | Destination |
|---|---|---|
| My standard ping request | `2.2.2.1` | `10.1.1.1` |
| Its reply | `10.1.1.1` | `2.2.2.1` |

R2 was using its outgoing interface address. I therefore needed to check how R1 reached `2.2.2.1`, rather than assuming the reply was going to `20.1.1.1`.

## 4. I found two return routes on R1

```text
R1# show ip route
```

I found these two equal-preference routes, shown here in simplified form:

```text
2.2.2.0/24 via 1.1.1.2
2.2.2.0/24 via 1.1.1.100
```

I recognised `1.1.1.2` as the provider's address. There was no valid router at `1.1.1.100`.

This matched my two-path hypothesis: in this lab, some replies used the valid next hop and others went toward the invalid one. I learned that this particular alternating pattern is not how every real device handles equal-cost routes.

**My prediction:** If I removed only the incorrect route, the original ping should stop alternating.

## 5. I removed the incorrect route and tested again

On R1, I entered:

```text
configure terminal
no ip route 2.2.2.0 255.255.255.0 1.1.1.100
end
```

I repeated the standard R2 ping. The repeated test returned:

```text
!!!!!
```

The original symptom was fixed. But I now understood that this test still used `2.2.2.1` as its source.

## 6. I tested from the internal Site B address

I tried to specify the source in an inline ping command, but Packet Tracer did not accept the syntax. I switched to interactive extended ping:

```text
R2# ping
Protocol [ip]:
Target IP address: 10.1.1.1
Repeat count [5]:
Datagram size [100]:
Timeout in seconds [2]:
Extended commands [n]: y
Source address or interface: 20.1.1.1
```

I accepted the other defaults.

**My expectation:** Since I had fixed the bad route, I expected this test to work too.

**My result:**

```text
.....
```

I got 0% success. That showed me that my first repair had not solved every routing problem.

## 7. I checked the new return destination

The reply now needed to reach `20.1.1.1`. I checked the provider's routing table:

```text
MPLS# show ip route
```

I found no matching route to that address and no usable default route. The provider could reach R2's WAN network directly, but did not know how to reach the internal Site B network.

**My prediction:** Adding a route to `20.1.1.0/24` through R2 should allow the reply to return.

## 8. I added the missing route

On the provider, I entered:

```text
configure terminal
ip route 20.1.1.0 255.255.255.0 2.2.2.1
end
```

I repeated the extended ping from `20.1.1.1` to `10.1.1.1`, and it succeeded.

## My result and next step

I resolved two separate faults: an incorrect route on R1 and a missing route on the provider. The most useful lesson was understanding why changing the source changed the return path I was testing.

I still needed a lot of help during this first attempt. My next step was to revisit the lab independently and use the OSI model to organise my reasoning.
