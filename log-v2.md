# V2 — My Independent Review Using the OSI Model

## What I wanted to check

After V1, I worked through the troubleshooting on my own, without AI guiding me. I already knew the faults; I wanted to explain the reasoning independently.

I used the OSI model to organise my thinking. In this log, I focus on my second-pass reasoning rather than repeat V1's numerical results.

## 1. I defined exactly what I was testing

My first question was: “Which address am I testing from?”

I separated the two tests before interpreting any result:

| My test | Source | Destination of the reply |
|---|---|---|
| Standard R2 ping to `10.1.1.1` | `2.2.2.1` | `2.2.2.1` |
| Extended ping to `10.1.1.1` | `20.1.1.1` | `20.1.1.1` |

This was a major change from V1. I could now explain why two pings to the same destination could test different return routes.

## 2. I started with the lower layers

At **Layer 1**, I focused on links and interface state. At **Layer 2**, I asked how a router reaches its selected next hop on an Ethernet link.

I connected this to ARP: R1 needs the MAC address of its local next hop, such as `1.1.1.2`, rather than the remote Site B endpoint.

These were the checks I associated with those questions:

```text
show ip interface brief
show interfaces
show arp
```

I understood that a green link does not prove correct routing.

## 3. I traced Layer 3 in both directions

I used the routing table to ask the same question at each router:

> “For this destination, which next hop would I choose?”

For the request, I followed R2, the provider and R1 toward `10.1.1.1`. For the reply, I started again with the original source as the new destination.

I connected `20.1.1.1` with its `20.1.1.0/24` subnet. R2 having a default route did not give the provider a return route.

## 4. I explained each fault before thinking about its fix

| Fault I revisited | My explanation | The change I expected to help |
|---|---|---|
| Incorrect R1 route through `1.1.1.100` | Replies to R2's WAN address could be sent toward the wrong next hop | Remove that route and keep the valid route through `1.1.1.2` |
| Missing provider route to `20.1.1.0/24` | Replies to the internal Site B address had no matching route on the provider | Add the route through R2 at `2.2.2.1` |

I could also separate the root cause from a possible consequence. If an incorrect route selects a nonexistent Ethernet neighbour, ARP may fail. The underlying configuration fault is still the incorrect route.

## 5. I made my verification more specific

I treated the standard ping and the source-specific ping as separate checks. Fixing the route to `2.2.2.0/24` would not create the missing route to `20.1.1.0/24`.

I kept my conclusion at Layer 3: ICMP reachability does not prove that TCP ports, logins or applications work.

## What I could explain on my own this time

I could connect route selection, local next-hop delivery and the return path into one explanation. I could say which test each fix should affect, rather than simply repeat the repair commands.

V1 helped me learn the concepts with substantial support. V2 helped me organise and apply them independently. My next challenge is to use the same process on a lab where I do not already know the faults.
