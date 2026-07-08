### Global DPI Analysis: Why blocking internet services is a terrible idea for the censorship agency

> **Disclaimer:**  
> The material presented herein does not encourage any violation of the law.  
> It serves to highlight the risks that any nation employing these systems may encounter.  
> The author has not verified and had no opportunity to verify this material for compliance with the legislative specifics of every country in the world.

We’re seeing a global surge in invasive, DPI-based censorship. Whether it's mandatory passport-based identity checks or hunting down VPNs, the censorship agency's approach is becoming a global playbook. But once the censorship agency pushes too hard, the logical move for users is to go for user-defined VPNs. And that’s where things get interesting.

## Table of Contents
- [Implementation Reference](#implementation-reference)
- [The User's Asymmetric Advantage](#the-users-asymmetric-advantage)
- [What about non-technical users?](#what-about-non-technical-users)
- [Breaking the Censor’s Workflow](#breaking-the-censors-workflow)
- [The Censorship's Scaling Problem](#the-censorships-scaling-problem)
- [On ISP-provided DNS and National Domain Name Systems](#on-isp-provided-dns-and-national-domain-name-systems)
- [On the Collection of VPN IP Addresses](#on-the-collection-of-vpn-ip-addresses)
- [On Session Duration and Port Rotation](#on-session-duration-and-port-rotation)
- [Countering Active Probing](#countering-active-probing-strategies-for-server-hiding)
- [Why Criminal Prosecution is a Sign of Technical Impotence](#why-criminal-prosecution-is-a-sign-of-technical-impotence)
- [The Failure of L3 Routing: The Split-Tunneling Trap](#the-failure-of-l3-routing-the-split-tunneling-trap)
- [Statistical Detection & Masking](#on-the-statistical-method-for-detecting-ip-addresses-of-user-defined-vpns)
- [The Nuclear Option: White-listing](#the-nuclear-option-white-listing-and-total-isolation)
- [The Final Statement](#the-final-statement)

---

## Implementation Reference
> [!NOTE]
> The theory is backed by a minimalist approach.  
> You can explore the practical implementation here: [simplest-vpn: 200-line skeleton](https://github.com/developer3389/simplest-vpn)

#### The User's Asymmetric Advantage
When users control the code, they own the bypass. They can tweak their own handshakes, inject custom jitter, or even program specific traffic patterns on the fly. Unlike commercial VPNs that have to play by network rules, individual users have zero constraints.

They can pull off the wildest stunts—like tunneling traffic inside Word docs or Excel sheets. This gives every user full control over their own traffic fingerprint. It’s safe to assume that at least 30% to 50% of the tech-savvy population will shift to these custom solutions.

#### What about non-technical users?
For those who can’t build their own tools, the solution is built on personal trust. Programmers can securely share compiled binaries directly with trusted acquaintances. By baking unique device-specific criteria—like UUIDs, Android versions, or time zones—directly into the binary, devs ensure the app only runs on the intended hardware. If a binary is leaked or handed over to a third party, it simply fails to run. Additionally, the programmer should pre-configure the binary compilation using specific compiler flags that obfuscate and minimize the binary, making reverse engineering extremely difficult.

#### Breaking the Censor’s Workflow
Censors are used to a simple game: identify the most popular VPNs, fingerprint them, and block them. But once the "user-defined VPN" wave hits, that game is over. They’ll be dealing with thousands of VPNs, each with a footprint of just a few users.

Detecting thousands of dynamically changing VPNs is a nightmare. As the rules bloat to millions of lines, the probability of widespread collateral damage to legitimate services becomes a certainty. This forces censors to implement 'hacky' workarounds, turning their rule database into a massive, unmaintainable mess that systemically cripples legitimate business operations.

#### The Censorship's Scaling Problem
Censors can’t just "hire more people" to fix this. Human cognitive capacity is a hard limit; you can't just throw more bodies at a million lines of code. If they try to automate this with AI, they’ll get buried in false positives. Businesses will start screaming when their legitimate traffic gets blocked, and AI models—trained on historical data—won't stand a chance against the creative, non-standard strategies of individual users.

In the end, this user-defined VPN shift will paralyze their censorship machine. It renders their multi-million dollar DPI gear utterly incapable of performing the core function of protecting the country's digital borders, as officially declared by the system designers. This effectively turns the DPI into a pile of rusted scrap metal, useless for any kind of targeted restriction.

##### On ISP-provided DNS and National Domain Name Systems
In some countries, the government strives to implement a national domain name system. Furthermore, by law, ISPs are mandated to use the national DNS as the source for their provider-side DNS. Traditionally, it is officially claimed that this system will provide digital sovereignty and protect digital borders.

Even if one accepts the statements of officials as truth, this system still grants providers the capability to block websites. In some countries, there is a practice of removing unwanted domains from the national DNS to reduce the load on DPI; as a result, for users whose phones are configured to use the provider's DNS, those devices do not make any requests, simply returning an `NXDOMAIN` error.

Users can bypass this restriction by configuring custom DNS from global providers such as Google or Cloudflare, as well as by enabling `DoH` or `DoT` to hide exactly which sites the user is visiting from the ISP.

#### On the Collection of VPN IP Addresses
Yes, in some countries, censors can collect VPN IP addresses by performing curl requests to various IP-lookup services.  
Standard practices currently used in those regions do not provide users with adequate protection against this.

However, there is a robust, fail-safe method:
`[Client in censorship country] -> [VPS1 (Abroad)] -> [VPS2 (Abroad)]`  

The core concept of this method is that smartphone applications will only detect the IP address of VPS2. Even if the censor identifies and blocks VPS2, the connection to the foreign network is maintained via the IP address of VPS1. The censor cannot see or block VPS1, provided that the VPN gateway resides on a device or virtual machine that is dedicated exclusively to this task and used for nothing else.

> [!TIP]
You can read [**our guide**](https://github.com/developer3389/vpn-gateway) how to setup vpn-gateway for entire home network.

#### On Session Duration and Port Rotation
Regardless of the protocol, a connection remaining active for 24 hours or more is likely to be flagged as a VPN, leading to server IP blocking. To avoid this, VPN sessions **must be rotated every few minutes or hours**. The exact interval depends on the type of traffic the VPN is masquerading as: mimicking typical web browsing requires frequent, short-lived sessions, while mimicking processes like database replication allows for slightly longer, but still bounded, session durations.

Implementation Strategy:

- Proactive Rotation: To stay within a "natural" range for the chosen traffic type, the client must generate a new source port and establish a new session before terminating the old one.

- Deterministic Server Ports: To maximize stealth, both the client and the server utilize a shared algorithm to predict the next server port at any given time, avoiding explicit command signaling inside the tunnel.

- Seamless Handover: To ensure zero disruption, the client must maintain the old session until the new connection is fully established and verified. Only then should the old connection be terminated.

#### Countering Active Probing: Strategies for Server Hiding
The censor can send requests to the users' servers to check how they will behave. If the server is silent (though this is not always the worst strategy), answers like a VPN, or looks like a VPN, the censor will block the IP address of such server. In order to combat active probing, users can implement responses to censor requests in a user-defined VPN.

This could be an HTTP response, text, an image, a video stream, or a stream of structured data files such as database dumps or binary blobs. The VPN tunnel disguises its traffic as routine business activity, such as database synchronization or cloud storage. Instead of spoofing a specific service via `SNI`, it mimics the general behavior of standard enterprise protocols.

> [!WARNING]  
> Users should avoid using hardcoded SNI strings like `googledrive.com` on a cheap VPS.  
> This is an obvious spoofing attempt that immediately reveals the proxy nature of the connection, as Google does not host its services on $1 VPS instances.

> [!WARNING]
> **Warning to Censors:**
> Users do not need perfect protocol imitation. The sheer scale of such obfuscation will force the system to flag legitimate connections as hidden VPN tunnels. Eventually, the censor will begin blocking actual business traffic, effectively paralyzing the nation’s infrastructure.

#### Why Criminal Prosecution is a Sign of Technical Impotence
Introducing criminal liability for VPN usage is not a display of power, but an admission of technical failure. If DPI (Deep Packet Inspection) systems were capable of effectively and accurately identifying VPN traffic, the state would block it automatically without resorting to social intimidation.

Technically, it is impossible to prove the use of a VPN if the protocol itself cannot be detected. Let us consider the examples.

Traditional VPNs are easily detectable targets due to their predictable symmetry: a single socket, a fixed pair of IP addresses, and a mirrored data stream:

```bash
[VPN CLIENT IP1] ------------------------> [VPN SERVER IP2]
                         (Request)

[VPN CLIENT IP1] <------------------------ [VPN SERVER IP2]
                         (Response)
```

To bypass DPI, users can reconfigure their VPN into a "triangular" or "rectangular" topology. This separates data streams across independent sockets and IP addresses:

##### Triangular Topology:

```text
               (Socket 1)
[CLIENT IP 1] >>>>>>>>>>> [ENTRY VPS IP 2]
                                  v
								  v (Socket 2)
                                  v
                                  v
[CLIENT IP 1] <<<<<<<<<<< [LAST VPS IP 3]
               (Socket 3)
```
* *Sockets `(IP 1, IP 2)`, `(IP 2, IP 3) and (IP 3, IP 1)` are independent.*

> [!NOTE]
> External internet access can be provided by either `IP 2` or `IP 3`

> [!WARNING]
> **Traffic Correlation**: While separating transit paths increases DPI complexity, it doesn't guarantee anonymity.
> Because traffic originates from the same client IP, a sophisticated censor **can still correlate** requests and responses via statistical analysis (packet timing and size matching).

##### Rectangular Topology:
```text
                      (Socket 1)     
[VPN CLIENT IP 1] >>>>>>>>>>>>>>>>>>> [ENTRY VPS IP 2]
       ^                                       v
       ^                                       v 
       ^ (Socket 4)                            v (Socket 2)
       ^                                       v
	   ^                                       v
[ LAST VPS IP 4 ] <<<<<<<<<<<<<<<<<<< [TRANSIT VPS IP 3]
                      (Socket 3)      			\
											     \
									  external internet IP 3
```

* *Sockets `(IP 1, IP 2)`, `(IP 2, IP 3)`, `(IP 3, IP 4)` and `(IP 4, IP 1)` are independent.*

> [!NOTE]
> Internet access is handled **exclusively** via `IP 3`. Because this node acts as the sole gateway to the external internet in the eyes of the censor, it serves as a sacrificial point for potential IP-based blocks. While `IP 3` may eventually be blocked, this architecture ensures that `IP 2` and `IP 4` remain hidden and immune to targeted bans, maintaining the integrity of core infrastructure.

> [!NOTE]
> Blocking `IP 3` is futile. Since `IP 2` and `IP 4` are **hosted abroad**, they bypass any local IP restrictions, ensuring that the tunnel between infrastructure nodes remains functional regardless of the censor's actions.

> [!WARNING]
> To eliminate stream correlation, receive and transmit sockets must operate with distinct rhythms, amplitudes, and intensities.

> [!TIP]
> Also, note that in these schemes, the connection can be initiated not only by the client within the country of censorship but also by external nodes (`ENTRY VPS`, `LAST VPS`), directing traffic inward.  
> In this scenario, the `VPN CLIENT` acts as a "passive receiver," making no suspicious outgoing requests.

> [!CAUTION]
> Regardless of network topology complexity, the user's VPN **remains vulnerable at the L7 (Application) layer.**  
> If the user utilizes a user-defined VPN but accesses national services directly via the IP addresses of their connection nodes, censors can easily identify the traffic as VPN-originated.  
> Obfuscation at the network layer (L3) does not negate the need for caution at the application layer.

Thus, by combining **the Rectangular Topology** with **custom user-defined protocols**, and maintaining **proper user discipline—including the strict avoidance of national services**—criminal prosecution for VPN usage becomes highly unlikely, as the traffic becomes indistinguishable from legitimate background activity.

#### The Failure of L3 Routing: The Split-Tunneling Trap
Current split-tunneling methods, which route traffic based on network-level IP addresses or domain zones, are fundamentally flawed.

The modern internet is global, and websites frequently load content from dozens of servers worldwide. When poorly configured tunnel settings force a site’s traffic to split between an encrypted tunnel and a direct connection at the network level (L3), it causes "split-brain" errors: the server sees the same user arriving from multiple locations simultaneously, leading to session errors and blocked access.

**We propose a new standard for browser architecture:**

The L7 Solution: Contextual Browser Routing
The core problem with current tools is the friction of constant VPN toggling. To restore a seamless experience, we must shift traffic management to the application layer (L7) by binding browser tab contexts to specific proxy-based tunnels.

##### Practical Implementation: Hierarchical Proxy Mapping
**Example Configuration:**

| Tab | Address Bar Domain | Rule (Mask) | Proxy Instance |
| :--- | :--- | :--- | :--- |
| 1, 2, 3 | `music.youtube.com` | `*.youtube.com` | `127.0.0.1:8080` (Proxy A) |
| 4 | `speedtest.net` | `*.net` | `127.0.0.1:8081` (Proxy B) |
| 5 | `mail.google.com` | `*.google.com` | `127.0.0.1:8082` (Proxy C) |
| 6 | `gemini.google.com` | `gemini.google.com` | `127.0.0.1:8083` (Proxy D) |
| 7 | `any-site.com` | `*` | `127.0.0.1:8084` (Proxy E) |


> [!NOTE]
> **Priority Rule:** A more specific domain mask always overrides a general one.  
> For instance, if a user visits `gemini.google.com`, the browser will prioritize the explicit `gemini.google.com` rule (Proxy D) over the broader `*.google.com` rule (Proxy C).

**This is our core vision**: users define traffic routing through domain masks, applying a hierarchical logic where precision dictates the path. By mapping specific masks to proxy instances, we transform browser architecture from a "global-on/off" model to a granular, context-aware routing system:

- Tab-to-Proxy Binding by Domain Mask: Users define domain masks (e.g., `*.youtube.com`) and assign them to specific proxy instances. The browser then binds the entire context of a tab to the appropriate proxy based on the domain in the address bar.

- Full Context Encapsulation: Once a tab is bound to a proxy via its domain, every request initiated within that tab—including all background scripts, API calls, and third-party trackers—is forced through that specific tunnel.

- Seamless Navigation: This approach eliminates the need for manual VPN toggling. Users simply open a tab, and the browser automatically routes the session based on the domain in the address bar. The connection state is maintained automatically, ensuring a smooth, uninterrupted browsing experience.

- Isolation: Because the tunnel is bound to the tab’s context, site data never leaks outside the designated path, ensuring a consistent digital presence without the errors caused by fragmented routing.

As internet censorship spreads, managing traffic with this level of surgical precision is the only way to keep the web accessible and stable, while reclaiming the comfort of a truly global internet.

#### On the statistical method for detecting IP addresses of user-defined VPNs
Yes, in certain countries, censors will likely shift to a policy of statistical analysis upon the widespread adoption of "user-defined VPN" methods. They will collect the frequency of requests to various IP addresses originating from a single sender's IP. Since the VPN IP address would be the most frequent in the statistics, that IP will be subject to blocking.

<details>
<summary>Click to expand: Statistical Masking and Safety Guidelines</summary>
   
###
Citizens can circumvent this by configuring the same VPN software to generate masking traffic: while the tunnel transmits real data to the designated server, the software simultaneously sends generated noise `from the same IP:PORT socket` to legitimate foreign services **outside the VPN tunnel**. The censor must see the data volume to these foreign services consistently higher than that of the VPN's IP address. For example, if YouTube is viewed at a speed of `10 Mbps via VPN`, the masking traffic to auxiliary IP addresses (generated by the same process) must also be of at least that same speed and volume—ideally `11 Mbps or more`. Consequently, in this example, the total channel utilization will be approximately `21 Mbps` on a standard `100 Mbps` connection, making it difficult for a censor to identify the true user-defined VPN IP address among the entire list of addresses.

#### CAUTION 1: Ethical Justification and Legitimacy
This statistical masking method should be considered exclusively as a last resort.  
Users have no ethical justification for utilizing it unless censors have abandoned DPI-based filtering and have actively begun employing frequency-based statistical analysis or strict "whitelist" access policies to identify and block VPN connections.  
This technique is designed strictly as a defensive response to extreme, infrastructure-level censorship, not as a tool for indiscriminate network traffic manipulation.  

#### CAUTION 2: Safety Calculation and Load Balancing
Users are strongly advised to avoid directing masking traffic to a single foreign IP address.  
Let's perform a simple calculation:

Suppose `one million` users in a country are simultaneously sending masking traffic at a rate of `11 Mbps` each.  
Assume these users decide to use a Google service to hide from the censor.  
If every user utilizes the exact same single foreign Google IP address,  
Google would receive a total load of `11,000,000 Mbps = 11 Tbps.`
   
We can see that such traffic concentration would result in a national-scale DDoS attack against a specific foreign service.
 
Now, suppose each of those `one million` users chooses `1,000` services and evenly distributes their masking traffic among them.  
Although the individual masking traffic remains `11 Mbps` per user (which exceeds the volume of the actual VPN traffic),  
each service receives only `11 Mbps / 1,000 = 11 Kbps` from a single user.  
Consequently, if all users happened to choose the exact same list,  
each service would receive
`11 Kbps * million users = 11,000,000 Kbps = 11 Gbps` total for the entire country.  
Furthermore, if users choose different lists of services, the load on the global internet becomes orders of magnitude lower.  

As we can see, the second approach allows users to hide from the censor while avoiding a national-scale DDoS attack against any individual foreign service.  
Failing to adhere to this requirement would allow users to bypass local censorship, but at the cost of collapsing the global internet.

#### CAUTION 3: Masking Speed Thresholds
Please note that `1,000` services is a safe threshold only for a masking speed of `10 Mbps`. If the user sets a higher masking speed, a proportionally larger pool of services is required. The user should refer to the safety calculation section above to determine the appropriate pool size for their specific configuration.

#### CAUTION 4: The Global Scaling Trap
Even if citizens in every country employing censorship strictly adhere to load distribution rules, the aggregate traffic on a global scale will inevitably result in a DDoS-like effect. The load risks reaching `terabits per second`. This reality proves that forcing citizens to employ extreme circumvention methods transforms local censorship into a threat against the stability of the entire global network, rather than an internal matter of individual states. Ultimately, the only effective solution remains diplomatic pressure that increases the cost of maintaining such censorship for a government to a level exceeding the political damage of allowing citizens free access to information. 

Can individual users be blamed for this situation? Hardly, as each creates a load of only `11 Kbps` per service, which falls well within standard internet usage patterns.

</details>

#### The Nuclear Option: White-listing and Total Isolation
A censor may attempt to cut off the global internet and transition the country to a "white-list" system, leaving only a state-run intranet. However, for patriots who sincerely care about the well-being of the country's constitutional order, it is crucial to advise their leader against implementing such measures: the economic collapse and social backlash triggered by isolation will only serve as the fastest path to destabilization. Furthermore, those promoting the concept of white-listing are, from the perspective of economists, inadvertently undermining the constitutional order and the nation's stability.

#### What if Whitelists are Enforced?
If a regime reaches the point of implementing a nationwide whitelist, citizens should simply grab some popcorn. When a country becomes so intellectually and technologically bankrupt that it chooses complete digital isolation, it has already sealed its own fate. The ensuing systemic collapse will be entirely self-inflicted, driven by the regime’s own incompetence.

#### The Final Statement
It is logical to conclude that when the cost of bypassing a nation’s digital defenses is reduced to a mere 200 lines of Go code, and users have the ability to transform their thoughts into code via AI, the verdict is clear: the current censorship infrastructure is obsolete.  
To the censors: your efforts are futile. The only recommendation that will actually assist you in preventing further damage to the national infrastructure is to resign from this unsustainable and unproductive line of work. A thriving and advanced digital economy requires one essential condition: unrestricted information exchange with the entire world.

## Let's Reclaim Internet Freedom!
