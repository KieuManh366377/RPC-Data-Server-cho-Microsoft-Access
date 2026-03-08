
# RPC Data Server cho Microsoft Access

**RPC Data Server cho Microsoft Access** là một **máy chủ trung gian (middleware)** cho phép các ứng dụng như **Microsoft Excel, Delphi, hoặc các client khác** truy cập cơ sở dữ liệu Microsoft Access thông qua giao thức **RPC (Remote Procedure Call)**.

Thay vì nhiều máy cùng mở trực tiếp file Access qua mạng LAN (dễ gây lỗi và hỏng database), hệ thống này tạo một **server trung gian quản lý truy cập dữ liệu**.

---

# Tại sao không nên chia sẻ trực tiếp file Access?

Trong nhiều hệ thống nội bộ nhỏ, người dùng thường:

```
\\Server\Database\database.accdb
```

và nhiều máy mở cùng lúc.

Cách này có thể gây:

* File database bị **corrupt**
* Lỗi **locking**
* Hiệu năng kém khi nhiều user
* Khó kiểm soát truy cập

Theo khuyến nghị của Microsoft, khi số lượng user tăng, nên tách **Data Layer** thành **server xử lý truy cập**.

Hệ thống này thực hiện mô hình:

```
Client
   ↓
RPC Server
   ↓
Plugin
   ↓
ADODB
   ↓
Microsoft Access
```

Ưu điểm:

* Chỉ **1 server truy cập database**
* Client không mở trực tiếp file Access
* Có thể **kiểm soát truy cập**
* Dễ mở rộng

---

# Kiến trúc hệ thống

```
+-------------------+
| Client Application|
| Excel / Delphi    |
+--------+----------+
         |
         | RPC Call
         |
+--------v----------+
|   RPC Data Server |
+--------+----------+
         |
         | Plugin System
         |
+--------v----------+
|      Plugin DLL   |
|  (ADO / Custom)   |
+--------+----------+
         |
         | SQL / Data Access
         |
+--------v----------+
| Microsoft Access  |
|   Database        |
+-------------------+
```

---

# Tính năng chính

* RPC Server TCP tốc độ cao
* Hỗ trợ nhiều client đồng thời
* Hệ thống **Plugin DLL mở rộng**
* Plugin mẫu **ADO cho Access**
* Có thể viết plugin riêng
* Tích hợp dễ dàng với Excel VBA
* Tích hợp Delphi hoặc các ứng dụng khác
* Thiết kế lightweight

---

# Quick Start

## 1. Tải và chạy server

Download release và chạy:

```
RPCServer.exe
```

Server mặc định:

```
Port: 9000
```

Log ví dụ:

```
RPC Server started 127.0.0.1:9000 Threads:32
```

---

## 2. Ví dụ gọi RPC

Client gửi command:

```
/CallDLL|MathPlugin.dll|Multiply|5|6
```

Server sẽ:

1. Load plugin
2. Gọi function
3. Trả kết quả về client

---

# Ví dụ Plugin toán học

Plugin đơn giản:

```delphi
library MathPlugin;

function Multiply(A, B: Integer): Integer; stdcall;
begin
  Result := A * B;
end;

exports
  Multiply;

begin
end.
```

Client gọi:

```
/CallDLL|MathPlugin.dll|Multiply|5|6
```

Kết quả:

```
30
```

---

# Plugin ADO cho Microsoft Access

Plugin mẫu cho phép thực thi SQL trực tiếp.

Ví dụ function:

```delphi
function ExecSQL(DBPath, SQL: PChar): PChar; stdcall;
```

Ví dụ RPC:

```
/CallDLL|AccessADOPlugin.dll|ExecSQL|C:\DB\data.accdb|INSERT INTO Users(Name) VALUES('John')
```

Server sẽ:

1. Plugin mở kết nối ADO
2. Thực thi SQL
3. Trả kết quả

---

# Phát triển Plugin riêng

Người dùng có thể **tự viết Plugin DLL theo tiêu chuẩn của hệ thống**.

Repository cung cấp **plugin mẫu kèm source code** để lập trình viên dễ hiểu cách hoạt động.

Plugin có thể dùng cho:

* Business logic
* Data processing
* Integration
* External API
* Automation

Ví dụ plugin đơn giản:

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

---

# Ví dụ sử dụng từ Excel VBA

```vba
Dim result As String

result = RPC_Call("/CallDLL|MathPlugin.dll|Multiply|10|20")

MsgBox result
```

---

# Cấu trúc dự án

```
RPC-Data-Server-cho-Microsoft-Access
│
├── server
│   RPCServer source code
│
├── plugins
│   Sample plugins
│
├── examples
│   Example clients
│
├── docs
│   Architecture and documentation
│
└── README.md
```

---

# Hiệu năng

Hệ thống được thiết kế cho môi trường **LAN nội bộ**.

Ưu điểm:

* Server quản lý connection
* Giảm conflict file
* Giảm network traffic
* Xử lý đa luồng

---

# Security Considerations

RPC server nên chạy trong **mạng nội bộ**.

Khuyến nghị:

* Không mở port ra internet
* Kiểm soát plugin DLL
* Validate input từ client
* Sử dụng firewall

---

# Use Cases

Phù hợp cho:

* Excel Database Systems
* Small Business Systems
* LAN Data Applications
* Delphi Applications
* Automation Tools

---

# Yêu cầu hệ thống

* Windows
* Delphi Runtime
* Microsoft Access Database Engine (ACE / ADO)

---

# Giấy phép

Open Source.

---

# Tác giả

Kieu Manh

GitHub:

[https://github.com/KieuManh366377](https://github.com/KieuManh366377)

---
.
