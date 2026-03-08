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

# Vì sao dự án này được xây dựng

Trong nhiều hệ thống nhỏ sử dụng Microsoft Access, database thường được chia sẻ trực tiếp qua mạng:

```
\\Server\Database.accdb
```

Khi nhiều máy cùng truy cập file này, các vấn đề thường xảy ra:

* xung đột khóa file
* lỗi ghi dữ liệu
* database bị hỏng
* hiệu năng chậm khi nhiều người dùng

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

# Giao thức RPC

Ví dụ client gửi:

```
/ExecSQL|SELECT * FROM Customers
```

Server thực thi truy vấn và trả kết quả.

---

# Các lệnh RPC cơ bản

### Kiểm tra server

```
/Ping
```

Phản hồi:

```
PONG
```

### Thực thi SQL

```
/ExecSQL|SELECT * FROM Products
```

### Gọi hàm plugin

```
/CallDLL|AccessPlugin.dll|ExecSQL|DBPath|SQL
```

---

# Hệ thống Plugin DLL

Server hỗ trợ **plugin DLL động**, cho phép mở rộng chức năng mà không cần sửa code máy chủ.

Plugin có thể cung cấp:

* xử lý dữ liệu
* tính toán
* tích hợp hệ thống khác
* truy cập database

---

# Ví dụ Plugin xử lý ADODB phía Server

Dự án cung cấp **plugin mẫu open source** sử dụng ADODB để thực thi SQL trên database Microsoft Access.

Plugin cung cấp **một hàm duy nhất** để xử lý mọi lệnh SQL ghi dữ liệu.

```delphi
{
  Hàm ExecSQL dùng chung cho mọi lệnh SQL:
  INSERT, UPDATE, DELETE, CREATE TABLE,
  DROP TABLE, ALTER TABLE

  Đây là một API duy nhất cho mọi thao tác ghi database.
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

Plugin sử dụng **ADODB để truy cập database trực tiếp trên máy chủ**.

---

# Cách sử dụng từ RPC Client

Client chỉ cần gọi RPC:

```
/CallDLL|AccessPlugin.dll|ExecSQL|C:\Data\Test.accdb|INSERT INTO A VALUES(1)
```

Server sẽ:

1. gọi hàm `ExecSQL` trong plugin
2. thực thi SQL bằng ADODB
3. trả kết quả về client

---

# Lập trình Client giống như dùng ADODB Local

Một ưu điểm quan trọng của hệ thống là:

**Client có thể sử dụng RPC server giống như gọi database trên máy local.**

Ví dụ trong Delphi:

```delphi
ExecSQL('C:\Data\Test.accdb',
        'INSERT INTO Customers VALUES(...)');
```

Thực tế phía sau sẽ là:

```
Client → RPC → Server → ADODB → Database
```

Nhưng đối với lập trình viên client, cách sử dụng **vẫn giống tiêu chuẩn ADODB của Microsoft**.

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

tùy loại truy vấn.

---

# Ưu điểm hệ thống

* chạy trên máy tính phổ thông
* giao thức RPC rất nhẹ
* tránh lỗi chia sẻ file Access
* mở rộng chức năng bằng plugin
* client lập trình giống database local

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

Xây dựng một **RPC Data Server đơn giản nhưng mạnh mẽ**, cho phép bất kỳ máy tính cá nhân nào cũng có thể hoạt động như một **máy chủ dịch vụ dữ liệu cho nhiều ứng dụng client**.

---

# Liên hệ

Email: [kieumanh366377@gmail.com](mailto:kieumanh366377@gmail.com)

Tel: 0929.278.279
Tel: 0929.278.379

---

License: MIT
