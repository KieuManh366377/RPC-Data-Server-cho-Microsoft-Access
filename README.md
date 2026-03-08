# RPC Data Server cho Microsoft Access

## Giới thiệu

**RPC Data Server** là một máy chủ **Remote Procedure Call (RPC)** chạy trên TCP, cho phép biến một máy tính cá nhân thông thường thành **máy chủ dịch vụ dữ liệu cho mạng LAN hoặc Internet**.

Hệ thống cho phép các ứng dụng client (Excel, Delphi hoặc các chương trình khác) **đọc và ghi dữ liệu vào cơ sở dữ liệu Microsoft Access thông qua giao thức RPC nhẹ và tốc độ cao**.

Thay vì nhiều máy truy cập trực tiếp file database Access qua mạng, tất cả truy vấn sẽ được xử lý tập trung tại máy chủ.

Điều này giúp:

* tăng độ ổn định hệ thống
* tránh lỗi khóa file khi nhiều người dùng
* giảm nguy cơ hỏng database
* tăng hiệu năng truy cập dữ liệu

Máy chủ có thể chạy liên tục trên một máy tính Windows phổ thông.

---

# Vì sao dự án này được xây dựng

Trong nhiều hệ thống nhỏ sử dụng Microsoft Access, database thường được chia sẻ trực tiếp qua mạng:

```
\\Server\Database.accdb
```

Khi nhiều máy truy cập cùng lúc, các vấn đề phổ biến xảy ra:

* xung đột khóa file
* lỗi ghi dữ liệu
* database bị hỏng
* hiệu năng giảm khi nhiều người dùng

Giải pháp của dự án này là **chuyển tất cả truy vấn database về một máy chủ RPC trung tâm**.

Luồng xử lý sẽ trở thành:

```
Client → RPC Server → ADODB → Database
```

Nhờ đó:

* database chỉ được truy cập tại **một máy duy nhất**
* tránh lỗi chia sẻ file
* hệ thống ổn định hơn
* dễ kiểm soát và mở rộng

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

# Quick Start

1. Tải project hoặc release từ GitHub
2. Chạy chương trình **RPC Server** trên máy chủ
3. Mở port RPC (ví dụ: `9000`)
4. Client kết nối tới server bằng TCP

Ví dụ log khi server chạy:

```
RPC Server started 127.0.0.1:9000
Client connected: 192.168.1.25
RPC Path=/ExecSQL
RPC Path=/Ping
```

---

# Giao thức RPC

Server sử dụng giao thức RPC dạng text đơn giản.

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

### Gọi hàm Plugin

```
/CallDLL|MathPlugin.dll|Multiply|5|6
```

Phản hồi:

```
30
```

---

# Hệ thống Plugin DLL

Server hỗ trợ **Plugin DLL động**, cho phép mở rộng chức năng mà không cần sửa code máy chủ.

Server sẽ:

1. nạp DLL khi cần
2. tìm hàm được export
3. thực thi hàm
4. trả kết quả về client

Nhờ cơ chế plugin, server có thể trở thành **nền tảng mở để phát triển nhiều module khác nhau**.

---

# Plugin mẫu

Dự án cung cấp **plugin mẫu kèm source code (open source)** để giúp lập trình viên hiểu cách hoạt động của hệ thống.

## 1. MathPlugin – Plugin ví dụ đơn giản

Plugin minh họa cách export hàm từ DLL.

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

Ví dụ gọi từ RPC:

```
/CallDLL|SamplePlugin.dll|Add|10|20
```

Kết quả:

```
30
```

Plugin này giúp hiểu:

* cách tạo DLL
* cách export hàm
* cách RPC Server gọi plugin

---

## 2. AccessPlugin – Plugin xử lý ADODB

Plugin thứ hai sử dụng **ADODB** để truy cập database Microsoft Access trực tiếp trên máy chủ.

Plugin cung cấp **một hàm chung cho mọi lệnh SQL ghi dữ liệu**.

```delphi
{
  Hàm ExecSQL để dùng chung cho mọi lệnh SQL:
  INSERT, UPDATE, DELETE,
  CREATE TABLE, DROP TABLE, ALTER TABLE
}

function ExecSQL(DBPath, SQL: PWideChar): Integer; stdcall;
var
  Conn: TADOConnection;
begin
  try
    Conn := GetConnection(DBPath);
    Conn.Execute(SQL);
    Result := 0;
  except
    Result := -1;
  end;
end;
```

Ví dụ RPC gọi plugin:

```
/CallDLL|AccessPlugin.dll|ExecSQL|C:\Data\Test.accdb|INSERT INTO A VALUES(1)
```

Server sẽ:

1. gọi hàm `ExecSQL`
2. thực thi SQL bằng ADODB
3. trả kết quả cho client

---

# Lập trình Client giống như dùng ADODB Local

Một ưu điểm quan trọng của hệ thống là:

Client có thể sử dụng RPC server **giống như gọi database local**.

Ví dụ trong Delphi:

```delphi
ExecSQL('C:\Data\Test.accdb',
        'INSERT INTO Customers VALUES(...)');
```

Thực tế phía sau:

```
Client → RPC → Server → ADODB → Database
```

Nhưng đối với lập trình viên client, cách sử dụng **vẫn giống tiêu chuẩn ADODB của Microsoft**.

Điều này giúp chuyển đổi ứng dụng từ **database local sang RPC server rất dễ dàng**.

---

# Ví dụ Client Delphi

```delphi
Client.IOHandler.WriteLn(
 '/CallDLL|AccessPlugin.dll|ExecSQL|C:\DB\Test.accdb|INSERT INTO A VALUES(1)'
);

Result := Client.IOHandler.ReadLn;
```

---

# Benchmark hiệu năng

Thử nghiệm trên mạng LAN:

```
CPU: Intel i5
RAM: 16GB
SSD
```

Server xử lý khoảng:

```
50 – 200 RPC calls / giây
```

tùy loại truy vấn database.

Do giao thức RPC rất nhẹ nên độ trễ thấp hơn nhiều so với HTTP API.

---

# Cấu trúc Project

```
RPC-Data-Server
│
├─ server
│   RPC Server core
│
├─ plugins
│   Sample plugins
│
├─ examples
│   Client examples
│
└─ docs
    Documentation
```

---

# Ưu điểm của hệ thống

* chạy trên máy tính phổ thông
* giao thức RPC rất nhẹ
* tránh lỗi chia sẻ file Access
* có thể mở rộng bằng plugin
* client lập trình giống database local

---

# Khả năng mở rộng

Kiến trúc server có thể mở rộng thêm:

* SQLite
* MySQL
* PostgreSQL
* xác thực người dùng
* mã hóa kết nối
* hệ thống log và monitoring

---

# Mục tiêu dự án

Xây dựng một **RPC Data Server đơn giản nhưng mạnh mẽ**, cho phép bất kỳ máy tính cá nhân nào cũng có thể hoạt động như một **máy chủ dịch vụ dữ liệu cho nhiều ứng dụng client**.

Giải pháp này đặc biệt phù hợp cho:

* hệ thống Excel nhiều người dùng
* ứng dụng Delphi nội bộ
* hệ thống quản lý nhỏ và vừa

---

# Liên hệ

**Kiều Mạnh (Việt Nam)**

Email: kieumanh366377@gmail.com

Tel: 0929.278.279  
Tel: 0929.278.379

---

License: **MIT**
