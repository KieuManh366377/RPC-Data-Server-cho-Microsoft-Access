# RPC Data Server cho Microsoft Access

## Giới thiệu

**RPC Data Server** là một máy chủ Remote Procedure Call (RPC) chạy trên TCP, cho phép biến một máy tính cá nhân thông thường thành **máy chủ dịch vụ dữ liệu trong mạng LAN hoặc Internet**.

Hệ thống cho phép các ứng dụng client (Excel, ứng dụng Delphi hoặc các chương trình khác) **đọc và ghi dữ liệu vào cơ sở dữ liệu Microsoft Access thông qua giao thức RPC nhẹ và tốc độ cao**.

Thay vì nhiều máy truy cập trực tiếp file database, tất cả truy vấn sẽ đi qua máy chủ. Điều này giúp:

* Tăng độ ổn định hệ thống
* Tránh lỗi khóa file
* Giảm nguy cơ hỏng database
* Cải thiện hiệu năng truy cập

Máy chủ có thể chạy liên tục trên một máy tính Windows phổ thông và phục vụ nhiều client đồng thời.

---

# Kiến trúc hệ thống

```
Client Application
       │
       │ TCP RPC
       ▼
RPC Server
       │
       │ Command Router
       ▼
Plugin DLL System
       │
       ▼
Database Engine
(Microsoft Access)
```

Máy chủ bao gồm các thành phần chính:

* TCP RPC Server
* Bộ phân tích và định tuyến lệnh
* Hệ thống Plugin DLL
* Lớp thực thi truy vấn database
* Hệ thống log và thống kê

---

# Giao thức RPC

Hệ thống sử dụng giao thức RPC dạng text đơn giản.

Ví dụ client gửi yêu cầu:

```
/ExecSQL|SELECT * FROM Customers
```

Server thực thi truy vấn và trả kết quả.

Một số lệnh RPC cơ bản:

**Kiểm tra server**

```
/Ping
```

Phản hồi:

```
PONG
```

**Thực thi SQL**

```
/ExecSQL|SELECT * FROM Products
```

**Gọi hàm plugin**

```
/CallDLL|MathPlugin.dll|Add|10|20
```

---

# Truy cập dữ liệu Microsoft Access

Máy chủ cho phép client thực hiện các thao tác SQL trên database Microsoft Access:

* SELECT
* INSERT
* UPDATE
* DELETE

Toàn bộ truy vấn được thực hiện tại máy chủ, giúp tránh việc nhiều máy truy cập trực tiếp file Access qua mạng, từ đó giảm nguy cơ lỗi và hỏng dữ liệu.

---

# Hệ thống Plugin DLL

Hệ thống hỗ trợ **plugin DLL động**, cho phép mở rộng chức năng mà không cần sửa code máy chủ.

Client có thể gọi hàm trong plugin bằng RPC.

Ví dụ:

```
/CallDLL|MathPlugin.dll|Multiply|5|6
```

Server sẽ nạp DLL, gọi hàm tương ứng và trả kết quả cho client.

---

# Phát triển Plugin riêng

Người dùng có thể **tự viết Plugin DLL theo tiêu chuẩn của hệ thống** để mở rộng chức năng.

Ví dụ plugin có thể:

* xử lý dữ liệu
* thực hiện tính toán
* tạo báo cáo
* tích hợp hệ thống khác
* truy cập database khác

Dự án cung cấp **source code plugin mẫu (open source)** để người dùng hiểu cách hoạt động của plugin và có thể phát triển thêm các module riêng.

Nhờ đó hệ thống có thể mở rộng rất linh hoạt mà không cần thay đổi chương trình server.

---

# Ưu điểm của hệ thống

* Hoạt động trên máy tính phổ thông
* Tốc độ nhanh do sử dụng TCP RPC nhẹ
* Tránh xung đột truy cập database Access
* Có thể mở rộng chức năng bằng Plugin DLL
* Có thể hoạt động trong LAN hoặc Internet

---

# Khả năng mở rộng

Kiến trúc hệ thống cho phép mở rộng trong tương lai như:

* hỗ trợ database khác (SQLite, MySQL…)
* xác thực người dùng
* mã hóa kết nối
* giám sát hiệu năng

---

# Mục tiêu dự án

Mục tiêu của dự án là xây dựng một **máy chủ dữ liệu RPC đơn giản nhưng mạnh mẽ**, cho phép bất kỳ máy tính cá nhân nào cũng có thể hoạt động như một **máy chủ dịch vụ dữ liệu cho nhiều ứng dụng client**.
