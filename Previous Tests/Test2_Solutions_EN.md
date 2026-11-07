# CS 6210 — Test 2 Complete Questions & Solutions (English Edition)

This document contains the complete English question prompts and exam-ready solutions for **[Fall 2025 Test 2](file:///D:/OMSCS/AOS/CS6210-Advanced-Operating-Systems-Notes/Previous%20Tests/Fall%2025/Test%202.pdf)** and **[Spring 2026 Test 2](file:///D:/OMSCS/AOS/CS6210-Advanced-Operating-Systems-Notes/Previous%20Tests/Spring%2026/Test%202.pdf)**.

Every answer strictly follows the grading-friendly format:
```text
[Yes / No / True / False / Direct Answer] — because [primary justification x]
because [detailed mechanism x]
unless [boundary condition / exception y]
also [additional consequence / supporting fact z]
```

---

## PART 1: FALL 2025 — TEST 2 (80 POINTS TOTAL)

### 1. Distributed Systems — Lamport's Logical Clock [8 pts]

#### Question 1 [3 pts]
In a multiplayer online shooter game, players move and shoot in real time. Each action `<playerID, action>` (for example, moving to a spot or firing a weapon) and any counteraction (such as dodging or returning fire) is broadcast to all other players in a peer-to-peer system. There is no message loss in the network; the network is reliable with variable latency, and messages between any two players arrive in FIFO order.

If we use Lamport's logical clocks to timestamp all actions, will this ensure that a counteraction to a move always appears after the original move in the exact order they were generated for every player? Justify your answer.

**Answer:**
```text
No — because Lamport's logical clocks only preserve causal happens-before ordering (A -> B implies C(A) < C(B)) rather than real-time total generation order or immediate network delivery order across all third-party nodes.
because if Player A broadcasts an original move and Player B receives it and immediately broadcasts a counteraction, variable network latency along the direct path A -> C versus the indirect path A -> B -> C can cause Player C to receive Player B's counteraction before Player A's original move arrives.
unless the game buffers incoming messages and runs a total-order broadcast / causal delivery barrier or uses a central server before rendering actions, out-of-order network arrival can still display the counteraction first.
also because C(X) < C(Y) does not imply X happened before Y for concurrent moves generated independently by different players.
```

---

#### Question 2 [5 pts]
Consider a distributed system with three processes (`A`, `B`, and `C`) shown in the following space-time diagram ( based on Lamport, 1978):
* **Process A**: `a1 -> a2 -> a3 -> a4 -> a5 -> a6`
* **Process B**: `b1 -> b2 -> b3 -> b4`
* **Process C**: `c1 -> c2 -> c3 -> c4`
* **Messages**: `a1 -> b2`, `b1 -> c1`, `c3 -> b2`, `b2 -> a3`, `b3 -> a4`, `b3 -> c2`, `a5 -> b4`.

* **2a [1 pt]**: Did the event `b3` happen before `a6`? Justify your answer.
* **2b [1 pt]**: Did the event `c3` happen before `a4`? Justify your answer.
* **2c [1 pt]**: Did the event `a2` happen before `c1`? Justify your answer.
* **2d [2 pts]**: How can the partial ordering of events in this system be extended to a total ordering?

**Answers:**

* **2a [1 pt]**:
```text
Yes — because there is a directed causal happens-before path from b3 to a6 (b3 -> a4 -> a5 -> a6).
because b3 is the message send event received at a4 (b3 -> a4), and a4 precedes a5 and a6 sequentially within Process A (a4 -> a5 -> a6).
also by transitivity of Lamport's happens-before relation, b3 -> a6.
```

* **2b [1 pt]**:
```text
Yes — because there is a directed causal happens-before path from c3 to a4 (c3 -> b2 -> a3 -> a4).
because c3 sends a message received at b2 (c3 -> b2), b2 sends a message received at a3 (b2 -> a3), and a3 precedes a4 sequentially within Process A (a3 -> a4).
also by transitivity of Lamport's happens-before relation, c3 -> a4.
```

* **2c [1 pt]**:
```text
No — because there is no causal path of sequential events and messages connecting a2 to c1 in either direction (a2 and c1 are concurrent events).
because c1 only causally follows b1 (b1 -> c1), and no message sent at or after a2 ever reaches Process B at or before b1 or Process C at or before c1.
unless a causal chain links two events, Lamport's happens-before relation does not order them.
```

* **2d [2 pts]**:
```text
By breaking logical clock ties deterministically using a well-known arbitrary total order over Process IDs — because concurrent events on different processes can have the exact same logical timestamp.
because if two events X (on Process i) and Y (on Process j) have equal logical timestamps C(X) == C(Y), ordering them by unique process identifiers (e.g., Process A < Process B < Process C) resolves all ties consistently.
also this deterministic tie-breaking requires no extra inter-node communication.
```

---

### 2. Lamport's Mutual Exclusion (ME) Algorithm [6 pts]

#### Question 1 [6 pts]
Assume a 3-node (`P1`, `P2`, `P3`) distributed system that uses Lamport's Mutual Exclusion (ME) algorithm to arbitrate access to a shared resource. Assume all assumptions of Lamport's ME algorithm hold true.
* `P1` requests access at logical timestamp `4` (`(P1, 4)`), and `P2` requests access at logical timestamp `3` (`(P2, 3)`).
* Timeline of events (`T1` to `T7`):
  * `T1`: `P1` receives `(P2, 3)` from `P2`.
  * `T2`: `P2` receives `(P1, 4)` from `P1`.
  * `T3`: `P3` receives `(P2, 3)` from `P2` and sends `ACK` to `P2`.
  * `T4`: `P2` receives `ACK` from `P3`.
  * `T5`: `P3` receives `(P1, 4)` from `P1` and sends `ACK` to `P1`.
  * `T6`: `P1` receives `ACK` from `P3`.
  * `T7`: `P2` sends `Release` to `P1` and `P3`.

* **1a [2 pts]**: Which node gets to access the shared resource first, and at which time (`T1–T7`)? Justify your answer.
* **1b [2 pts]**: What is a potential problem if the "eventual message delivery" assumption is violated? Explain your answer.
* **1c [2 pts]**: What is a simple method to prevent the algorithm from failing due to message loss?

**Answers:**

* **1a [2 pts]**:
```text
P2 at time T4 — because P2's request has the smallest logical timestamp (3 < 4) and satisfies both of Lamport's mutual exclusion conditions at T4.
because Condition 1 holds since (P2, 3) is at the head of P2's local request queue, and Condition 2 is satisfied at T4 once P2 has received a message with timestamp > 3 from every other node (it received P1's request with TS=4 > 3 at T2, and received P3's ACK at T4).
unless P2 has heard from both P1 and P3, it cannot safely enter the critical section prior to T4.
```

* **1b [2 pts]**:
```text
Mutual exclusion violation (safety failure) or Deadlock/Starvation (liveness failure) — because message loss causes nodes to have incomplete or permanently stalled request queues.
because if a request message from P1 to P2 is lost, P2 will not know P1 wants the lock and may enter the critical section concurrently with P1; conversely, if an ACK or release message is lost, waiting nodes will block forever waiting for a message that never arrives.
also a lost release message leaves the released request permanently stuck at the head of peers' local queues.
```

* **1c [2 pts]**:
```text
Use a timeout-and-retransmission protocol with acknowledgments (ACKs) and sequence numbers — because reliable transport detects and recovers lost packets automatically.
because nodes can acknowledge every message and retransmit if no ACK arrives before a timeout expires, while sequence numbers allow receivers to detect missing messages and discard duplicate retransmissions.
```

---

### 3. RPC Latency Limits [6 pts]

#### Question 1 [6 pts]
Your team is designing an RPC framework for a high-throughput storage service deployed over 100 Gbps Ethernet. During testing, you observe occasional packet corruption even when link-layer CRCs pass. You are asked to evaluate two reliability approaches:
* **Hardware-only**: Rely solely on NIC checksum offload, with no software checksums in the RPC layer.
* **Hardware Software Combined**: Use NIC checksum offload and a lightweight application-level RPC checksum.

* **1a [2 pts]**: Suppose the network is upgraded to 400 Gbps Ethernet in a LAN. Which approach would you recommend, and why?
* **1b [2 pts]**: Suppose the system is deployed for cross-datacenter communication across the Wide Area Internet. Which approach would you recommend, and why?
* **1c [2 pts]**: A teammate argues: *"Modern NICs and TCP already guarantee reliability. Adding another checksum layer in RPC just increases latency."* Would you agree or disagree with this statement? Justify your position.

**Answers:**

* **1a [2 pts]**:
```text
Hardware-only checksums — because a Local Area Network (LAN) is highly reliable with very low bit-error rates, so we should leverage hardware offload and avoid software checksum latency.
because computing an extra software checksum in the RPC layer at 400 Gbps adds significant CPU and critical-path latency overhead without meaningful reliability benefit in a controlled LAN.
```

* **1b [2 pts]**:
```text
Hardware Software Combined — because Wide Area Network (WAN) communication traverses numerous intermediate routers and links where the probability of bit errors and in-router corruption is much higher.
because hardware link-layer CRCs only validate single hops and cannot detect corruption occurring inside intermediate WAN router memories or buses, making an end-to-end software RPC checksum necessary.
```

* **1c [2 pts]**:
```text
Agree — because the TCP protocol already provides end-to-end reliability, making an additional checksum in the RPC layer redundant.
because TCP inherently performs end-to-end checksumming, sequence numbering, acknowledgments, and automatic retransmissions to guarantee corruption-free, in-order delivery.
also adding another checksum at the RPC layer duplicates TCP's work and only increases end-to-end RPC latency.
```

---

### 4. Active Networks [4 pts]

#### Question 1 [4 pts]
In Active Networks, nodes obtain code from the previous (`prev`) node and store it in the soft store. Describe two potential security risks with this design and explain how ANTS mitigates them.

**Answer:**
```text
Code Spoofing and Soft-Store Corruption/Resource Interference — because dynamically downloading executable code and sharing router memory across untrusted flows exposes active routers to counterfeit code and state tampering.
because:
1. Code Spoofing Risk: A malicious node could supply fake or malicious code for a legitimate protocol TYPE; ANTS mitigates this by setting the capsule's TYPE field to a cryptographically strong fingerprint (e.g., 128-bit MD5 hash) of the actual code and verifying that hash upon receipt before caching or executing the code.
2. Soft-Store Integrity & Exhaustion Risk: Malicious or buggy capsule code could corrupt other flows' cached code/breadcrumbs or exhaust router memory; ANTS mitigates this via Java sandboxing and a restricted runtime API that isolates flows and bounds how soft state is accessed and retained.
```

---

### 5. Distributed Objects and Middleware [16 pts]

#### Spring OS — Question 1 [3 pts]
Two development teams are building services on the Spring Operating System: one team implements its service in C, and the other in C++. What artifacts must each team produce, and how are they generated, so that their services can invoke each other's methods?

**Answer:**
```text
IDL interface definitions, language-specific header files, client-side stubs, and server-side stubs — because Spring OS uses its Interface Definition Language (IDL) compiler to bridge different programming languages.
because each team first writes its service interfaces in Spring's language-independent IDL, and then compiles those IDL files with the IDL compiler to generate: (1) language-specific C or C++ header files, (2) client-side stub code for marshaling calls, and (3) server-side stub code for unmarshaling and dispatching calls.
```

---

#### Spring OS — Question 2 [8 pts]
Consider a distributed environment running the Spring Operating System. A client application on **Node A** needs to invoke `invoice.submit()` on an Invoice Service running on **Node B** over a local area network.
* **2a [2 pts]**: What is a `door`, and what is a `network proxy` in the Spring OS?
* **2b [3 pts]**: How is this set up using `subcontracts`?
* **2c [3 pts]**: Once set up, what is the runtime flow of the invocation from the client to the invoice service?

**Answers:**

* **2a [2 pts]**:
```text
A door is a kernel-managed software capability for cross-domain object invocation on a single node, while a network proxy is a user-level domain that extends doors across network nodes — because Spring separates fast local microkernel IPC from network transport.
because a door represents a validated entry point into a target domain managed by the local Nucleus (microkernel), whereas network proxies connect the nuclei of different machines and transparently forward door invocations over the network.
```

* **2b [3 pts]**:
```text
Subcontracts instantiate Network Proxies and bind local doors to remote network handles on both nodes — because subcontracts encapsulate the underlying cross-machine communication setup beneath the stubs.
because the subcontract sets up: (1) Network Proxy A on Node A and Network Proxy B on Node B using a mutually compatible network protocol, (2) a door entry point in Network Proxy B on Node B (bound to the Invoice Service) that is exported as a network handle to Network Proxy A, and (3) a local door entry point in Network Proxy A on Node A for the client's Nucleus to invoke.
```

* **2c [3 pts]**:
```text
Client (Node A) -> Nucleus A -> Network Proxy A -> Network Proxy B -> Nucleus B -> Invoice Service (Node B) — because cross-node invocation chains local Nucleus door calls at both endpoints with proxy-to-proxy network forwarding.
because:
1. The Client on Node A invokes its local door handle for invoice.submit(), causing Nucleus A to transfer control to the door entry point in Network Proxy A.
2. Network Proxy A marshals the request and sends it across the LAN via the network handle to Network Proxy B on Node B.
3. Network Proxy B receives the packet and performs a local door call through Nucleus B to the door entry point of the Invoice Service domain on Node B.
```

---

#### EJB — Question 1 [2 pts]
You are reviewing a proposed EJB architecture for a distributed enterprise application that will soon go into production. The design currently exposes fine-grained Entity Beans directly to clients over a public WAN. Before approval, you recommend introducing a **Session Facade** (i.e., Session Bean with Entity Bean) pattern. Why would you make this recommendation?

**Answer:**
```text
To protect internal business logic (security) and reduce WAN round-trip overhead while maintaining parallel database access (performance) — because exposing fine-grained Entity Beans directly over an untrusted public WAN is both insecure and slow.
because (1) a Session Facade encapsulates and hides sensitive business logic inside the internal corporate EJB container rather than exposing it over the public WAN, and (2) it aggregates fine-grained operations into coarse-grained WAN calls while still issuing concurrent, parallel requests to underlying Entity Beans locally.
```

---

#### Java RMI — Question 1 [3 pts]
Briefly explain the three layers of Java RMI (`Stub/Skeleton Layer`, `Remote Reference Layer`, and `Transport Layer`).

**Answer:**
```text
Stub/Skeleton handles marshaling, Remote Reference Layer handles invocation semantics, and Transport Layer handles network connections — because Java RMI separates data serialization, call semantics, and transport management into three distinct layers.
because:
1. Stub/Skeleton Layer: Acts as the interface between the application and the RMI system, initiating remote calls and marshaling/unmarshaling (serializing/deserializing) arguments and return values over streams provided by the RRL.
2. Remote Reference Layer (RRL): Implements invocation semantics (e.g., unicast/singleton vs. multicast/replicated servers and reconnection policies) and manages marshaling streams between the stub/skeleton and transport layer.
3. Transport Layer: Handles endpoint addressing, connection setup/teardown/management, channel selection (TCP/UDP), connection liveness monitoring, and tracking/dispatching to remote objects in the local address space.
```

---

### 6. Distributed Subsystems [24 pts]

#### GMS — Question 1 [5 pts]
Suppose there is a 10-node cluster (nodes `A–J`) running the Global Memory System (GMS). A process `x` on **Node A** read-shares a page that is actively used only by a process `y` on **Node B**. Assume there are no co-location constraints on any data structures.
* **1A**: Which node's `POD` is accessed?
* **1B**: Which node's `GCD` is accessed?
* **1C**: Which node's `PFD` is accessed?
* **1D**: What is the minimum number of network hops required in the worst case?
* **1E**: Now suppose a new process `z` on **Node A** also read-shares the same page. Which node's `PFD` is accessed this time?

**Answers:**

* **1A (POD)**:
```text
Node A — because the Page Ownership Directory (POD) is replicated on every node in the cluster and is always accessed locally first.
```

* **1B (GCD)**:
```text
The node that owns the GCD hash partition for the page's UID (any of the 10 nodes A-J identified by POD) — because the Global Cache Directory (GCD) is hash-partitioned across the cluster by UID.
```

* **1C (PFD)**:
```text
Node B — because Node B is currently hosting the target page in its physical DRAM and maintains the Page Frame Directory (PFD) entry mapping the UID to its physical frame.
```

* **1D (Minimum network hops in the worst case)**:
```text
3 network hops (Node A -> GCD Owner Node -> Node B -> Node A) — because in the worst case with no co-location, the GCD owner is a third node distinct from both Node A and Node B.
because Node A sends the request to the remote GCD partition owner (hop 1), the GCD owner forwards the request to Node B holding the PFD and page (hop 2), and Node B sends the shared page copy directly back to Node A (hop 3).
```

* **1E (Which PFD is accessed when process `z` on Node A read-shares the same page)**:
```text
Node A — because after process x's read fault finishes, the shared page is already cached locally in Node A's DRAM (in its Unified Buffer Cache / local cache), so Node A's local PFD satisfies the hit with 0 network hops.
```

---

#### GMS — Question 2 [5 pts]
Consider a 4-node cluster (`A`, `B`, `C`, `D`) running GMS. **Node A** needs to evict two pages from its local cache:
* **Page `P1`** with age `15 minutes`
* **Page `P2`** with age `28 minutes`

At the start of the current epoch, the initiator computed:
* `minAge = 25 minutes`
* `weights = {A: 0.2, B: 0.3, C: 0.1, D: 0.4}`

* **2a [2 pts]**: For each page (`P1` and `P2`), what decision will GMS make? Justify your answer.
* **2b [1 pt]**: If a page is sent to another node, which node is most likely to receive it, and why?
* **2c [2 pts]**: What additional work must be done when pages are evicted, and what is the justification for doing so?

**Answers:**

* **2a [2 pts]**:
```text
Send P1 to a peer node's global memory and discard P2 locally — because GMS compares each evicted page's age against minAge = 25 minutes to approximate global LRU.
because P1's age (15 < 25) is younger than minAge, meaning it is younger than the M oldest pages in the cluster and should be kept in global memory, whereas P2's age (28 >= 25) is older than minAge, meaning it is old enough to be discarded locally.
```

* **2b [1 pt]**:
```text
Node D — because Node D has the highest weight (0.4), meaning it holds the largest share of the M oldest idle pages in the cluster.
```

* **2c [2 pts]**:
```text
Write the page to disk first if it is dirty — because GMS maintains the strict invariant that all pages in any peer's global memory cache must be clean.
because flushing dirty pages to disk prior to global caching or discard guarantees that if a peer node crashes, no data is ever lost.
```

---

#### DSM — Question 1 [2 pts]
A programmer writes a multiprocess DSM application that has no data races, uses mutex locks properly, and runs on a **Sequential Consistency (SC)** memory model with page-level granularity. Yet the program shows poor performance. Explain why.

**Answer:**
```text
False sharing at page-level granularity and eager per-access coherence overhead under Sequential Consistency — because SC cannot distinguish synchronization accesses from ordinary data reads/writes.
because (1) under SC, every individual read or write to shared memory must globally complete cluster-wide coherence operations (invalidations/updates) immediately rather than deferring coherence to lock boundaries, and (2) page-level granularity causes false sharing (page ping-ponging across nodes) whenever different nodes access independent variables located on the same virtual page.
```

---

#### DSM — Question 2 [8 pts]
Consider a distributed shared memory system implemented using **TreadMarks**. Consider the following critical section on node `N1`:
```c
Lock (L1);
Write to Page X;
Unlock (L1);
```
Assume that:
* `X` is the only page modified by prior acquirers of `L1`.
* `X` is not present in `N1`.
* Diffs `Xd2`, `Xd3`, and `Xd4` exist on nodes `N2`, `N3`, and `N4`, respectively.
* The synchronization causality for `L1` up to this point is: `N3 -> N2 -> N1`.

* **2a [2 pts]**: What actions take place at the lock acquisition `Lock(L1)`, and why?
* **2b [2 pts]**: What additional actions take place at `N1` before the critical section completes?
* **2c [2 pts]**: What actions are taken upon exiting the critical section `Unlock(L1)` at `N1`?
* **2d [2 pts]**: Later, `N1` executes `Lock(L1); Read page X; Unlock(L1);` and there have been no intervening acquirers of `L1`. What actions are taken by the DSM at `N1`?

**Answers:**

* **2a [2 pts]**:
```text
Invalidate Page X at N1 (record piggybacked write notices from N3 and N2 for Page X) — because X is the only page modified by prior acquirers of lock L1 along L1's causal chain (N3 -> N2 -> N1).
because Lazy Release Consistency piggybacks invalidation notices for causally preceding modifications onto the lock grant message sent from N2 to N1.
```

* **2b [2 pts]**:
```text
Fetch the clean copy of X from its owner, fetch and apply diffs Xd3 then Xd2 in causal order, create a twin of X, and mark X read-write — because X is not present at N1 and is written inside the critical section.
because upon faulting on Write to Page X, N1 must: (1) fetch the pristine copy of X from X's owner node, (2) request and apply only the causally related diffs Xd3 (from N3) and Xd2 (from N2) in that exact causal order (ignoring Xd4 on N4 since N4 is not in L1's causal chain), and (3) create an identical twin of X in system space and set X's page protection to read-write.
```

* **2c [2 pts]**:
```text
Create diff Xd1, write-protect Page X (mark it read-only), and discard the twin — because TreadMarks summarizes modifications at release boundaries and resets page protection to trap future writes.
because N1 compares the modified Page X word-by-word against its twin to generate a run-length encoded diff Xd1, asks the OS to mark Page X read-only, and frees the twin from system memory.
```

* **2d [2 pts]**:
```text
No DSM action is taken — because no other node acquired L1 or modified Page X in the interim, and N1 already holds a valid, up-to-date copy of Page X mapped as read-only.
because Lock(L1) carries no new invalidations and reading Page X succeeds locally at hardware speed without triggering a page protection fault.
```

---

#### DFS — Question 1 [2 pts]
Describe the high-level algorithm for **xFS Metadata Manager Recovery** after a crash.

**Answer:**
```text
Consensus-based Manager Map regeneration, Server-Driven Polling for cache consistency state, and Checkpoint plus Roll-Forward recovery for disk metadata — because xFS distributes metadata across managers, client caches, and storage server logs.
because:
1. Manager Map Recovery: Active nodes run a distributed consensus algorithm to regenerate the Manager Map (mmap) and reassign the failed manager's index number space to a surviving manager.
2. Cache Consistency Metadata Recovery: The new manager uses server-driven polling to query all clients for which blocks they currently cache or hold write tokens for.
3. Disk Location Metadata Recovery (imap and index nodes): The new manager reads the latest manager checkpoint (ifile) from the storage server logs and rolls forward by applying log deltas written by clients after the checkpoint.
```

---

#### DFS — Question 2 [2 pts]
Compared to xFS's distributed metadata management, state one **Pro** and one **Con** of using a **single Master** for metadata as in the Google File System (GFS).

**Answer:**
```text
Pro: Design simplicity and direct client lookup; Con: Centralized scalability bottleneck and single point of failure — because concentrating metadata on one Master removes distributed coordination at the expense of scalability.
because:
- Pro: Every client knows the single Master to contact for metadata without maintaining or running consensus on a distributed Manager Map (mmap).
- Con: The single Master can become a performance bottleneck for metadata-heavy/small-file workloads and represents a single point of failure unless shadow/failover masters are maintained.
```

---

### 7. HIDDEN — Potpourri [16 pts]

#### Question 1 [2 pts]
**True/False**: For the correctness of a multiprocess application using DSM on a LAN cluster, a given virtual page should be mapped to the same physical frame number in each node. Explain your answer.

**Answer:**
```text
False — because DSM provides address equivalence only at the virtual page (virtual address) level, not at the physical frame level.
because each node's local operating system virtual memory manager independently maps a global virtual page to whatever physical DRAM frame is locally available on that node.
```

---

#### Question 2 [2 pts]
**True/False**: Multi-writer coherence protocol works with SC, RC, and LRC memory models. Explain your answer.

**Answer:**
```text
False — because multi-writer coherence protocols only work with Release Consistency (RC) and Lazy Release Consistency (LRC), not with Sequential Consistency (SC).
because multi-writer protocols defer coherence merging until synchronization release/acquire boundaries, which requires a memory model (like RC or LRC) that distinguishes synchronization accesses from ordinary data accesses (whereas SC requires every ordinary read/write to be globally ordered immediately).
```

---

#### Question 3 [2 pts]
**True/False**: In TreadMarks, upon a page fault for page `X` in Node `N1`, the DSM software on `N1` broadcasts the VPN of the faulting page to all the peer nodes. Explain your answer.

**Answer:**
```text
False — because TreadMarks uses point-to-point messages to known page owners and diff holders rather than broadcasting on a page fault.
because each virtual page has a statically assigned owner/manager node (via partitioning/hashing) and piggybacked invalidation notices explicitly record which nodes hold diffs, so N1 contacts only the specific owner/diff nodes directly.
```

---

#### Question 4a [2 pts]
**True/False**: In active networks, if the code requested by a node is not present in the `prev` active node, the request is recursively sent to all the previous active nodes until the code is obtained. Explain your answer.

**Answer:**
```text
False — because if the immediate PREV active node does not have the requested code in its soft store, the capsule is simply dropped rather than triggering recursive upstream requests.
because ANTS only queries the immediate PREV node to keep routers simple and stateless, relying on end-to-end transport retransmission from the source if PREV evicted the code.
```

---

#### Question 4b [2 pts]
**True/False**: Active Networks require all the intermediate routers between the source and the destination to be "active". Explain your answer.

**Answer:**
```text
False — because capsules are encapsulated inside standard IP packets that conventional passive routers forward normally.
because passive routers simply inspect the outer IP header and forward the packet using standard IP routing tables, while only active routers inspect and execute the ANTS header.
```

---

#### Question 5 [6 pts]
Assume two RPC designs:
* **Design A**: Shared-memory RPC on a Symmetric Multiprocessor (SMP) machine (similar to LRPC), where the client blocks waiting for the server response.
* **Design B**: Traditional RPC across a LAN between uniprocessor machines (as analyzed in the Thekkath & Levy paper).

* **5a [1 pt]**: What is the minimum number of context switches for Design A?
* **5b [1 pt]**: What are those context switches in Design A?
* **5c [1 pt]**: How can we reduce the number of context switches in Design A even further?
* **5d [1 pt]**: What is the minimum number of context switches for Design B (traditional blocking RPC without overlap optimizations)?
* **5e [2 pts]**: What are those context switches in Design B?

**Answers:**

* **5a [1 pt]**:
```text
2 context switches — because blocking shared-memory RPC switches once from the client thread to the server thread and once back from the server thread to the client thread.
```

* **5b [1 pt]**:
```text
(1) Client -> Server (C -> S) when the client blocks on the call, and (2) Server -> Client (S -> C) when the server completes the procedure and wakes the client.
```

* **5c [1 pt]**:
```text
Have the client thread spin (busy-wait) on a shared-memory flag instead of blocking — because spinning on an SMP core avoids invoking the OS scheduler, reducing context switches to 0 (if the server is already running on another core) or 1.
```

* **5d [1 pt]**:
```text
4 context switches (or 2 on the critical path if post-send context switches overlap with network transmission) — because both the client and server nodes switch away after sending/blocking and switch back upon packet arrival.
```

* **5e [2 pts]**:
```text
(1) C -> C1 on the client node (client sends request and blocks, OS switches to another process C1);
(2) S1 -> S on the server node (interrupt arrives, server OS switches from running process S1 to server process S);
(3) S -> S2 on the server node (server sends reply and blocks, server OS switches from S to process S2);
(4) C1 -> C on the client node (reply arrives, client OS switches from C1 back to the waiting client process C).
```

---
---

## PART 2: SPRING 2026 — TEST 2 (85 POINTS TOTAL)

### 1. Distributed Systems — Lamport's Logical Clock [7 pts]

#### Question 1 [7 pts]
Consider a replicated database system with three nodes (`DB-A`, `DB-B`, and `DB-C`) and the following space-time diagram:
* **`DB-A`**: `a1: client write W1` $\rightarrow$ `a2: send W1 to B` ($\rightarrow b1$) $\rightarrow$ `a3: send W1 to C` ($\rightarrow c1$) $\rightarrow$ `a4: recv ACK from B` ($\leftarrow b3$) $\rightarrow$ `a5: recv W2 from B` ($\leftarrow b5$) $\rightarrow$ `a6: apply W2`
* **`DB-B`**: `b1: recv W1 from A` ($\leftarrow a2$) $\rightarrow$ `b2: apply W1` $\rightarrow$ `b3: send ACK to A` ($\rightarrow a4$) $\rightarrow$ `b4: client write W2` $\rightarrow$ `b5: send W2 to A` ($\rightarrow a5$) $\rightarrow$ `b6: send W2 to C` ($\rightarrow c3$)
* **`DB-C`**: `c1: recv W1 from A` ($\leftarrow a3$) $\rightarrow$ `c2: apply W1` $\rightarrow$ `c3: recv W2 from B` ($\leftarrow b6$) $\rightarrow$ `c4: apply W2` $\rightarrow$ `c5: client read R2` $\rightarrow$ `c6: reply to client`

* **1a [2 pts]**: At event `c5`, which writes are guaranteed to already be reflected at `DB-C`: `W1`, `W2`, `both`, or `neither`? Justify your answer.
* **1b [2 pts]**: At event `a4`, can `DB-A` conclude that all three database nodes have already seen `W1`? Justify your answer.
* **1c [3 pts]**: Assume `W1` updates `x = 1` and `W2` updates `x = 2`. We say a read is *globally consistent* if, at the time it is served, all replicas (`DB-A`, `DB-B`, and `DB-C`) would return the same value of `x`. Now suppose we add a read event `Rc` on `DB-C` after `c3` but before `c4`. Will the read at `Rc` be globally consistent? Justify your answer.

**Answers:**

* **1a [2 pts]**:
```text
Both (W1 and W2) — because both c2 (apply W1) and c4 (apply W2) precede c5 along DB-C's sequential process order (c1 -> c2 -> c3 -> c4 -> c5).
because by sequential ordering within DB-C and transitivity of the happens-before relation, both W1 and W2 have already been applied locally at DB-C prior to c5.
```

* **1b [2 pts]**:
```text
No — because at event a4, DB-A only has causal confirmation from DB-B (via a2 -> b1 -> b2 -> b3 -> a4), whereas DB-C's receipt of W1 at c1 is concurrent with a4.
because there is no causal path from c1 back to a4 (a3 -> c1 is a one-way message over a variable-latency network with no ACK back to DB-A), so W1 could still be in transit to DB-C when a4 occurs.
unless DB-C sends an acknowledgment to DB-A that arrives before a4, DB-A cannot know whether DB-C has received W1.
```

* **1c [3 pts]**:
```text
No — because at Rc (between c3 and c4), DB-C returns x = 1 whereas DB-B already holds x = 2, so the replicas do not return the same value of x.
because at Rc, DB-C has applied W1 at c2 (x = 1) and received W2 at c3, but has not yet applied W2 until c4; meanwhile, c3 causally follows b6 (b4 -> b5 -> b6 -> c3), meaning DB-B has already executed b4 (client write W2) and holds x = 2 (while DB-A may hold either x = 1 or x = 2 since a5/a6 are concurrent with Rc).
unless DB-C applies W2 at c4 and all replicas synchronize before serving the read, Rc violates global consistency.
```

---

### 2. Lamport's Mutual Exclusion (ME) Algorithm [8 pts]

#### Question 1 [8 pts]
Consider a 4-node distributed system (`P1`, `P2`, `P3`, `P4`) running Lamport's Mutual Exclusion algorithm shown in the timeline diagram:
* `P1` requests the lock with `[(10, P1)]` and sends `rq_12`, `rq_13`, `rq_14`.
* `P4` requests the lock with `[(15, P4)]` and sends `rq_41`, `rq_42`, `rq_43`.
* Prior to global timestamp line `T`: `P1` receives `ak_21` (from `P2`) and `rq_41` (from `P4`, timestamp 15). However, `ak_41` (from `P4`) and `ak_31` (from `P3`) arrive at `P1` **after** line `T`.
* After line `T`: `P1` receives `ak_41` and `ak_31`, enters the critical section (`CS`), and then sends `rl_12`, `rl_13`, `rl_14`. Also `P2` sends `ak_24`, `P3` sends `ak_34`, and `P1` sends `ak_14` to `P4`.

* **1a [2 pts]**: Can `P1` acquire the lock at global timestamp `T`? If no, justify why based on the two conditions of the algorithm, and explain what could go wrong if `P1` acquired the lock at `T`.
* **1b [2 pts]**: (i) Suggest an improvement to reduce the number of messages below $3(N - 1)$ without violating the safety of the lock algorithm. (ii) Give **ONE** example of a message in the current timeline diagram that can be discarded, with justification.
* **1c [2 pts]**: A colleague suggests sending lock release (`rl`) messages **ONLY** to nodes that are currently awaiting to use the lock (i.e., nodes from which the lock holder has received an `rq` message). Does this improvement honor the safety and correctness of the algorithm? What messages could be discarded in our timeline under this proposal?
* **1d [2 pts]**: What would be the message complexity under the colleague's improvement for:
  * **Low contention** (only 1 node requests the lock at any time)
  * **High contention** (all $N$ nodes actively request locks)

**Answers:**

* **1a [2 pts]**:
```text
No — because at global timestamp T, P1 violates Lamport's second mutual exclusion condition since it has not yet received any message with timestamp > 10 from P3 (ak_31 arrives after T).
because although Condition 1 holds locally ((10, P1) is at the head of P1's local queue ahead of (15, P4)) and P1 has heard from P2 (ak_21) and P4 (rq_41 with TS=15 > 10), P1 has not yet heard from P3; if P1 entered the critical section at T, P3 could have an in-transit lock request with a smaller timestamp (e.g., TS < 10), allowing both P1 and P3 to enter the critical section simultaneously and violating mutual exclusion.
```

* **1b [2 pts]**:
```text
(i) Suppress explicit ACK messages when a node has already sent a higher-timestamped request (or defer ACKs and piggyback them onto Lock Release messages, reducing complexity toward 2(N-1)); (ii) Discard ak_41 (or ak_14) — because FIFO ordering and request timestamps make these explicit ACKs redundant.
because:
- Example 1 (Discard ak_41 from P4 to P1): P1 already received rq_41 with timestamp (15, P4) from P4 prior to time T. Because 15 > 10 and channels are FIFO, receiving rq_41 already proves to P1 that P4 has no request earlier than TS=10, making ak_41 redundant.
- Example 2 (Discard ak_14 from P1 to P4): Since P1's request (10, P1) has higher priority (10 < 15) than P4's request (15, P4), P1 can defer ak_14 and let its release message rl_14 serve as both the release of (10, P1) and the acknowledgment of (15, P4).
```

* **1c [2 pts]**:
```text
No, it violates correctness (causes deadlock/starvation for future requests), and in our timeline it would discard rl_12 and rl_13 — because only P4 sent an rq message, so P1 would only send rl_14 to P4.
because in Lamport's algorithm, every node (including P2 and P3) inserts (10, P1) into its local queue upon receiving rq_12 and rq_13 and only removes it upon receiving a release message; omitting rl_12 and rl_13 leaves (10, P1) permanently stuck at the head of P2's and P3's local queues, preventing P2 and P3 from ever acquiring the lock in the future.
unless non-requesting nodes do not maintain full request queues (which would fail if P2 or P3 requested the lock while P1 was still in the critical section), dropping rl_12 and rl_13 breaks the algorithm.
```

* **1d [2 pts]**:
```text
Low contention: 2(N - 1) messages; High contention: 3(N - 1) messages (or 2(N - 1) if combined with deferred ACKs) — because the number of release messages equals the number of waiting peers.
because:
- Low contention (only 1 node requests the lock, 0 other nodes waiting): (N - 1) request messages + (N - 1) ACK messages + 0 release messages = 2(N - 1) messages.
- High contention (all N nodes actively request the lock, so all N - 1 peers are waiting): (N - 1) request messages + (N - 1) ACK messages + (N - 1) release messages = 3(N - 1) messages (or 2(N - 1) messages if deferred ACKs are merged with release messages).
```

---

### 3. RPC Latency Limits [6 pts]

#### Question 1 [6 pts]
Consider a synchronous (blocking) RPC call from client `C` to procedure `P` on server `S`. You are given the following primitive times:
* $t_c$: time to copy $n$ bytes of data.
* $t_x$: time to context switch between processes $P$ and $P'$.
* $t_n$: time to transmit $n$ bytes over the network.
* $t_s$: time for procedure $P$ to execute on server $S$.

Assume both the client's arguments and the server's response are $n$ bytes in size.
* **1a [2 pts]**: Assuming no optimizations, write an expression for the total RPC latency $t$ in terms of $t_c$, $t_x$, $t_n$, and $t_s$.
* **1b [2 pts]**: To reduce latency, the kernel guarantees that any context switches after sending data occur in parallel with network transmission. Write the updated expression for $t$.
* **1c [2 pts]**: In addition to the optimization in 1b, the kernel provides a shared buffer/descriptor that is directly used by both the client and server stubs. Write the updated expression for $t$.

**Answers:**

* **1a [2 pts]**:
```text
t = 6*t_c + 4*t_x + 2*t_n + t_s (or 4*t_c + 4*t_x + 2*t_n + t_s if counting only software copies and excluding NIC DMA copies; or 8*t_c if counting both send and receive DMA copies) — because an unoptimized blocking RPC performs data copies, context switches, and network transfers on both the call and return paths, plus server execution.
because:
- Data copies (6*t_c total based on 3 copies per direction in Lecture 05c, or 4*t_c software-only): On each direction, data is copied (1) between the application stack and user-space RPC stub buffer, (2) between the user stub buffer and kernel buffer, and (3) via DMA between the kernel buffer and NIC controller buffer.
- Context switches (4*t_x): 4 context switches occur across the round trip (C -> C1 on client after request send, S1 -> S on server upon request arrival, S -> S2 on server after reply send, and C1 -> C on client upon reply arrival).
- Network transmission (2*t_n): 1 network transmission of n bytes for the call (t_n) + 1 network transmission of n bytes for the return (t_n).
- Server execution (t_s): 1 execution of procedure P on server S.
```

* **1b [2 pts]**:
```text
t = 6*t_c + 2*t_x + 2*t_n + t_s (or 4*t_c + 2*t_x + 2*t_n + t_s for software-only copies, assuming t_x <= t_n; more generally 6*t_c + 2*t_x + 2*max(t_n, t_x) + t_s) — because overlapping post-send context switches with network transmission removes 2 context switches from the critical path.
because the 1st context switch (C -> C1 after client send) and the 3rd context switch (S -> S2 after server send) happen in parallel with the network wire transmissions (2*t_n), leaving only the 2 receive-side context switches (S1 -> S and C1 -> C, costing 2*t_x) on the critical path.
```

* **1c [2 pts]**:
```text
t = 4*t_c + 2*t_x + 2*t_n + t_s (or 2*t_c + 2*t_x + 2*t_n + t_s if counting only software copies) — because a shared kernel buffer/descriptor used directly by client and server stubs eliminates 1 user-to-kernel copy on the call path and 1 on the return path (saving 2*t_c).
because stubs marshal and unmarshal arguments and return values directly into and out of the shared kernel buffer rather than copying between separate user stub buffers and kernel buffers on both directions.
```

---

### 4. Active Networks [7 pts]

#### Question 1 [4 pts]
Consider three active nodes `A -> B -> C`. A capsule belonging to a new protocol traverses `A`, then `B`, and has just arrived at `C`. Assume `B` already has the code for the capsule's `TYPE` in its soft store, while `C` has never seen this `TYPE` before.
* **1a [2 pts]**: Walk through step-by-step what happens at `C` from the moment the capsule arrives until it is forwarded.
* **1b [2 pts]**: Now suppose that just before `C` requests the code, `B`'s soft store was cleared due to a router restart and no longer has the code. How does this change the outcome at `C`, and what does this tell us about the `PREV`-based design?

**Answers:**

* **1a [2 pts]**:
```text
Soft-Store Miss at C -> Code Request to PREV (B) -> Cryptographic Hash Verification -> Cache, Execute, and Forward — because ANTS loads and verifies missing forwarding routines on demand from the immediate upstream active router.
because:
1. Soft-store lookup miss: Node C inspects the capsule's ANTS header, extracts the TYPE field (128-bit MD5 hash of the protocol code), and looks it up in C's local soft store, resulting in a miss.
2. Code request to PREV: Node C reads the PREV field of the capsule header (pointing to Node B) and sends a code request message for TYPE to Node B.
3. Code transfer & verification: Node B fetches the code from its soft store and sends it to C; C computes the MD5 fingerprint of the received code and verifies that it matches the capsule's TYPE field to prevent code spoofing.
4. Caching, execution, and forwarding: Once verified, C caches the code in its soft store for future capsules, executes the code in its Java sandbox, updates the capsule's PREV field to C, and forwards the capsule.
```

* **1b [2 pts]**:
```text
Node C drops the capsule and relies on end-to-end retransmission from the source — because ANTS does not recursively query upstream nodes beyond PREV.
because when B no longer has the code after a restart, C does not chain requests backward to A and simply discards the capsule; this shows that ANTS's PREV-based mechanism is a lightweight, best-effort, soft-state design that keeps routers simple and relies on end-to-end transport protocols at the source (A) to time out, retransmit, and repopulate soft stores along the path.
```

---

#### Question 2 [3 pts]
A student proposes embedding the full executable code directly inside every capsule instead of using a `TYPE` field and a soft store. State one **Advantage** and one **Disadvantage** of this approach compared to ANTS.

**Answer:**
```text
Advantage: Eliminates cold-start code-fetch delay and capsule drops caused by soft-store evictions; Disadvantage: Massive network bandwidth and transmission latency overhead on every packet — because carrying code in every capsule trades link bandwidth for stateless forwarding.
because:
- Advantage: Every capsule is completely self-contained, so routers never stall waiting to fetch missing code from PREV on cold-start packets and never drop capsules if upstream routers evict code or restart.
- Disadvantage: Repeatedly transmitting the full executable code in every packet of a flow wastes enormous network bandwidth and increases packet transmission delay on every hop, whereas ANTS amortizes a single code transfer across the entire flow via soft-store caching.
```

---

### 5. Distributed Objects and Middleware [10 pts]

#### Spring OS — Question 1 [7 pts]
Originally, four components (`Client A`, `Client B`, `Server Y`, and `Server Z`) run on a single node running the Spring Operating System. Later, the system is split across two nodes connected by a LAN:
* **Node 1**: `Client A`, `Server Z`
* **Node 2**: `Client B`, `Server Y`

* **1a [2 pts]**: A coworker claims that the **client-side stubs** must be changed so that `Client A` (on Node 1) can access the now-remote `Server Y` (on Node 2). Point out the fallacy in this claim and explain what actually changes.
* **1b [3 pts]**: Suppose `Object Y` (hosted by `Server Y`) supports both `read` and `write` operations. `Client A` requires `read/write` access, whereas `Client B` should only have `read-only` access. Explain how Spring's mechanisms (`capabilities`, `access control`, `doors`) work together to achieve this.
* **1c [2 pts]**: Suppose `Client A` later passes its `read/write` capability for `Object Y` to `Client B`. How does `Client B` gain `write` access to `Object Y`?

**Answers:**

* **1a [2 pts]**:
```text
Fallacy: Client-side stubs are completely location-independent and do NOT change; What actually changes is the underlying Subcontract and Network Proxies — because Spring decouples stub marshaling from object communication mechanics.
because client stubs depend solely on the IDL interface and invoke the subcontract interface identically whether the server is local or remote; to cross the LAN from Node 1 to Node 2, only the subcontract changes to route calls through local Doors into Network Proxy domains on Node 1 and Node 2 connected by a network handle.
```

* **1b [3 pts]**:
```text
Server Y uses Front Objects, an Access Control List (ACL), distinct Doors per privilege level, and kernel-protected Door Capabilities — because Spring enforces access control at domain entry points.
because:
1. Front Object & ACL: Server Y places a Front Object (or multiple Front Objects) in front of the underlying Object Y implementation and checks an Access Control List (ACL) defining each client's permissions.
2. Distinct Doors: The Front Object creates separate Doors registered with the Nucleus corresponding to different access levels—one Door entry point allowing both read and write operations, and a second Door entry point allowing only read operations.
3. Software Capabilities (Door Handles): Client A is issued a software capability (a door handle in its domain's door table, bridged via Network Proxies) pointing to the read/write Door, whereas Client B is issued a door handle pointing only to the read-only Door.
```

* **1c [2 pts]**:
```text
Client A's door handle is a transferable software capability that the Nucleus and Subcontract marshal and install into Client B's door table — because holding a door capability in Spring directly grants the access rights associated with that door.
because when Client A passes its read/write object reference as a parameter to Client B, the subcontract and nuclei (via Network Proxies between Node 1 and Node 2) copy that capability into Client B's domain door table referencing the same high-privilege read/write Door/Front Object on Server Y, allowing Client B to invoke write operations through that handle.
```

---

#### Java RMI — Question 1 [3 pts]
A client connects to a remote object using Java RMI. During execution, the underlying network connection is abruptly severed, yet the client automatically reconnects to the remote object without propagating any error to the client application. Describe the role of each of the three Java RMI layers (`Stub/Skeleton Layer`, `Remote Reference Layer`, and `Transport Layer`) in handling this scenario.

**Answer:**
```text
Stub/Skeleton shields the client application, Remote Reference Layer executes transparent retry/reconnection semantics, and Transport Layer detects the broken link and establishes a new network connection — because Java RMI encapsulates connection recovery below the stub layer.
because:
1. Stub/Skeleton Layer: Remains completely unchanged and transparent to the failure; the client stub simply blocks waiting for the invocation stream from the RRL without throwing any exception to the client application.
2. Remote Reference Layer (RRL): Implements the invocation and reconnection semantics; when the transport layer reports a lost connection, the RRL intercepts the failure, requests connection re-establishment to the server endpoint, and re-transmits the marshaled call instead of raising a RemoteException to the stub.
3. Transport Layer: Detects the severed network link via connection liveness monitoring, cleans up the dead TCP/UDP channel, and opens a fresh network connection to the remote endpoint when instructed by the RRL.
```

---

### 6. Distributed Subsystems [32 pts]

#### GMS — Question 1 [5 pts]
Consider a 3-node cluster (`A`, `B`, `C`) running GMS. For `Page P`, **Node A** is the **Home Node** (hosting the GCD entry for `P`). At time `T0`, the state is:
* GCD at Home Node (A) for P: `{A}`
* PFD at Node A: `Local-Private`
* PFD at Node B: `Non-existent`
* PFD at Node C: `Non-existent`

* **1a [2 pts]**: **Event 1 (`T1 > T0`)**: A process on **Node B** page faults on `Page P` (Case 4: shared page in Node A's local memory). What is the state of the 4 entries above after Event 1?
* **1b [1 pt]**: **Event 2 (`T2 > T1`)**: **Node A** experiences memory pressure and intends to evict clean `Page P`. Assume **Node C** would be the candidate peer node if `Page P` were placed in global memory. State whether the following statement is **True** or **False** and justify: *"Node A will send the Page P to Node C's global part of the DRAM."*
* **1c [2 pts]**: What is the state of the 4 entries above after Event 2?

**Answers:**

* **1a [2 pts]**:
```text
1. GCD at Home Node (A) for Page P: {A, B}
2. PFD at Node A: Local-Shared
3. PFD at Node B: Local-Shared
4. PFD at Node C: Non-existent
— because when Node B faults on Page P residing in Node A's local memory (Case 4), Node A sends a shared copy of P to Node B's local cache, both A and B mark P as Local-Shared in their PFDs, and the GCD at A records both A and B as sharers.
```

* **1b [1 pt]**:
```text
False — because Page P is in the Local-Shared state with a valid duplicate copy already present in Node B's local DRAM ({A, B}), so Node A simply discards its local copy of P rather than sending a redundant copy to Node C's global memory.
because GMS only forwards an evicted page to a peer's global memory when the evicted copy is the sole copy in the cluster (Local-Private); when another node (Node B) already caches the page, sending it to Node C would waste network bandwidth and global DRAM.
```

* **1c [2 pts]**:
```text
1. GCD Owner List at Home Node (A) for Page P: {B}
2. PFD at Node A: Non-existent
3. PFD at Node B: Local-Private (or remains Local-Shared locally if Node B's PFD is updated lazily, since B is now the sole holder)
4. PFD at Node C: Non-existent
— because Node A drops its local copy of P and removes itself from the GCD sharer list at A, leaving Node B as the sole holder of Page P while Node C never receives the page.
```

---

#### GMS — Question 2 [2 pts]
**True/False**: In GMS, when a node decides to send a page to a peer node's global part of the DRAM, it writes the page to the disk if the page is dirty. Explain your answer.

**Answer:**
```text
True — because GMS enforces the strict invariant that all pages stored in any peer node's global memory cache must be clean.
because flushing dirty pages to disk before placing them into a peer's global memory ensures that if a peer node crashes, no uncommitted data is lost.
```

---

#### GMS — Question 3 [2 pts]
**True/False**: A simpler and more efficient design in GMS would be to broadcast the UID of the missing page to all the peers upon a page fault at a node. Explain your answer.

**Answer:**
```text
False — because broadcasting on every page fault incurs O(N) network messages and CPU interrupt/lookup overhead at every peer in the cluster, which scales poorly compared to GMS's directory hierarchy.
because GMS's POD -> GCD -> PFD hierarchy locates any page in at most 2 to 3 point-to-point network hops without disturbing unrelated nodes.
```

---

#### GMS — Question 4 [2 pts]
A co-worker argues for maintaining a replicated data structure `UID -> Node currently hosting the page` at every node in the cluster so that any page fault takes only 1 network hop. Provide two succinct bullets to counter her argument.

**Answer:**
```text
Prohibitive cluster-wide update broadcast traffic and excessive local DRAM memory footprint — because page locations change constantly across the cluster.
because:
1. Massive Update/Coherence Traffic: Unlike the POD (which maps coarse partitions and changes only when nodes join or leave), a replicated UID-to-Host table would require broadcasting update messages to all N nodes on every single page fault, eviction, or disk read across the cluster, saturating the network.
2. Huge Memory Footprint: Replicating fine-grained mappings for every cached page UID across the entire cluster in every node's RAM would consume excessive local DRAM.
```

---

#### DSM — Question 1a [5 pts]
Consider a **TreadMarks** DSM system with 8 nodes (`N1..N8`), where virtual page `X` is initially replicated across all 8 nodes. A global lock `L1` is acquired sequentially in the order `N1, N2, N3, N4, N5, N6, N7, N8`:
* `N1` writes to page `X` inside its critical section under `L1`.
* `N2..N7` acquire and release `L1` in order, but do **NOT** access page `X`.
* `N8` acquires `L1` and **reads** page `X`.

Which consistency model performs better here — **Eager Release Consistency (ERC)** or **Lazy Release Consistency (LRC)**? Calculate the total number of DSM-specific messages (invalidation messages, update messages, or diff requests/responses — excluding lock transfer messages) in both ERC and LRC to justify your answer.

**Answer:**
```text
Lazy Release Consistency (LRC) performs significantly better (2 DSM messages in LRC vs. 9 DSM messages in invalidate-based ERC, or 16 with ACKs, or 7 in update-based ERC) — because LRC piggybacks invalidation notices on lock transfers and avoids sending coherence messages to nodes (N2..N7) that never touch Page X.
because:
- Under Lazy Release Consistency (LRC): When N1 releases L1, 0 standalone invalidation messages are sent because the write notice for X is piggybacked on the lock L1 transfer messages (N1 -> N2 -> ... -> N8). Since N2..N7 never access X, they send 0 diff requests. When N8 reads X, N8 faults and sends 1 diff request to N1 and receives 1 diff response from N1, totaling 2 DSM-specific messages.
- Under Eager Release Consistency (ERC): When N1 releases L1, it eagerly broadcasts invalidations (or updates) for X to all 7 other caching nodes N2..N8 (7 messages, or 14 messages if invalidation ACKs are counted). When N8 later reads X, an invalidate-based ERC faults and sends 1 page/diff request to N1 and receives 1 response (2 messages), totaling 9 DSM-specific messages (or 16 with ACKs; even update-based ERC sends 7 eager update messages vs. LRC's 2 messages).
```

---

#### DSM — Question 2 [8 pts]
Consider a **TreadMarks** DSM system executing the following sequence across timestamps `T1 < T2 < T3 < T4` on four processors (`P1`, `P2`, `P3`, `P4`):
* **`T1` (`P1`)**: `acq(L1); write X; rel(L1)`
* **`T2` (`P2`)**: `acq(L1); write Y; rel(L1)`
* **`T3` (`P3`)**: `acq(L2); write X; rel(L2)` *(Note: uses lock `L2`!)*
* **`T4` (`P4`)**: `acq(L1); read X; write Y; rel(L1)`

Assume pristine copies of both `X` and `Y` reside at owner node `O` throughout, and neither `X` nor `Y` is initially present at `P1..P4`.
* **2a [2 pts]**: What actions occur when `P4` acquires lock `L1` at `T4`?
* **2b [4 pts]**: While executing its critical section under `L1`, `P4` first reads `X` and then writes `Y` (`read X; write Y`). What actions are needed?
* **2c [2 pts]**: What happens when `P4` releases lock `L1` (`rel(L1)`)?

**Answers:**

* **2a [2 pts]**:
```text
P4 receives piggybacked write notices for Page X (from P1 at T1) and Page Y (from P2 at T2) along L1's causal chain, and records that X and Y require those diffs (no local pages are invalidated since X and Y are not present at P4) — because P2 transfers lock L1 to P4 together with L1's causal invalidation notices.
because L1's causal synchronization chain is P1 (T1) -> P2 (T2) -> P4 (T4), so the lock grant includes write notices for P1's modification to X and P2's modification to Y, while ignoring P3's modification to X at T3 because T3 used a different, causally unrelated lock (L2).
```

* **2b [4 pts]**:
```text
On read X: fetch clean copy of X from Owner O, fetch and apply only diff Xd1 from P1, and map X as read-only (no twin created); On write Y: fetch clean copy of Y from Owner O, fetch and apply diff Yd2 from P2, create a twin of Y, and map Y as read-write — because TreadMarks only applies causally preceding diffs and only creates twins on writes.
because:
1. On read X: P4 faults on missing Page X, fetches the pristine copy of X from Owner O, requests and applies only diff Xd1 from P1 (it does NOT fetch Xd3 from P3 because T3 was protected by L2, which has no happens-before relation with L1), and maps X as read-only without creating a twin.
2. On write Y: P4 faults on missing Page Y, fetches the pristine copy of Y from Owner O, requests and applies diff Yd2 from P2 (from T2 under L1), allocates a twin of Y in system space, and maps Y as read-write so the write can proceed.
```

* **2c [2 pts]**:
```text
P4 compares Page Y against its twin to create diff Yd4, marks Page Y read-only (write-protected), and discards Y's twin (Page X remains untouched as read-only) — because P4 only modified Page Y during its critical section.
because upon rel(L1), TreadMarks generates run-length encoded diff Yd4 from Y and its twin, downgrades Y's OS protection back to read-only so future writes will trap, and frees Y's twin.
```

---

#### DFS — Question 1 [2 pts]
In the **xFS** distributed file system, the **"First Writer"** policy co-locates a file's Metadata Manager on the same workstation as the client that created the file. Based on the evaluation in the xFS paper, explain why this policy helps **writes** much more than **reads**.

**Answer:**
```text
Write-sharing across different clients is rare so First Writer makes ~90% of write metadata messages local (0 network hops), whereas reads either hit in the local client cache already or require remote network hops to storage servers/peers anyway — because file creation and subsequent overwrites exhibit strong single-client write locality.
because in typical workloads, the creator of a file is almost always the same client that later overwrites or deletes it (96% of overwrites are by the previous writer), making 90% of write-ownership requests and 90% of disk-flush notifications to the manager completely local; by contrast, over 75% of reads already hit in the client's local cache without contacting the manager, and read misses must traverse the network to remote storage servers or peer caches regardless.
```

---

#### DFS — Question 2 [3 pts]
Suppose an engineer proposes simplifying **xFS** by removing **stripe groups** altogether and instead striping each log segment across **ALL** hundreds of storage servers in the cluster. Based on the xFS paper, give **3 distinct issues** that would arise from this design.

**Answer:**
```text
Small fragment/memory buffering overhead, loss of parallel I/O bandwidth, high log cleaning overhead, and drastically reduced fault tolerance — because striping every segment across hundreds of servers breaks scalability and reliability boundaries.
because (any 3 of the following 4 issues from the xFS paper):
1. Small-Write / Memory Buffering Problem: Striping a segment across hundreds of servers either produces tiny inefficient network/disk fragments per server or forces each client to buffer an impractically huge segment in RAM before flushing.
2. Loss of Write Parallelism: Every client write touches all storage servers simultaneously instead of allowing different clients to write in parallel at full bandwidth to disjoint subsets of disks.
3. High Log Cleaning Overhead: Huge segments spanning all servers remain "partially full" much longer than smaller stripe-group segments, forcing cleaners to read enormous partially-full segments across the network.
4. Reduced Availability / Fault Tolerance: With single-parity RAID across all hundreds of servers, any 2 simultaneous server failures anywhere in the cluster cause permanent data loss and halt the entire system, whereas independent stripe groups survive multiple failures across different groups.
```

---

#### DFS — Question 3 [3 pts]
In **xFS**, **index nodes** (which contain the disk log addresses of a file's data blocks) are cached at **Metadata Managers**, but explicitly **NOT** cached at **Clients**.
* **3a [1 pt]**: If index nodes were cached at clients, what would be the performance benefit? Specify which network messages would be eliminated.
* **3b [2 pts]**: State **two distinct drawbacks** from the xFS paper that explain why the authors chose not to cache index nodes at clients.

**Answers:**

* **3a [1 pt]**:
```text
Clients could read data blocks directly from Storage Servers on a local cache miss, eliminating the Client -> Manager read request message and the Manager -> Storage Server forwarding message (saving 2 network hops) — because the client could look up the data block's disk log address locally in its cached index node.
```

* **3b [2 pts]**:
```text
Loss of cooperative caching from peer client memories, continued need to notify the manager for cache consistency, and added index-node coherence complexity — because bypassing the manager on reads undermines xFS's cooperative caching and consistency tracking.
because (any 2 of the following 3 drawbacks from the xFS paper):
1. Loses Cooperative Caching: Reading directly from a Storage Server's disk using a local index node bypasses the Manager and loses the opportunity to fetch the block much faster from another peer client's memory via cooperative caching.
2. Manager Notification Still Required: Even if a client reads a block directly from disk, it must still send a network message to notify the Manager that it is now caching the block so the Manager can track sharers and invalidate the block on future writes.
3. Extra Coherence Complexity: Caching index nodes at clients would require implementing an additional cache invalidation/consistency protocol for the index nodes themselves whenever a file is written and its index node moves in the log.
```

---

### 7. HIDDEN — Potpourri [15 pts]

#### Question 1 [2 pts]
**True/False**: In Lamport's mutual exclusion lock algorithm, when a node `A` determines that it has the lock, the contents of the local queues on all the nodes are identical. Explain your answer.

**Answer:**
```text
False — because while Node A's request is guaranteed to be at the head of every node's queue, other nodes' later requests or release messages may still be in transit across the variable-latency network.
because Node A only needs local confirmation that its own request is at the head of its queue and that every peer has sent a message with a higher timestamp, leaving the tail entries of different nodes' local queues temporarily inconsistent.
```

---

#### Question 2 [2 pts]
**True/False**: Doubling the soft store size at an active router is unnecessary because any evicted breadcrumb or code can always be recovered by requesting the code from the `PREV` node. Explain your answer.

**Answer:**
```text
False — because the PREV node also has a finite soft store that may have evicted the code (causing the capsule to be dropped), and dynamic per-flow breadcrumbs cannot be recovered simply by fetching static code from PREV.
because (1) ANTS does not recursively query upstream beyond PREV, so if PREV also evicted the code the capsule is dropped, (2) fetching evicted code from PREV adds network round-trip latency on the critical path, and (3) breadcrumbs hold dynamic per-flow state left at that specific router rather than static protocol code.
```

---

#### Question 3 [2 pts]
Alice submits a request `A` to a remote server. After submitting, she calls Bob on the phone to tell him about it. Bob then submits a request `B` to the same remote server. The distributed system uses Lamport's Logical Clock (breaking ties using machine ID) to totally order events. There is no message loss, and messages between any two nodes are delivered in FIFO order. Will the remote server always fulfill Alice's expectation that her request `A` is satisfied before Bob's request `B`? Explain your answer.

**Answer:**
```text
No (False) — because the phone call between Alice and Bob is an out-of-band (external physical) communication channel outside the distributed system, so Lamport's logical clocks cannot capture the causal dependency between Request A and Request B.
because Alice's machine and Bob's machine did not exchange an in-system message carrying Alice's logical clock value before Bob sent Request B; therefore, Bob's logical clock C(B) may be smaller than Alice's C(A) (or B may arrive at the server before A), causing the server to order and execute B ahead of A.
unless Alice and Bob communicate through in-band system messages stamped with logical clocks (or use synchronized physical clocks), external real-time causality is not preserved.
```

---

#### Question 4 [4 pts]
State one **Advantage (Pro)** and one **Disadvantage (Con)** each for the following EJB application design patterns:
1. **Entity Beans (Data Access Objects)**
2. **Session Facade (Session Bean with Entity Bean)**

**Answer:**
```text
Entity Beans provide high database concurrency at the cost of exposing business logic and IPC overhead, whereas Session Facade hides business logic and reduces WAN round-trips while preserving concurrency at the cost of an extra indirection layer — because Session Facade wraps fine-grained Entity Beans behind a coarse-grained Session Bean.
because:
1. Entity Beans (Data Access Objects):
   - Advantage (Pro): High concurrency and parallelism for database access (multiple entity bean instances can cache and query different database rows in parallel across clients).
   - Disadvantage (Con): Exposes internal business logic to external callers (e.g., web container/WAN) and incurs high container management and IPC overhead for fine-grained remote calls.
2. Session Facade (Session Bean with Entity Bean):
   - Advantage (Pro): Achieves the best of both worlds by encapsulating business logic securely inside the EJB container (reducing fine-grained WAN round-trips) while still exploiting parallel database access via the underlying Entity Beans.
   - Disadvantage (Con): Adds an extra layer of indirection and inter-bean invocation overhead between the Session Bean and Entity Beans (unless mitigated using co-located local interfaces).
```

---

#### Question 5 [5 pts]
Consider the **TreadMarks** DSM system:
* **5a [2 pts]**: In TreadMarks' multiple-writer protocol, what actions occur when a processor performs the **first write** to a shared page in an interval?
* **5b [3 pts]**: **True/False**: If two processors modify overlapping words of the same page without intervening synchronization, TreadMarks will deterministically merge both updates by applying the diffs in order. Explain your answer.

**Answers:**

* **5a [2 pts]**:
```text
An OS write-protection fault traps into TreadMarks, which allocates a "twin" (an identical copy of the clean page in system space) and upgrades the page's OS protection from read-only to read-write — because TreadMarks needs a clean baseline copy of the page to compute a word-by-word diff at release time.
because valid shared pages are initially mapped read-only; the first write triggers a protection fault so TreadMarks can copy the page into a twin and make the page writable, allowing all subsequent writes to that page in the same interval to proceed at hardware speed without faulting.
```

* **5b [3 pts]**:
```text
False — because modifying overlapping words of the same page without intervening synchronization is a data race, leaving the two write intervals concurrent with no happens-before causal ordering between their diffs.
because Lazy Release Consistency in TreadMarks relies on synchronization operations (locks/barriers) to establish a causal happens-before order over intervals and diffs; when two processors write to the same word concurrently without synchronization, TreadMarks has no deterministic order in which to merge the conflicting diffs.
```
