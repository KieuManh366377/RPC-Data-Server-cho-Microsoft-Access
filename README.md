# RPC Data Server cho Microsoft Access

## Giới thiệu

**RPC Data Server** là một máy chủ Remote Procedure Call (RPC) chạy trên TCP, cho phép biến một máy tính cá nhân thông thường thành **máy chủ dịch vụ dữ liệu cho mạng LAN hoặc Internet**.

Hệ thống cho phép các ứng dụng client (Excel, Delphi hoặc các chương trình khác) **đọc và ghi dữ liệu vào cơ sở dữ liệu Microsoft Access thông qua giao thức RPC nhẹ và tốc độ cao**.

Thay vì nhiều máy mở trực tiếp file database Access qua mạng, tất cả truy vấn sẽ được xử lý tại máy chủ.

Điều này giúp:

* tăng độ ổn định
* tránh lỗi khóa file
* giảm nguy cơ hỏng database
* tăng hiệu năng truy cập

Máy chủ có thể chạy liên tục trên một máy tính Windows phổ thông.

---

# Kiến trúc hệ thống

```
           Client Applications
        ┌──────────┬──────────┬──────────┐
        │          │          │          │
      Excel      Delphi     Script     Other Apps
        │          │          │
        └──────────┴──────────┴──────────┘
                     │
                     │ TCP RPC
                     ▼
              RPC Data Server
                     │
           ┌─────────┴─────────┐
           │                   │
      Command Router      Plugin System
           │                   │
           ▼                   ▼
     SQL Execution        Plugin DLL
           │
           ▼
    Microsoft Access DB
```

---

# Giao thức RPC

Hệ thống sử dụng giao thức RPC dạng text đơn giản chạy trên TCP socket.

Ví dụ client gửi:

```
/ExecSQL|SELECT * FROM Customers
```

Server thực thi truy vấn và trả kết quả.

---

# Các lệnh RPC cơ bản

### Kiểm tra kết nối

```
/Ping
```

Phản hồi:

```
PONG
```

---

### Thực thi SQL

```
/ExecSQL|SELECT * FROM Products
```

---

### Gọi hàm plugin

```
/CallDLL|MathPlugin.dll|Add|10|20
```

Phản hồi:

```
30
```

---

# Truy cập dữ liệu Microsoft Access

Server cho phép client thực hiện các thao tác SQL:

* SELECT
* INSERT
* UPDATE
* DELETE

Toàn bộ truy vấn được thực hiện tại máy chủ.

Điều này giúp tránh việc nhiều máy truy cập trực tiếp file Access:

```
\\Server\Database.accdb
```

và giảm nguy cơ lỗi hoặc hỏng database.

---

# Hệ thống Plugin DLL

Server hỗ trợ **plugin DLL động**, cho phép mở rộng chức năng mà không cần sửa code máy chủ.

Plugin có thể cung cấp các chức năng như:

* tính toán
* xử lý dữ liệu
* tạo báo cáo
* tích hợp hệ thống khác

Ví dụ RPC gọi plugin:

```
/CallDLL|MathPlugin.dll|Multiply|5|6
```

---

# Phát triển Plugin riêng

Người dùng có thể **tự viết Plugin DLL theo tiêu chuẩn của hệ thống**.

Dự án cung cấp **plugin mẫu kèm source code (open source)** để lập trình viên hiểu cách hoạt động và phát triển thêm module riêng.

Ví dụ cấu trúc plugin đơn giản:

```delphi
library SamplePlugin;

function Add(A, B: Integer): Integer; stdcall;
begin
  Result := A + B;
end;

exports
  Add;

begin
end.
```

Sau khi compile DLL, đặt file vào thư mục:

```
Plugins/
```

Server sẽ có thể gọi hàm trong DLL thông qua RPC.

---

# Ví dụ Client Delphi

```delphi
Client.IOHandler.WriteLn('/Ping');
Result := Client.IOHandler.ReadLn;
```

Ví dụ gọi SQL:

```delphi
Client.IOHandler.WriteLn('/ExecSQL|SELECT * FROM Customers');
Result := Client.IOHandler.ReadLn;
```

---

# Ví dụ Client Excel

Excel có thể gửi RPC request để truy vấn dữ liệu.

Ví dụ:

```
Send: /ExecSQL|SELECT * FROM Orders
Receive: Result data
```

---

# Demo chạy server

Ví dụ log khi server chạy:

```
RPC Server started 127.0.0.1:9000
Client connected: 192.168.1.25
RPC Path=/ExecSQL
RPC Path=/Ping
```

Server có thể phục vụ nhiều client cùng lúc.

---

# Benchmark hiệu năng

Trong thử nghiệm nội bộ trên mạng LAN:

```
CPU: Intel i5
RAM: 16GB
SSD
```

Server có thể xử lý:

```
50 – 200 RPC calls / giây
```

tùy loại truy vấn database.

Nhờ giao thức RPC nhẹ nên độ trễ thấp hơn nhiều so với HTTP API.

---

# Hướng dẫn viết Plugin DLL

Các bước tạo plugin mới:

1. Tạo project **DLL** trong Delphi
2. Viết hàm cần cung cấp
3. Export hàm
4. Compile thành DLL
5. Copy DLL vào thư mục `Plugins`

Ví dụ gọi từ client:

```
/CallDLL|MyPlugin.dll|FunctionName|Param1|Param2
```

Server sẽ:

1. nạp DLL
2. tìm hàm
3. thực thi
4. trả kết quả

---

# Ưu điểm hệ thống

* chạy trên máy tính phổ thông
* giao thức RPC rất nhẹ
* tránh lỗi chia sẻ file Access
* mở rộng chức năng bằng plugin
* hoạt động trong LAN hoặc Internet

---

# Khả năng mở rộng

Kiến trúc server cho phép mở rộng thêm:

* SQLite
* MySQL
* PostgreSQL
* xác thực người dùng
* mã hóa kết nối
* giám sát hiệu năng

---

# Mục tiêu dự án

Mục tiêu của dự án là xây dựng một **RPC Data Server đơn giản nhưng mạnh mẽ**, cho phép bất kỳ máy tính cá nhân nào cũng có thể hoạt động như một **máy chủ dịch vụ dữ liệu cho nhiều ứng dụng client**.

Điều này giúp xây dựng hệ thống nhiều người dùng mà không cần hạ tầng server phức tạp.

---

# License

MIT License
