# CS 6210 — Đề Cương Ôn Tập Test 2 (Chapters 5, 6, 7 & Giải Chi Tiết Fall 25 + Spring 26)

> **Quy tắc trình bày bài thi (Mandatory Exam Answer Format):**
> Trên bài thi CS 6210, TA chấm theo rubric rất nhanh. Luôn **nêu trực tiếp đáp án ngay đầu dòng đầu tiên và bắt đầu giải thích trên cùng dòng đó**, sau đó triển khai theo cấu trúc:
> ```text
> [Yes / No / True / False / Direct Answer] — because [lí do cốt lõi x]
> because [chi tiết cơ chế x]
> unless [ngoại lệ / điều kiện biên y]
> also [ý bổ sung / hệ quả z]
> ```

---

## PHẦN 1: TÓM TẮT CỐT LÕI CHƯƠNG 5, 6, 7 (CORE CHEAT SHEET)

### 1. Chương 5 (`L05`): Distributed Systems

#### 1.1. Lamport's Logical Clocks
* **Quan hệ "Happens-Before" ($\rightarrow$)**:
  1. **Cùng tiến trình (Sequential order)**: Nếu $A$ xảy ra trước $B$ trong cùng một tiến trình $\Rightarrow A \rightarrow B$.
  2. **Truyền thông điệp (Message passing)**: Nếu $A$ là sự kiện gửi (`send`) và $B$ là sự kiện nhận (`recv`) của cùng một thông điệp $\Rightarrow A \rightarrow B$.
  3. **Bắc cầu (Transitivity)**: Nếu $A \rightarrow B$ và $B \rightarrow C \Rightarrow A \rightarrow C$.
* **Sự kiện đồng thời (Concurrent events $A \parallel B$)**: Khi $A \not\rightarrow B$ và $B \not\rightarrow A$.
* **Chiều suy diễn (Cực kỳ hay bẫy T/F)**:
  * $A \rightarrow B \implies C(A) < C(B)$ (**ĐÚNG**).
  * $C(A) < C(B) \implies A \rightarrow B$ (**SAI** — vì $A$ và $B$ có thể là 2 sự kiện đồng thời $A \parallel B$).
* **Từ Partial Order $\rightarrow$ Total Order**: Phá vỡ hòa (break ties) một cách tất định bằng **ID của tiến trình/node** (ví dụ: nếu $C(a) = C(b)$, xếp $a < b$ nếu $\text{ProcessID}(A) < \text{ProcessID}(B)$).
* **Bẫy kinh điển trong đề thi**:
  * **Out-of-band communication (Alice gọi điện cho Bob)**: Lamport clock **KHÔNG** bắt được nhân quả ngoài hệ thống (gọi điện thoại, nhắn tin ngoài luồng), nên server có thể xử lý yêu cầu của Bob trước Alice.
  * **Game nhiều người chơi / Real-time ordering**: Lamport clock chỉ đảm bảo thứ tự nhân quả logic khi được sắp xếp lại, **KHÔNG** đảm bảo thứ tự thời gian thực hoặc thứ tự nhận tức thời tại mọi node nếu độ trễ mạng biến thiên (variable network latency).

#### 1.2. Lamport's Mutual Exclusion (ME) Algorithm
* **Giả định bắt buộc**: Mạng tin cậy (**eventual message delivery**) và **FIFO** giữa mọi cặp node.
* **2 điều kiện để Node $i$ (với yêu cầu $(T_i, i)$) vào vùng găng (Critical Section - CS)**:
  1. **Điều kiện 1 (Local Queue Head)**: Yêu cầu $(T_i, i)$ nằm ở **đầu hàng đợi cục bộ** $Q_i$ (nhỏ nhất theo thứ tự toàn phần).
  2. **Điều kiện 2 (Heard from All Peers)**: Node $i$ đã nhận được một thông điệp (có thể là `ACK` hoặc một thông điệp bất kỳ như `REQ`/`REL`) từ **TẤT CẢ $(N-1)$ node khác** có nhãn thời gian **lớn hơn $T_i$** ($> T_i$).
* **Độ phức tạp thông điệp (Message Complexity)**:
  * **Cơ bản**: $3(N - 1)$ thông điệp mỗi lần vào/ra CS: $(N-1)$ `REQ` + $(N-1)$ `ACK` + $(N-1)$ `REL`.
  * **Tối ưu hóa xuống $2(N - 1)$**:
    * Nếu $P_j$ đã gửi `REQ` có timestamp lớn hơn tới $P_i$, thông điệp `REQ` đó đã đóng vai trò xác nhận (do kênh FIFO), nên $P_j$ không cần gửi thêm `ACK` riêng.
    * Hoặc nếu $P_i$ đang có yêu cầu ưu tiên cao hơn $P_j$ ($T_i < T_j$), $P_i$ **trì hoãn (defer) gửi `ACK`** cho $P_j$ và gộp chung vào thông điệp **`REL`** khi $P_i$ thoát khỏi CS.
* **Bẫy đề thi**:
  * Khi Node A vào được CS, hàng đợi cục bộ trên tất cả các node có giống hệt nhau không? **False** — chỉ phần tử đầu hàng đợi (`head`) là chắc chắn giống nhau (đều là yêu cầu của A), phần đuôi hàng đợi có thể khác nhau do các thông điệp đang bay trên mạng.
  * Nếu chỉ gửi `REL` cho các node đang đợi khóa (có gửi `REQ`) mà bỏ qua các node không xin khóa thì sao? **Sai / Gây Deadlock** — vì trong thuật toán Lamport chuẩn, mọi node đều lưu yêu cầu $(T_A, A)$ vào hàng đợi cục bộ khi nhận `REQ`. Nếu node không xin khóa không nhận được `REL`, $(T_A, A)$ sẽ kẹt vĩnh viễn ở đầu hàng đợi của nó, khiến nó không bao giờ lấy được khóa trong tương lai!

#### 1.3. Limits of RPC Latency (Thekkath & Levy)
* **5 thành phần chi phí trên đường găng (Critical Path) của RPC truyền thống**:
  1. Call stub marshaling.
  2. **Context Switches ($t_x$)**:
     * **Shared-memory / LRPC trên SMP (Blocking)**: Tối thiểu **2** context switches (`Client -> Server` và `Server -> Client`). Có thể giảm xuống **0 (hoặc 1)** nếu client **spin (busy-wait)** trên cờ bộ nhớ dùng chung.
     * **Traditional Cross-Machine RPC (Unoptimized)**: **4** context switches:
       1. `C -> C1` (Client gửi xong bị block, OS chuyển sang tiến trình khác `C1` — *không nằm trên critical path nếu overlap với truyền mạng*).
       2. `S1 -> S` (Gói tin tới Server, OS chuyển từ `S1` sang tiến trình Server `S` — **nằm trên critical path**).
       3. `S -> S2` (Server gửi trả lời xong bị block, OS chuyển sang `S2` — *không nằm trên critical path nếu overlap với truyền mạng*).
       4. `C1 -> C` (Phản hồi về tới Client, OS chuyển từ `C1` về lại `C` — **nằm trên critical path**).
     * **Sau khi overlap với truyền mạng**: Chỉ còn **2** context switches trên đường găng (`S1 -> S` và `C1 -> C`).
  3. **Data Copying ($t_c$)**:
     * Trên 1 chiều truyền thống có **3 lần copy** (hoặc 2 lần copy phần mềm + 1 lần DMA):
       1. Client stub copy tham số từ stack vào RPC message buffer (`user -> user`).
       2. Kernel copy từ RPC buffer vào kernel mbuf (`user -> kernel`).
       3. NIC DMA copy từ kernel buffer xuống card mạng (`kernel -> NIC`).
     * Cả đi lẫn về (Round-trip): **$6\,t_c$** (nếu tính 3 copies/chiều theo slide bài giảng) hoặc **$4\,t_c$** (nếu chỉ tính copy phần mềm) hoặc **$8\,t_c$** (nếu tính cả DMA chiều nhận). Khi dùng shared kernel descriptor/buffer trực tiếp cho stub: giảm được **$2\,t_c$** (bỏ bước 1 ở cả 2 chiều).
  4. **Protocol Processing & Checksums**:
     * **LAN (100G/400G Ethernet)**: Dùng **Hardware-only checksum** (tận dụng tối đa phần cứng vì tỷ lệ lỗi bit trong LAN cực thấp).
     * **WAN (Wide Area Internet)**: Dùng **Hardware + Software Combined** (vì gói tin đi qua nhiều router trung gian ngoài tầm kiểm soát).
     * **Nếu đã chạy trên TCP**: Không cần thêm RPC checksum vì TCP đã đảm bảo độ tin cậy end-to-end.

#### 1.4. Active Networks (ANTS) & Ensemble/Nuprl
* **Cấu trúc Capsule trong ANTS**: `[ IP Header | ANTS Header (Version, TYPE = MD5 hash 128-bit, PREV = địa chỉ active node liền trước, Header Length) | Type-dependent Header | Payload ]`.
* **Quy trình xử lý khi Capsule tới Active Node C**:
  1. Đọc trường `TYPE` (mã băm MD5 của đoạn code xử lý) và tra cứu trong **Soft Store** (bộ nhớ đệm code cục bộ).
  2. Nếu **Hit**: Thực thi ngay đoạn code trong môi trường **Java Sandbox**.
  3. Nếu **Miss**: Gửi yêu cầu lấy code ngược về node ghi trong trường **`PREV`** (node chủ động liền trước).
  4. Khi nhận được code từ `PREV`: Tính mã băm MD5 của code nhận được, đối chiếu với `TYPE` (chống giả mạo code - **code spoofing**). Nếu khớp, lưu vào Soft Store, thực thi, cập nhật `PREV = C`, và chuyển tiếp.
* **Điều gì xảy ra nếu `PREV` cũng không còn code (do bị evict hoặc restart)?**
  * Node C **KHÔNG** hỏi đệ quy (recursively) ngược về các node trước đó nữa! Capsule bị **drop (hủy bỏ)** ngay tại C, và hệ thống dựa vào cơ chế truyền lại đầu-cuối (**end-to-end transport retransmission**) từ node nguồn.
* **Yêu cầu triển khai**: **KHÔNG** cần mọi router trên đường đi đều là Active Router (router thường chỉ đọc IP header và forward bình thường).

---

### 2. Chương 6 (`L06`): Distributed Objects and Middleware

#### 2.1. Spring OS (Microkernel Hướng Đối Tượng của Sun)
* **IDL & Stubs**: Định nghĩa giao diện bằng IDL $\rightarrow$ trình biên dịch IDL sinh ra: (1) Language-specific header files, (2) Client-side stub code, (3) Server-side stub code. **Client stub hoàn toàn độc lập với vị trí của Server (location-independent)** — dù Server ở cùng node hay khác node qua mạng LAN, Client stub **KHÔNG thay đổi**.
* **Subcontract**: Lớp ẩn bên dưới stub quyết định **cơ chế truyền thông và ngữ nghĩa gọi hàm** (singleton vs. replicated, local vs. remote, caching, marshaling/unmarshaling). Khi chuyển Server từ local sang remote node, chỉ có **Subcontract** (và Network Proxies) thay đổi.
* **Door vs. Network Proxy**:
  * **Door**: Một **software capability** do **Nucleus** (vi nhân) quản lý, đại diện cho một điểm nhập (entry point) vào một domain đích trên **cùng một node** để thực hiện lời gọi thủ tục bảo vệ xuyên không gian địa chỉ cực nhanh.
  * **Network Proxy**: Một user-level domain đặc biệt kết nối các Nucleus của **hai node khác nhau** qua mạng, mở rộng cơ chế Door xuyên mạng một cách trong suốt.
* **Luồng gọi xuyên mạng (Cross-Machine Invocation Flow: Node A $\rightarrow$ Node B)**:
  * **Setup (Subcontract)**: Tạo Proxy A trên Node A và Proxy B trên Node B; tạo một door trong Proxy B và xuất (export) thành một **network handle** gửi cho Proxy A; tạo một door cục bộ trong Proxy A cho Client trên Node A gọi.
  * **Runtime**: `Client (Node A) -> Nucleus A -> Network Proxy A -> [LAN Network Handle] -> Network Proxy B -> Nucleus B -> Server Domain (Node B)`.
* **Bảo mật & Phân quyền (Read/Write vs. Read-Only)**:
  * Đặt một **Front Object** (hoặc nhiều Front Objects) phía trước đối tượng thực.
  * Front Object kiểm tra **ACL (Access Control List)** và đăng ký các **Doors** khác nhau tương ứng với các mức quyền hạn khác nhau (ví dụ: Door 1 cho Read/Write, Door 2 cho Read-Only).
  * Trao cho mỗi Client một **Software Capability (Door Handle)** tương ứng trong bảng door của domain đó. Vì Door Handle là capability có thể truyền được (transferable), nếu Client A truyền capability Read/Write của mình cho Client B qua Nucleus/Subcontract, Client B sẽ nhận được door handle trỏ tới Door Read/Write và có quyền ghi!

#### 2.2. Java RMI (3 Layers)
1. **Stub/Skeleton Layer**: Giao diện giữa ứng dụng và hệ thống RMI; khởi tạo lời gọi, thực hiện tuần tự hóa/giải tuần tự hóa (**marshaling/unmarshaling** qua Java Object Serialization) các tham số và giá trị trả về thông qua luồng (stream) do tầng RRL cung cấp.
2. **Remote Reference Layer (RRL)**: Xử lý **ngữ nghĩa lời gọi (invocation semantics)** — ví dụ: gọi tới một server đơn lẻ (unicast/singleton) hay nhóm server nhân bản (multicast/replicated), và tự động kết nối lại (**reconnection semantics**) khi đứt kết nối mà không làm lộ lỗi lên tầng Stub.
3. **Transport Layer**: Quản lý địa chỉ đầu cuối (endpoint addressing), thiết lập/đóng kết nối (TCP/UDP), chọn kênh truyền, **theo dõi sự sống của kết nối (liveness monitoring)**, và định tuyến lời gọi vào bảng đối tượng từ xa trong không gian địa chỉ đích.

#### 2.3. Enterprise Java Beans (EJB)
* **Entity Beans (Fine-grained Data Access Objects)**:
  * *Ưu điểm (Pro)*: Tăng tính song song (**concurrency/parallelism**) khi truy cập CSDL (nhiều Entity Bean có thể truy vấn/cache các dòng dữ liệu khác nhau song song).
  * *Nhược điểm (Con)*: Làm lộ logic nghiệp vụ (business logic) ra ngoài Web Container/mạng WAN nếu gọi trực tiếp, và tăng chi phí quản lý/IPC của container.
* **Session Facade Pattern (Session Bean bọc ngoài Entity Beans)**:
  * *Ưu điểm (Pro)*: Kết hợp ưu điểm của cả hai — **bảo mật** (ẩn toàn bộ business logic bên trong EJB Container nội bộ, không lộ ra mạng WAN công cộng), **giảm số vòng lặp thông điệp qua WAN** (gom các lệnh gọi nhỏ lẻ thành 1 lời gọi), đồng thời vẫn **khai thác tính song song** của các Entity Beans bên dưới.
  * *Nhược điểm (Con)*: Thêm một tầng gián tiếp (indirection overhead) giữa Session Bean và Entity Bean (có thể giảm thiểu bằng *local interfaces* trong cùng JVM).

---

### 3. Chương 7 (`L07`): Distributed Subsystems

#### 3.1. Global Memory System (GMS)
* **Nguyên lý cơ bản**: Đổi I/O đĩa chậm lấy truyền trang qua mạng LAN nhanh.
  * Bộ nhớ mỗi node chia làm 2 phần biến đổi động: **Local Cache** (chứa trang đang hoạt động của node đó) và **Global Cache** (lưu trữ hộ các trang rỗi bị đẩy sang từ các node khác).
  * **Bất biến cực kỳ quan trọng (Clean Global Memory Invariant)**: Mọi trang nằm trong **Global Memory** của bất kỳ node nào đều **BẮT BUỘC PHẢI SẠCH (CLEAN)**. Nếu một trang ở Local Memory bị bẩn (dirty) và bị chọn đẩy sang Global Memory của node khác, node đó **phải ghi trang xuống đĩa trước** rồi mới gửi bản sạch đi!
* **3 Cấu trúc dữ liệu định vị trang (UID)**:
  1. **POD (Page Ownership Directory)**: Ánh xạ `UID -> Home Node` (node quản lý phân vùng GCD của UID đó). Được **nhân bản (replicated) trên TẤT CẢ các node**; chỉ thay đổi khi có node gia nhập/rời cụm. Tra cứu tốn **0 hop mạng** (tra cục bộ).
  2. **GCD (Global Cache Directory)**: Chia theo bảng băm (hash-partitioned) trên các node; ánh xạ `UID -> Danh sách các node đang giữ trang` và trạng thái trang (`Local-Private`, `Local-Shared`, `Global-Private`).
  3. **PFD (Page Frame Directory)**: Nằm cục bộ tại từng node; ánh xạ `UID -> Physical Frame` thực tế trong DRAM của node đó và trạng thái trang.
* **Số bước nhảy mạng (Network Hops) khi Page Fault tại Node A**:
  * **Trường hợp thường gặp (Non-shared page)**: `POD` và `GCD` của trang không chia sẻ thường được đặt cùng trên node khởi tạo, hoặc `A -> GCD Node -> Peer B (PFD) -> A`:
    * Nếu GCD nằm trên chính Node B đang giữ trang: **2 hops** (`A -> B -> A`).
    * Trường hợp xấu nhất (Worst-case khi GCD nằm ở một Node trung gian thứ 3): **3 hops** (`A -> GCD Node -> Node B (PFD) -> A`).
* **4 Trường hợp Page Fault tại Node P (khi trang nằm ở Node Q)**:
  * **Case 1 (Trang nằm trong Global Cache của Q)**: Đổi chỗ (swap) trang giữa P và Q — P nhận trang về Local Cache (giảm Global/Local của P), Q nhận trang bị thay thế từ P vào Global Cache.
  * **Case 2 (Local Cache của P đã đầy, trang lấy từ đĩa)**: P đẩy trang già nhất của mình sang Global Cache của một node Q có trang rỗi già nhất cụm.
  * **Case 3 (Trang lấy từ đĩa, trên P đang có trang thuộc Global Cache)**: P tự thu hồi khung trang trong Global Cache của chính nó để dùng cho Local Cache (Local của P tăng, Global của P giảm).
  * **Case 4 (Trang đang là `Local-Shared` trong Local Cache của Q)**: **Sao chép (copy)** trang từ Q sang Local Cache của P (cả P và Q đều đánh dấu `Local-Shared`; GCD ghi nhận `{Q, P}`). P phải đẩy 1 trang cũ sang Global Cache của một node R.
* **Lưu ý khi Evict một trang `Local-Shared` (Spring 26 Q1b)**:
  * Nếu Node A muốn đẩy (evict) một trang đang ở trạng thái **`Local-Shared`** (tức là đã có bản sao hợp lệ trên Node B): Node A **KHÔNG** gửi trang đó sang Global Memory của Node C mà **hủy (discard/drop) trực tiếp tại chỗ**, vì cụm đã có bản sao ở Node B! Sau đó GCD cập nhật còn `{B}`, PFD tại A thành `Non-existent`, PFD tại B thành `Local-Private`.
* **Thuật toán Quản lý Tuổi Toàn Cục (Global Age Management / Epoch $T, M$)**:
  * Ở đầu mỗi chu kỳ (epoch), **Initiator Node** thu thập thông tin tuổi từ các node và tính:
    * **`minAge`**: Tuổi của trang thứ $M$ già nhất trong toàn cụm.
    * **`weights` ($w_i$)**: Tỷ lệ số trang nằm trong nhóm $M$ trang già nhất cụm đang thuộc về Node $i$.
  * Khi một node cần đẩy trang có tuổi `age`:
    * Nếu **`age < minAge`** (trang còn trẻ/đang hoạt động so với cụm): **Gửi trang sang một node khác** theo phân phối xác suất `weights` (node có **weight cao nhất** có xác suất nhận trang cao nhất).
    * Nếu **`age >= minAge`** (trang đã quá già): **Hủy bỏ (discard) trực tiếp tại chỗ** (sau khi ghi xuống đĩa nếu dirty).

#### 3.2. Distributed Shared Memory (DSM) & TreadMarks
* **Tại sao Sequential Consistency (SC) với kích thước trang (Page-level Granularity) lại chậm dù không có Data Race?**
  1. **SC không phân biệt được truy cập đồng bộ hóa (synchronization) và truy cập dữ liệu thường (ordinary read/write)** $\Rightarrow$ mọi lệnh đọc/ghi đều phải thực hiện truyền thông duy trì tính nhất quán tức thời trên toàn cụm.
  2. **False Sharing (Ping-pong trang)**: Nhiều biến độc lập nằm chung trên một trang ảo lớn khiến trang bị vô hiệu hóa/chuyển qua lại liên tục giữa các node.
* **Eager Release Consistency (ERC) vs. Lazy Release Consistency (LRC)**:
  * **ERC**: Ngay khi `Release(L)`, node vừa sửa trang sẽ **broadcast/multicast thông điệp vô hiệu hóa (invalidation)** hoặc cập nhật tới **TẤT CẢ** các node đang cache trang đó.
  * **LRC (TreadMarks)**: Khi `Release(L)`, **KHÔNG gửi bất kỳ thông điệp nào**! Chỉ khi một node khác gọi `Acquire(L)`, thông báo vô hiệu hóa (**write notices / invalidation notices**) của các trang bị sửa theo chuỗi nhân quả của khóa `L` mới được **đính kèm (piggybacked)** trên thông điệp trao khóa `L`.
* **Cơ chế Multiple-Writer & Twin/Diff trong TreadMarks**:
  1. **Tại `Acquire(L)`**: Nhận các write notices được đính kèm theo khóa `L` (chỉ các thay đổi có quan hệ nhân quả $\rightarrow$ với `L`) và **vô hiệu hóa (invalidate)** các trang tương ứng nếu chúng đang có mặt tại node đó.
  2. **Tại lần ĐỌC đầu tiên (`Read X`) vào trang không hợp lệ**: Xảy ra page fault $\rightarrow$ lấy bản gốc (clean/pristine copy) từ Owner (nếu chưa có trang) + **chỉ xin các bản `diff` trên chuỗi nhân quả của khóa `L`** $\rightarrow$ áp dụng các `diff` theo đúng thứ tự nhân quả $\rightarrow$ ánh xạ trang ở chế độ **Read-Only** (**KHÔNG tạo twin** khi chỉ đọc!).
  3. **Tại lần GHI đầu tiên (`Write Y`) vào trang trong một interval**:
     * Nếu trang chưa hợp lệ: Lấy bản gốc + áp dụng các `diff` nhân quả như bước đọc.
     * Bẫy bảo vệ ghi (OS write-protection fault) kích hoạt $\rightarrow$ DSM cấp phát và tạo một bản sao y hệt của trang trong vùng nhớ hệ thống gọi là **`twin`** $\rightarrow$ mở quyền trang thành **Read-Write** để các lần ghi tiếp theo trong cùng vùng găng chạy ở tốc độ phần cứng.
  4. **Tại `Release(L)`**: So sánh từng từ (word-by-word) giữa trang đã sửa và bản `twin` của nó để tạo ra bản **`diff`** (nén theo run-length encoding) $\rightarrow$ **khóa quyền ghi của trang về lại Read-Only** $\rightarrow$ **hủy bỏ (discard) bản `twin`**.
* **Bẫy đề thi TreadMarks**:
  * TreadMarks có dùng broadcast khi page fault không? **False** — mỗi trang có một Owner/Manager xác định trước và các write notices đã ghi rõ node nào giữ diff nào, nên chỉ dùng **point-to-point messages**.
  * Nếu 2 node cùng ghi đè lên **cùng một word** trên một trang mà không có đồng bộ hóa (data race), TreadMarks có gộp (merge) tất định được không? **False** — vì thiếu đồng bộ hóa nên hai khoảng thời gian (intervals) là đồng thời (concurrent), không có thứ tự nhân quả để xếp thứ tự các diff.

#### 3.3. Distributed File Systems (DFS) — xFS (Serverless Network File System)
* **Kiến trúc "Serverless" & 4 thành phần của xFS**: Clients, Metadata Managers, Storage Servers, Cleaners (mọi máy trạm trong cụm đều có thể đảm nhiệm cả 4 vai trò).
* **Cấu trúc Metadata phân tán**:
  * **`Manager Map` (`mmap`)**: Được nhân bản tại mọi client/node; ánh xạ `File Index Number -> Metadata Manager Node`.
  * **Tại Metadata Manager**: Giữ **Cache Consistency State** (ai đang cache block nào, ai giữ quyền ghi) + **`imap`** (ánh xạ `Index Number -> Disk Log Address của Index Node`) + bộ nhớ đệm của các **`Index Nodes`** (chứa địa chỉ đĩa của các data blocks).
  * **Tại sao xFS KHÔNG cache Index Nodes tại Client? (Spring 26 DFS Q3)**:
    * *Lợi ích nếu cache*: Bỏ được 2 bước nhảy mạng `Client -> Manager` và `Manager -> Storage Server` khi đọc đĩa.
    * *3 Lý do xFS không làm vậy*: (1) **Mất Cooperative Caching** (client đọc thẳng từ đĩa mà không biết có peer client khác đang giữ block đó trong RAM nhanh hơn nhiều); (2) **Vẫn phải gửi thông điệp báo cho Manager** biết mình đang cache block để Manager quản lý tính nhất quán; (3) **Tăng độ phức tạp** vì phải duy trì thêm giao thức nhất quán cho chính các Index Nodes khi file bị ghi.
* **Chính sách "First Writer" (Đặt Manager của file tại chính Client tạo ra file đó — Spring 26 DFS Q1)**:
  * **Tại sao giúp lệnh GHI (Writes) nhiều hơn lệnh ĐỌC (Reads)?**
    * Với **Writes**: Chia sẻ ghi (write-sharing) rất hiếm (96% lần ghi đè/xóa là do cùng một người ghi trước đó), nên đặt Manager cùng chỗ với người tạo file giúp **90% yêu cầu xin quyền ghi (write-ownership)** và **90% thông báo đẩy đĩa (disk-flush notifications)** diễn ra **cục bộ (0 network hops)**.
    * Với **Reads**: Hơn 75% lượt đọc đã trúng ngay **Local Cache** tại client mà không cần hỏi Manager; còn khi đọc bị miss thì thường phải lấy từ Storage Server ở xa hoặc từ RAM của một peer khác (vốn vẫn tốn các bước nhảy mạng từ xa).
* **Tại sao xFS phải dùng Stripe Groups thay vì trải (stripe) qua TẤT CẢ hàng trăm Storage Servers? (Spring 26 DFS Q2)**:
  1. **Trở lại bài toán Small-Write / Tốn RAM đệm**: Trải một segment qua hàng trăm server buộc client hoặc phải gửi các mảnh (fragments) cực nhỏ gây lãng phí băng thông/seek đĩa, hoặc phải đệm một lượng dữ liệu khổng lồ trong RAM trước khi ghi.
  2. **Mất tính song song (Loss of Parallelism)**: Mọi client khi ghi đều làm bận tất cả các đĩa cùng lúc thay vì cho phép các nhóm client ghi song song độc lập lên các nhóm đĩa khác nhau.
  3. **Chi phí dọn rác (Log Cleaning Overhead) cực lớn**: Các segment quá lớn luôn ở trạng thái "vơi một phần" (partially full), buộc Cleaner phải đọc toàn bộ segment khổng lồ qua mạng.
  4. **Giảm độ sẵn sàng (Reduced Availability / Fault Tolerance)**: Với RAID-4/5 một đĩa chẵn lẻ (single parity), nếu trải qua toàn bộ $N$ server thì chỉ cần **2 server bất kỳ trong cụm chết cùng lúc** là mất dữ liệu toàn hệ thống; trong khi chia thành nhiều Stripe Groups độc lập giúp chịu được nhiều lỗi server miễn không nằm cùng một group.
* **Thuật toán Phục hồi khi Metadata Manager bị sập (xFS Manager Crash Recovery — Fall 25 DFS Q1)**:
  1. **Tái tạo `Manager Map` bằng Consensus**: Các node đang hoạt động chạy thuật toán đồng thuận phân tán (distributed consensus) để bầu và gán lại vùng không gian Index Number của manager bị chết cho một manager mới.
  2. **Phục hồi Cache Consistency Metadata bằng Server-Driven Polling**: Manager mới chủ động hỏi (poll) tất cả các clients xem họ đang cache những block nào hoặc đang giữ quyền ghi (write token) của file nào thuộc phân vùng của nó.
  3. **Phục hồi Disk Location Metadata (`imap` & Index Nodes) bằng Checkpoint + Roll-Forward**: Manager mới đọc bản checkpoint mới nhất của **`ifile`** từ log trên các Storage Servers, sau đó **cuộn tiến (roll forward)** bằng cách đọc các đoạn log deltas do các clients ghi sau thời điểm checkpoint.

---
---

## PHẦN 2: GIẢI CHI TIẾT ĐỀ THI FALL 2025 — TEST 2 (THEO FORMAT CHUẨN)

> Nguồn đề & rubric gốc: [Fall 25 Test 2.pdf](file:///D:/OMSCS/AOS/CS6210-Advanced-Operating-Systems-Notes/Previous%20Tests/Fall%2025/Test%202.pdf)

---

### Section 1: Distributed Systems — Lamport's Logical Clock [8 pts]

#### Q1 [3 pts]
**Đề bài**: Game bắn súng nhiều người chơi ngang hàng (peer-to-peer), mỗi hành động `<playerID, action>` và hành động phản đòn (counteraction) được quảng bá cho mọi người chơi. Mạng không mất gói, độ trễ biến thiên, đảm bảo FIFO giữa từng cặp người chơi. Sử dụng Lamport's logical clocks có đảm bảo rằng một hành động phản đòn (counteraction) luôn xuất hiện sau hành động gốc theo đúng thứ tự sinh ra trên máy của mọi người chơi không?

**Exam Answer**:
```text
No — because Lamport's logical clocks only preserve causal happens-before ordering (A -> B implies C(A) < C(B)) rather than real-time total generation order or immediate network delivery order across all third-party nodes.
because if Player A broadcasts a move and Player B receives it and immediately broadcasts a counteraction, variable network latency along the direct path A -> C versus A -> B -> C can cause Player C to receive Player B's counteraction before Player A's original move (and independent moves remain concurrent with arbitrary clock ties).
unless the system buffers incoming messages and uses a total-order broadcast / vector clock / central coordinator protocol before rendering actions to players.
also because C(A) < C(B) does not imply A happened before B for concurrent moves generated by different players without causal communication.
```

---

#### Q2 [5 pts]
**Đề bài**: Cho sơ đồ thời gian không gian của 3 tiến trình `A`, `B`, `C` (các sự kiện `a1..a6`, `b1..b4`, `c1..c4` và các mũi tên thông điệp: `a1 -> b2`, `b1 -> c1`, `c3 -> b2`, `b2 -> a3`, `b3 -> a4`, `b3 -> c2`, `a5 -> b4`).
* **2a [1 pt]**: Did event `b3` happen before `a6`? Justify.
* **2b [1 pt]**: Did event `c3` happen before `a4`? Justify.
* **2c [1 pt]**: Did event `a2` happen before `c1`? Justify.
* **2d [2 pts]**: How can the partial ordering of events in this system be extended to a total ordering?

**Exam Answers**:

* **2a [1 pt]**:
```text
Yes — because there is a directed causal path from b3 to a6 (b3 -> a4 -> a5 -> a6).
because b3 is the message send event received at a4 (b3 -> a4), and a4 precedes a5 and a6 sequentially within Process A (a4 -> a5 -> a6).
also by transitivity of the happens-before relation, b3 -> a6.
```

* **2b [1 pt]**:
```text
Yes — because there is a directed causal path from c3 to a4 (c3 -> b2 -> a3 -> a4).
because c3 sends a message received at b2 (c3 -> b2), b2 sends a message received at a3 (b2 -> a3), and a3 precedes a4 in Process A (a3 -> a4).
also by transitivity of the happens-before relation, c3 -> a4.
```

* **2c [1 pt]**:
```text
No — because there is no causal happens-before path connecting a2 and c1 in either direction (a2 and c1 are concurrent events).
because the only message into c1 originates from b1 (b1 -> c1), and no message sent at or after a2 ever reaches Process B at or before b1 or Process C at or before c1.
unless a causal chain of messages and sequential events links two events, Lamport's happens-before relation does not order them.
```

* **2d [2 pts]**:
```text
By breaking timestamp ties deterministically using a global arbitrary tie-breaker such as unique Process IDs — because events on different processes can have identical logical clock values when they are concurrent.
because assigning a fixed total order over process identifiers (e.g., Process A < Process B < Process C) allows ordering any two events X (on Pi) and Y (on Pj) whenever C(X) == C(Y).
also this guarantees a unique, consistent total ordering across all nodes without requiring any extra communication.
```

---

### Section 2: Lamport's ME Algorithm [6 pts]

#### Q1 [6 pts]
**Đề bài**: Cụm 3 node `P1, P2, P3`. `P1` yêu cầu khóa tại `TS = 4` (`P1, 4`), `P2` yêu cầu khóa tại `TS = 3` (`P2, 3`). Đường thời gian `T1..T7`:
* Tại `T1`: `P1` nhận `(P2, 3)`.
* Tại `T2`: `P2` nhận `(P1, 4)`.
* Tại `T3`: `P3` nhận `(P2, 3)` và gửi `ACK` cho `P2`.
* Tại `T4`: `P2` nhận `ACK` từ `P3`.
* Tại `T5`: `P3` nhận `(P1, 4)` và gửi `ACK` cho `P1`.
* Tại `T6`: `P1` nhận `ACK` từ `P3`.
* Tại `T7`: `P2` gửi `Release` cho `P1` và `P3`.

* **1a [2 pts]**: Which node gets to access the shared resource first, and at which time (`T1-T7`)? Justify.
* **1b [2 pts]**: What is a potential problem if the "eventual message delivery" assumption is violated? Explain.
* **1c [2 pts]**: What is a simple method to prevent the algorithm from failing due to message loss?

**Exam Answers**:

* **1a [2 pts]**:
```text
P2 at time T4 — because P2's request has the earliest logical timestamp (3 < 4) and satisfies both of Lamport's mutual exclusion conditions at T4.
because Condition 1 is satisfied since (P2, 3) is at the head of P2's local request queue, and Condition 2 is satisfied at T4 when P2 has heard from all other nodes with timestamps greater than 3 (it received P1's request with TS=4 > 3 at T2, and received P3's ACK at T4).
unless P2 has heard from both P1 and P3, it cannot safely enter the critical section before T4.
```

* **1b [2 pts]**:
```text
Mutual exclusion violation (safety failure) or Deadlock/Starvation (liveness failure) — because lost messages leave nodes with inconsistent views of active lock requests or completions.
because if a request message from P1 to P2 is lost, P2 may remain unaware of P1's earlier request and enter the critical section concurrently with P1; conversely, if an ACK or release message is lost, the waiting node will block forever waiting for a message that never arrives.
also lost release messages leave stale requests permanently stuck at the head of peers' local queues.
```

* **1c [2 pts]**:
```text
Use a timeout-and-retransmission protocol with acknowledgments (ACKs) and sequence numbers — because reliable point-to-point transport detects and recovers lost messages automatically.
because if a sender does not receive an ACK for a message within a timeout window, it retransmits the message, while sequence numbers allow receivers to detect missing packets and ignore duplicate retransmissions.
```

---

### Section 3: RPC Latency Limits [6 pts]

#### Q1 [6 pts]
**Đề bài**: Hệ thống RPC trên mạng 100 Gbps Ethernet. Hai phương án: `Hardware-only` (chỉ dùng checksum phần cứng trên NIC) và `Hardware Software Combined` (dùng cả NIC offload + RPC checksum nhẹ ở tầng ứng dụng).
* **1a [2 pts]**: Mạng nâng cấp lên 400 Gbps Ethernet trong mạng **LAN**. Chọn phương án nào?
* **1b [2 pts]**: Giao tiếp liên trung tâm dữ liệu qua mạng **Wide Area Internet (WAN)**. Chọn phương án nào?
* **1c [2 pts]**: Đồng nghiệp nói: *"Modern NICs and TCP already guarantee reliability. Adding another checksum layer in RPC just increases latency."* Bạn đồng ý (`Agree`) hay phản đối (`Disagree`)?

**Exam Answers**:

* **1a [2 pts]**:
```text
Hardware-only checksums — because a local area network (LAN) is highly reliable with extremely low bit-error rates, so the system should leverage hardware capabilities and avoid redundant software overhead.
because computing an additional software checksum in the RPC layer over a 400 Gbps LAN adds unnecessary CPU and latency overhead on the critical path without meaningful reliability gains.
```

* **1b [2 pts]**:
```text
Hardware Software Combined — because wide-area network (WAN) traffic traverses many intermediate routers and heterogeneous links where packet corruption is significantly more likely.
because link-layer NIC CRC/checksums only protect individual hops and cannot catch memory or routing corruption inside intermediate WAN switches/routers, making an end-to-end software RPC checksum valuable.
```

* **1c [2 pts]**:
```text
Agree — because the TCP transport protocol already guarantees end-to-end reliability, making an additional checksum in the RPC layer redundant.
because TCP inherently includes end-to-end checksums, sequence numbers, acknowledgments, and automatic retransmissions to detect and recover from corrupted or lost segments.
also adding another checksum at the RPC layer duplicates work already done by TCP and only increases RPC latency.
```

---

### Section 4: Active Networks [4 pts]

#### Q1 [4 pts]
**Đề bài**: Trong Active Networks (ANTS), các node lấy code từ node trước (`PREV`) và lưu vào soft store. Nêu **2 rủi ro bảo mật** tiềm ẩn của thiết kế này và giải thích cách ANTS giảm thiểu (mitigate) từng rủi ro.

**Exam Answer**:
```text
Code Spoofing and Soft-Store Corruption/Interference — because dynamically fetching executable code and sharing router memory across untrusted flows exposes active routers to malicious code injection and resource abuse.
because (1) for Code Spoofing, a malicious actor could supply counterfeit code for a protocol TYPE, which ANTS mitigates by making the capsule's TYPE field a cryptographically strong fingerprint (e.g., 128-bit MD5 hash) of the actual code so the receiving router verifies the hash before caching or executing it; and (2) for Soft-Store Integrity/Resource Exhaustion, one capsule flow could corrupt another flow's cached code/breadcrumbs or exhaust router memory, which ANTS mitigates by executing capsules inside a restricted Java sandbox with a narrow API and bounding soft-store allocations with TTL/LRU eviction.
```

---

### Section 5: Distributed Objects and Middleware [16 pts]

#### Spring OS — Q1 [3 pts]
**Đề bài**: Hai nhóm phát triển dịch vụ trên Spring OS bằng C và C++. Mỗi nhóm phải tạo ra những thành phần (artifacts) nào và bằng cách nào để dịch vụ của họ gọi được nhau?

**Exam Answer**:
```text
IDL interface definitions, language-specific header files, client-side stubs, and server-side stubs — because Spring OS achieves language interoperability through its Interface Definition Language (IDL) toolchain.
because each team first defines its service interfaces in Spring's language-neutral IDL, and then runs the IDL compiler to automatically generate: (1) language-specific C or C++ header files, (2) client-side stub code that marshals outgoing calls, and (3) server-side stub code that unmarshals incoming calls and invokes the target implementation.
```

#### Spring OS — Q2 [8 pts]
**Đề bài**: Client trên Node A gọi `invoice.submit()` tới Invoice Service trên Node B qua mạng LAN.
* **2a [2 pts]**: `door` là gì và `network proxy` là gì trong Spring OS?
* **2b [3 pts]**: Thiết lập ban đầu (setup) bằng `subcontracts` diễn ra như thế nào?
* **2c [3 pts]**: Luồng thực thi lúc chạy (runtime flow) từ Client tới Invoice Service diễn ra như thế nào?

**Exam Answers**:

* **2a [2 pts]**:
```text
A door is a kernel-protected software capability for fast cross-domain invocation on a single node, while a network proxy is a user-level domain that extends door invocations across different network nodes — because Spring separates local cross-address-space calls from network transport.
because a door represents a validated entry point into a target domain managed by the local Nucleus (microkernel), whereas network proxies connect the nuclei of remote machines and transparently forward local door calls over the network.
```

* **2b [3 pts]**:
```text
Subcontracts instantiate Network Proxies and bind local doors to remote network handles on both nodes — because cross-machine communication in Spring is configured transparently beneath the stubs by subcontracts.
because during setup, the subcontract: (1) instantiates Network Proxy A on Node A and Network Proxy B on Node B speaking a compatible network protocol, (2) creates a door entry point in Network Proxy B on Node B (representing the Invoice Service) and exports it as a network handle to Network Proxy A, and (3) creates a local door entry point in Network Proxy A on Node A for the client's Nucleus to invoke.
```

* **2c [3 pts]**:
```text
Client (Node A) -> Nucleus A -> Network Proxy A -> Network Proxy B -> Nucleus B -> Invoice Service (Node B) — because cross-node calls combine local Nucleus door invocations at each endpoint with proxy-to-proxy network forwarding.
because (1) the Client invokes its local door handle for invoice.submit(), trapping into Nucleus A, which transfers control to the door entry point of Network Proxy A; (2) Network Proxy A marshals the request and transmits it over the LAN via the network handle to Network Proxy B on Node B; and (3) Network Proxy B unmarshals the request and performs a local door invocation via Nucleus B into the door entry point of the Invoice Service domain.
```

---

#### EJB — Q1 [2 pts]
**Đề bài**: Đánh giá kiến trúc EJB định phơi bày trực tiếp các Entity Beans mịn (fine-grained) ra mạng WAN công cộng và đề xuất dùng **Session Facade** (Session Bean bọc ngoài Entity Beans). Tại sao cần dùng Session Facade trước khi đưa lên production?

**Exam Answer**:
```text
To protect business logic security and improve WAN performance while preserving database concurrency — because exposing fine-grained Entity Beans directly over an untrusted public WAN leaks internal business logic and incurs high network round-trip latency.
because (1) a Session Facade encapsulates and hides sensitive business logic inside the internal corporate EJB container rather than exposing it to external clients over the WAN, and (2) it collapses multiple fine-grained WAN calls into a single coarse-grained request while still issuing parallel requests to underlying Entity Beans locally.
```

---

#### Java RMI — Q1 [3 pts]
**Đề bài**: Giải thích ngắn gọn vai trò của 3 tầng trong Java RMI: `Stub/Skeleton Layer`, `Remote Reference Layer`, và `Transport Layer`.

**Exam Answer**:
```text
Stub/Skeleton handles marshaling, Remote Reference Layer handles invocation semantics, and Transport Layer handles network connections — because Java RMI modularizes remote method invocation into three clean layers.
because:
1. Stub/Skeleton Layer: Acts as the proxy interface between the application and the RMI system, initiating remote calls and marshaling/unmarshaling (serializing/deserializing) method arguments and return values over streams provided by the RRL.
2. Remote Reference Layer (RRL): Manages invocation semantics (such as unicast/singleton vs. multicast/replicated servers and transparent reconnection) and provides marshaling streams to the stub/skeleton while passing data to the transport layer.
3. Transport Layer: Manages endpoint addressing, connection setup/teardown, channel selection (e.g., TCP/UDP), connection liveness monitoring, and dispatching incoming calls to remote objects residing in the local address space.
```

---

### Section 6: Distributed Subsystems [24 pts]

#### GMS — Q1 [5 pts]
**Đề bài**: Cụm 10 node `A..J` chạy GMS. Tiến trình `x` trên Node `A` đọc chia sẻ (read-shares) một trang hiện chỉ đang được sử dụng tích cực bởi tiến trình `y` trên Node `B`. Không có ràng buộc đồng vị trí (no co-location constraints).
* **1A**: Truy cập `POD` ở node nào?
* **1B**: Truy cập `GCD` ở node nào?
* **1C**: Truy cập `PFD` ở node nào?
* **1D**: Số bước nhảy mạng (network hops) tối thiểu trong trường hợp xấu nhất (worst case)?
* **1E**: Một tiến trình mới `z` trên Node `A` cũng đọc chia sẻ cùng trang đó. Lần này `PFD` của node nào được truy cập?

**Exam Answers**:

* **1A (POD)**:
```text
Node A — because the Page Ownership Directory (POD) is replicated on every node in the cluster and is always looked up locally first.
```

* **1B (GCD)**:
```text
The node that owns the GCD hash partition for the page's UID (any of the 10 nodes A-J mapped by POD) — because the GCD is distributed across the cluster by hashing the page UID, and POD identifies that partition owner.
```

* **1C (PFD)**:
```text
Node B — because Node B is the node currently holding the target page in its physical DRAM frames, and its local Page Frame Directory maps the UID to the physical frame.
```

* **1D (Minimum network hops in the worst case)**:
```text
3 network hops (Node A -> GCD Owner Node -> Node B -> Node A) — because in the worst case with no co-location, the GCD owner is on a third node distinct from both A and B.
because Hop 1 sends the fault request from Node A to the remote GCD node, Hop 2 forwards the request from the GCD node to Node B (which holds the PFD and physical page), and Hop 3 transfers the page copy directly from Node B back to Node A.
```

* **1E (Which PFD is accessed when process `z` on Node `A` reads the same page)**:
```text
Node A — because after process x's fault completes, a shared copy of the page is already cached in Node A's local DRAM (Unified Buffer Cache / local cache), so Node A's own PFD satisfies process z's read locally with 0 network hops.
```

---

#### GMS — Q2 [5 pts]
**Đề bài**: Cụm 4 node `A, B, C, D` chạy GMS. Node A cần đẩy 2 trang: `P1 (age = 15 minutes)` và `P2 (age = 28 minutes)`. Đầu epoch hiện tại: `minAge = 25 minutes`, trọng số `weights = {A: 0.2, B: 0.3, C: 0.1, D: 0.4}`.
* **2a [2 pts]**: Quyết định của GMS cho `P1` và `P2` là gì? Giải thích.
* **2b [1 pt]**: Nếu một trang được gửi sang node khác, node nào có khả năng nhận cao nhất và tại sao?
* **2c [2 pts]**: Công việc bổ sung nào phải thực hiện khi trang bị đẩy đi (evicted) và tại sao?

**Exam Answers**:

* **2a [2 pts]**:
```text
Send P1 to a peer node's global memory and discard P2 locally — because GMS compares each evicted page's age against the cluster-wide threshold minAge = 25 minutes.
because P1's age (15 < 25) is younger than minAge, indicating it is still relatively active and should be preserved in cluster global memory, whereas P2's age (28 >= 25) is older than minAge, meaning it is among the oldest pages in the cluster and can be discarded locally.
```

* **2b [1 pt]**:
```text
Node D — because Node D has the highest weight (0.4) among all nodes in the current epoch, indicating it holds the largest proportion of the M oldest idle pages in the cluster.
```

* **2c [2 pts]**:
```text
Write the page to disk first if it is dirty — because GMS enforces the strict invariant that all pages stored in any peer node's global memory cache must be clean.
because flushing dirty pages to disk before eviction or demotion to global memory ensures that if a peer node hosting global pages crashes, no uncommitted data is ever lost.
```

---

#### DSM — Q1 [2 pts]
**Đề bài**: Một ứng dụng DSM đa tiến trình không có data race, dùng mutex locks đúng chuẩn, chạy trên mô hình bộ nhớ **Sequential Consistency (SC)** với độ mịn cấp trang (page-level granularity), nhưng hiệu năng rất tệ. Giải thích tại sao.

**Exam Answer**:
```text
False sharing at page granularity and eager per-access coherence overhead under Sequential Consistency — because SC cannot distinguish synchronization accesses from ordinary data accesses and enforces coherence on every memory operation.
because (1) under SC, every single ordinary read or write must globally order and complete coherence actions (invalidations/updates) across the cluster before proceeding instead of deferring coherence to lock acquire/release boundaries, and (2) page-level granularity causes false sharing (page ping-ponging across nodes) whenever different nodes access independent variables that happen to reside on the same virtual page.
```

---

#### DSM — Q2 [8 pts]
**Đề bài**: Hệ thống DSM **TreadMarks**. Xét đoạn code trên Node `N1`:
`Lock(L1); Write to Page X; Unlock(L1);`
Giả sử `X` là trang duy nhất bị sửa bởi các node giữ `L1` trước đó; `X` chưa có mặt tại `N1`; các bản diff `Xd2, Xd3, Xd4` đang nằm tại `N2, N3, N4`. Chuỗi nhân quả đồng bộ hóa (synchronization causality) của `L1` tới thời điểm hiện tại là: `N3 -> N2 -> N1`.
* **2a [2 pts]**: Hành động nào diễn ra tại thời điểm lấy khóa `Lock(L1)` và tại sao?
* **2b [2 pts]**: Hành động bổ sung nào diễn ra tại `N1` trước khi vùng găng hoàn tất?
* **2c [2 pts]**: Hành động nào diễn ra khi thoát vùng găng `Unlock(L1)` tại `N1`?
* **2d [2 pts]**: Sau đó, `N1` thực thi `Lock(L1); Read page X; Unlock(L1);` và không có node nào khác lấy `L1` ở giữa. Hệ thống DSM tại `N1` thực hiện hành động gì?

**Exam Answers**:

* **2a [2 pts]**:
```text
N1 receives piggybacked write notices for Page X (from N3 and N2) and marks Page X invalid at N1 — because Lazy Release Consistency piggybacks invalidation notices on the lock grant message from the previous holder (N2).
because X is the only page modified along L1's causal synchronization chain (N3 -> N2 -> N1), so N1 records that X has been modified by N3 and N2 (even though X is not currently present in N1's local memory).
```

* **2b [2 pts]**:
```text
N1 faults on Write to Page X, fetches the clean copy of X from its owner, fetches and applies diffs Xd3 then Xd2 in causal order, creates a twin of X, and marks X read-write — because X is missing locally and modified along L1's causal chain.
because upon the write protection/page fault, N1 must: (1) obtain the pristine copy of X from X's owner node, (2) request and apply only the causally preceding diffs Xd3 (from N3) followed by Xd2 (from N2) in that order (ignoring Xd4 on N4 because N4 is not in L1's causal chain), and (3) allocate a twin (an identical copy of X in system space) and unprotect X to read-write before performing the write.
```

* **2c [2 pts]**:
```text
N1 creates a run-length encoded diff Xd1, write-protects Page X (marks it read-only), and discards the twin — because TreadMarks captures modifications at interval/release boundaries and prepares the page to trap future writes.
because comparing the modified Page X word-by-word against its twin produces diff Xd1 stored at N1, marking X read-only ensures any subsequent write triggers an OS protection fault to create a new twin, and freeing the twin reclaims system memory.
```

* **2d [2 pts]**:
```text
No DSM action is taken (0 network messages and 0 faults) — because N1 was the last holder of L1 and already caches the valid, up-to-date Page X mapped as read-only in its local memory.
because no other node acquired L1 or modified X in the interim, so Lock(L1) succeeds locally with no invalidations, and reading X succeeds at hardware speed without triggering a protection fault since X is already mapped read-only.
```

---

#### DFS — Q1 [2 pts]
**Đề bài**: Trình bày thuật toán mức cao để phục hồi khi một **Metadata Manager** trong **xFS** bị sập (crash).

**Exam Answer**:
```text
Consensus-based Manager Map regeneration, Server-Driven Polling for cache state, and Checkpoint plus Roll-Forward recovery for disk metadata — because xFS distributes metadata across managers, client caches, and storage server logs.
because:
1. Manager Map Recovery: Active nodes run a distributed consensus algorithm to regenerate the Manager Map (mmap) and reassign the crashed manager's index number space to a surviving manager.
2. Cache Consistency Metadata Recovery: The new manager performs server-driven polling, querying all clients to reconstruct which file blocks they currently cache and which client holds write ownership.
3. Disk Location Metadata Recovery (imap and index nodes): The new manager reads the latest manager checkpoint (ifile) from the storage servers and rolls forward by scanning log deltas written by clients after that checkpoint.
```

---

#### DFS — Q2 [2 pts]
**Đề bài**: So với thiết kế quản lý metadata phân tán của xFS, nêu **1 Ưu điểm (Pro)** và **1 Nhược điểm (Con)** của việc dùng một **Master duy nhất** cho metadata như trong Google File System (GFS).

**Exam Answer**:
```text
Pro: Design simplicity and fast deterministic lookup; Con: Centralized scalability bottleneck and single point of failure — because centralizing metadata on one Master eliminates distributed coordination overhead at the cost of scalability.
because:
- Pro: Every client knows the single Master to contact for metadata without maintaining or running consensus on a distributed Manager Map (mmap), greatly simplifying management and recovery.
- Con: The single Master can become a CPU/memory/network bottleneck under metadata-intensive or small-file workloads and acts as a single point of failure unless shadow/failover masters are maintained.
```

---

### Section 7: HIDDEN - Potpourri [16 pts]

#### Q1 [2 pts]
**Đề bài**: **True/False**: Để đảm bảo tính đúng đắn của ứng dụng đa tiến trình dùng DSM trên cụm LAN, một trang ảo (virtual page) nhất định phải được ánh xạ tới cùng một số hiệu khung trang vật lý (physical frame number) trên mọi node.

**Exam Answer**:
```text
False — because DSM enforces address equivalence only at the shared virtual page (virtual address) level, not at the physical frame level.
because each node's local operating system virtual memory manager independently maps a global virtual page into whatever free physical DRAM frame is available locally on that node.
```

---

#### Q2 [2 pts]
**Đề bài**: **True/False**: Giao thức nhất quán đa người ghi (Multi-writer coherence protocol) hoạt động được với cả 3 mô hình bộ nhớ SC, RC, và LRC.

**Exam Answer**:
```text
False — because multi-writer coherence protocols only work with Release Consistency (RC) and Lazy Release Consistency (LRC), not with Sequential Consistency (SC).
because multi-writer protocols rely on deferring and merging concurrent diffs at explicit synchronization release/acquire boundaries, whereas SC cannot distinguish synchronization accesses from ordinary data accesses and requires every read/write to be globally serialized immediately.
```

---

#### Q3 [2 pts]
**Đề bài**: **True/False**: Trong TreadMarks, khi xảy ra page fault cho trang X tại Node N1, phần mềm DSM trên N1 sẽ quảng bá (broadcast) số hiệu trang ảo (VPN) của trang bị lỗi tới tất cả các node ngang hàng.

**Exam Answer**:
```text
False — because TreadMarks uses point-to-point messages to known page owners and diff holders rather than broadcasting on page faults.
because each virtual page has a statically assigned manager/owner node (via hashing/partitioning) and piggybacked write notices explicitly identify which nodes hold the required diffs, allowing N1 to contact only the relevant node(s) directly.
```

---

#### Q4a [2 pts]
**Đề bài**: **True/False**: Trong Active Networks, nếu đoạn code được yêu cầu bởi một node không có mặt tại active node liền trước (`prev`), yêu cầu sẽ được gửi đệ quy (recursively) ngược về tất cả các active nodes trước đó cho tới khi lấy được code.

**Exam Answer**:
```text
False — because if the immediate PREV active node no longer has the requested code in its soft store, the capsule is simply dropped rather than triggering recursive upstream requests.
because ANTS keeps routers simple and stateless by querying only the immediate PREV node; if PREV evicted the code due to limited soft-store capacity or a restart, recovery relies on end-to-end transport retransmission from the source.
```

---

#### Q4b [2 pts]
**Đề bài**: **True/False**: Active Networks yêu cầu tất cả các router trung gian giữa nguồn và đích đều phải là router "chủ động" ("active").

**Exam Answer**:
```text
False — because ANTS capsules are encapsulated inside standard IP packets that conventional passive routers can forward normally.
because passive routers along the path simply inspect the outer IP header and forward the packet using standard IP routing tables, while only active routers inspect and execute the ANTS header.
```

---

#### Q5 [6 pts]
**Đề bài**: Thiết kế A (LRPC dùng bộ nhớ chia sẻ trên máy đa xử lý SMP, client block chờ phản hồi) vs. Thiết kế B (RPC truyền thống qua mạng LAN giữa các máy đơn xử lý theo bài báo Thekkath & Levy).
* **5a [1 pt]**: Số lần chuyển ngữ cảnh (context switches) tối thiểu cho Thiết kế A?
* **5b [1 pt]**: Các lần chuyển ngữ cảnh trong Thiết kế A là gì?
* **5c [1 pt]**: Làm thế nào để giảm thêm số lần chuyển ngữ cảnh trong Thiết kế A?
* **5d [1 pt]**: Số lần chuyển ngữ cảnh tối thiểu cho Thiết kế B (RPC truyền thống không tối ưu hóa)?
* **5e [2 pts]**: Các lần chuyển ngữ cảnh trong Thiết kế B là gì?

**Exam Answers**:

* **5a [1 pt]**:
```text
2 context switches — because in blocking shared-memory RPC on an SMP, the processor switches once from client to server and once from server back to client.
```

* **5b [1 pt]**:
```text
(1) Client -> Server (C -> S) when the client blocks after placing parameters in shared memory, and (2) Server -> Client (S -> C) when the server completes execution and wakes the client.
```

* **5c [1 pt]**:
```text
Have the client thread spin (busy-wait) on a shared-memory completion flag while the server thread executes on another SMP core — because spinning avoids blocking the client in the kernel scheduler, reducing context switches to 0 (if the server is already running on another core) or 1.
```

* **5d [1 pt]**:
```text
4 context switches (or 2 on the critical path if post-send context switches overlap with network transmission) — because both the client machine and server machine switch away when blocking on network I/O and switch back when packets arrive.
```

* **5e [2 pts]**:
```text
(1) C -> C1 on the client node (client sends request and blocks, OS switches to background process C1);
(2) S1 -> S on the server node (request packet arrives, server OS switches from running process S1 to RPC server process S);
(3) S -> S2 on the server node (server sends reply and blocks, server OS switches from S to process S2);
(4) C1 -> C on the client node (reply packet arrives, client OS switches from C1 back to the waiting client C).
```

---
---

## PHẦN 3: GIẢI CHI TIẾT ĐỀ THI SPRING 2026 — TEST 2 (THEO FORMAT CHUẨN)

> Nguồn đề gốc: [Spring 26 Test 2.pdf](file:///D:/OMSCS/AOS/CS6210-Advanced-Operating-Systems-Notes/Previous%20Tests/Spring%2026/Test%202.pdf)

---

### Section 1: Distributed Systems — Lamport's Logical Clock [7 pts]

#### Q1 [7 pts]
**Đề bài**: Cho sơ đồ hệ CSDL nhân bản gồm 3 node `DB-A`, `DB-B`, `DB-C`:
* **`DB-A`**: `a1: client write W1` $\rightarrow$ `a2: send W1 to B` ($\rightarrow b1$) $\rightarrow$ `a3: send W1 to C` ($\rightarrow c1$) $\rightarrow$ `a4: recv ACK from B` ($\leftarrow b3$) $\rightarrow$ `a5: recv W2 from B` ($\leftarrow b5$) $\rightarrow$ `a6: apply W2`.
* **`DB-B`**: `b1: recv W1 from A` ($\leftarrow a2$) $\rightarrow$ `b2: apply W1` $\rightarrow$ `b3: send ACK to A` ($\rightarrow a4$) $\rightarrow$ `b4: client write W2` $\rightarrow$ `b5: send W2 to A` ($\rightarrow a5$) $\rightarrow$ `b6: send W2 to C` ($\rightarrow c3$).
* **`DB-C`**: `c1: recv W1 from A` ($\leftarrow a3$) $\rightarrow$ `c2: apply W1` $\rightarrow$ `c3: recv W2 from B` ($\leftarrow b6$) $\rightarrow$ `c4: apply W2` $\rightarrow$ `c5: client read R2` $\rightarrow$ `c6: reply to client`.

* **1a [2 pts]**: Tại sự kiện `c5`, những lệnh ghi nào chắc chắn đã được phản ánh tại `DB-C`: `W1`, `W2`, `both`, hay `neither`? Giải thích.
* **1b [2 pts]**: Tại sự kiện `a4`, `DB-A` có thể kết luận rằng cả 3 node CSDL đều đã nhìn thấy `W1` hay không? Giải thích.
* **1c [3 pts]**: Giả sử `W1` cập nhật `x = 1` và `W2` cập nhật `x = 2`. Một lệnh đọc là *nhất quán toàn cục (globally consistent)* nếu tại thời điểm nó được phục vụ, cả 3 bản sao đều trả về cùng một giá trị của `x`. Giả sử ta thêm sự kiện đọc `Rc` trên `DB-C` ngay sau `c3` nhưng trước `c4`. Lệnh đọc tại `Rc` có nhất quán toàn cục không? Giải thích.

**Exam Answers**:

* **1a [2 pts]**:
```text
Both (W1 and W2) — because both c2 (apply W1) and c4 (apply W2) happen before c5 along DB-C's sequential process timeline (c1 -> c2 -> c3 -> c4 -> c5).
because by Lamport's sequential process ordering and transitivity, DB-C has already locally applied W1 at c2 and W2 at c4 before serving the read at c5.
```

* **1b [2 pts]**:
```text
No — because at event a4, DB-A only has causal confirmation that DB-B received and applied W1 (via a2 -> b1 -> b2 -> b3 -> a4), while DB-C's receipt of W1 (c1) is concurrent with a4.
because there is no causal happens-before path from c1 (or c2) back to a4 (a3 -> c1 is an unacknowledged one-way message over a variable-latency network), so the message from a3 to c1 could still be in transit when a4 occurs.
unless DB-C also sends an ACK back to DB-A that arrives before a4, DB-A cannot know whether DB-C has seen W1.
```

* **1c [3 pts]**:
```text
No — because at Rc (between c3 and c4), the three replicas do not hold the same value of x: DB-C still returns x = 1 while DB-B already returns x = 2.
because at Rc, DB-C has applied W1 at c2 (x = 1) and received W2 at c3, but has not yet applied W2 until c4; meanwhile, c3 causally follows b6 (b4 -> b5 -> b6 -> c3), which means DB-B has already executed b4 (client write W2) and holds x = 2 (and DB-A may hold either x = 1 or x = 2 since a5/a6 are concurrent with Rc).
unless all replicas synchronize or wait for W2 to be applied everywhere before serving Rc, reading at Rc violates global consistency.
```

---

### Section 2: Lamport's ME Algorithm [8 pts]

#### Q1 [8 pts]
**Đề bài**: Cho sơ đồ 4 node (`P1, P2, P3, P4`) chạy thuật toán Lamport ME:
* `P1` khởi tạo yêu cầu `[(10, P1)]`, gửi `rq_12`, `rq_13`, `rq_14`.
* `P4` khởi tạo yêu cầu `[(15, P4)]`, gửi `rq_41`, `rq_42`, `rq_43`.
* Trước đường kẻ dọc thời gian toàn cục `T`: `P1` đã nhận `ak_21` (từ `P2`) và nhận `rq_41` (từ `P4` với `TS = 15`). Tuy nhiên, `ak_41` (từ `P4`) và `ak_31` (từ `P3`) đều đến `P1` **SAU** vạch `T`!
* Sau vạch `T`: `P1` nhận `ak_41` và `ak_31`, vào vùng găng (`CS`), rồi gửi `rl_12, rl_13, rl_14`. Ngoài ra `P2` gửi `ak_24`, `P3` gửi `ak_34`, `P1` gửi `ak_14` cho `P4`.

* **1a [2 pts]**: `P1` có thể lấy khóa tại thời điểm toàn cục `T` không? Nếu không, giải thích dựa trên 2 điều kiện của thuật toán và nêu điều gì có thể sai nếu `P1` cứ lấy khóa tại `T`.
* **1b [2 pts]**: (i) Đề xuất một cải tiến để giảm số lượng thông điệp xuống dưới $3(N-1)$ mà không vi phạm tính an toàn (safety). (ii) Nêu **MỘT** ví dụ cụ thể về một thông điệp trong sơ đồ hiện tại có thể bỏ đi và giải thích tại sao.
* **1c [2 pts]**: Một đồng nghiệp đề xuất chỉ gửi thông điệp nhả khóa (`rl`) cho các node hiện đang chờ lấy khóa (tức là các node mà người giữ khóa đã nhận được thông điệp `rq` từ họ). Cải tiến này có đảm bảo tính an toàn và đúng đắn (safety and correctness) không? Những thông điệp nào trong sơ đồ sẽ bị bỏ đi theo đề xuất này?
* **1d [2 pts]**: Độ phức tạp thông điệp (message complexity) theo đề xuất của đồng nghiệp trong trường hợp **Low contention** (chỉ 1 node xin khóa tại một thời điểm) và **High contention** (tất cả $N$ node đều tích cực xin khóa) là bao nhiêu?

**Exam Answers**:

* **1a [2 pts]**:
```text
No — because at global timestamp T, P1 violates Lamport's second mutual exclusion condition since it has not yet heard from P3 (ak_31 arrives after T).
because although Condition 1 holds locally ((10, P1) is at the head of P1's queue ahead of (15, P4)) and P1 has heard from P2 (ak_21) and P4 (rq_41 with TS=15 > 10), P1 has received no message with timestamp > 10 from P3 by time T; if P1 acquired the lock at T, P3 could have an earlier lock request (e.g., TS < 10) still in transit to P1, allowing both P1 and P3 to enter the critical section simultaneously and violating mutual exclusion.
```

* **1b [2 pts]**:
```text
(i) Suppress redundant ACKs when a later-timestamped REQ has already been sent or defer ACKs and piggyback them on Lock Release (reducing messages toward 2(N-1)); (ii) Discard ak_41 (or ak_14) — because FIFO channels and request timestamps make these explicit ACKs redundant.
because:
- Option A (Discard ak_41 from P4 to P1): Before sending ak_41, P4 already sent rq_41 with timestamp (15, P4) to P1, which arrived at P1 before T. Since 15 > 10 and channels are FIFO, receiving rq_41 already guarantees P1 that P4 has no request earlier than TS=10, making ak_41 completely redundant.
- Option B (Discard ak_14 from P1 to P4): Because P1's request (10, P1) has higher priority (10 < 15) than P4's request (15, P4), P1 can defer its acknowledgment ak_14 and let its release message rl_14 serve as both the release of (10, P1) and the ACK for (15, P4).
```

* **1c [2 pts]**:
```text
No, it violates correctness (causes deadlock/starvation for future requests), and in our timeline it would discard rl_12 and rl_13 — because P2 and P3 did not send any rq messages, so P1 would only send rl_14 to P4.
because in Lamport's algorithm every node (including P2 and P3) inserts (10, P1) into its local queue upon receiving rq_12 and rq_13, and only removes (10, P1) when it receives a release message; if rl_12 and rl_13 are omitted, (10, P1) remains permanently stuck at the head of Q2 and Q3, preventing P2 and P3 from ever acquiring the lock in the future.
unless non-requesting nodes do not store requests in their queues (which would break ordering if P2 or P3 requests the lock while P1 is still in the critical section), omitting rl_12 and rl_13 breaks the algorithm.
```

* **1d [2 pts]**:
```text
Low contention: 2(N - 1) messages; High contention: 3(N - 1) messages (or 2(N - 1) if combined with deferred ACKs) — because the number of release messages scales with the number of waiting peers.
because:
- Low contention (only 1 node requests the lock, 0 other nodes waiting): The requester sends (N - 1) request messages, receives (N - 1) acknowledgment messages, and sends 0 release messages, totaling 2(N - 1) messages.
- High contention (all N nodes concurrently request the lock, so all N - 1 peers are waiting): Each lock acquisition requires (N - 1) requests + (N - 1) ACKs + (N - 1) release messages sent to all (N - 1) waiting peers, totaling 3(N - 1) messages (or 2(N - 1) messages if deferred ACKs are merged with release messages).
```

---

### Section 3: RPC Latency Limits [6 pts]

#### Q1 [6 pts]
**Đề bài**: Cho 4 biến chi phí cơ bản của một lời gọi RPC đồng bộ (blocking):
* $t_c$: thời gian copy $n$ bytes dữ liệu.
* $t_x$: thời gian chuyển ngữ cảnh (context switch) giữa $P$ và $P'$.
* $t_n$: thời gian truyền $n$ bytes qua mạng.
* $t_s$: thời gian thực thi thủ tục $P$ trên máy chủ $S$.
Biết cả tham số đầu vào của client và kết quả trả về của server đều có kích thước $n$ bytes.

* **1a [2 pts]**: Giả sử không có bất kỳ tối ưu hóa nào, hãy viết biểu thức tính tổng thời gian trễ $t$ của lời gọi RPC theo $t_c, t_x, t_n, t_s$.
* **1b [2 pts]**: Để giảm độ trễ, kernel đảm bảo rằng các lần chuyển ngữ cảnh sau khi gửi dữ liệu sẽ diễn ra song song với quá trình truyền dữ liệu trên mạng (overlapping context switches with network transmission). Viết lại biểu thức tính $t$.
* **1c [2 pts]**: Bên cạnh cải tiến ở câu 1b, kernel cung cấp thêm một vùng đệm chung (shared kernel buffer/descriptor) được sử dụng trực tiếp bởi cả client stub và server stub. Viết lại biểu thức tính $t$.

**Exam Answers**:

* **1a [2 pts]**:
```text
t = 6*t_c + 4*t_x + 2*t_n + t_s (or 4*t_c + 4*t_x + 2*t_n + t_s if counting only software copies and excluding NIC DMA copies; or 8*t_c if counting both send and receive DMA copies) — because an unoptimized blocking RPC incurs data copies, context switches, network transfers on both call and return paths, plus server execution.
because:
- Data copies (6*t_c total per Lecture 05c "3 copies" per direction, or 4*t_c software-only): On both the request path and reply path, data is copied (1) between stack and user-space RPC stub buffer, (2) between user stub buffer and kernel buffer, and (3) via DMA between kernel buffer and NIC controller buffer.
- Context switches (4*t_x): 4 context switches occur across the round trip (C -> C1 after client send, S1 -> S upon server receive, S -> S2 after server reply send, and C1 -> C upon client reply receive).
- Network transmission (2*t_n): 1 network transfer of n bytes for the request (t_n) + 1 network transfer of n bytes for the response (t_n).
- Server execution (t_s): 1 execution of procedure P on server S.
```

* **1b [2 pts]**:
```text
t = 6*t_c + 2*t_x + 2*t_n + t_s (or 4*t_c + 2*t_x + 2*t_n + t_s for software-only copies, assuming t_x <= t_n; more generally 6*t_c + 2*t_x + 2*max(t_n, t_x) + t_s) — because overlapping post-send context switches with network wire transmission removes 2 context switches from the critical path.
because the 1st context switch (C -> C1 on the client after sending the request) and the 3rd context switch (S -> S2 on the server after sending the reply) occur in parallel with the two network transmissions (2*t_n), leaving only the 2 receive-side context switches (S1 -> S and C1 -> C, costing 2*t_x) on the critical path.
```

* **1c [2 pts]**:
```text
t = 4*t_c + 2*t_x + 2*t_n + t_s (or 2*t_c + 2*t_x + 2*t_n + t_s if counting only software copies) — because a shared kernel buffer/descriptor used directly by client and server stubs eliminates 1 intermediate user-space copy on the call path and 1 on the return path (saving 2*t_c).
because stubs marshal and unmarshal arguments/results directly into/from the kernel-accessible buffer rather than copying between a user-space RPC buffer and a separate kernel buffer on both the request and response paths.
```

---

### Section 4: Active Networks [7 pts]

#### Q1 [4 pts]
**Đề bài**: Xét 3 active nodes `A -> B -> C`. Một capsule của giao thức mới đi qua `A`, `B`, và vừa tới `C`. Giả sử `B` đang có sẵn code cho `TYPE` của capsule trong soft store, còn `C` chưa từng thấy `TYPE` này trước đây.
* **1a [2 pts]**: Trình bày từng bước điều gì xảy ra tại `C` từ lúc capsule đến cho tới khi nó được chuyển tiếp đi.
* **1b [2 pts]**: Giả sử ngay trước khi `C` yêu cầu code, soft store của `B` bị xóa sạch do router khởi động lại nên `B` không còn code nữa. Kết quả tại `C` thay đổi thế nào, và điều này cho ta biết gì về thiết kế dựa trên `PREV`?

**Exam Answers**:

* **1a [2 pts]**:
```text
Cache Miss at C -> Code Request to PREV (B) -> Hash Verification -> Cache, Execute, and Forward — because ANTS dynamically loads and verifies missing capsule code on demand from the immediate upstream active node.
because:
1. Soft-store lookup miss: Node C reads the capsule's ANTS header, extracts the TYPE field (the 128-bit MD5 cryptographic hash of the forwarding routine), and checks its local soft store, resulting in a cache miss.
2. On-demand code request: Node C reads the PREV field in the capsule header (which identifies upstream active node B) and sends a request message to B asking for the code corresponding to TYPE.
3. Code transfer & fingerprint verification: Node B retrieves the code from its soft store and sends it to C; C computes the MD5 hash of the received code and verifies that it matches the capsule's TYPE field to prevent code spoofing.
4. Caching, sandboxed execution, and forwarding: Once verified, C caches the code in its local soft store for future capsules of the flow, executes the forwarding routine in its Java sandbox, updates the capsule's PREV field to C, and forwards the capsule toward its destination.
```

* **1b [2 pts]**:
```text
Node C drops the capsule and relies on end-to-end retransmission from the source — because ANTS does not recursively query earlier upstream nodes beyond PREV.
because when B cannot supply the code after a restart, B does not chain the request back to A, and C simply discards the sleeping capsule; this demonstrates that ANTS's PREV-based code distribution is a lightweight, best-effort, soft-state mechanism that keeps routers stateless and relies on end-to-end transport protocols at the source (A) to time out, retransmit, and repopulate soft stores along the path.
```

---

#### Q2 [3 pts]
**Đề bài**: Một sinh viên đề xuất nhúng toàn bộ đoạn code thực thi trực tiếp vào trong **mọi capsule** thay vì dùng trường `TYPE` và soft store. Nêu **1 Ưu điểm (Advantage)** và **1 Nhược điểm (Disadvantage)** của cách tiếp cận này so với thiết kế của ANTS.

**Exam Answer**:
```text
Advantage: Eliminates cold-start code-fetch latency and capsule drops from soft-store misses; Disadvantage: Massive network bandwidth and transmission latency overhead on every packet — because self-contained capsules trade network bandwidth for router simplicity.
because:
- Advantage: Every capsule is completely self-contained, so routers never stall waiting to fetch missing code from PREV on the first packet of a flow and never drop capsules when upstream soft stores evict code or restart.
- Disadvantage: Carrying the full executable code in every single packet wastes enormous link bandwidth and increases serialization/transmission delay on every hop, whereas ANTS amortizes a single code transfer across all subsequent capsules of a flow via soft-store caching.
```

---

### Section 5: Distributed Objects and Middleware [10 pts]

#### Spring OS — Q1 [7 pts]
**Đề bài**: Ban đầu 4 thành phần `Client A, Client B, Server Y, Server Z` chạy trên cùng 1 node. Sau đó tách thành 2 node nối qua LAN: `Node 1 (Client A, Server Z)` và `Node 2 (Client B, Server Y)`.
* **1a [2 pts]**: Đồng nghiệp khẳng định rằng **client-side stubs** phải được sửa đổi để `Client A` (trên Node 1) gọi được `Server Y` (trên Node 2). Chỉ ra điểm sai lầm (fallacy) trong lập luận này và giải thích thành phần nào thực sự thay đổi.
* **1b [3 pts]**: `Object Y` hỗ trợ cả thao tác đọc (read) và ghi (write). `Client A` cần quyền **read/write**, trong khi `Client B` chỉ được cấp quyền **read-only**. Giải thích cách các cơ chế của Spring (`capabilities`, `access control`, `doors`) phối hợp để đạt được điều này.
* **1c [2 pts]**: Giả sử sau đó `Client A` truyền capability `read/write` của mình đối với `Object Y` cho `Client B`. Làm thế nào `Client B` có được quyền ghi vào `Object Y`?

**Exam Answers**:

* **1a [2 pts]**:
```text
Fallacy: Client-side stubs are completely location-independent and do NOT change; What actually changes is the underlying Subcontract and Network Proxies — because Spring decouples stub marshaling from object communication mechanics.
because the client stub is generated strictly from the IDL interface and simply invokes the subcontract interface regardless of whether the target domain is local or remote; to cross the LAN from Node 1 to Node 2, only the subcontract implementation changes to route the invocation through local Doors into Network Proxy domains on Node 1 and Node 2 connected by a network handle.
```

* **1b [3 pts]**:
```text
Server Y uses Front Objects, an Access Control List (ACL), distinct Doors per privilege level, and unforgeable Door Capabilities — because Spring enforces fine-grained access control at domain entry points.
because:
1. Front Object & ACL: Server Y places a Front Object (or multiple Front Objects) in front of the underlying implementation of Object Y and maintains an Access Control List (ACL) specifying principal permissions.
2. Distinct Doors: The Front Object registers separate Doors with the Nucleus corresponding to different access rights—one Door entry point permitting both read and write operations, and a second Door entry point restricted to read-only operations.
3. Software Capabilities: Upon authentication, Client A is granted a kernel-protected software capability (door handle in its domain's door table) pointing to the read/write Door (via Network Proxies), while Client B is granted a door handle pointing only to the read-only Door.
```

* **1c [2 pts]**:
```text
Client A's door handle is a transferable software capability that the Nucleus and Subcontract marshal and install into Client B's door table — because possession of a door capability in Spring confers the access rights bound to that door.
because when Client A passes its read/write object reference as an invocation parameter to Client B, the subcontract and nuclei (via Network Proxies between Node 1 and Node 2) translate and copy that capability into Client B's domain door table pointing to the same high-privilege read/write Door/Front Object on Server Y, enabling Client B to invoke write methods through the newly acquired handle.
```

---

#### Java RMI — Q1 [3 pts]
**Đề bài**: Một client đang kết nối với một remote object thì kết nối mạng đột ngột bị đứt. Client tự động kết nối lại với remote object mà **không làm lộ bất kỳ lỗi nào lên ứng dụng client**. Mô tả vai trò của từng tầng trong 3 tầng của Java RMI (`Stub/Skeleton Layer`, `Remote Reference Layer`, `Transport Layer`) trong tình huống này.

**Exam Answer**:
```text
Stub/Skeleton shields the application, Remote Reference Layer executes transparent retry/reconnection semantics, and Transport Layer detects the dead link and establishes a new network channel — because Java RMI isolates fault recovery inside the RRL and Transport layers.
because:
1. Stub/Skeleton Layer: Remains completely unaffected and hides the failure from the client application; the client stub simply blocks waiting for the call to return on the stream provided by the RRL without raising an exception.
2. Remote Reference Layer (RRL): Implements the invocation and reconnection semantics; when notified by the transport layer that the connection dropped, the RRL intercepts the transport error, requests a new connection to the remote endpoint, and transparently re-transmits the marshaled invocation instead of propagating a RemoteException up to the stub.
3. Transport Layer: Detects the severed connection via connection liveness monitoring, tears down the broken TCP/UDP socket and channel state, and establishes a fresh network connection to the remote server's endpoint when instructed by the RRL.
```

---

### Section 6: Distributed Subsystems [32 pts]

#### GMS — Q1 [5 pts]
**Đề bài**: Cụm 3 node (`A, B, C`) chạy GMS. Với trang `P`, Node A là **Home Node** (chứa GCD của P). Trạng thái tại `T0`:
* GCD tại Home Node A của P: `{A}`
* PFD tại Node A: `Local-Private`
* PFD tại Node B: `Non-existent`
* PFD tại Node C: `Non-existent`

* **1a [2 pts]**: **Event 1 (`T1 > T0`)**: Một tiến trình trên Node B bị page fault trên trang P (Case 4: trang chia sẻ đang nằm trong bộ nhớ cục bộ của Node A). Cho biết trạng thái của 4 cấu trúc trên sau Event 1.
* **1b [1 pt]**: **Event 2 (`T2 > T1`)**: Node A gặp áp lực bộ nhớ và muốn đẩy (evict) trang sạch P. Giả sử nếu trang được giữ trong global memory của cụm thì Node C là ứng viên nhận trang. Phát biểu sau là **True** hay **False**: *"Node A will send the Page P to Node C's global part of the DRAM."* Giải thích.
* **1c [2 pts]**: Cho biết trạng thái của 4 cấu trúc trên sau Event 2.

**Exam Answers**:

* **1a [2 pts]**:
```text
1. GCD at Home Node (A) for Page P: {A, B}
2. PFD at Node A: Local-Shared
3. PFD at Node B: Local-Shared
4. PFD at Node C: Non-existent
— because when Node B faults on Page P held in Node A's local cache (Case 4), Node A sends a copy of P to Node B's local cache, both A and B mark P as Local-Shared in their PFDs, and the GCD at A records both A and B as sharers.
```

* **1b [1 pt]**:
```text
False — because Page P is in the Local-Shared state with a valid duplicate copy already cached in Node B's local DRAM ({A, B}), so Node A simply discards its local copy of P rather than sending a redundant copy to Node C's global memory.
because GMS only sends evicted pages to a peer's global memory when the evicted copy is the sole copy in the cluster (Local-Private); when another node (Node B) already holds a copy, sending it to Node C would waste network bandwidth and global DRAM.
```

* **1c [2 pts]**:
```text
1. GCD Owner List at Home Node (A) for Page P: {B}
2. PFD at Node A: Non-existent
3. PFD at Node B: Local-Private (or remains Local-Shared locally if Node B's PFD is updated lazily, as only B holds the page now)
4. PFD at Node C: Non-existent
— because Node A drops its local copy of P and removes itself from the GCD sharer list at A, leaving Node B as the sole holder of Page P in the cluster while Node C never receives the page.
```

---

#### GMS — Q2 [2 pts]
**Đề bài**: **True/False**: Trong GMS, khi một node quyết định gửi một trang sang phần bộ nhớ toàn cục (global part of the DRAM) của một node ngang hàng, nó sẽ ghi trang đó xuống đĩa nếu trang đó đang bị bẩn (dirty).

**Exam Answer**:
```text
True — because GMS enforces the strict invariant that all pages residing in any node's global memory cache must be clean.
because writing dirty pages back to disk before transferring them to a peer's global memory guarantees that a crash of a peer node hosting global pages never results in data loss or requires complex distributed recovery.
```

---

#### GMS — Q3 [2 pts]
**Đề bài**: **True/False**: Một thiết kế đơn giản và hiệu quả hơn trong GMS là quảng bá (broadcast) mã định danh UID của trang bị thiếu tới tất cả các node ngang hàng mỗi khi xảy ra page fault tại một node.

**Exam Answer**:
```text
False — because broadcasting on every page fault causes O(N) network traffic and CPU interrupt/lookup overhead at every node in the cluster, which scales poorly compared to GMS's point-to-point directory hierarchy.
because GMS's POD -> GCD -> PFD directory design resolves page faults using only 2 to 3 point-to-point network hops without disturbing unrelated nodes in the cluster.
```

---

#### GMS — Q4 [2 pts]
**Đề bài**: Một đồng nghiệp đề xuất duy trì một cấu trúc dữ liệu nhân bản (replicated data structure) ánh xạ trực tiếp `UID -> Node currently hosting the page` tại mọi node trong cụm để mọi page fault chỉ tốn đúng 1 hop mạng. Nêu **2 ý ngắn gọn** để phản biện lại lập luận của cô ấy.

**Exam Answer**:
```text
Prohibitive cluster-wide update traffic on every page movement and excessive local DRAM footprint — because exact page locations change dynamically on every page fault, eviction, and disk read across the cluster.
because:
1. Massive Coherence/Update Broadcast Overhead: Unlike the POD (which maps coarse partitions and only changes on node joins/failures), a replicated UID-to-Host table would require broadcasting update messages to all N nodes on every single page fault, eviction, or replacement across the cluster, saturating the network.
2. Excessive Memory Footprint: Replicating fine-grained entries for every cached UID across the entire cluster in every node's RAM would consume a huge amount of local DRAM, defeating the goal of maximizing usable memory.
```

---

#### DSM — Q1a [5 pts]
**Đề bài**: Xét hệ thống DSM **TreadMarks** gồm 8 node (`N1..N8`) với một trang ảo `X` ban đầu được nhân bản hợp lệ trên cả 8 node. Một khóa toàn cục `L1` được lấy lần lượt theo thứ tự: `N1, N2, N3, N4, N5, N6, N7, N8`.
* `N1` ghi vào trang `X` trong vùng găng bảo vệ bởi `L1`.
* `N2..N7` lần lượt lấy và nhả `L1`, nhưng **KHÔNG** truy cập trang `X` trong vùng găng của họ.
* `N8` lấy `L1` và **ĐỌC (reads)** trang `X` trong vùng găng của nó.
Mô hình nào cho hiệu năng tốt hơn — **Eager Release Consistency (ERC)** hay **Lazy Release Consistency (LRC)**? Tính tổng số thông điệp đặc thù của DSM (invalidation messages, update messages, hoặc diff requests/responses — không tính các thông điệp chuyển khóa `L1`) trong cả ERC và LRC để chứng minh.

**Exam Answer**:
```text
Lazy Release Consistency (LRC) performs significantly better (2 DSM messages in LRC vs. 9 DSM messages in invalidate-based ERC, or 16 with ACKs, or 7 in update-based ERC) — because LRC piggybacks invalidation notices on lock transfers and avoids notifying nodes (N2..N7) that never access Page X.
because:
- In Lazy Release Consistency (LRC): When N1 releases L1, 0 standalone invalidation messages are sent because the write notice for X is piggybacked on the lock transfer messages (N1 -> N2 -> ... -> N8). Since N2..N7 never access X, they incur 0 DSM messages. When N8 reads X, N8 faults and sends 1 diff request to N1 and receives 1 diff response from N1, totaling 2 DSM-specific messages.
- In Eager Release Consistency (ERC): When N1 releases L1, it immediately broadcasts invalidations (or updates) for X to all 7 other caching nodes N2..N8 (7 messages, or 14 messages if invalidation ACKs are counted). Later, when N8 reads X, an invalidate-based ERC faults and sends 1 page/diff request to N1 and receives 1 response (2 messages), totaling 9 DSM-specific messages (or 16 with ACKs; even in update-based ERC, N1 sends 7 eager update messages vs. LRC's 2 messages).
```

---

#### DSM — Q2 [8 pts]
**Đề bài**: Hệ thống DSM **TreadMarks**. Xét chuỗi thực thi qua các mốc thời gian `T1 < T2 < T3 < T4` trên 4 tiến trình (`P1, P2, P3, P4`):
* **`T1` (`P1`)**: `acq(L1); write X; rel(L1)`
* **`T2` (`P2`)**: `acq(L1); write Y; rel(L1)`
* **`T3` (`P3`)**: `acq(L2); write X; rel(L2)` *(Lưu ý: dùng khóa `L2`!)*
* **`T4` (`P4`)**: `acq(L1); read X; write Y; rel(L1)`
Giả sử các bản gốc (pristine copies) của cả hai trang `X` và `Y` luôn nằm tại owner node `O` trong suốt quá trình, và ban đầu cả `X` lẫn `Y` đều **chưa có mặt** tại `P1..P4`.

* **2a [2 pts]**: Những hành động nào xảy ra khi `P4` lấy khóa `acq(L1)` tại `T4`?
* **2b [4 pts]**: Trong khi thực thi vùng găng dưới khóa `L1`, `P4` đọc `X` trước rồi ghi `Y` sau (`read X; write Y`). Những hành động nào cần thực hiện?
* **2c [2 pts]**: Điều gì xảy ra khi `P4` nhả khóa `rel(L1)`?

**Exam Answers**:

* **2a [2 pts]**:
```text
P4 receives piggybacked write notices for Page X (modified by P1 at T1) and Page Y (modified by P2 at T2) along L1's causal chain, and records that X and Y require those diffs (no local pages are invalidated since X and Y are not yet present at P4) — because P2 passes L1 to P4 along with L1's causal invalidation notices.
because L1's synchronization causality chain is P1 (T1) -> P2 (T2) -> P4 (T4), so the lock grant message carries write notices for P1's write to X and P2's write to Y, while completely ignoring P3's write to X at T3 because T3 was protected by a different, causally unrelated lock (L2).
```

* **2b [4 pts]**:
```text
On read X: fetch clean copy of X from Owner O, fetch and apply only diff Xd1 from P1, and map X as read-only (no twin created); On write Y: fetch clean copy of Y from Owner O, fetch and apply diff Yd2 from P2, create a twin of Y, and map Y as read-write — because TreadMarks applies only causally preceding diffs and creates twins only on writes.
because:
1. When P4 executes read X: Since X is not present at P4, a page fault occurs. P4 fetches the pristine copy of X from Owner O, requests and applies only diff Xd1 from P1 (it does NOT fetch Xd3 from P3 because T3 used lock L2, which has no happens-before relation with L1), and maps X as read-only without creating a twin.
2. When P4 executes write Y: Since Y is not present at P4, a write fault occurs. P4 fetches the pristine copy of Y from Owner O, requests and applies diff Yd2 from P2 (from T2 under L1), allocates a twin (a clean copy of the updated page Y in system space), and maps Y with read-write permissions so the write can complete.
```

* **2c [2 pts]**:
```text
P4 compares Page Y against its twin to create diff Yd4, marks Page Y as read-only (write-protected), and discards Y's twin (Page X remains untouched as read-only) — because P4 only modified Page Y during its critical section.
because at rel(L1), TreadMarks generates a run-length encoded diff Yd4 capturing P4's modifications to Y, downgrades Y's OS page protection back to read-only so future writes will trap, and frees the twin of Y.
```

---

#### DFS — Q1 [2 pts]
**Đề bài**: Trong hệ thống file phân tán **xFS**, chính sách **"First Writer"** đặt Metadata Manager của một file trên cùng máy trạm với client đã tạo ra file đó. Dựa trên kết quả đánh giá trong bài báo xFS, giải thích tại sao chính sách này giúp ích cho các thao tác **GHI (writes)** nhiều hơn hẳn so với các thao tác **ĐỌC (reads)**.

**Exam Answer**:
```text
Write-sharing across different clients is rare so First Writer makes ~90% of write metadata messages local (0 network hops), whereas reads either hit in the local client cache already or require remote network hops to storage servers/peers anyway — because file creation and subsequent modifications exhibit strong single-client write locality.
because in typical workloads, the client that creates a file is almost always the same client that later overwrites or deletes it (96% of overwrites are by the previous writer), so co-locating the manager with the First Writer eliminates network messages for 90% of write-ownership requests and 90% of disk-flush notifications; by contrast, over 75% of reads are already satisfied locally by the client's memory cache without contacting the manager, and read misses must still traverse the network to fetch data from remote storage servers or peer caches.
```

---

#### DFS — Q2 [3 pts]
**Đề bài**: Giả sử một kỹ sư đề xuất đơn giản hóa **xFS** bằng cách bỏ toàn bộ khái niệm **stripe groups** và thay vào đó trải (striping) mỗi log segment qua **TẤT CẢ** hàng trăm storage servers trong cụm. Dựa trên bài báo xFS, nêu **3 vấn đề khác nhau** sẽ phát sinh từ thiết kế này.

**Exam Answer**:
```text
Small fragment/memory buffering overhead, loss of aggregate parallel I/O bandwidth, high log cleaning overhead, and drastically reduced fault tolerance — because striping every log segment across hundreds of disks violates scalability and reliability boundaries.
because (any 3 of the following 4 issues from the xFS paper):
1. Small-Write / Memory Buffering Problem: Striping a fixed-size segment across hundreds of servers produces tiny per-server disk writes that ruin disk seek/transfer efficiency, or else forces each client to buffer an impractically large segment in RAM before flushing.
2. Loss of Write Parallelism: Every client write touches all storage servers simultaneously, creating contention across every disk instead of allowing different clients to write in parallel at full speed to independent subsets of servers.
3. Severe Log Cleaning Overhead: Huge segments spanning all servers remain "partially full" much longer than smaller stripe-group segments, forcing log cleaners to read enormous partially-full segments across the network.
4. Reduced Availability / Reliability: Under single-parity RAID striping across all hundreds of servers, any 2 simultaneous server crashes anywhere in the entire cluster cause permanent data loss and halt the whole file system, whereas independent stripe groups survive multiple failures in different groups.
```

---

#### DFS — Q3 [3 pts]
**Đề bài**: Trong **xFS**, các **index nodes** (chứa địa chỉ đĩa của các khối dữ liệu của file) được cache tại các **Metadata Managers**, nhưng **KHÔNG** được cache tại các **Clients**.
* **3a [1 pt]**: Nếu index nodes được phép cache tại clients, lợi ích hiệu năng sẽ là gì? Nói rõ những thông điệp mạng nào sẽ được loại bỏ.
* **3b [2 pts]**: Nêu **2 nhược điểm khác nhau** từ bài báo xFS giải thích tại sao các tác giả quyết định không cache index nodes tại clients.

**Exam Answers**:

* **3a [1 pt]**:
```text
Clients could read data blocks directly from Storage Servers on a local cache miss, eliminating the Client -> Manager read request message and the Manager -> Storage Server forwarding message (saving 2 network hops) — because the client could look up the block's disk log address locally in its cached index node.
```

* **3b [2 pts]**:
```text
Loss of cooperative caching from peer client memories, continued need to notify the manager for cache consistency, and added index-node coherence complexity — because bypassing the manager on reads defeats xFS's centralized consistency and memory-sharing optimizations.
because (any 2 of the following 3 drawbacks from the xFS paper):
1. Loses Cooperative Caching: If a client reads directly from a Storage Server's disk using a locally cached index node, it bypasses the Manager and loses the ability to fetch the block much faster from another peer client's DRAM via cooperative caching.
2. Manager Notification Still Required: Even when reading directly from disk, the client must still send a network message to inform the Manager that it is now caching the block so the Manager can track sharers and invalidate the block on future writes.
3. Extra Coherence Complexity: Caching index nodes at clients would require an additional cache invalidation/consistency protocol for the index nodes themselves whenever a file is modified and its index node is rewritten to a new log address.
```

---

### Section 7: HIDDEN - Potpourri [15 pts]

#### Q1 [2 pts]
**Đề bài**: **True/False**: Trong thuật toán khóa loại trừ tương hỗ của Lamport (Lamport's mutual exclusion lock algorithm), khi một node "A" xác định rằng nó đã lấy được khóa, nội dung của các hàng đợi cục bộ (local queues) trên tất cả các node là giống hệt nhau.

**Exam Answer**:
```text
False — because while Node A's request is guaranteed to be at the head of every node's queue, other nodes' later requests or release messages may still be in transit across the variable-latency network.
because Node A only needs local confirmation that its request is at the head of its own queue and that every peer has sent a message with a higher timestamp, leaving the tail entries of different nodes' local queues temporarily different.
```

---

#### Q2 [2 pts]
**Đề bài**: **True/False**: Việc tăng gấp đôi kích thước **soft store** tại một active router là không cần thiết vì bất kỳ breadcrumb hoặc đoạn code nào bị đẩy ra (evicted) đều luôn có thể được khôi phục bằng cách yêu cầu lại code từ node `PREV`.

**Exam Answer**:
```text
False — because the PREV node also has a finite soft store that may have evicted the code (causing the capsule to be dropped), and breadcrumbs (flow-specific dynamic state) cannot be reconstructed simply by fetching static code from PREV.
because (1) ANTS does not recursively query upstream beyond PREV, so if PREV also evicted the code the capsule is lost, (2) fetching evicted code from PREV still adds extra network round-trip latency on the critical path, and (3) breadcrumbs store dynamic per-flow routing/service state left behind at that specific router rather than static protocol code.
```

---

#### Q3 [2 pts]
**Đề bài**: Alice gửi một yêu cầu `A` tới một remote server. Sau khi gửi xong, cô ấy gọi điện thoại cho Bob để báo tin. Bob nghe máy xong liền gửi một yêu cầu `B` tới cùng remote server đó. Hệ thống phân tán sử dụng Lamport's Logical Clock (phá vỡ hòa bằng machine ID) để sắp xếp toàn phần các sự kiện. Mạng không mất gói và đảm bảo FIFO giữa từng cặp node. Remote server có luôn đáp ứng kỳ vọng của Alice rằng yêu cầu `A` của cô ấy được xử lý trước yêu cầu `B` của Bob hay không?

**Exam Answer**:
```text
No (False) — because the phone call between Alice and Bob is an out-of-band (external physical) communication channel outside the distributed system, so Lamport's logical clocks do not capture the causal dependency between Request A and Request B.
because Alice's node and Bob's node did not exchange any in-system message carrying Alice's logical timestamp before Bob submitted Request B; consequently, Bob's local logical clock C(B) may be smaller than Alice's C(A) (or B may arrive at the server before A), causing the server to order and process B before A.
unless Alice and Bob communicate through messages tracked by the system's logical clocks (or use synchronized physical clocks), external real-time causality is invisible to Lamport clocks.
```

---

#### Q4 [4 pts]
**Đề bài**: Nêu **1 Ưu điểm (Advantage)** và **1 Nhược điểm (Disadvantage)** cho từng phương pháp thiết kế ứng dụng trong **EJB**:
1. **Entity Beans (Data Access Objects)**
2. **Session Facade (Session Bean with Entity Bean)**

**Exam Answer**:
```text
Entity Beans offer high database concurrency but expose business logic/overhead, whereas Session Facade hides business logic and reduces WAN round-trips while preserving concurrency at the cost of an extra indirection layer — because Session Facade combines coarse-grained session management with fine-grained entity caching.
because:
1. Entity Beans (Data Access Objects):
   - Advantage (Pro): Enables high concurrency and parallelism for database access (multiple entity bean instances can cache and access different database rows in parallel across clients).
   - Disadvantage (Con): Exposes internal business logic and database schema to external callers (e.g., the web tier/WAN) and incurs high container management and remote invocation overhead for fine-grained calls.
2. Session Facade (Session Bean with Entity Bean):
   - Advantage (Pro): Achieves the best of both worlds by encapsulating and protecting business logic inside the EJB container (reducing fine-grained WAN round-trips) while still leveraging parallel database access through the underlying Entity Beans.
   - Disadvantage (Con): Introduces an extra layer of indirection and inter-bean communication overhead between the Session Bean and the Entity Beans (unless mitigated via co-located local interfaces).
```

---

#### Q5 [5 pts]
**Đề bài**: Xét hệ thống DSM **TreadMarks**:
* **5a [2 pts]**: Trong giao thức đa người ghi (multiple-writer protocol) của TreadMarks, những hành động nào xảy ra khi một bộ xử lý thực hiện **lần ghi đầu tiên (first write)** vào một trang chia sẻ trong một khoảng đồng bộ hóa (interval)?
* **5b [3 pts]**: **True/False**: Nếu hai bộ xử lý cùng sửa đổi các từ bị trùng lắp (overlapping words) trên cùng một trang mà **không có đồng bộ hóa ở giữa (without intervening synchronization)**, TreadMarks sẽ gộp (merge) một cách tất định cả hai cập nhật bằng cách áp dụng các bản diff theo thứ tự.

**Exam Answers**:

* **5a [2 pts]**:
```text
An OS write-protection fault traps into TreadMarks, which allocates a "twin" (an identical copy of the clean page in system space) and upgrades the page's OS protection from read-only to read-write — because TreadMarks must record the pre-modification state of the page to compute a diff later.
because valid shared pages are initially mapped read-only; the first write triggers a page protection fault so TreadMarks can copy the pristine page into a twin and make the working page writable, allowing all subsequent writes in the same interval to execute at full hardware speed without faulting.
```

* **5b [3 pts]**:
```text
False — because modifying overlapping words of the same page without intervening synchronization is a data race, leaving the two write intervals concurrent with no happens-before causal ordering between their diffs.
because Lazy Release Consistency in TreadMarks relies strictly on synchronization operations (locks/barriers) to establish a partial causal order over intervals and diffs; when two processors write to the same word concurrently without synchronization, TreadMarks cannot deterministically order or merge the conflicting diffs, resulting in undefined/inconsistent application behavior.
```
