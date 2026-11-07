# CS 6210 — Giải Chi Tiết Đề Thi Fall 2026 Test 2 (78 Points)

> **Nguồn đề**: [CS6210 Fall 2026 Test 2 Release.pdf](file:///D:/OMSCS/AOS/CS6210-Advanced-Operating-Systems-Notes/Previous%20Tests/CS6210%20Fall%202026%20Test%202%20Release.pdf)
>
> **Quy tắc trình bày chuẩn chấm thi (Mandatory Exam Format)**:
> ```text
> [Yes / No / True / False / Direct Answer] — because [primary justification x]
> because [detailed mechanism x]
> unless [boundary condition / exception y]
> also [additional consequence / supporting fact z]
> ```

---

## 1. Distributed Systems [24 points]

### 1.1. Lamport's Logical Clock [10 points]

#### Question 1 [8 points]
Consider a distributed system consisting of three processes `P`, `Q`, and `R`, each maintaining a Lamport logical clock $C_p$, $C_q$, and $C_r$, respectively. All logical clocks are initialized to 0:
$$C_p(p_1) = C_q(q_1) = C_r(r_1) = 0$$
When a clock is incremented, it is incremented by the smallest positive integer ($+1$) necessary to satisfy Lamport's logical clock conditions. Each message carries the sender's logical timestamp.
From the diagram:
* $m_1$: $p_2 \rightarrow q_2$
* $m_2$: $q_3 \rightarrow r_3$
* $m_3$: $p_3 \rightarrow q_4$
* $m_4$: $r_4 \rightarrow p_5$
* $m_5$: $q_5 \rightarrow r_5$

> **Bảng tra cứu nhanh toàn bộ giá trị đồng hồ logic trên sơ đồ**:
> * **Process P**:
>   * $C_p(p_1) = 0$
>   * $C_p(p_2) = 0 + 1 = \mathbf{1}$ *(gửi $m_1$ với $TS = 1$ tới $q_2$)*
>   * $C_p(p_3) = 1 + 1 = \mathbf{2}$ *(gửi $m_3$ với $TS = 2$ tới $q_4$)*
>   * $C_p(p_4) = 2 + 1 = \mathbf{3}$
>   * $C_p(p_5) = \max(C_p(p_4), C_r(r_4)) + 1 = \max(3, 5) + 1 = \mathbf{6}$ *(nhận $m_4$ từ $r_4$)*
>   * $C_p(p_6) = 6 + 1 = \mathbf{7}$
> * **Process Q**:
>   * $C_q(q_1) = 0$
>   * $C_q(q_2) = \max(C_q(q_1), C_p(p_2)) + 1 = \max(0, 1) + 1 = \mathbf{2}$ *(nhận $m_1$ từ $p_2$)*
>   * $C_q(q_3) = 2 + 1 = \mathbf{3}$ *(gửi $m_2$ với $TS = 3$ tới $r_3$)*
>   * $C_q(q_4) = \max(C_q(q_3), C_p(p_3)) + 1 = \max(3, 2) + 1 = \mathbf{4}$ *(nhận $m_3$ từ $p_3$)*
>   * $C_q(q_5) = 4 + 1 = \mathbf{5}$ *(gửi $m_5$ với $TS = 5$ tới $r_5$)*
>   * $C_q(q_6) = 5 + 1 = \mathbf{6}$
> * **Process R**:
>   * $C_r(r_1) = 0$
>   * $C_r(r_2) = 0 + 1 = \mathbf{1}$
>   * $C_r(r_3) = \max(C_r(r_2), C_q(q_3)) + 1 = \max(1, 3) + 1 = \mathbf{4}$ *(nhận $m_2$ từ $q_3$)*
>   * $C_r(r_4) = 4 + 1 = \mathbf{5}$ *(gửi $m_4$ với $TS = 5$ tới $p_5$)*
>   * $C_r(r_5) = \max(C_r(r_4), C_q(q_5)) + 1 = \max(5, 5) + 1 = \mathbf{6}$ *(nhận $m_5$ từ $q_5$)*
>   * $C_r(r_6) = 6 + 1 = \mathbf{7}$

---

##### 1a) [2 points] What is the value of $C_p$ at event $p_3$?

**Exam Answer:**
```text
Cp(p3) = 2 — because Cp starts at Cp(p1) = 0 and increments by 1 at each sequential event on Process P.
because p2 is the second event on P with no incoming message, giving Cp(p2) = Cp(p1) + 1 = 1, and p3 is the next sequential event on P with no incoming message, giving Cp(p3) = Cp(p2) + 1 = 2.
```

---

##### 1b) [2 points] What is the value of $C_q$ at event $q_3$? What is the relative ordering of the events $p_3$ and $q_3$? Explain.

**Exam Answer:**
```text
Cq(q3) = 3, and events p3 and q3 are causally concurrent (p3 || q3, neither happens before the other; though under a total logical clock ordering p3 precedes q3 since Cp(p3) = 2 < Cq(q3) = 3) — because there is no directed causal path of messages and sequential events between p3 and q3.
because:
1. Value of Cq(q3): Cq(q1) = 0; at q2, Process Q receives message m1 from p2 (where Cp(p2) = 1), so Cq(q2) = max(Cq(q1), Cp(p2)) + 1 = max(0, 1) + 1 = 2; then sequential event q3 increments Cq by 1 to Cq(q3) = 2 + 1 = 3.
2. Relative Ordering of p3 and q3: Event p3 and event q3 are concurrent (p3 -/-> q3 and q3 -/-> p3) because message m3 sent at p3 is not received by Q until q4 (after q3), and no message sent at or after q3 reaches P at or before p3.
also note that while Cp(p3) = 2 < Cq(q3) = 3 places p3 before q3 in a Lamport total order, in Lamport clocks C(a) < C(b) does not imply a -> b.
```

---

##### 1c) [2 points] What is the value of $C_r$ at event $r_3$?

**Exam Answer:**
```text
Cr(r3) = 4 — because at r3 Process R receives message m2 sent by Process Q at q3 with timestamp Cq(q3) = 3.
because Cr starts at Cr(r1) = 0, increments by 1 at internal event r2 to Cr(r2) = 1, and upon receiving m2 at r3 updates to Cr(r3) = max(Cr(r2), Cq(q3)) + 1 = max(1, 3) + 1 = 4.
```

---

##### 1d) [2 points] What is the value of $C_p$ at event $p_5$?

**Exam Answer:**
```text
Cp(p5) = 6 — because at p5 Process P receives message m4 sent by Process R at r4 with timestamp Cr(r4) = 5.
because on Process P, Cp(p3) = 2 and internal event p4 increments Cp to Cp(p4) = 3; meanwhile on Process R, Cr(r3) = 4 and sending m4 at r4 increments Cr to Cr(r4) = 5; therefore at receive event p5, Cp(p5) = max(Cp(p4), Cr(r4)) + 1 = max(3, 5) + 1 = 6.
```

---

#### Question 2 [2 points]
Your friend argues that a state-of-the-art multicore CPU is a distributed system per Lamport's definition. Is she right? Justify your answer succinctly. No credit without justification.

* **Giải thích tiếng Việt**: Theo định nghĩa của Lamport trong bài giảng `L05`, một hệ thống là hệ phân tán khi **độ trễ truyền thông điệp ($T_{\text{send}}$) không thể bỏ qua (non-negligible) so với khoảng thời gian giữa các sự kiện/lệnh trong một tiến trình đơn lẻ ($T_{\text{step}}$)**. Trên một chip CPU đa nhân hiện đại, việc truyền tín hiệu qua interconnect/bus giữa các core và bộ nhớ đệm mất nhiều chu kỳ xung nhịp, lớn hơn đáng kể so với thời gian thực thi một lệnh nội bộ trên 1 core, nên bạn của bạn **ĐÚNG (Yes)**.

**Exam Answer:**
```text
Yes (True) — because per Lamport's definition, a system is distributed whenever the message transmission delay between components is not negligible compared to the time between events within a single process.
because inside a modern multicore CPU, inter-core interconnect and cache-coherence communication latency across cores takes multiple clock cycles and is non-negligible compared to the sub-nanosecond execution time between instructions on a single core.
```

---

### 1.2. Lamport's ME Algorithm [8 points]

#### Question 1 [3 points]
Consider a distributed system with 5 nodes (`P1, P2, P3, P4, P5`) using Lamport's distributed mutual exclusion algorithm. `P2` wishes to acquire a lock. The current state of `P2`'s queue is shown below, and `P2` has not received any additional messages. Can `P2` acquire the lock? If yes, explain why. Otherwise, what additional messages does `P2` need to wait for?
* `P2`'s state (top to bottom): `(P3, 4)`, `(P2, 7)`, `(P5, 9)`, `(P1, 10)`

* **Giải thích tiếng Việt**: Hệ thống có 5 node (`P1..P5`). `P2` **KHÔNG** thể lấy khóa lúc này vì vi phạm cả 2 điều kiện của Lamport ME:
  1. `(P2, 7)` **chưa nằm ở đầu hàng đợi** vì `(P3, 4)` đang đứng đầu (`4 < 7`) $\Rightarrow$ `P2` phải đợi **thông điệp `Release (UNLOCK)` từ `P3`** (để xóa `(P3, 4)` khỏi đầu hàng đợi).
  2. Trong 4 node còn lại (`P1, P3, P4, P5`), `P2` mới chỉ biết `P5` (`9 > 7`) và `P1` (`10 > 7`) có timestamp lớn hơn `7`, nhưng **chưa nhận được bất kỳ thông điệp nào từ `P4`** (và thông điệp từ `P3` đang có timestamp `4 < 7`) $\Rightarrow$ `P2` phải đợi thêm **thông điệp `ACK` (hoặc một thông điệp có timestamp $> 7$) từ `P4`** (cùng với thông điệp `Release` có timestamp $> 7$ từ `P3`).

**Exam Answer:**
```text
No — because P2's request (P2, 7) is neither at the head of P2's local queue nor has P2 received messages with timestamps greater than 7 from all other 4 nodes in the 5-node system.
because to satisfy Lamport's two mutual exclusion conditions, P2 must still wait for:
1. A lock release (UNLOCK / REL) message from P3 (which removes (P3, 4) from the head of P2's queue so (P2, 7) becomes the head, and also provides a message from P3 with timestamp > 7), and
2. An acknowledgment (ACK) message (or a request/message) from P4 with a timestamp greater than (P2, 7), since P4 is completely absent from P2's received messages (while P5 at TS=9 and P1 at TS=10 have already sent messages with timestamps > 7).
```

---

#### Question 2 [5 points]
Consider the timeline diagram of Lamport's mutual exclusion lock algorithm with `P1`, `P2`, `P3`:
* `P1` requests lock at timestamp 5: `(P1, 5)` (sends red `L` arrows to `P2` and `P3`).
* `P3` requests lock at timestamp 8: `(P3, 8)` (sends red `L` arrows to `P1` and `P2`).
* Timeline events (`T1 < T2 < ... < T8`):
  * **`T1`**: `P1` receives red request `L` (`P3, 8`) from `P3`.
  * **`T2`**: `P2` receives red request `L` (`P1, 5`) from `P1` and red request `L` (`P3, 8`) from `P3`, and sends green `A` (ACK) arrows to `P1` and `P3`.
  * **`T3`**: `P3` receives red request `L` (`P1, 5`) from `P1`, and sends green `A` (ACK) arrow to `P1`.
  * **`T5`**: `P3` receives green `A` (ACK) arrow from `P2`.
  * **`T7`**: `P1` receives green `A` (ACK) arrow from `P2`.
  * **`T8`**: `P1` receives green `A` (ACK) arrow from `P3`.

---

##### 2a) [2 points] With reference to the above timeline of actions, which process will acquire the lock first? And at what timestamp?

* **Giải thích tiếng Việt**:
  * Tiến trình lấy khóa đầu tiên chắc chắn là **`P1`** vì yêu cầu `(P1, 5)` có nhãn thời gian nhỏ hơn `(P3, 8)` (`5 < 8`).
  * Về thời điểm (`timestamp`):
    * Theo **điều kiện tổng quát của Lamport** (Condition 2: nhận được *bất kỳ thông điệp nào* có timestamp $> 5$ từ mọi node khác): `P1` đã nhận được yêu cầu `(P3, 8)` (có timestamp $8 > 5$) từ `P3` ngay tại **`T1`**, và nhận được `ACK` từ `P2` tại **`T7`**. Do đó tại **`T7`**, `P1` đã nghe đủ từ cả `P2` và `P3` với timestamp $> 5$ và lấy được khóa!
    * Theo **quy tắc đợi đủ các mũi tên xanh `ACK` tường minh** trên hình: `P1` nhận `ACK` từ `P2` tại `T7` và nhận `ACK` từ `P3` tại **`T8`**, nên lấy khóa tại **`T8`**.
    * **Mẹo làm bài**: Ghi rõ cả **`T7` (or `T8` if waiting for explicit ACK messages from all peers)** ngay dòng đầu tiên để ăn trọn điểm dù rubric chấm theo cách nào!

**Exam Answer:**
```text
P1 acquires the lock first, at timestamp T7 (or T8 if requiring explicit ACK messages from every peer) — because P1's request (P1, 5) has the earliest logical timestamp (5 < 8) and is at the head of the request queue.
because:
- At T7 (under Lamport's general Condition 2 requiring any message with timestamp > 5 from every peer): P1 already received P3's request (P3, 8) with timestamp 8 > 5 at T1, and receives P2's ACK (timestamp > 5) at T7, satisfying both conditions at T7.
- At T8 (under the strict explicit-ACK rule): P1 receives the explicit green ACK message from P2 at T7 and the explicit green ACK message from P3 at T8, completing all explicit ACKs at T8.
```

---

##### 2b) [3 points] In the figure above, assume that the lock request message from P1 to P3 is lost due to a network glitch. With concise bullets summarize what could lead to both P1 and P3 acquiring the lock simultaneously.

**Exam Answer:**
```text
Both P1 and P3 see their own request at the head of their local queues and receive higher-timestamped messages from all peers, violating mutual exclusion — because the lost request from P1 to P3 leaves P3 unaware of (P1, 5).
because:
- P1's request (P1, 5) to P3 is lost in the network, so P3 never inserts (P1, 5) into its local queue and believes its own request (P3, 8) is at the head of its queue (satisfying Condition 1 at P3).
- P1 receives P3's request (P3, 8) at T1 (which has timestamp 8 > 5, satisfying P1's Condition 2 for P3) and sends an ACK (with timestamp > 8) back to P3; when P1 also receives P2's ACK at T7, P1 sees (P1, 5) at the head of its queue and enters the critical section.
- Meanwhile, P3 receives P2's ACK at T5 and receives P1's ACK (both with timestamps > 8), so P3 satisfies both Condition 1 and Condition 2 locally and also enters the critical section simultaneously with P1.
```

---

### 1.3. Active Networks [6 points]

#### Question 1 [3 points]
In the ANTS framework, an active router receives a capsule containing a `type` field that it has not seen before.
* **1a [1 pt]**: How does the active router obtain the code that corresponds to the type field?
* **1b [2 pts]**: Is it possible that the active router is unable to process the received capsule? How does ANTS deal with capsules that cannot be processed?

**Exam Answers:**

* **1a [1 pt]**:
```text
By sending a code request message to the immediate upstream active node specified in the capsule's PREV header field — because the previous active router just executed that capsule's forwarding routine and caches it in its soft store (and once received, the router verifies that the code's MD5 cryptographic hash matches the TYPE field before caching it locally).
```

* **1b [2 pts]**:
```text
Yes, it is possible, and ANTS simply drops (discards) the capsule and relies on end-to-end transport retransmission from the source — because the upstream PREV router may have evicted the code from its finite soft store (or restarted), or the code request/verification may fail.
because ANTS does not recursively query earlier upstream nodes beyond PREV, dropping the unprocessable capsule keeps routers simple and stateless.
```

---

#### Question 2 [3 points]
* **2a [2 pts]**: Give ONE pro and one CON of ANTS decision NOT to send code in every capsule.
* **2b [1 pt]**: Why is a sandboxed execution environment needed in ANTS framework?

**Exam Answers:**

* **2a [2 pts]**:
```text
Pro: Saves massive network bandwidth and reduces per-packet transmission latency; Con: Incurs cold-start code-fetch delay on soft-store misses and risks capsule drops if PREV evicted the code — because ANTS caches code in router soft stores rather than embedding it in every packet.
because:
- Pro: Capsules only carry a compact 128-bit MD5 TYPE fingerprint while the executable code is fetched once on demand and amortized across all subsequent capsules of the flow.
- Con: The first capsule of a flow stalls at each router to fetch code from PREV over the network, and capsules are dropped if PREV no longer has the code in its soft store.
```

* **2b [1 pt]**:
```text
To protect active routers and other flows from untrusted, buggy, or malicious capsule code — because active routers dynamically download and execute foreign code from untrusted users, so a sandbox (with bytecode verification, restricted APIs, and resource/TTL bounds) is required to prevent capsules from corrupting router state, interfering with other flows' soft-store state, or exhausting CPU/memory.
```

---

## 2. Distributed Objects and Middleware [14 points]

### 2.1. Spring OS [8 points]

#### Question 1 [4 points]
During a design discussion about remote procedure calls in Spring OS, a colleague argues:
*"Building distributed applications on Spring OS is completely effortless because the IDL compiler automatically generates Network Proxies directly from our interface definitions, eliminating the need for any runtime network infrastructure or manual networking code when making cross-node calls."*

* **1a [2 pts]**: Is your colleague's argument valid? If yes, why? If not, what artifacts does IDL compiler provide for the app developer?
* **1b [2 pts]**: Another colleague argues by saying: *"I disagree. Application developers need to write client-side and server-side proxies on top of application logic to bridge the network, which makes building software for Spring OS difficult."* Do you agree or disagree with your colleague? Justify.

**Exam Answers:**

* **1a [2 pts]**:
```text
No (Invalid) — the IDL compiler does NOT generate Network Proxies; instead, it generates (1) language-specific header files, (2) client-side stub code, and (3) server-side stub code — because the IDL compiler only translates interface definitions into language bindings and marshaling/unmarshaling stubs.
because cross-node networking is provided at runtime by Subcontracts and separate Network Proxy domains rather than being generated by the IDL compiler.
```

* **1b [2 pts]**:
```text
Disagree — because application developers do NOT write client-side or server-side network proxies on top of application logic; Spring OS provides Network Proxies and Subcontracts transparently as reusable system infrastructure.
because client and server stubs generated from IDL are completely location-independent, while the underlying Subcontract automatically routes cross-node calls through local Doors and system-supplied Network Proxy domains without requiring application developers to write networking code.
```

---

#### Question 2 [4 points]
Alice is reviewing the microkernel communication layer for a high-throughput microservice architecture based on Spring OS. After review she makes the following claim during a design review:
*"Because Spring uses a microkernel, every Door invocation across domains requires the Nucleus to inspect the caller's process ID (PID) and ACL permissions to verify whether the client is authorized to make the call. Furthermore, transferring execution between client and server domains incurs a high explicit cost."*

* **2a [2 pts]**: (True/False with Justification) Is Alice's claim regarding Nucleus PID/ACL inspection on every door call True or False?
* **2b [2 pts]**: (True/False with Justification) Is Alice's claim regarding high explicit cost for cross-domain calls correct?

**Exam Answers:**

* **2a [2 pts]**:
```text
False — because a Door in Spring is an unforgeable kernel-protected software capability, so simply possessing a valid door handle in the calling domain's door table proves authorization without the Nucleus inspecting PIDs or ACLs on every invocation.
because authentication and ACL checks are performed once when the object reference/door capability is initially issued (or at user level via Front Objects in the server domain), keeping the Nucleus fast-path door invocation free of ACL lookups.
```

* **2b [2 pts]**:
```text
False — because Spring's Nucleus and Doors mechanism are specifically designed to make cross-domain invocations ultra-fast (comparable to a protected procedure call, ~11 microseconds on SPARC) with minimal explicit overhead.
because on a door call, the Nucleus directly hands off execution from the calling thread to a pre-allocated server thread waiting at the target domain's door entry point and passes small arguments in hardware registers without heavy scheduler context switches or buffer copying.
```

---

### 2.2. EJB [4 points]

#### Question 1 [4 points]
An EJB flight-search service initially uses a coarse-grained session bean to perform the business logic for each client request. Within each request, airline lookups run sequentially. A developer replaces this with a finer-grained design in which the servlet coordinates entity-bean calls and runs independent airline lookups concurrently. With the same server hardware and the same airline queries, measurements show lower overall throughput (completed client requests per second).

* **1a [2 pts]**: Explain how the finer-grained EJB interactions could account for the lower throughput, despite the additional parallelism.
* **1b [2 pts]**: In the original design, does sequential processing within one client request prevent concurrent processing of requests from different clients? Explain.

**Exam Answers:**

* **1a [2 pts]**:
```text
High inter-tier communication, serialization, and EJB container management overhead across many fine-grained Entity Bean calls — because moving coordination from the Session Bean to the servlet forces every fine-grained lookup to cross the web-container-to-EJB-container boundary.
because each individual Entity Bean invocation from the servlet incurs remote procedure call (RMI/IPC) marshaling/unmarshaling, security and transaction context checks, thread synchronization, and bean lifecycle/pooling overhead in the EJB container, consuming CPU and memory resources that outweigh intra-request lookup parallelism on the same hardware.
```

* **1b [2 pts]**:
```text
No — because the EJB container maintains a pool of multiple Session Bean instances and worker threads that process requests from different clients concurrently (inter-request concurrency).
because even though a single client's airline lookups run sequentially inside one Session Bean instance, multiple client requests are served in parallel across separate Session Bean instances on the server.
```

---

### 2.3. Java RMI [2 points]

#### Question 1 [2 points]
A high-availability Java RMI server process is deployed in a shared cloud infrastructure. To support dynamic extensibility, the server dynamically downloads and executes unverified remote stub and parameter classes over the network at runtime. A malicious client attempts a denial-of-service attack by flooding the server with downloaded classes designed to aggressively consume excessive host CPU cycles and memory.
* **(True/False with justification)**: The in-built resource consumption regulation mechanisms of Java sandboxing will prevent these dynamically loaded classes from abusing host resources.

**Exam Answer:**
```text
False — because standard Java sandboxing only enforces access control (restricting which files, sockets, or system APIs a downloaded class may access via the SecurityManager and ClassLoader), and does NOT regulate or bound the amount of CPU cycles or heap memory a downloaded class can consume.
because Java lacks built-in per-class resource consumption quotas, malicious dynamically loaded classes can still execute infinite loops or allocate excessive memory in the shared JVM heap to cause a Denial-of-Service (DoS) attack.
```

---

## 3. Distributed Subsystems [26 points]

### 3.1. Global Memory Systems (GMS) [8 points]

#### Question 1 [5 points]
Consider a GMS cluster with nodes `A`, `B`, and `C`, where the page size is `4KB`. A process (`P1`) running on **Node A** faults on `page Y`. `Page Y` is present in the **local memory** of **Node B**, where it is actively being used by a process on **Node B**. The globally least recently used (LRU) page is currently in the **global memory** of **Node C**.
Assume:
* On all nodes, `local memory > 0` and `global memory > 0`.
* GMS uses a strict LRU replacement policy.

* **1a [3 pts]**: Describe the sequence of steps involved in handling the page fault.
* **1b [2 pts]**: Quantify the change in local and global memory on Nodes A, B, and C after the fault is handled.

**Exam Answers:**

* **1a [3 pts]**:
```text
Locate Page Y via POD/GCD/PFD on Node B -> Copy Page Y from Node B to Node A's local memory (marking both Local-Shared) -> Evict a global page from Node A to Node C's global memory -> Discard the globally LRU page from Node C's global memory — because faulting on an active shared page (Case 4) increases total cluster working-set pages by 1.
because:
1. Directory Lookup: Node A consults its local POD to identify the GCD owner for Page Y's UID, which checks the GCD and forwards the getpage request to Node B (whose PFD holds Page Y in its local memory).
2. Shared Page Copy (B -> A): Because Page Y is actively used on Node B, Node B retains its local copy, updates its PFD state for Y to Local-Shared, and sends a 4KB copy of Page Y to Node A's local memory (where Node A also marks Y as Local-Shared and the GCD records {A, B}).
3. Global Eviction & Victim Replacement (A -> C): Because Node A's local memory grows by 1 frame (+4KB), Node A shrinks its global memory by 1 frame (-4KB) by sending a page from Node A's global memory to Node C (which hosts the globally LRU page); Node C places this incoming page into its global memory and discards (drops) its globally LRU global page (which is guaranteed clean).
```

* **1b [2 pts]**:
```text
Node A: Local +4KB (+1 page), Global -4KB (-1 page); Node B: Local 0KB, Global 0KB; Node C: Local 0KB, Global 0KB — because Node A trades 1 global frame for 1 local frame, Node B keeps its shared local page, and Node C replaces 1 global frame with another global frame.
because:
- Node A: Local memory increases by +4KB (to accommodate the copy of Page Y) and Global memory decreases by -4KB (one global page sent to Node C).
- Node B: Local memory changes by 0KB and Global memory changes by 0KB (Page Y remains in Node B's local memory and is merely copied to Node A).
- Node C: Local memory changes by 0KB and Global memory changes by 0KB (Node C receives 1 global page (+4KB) from Node A and discards 1 globally LRU page (-4KB) from its global memory).
```

---

#### Question 2 [3 points]
Refer to the architecture of the modified OSF/1 memory management system (`VM`, `UBC`, `Pageout Daemon`, `Free List`, `TLB`, `GMS`, `Disk/NFS`, `Remote GMS`). Consider a GMS cluster with nodes A and B.
* **2a [2 pts]**: A `UBC` page and a `VM` page on the same node were last accessed at the same time. For which of the two can GMS easily obtain accurate age information and why? (no points without reasoning)
* **2b [1 pt]**: Node B crashes while holding pages from Node A in its global memory. Explain why this does not compromise the GMS integrity.

**Exam Answers:**

* **2a [2 pts]**:
```text
The UBC (Unified Buffer Cache) page — because file reads and writes to the UBC are explicit OS system calls that GMS can directly intercept in software to record exact access times.
because VM (Virtual Memory) anonymous pages (stack/heap) are accessed directly by CPU instructions in hardware without trapping into the OS on hits, forcing GMS to rely on a periodic background daemon that dumps the processor's TLB (Translation Lookaside Buffer) to approximate VM page age.
```

* **2b [1 pt]**:
```text
All pages stored in any peer's global memory are guaranteed to be clean (unmodified) copies that are already backed on disk — because GMS writes dirty pages to disk before placing them in global memory, so when Node B crashes, no data is lost and Node A can simply fetch those pages from disk on a global miss.
```

---

### 3.2. Distributed Shared Memory (DSM) [14 points]

#### Question 1 [9 points]
Consider the following sequence of actions in a **TreadMarks** DSM system. Assume pristine copies of pages `X` and `Y` are at their owner node `N0` throughout, and `X` and `Y` are initially not present on any other node. `T1–T4` are in increasing time order:
* `T1: N1: acq(L1)  write X           rel(L1)`
* `T2: N2: acq(L1)  read X   write Y  rel(L1)`
* `T3: N3: acq(L1)  write X           rel(L1)`
* `T4: N4: acq(L1)  read X            rel(L1)`

* **1a [2 pts]**: What happens at `N2` when it acquires `L1` and then reads `X`?
* **1b [2 pts]**: What happens when `N2` writes `Y`, and when it releases `L1`?
* **1c [2 pts]**: When `N4` reads `X`, whose diffs are applied to `X`, and in what order?
* **1d [1 pt]**: Will pristine copy and associated diff for `Page Y` be brought to `N4` during its critical section execution? Explain why or why not.
* **1e [2 pts]**: Assume each diff is fetched from the node that created it. Counting each request and each reply as distinct messages, how many messages does `N4`'s read of `X` generate?

**Exam Answers:**

* **1a [2 pts]**:
```text
At acq(L1), N2 receives a piggybacked write notice for Page X from N1; at read X, N2 faults, fetches the pristine copy of X from N0, fetches and applies diff Xd1 from N1, and maps X as read-only (with no twin created) — because X is initially missing at N2 and was modified by N1 at T1.
because Lazy Release Consistency piggybacks N1's invalidation notice for X onto the lock grant message to N2, and upon the read page fault N2 reconstructs the latest version of X from N0's pristine copy plus N1's diff Xd1 and maps X read-only since it is only reading X.
```

* **1b [2 pts]**:
```text
At write Y, N2 faults, fetches the pristine copy of Y from N0, creates a twin of Y, and marks Y read-write; at rel(L1), N2 creates diff Yd2 by comparing Y against its twin, marks Y read-only, and discards Y's twin — because TreadMarks uses twins and diffs to track writes per interval.
because Y is initially not present at N2 and has no prior diffs, so N2 fetches the clean page Y from N0 and makes a twin before writing; upon releasing L1, N2 summarizes its modifications into run-length encoded diff Yd2, write-protects Y to trap future writes, and frees the twin (no diff is created for X since N2 only read X).
```

* **1c [2 pts]**:
```text
Diffs from N1 and N3, applied in causal timestamp order: N1's diff (Xd1) first, followed by N3's diff (Xd3) — because N1 (at T1) and N3 (at T3) are the only nodes that modified Page X along L1's causal chain (N2 only read X at T2).
because applying N1's earlier diff Xd1 before N3's later diff Xd3 ensures that any overlapping words modified by both N1 and N3 reflect N3's more recent write at T3.
```

* **1d [1 pt]**:
```text
No — because under Lazy Release Consistency (LRC), even though N4 receives a piggybacked write notice for Page Y when acquiring L1, N4 never accesses (reads or writes) Page Y during T4, so no page fault occurs for Y and neither Y's pristine copy nor its diff is ever fetched.
```

* **1e [2 pts]**:
```text
6 messages total (4 messages to fetch diffs from N1 and N3 + 2 messages to fetch the pristine copy of X from owner N0) — because each network fetch requires 1 request message and 1 reply message.
because when N4 faults on reading missing Page X, N4 sends:
1. 1 request to owner N0 and receives 1 reply from N0 for the pristine copy of X (2 messages),
2. 1 diff request to N1 and receives 1 diff reply from N1 for Xd1 (2 messages), and
3. 1 diff request to N3 and receives 1 diff reply from N3 for Xd3 (2 messages),
totaling 3 requests + 3 replies = 6 messages (or 4 diff-specific messages if excluding the pristine page fetch from N0).
```

---

#### Question 2 [5 points]
Pages `X` and `Y` are shared (other nodes also cache them), but neither is present at `N1` at the start. `N1` runs the two critical sections below, and no other node acquires `L1` in between:
```text
acq(L1)
read X
write X
write X
read Y
rel(L1)

acq(L1)
write X
rel(L1)
```
* **2a [2 pts]**: In the first critical section, how many page faults occur, and how many twins are created?
* **2b [1 pt]**: How many diffs are created at the first release?
* **2c [2 pts]**: Does the write to `X` in the second critical section cause a page fault? Explain.

**Exam Answers:**

* **2a [2 pts]**:
```text
3 page faults occur, and 1 twin is created — because missing pages fault on first access and read-only pages fault on their first write in an interval.
because:
1. read X causes Page Fault #1 (X is not present at N1; N1 fetches X and maps it read-only; 0 twins created).
2. The first write X causes Page Fault #2 (write-protection fault because X is mapped read-only; N1 creates Twin #1 for X and upgrades X to read-write).
3. The second write X causes 0 faults and 0 twins because X is already mapped read-write in this interval.
4. read Y causes Page Fault #3 (Y is not present at N1; N1 fetches Y and maps it read-only; 0 twins created since Y is never written).
```

* **2b [1 pt]**:
```text
1 diff is created (for Page X only) — because only Page X was written (and had a twin created) during the first critical section, whereas Page Y was only read.
```

* **2c [2 pts]**:
```text
Yes — because at the first rel(L1), TreadMarks created the diff for Page X, discarded X's twin, and downgraded Page X's OS protection back to read-only (write-protected).
because even though Page X remains valid in N1's local memory and no other node acquired L1 in between, the write to X in the second critical section traps on a write-protection page fault so TreadMarks can allocate a fresh twin for the new interval and re-enable read-write access.
```

---

### 3.3. Distributed File Systems (DFS) [4 points]

#### Question 1 [4 points]
In **xFS**, the metadata manager for a file is statically assigned at file creation time.
* Your teammate **Joe** proposes: when a Node `N` creates a file, assign `N` as the metadata manager for that file (*First Writer policy*).
* Another teammate **Dick** proposes: choose the metadata manager to be different from the Node `N` where the file is created.

* **1a [2 pts]**: Client `C` created file `F` and now reads block `b` of `F`. The block is not in `C`'s cache, but it is cached at client `D`. Count each message between two different nodes as one network hop.
  * **(i)** How many network hops does it take for `b` to reach `C` with **Joe's policy**?
  * **(ii)** How many network hops does it take for `b` to reach `C` with **Dick's policy**?
* **1b [2 pts]**: Describe one concrete workload where **Joe's policy** would hurt performance and explain why.

**Exam Answers:**

* **1a [2 pts]**:
```text
(i) Joe's policy: 2 network hops (C -> D -> C); (ii) Dick's policy: 3 network hops (C -> M -> D -> C, unless Manager M happens to be chosen as D itself, which would take 2 hops) — because co-locating the Metadata Manager on creator C eliminates the initial Client-to-Manager network hop.
because:
- (i) Under Joe's policy (First Writer, Manager M = C): Client C looks up the metadata manager locally on C (0 network hops), sends a read request directly to peer Client D caching block b (Hop 1: C -> D), and Client D sends block b back to Client C (Hop 2: D -> C), totaling 2 network hops.
- (ii) Under Dick's policy (Manager M != C): Client C sends the read request over the network to remote Manager M (Hop 1: C -> M), Manager M forwards the read request to caching Client D (Hop 2: M -> D), and Client D transfers block b to Client C (Hop 3: D -> C), totaling 3 network hops (unless Dick's policy happened to assign D itself as Manager M, in which case C -> D -> C takes 2 hops).
```

* **1b [2 pts]**:
```text
A single-producer / bulk-file-creator workload (where one node creates or unpacks thousands of files that are then read and written by many other nodes across the cluster) — because Joe's policy statically assigns that single creator node as the Metadata Manager for all created files, creating a severe centralized bottleneck and hotspot.
because:
1. Hotspot & Load Imbalance: All metadata requests (cache misses, write-ownership requests, invalidations, and imap updates) from every client in the cluster for those files will flood the single creator node, exhausting its CPU, memory, and network bandwidth while other nodes' metadata managers remain idle.
2. Loss of Write Locality for Consumers: If Node C creates files that another Node D subsequently overwrites and updates repeatedly (a producer-consumer pipeline), keeping the manager on creator C forces Node D to pay remote network hops on every write instead of co-locating the manager with the actual active writer.
```
