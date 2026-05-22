# BÁO CÁO ĐỒ ÁN TỐT NGHIỆP

**Tên đề tài:** Xây dựng nền tảng Cờ vua trực tuyến phân tán, ứng dụng kiến trúc Đa ngôn ngữ (Polyglot) với Golang và WebAssembly.

---

## CHƯƠNG 1: MỞ ĐẦU
### 1.1. Đặt vấn đề
Vận hành một hệ thống game cờ vua thời gian thực đòi hỏi giải quyết song song hai bài toán lớn: (1) Sức mạnh tính toán Trí tuệ nhân tạo (AI) và (2) Khả năng duy trì hàng ngàn kết nối đồng thời với độ trễ thấp. Các giải pháp tập trung (Centralized) truyền thống thường gây lãng phí tài nguyên và dễ quá tải. Đồ án này đề xuất giải pháp kiến trúc phân tán: chuyển dịch toàn bộ gánh nặng tính toán AI xuống thiết bị người dùng (Client-side Computing) và sử dụng một máy chủ trung chuyển siêu nhẹ (Lightweight Relay Server) để điều phối kết nối.

### 1.2. Mục tiêu đề tài
* Xây dựng lõi Chess Engine hiệu năng cao bằng C#.
* Ứng dụng WebAssembly (Wasm) để mang cấu trúc xử lý cấp thấp của C# lên trình duyệt Web.
* Xây dựng máy chủ điều phối thời gian thực bằng **Golang**, tối ưu hóa khả năng chịu tải kết nối (Concurrency) trong môi trường tài nguyên phần cứng giới hạn (Cloud Free Tier).
* Triển khai hệ thống đa ngôn ngữ (Polyglot), tách biệt hoàn toàn Logic Game và Logic Mạng.

---

## CHƯƠNG 2: CƠ SỞ LÝ THUYẾT VÀ CÔNG NGHỆ ÁP DỤNG
### 2.1. Ngôn ngữ Golang và Kiến trúc hướng đồng thời (Concurrency)
* **Khái niệm:** Golang (Go) là ngôn ngữ được Google phát triển, biên dịch tĩnh (Statically Typed), sinh ra mã máy trực tiếp (Native Binary) với tốc độ khởi động siêu tốc.
* **Goroutines & Channels:** Cơ chế xử lý đa luồng đặc biệt của Go. Khác với Thread truyền thống của hệ điều hành tốn nhiều Megabyte RAM, một Goroutine chỉ tiêu thụ khoảng 2KB. Điều này cho phép Server duy trì hàng vạn kết nối WebSocket song song chỉ với vài chục Megabyte bộ nhớ.
* **Vai trò trong hệ thống:** Go được chọn làm "Trái tim mạng lưới", đóng vai trò trạm trung chuyển thông điệp và xác thực dữ liệu.

### 2.2. WebAssembly (Wasm) và Client-side Computing
* Khái niệm WebAssembly và vai trò trong việc đưa mã nguồn hệ thống (C/C++, C#) chạy trên trình duyệt với tốc độ tiệm cận mã máy.
* Cơ chế Web Workers để xử lý AI không làm nghẽn luồng giao diện chính (UI Thread).

### 2.3. Lập trình Chess Engine với C#
* Tối ưu hóa mã lệnh cấp thấp (Low-level Optimization): Struct, Bitboard 64-bit, Unsafe pointers.
* Thuật toán tìm kiếm Alpha-Beta Pruning.

---

## CHƯƠNG 3: THIẾT KẾ CƠ SỞ HẠ TẦNG VÀ KIẾN TRÚC HỆ THỐNG
### 3.1. Kiến trúc Đa ngôn ngữ tách biệt (Decoupled Polyglot Architecture)
Hệ thống được thiết kế theo mô hình **Thin Server - Fat Client**:
* **Khối Server (Golang):** Đóng vai trò WebSocket Broker và Security Validator. Tiêu thụ RAM cực thấp, phản hồi I/O tốc độ cao.
* **Khối Client Desktop (WPF - C#):** Chạy Engine Native tận dụng 100% phần cứng máy tính người dùng.
* **Khối Client Web (HTML/JS + Wasm):** Trình duyệt nạp file `.wasm` (biên dịch từ C#) để chạy Engine AI độc lập.

Giao tiếp giữa Client (C#) và Server (Golang) được chuẩn hóa hoàn toàn thông qua định dạng JSON và chuỗi định dạng cờ vua quốc tế (FEN).

### 3.2. Hạ tầng xử lý AI phân tán (Edge Computing)
* Thay vì chạy AI trên Server gây nghẽn CPU, toàn bộ thuật toán tìm kiếm nước đi được phân tán về thiết bị đầu cuối. Server chỉ chịu trách nhiệm đồng bộ trạng thái giữa hai Client.

### 3.3. Cơ chế Khôi phục trạng thái và Chống gian lận (Disaster Recovery & Anti-cheat)
* **Server-side Validation:** Mặc dù Server Golang không chạy AI tìm kiếm, nó vẫn tích hợp một module trọng tài tĩnh (Stateless Validator) để kiểm tra tính hợp lệ của nước đi (Pseudo-legal move check) nhằm chống gian lận.
* **Chữ ký số điện tử (Digital Signature):** Server Golang sử dụng thuật toán HMAC-SHA256 hoặc JWT để ký tên lên chuỗi FEN. Client lưu trữ trạng thái này và dùng nó để khôi phục ván đấu nếu mạng bị đứt hoặc máy chủ khởi động lại.

---

## CHƯƠNG 4: TRIỂN KHAI THỰC TẾ VÀ TỐI ƯU HÓA HIỆU NĂNG
### 4.1. Tối ưu hóa WebSocket Broker với Golang
* Sử dụng thư viện `gorilla/websocket` của Go để quản lý kết nối.
* Áp dụng mô hình Hub & Client: Mỗi trận đấu là một "Phòng" (Room). Tín hiệu (Channel) được sử dụng để định tuyến (route) thông điệp nước đi từ người chơi A sang người chơi B mà không xảy ra tình trạng Race Condition.
* Đánh giá mức tiêu thụ RAM (Memory Footprint) của Server Go khi chịu tải 1000 kết nối đồng thời trên hạ tầng Render Free (512MB RAM).

### 4.2. Chuyển đổi mã C# sang WebAssembly
* Quá trình biên dịch thư viện Chess.Core bằng C# thành tệp nhị phân `.wasm`.
* Tích hợp Web Worker kết nối với file `.wasm` để đảm bảo luồng giao diện Web hoạt động mượt mà ở 60FPS khi AI đang tính toán.

### 4.3. Sự đánh đổi ngôn ngữ (The Polyglot Trade-off)
*(Phần này thể hiện tư duy phản biện của kỹ sư)*
* Để Server Golang có thể xác thực tính hợp lệ của nước đi, một phần logic luật cờ vua (Move Generation/Validation) đã được tái hiện lại bằng Golang, trong khi thuật toán AI phức tạp (Search/Evaluation) vẫn giữ nguyên bằng C# tại Client. Điều này tạo ra sự nhân bản logic (Code Duplication) nhưng bù lại giải quyết triệt để bài toán hiệu năng mạng.

---

## CHƯƠNG 5: KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN
### 5.1. Kết quả đạt được
* Xây dựng thành công hệ thống cờ vua phân tán với chi phí máy chủ tiệm cận 0 nhờ áp dụng tư duy Edge Computing.
* Khai thác thành công thế mạnh tuyệt đối của hai ngôn ngữ: C# (chuyên trị xử lý toán học bitwise, cấp thấp cho Engine) và Golang (chuyên trị xử lý I/O mạng đồng thời).
* Khắc phục hiệu quả các rào cản của môi trường Cloud miễn phí nhờ khả năng khởi động lạnh (Cold-start) siêu tốc của Golang và cơ chế phục hồi FEN mã hóa.

### 5.2. Hướng phát triển tương lai
* Triển khai bộ cân bằng tải (Load Balancer) để mở rộng hệ thống Golang theo chiều ngang (Scale-out).
* Áp dụng gRPC hoặc Protocol Buffers để nén dữ liệu truyền tải qua mạng thay vì dùng văn bản JSON thuần túy nhằm tối ưu hơn nữa tốc độ phản hồi.
