# SQL Server Interview Questions - Senior Level

---

## Câu 1: Index là gì?

### Định nghĩa

Index là một **cấu trúc dữ liệu** được tạo trên một hoặc nhiều columns của table, giúp SQL Server **tìm kiếm dữ liệu nhanh hơn** mà không cần quét toàn bộ bảng. Tương tự như **mục lục của một cuốn sách** -- thay vì đọc từ trang 1 đến trang 500 để tìm một chủ đề, bạn chỉ cần mở mục lục, tra cứu tên chủ đề, và nhảy thẳng đến số trang tương ứng.

### Cấu trúc bên trong: B-Tree (Balanced Tree)

SQL Server sử dụng cấu trúc **B-Tree** cho hầu hết các loại index:

```
            [Root Node]
           /     |     \
    [Inter-1] [Inter-2] [Inter-3]     ← Intermediate Nodes
    /   \      /   \      /   \
  [L1] [L2] [L3] [L4]  [L5] [L6]    ← Leaf Nodes
```

- **Root Node**: điểm bắt đầu, chứa các key range để định hướng tìm kiếm
- **Intermediate Nodes**: các tầng trung gian, thu hẹp phạm vi tìm kiếm
- **Leaf Nodes**: tầng lá, chứa dữ liệu thực tế hoặc con trỏ đến dữ liệu

Với B-Tree, mỗi lần tìm kiếm chỉ cần duyệt **O(log n)** nodes thay vì **O(n)** rows. Một bảng 1 triệu rows chỉ cần khoảng 3-4 lần nhảy (levels) là tìm được row cần thiết.

### Clustered Index vs Non-Clustered Index

| Đặc điểm | Clustered Index | Non-Clustered Index |
|---|---|---|
| Leaf node chứa | **Dữ liệu thực tế** (data rows) | **Con trỏ** (RID hoặc Clustered Key) |
| Số lượng / table | **Chỉ 1** | **Tối đa 999** |
| Thứ tự vật lý | Quyết định thứ tự lưu trữ vật lý của data | Không ảnh hưởng thứ tự vật lý |
| Kích thước | Chính là bảng dữ liệu | Cấu trúc riêng biệt, nhỏ hơn |

```
-- Clustered Index (leaf = data rows)
Leaf: [1, "Nguyen Van A", "HN"] → [2, "Tran Thi B", "HCM"] → [3, "Le Van C", "DN"]

-- Non-Clustered Index (leaf = pointers)
Leaf: ["HCM" → Row 2] → ["DN" → Row 3] → ["HN" → Row 1]
            ↓                    ↓                  ↓
     (quay lại Clustered Index hoặc RID để lấy full row)
```

### Trade-offs

**Lợi ích:**
- Tăng tốc **SELECT** đáng kể -- từ quét toàn bộ bảng xuống chỉ vài IO operations
- Hỗ trợ **ORDER BY**, **GROUP BY**, **JOIN** hiệu quả hơn

**Chi phí:**
- **INSERT**: phải chèn thêm entry vào mọi index liên quan
- **UPDATE**: nếu update column nằm trong index, phải cập nhật cả data lẫn index
- **DELETE**: phải xóa entry tương ứng trong tất cả indexes
- **Storage**: mỗi index chiếm thêm dung lượng disk (có thể lên tới 20-40% kích thước bảng gốc)

**Ví dụ thực tế:** Một bảng `Orders` 10 triệu rows có 8 indexes. Mỗi lần INSERT 1 row, SQL Server phải ghi vào 9 nơi (1 data + 8 indexes). Nếu batch insert 100k rows/ngày, chi phí maintain index là đáng kể.

> **Quy tắc**: Index giống như "đánh đổi thời gian ghi để lấy thời gian đọc". Phù hợp khi hệ thống **đọc nhiều hơn ghi** (OLTP thông thường: 80% read, 20% write).

---

## Câu 2: Có bao nhiêu loại index?

SQL Server hỗ trợ nhiều loại index, mỗi loại phục vụ mục đích khác nhau:

### 1. Clustered Index

```sql
-- Mac dinh, Primary Key tao Clustered Index
CREATE TABLE Employees (
    EmployeeId INT IDENTITY(1,1) PRIMARY KEY, -- Clustered Index tu dong
    FullName NVARCHAR(200),
    Email VARCHAR(255)
);

-- Hoac tao tuong minh
CREATE CLUSTERED INDEX CIX_Employees_EmployeeId
ON Employees(EmployeeId);
```

- **Chỉ 1 per table** vì nó quyết định thứ tự lưu trữ vật lý của dữ liệu
- **Nên chọn column**: narrow (nhỏ), unique, ever-increasing (IDENTITY, NEWSEQUENTIALID)
- **Tránh dùng GUID (NEWID())** làm clustered key vì random → page splits → fragmentation

### 2. Non-Clustered Index

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Email
ON Employees(Email);
```

- Tối đa **999** non-clustered indexes per table
- Tạo cấu trúc B-Tree riêng biệt, leaf node chứa:
  - **Clustered Key** (nếu bảng có clustered index) để quay lại lấy data
  - **RID** - Row Identifier (nếu bảng là heap, không có clustered index)
- Mỗi key lookup từ non-clustered về clustered index là **1 random I/O** → đắt nếu nhiều rows

### 3. Unique Index

```sql
CREATE UNIQUE INDEX UX_Employees_Email
ON Employees(Email);
```

- Đảm bảo **không có giá trị trùng lặp** trong column(s)
- SQL Server cho phép **chỉ 1 NULL** trong Unique Index (vì coi NULL = NULL khi so sánh uniqueness)
- Muốn nhiều NULLs + unique cho non-null → dùng Filtered Index (xem bên dưới)
- `UNIQUE CONSTRAINT` thực chất tạo Unique Index phía sau

### 4. Composite Index (Multi-column Index)

```sql
CREATE INDEX IX_Orders_Status_Date
ON Orders(Status, OrderDate DESC, CustomerId);
```

**Leftmost Prefix Rule** -- rất quan trọng:

```
Index: (Status, OrderDate, CustomerId)

-- SỬ DỤNG ĐƯỢC index:
WHERE Status = 'Active'                                    -- dùng cột 1
WHERE Status = 'Active' AND OrderDate > '2024-01-01'      -- dùng cột 1, 2
WHERE Status = 'Active' AND OrderDate > '2024-01-01'
      AND CustomerId = 5                                   -- dùng cả 3 cột

-- KHÔNG SỬ DỤNG ĐƯỢC index:
WHERE OrderDate > '2024-01-01'                  -- thiếu cột 1 (Status)
WHERE CustomerId = 5                            -- thiếu cột 1, 2
WHERE OrderDate > '2024-01-01' AND CustomerId = 5  -- thiếu cột 1
```

**Thứ tự columns trong composite index rất quan trọng:**
- Đặt column có **equality filter** (=) trước
- Đặt column có **range filter** (>, <, BETWEEN) sau
- Đặt column dùng cho **ORDER BY** cuối cùng

### 5. Covering Index / INCLUDE Columns

```sql
CREATE INDEX IX_Orders_CustomerId
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount, Status);
```

- **INCLUDE columns** được lưu ở **leaf level** của index nhưng **không nằm trong key**
- Giúp **tránh Key Lookup** hoàn toàn -- query chỉ cần đọc index mà không cần quay lại table
- INCLUDE columns **không ảnh hưởng thứ tự sắp xếp** của index
- Khi nào dùng: query cần thêm vài columns ngoài key columns để SELECT

```sql
-- Query này được "cover" hoàn toàn bởi index trên:
SELECT OrderDate, TotalAmount, Status
FROM Orders
WHERE CustomerId = 123;
-- Execution plan sẽ hiện: Index Seek (không có Key Lookup)
```

### 6. Filtered Index

```sql
-- Chỉ index những orders đang active
CREATE INDEX IX_Orders_Active
ON Orders(OrderDate, CustomerId)
WHERE Status = 'Active';

-- Unique index cho phép nhiều NULLs
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN)
WHERE SSN IS NOT NULL;
```

- Chỉ index **một subset** của rows thỏa điều kiện WHERE
- **Nhỏ hơn**, **nhanh hơn** maintain, **ít tốn storage**
- Rất hữu ích khi phần lớn queries chỉ quan tâm một phần dữ liệu (VD: chỉ records active, chỉ đơn hàng chưa xử lý)

### 7. Columnstore Index

```sql
-- Clustered Columnstore (thay thế toàn bộ storage của table)
CREATE CLUSTERED COLUMNSTORE INDEX CCIX_FactSales
ON FactSales;

-- Non-Clustered Columnstore (thêm vào table có sẵn)
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCIX_Orders_Analytics
ON Orders(OrderDate, ProductId, Quantity, TotalAmount);
```

- Lưu trữ dữ liệu **theo cột** thay vì theo hàng
- **Nén dữ liệu** rất tốt (10x compression ratio là bình thường)
- **Batch mode execution**: xử lý hàng ngàn rows cùng lúc thay vì từng row
- Phù hợp cho **analytics, reporting, data warehouse** (OLAP)
- Không phù hợp cho **point lookups** (tìm 1 row cụ thể)

### 8. Full-Text Index

```sql
CREATE FULLTEXT CATALOG ftCatalog AS DEFAULT;

CREATE FULLTEXT INDEX ON Articles(Title, Content)
KEY INDEX PK_Articles ON ftCatalog;

-- Sử dụng:
SELECT * FROM Articles
WHERE CONTAINS(Content, '"SQL Server" NEAR "performance"');

SELECT * FROM Articles
WHERE FREETEXT(Content, N'tối ưu hiệu suất truy vấn');
```

- Tìm kiếm **nội dung văn bản** phức tạp: từ đồng nghĩa, cụm từ, gần nhau
- SQL Server dùng **inverted index** riêng biệt, không phải B-Tree

### 9. Spatial Index

```sql
CREATE SPATIAL INDEX SIX_Stores_Location
ON Stores(GeoLocation);
```

- Dành cho dữ liệu **geography/geometry**: tọa độ GPS, vùng, đường
- Sử dụng cấu trúc grid hierarchy để chia không gian

### 10. XML Index

```sql
CREATE PRIMARY XML INDEX PIX_Config
ON Settings(XmlData);

CREATE XML INDEX SIX_Config_Path
ON Settings(XmlData)
USING XML INDEX PIX_Config FOR PATH;
```

- Primary XML Index: shred XML thành relational format nội bộ
- Secondary XML Index: tối ưu cho PATH, VALUE, hoặc PROPERTY queries

### Tong hop nhanh

| Loai | Use Case Chinh |
|---|---|
| Clustered | Physical order, PK, range queries |
| Non-Clustered | Tim kiem tren non-PK columns |
| Unique | Enforce uniqueness |
| Composite | Filter nhieu columns |
| Covering/INCLUDE | Tranh Key Lookup |
| Filtered | Subset du lieu, sparse columns |
| Columnstore | Analytics, aggregation, warehouse |
| Full-Text | Tim kiem noi dung van ban |
| Spatial | Du lieu dia ly |
| XML | Du lieu XML |

---

## Cau 3: Gender (bit 1-0 / string) - Co nen danh index khong?

### Cau tra loi ngan: KHONG NEN danh index don le cho Gender

### Ly do: Low Cardinality va Selectivity thap

**Selectivity** la ty le so gia tri phan biet so voi tong so rows:

```
Selectivity = Number of Distinct Values / Total Rows

Gender:  2 / 1,000,000 = 0.000002  → Cực kỳ thấp
Email:   1,000,000 / 1,000,000 = 1.0  → Cực kỳ cao (lý tưởng)
Status:  5 / 1,000,000 = 0.000005  → Thấp
```

Khi selectivity thấp, index trả về **quá nhiều rows** (50% nếu Male/Female đồng đều). SQL Server Query Optimizer sẽ **tự động bỏ qua index** và chọn **Table Scan** (hoặc Clustered Index Scan) vì:

```
-- Giả sử 1 triệu rows, 50% Male, 50% Female
-- Dùng index: 500,000 Index Seeks + 500,000 Key Lookups = 1,000,000 random I/O
-- Table scan: 1 sequential scan qua toàn bộ table = nhanh hơn nhiều
```

**Ngưỡng selectivity**: SQL Server thường chọn index khi query trả về khoảng **< 15-30%** tổng rows (còn tùy vào nhiều yếu tố khác). Gender với 50/50 sẽ không bao giờ đạt ngưỡng này.

### Khi nào CÓ THỂ đánh index cho Gender?

**Trường hợp 1: Filtered Index khi dữ liệu lệch (skewed data)**

```sql
-- 95% Male, chỉ 5% Female → filter Female có selectivity cao
CREATE INDEX IX_Employees_Gender_Female
ON Employees(Gender, HireDate)
WHERE Gender = 'Female';

-- Query này sẽ dùng index:
SELECT * FROM Employees
WHERE Gender = 'Female' AND HireDate > '2024-01-01';
```

**Trường hợp 2: Composite Index kết hợp với columns khác**

```sql
-- Gender kết hợp với columns có selectivity cao
CREATE INDEX IX_Employees_Gender_DeptId_Salary
ON Employees(DepartmentId, Gender, Salary);

-- Query này có thể dùng index hiệu quả:
SELECT * FROM Employees
WHERE DepartmentId = 10 AND Gender = 'Male' AND Salary > 50000;
```

**Trường hợp 3: Covering Index**

```sql
CREATE INDEX IX_Employees_Gender_Cover
ON Employees(Gender)
INCLUDE (FullName, Email);

-- Nếu query chỉ cần các columns trong index → Index-only scan, không Key Lookup
SELECT FullName, Email FROM Employees WHERE Gender = 'Female';
-- Vẫn scan nhiều rows nhưng tránh được random I/O về clustered index
```

### bit vs string cho Gender

```sql
-- bit: 1 byte cho mỗi 8 bit columns (tiết kiệm)
Gender BIT -- 0 hoặc 1

-- string: tốn hơn nhưng đọc được
Gender VARCHAR(10) -- 'Male', 'Female'
Gender CHAR(1) -- 'M', 'F'
```

Dù dùng kiểu dữ liệu nào, **vấn đề cốt lõi là selectivity thấp** → index đơn lẻ không hiệu quả. Tuy nhiên, `BIT` hoặc `CHAR(1)` giúp **Composite Index nhỏ hơn** (vì key size nhỏ), nên ưu tiên kiểu dữ liệu nhỏ.

### Kết luận

| Tình huống | Đánh index? |
|---|---|
| Index đơn lẻ trên Gender | **Không** |
| Dữ liệu lệch (5/95%) + Filtered Index | **Có thể** |
| Composite Index với columns khác | **Có** |
| Covering Index tránh Key Lookup | **Xem xét** |

---

## Câu 4: Có đánh được index trên trường Nullable không?

### Câu trả lời: CÓ - SQL Server hoàn toàn hỗ trợ

SQL Server **lưu trữ giá trị NULL trong index B-Tree** bình thường. NULL được coi là giá trị nhỏ nhất, nên:
- Với index **ASC**: các NULL entries nằm ở **đầu** của index
- Với index **DESC**: các NULL entries nằm ở **cuối** của index

### NULL và Index Seek

```sql
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    DiscontinuedDate DATETIME NULL,
    ProductName NVARCHAR(200)
);

CREATE INDEX IX_Products_DiscontinuedDate
ON Products(DiscontinuedDate);
```

```sql
-- CẢ HAI query đều CÓ THỂ sử dụng Index Seek:
SELECT * FROM Products WHERE DiscontinuedDate IS NULL;
-- → Index Seek trên phần đầu của index (vì NULL nằm đầu với ASC)

SELECT * FROM Products WHERE DiscontinuedDate IS NOT NULL;
-- → Index Seek (range scan phần không NULL)

SELECT * FROM Products WHERE DiscontinuedDate = '2024-06-15';
-- → Index Seek bình thường
```

### Filtered Index tối ưu cho Nullable columns

Nếu phần lớn rows có giá trị NULL và query thường chỉ quan tâm non-null:

```sql
-- Index đầy đủ: 1 triệu rows, 900k NULL, 100k có giá trị
CREATE INDEX IX_Products_DiscontinuedDate
ON Products(DiscontinuedDate);
-- Kích thước: lớn, chứa cả 900k NULL entries không cần thiết

-- Filtered Index: chỉ 100k rows
CREATE INDEX IX_Products_DiscontinuedDate_NotNull
ON Products(DiscontinuedDate)
WHERE DiscontinuedDate IS NOT NULL;
-- Kích thước: nhỏ hơn 10x, nhanh hơn maintain
```

### Unique Index và NULL - Đặc biệt quan trọng

```sql
-- Unique Index chỉ cho phép TỐI ĐA 1 NULL
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN);

INSERT INTO Employees (SSN) VALUES (NULL);  -- OK
INSERT INTO Employees (SSN) VALUES (NULL);  -- LỖI! Duplicate key
```

SQL Server coi **NULL = NULL** trong ngữ cảnh Unique Index (khác với phép so sánh thông thường nơi NULL = NULL trả về UNKNOWN).

**Giải pháp: Filtered Unique Index**

```sql
-- Cho phép nhiều NULLs, nhưng đảm bảo unique cho các giá trị non-null
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN)
WHERE SSN IS NOT NULL;

INSERT INTO Employees (SSN) VALUES (NULL);           -- OK
INSERT INTO Employees (SSN) VALUES (NULL);           -- OK (không bị kiểm tra)
INSERT INTO Employees (SSN) VALUES ('123-45-6789');  -- OK
INSERT INTO Employees (SSN) VALUES ('123-45-6789');  -- LỖI! Duplicate
```

### ANSI_NULLS và ảnh hưởng đến queries

```sql
SET ANSI_NULLS ON; -- Mặc định, tuân thủ chuẩn SQL
-- WHERE Column = NULL  → không trả về gì (sai)
-- WHERE Column IS NULL → đúng

SET ANSI_NULLS OFF; -- Không khuyến khích, sẽ bị loại bỏ trong tương lai
-- WHERE Column = NULL  → hoạt động (nhưng không nên dùng)
```

### Best Practices

| Tình huống | Khuyến nghị |
|---|---|
| Column phần lớn NULL, query filter IS NOT NULL | **Filtered Index WHERE col IS NOT NULL** |
| Cần unique nhưng cho phép nhiều NULLs | **Filtered Unique Index WHERE col IS NOT NULL** |
| Column ít NULL, query cả NULL lẫn non-null | **Index bình thường** |
| Column trong composite index có thể NULL | **Đặt nullable column sau** trong index key |

---

## Câu 5: WHERE chậm thì đánh index - SELECT nhanh hơn đúng không?

### Câu trả lời: KHÔNG PHẢI LÚC NÀO CŨNG ĐÚNG

Đây là **misconception phổ biến nhất** về indexing. Đánh index không phải "viên đạn bạc" -- có nhiều trường hợp index tồn tại nhưng **hoàn toàn vô dụng**.

### 1. Non-SARGable Queries (Search ARGument-able)

**SARGable** là khả năng SQL Server dùng index để tìm kiếm. Các trường hợp **KHÔNG SARGable**:

#### a) Function trên column

```sql
-- KHÔNG dùng index (scan toàn bộ)
SELECT * FROM Orders
WHERE YEAR(OrderDate) = 2024;
-- SQL Server phải tính YEAR() cho TỪNG ROW để so sánh

-- FIX: viết lại thành range query
SELECT * FROM Orders
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
-- Index Seek trên OrderDate

-- Tương tự:
WHERE UPPER(Name) = 'NGUYEN VAN A'     -- KHÔNG SARGable
WHERE Name = 'Nguyen Van A'             -- SARGable (dùng collation)

WHERE DATEDIFF(DAY, CreateDate, GETDATE()) < 30  -- KHÔNG
WHERE CreateDate > DATEADD(DAY, -30, GETDATE())  -- SARGable
```

#### b) LIKE với wildcard ở đầu

```sql
WHERE Name LIKE '%abc'     -- Full scan, KHÔNG dùng index
WHERE Name LIKE '%abc%'    -- Full scan, KHÔNG dùng index
WHERE Name LIKE 'abc%'     -- Index Seek (SARGable)
```

#### c) Implicit Conversion (chuyển đổi ngầm định)

```sql
-- Column là VARCHAR, nhưng so sánh với INT
-- PhoneNumber VARCHAR(20), có index
SELECT * FROM Customers
WHERE PhoneNumber = 0912345678;
-- SQL Server phải CONVERT toàn bộ PhoneNumber sang INT để so sánh → SCAN

-- FIX: truyền đúng kiểu dữ liệu
SELECT * FROM Customers
WHERE PhoneNumber = '0912345678';
-- Index Seek
```

**Quy tắc Data Type Precedence**: SQL Server convert kiểu có độ ưu tiên thấp sang kiểu có độ ưu tiên cao. VARCHAR < INT, nên VARCHAR bị convert → scan toàn bộ column.

#### d) Phép phủ định

```sql
WHERE Status != 'Active'       -- Thường scan (trả về nhiều rows)
WHERE Status NOT IN ('A','B')   -- Thường scan
WHERE NOT EXISTS (...)          -- Tùy trường hợp
```

#### e) OR conditions

```sql
-- Có thể không dùng index hiệu quả
SELECT * FROM Orders
WHERE CustomerId = 123 OR ProductId = 456;

-- FIX: dùng UNION
SELECT * FROM Orders WHERE CustomerId = 123
UNION ALL
SELECT * FROM Orders WHERE ProductId = 456;
-- Mỗi query dùng được index riêng
```

### 2. Selectivity thấp - Optimizer chọn Scan

```sql
-- Index tồn tại trên Status, nhưng chỉ có 3 giá trị phân biệt
-- Status = 'Active' trả về 80% rows
SELECT * FROM Orders WHERE Status = 'Active';
-- Optimizer: "Index Seek 800k rows + 800k Key Lookups > Table Scan"
-- → Chọn Table Scan (đúng)
```

### 3. Key Lookup quá đắt

```sql
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);

SELECT CustomerId, OrderDate, TotalAmount, ShipAddress, Notes
FROM Orders
WHERE CustomerId = 123;
-- Tìm được 500 rows qua index → 500 Key Lookups về Clustered Index
-- Mỗi Key Lookup là 1 random I/O → chậm

-- FIX: Covering Index
CREATE INDEX IX_Orders_CustomerId
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount, ShipAddress, Notes);
-- 0 Key Lookups → nhanh hơn nhiều
```

### 4. Parameter Sniffing

```sql
CREATE PROCEDURE GetOrders @Status VARCHAR(20)
AS
    SELECT * FROM Orders WHERE Status = @Status;

-- Lần đầu gọi với 'Cancelled' (1% rows) → plan dùng Index Seek
EXEC GetOrders 'Cancelled';

-- Lần sau gọi với 'Active' (80% rows) → REUSE plan cũ → Index Seek + 800k Key Lookups
EXEC GetOrders 'Active';
-- Cực kỳ chậm! Plan không phù hợp với giá trị này
```

### 5. Quá nhiều Indexes → Write chậm

```sql
-- Bảng Orders có 15 indexes
INSERT INTO Orders (...) VALUES (...);
-- SQL Server phải ghi vào: 1 Clustered Index + 15 Non-Clustered Indexes = 16 write operations

-- Mỗi UPDATE trên column nằm trong index phải update cả index
UPDATE Orders SET Status = 'Shipped' WHERE OrderId = 123;
-- Nếu Status nằm trong 5 indexes → 5 index updates
```

### Kết luận: Luôn xem Execution Plan

```sql
-- Bật Actual Execution Plan trước và sau khi đánh index
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Xem Execution Plan
-- Kiểm tra: Scan vs Seek, Key Lookup có không, Estimated vs Actual rows
-- So sánh logical reads trước/sau
```

**Quy trình đúng:**
1. Xác định query chậm (qua monitoring hoặc Query Store trong SQL Server)
2. Xem **Execution Plan** hiện tại
3. Phân tích **WHERE, JOIN, ORDER BY** đang dùng columns nào
4. Kiểm tra query có **SARGable** không -- fix query trước
5. Tạo index phù hợp (composite, covering, filtered)
6. So sánh Execution Plan mới -- xác nhận cải thiện
7. Monitor **write performance** để đảm bảo không bị ảnh hưởng quá nhiều

---

## Câu 6: Câu lệnh SELECT và thứ tự xử lý logic

### Thứ tự VIẾT vs Thứ tự XỬ LÝ LOGIC

Đây là **một trong những kiến thức nền tảng quan trọng nhất** của SQL. Thứ tự viết và thứ tự xử lý **hoàn toàn khác nhau**:

```
THỨ TỰ VIẾT (Syntax)          THỨ TỰ XỬ LÝ LOGIC (Execution)
─────────────────              ─────────────────────────────
SELECT          (5)    ←──    1. FROM + JOINs
DISTINCT        (6)    ←──    2. ON (điều kiện JOIN)
TOP             (8)    ←──    3. WHERE
  columns...           ←──    4. GROUP BY
FROM            (1)    ←──    5. HAVING
JOIN            (1)    ←──    6. SELECT
ON              (2)    ←──    7. DISTINCT
WHERE           (3)    ←──    8. ORDER BY
GROUP BY        (4)    ←──    9. TOP / OFFSET-FETCH
HAVING          (5)
ORDER BY        (7)
OFFSET-FETCH    (8)
```

### Chi tiết từng bước

| Bước | Mệnh đề | Chức năng |
|------|---------|-----------|
| 1 | **FROM + JOINs** | Xác định nguồn dữ liệu, thực hiện phép JOIN tạo Virtual Table |
| 2 | **ON** | Áp dụng điều kiện JOIN, lọc các rows match |
| 3 | **WHERE** | Lọc rows theo điều kiện (trước khi group) |
| 4 | **GROUP BY** | Nhóm các rows có cùng giá trị thành các groups |
| 5 | **HAVING** | Lọc groups theo điều kiện aggregate |
| 6 | **SELECT** | Chọn columns, tính toán expressions, alias |
| 7 | **DISTINCT** | Loại bỏ các rows trùng lặp |
| 8 | **ORDER BY** | Sắp xếp kết quả |
| 9 | **TOP / OFFSET-FETCH** | Giới hạn số lượng rows trả về |

### Ví dụ thực tế: Báo cáo doanh thu theo khách hàng

```sql
SELECT DISTINCT TOP 10
    c.CustomerName,                          -- (6) SELECT
    COUNT(o.OrderId) AS TotalOrders,
    SUM(o.TotalAmount) AS Revenue
FROM Customers c                             -- (1) FROM
INNER JOIN Orders o                          -- (1) JOIN
    ON c.CustomerId = o.CustomerId           -- (2) ON
WHERE o.OrderDate >= '2024-01-01'            -- (3) WHERE
    AND o.Status = 'Completed'
GROUP BY c.CustomerName                      -- (4) GROUP BY
HAVING SUM(o.TotalAmount) > 1000000          -- (5) HAVING
ORDER BY Revenue DESC                        -- (8) ORDER BY
-- TOP 10                                    -- (9) TOP
```

**Walk-through từng bước:**

**Bước 1 - FROM + JOIN:** Kết hợp bảng `Customers` với `Orders` qua `CustomerId`. Tạo virtual table chứa tất cả các cặp (customer, order) match.

**Bước 2 - ON:** Chỉ giữ các rows có `c.CustomerId = o.CustomerId`.

**Bước 3 - WHERE:** Lọc chỉ những orders từ 2024 trở đi và có Status = 'Completed'. Rows không thỏa bị loại.

**Bước 4 - GROUP BY:** Nhóm các rows còn lại theo `c.CustomerName`. Mỗi group là một khách hàng với tất cả orders của họ.

**Bước 5 - HAVING:** Chỉ giữ những groups có `SUM(TotalAmount) > 1,000,000`. Khách hàng doanh thu thấp bị loại.

**Bước 6 - SELECT:** Tính `COUNT(o.OrderId)`, `SUM(o.TotalAmount)`, lấy `c.CustomerName`. Đặt alias `TotalOrders`, `Revenue`.

**Bước 7 - DISTINCT:** Loại bỏ các rows trùng lặp (nếu có).

**Bước 8 - ORDER BY:** Sắp xếp theo `Revenue DESC`. Lưu ý: ORDER BY có thể dùng alias vì SELECT đã chạy trước.

**Bước 9 - TOP 10:** Lấy 10 rows đầu tiên sau khi sắp xếp.

### Tại sao thứ tự này quan trọng?

**Câu hỏi thường gặp trong interview:**

```sql
-- Tại sao query này LỖI?
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE OrderYear = 2024;
-- Lỗi: "Invalid column name 'OrderYear'"
-- Vì WHERE (bước 3) chạy TRƯỚC SELECT (bước 6)
-- Alias 'OrderYear' chưa tồn tại khi WHERE thực thi

-- FIX:
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE YEAR(OrderDate) = 2024;

-- Hoặc tốt hơn (SARGable):
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
```

```sql
-- Tại sao ORDER BY có thể dùng alias nhưng WHERE thì không?
SELECT FullName, SUM(Amount) AS TotalAmount
FROM Orders
GROUP BY FullName
ORDER BY TotalAmount DESC;  -- OK! ORDER BY (bước 8) chạy SAU SELECT (bước 6)
```

```sql
-- Tại sao HAVING dùng được aggregate nhưng WHERE thì không?
-- WHERE (bước 3) chạy TRƯỚC GROUP BY (bước 4) → chưa có groups để aggregate
-- HAVING (bước 5) chạy SAU GROUP BY (bước 4) → đã có groups
SELECT DepartmentId, AVG(Salary)
FROM Employees
WHERE AVG(Salary) > 5000  -- LỖI! Chưa có groups
GROUP BY DepartmentId;

SELECT DepartmentId, AVG(Salary)
FROM Employees
GROUP BY DepartmentId
HAVING AVG(Salary) > 5000;  -- OK! Đã group xong
```

---

## Câu 7: Có bao nhiêu kiểu JOIN?

SQL Server hỗ trợ **6 kiểu JOIN** chính:

### 1. INNER JOIN

```
Customers         Orders            Result (INNER JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Bình  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cường │   │ 2  │ 150$ │     │ Bình  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     └───────┴──────┘
                  └────┴──────┘
-- Cường (ID=3) không có order → bị loại
-- Order của CustomerId=4 không có customer → bị loại
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Chỉ trả về rows có match Ở CẢ HAI bảng
```

### 2. LEFT JOIN (LEFT OUTER JOIN)

```
Customers         Orders            Result (LEFT JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Bình  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cường │   │ 2  │ 150$ │     │ Bình  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     │ Cường │ NULL │
                  └────┴──────┘     └───────┴──────┘
-- Cường không có order → vẫn giữ, Orders columns = NULL
-- Order CustomerId=4 không có customer → bị loại (vì không nằm ở LEFT)
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Tất cả customers, kể cả những người chưa có order

-- Tìm customers CHƯA có order:
SELECT c.CustomerName
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.CustomerId IS NULL;
```

### 3. RIGHT JOIN (RIGHT OUTER JOIN)

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
RIGHT JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Tất cả orders, kể cả những orders không có customer match
-- Tương đương với LEFT JOIN nhưng đảo 2 bảng
```

Trong thực tế, **ít khi dùng RIGHT JOIN** -- thường viết lại thành LEFT JOIN cho dễ đọc.

### 4. FULL OUTER JOIN

```
Customers         Orders            Result (FULL OUTER JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Bình  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cường │   │ 2  │ 150$ │     │ Bình  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     │ Cường │ NULL │
                  └────┴──────┘     │ NULL  │ 300$ │
                                    └───────┴──────┘
-- Cường không có order → giữ, order = NULL
-- Order CustomerId=4 không có customer → giữ, customer = NULL
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
FULL OUTER JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Tất cả rows từ CẢ HAI bảng, NULL cho phía không match

-- Use case: đối chiếu dữ liệu giữa 2 hệ thống
SELECT
    COALESCE(a.Id, b.Id) AS Id,
    a.Amount AS SystemA_Amount,
    b.Amount AS SystemB_Amount
FROM SystemA a
FULL OUTER JOIN SystemB b ON a.Id = b.Id
WHERE a.Id IS NULL OR b.Id IS NULL;  -- Tìm records chỉ có ở 1 phía
```

### 5. CROSS JOIN

```
Colors           Sizes             Result (CROSS JOIN)
┌───────┐       ┌────────┐        ┌───────┬────────┐
│ Red   │       │ Small  │        │ Red   │ Small  │
│ Blue  │       │ Medium │        │ Red   │ Medium │
│ Green │       │ Large  │        │ Red   │ Large  │
└───────┘       └────────┘        │ Blue  │ Small  │
 3 rows          3 rows           │ Blue  │ Medium │
                                  │ Blue  │ Large  │
                                  │ Green │ Small  │
                                  │ Green │ Medium │
                                  │ Green │ Large  │
                                  └───────┴────────┘
                                   3 x 3 = 9 rows
```

```sql
-- Tích Descartes: mỗi row bảng A kết hợp với MỌI row bảng B
SELECT c.Color, s.Size
FROM Colors c
CROSS JOIN Sizes s;

-- Use case: tạo lịch
SELECT d.Date, s.ShiftName
FROM Calendar d
CROSS JOIN Shifts s
WHERE d.Date BETWEEN '2024-01-01' AND '2024-12-31';

-- CẢNH BÁO: 10,000 rows x 10,000 rows = 100 TRIỆU rows!
```

### 6. SELF JOIN

```sql
-- Bảng Employees có cột ManagerId trỏ về chính bảng đó
CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    FullName NVARCHAR(200),
    ManagerId INT REFERENCES Employees(EmployeeId)
);

-- Tìm nhân viên và tên manager của họ
SELECT
    e.FullName AS Employee,
    m.FullName AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;

-- CEO/Director không có manager → ManagerId = NULL → LEFT JOIN giữ lại
```

### Performance Tips

```sql
-- 1. INNER JOIN nhanh nhất (loại nhiều rows nhất)
-- Thứ tự ưu tiên: INNER > LEFT/RIGHT > FULL > CROSS

-- 2. Đảm bảo có index trên JOIN columns
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);
-- LEFT JOIN Customers c → Orders o: index trên o.CustomerId rất quan trọng

-- 3. Tránh Implicit Conversion trong JOIN
-- BAD: CustomerId là INT ở bảng A nhưng VARCHAR ở bảng B
FROM TableA a JOIN TableB b ON a.CustomerId = b.CustomerId
-- Nếu khác data type → convert toàn bộ 1 phía → không dùng index

-- 4. SQL Server Optimizer tự chọn JOIN algorithm:
-- Nested Loop: tốt cho bảng nhỏ INNER + bảng lớn có index OUTER
-- Merge Join: tốt khi cả 2 input đã sorted (VD: 2 clustered index)
-- Hash Join: tốt khi không có index, bảng lớn, không sort
-- Thường không cần dùng JOIN hints, để optimizer tự quyết định
```

---

## Câu 8: WHERE vs HAVING?

### Sự khác biệt cốt lõi

| Đặc điểm | WHERE | HAVING |
|----------|-------|--------|
| Thời điểm thực thi | **Trước GROUP BY** (bước 3) | **Sau GROUP BY** (bước 5) |
| Lọc | **Từng row** riêng lẻ | **Từng group** |
| Aggregate functions | **KHÔNG** dùng được | **CÓ** dùng được |
| Hiệu suất | **Tốt hơn** (giảm dữ liệu sớm) | Phải xử lý nhiều dữ liệu hơn |

### Ví dụ minh họa

```sql
-- Bài toán: Tìm phòng ban có hơn 10 nhân viên đang Active

-- CÁCH SAI: filter trong HAVING (chậm)
SELECT DepartmentId, COUNT(*) AS EmpCount
FROM Employees
GROUP BY DepartmentId
HAVING COUNT(*) > 10 AND DepartmentId != 5;
-- Bước 3 (WHERE): không có → xử lý TẤT CẢ rows (kể cả Dept 5)
-- Bước 4 (GROUP BY): nhóm tất cả rows, kể cả Dept 5
-- Bước 5 (HAVING): loại Dept 5 và groups <= 10
-- → Lãng phí: đã group Dept 5 rồi mới loại

-- CÁCH ĐÚNG: filter trong WHERE (nhanh)
SELECT DepartmentId, COUNT(*) AS EmpCount
FROM Employees
WHERE DepartmentId != 5        -- Loại Dept 5 TRƯỚC khi group
GROUP BY DepartmentId
HAVING COUNT(*) > 10;
-- Bước 3 (WHERE): loại tất cả rows Dept 5 → ít rows hơn
-- Bước 4 (GROUP BY): nhóm ít rows hơn → nhanh hơn
-- Bước 5 (HAVING): chỉ kiểm tra aggregate condition
```

### Quy tắc vàng

> **Nếu điều kiện KHÔNG cần aggregate function → dùng WHERE.**
> **Nếu điều kiện CẦN aggregate function (SUM, COUNT, AVG, MIN, MAX) → dùng HAVING.**

```sql
-- WHERE: lọc trên column values của từng row
WHERE Status = 'Active'
WHERE OrderDate >= '2024-01-01'
WHERE DepartmentId IN (1, 2, 3)

-- HAVING: lọc trên kết quả aggregate của group
HAVING COUNT(*) > 10
HAVING SUM(TotalAmount) > 1000000
HAVING AVG(Salary) BETWEEN 5000 AND 10000
```

### Trường hợp đặc biệt: HAVING không có GROUP BY

```sql
-- Treats toàn bộ result set như 1 group duy nhất
SELECT COUNT(*) AS TotalActive
FROM Employees
HAVING COUNT(*) > 100;
-- Nếu có hơn 100 employees → trả về count
-- Nếu không → trả về 0 rows (không phải 0, mà là KHÔNG CÓ ROW nào)
```

### So sánh performance thực tế

```sql
-- Bảng Orders: 10 triệu rows, 1 triệu rows có Status = 'Cancelled'

-- Query 1: HAVING filter (chậm)
SELECT CustomerId, SUM(TotalAmount)
FROM Orders
GROUP BY CustomerId
HAVING CustomerId != 999;
-- Xử lý: 10 triệu rows → GROUP BY → HAVING loại 1 customer
-- Logical reads: ~50,000 pages

-- Query 2: WHERE filter (nhanh)
SELECT CustomerId, SUM(TotalAmount)
FROM Orders
WHERE CustomerId != 999
GROUP BY CustomerId;
-- Xử lý: loại rows của Customer 999 trước → GROUP BY ít rows hơn
-- Logical reads: ~49,950 pages (tiết kiệm I/O)

-- Chênh lệch tùy thuộc vào lượng dữ liệu bị lọc.
-- Càng nhiều rows bị lọc bởi WHERE → càng tiết kiệm.
```

---

## Câu 9: UNION vs UNION ALL?

### Sự khác biệt chính

| Đặc điểm | UNION | UNION ALL |
|----------|-------|-----------|
| Kết quả trùng lặp | **Loại bỏ** | **Giữ tất cả** |
| Internal operation | Sort + Distinct (hoặc Hash Match) | Chỉ nối (Concatenation) |
| Performance | **Chậm hơn** | **Nhanh hơn** |
| Độ phức tạp | O(n log n) - phải sort/hash | O(n) - chỉ append |

### Ví dụ minh họa

```sql
-- Dữ liệu:
-- TableA: 1, 2, 3
-- TableB: 2, 3, 4

SELECT Id FROM TableA
UNION
SELECT Id FROM TableB;
-- Kết quả: 1, 2, 3, 4 (loại duplicate 2 và 3)

SELECT Id FROM TableA
UNION ALL
SELECT Id FROM TableB;
-- Kết quả: 1, 2, 3, 2, 3, 4 (giữ tất cả 6 rows)
```

### Execution Plan so sánh

```
UNION:                              UNION ALL:
┌──────────┐                        ┌──────────────┐
│ Sort     │  ← Chi phí cao         │ Concatenation │  ← Chi phí thấp
│ Distinct │  O(n log n)            │              │  O(n)
├──────────┤                        ├──────────────┤
│ Concat   │                        │ Table A      │
├──────────┤                        │ Table B      │
│ Table A  │                        └──────────────┘
│ Table B  │
└──────────┘
```

UNION phải **đọc toàn bộ dữ liệu, sort, và loại duplicates** -- với dữ liệu lớn, chi phí này rất đáng kể (tempdb spills, CPU).

### Khi nào dùng cái nào?

**Dùng UNION ALL khi:**

```sql
-- 1. Biết chắc không có duplicate (mỗi query từ bảng/partition khác nhau)
SELECT OrderId, Amount FROM Orders_2023
UNION ALL
SELECT OrderId, Amount FROM Orders_2024;
-- Mỗi partition có dữ liệu riêng biệt → không trùng

-- 2. Chấp nhận duplicate (không ảnh hưởng logic)
SELECT ProductId FROM RecentlyViewed
UNION ALL
SELECT ProductId FROM Recommendations;
-- Trùng không sao, chỉ cần danh sách để hiển thị

-- 3. Performance là ưu tiên
-- Dữ liệu lớn, cần kết quả nhanh
```

**Dùng UNION khi:**

```sql
-- Cần đảm bảo unique (VD: báo cáo tổng hợp không trùng)
SELECT Email FROM Newsletter_Subscribers
UNION
SELECT Email FROM Active_Customers;
-- Một người vừa subscribe vừa là customer → chỉ hiện 1 lần
```

### Các quy tắc quan trọng

```sql
-- 1. SỐ LƯỢNG COLUMNS phải bằng nhau
SELECT Id, Name FROM TableA
UNION ALL
SELECT Id FROM TableB;        -- LỖI! Khác số columns

-- 2. DATA TYPES phải tương thích (compatible)
SELECT Id, Name FROM TableA         -- Id: INT, Name: VARCHAR
UNION ALL
SELECT Code, Description FROM TableB -- Code: INT, Description: VARCHAR
-- OK nếu data types tương thích. SQL Server sẽ implicit convert nếu cần.

-- 3. Column names lấy từ query ĐẦU TIÊN
SELECT Id AS MaKH, Name AS TenKH FROM TableA
UNION ALL
SELECT Code, Description FROM TableB;
-- Kết quả có columns: MaKH, TenKH (lấy từ query đầu)

-- 4. ORDER BY chỉ dùng được ở CUỐI CÙNG
SELECT Id FROM TableA
UNION ALL
SELECT Id FROM TableB
ORDER BY Id;    -- ORDER BY áp dụng cho TOÀN BỘ kết quả

-- Muốn sort từng phần → dùng subquery hoặc CTE
```

### Ví dụ thực tế: Combine dữ liệu từ nhiều nguồn

```sql
-- Báo cáo tổng hợp doanh thu từ nhiều kênh bán hàng
SELECT
    'Online' AS Channel,
    OrderDate,
    SUM(Amount) AS Revenue
FROM OnlineOrders
WHERE OrderDate >= '2024-01-01'
GROUP BY OrderDate

UNION ALL

SELECT
    'Store' AS Channel,
    SaleDate,
    SUM(Amount) AS Revenue
FROM StoreTransactions
WHERE SaleDate >= '2024-01-01'
GROUP BY SaleDate

UNION ALL

SELECT
    'Wholesale' AS Channel,
    InvoiceDate,
    SUM(Amount) AS Revenue
FROM WholesaleInvoices
WHERE InvoiceDate >= '2024-01-01'
GROUP BY InvoiceDate

ORDER BY OrderDate, Channel;
-- UNION ALL vì mỗi bảng chứa dữ liệu riêng biệt, không trùng
-- Performance: chỉ Concatenation, không Sort Distinct
```

### Mẹo nhớ

| Câu hỏi | Trả lời |
|---------|---------|
| Mặc định nên dùng gì? | **UNION ALL** (nhanh hơn, chỉ dùng UNION khi thật sự cần loại trùng) |
| Có bao nhiêu UNION/UNION ALL? | Không giới hạn, nhưng càng nhiều → càng chậm |
| Có thể kết hợp UNION và UNION ALL? | **Có** |
| UNION có dùng index không? | Mỗi SELECT con có thể dùng index riêng, nhưng Sort Distinct ở ngoài thì không |

```sql
-- Kết hợp cả UNION và UNION ALL
SELECT Id FROM TableA
UNION ALL          -- Giữ duplicate giữa A và B
SELECT Id FROM TableB
UNION              -- Loại duplicate với C
SELECT Id FROM TableC;
-- Lưu ý: thứ tự ưu tiên có thể gây nhầm lẫn → dùng parentheses nếu cần
```

---

## Câu 10: UNION vs MINUS vs INTERSECT?

### Khái niệm

Ba toán tử tập hợp (set operators) dùng để kết hợp kết quả từ nhiều câu SELECT. Yêu cầu chung: **số cột và kiểu dữ liệu phải tương thích** giữa các query.

### UNION / UNION ALL

**UNION** kết hợp kết quả của 2 query, **loại bỏ duplicates** (thực hiện DISTINCT ngầm → tốn thêm chi phí sort).

**UNION ALL** giữ lại tất cả rows, kể cả trùng → **nhanh hơn** vì không cần sort/distinct.

```sql
-- Lấy tất cả khách hàng từ 2 hệ thống, loại trùng
SELECT Email, FullName FROM CustomersHCM
UNION
SELECT Email, FullName FROM CustomersHN;

-- Giữ tất cả (nhanh hơn), dùng khi biết chắc không trùng hoặc cần giữ trùng
SELECT Email, FullName FROM CustomersHCM
UNION ALL
SELECT Email, FullName FROM CustomersHN;
```

> **Best practice**: Luôn ưu tiên `UNION ALL` nếu biết chắc không có duplicates hoặc duplicates không ảnh hưởng. UNION phải sort toàn bộ result set để loại trùng → chi phí O(n log n).

### INTERSECT (Giao)

Trả về rows **có trong CẢ 2** queries.

```sql
-- Tìm sản phẩm được bán ở CẢ 2 chi nhánh
SELECT ProductId FROM SalesHCM
INTERSECT
SELECT ProductId FROM SalesHN;
```

### EXCEPT (Hiệu)

SQL Server dùng **EXCEPT**, Oracle dùng **MINUS**. Trả về rows có trong query 1 nhưng **KHÔNG có** trong query 2.

```sql
-- Tìm khách hàng chưa từng đặt hàng
SELECT CustomerId FROM Customers
EXCEPT
SELECT CustomerId FROM Orders;
```

### So sánh EXCEPT vs NOT IN vs NOT EXISTS

```sql
-- Cách 1: EXCEPT (recommended)
SELECT CustomerId FROM Customers
EXCEPT
SELECT CustomerId FROM Orders;

-- Cách 2: NOT EXISTS (recommended, linh hoạt hơn)
SELECT c.CustomerId FROM Customers c
WHERE NOT EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerId = c.CustomerId);

-- Cách 3: NOT IN (CẨN THẬN với NULL!)
SELECT CustomerId FROM Customers
WHERE CustomerId NOT IN (SELECT CustomerId FROM Orders);
```

**Vấn đề NULL với NOT IN** — đây là điểm rất quan trọng:

```sql
-- Giả sử Orders có 1 row với CustomerId = NULL
SELECT CustomerId FROM Customers
WHERE CustomerId NOT IN (SELECT CustomerId FROM Orders);
-- → Trả về EMPTY result set!
-- Lý do: NOT IN (1, 2, NULL) → CustomerId <> 1 AND CustomerId <> 2 AND CustomerId <> NULL
-- Bất kỳ so sánh nào với NULL đều là UNKNOWN → toàn bộ điều kiện = UNKNOWN → không trả row nào
```

**EXCEPT và NOT EXISTS xử lý NULL đúng** — chúng dùng phép so sánh tập hợp (IS NOT DISTINCT FROM), không bị ảnh hưởng bởi NULL.

### Performance so sánh

| Phương pháp | Xử lý NULL | Performance | Linh hoạt |
|---|---|---|---|
| EXCEPT | Đúng | Tốt (hash/merge) | Thấp (chỉ so toàn bộ row) |
| NOT EXISTS | Đúng | Tốt (semi join) | Cao (multi-column, complex conditions) |
| NOT IN | **SAI khi có NULL** | OK nhưng rủi ro | Trung bình |
| LEFT JOIN ... IS NULL | Đúng | Tương đương NOT EXISTS | Cao |

### Real-world Use Cases

```sql
-- 1. Tìm sản phẩm chưa bán được trong tháng này (EXCEPT)
SELECT ProductId FROM Products
EXCEPT
SELECT ProductId FROM OrderDetails WHERE OrderDate >= '2026-03-01';

-- 2. Tìm skills chung giữa 2 ứng viên (INTERSECT)
SELECT SkillName FROM CandidateSkills WHERE CandidateId = 1
INTERSECT
SELECT SkillName FROM CandidateSkills WHERE CandidateId = 2;

-- 3. Hợp nhất danh sách email marketing (UNION)
SELECT Email FROM NewsletterSubscribers
UNION
SELECT Email FROM PurchaseCustomers;
```

---

## Câu 11: SQL Injection là gì?

### Definition

SQL Injection là kỹ thuật tấn công bảo mật trong đó attacker **chèn mã SQL độc hại** vào user input, từ đó thay đổi logic của câu truy vấn gốc. Đây là một trong những lỗ hổng phổ biến và nguy hiểm nhất (OWASP Top 10).

### Classic Examples

```sql
-- Code vulnerable:
DECLARE @sql NVARCHAR(500) = 'SELECT * FROM Users WHERE Username = ''' + @input + ''' AND Password = ''' + @pass + ''''
EXEC(@sql)

-- Attacker nhập Username: ' OR 1=1 --
-- Câu SQL trở thành:
SELECT * FROM Users WHERE Username = '' OR 1=1 --' AND Password = '...'
-- → OR 1=1 luôn TRUE, -- comment phần còn lại → bypass login

-- Tấn công nguy hiểm hơn: '; DROP TABLE Users; --
-- Câu SQL trở thành:
SELECT * FROM Users WHERE Username = ''; DROP TABLE Users; --'
-- → Xóa sạch bảng Users
```

### Phân loại SQL Injection

**1. In-band SQLi** (phổ biến nhất):
- **Error-based**: lợi dụng error messages để lấy thông tin schema (table names, column names)
- **Union-based**: dùng UNION SELECT để trích xuất data từ các bảng khác

**2. Blind SQLi** (không thấy output trực tiếp):
- **Boolean-based**: gửi câu TRUE/FALSE, quan sát response khác nhau
  ```
  -- Đoán tên bảng từng ký tự
  ' AND SUBSTRING(DB_NAME(),1,1) = 'M' --
  ```
- **Time-based**: dùng WAITFOR DELAY, đo response time
  ```
  ' IF (SELECT COUNT(*) FROM Users) > 0 WAITFOR DELAY '0:0:5' --
  ```

**3. Out-of-band SQLi**: gửi data qua DNS hoặc HTTP request (ít phổ biến, dùng khi blind quá chậm)

### Prevention (Quan trọng nhất)

**1. Parameterized Queries — Biện pháp số 1**

```sql
-- VULNERABLE: String concatenation
DECLARE @sql NVARCHAR(500) = 'SELECT * FROM Users WHERE Name = ''' + @input + ''''
EXEC(@sql)

-- SAFE: Parameterized
SELECT * FROM Users WHERE Name = @input

-- SAFE: sp_executesql cho dynamic SQL
DECLARE @sql NVARCHAR(500) = N'SELECT * FROM Users WHERE Name = @Name'
EXEC sp_executesql @sql, N'@Name NVARCHAR(100)', @Name = @input
```

**2. Stored Procedures — nhưng KHÔNG tự động an toàn**

```sql
-- VULNERABLE SP: dynamic SQL bên trong vẫn bị inject
CREATE PROCEDURE SearchUsers @SearchTerm NVARCHAR(100)
AS
BEGIN
    EXEC('SELECT * FROM Users WHERE Name LIKE ''%' + @SearchTerm + '%''')
END

-- SAFE SP: dùng parameterized
CREATE PROCEDURE SearchUsers @SearchTerm NVARCHAR(100)
AS
BEGIN
    SELECT * FROM Users WHERE Name LIKE '%' + @SearchTerm + '%'
    -- Hoặc nếu cần dynamic SQL:
    -- EXEC sp_executesql N'SELECT * FROM Users WHERE Name LIKE @Term',
    --     N'@Term NVARCHAR(102)', @Term = '%' + @SearchTerm + '%'
END
```

> **Misconception phổ biến**: "Dùng Stored Procedure là an toàn" → **SAI**. SP chỉ an toàn nếu bên trong **không dùng string concatenation với EXEC()**. Nếu SP build dynamic SQL bằng cách nối chuỗi rồi EXEC, vẫn bị inject.

**3. ORM (Entity Framework Core)**

```csharp
// EF Core tự parameterize LINQ queries → SAFE
var user = await context.Users
    .Where(u => u.Name == input)
    .FirstOrDefaultAsync();

// VULNERABLE: FromSqlRaw với string interpolation
var users = context.Users
    .FromSqlRaw($"SELECT * FROM Users WHERE Name = '{input}'")
    .ToList();

// SAFE: FromSqlInterpolated (tự động parameterize)
var users = context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {input}")
    .ToList();

// SAFE: FromSqlRaw với explicit parameters
var users = context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = {0}", input)
    .ToList();
```

> **Chú ý**: `FromSqlRaw` với `$""` (string interpolation) là **vulnerable** vì C# interpolation chạy TRƯỚC khi EF xử lý. `FromSqlInterpolated` nhận `FormattableString` nên EF sẽ tự chuyển thành parameters.

**4. Input Validation / Whitelist**

```csharp
// Whitelist cho column names (dynamic ORDER BY)
var allowedColumns = new HashSet<string> { "Name", "Email", "CreatedAt" };
if (!allowedColumns.Contains(sortColumn))
    throw new ArgumentException("Invalid sort column");

// Không dùng input validation thay thế parameterized queries
// → chỉ là defense-in-depth layer
```

**5. Least Privilege DB Accounts**

```sql
-- App account chỉ cần EXEC quyền trên SPs, không cần trực tiếp SELECT/DELETE trên tables
CREATE USER AppUser FOR LOGIN AppLogin;
GRANT EXECUTE ON SCHEMA::dbo TO AppUser;
-- KHÔNG grant db_owner hay sysadmin cho app account
```

**6. Web Application Firewall (WAF)**: layer bảo vệ bên ngoài, detect và block SQL injection patterns. Không thay thế được parameterized queries, chỉ là lớp bảo vệ bổ sung.

### Tóm tắt phòng chống theo mức ưu tiên

1. **Parameterized queries** (bắt buộc)
2. **Least privilege** (bắt buộc)
3. **Input validation / whitelist** (defense-in-depth)
4. **WAF** (defense-in-depth)
5. **Error handling**: không expose chi tiết error ra client (ẩn stack trace, SQL errors)

---

## Câu 12: DROP vs DELETE vs TRUNCATE?

### Bảng so sánh chi tiết

| Feature | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| **Loại lệnh** | DML (Data Manipulation) | DDL (Data Definition) | DDL (Data Definition) |
| **Chức năng** | Xóa rows theo điều kiện | Xóa TẤT CẢ rows | Xóa toàn bộ table (structure + data) |
| **WHERE clause** | Có | Không | N/A |
| **Logging** | Full logging (từng row) | Minimal logging (page deallocation) | Full logging |
| **Rollback** | Có | **Có** (trong transaction) | **Có** (trong transaction) |
| **Trigger** | Fires DELETE trigger | **KHÔNG** fire trigger | N/A |
| **Identity** | Không reset | **Reset** về seed ban đầu | N/A |
| **Speed** | Chậm nhất | Nhanh | Nhanh nhất |
| **FK Constraint** | OK nếu không violate | **KHÔNG được** nếu có FK reference đến | Phải DROP FK trước |
| **Lock type** | Row locks (có thể escalate) | Table lock (SCH-M) | Table lock |
| **Permission** | DELETE permission | ALTER TABLE permission | ALTER permission |
| **Space reclaim** | Không tự động (cần SHRINK) | Giải phóng ngay (trừ min extent) | Giải phóng ngay |

### Ví dụ cụ thể

```sql
-- DELETE: Xóa có điều kiện, log từng row
DELETE FROM Orders WHERE OrderDate < '2020-01-01';
-- → Log mỗi row vào transaction log
-- → Fire DELETE trigger (nếu có)
-- → Identity KHÔNG reset

-- TRUNCATE: Xóa tất cả, nhanh
TRUNCATE TABLE StagingData;
-- → Chỉ log page deallocation (minimal)
-- → KHÔNG fire trigger
-- → Identity reset về seed (VD: 1)
-- → Nhanh hơn DELETE FROM table rất nhiều lần

-- DROP: Xóa luôn table
DROP TABLE IF EXISTS TempReport;
-- → Table biến mất hoàn toàn (schema + data + indexes + constraints)
```

### Misconception quan trọng: "TRUNCATE không thể ROLLBACK"

Đây là **SAI** — một trong những misconception phổ biến nhất:

```sql
BEGIN TRANSACTION
    SELECT COUNT(*) FROM Products; -- 1000 rows

    TRUNCATE TABLE Products;
    SELECT COUNT(*) FROM Products; -- 0 rows

ROLLBACK TRANSACTION
SELECT COUNT(*) FROM Products; -- 1000 rows → data trở lại!
```

TRUNCATE **hoàn toàn có thể rollback** trong một transaction. Sở dĩ nhiều người hiểu sai là vì TRUNCATE là DDL, và trong Oracle, DDL tự động COMMIT (auto-commit). Nhưng **SQL Server không auto-commit DDL**, nên TRUNCATE (và cả DROP) đều rollback được.

### Khi nào dùng gì?

```sql
-- DELETE: Xóa selective, cần trigger fire, cần audit
DELETE FROM AuditLogs WHERE CreatedAt < DATEADD(YEAR, -7, GETDATE());

-- TRUNCATE: Xóa sạch staging/temp tables, ETL reload
TRUNCATE TABLE Staging_ImportData;
BULK INSERT Staging_ImportData FROM 'data.csv' WITH (...);

-- DROP: Remove table không cần nữa, cleanup
DROP TABLE IF EXISTS #TempResults;
DROP TABLE IF EXISTS ObsoleteReport_2020;
```

### Lưu ý về DELETE lượng lớn

Khi DELETE hàng triệu rows, **không nên DELETE một lần** vì sẽ giữ lock lâu, transaction log phình to:

```sql
-- BAD: Lock toàn bộ, log phình to
DELETE FROM Logs WHERE CreatedAt < '2024-01-01'; -- 50 triệu rows

-- GOOD: Batch delete
DECLARE @BatchSize INT = 10000;
WHILE 1 = 1
BEGIN
    DELETE TOP (@BatchSize) FROM Logs WHERE CreatedAt < '2024-01-01';
    IF @@ROWCOUNT < @BatchSize BREAK;
    -- Cho phép các transaction khác chạy xen kẽ
END
```

---

## Câu 13: Làm sao để cải thiện Performance?

Đây là câu hỏi senior-level, cần trả lời có hệ thống theo nhiều tầng.

### 1. Query Optimization (Tầng truy vấn)

**Tránh SELECT \***
```sql
-- BAD: Lấy tất cả columns, gây Key Lookup không cần thiết
SELECT * FROM Orders WHERE CustomerId = 123;

-- GOOD: Chỉ lấy columns cần
SELECT OrderId, OrderDate, TotalAmount FROM Orders WHERE CustomerId = 123;
```

**SARGable WHERE Clauses** (Search ARGument ABLE)

Điều kiện WHERE phải cho phép SQL Server sử dụng index seek, không phải scan.

```sql
-- NON-SARGable: function trên indexed column → Index SCAN
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2026;
SELECT * FROM Users WHERE UPPER(Email) = 'TEST@MAIL.COM';
SELECT * FROM Products WHERE Price + 10 > 100;

-- SARGable: Index SEEK
SELECT * FROM Orders WHERE OrderDate >= '2026-01-01' AND OrderDate < '2027-01-01';
SELECT * FROM Users WHERE Email = 'test@mail.com'; -- dùng case-insensitive collation
SELECT * FROM Products WHERE Price > 90;
```

**Dùng EXISTS thay NOT IN**
```sql
-- RISKY + SLOW: NOT IN scan toàn bộ, NULL gây bug
SELECT * FROM Customers
WHERE CustomerId NOT IN (SELECT CustomerId FROM Orders);

-- BETTER: NOT EXISTS, optimizer tạo anti semi join hiệu quả
SELECT * FROM Customers c
WHERE NOT EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerId = c.CustomerId);
```

**Batch large operations**
```sql
-- Tránh UPDATE 10 triệu rows một lần
WHILE 1 = 1
BEGIN
    UPDATE TOP (5000) Products SET Price = Price * 1.1 WHERE CategoryId = 5 AND IsUpdated = 0;
    IF @@ROWCOUNT = 0 BREAK;
END
```

### 2. Indexing Strategy (Tầng index)

**Covering Index với INCLUDE**
```sql
-- Query: SELECT OrderDate, TotalAmount FROM Orders WHERE CustomerId = 123
-- Nếu chỉ có: CREATE INDEX IX_CustomerId ON Orders(CustomerId)
-- → Index Seek + Key Lookup (quay lại clustered index lấy OrderDate, TotalAmount)

-- Covering index: không cần Key Lookup
CREATE NONCLUSTERED INDEX IX_Orders_Customer_Covering
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount);
```

**Filtered Index cho partial data**
```sql
-- Chỉ 5% orders là Active, 95% là Completed
-- Filtered index nhỏ hơn, nhanh hơn
CREATE NONCLUSTERED INDEX IX_Orders_Active
ON Orders(OrderDate, CustomerId)
WHERE Status = 'Active';
```

**Composite Index — thứ tự quan trọng**
```sql
-- Query: WHERE Status = 'Active' AND CustomerId = 123
-- Status có 5 values, CustomerId có 100K values

-- GOOD: most selective first (nhưng cũng xét equality vs range)
CREATE INDEX IX_Orders_Customer_Status ON Orders(CustomerId, Status);
-- CustomerId = 123 lọc xuống ~100 rows → Status lọc tiếp rất nhanh

-- Quy tắc chung: Equality columns trước, Range columns sau
-- WHERE CustomerId = 123 AND OrderDate > '2026-01-01'
CREATE INDEX IX ON Orders(CustomerId, OrderDate); -- CustomerId (equality) trước
```

**Index Maintenance**
```sql
-- Kiểm tra fragmentation
SELECT
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10;

-- Fragmentation 10-30%: REORGANIZE (online, nhẹ)
ALTER INDEX IX_Orders_Customer ON Orders REORGANIZE;

-- Fragmentation > 30%: REBUILD (nặng hơn, nhưng hiệu quả hơn)
ALTER INDEX IX_Orders_Customer ON Orders REBUILD WITH (ONLINE = ON);
```

**Tìm và xóa Unused Indexes**
```sql
-- Index tồn tại nhưng không ai dùng → chỉ tốn chi phí INSERT/UPDATE
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    us.user_seeks, us.user_scans, us.user_lookups,
    us.user_updates -- số lần phải maintain index do DML
FROM sys.dm_db_index_usage_stats us
JOIN sys.indexes i ON us.object_id = i.object_id AND us.index_id = i.index_id
WHERE us.database_id = DB_ID()
    AND us.user_seeks + us.user_scans + us.user_lookups = 0
    AND us.user_updates > 0
    AND i.is_primary_key = 0;
```

### 3. Execution Plan Analysis (Tầng phân tích)

**Cách đọc Execution Plan — những dấu hiệu quan trọng:**

| Dấu hiệu trong Plan | Vấn đề | Giải pháp |
|---|---|---|
| **Key Lookup** (Nested Loops) | Non-covering index | Thêm INCLUDE columns |
| **Table Scan / Clustered Index Scan** | Thiếu index hoặc non-SARGable | Tạo index, sửa WHERE clause |
| **Sort** (cost cao) | Không có index pre-sorted | Index theo ORDER BY columns |
| **Hash Match** trên bảng nhỏ | Statistics outdated | UPDATE STATISTICS |
| **Warning: Implicit Conversion** | Kiểu dữ liệu không khớp | Sửa data type hoặc CAST đúng chỗ |
| **Thick arrows** (nhiều rows) | Estimates sai hoặc missing index | Update statistics, thêm index |
| **Missing Index suggestion** | SQL Server gợi ý index | Đánh giá và tạo nếu hợp lý |

```sql
-- Bật Actual Execution Plan
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Xem plan mà không chạy query
SET SHOWPLAN_XML ON;
GO
SELECT ... FROM ...
GO
SET SHOWPLAN_XML OFF;
```

**Implicit Conversion — performance killer ẩn:**
```sql
-- Column Email là VARCHAR, nhưng parameter là NVARCHAR
-- → SQL Server phải CONVERT mỗi row → Index Scan thay vì Seek
DECLARE @email NVARCHAR(100) = N'test@mail.com';
SELECT * FROM Users WHERE Email = @email;
-- Fix: đảm bảo kiểu dữ liệu khớp
DECLARE @email VARCHAR(100) = 'test@mail.com';
```

### 4. Database Design (Tầng thiết kế)

**Data types phù hợp**
```sql
-- BAD: dùng NVARCHAR cho mọi thứ
CREATE TABLE Products (
    Id BIGINT,          -- Chỉ cần INT nếu < 2 tỷ records
    Code NVARCHAR(50),  -- Chỉ ASCII → VARCHAR tiết kiệm 50% space
    Price FLOAT,        -- Dùng DECIMAL cho tiền tệ (tránh rounding errors)
    Notes NVARCHAR(MAX) -- MAX khi chỉ cần NVARCHAR(500)
);

-- GOOD
CREATE TABLE Products (
    Id INT IDENTITY,
    Code VARCHAR(20),
    Price DECIMAL(12,2),
    Notes NVARCHAR(500)
);
```

**Table Partitioning cho bảng lớn**
```sql
-- Partition bảng Orders theo năm
CREATE PARTITION FUNCTION PF_OrderDate (DATE)
AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01', '2026-01-01');

CREATE PARTITION SCHEME PS_OrderDate
AS PARTITION PF_OrderDate ALL TO ([PRIMARY]);

CREATE TABLE Orders (
    OrderId INT IDENTITY,
    OrderDate DATE,
    CustomerId INT,
    TotalAmount DECIMAL(12,2)
) ON PS_OrderDate(OrderDate);

-- Query chỉ scan partition cần thiết (partition elimination)
SELECT * FROM Orders WHERE OrderDate >= '2026-01-01';
```

### 5. Server Configuration

```sql
-- MAXDOP: tránh parallelism quá mức
-- Recommendation: số cores per NUMA node, nhưng không quá 8
EXEC sp_configure 'max degree of parallelism', 4;

-- Cost Threshold for Parallelism: default 5 quá thấp
-- Recommendation: 25-50 cho OLTP workload
EXEC sp_configure 'cost threshold for parallelism', 50;

-- TempDB optimization: nhiều data files = giảm contention
-- Recommendation: 1 file per CPU core (tối đa 8 files), equal size
-- Đặt TempDB trên SSD riêng
```

### 6. Monitoring và Diagnostics

**Query Store (SQL Server 2016+)** — built-in performance monitoring:
```sql
-- Bật Query Store
ALTER DATABASE MyDB SET QUERY_STORE = ON (
    OPERATION_MODE = READ_WRITE,
    MAX_STORAGE_SIZE_MB = 1024,
    INTERVAL_LENGTH_MINUTES = 30
);

-- Tìm top queries tốn resource nhất
SELECT TOP 10
    qsq.query_id,
    qsqt.query_sql_text,
    qsrs.avg_duration / 1000.0 AS avg_duration_ms,
    qsrs.avg_cpu_time / 1000.0 AS avg_cpu_ms,
    qsrs.avg_logical_io_reads,
    qsrs.count_executions
FROM sys.query_store_query qsq
JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
JOIN sys.query_store_plan qsp ON qsq.query_id = qsp.query_id
JOIN sys.query_store_runtime_stats qsrs ON qsp.plan_id = qsrs.plan_id
ORDER BY qsrs.avg_duration * qsrs.count_executions DESC;
```

**Wait Statistics — biết hệ thống đang "đợi" gì:**
```sql
SELECT TOP 10
    wait_type,
    wait_time_ms / 1000.0 AS wait_time_sec,
    signal_wait_time_ms / 1000.0 AS signal_wait_sec,
    waiting_tasks_count
FROM sys.dm_os_wait_stats
WHERE wait_type NOT IN ('SLEEP_TASK', 'BROKER_TO_FLUSH', 'SQLTRACE_BUFFER_FLUSH')
ORDER BY wait_time_ms DESC;

-- PAGEIOLATCH_SH: I/O bottleneck → cần SSD hoặc thêm RAM
-- CXPACKET: parallelism waits → tune MAXDOP
-- LCK_M_X: lock contention → optimize transactions, indexing
-- SOS_SCHEDULER_YIELD: CPU pressure → optimize queries
```

### 7. Application Level

**Pagination hiệu quả**
```sql
-- OFFSET-FETCH: OK cho trang đầu, chậm dần khi page lớn
SELECT OrderId, OrderDate, TotalAmount
FROM Orders
ORDER BY OrderId
OFFSET 100000 ROWS FETCH NEXT 20 ROWS ONLY;
-- → Phải skip 100K rows

-- KEYSET Pagination: performance ổn định mọi trang
SELECT TOP 20 OrderId, OrderDate, TotalAmount
FROM Orders
WHERE OrderId > @lastSeenId  -- truyền Id cuối cùng của trang trước
ORDER BY OrderId;
-- → Index Seek, không skip row nào
```

**Caching, Connection Pooling, Async**
```csharp
// Redis caching cho data ít thay đổi
var products = await cache.GetOrCreateAsync("products:category:5", async entry =>
{
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
    return await dbContext.Products.Where(p => p.CategoryId == 5).ToListAsync();
});

// Connection Pooling: .NET tự quản lý, đảm bảo connection string giống nhau
// Async queries: không block thread pool
var orders = await dbContext.Orders
    .Where(o => o.CustomerId == customerId)
    .ToListAsync(); // async, không block
```

---

## Câu 14: View vs Stored Procedure?

### Bảng so sánh

| Feature | View | Stored Procedure |
|---|---|---|
| **Bản chất** | Virtual table (SELECT query được đặt tên) | Compiled program lưu trong DB |
| **Parameters** | Không | Có (INPUT, OUTPUT) |
| **DML bên trong** | Chỉ SELECT | SELECT, INSERT, UPDATE, DELETE, DDL, control flow |
| **Return** | 1 result set (dùng trong FROM, JOIN) | Multiple result sets + OUTPUT params + RETURN value |
| **Dùng trong SELECT** | Có (`SELECT * FROM MyView`) | Không trực tiếp (phải dùng INSERT INTO ... EXEC) |
| **Plan caching** | Inline: không cache riêng, expand vào query gọi. Indexed View: cache data vật lý | Plan compiled và cached |
| **Security** | GRANT SELECT | GRANT EXECUTE |
| **Schema binding** | Optional (bắt buộc cho Indexed View) | Optional |

### View — Use Cases và Chi tiết

**1. Simplify Complex JOINs**
```sql
CREATE VIEW vw_OrderSummary
AS
SELECT
    o.OrderId,
    c.CustomerName,
    o.OrderDate,
    SUM(od.Quantity * od.UnitPrice) AS TotalAmount,
    COUNT(od.ProductId) AS ItemCount
FROM Orders o
JOIN Customers c ON o.CustomerId = c.CustomerId
JOIN OrderDetails od ON o.OrderId = od.OrderId
GROUP BY o.OrderId, c.CustomerName, o.OrderDate;

-- Sử dụng đơn giản
SELECT * FROM vw_OrderSummary WHERE OrderDate >= '2026-01-01';
```

**2. Security — ẩn columns nhạy cảm**
```sql
-- Users chỉ thấy thông tin công khai, không thấy Salary, SSN
CREATE VIEW vw_EmployeePublic
AS
SELECT EmployeeId, FullName, Department, Title, Email
FROM Employees;

GRANT SELECT ON vw_EmployeePublic TO ReportingRole;
-- KHÔNG grant SELECT trên table Employees
```

**3. Backward Compatibility**
```sql
-- Khi đổi schema table, tạo view giữ tên cũ để app cũ không bị lỗi
-- Bảng cũ: CustomerInfo(Id, Name, Address)
-- Bảng mới: Customers(Id, FirstName, LastName, Street, City)
CREATE VIEW CustomerInfo AS
SELECT Id, FirstName + ' ' + LastName AS Name, Street + ', ' + City AS Address
FROM Customers;
```

### Indexed View (Materialized View)

Data được lưu vật lý trên disk, tự động cập nhật khi base tables thay đổi. Rất hiệu quả cho aggregate queries chạy thường xuyên.

```sql
CREATE VIEW vw_ProductSalesStats
WITH SCHEMABINDING -- BẮT BUỘC cho Indexed View
AS
SELECT
    p.ProductId,
    p.ProductName,
    COUNT_BIG(*) AS SalesCount,  -- BẮT BUỘC có COUNT_BIG(*)
    SUM(od.Quantity) AS TotalQuantity,
    SUM(od.Quantity * od.UnitPrice) AS TotalRevenue
FROM dbo.Products p
JOIN dbo.OrderDetails od ON p.ProductId = od.ProductId
GROUP BY p.ProductId, p.ProductName;

-- Tạo unique clustered index → view trở thành materialized
CREATE UNIQUE CLUSTERED INDEX IX_vw_ProductSalesStats
ON vw_ProductSalesStats(ProductId);
```

**Restrictions của Indexed View:**
- Bắt buộc SCHEMABINDING
- Không dùng OUTER JOIN, subqueries, UNION
- Không dùng TOP, DISTINCT, HAVING (trong một số trường hợp)
- Phải có COUNT_BIG(*) nếu có aggregate
- Enterprise Edition: optimizer tự dùng Indexed View. Standard: phải query trực tiếp view hoặc dùng NOEXPAND hint.

### Stored Procedure — Use Cases

```sql
-- Business logic phức tạp
CREATE PROCEDURE sp_PlaceOrder
    @CustomerId INT,
    @Items OrderItemType READONLY, -- Table-Valued Parameter
    @OrderId INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRANSACTION
            -- Validate inventory
            IF EXISTS (
                SELECT 1 FROM @Items i
                JOIN Products p ON i.ProductId = p.ProductId
                WHERE p.InventoryCount < i.Quantity
            )
            BEGIN
                ;THROW 50001, 'Insufficient inventory', 1;
            END

            -- Create order
            INSERT INTO Orders(CustomerId, OrderDate, Status)
            VALUES (@CustomerId, GETDATE(), 'Pending');
            SET @OrderId = SCOPE_IDENTITY();

            -- Add order details
            INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
            SELECT @OrderId, i.ProductId, i.Quantity, p.Price
            FROM @Items i JOIN Products p ON i.ProductId = p.ProductId;

            -- Update inventory
            UPDATE p SET p.InventoryCount = p.InventoryCount - i.Quantity
            FROM Products p JOIN @Items i ON p.ProductId = i.ProductId;

        COMMIT TRANSACTION
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;
        THROW;
    END CATCH
END
```

### View Pitfalls

**Nested Views — Performance Nightmare**
```sql
-- View lồng view lồng view → SQL Server expand tất cả → plan phức tạp, khó optimize
CREATE VIEW vw_Level1 AS SELECT ... FROM TableA JOIN TableB ...
CREATE VIEW vw_Level2 AS SELECT ... FROM vw_Level1 JOIN TableC ...
CREATE VIEW vw_Level3 AS SELECT ... FROM vw_Level2 JOIN TableD ...

-- Khi chạy: SELECT col1 FROM vw_Level3 WHERE ...
-- SQL Server expand ra full query với tất cả JOINs, kể cả bạn chỉ cần 1 column
-- Rất khó đọc execution plan, rất khó debug performance
```

> **Best practice**: Giới hạn tối đa 1-2 levels nesting. Nếu cần phức tạp hơn, dùng SP hoặc inline TVF.

### SP vs Inline Table-Valued Function

```sql
-- Inline TVF: giống view nhưng CÓ parameters, có thể JOIN được
CREATE FUNCTION fn_GetOrdersByCustomer(@CustomerId INT)
RETURNS TABLE
AS
RETURN
    SELECT OrderId, OrderDate, TotalAmount
    FROM Orders
    WHERE CustomerId = @CustomerId;

-- Dùng trong FROM clause (SP không làm được)
SELECT c.CustomerName, o.OrderDate, o.TotalAmount
FROM Customers c
CROSS APPLY fn_GetOrdersByCustomer(c.CustomerId) o;
```

| | View | Inline TVF | Stored Procedure |
|---|---|---|---|
| Parameters | Không | Có | Có |
| Dùng trong FROM/JOIN | Có | Có | Không |
| DML | Chỉ SELECT | Chỉ SELECT | Mọi DML |
| Performance | Inline expand | Inline expand (tốt) | Compiled plan |

---

## Câu 15: Transaction trong SQL?

### ACID Properties

| Property | Ý nghĩa | Ví dụ |
|---|---|---|
| **Atomicity** | Tất cả hoặc không gì. Nếu 1 bước lỗi, rollback toàn bộ | Chuyển tiền: trừ A + cộng B phải cùng thành công hoặc cùng thất bại |
| **Consistency** | Data luôn valid trước và sau transaction | Tổng tiền trong hệ thống không đổi sau chuyển khoản |
| **Isolation** | Transactions đồng thời không ảnh hưởng lẫn nhau | 2 người cùng chuyển tiền từ 1 tài khoản không gây race condition |
| **Durability** | Data đã committed tồn tại vĩnh viễn, kể cả server crash | Sau khi COMMIT, dù mất điện, data vẫn còn (write-ahead logging) |

### Transaction Syntax với Error Handling đúng chuẩn

```sql
BEGIN TRY
    BEGIN TRANSACTION

        -- Trừ tiền tài khoản nguồn
        UPDATE Accounts SET Balance = Balance - 1000
        WHERE AccountId = 1 AND Balance >= 1000; -- kiểm tra đủ tiền

        IF @@ROWCOUNT = 0
            THROW 50001, 'Insufficient balance or account not found', 1;

        -- Cộng tiền tài khoản đích
        UPDATE Accounts SET Balance = Balance + 1000
        WHERE AccountId = 2;

        IF @@ROWCOUNT = 0
            THROW 50002, 'Destination account not found', 1;

        -- Ghi log giao dịch
        INSERT INTO TransactionLog(FromAccount, ToAccount, Amount, TransactionDate)
        VALUES (1, 2, 1000, GETDATE());

    COMMIT TRANSACTION

END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    -- Log error chi tiết
    INSERT INTO ErrorLog(ErrorMessage, ErrorLine, ErrorProcedure, ErrorTime)
    VALUES (ERROR_MESSAGE(), ERROR_LINE(), ERROR_PROCEDURE(), GETDATE());

    THROW; -- Re-raise error cho caller
END CATCH
```

### Isolation Levels — Chi tiết và Trade-offs

#### Concurrency Problems

| Problem | Mô tả |
|---|---|
| **Dirty Read** | Đọc data chưa committed từ transaction khác (có thể bị rollback) |
| **Non-repeatable Read** | Đọc cùng row 2 lần trong 1 transaction, giá trị khác nhau (bị UPDATE bởi transaction khác) |
| **Phantom Read** | Chạy cùng query 2 lần, lần 2 có thêm/bớt rows (bị INSERT/DELETE bởi transaction khác) |

#### Ma trận Isolation Levels

| Isolation Level | Dirty Read | Non-repeatable Read | Phantom Read | Locking |
|---|:---:|:---:|:---:|---|
| READ UNCOMMITTED | Có | Có | Có | Không shared lock |
| READ COMMITTED (default) | Không | Có | Có | Shared lock giữ trong lúc đọc |
| REPEATABLE READ | Không | Không | Có | Shared lock giữ đến hết transaction |
| SERIALIZABLE | Không | Không | Không | Range locks |
| SNAPSHOT | Không | Không | Không | Row versioning (TempDB) |

#### Chi tiết từng level

**READ UNCOMMITTED**
```sql
-- Dùng cho reports không cần chính xác tuyệt đối
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- Hoặc dùng hint
SELECT COUNT(*) FROM Orders WITH (NOLOCK); -- tương đương

-- Use case: dashboard realtime hiển thị số lượng đơn hàng
-- Sai lệch 1-2 đơn không ảnh hưởng, nhưng performance rất tốt
```

**READ COMMITTED (Default)**
```sql
-- Default của SQL Server
-- Shared lock lấy khi đọc, release ngay sau khi đọc xong row
-- → Transaction khác có thể UPDATE row đó trước khi transaction hiện tại kết thúc
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN TRANSACTION
    SELECT Balance FROM Accounts WHERE Id = 1; -- Balance = 1000
    -- Transaction B cập nhật Balance = 500 và COMMIT
    WAITFOR DELAY '00:00:02';
    SELECT Balance FROM Accounts WHERE Id = 1; -- Balance = 500 (non-repeatable read)
COMMIT;
```

**REPEATABLE READ**
```sql
-- Shared lock giữ đến hết transaction → row đã đọc không bị UPDATE
-- Nhưng transaction khác vẫn có thể INSERT rows mới → phantom reads
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN TRANSACTION
    SELECT COUNT(*) FROM Orders WHERE Status = 'Pending'; -- 10
    -- Transaction B INSERT thêm 1 order 'Pending' và COMMIT
    WAITFOR DELAY '00:00:02';
    SELECT COUNT(*) FROM Orders WHERE Status = 'Pending'; -- 11 (phantom!)
COMMIT;
```

**SERIALIZABLE**
```sql
-- Full isolation, range locks → không phantom reads
-- NHƯNG: deadlock risk rất cao, throughput thấp
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Chỉ dùng khi thật sự cần (financial calculations, critical balances)
```

**SNAPSHOT — Recommended cho high-concurrency apps**
```sql
-- Bật ở database level
ALTER DATABASE MyDB SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Sử dụng
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION
    -- Đọc "snapshot" của data tại thời điểm BEGIN TRANSACTION
    -- Readers KHÔNG block writers, writers KHÔNG block readers
    SELECT Balance FROM Accounts WHERE Id = 1;
    -- Transaction B thay đổi Balance → không ảnh hưởng snapshot này
    WAITFOR DELAY '00:00:02';
    SELECT Balance FROM Accounts WHERE Id = 1; -- Vẫn giá trị cũ (consistent)
COMMIT;
```

**SNAPSHOT vs READ COMMITTED SNAPSHOT (RCSI)**:
```sql
-- RCSI: mỗi STATEMENT thấy snapshot tại thời điểm statement bắt đầu
ALTER DATABASE MyDB SET READ_COMMITTED_SNAPSHOT ON;
-- → Tự động thay đổi behavior của READ COMMITTED
-- → Không cần đổi code, không cần SET ISOLATION LEVEL
-- → Recommended cho hầu hết ứng dụng production

-- SNAPSHOT Isolation: toàn transaction thấy snapshot tại thời điểm BEGIN TRAN
-- → Cần SET TRANSACTION ISOLATION LEVEL SNAPSHOT explicitly
```

### SAVEPOINT — Partial Rollback

```sql
BEGIN TRANSACTION

    INSERT INTO Orders(CustomerId, OrderDate) VALUES (1, GETDATE());
    SAVE TRANSACTION SavePoint1;

    INSERT INTO OrderDetails(OrderId, ProductId, Quantity) VALUES (100, 5, 999);
    -- Giả sử lỗi logic, cần rollback chỉ phần OrderDetails
    ROLLBACK TRANSACTION SavePoint1; -- Chỉ rollback đến SavePoint1
    -- Order vẫn còn, OrderDetails bị rollback

    -- Thử lại với số lượng đúng
    INSERT INTO OrderDetails(OrderId, ProductId, Quantity) VALUES (100, 5, 10);

COMMIT TRANSACTION; -- Commit cả Order + OrderDetails (lần 2)
```

### Nested Transactions — Quan trọng phải hiểu đúng

SQL Server **KHÔNG thực sự support nested transactions**:

```sql
BEGIN TRANSACTION -- @@TRANCOUNT = 1
    INSERT INTO Table1 VALUES (1);

    BEGIN TRANSACTION -- @@TRANCOUNT = 2 (chỉ tăng counter)
        INSERT INTO Table2 VALUES (2);
    COMMIT TRANSACTION -- @@TRANCOUNT = 1 (giảm counter, KHÔNG commit thật)

    -- ROLLBACK ở đây sẽ rollback CẢ 2 inserts
    ROLLBACK TRANSACTION -- @@TRANCOUNT = 0, rollback toàn bộ
```

> **Key insight**: Inner COMMIT chỉ giảm `@@TRANCOUNT`, không commit gì cả. Chỉ outer COMMIT (khi `@@TRANCOUNT` về 0) mới thực sự commit. Nhưng ROLLBACK ở bất kỳ đâu đều rollback **toàn bộ** về 0.

### Best Practices

1. **Keep transactions short** — giữ locks ngắn nhất có thể
2. **Không có user interaction** trong transaction (đợi user confirm → giữ lock)
3. **Luôn có error handling** — TRY/CATCH + ROLLBACK
4. **Consistent access order** — tránh deadlock bằng cách access tables theo thứ tự cố định
5. **Dùng RCSI** cho production apps (readers không block writers)

---

## Câu 16: Backup trong SQL?

### Recovery Models

| Recovery Model | Log Backup | Point-in-Time Recovery | Bulk Operations | Use Case |
|---|:---:|:---:|---|---|
| **Simple** | Không | Không | Minimal logged | Dev, Test, DB ít thay đổi |
| **Full** | Bắt buộc | Có | Full logged | **Production** |
| **Bulk-Logged** | Bắt buộc | Không (trong thời gian bulk) | Minimal logged | Bulk import windows |

```sql
-- Xem recovery model hiện tại
SELECT name, recovery_model_desc FROM sys.databases WHERE name = 'MyDB';

-- Đổi recovery model
ALTER DATABASE MyDB SET RECOVERY FULL;
```

**Lưu ý quan trọng**: Ở **Simple** recovery model, transaction log tự truncate tại checkpoint. Ở **Full**, log **chỉ truncate sau khi backup log**. Nếu dùng Full mà không backup log → log file phình to liên tục.

### Backup Types

**Full Backup** — Base cho mọi chiến lược
```sql
-- Full backup với compression và checksum
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_Full_20260323.bak'
WITH
    COMPRESSION,              -- Giảm 60-80% dung lượng
    CHECKSUM,                 -- Verify integrity
    STATS = 10,               -- Progress mỗi 10%
    NAME = 'MyDB Full Backup',
    DESCRIPTION = 'Weekly full backup';
```

**Differential Backup** — Changes since last FULL (không phải last diff)
```sql
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_Diff_20260323.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM;

-- Quan trọng: Diff luôn dựa trên last FULL backup
-- Sunday: Full → Monday: Diff1 (changes since Sunday)
-- Tuesday: Diff2 (changes since Sunday, BAO GỒM changes trong Diff1)
-- → Chỉ cần restore Full + latest Diff, không cần tất cả Diffs
```

**Transaction Log Backup** — Incremental, enable point-in-time recovery
```sql
BACKUP LOG MyDB
TO DISK = 'D:\Backup\MyDB_Log_20260323_1400.trn'
WITH COMPRESSION, CHECKSUM;

-- Log backup TRUNCATE log (cho phép reuse space)
-- → Phải backup log thường xuyên để log file không phình to
```

**Copy-Only Backup** — Không ảnh hưởng backup chain
```sql
-- Ad-hoc backup (ví dụ: trước khi deploy) mà không break backup chain
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_CopyOnly.bak'
WITH COPY_ONLY, COMPRESSION;
-- Differential backup tiếp theo vẫn dựa trên Full backup trước đó, không phải copy-only này
```

### Production Backup Strategy

```
Timeline:
Sunday 00:00    → FULL Backup
Mon-Sat 00:00   → DIFFERENTIAL Backup
Mỗi 15 phút    → TRANSACTION LOG Backup

RPO (Recovery Point Objective) = tối đa 15 phút data loss
```

```sql
-- 1. Full Backup (Sunday 00:00)
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_Full.bak'
WITH COMPRESSION, CHECKSUM, INIT;

-- 2. Differential (Mon-Sat 00:00)
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_Diff.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM, INIT;

-- 3. Transaction Log (mỗi 15 phút)
BACKUP LOG MyDB
TO DISK = 'D:\Backup\MyDB_Log.trn'
WITH COMPRESSION, CHECKSUM, NOINIT;
```

### Restore Process

Thứ tự restore: **Full → Latest Diff → Tất cả Log backups sau Diff**

```sql
-- Scenario: Lỗi xảy ra lúc Wednesday 14:30
-- Cần restore: Full (Sunday) → Diff (Wednesday 00:00) → Logs (00:00 → 14:15)

-- Bước 1: Restore Full với NORECOVERY
RESTORE DATABASE MyDB
FROM DISK = 'D:\Backup\MyDB_Full.bak'
WITH NORECOVERY, REPLACE;

-- Bước 2: Restore Latest Differential với NORECOVERY
RESTORE DATABASE MyDB
FROM DISK = 'D:\Backup\MyDB_Diff_Wed.bak'
WITH NORECOVERY;

-- Bước 3: Restore từng Transaction Log theo thứ tự
RESTORE LOG MyDB FROM DISK = 'D:\Backup\MyDB_Log_Wed_0015.trn' WITH NORECOVERY;
RESTORE LOG MyDB FROM DISK = 'D:\Backup\MyDB_Log_Wed_0030.trn' WITH NORECOVERY;

-- Bước 4: Point-in-time recovery - dừng tại 14:25 (trước khi lỗi)
RESTORE LOG MyDB
FROM DISK = 'D:\Backup\MyDB_Log_Wed_1415.trn'
WITH RECOVERY, STOPAT = '2026-03-18T14:25:00';
```

### Tail-Log Backup — Trước khi restore

```sql
-- NẾU database vẫn online, backup tail of the log trước khi restore
BACKUP LOG MyDB
TO DISK = 'D:\Backup\MyDB_TailLog.trn'
WITH NORECOVERY;
```

### Best Practices

**1. 3-2-1 Rule**: 3 copies, 2 media types (disk + tape/cloud), 1 offsite

**2. Test Restore thường xuyên**
```sql
RESTORE VERIFYONLY FROM DISK = 'D:\Backup\MyDB_Full.bak' WITH CHECKSUM;
-- VERIFYONLY chỉ check structure, nên restore thật lên test server định kỳ
```

**3. DBCC CHECKDB sau khi restore**
```sql
DBCC CHECKDB ('MyDB') WITH NO_INFOMSGS, ALL_ERRORMSGS;
```

**4. Backup Encryption (SQL Server 2014+)**
```sql
BACKUP DATABASE MyDB
TO DISK = 'D:\Backup\MyDB_Encrypted.bak'
WITH
    COMPRESSION,
    ENCRYPTION (ALGORITHM = AES_256, SERVER CERTIFICATE = BackupCert);
```

**5. Monitoring backup jobs**
```sql
SELECT
    database_name,
    type AS backup_type,  -- D=Full, I=Diff, L=Log
    backup_start_date,
    backup_finish_date,
    compressed_backup_size / 1024 / 1024 AS size_MB
FROM msdb.dbo.backupset
WHERE database_name = 'MyDB'
ORDER BY backup_start_date DESC;
```

---

## Câu 17: Sync Database?

### Tổng quan các giải pháp

| Giải pháp | Loại | Độ trễ | Hướng | Use Case chính |
|---|---|---|---|---|
| Always On AG | HA/DR | Realtime / Seconds | Uni-directional | Production HA |
| Transactional Replication | Replication | Seconds | Publisher → Subscriber | Read replicas |
| Merge Replication | Replication | Minutes | Bi-directional | Distributed offices |
| Snapshot Replication | Replication | Scheduled | Uni-directional | Small reference data |
| Log Shipping | DR | Minutes-Hours | Uni-directional | Cheap DR |
| CDC | Change Capture | Near real-time | Capture only | ETL, Audit |
| Change Tracking | Change Capture | Sync-based | Tracking only | Offline sync apps |

### 1. Always On Availability Groups (Recommended cho HA/DR)

```
Primary Replica ──synchronous──► Secondary Replica (HA, same datacenter)
       │
       └──────asynchronous──────► Secondary Replica (DR, remote site)
```

**Synchronous Commit**: Zero data loss, automatic failover, latency tăng nhẹ. Dùng cho HA trong cùng datacenter.

**Asynchronous Commit**: Có thể mất data, manual failover, không ảnh hưởng performance primary. Dùng cho DR ở remote site.

**Read-Only Secondary**:
```sql
-- Offload reporting queries sang secondary
-- Connection string: ApplicationIntent=ReadOnly → tự redirect sang readable secondary
Server=AG_Listener;Database=MyDB;ApplicationIntent=ReadOnly;
```

### 2. Transactional Replication

```
Publisher (source) → Distributor (middle) → Subscriber(s) (target)
```

Near real-time, one-directional. Publisher track changes qua transaction log. Use cases: read replicas, distribute data đến remote offices.

Hạn chế: tables phải có Primary Key, schema changes cần apply thủ công.

### 3. Merge Replication

Bi-directional sync: cả 2 bên đều có thể INSERT/UPDATE/DELETE. SQL Server tự thêm column rowguid để track. Conflict resolution: priority-based, first-wins, custom resolver.

Use case: distributed offices, mobile field workers. Đang bị deprecated dần → xem xét alternatives.

### 4. Log Shipping

```
Primary DB → Backup Log → Copy file → Restore on Standby (15-30 phút/lần)
```

```sql
-- STANDBY mode: cho phép đọc giữa các lần restore
RESTORE LOG MyDB FROM DISK = 'D:\LogShip\MyDB_Log.trn'
WITH STANDBY = 'D:\LogShip\MyDB_Undo.ldf';
```

Ưu điểm: đơn giản, không cần Enterprise, cross-version. Nhược điểm: manual failover, RPO = interval giữa các log backups.

### 5. Change Data Capture (CDC)

Track mọi thay đổi (INSERT/UPDATE/DELETE) ở table level, lưu vào change tables.

```sql
-- Bật CDC
EXEC sys.sp_cdc_enable_db;
EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo',
    @source_name = N'Products',
    @role_name = N'cdc_reader';

-- Query changes
DECLARE @from_lsn BINARY(10) = sys.fn_cdc_get_min_lsn('dbo_Products');
DECLARE @to_lsn BINARY(10) = sys.fn_cdc_get_max_lsn();
SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_Products(@from_lsn, @to_lsn, N'all');
-- __$operation: 1=delete, 2=insert, 3=before update, 4=after update
```

Use cases: ETL pipelines, audit trail, sync sang data warehouse, stream changes via Kafka/Debezium.

### 6. Change Tracking

Lightweight hơn CDC: chỉ biết **row nào thay đổi**, không biết **giá trị cũ**.

```sql
ALTER DATABASE MyDB SET CHANGE_TRACKING = ON
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON);
ALTER TABLE Products ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);

-- Query changes since version
DECLARE @lastVersion BIGINT = 0;
SELECT ct.ProductId, ct.SYS_CHANGE_OPERATION, p.ProductName, p.Price
FROM CHANGETABLE(CHANGES Products, @lastVersion) ct
LEFT JOIN Products p ON ct.ProductId = p.ProductId;
```

Use case: offline/mobile apps sync, nhẹ hơn CDC.

### 7. Application-Level Sync

**Outbox Pattern (recommended cho giao dịch chéo module)**:
```csharp
using var transaction = await dbContext.Database.BeginTransactionAsync();

var order = new Order { CustomerId = 1, TotalAmount = 500 };
dbContext.Orders.Add(order);

// Outbox event (cùng transaction → atomic)
dbContext.OutboxMessages.Add(new OutboxMessage
{
    EventType = "OrderCreated",
    Payload = JsonSerializer.Serialize(order),
    CreatedAt = DateTime.UtcNow,
    ProcessedAt = null
});

await dbContext.SaveChangesAsync();
await transaction.CommitAsync();
// Background worker poll OutboxMessages và publish lên RabbitMQ/Kafka
```

**CDC + Debezium + Kafka** (cross-platform sync):
```
SQL Server (CDC) → Debezium Connector → Kafka Topics → Consumers (any platform)
```

### Khi nào dùng gì?

| Nhu cầu | Giải pháp phù hợp |
|---|---|
| **HA/DR production** | Always On Availability Groups |
| **Read replicas** | AG readable secondary hoặc Transactional Replication |
| **Reporting database** | CDC + ETL (SSIS) hoặc AG secondary |
| **Cross-platform sync** | CDC + Kafka/Debezium |
| **Simple DR, budget thấp** | Log Shipping |
| **Offline/mobile sync** | Change Tracking |
| **Audit trail** | CDC (lưu old/new values) |
| **Data sync giữa các module** | Outbox Pattern + Background Worker |
| **Bi-directional (legacy)** | Merge Replication (cân nhắc alternatives) |

### Lưu ý quan trọng

1. **Always On AG** yêu cầu Enterprise Edition (Standard chỉ hỗ trợ Basic AG)
2. **CDC** tăng I/O trên transaction log, monitor disk performance
3. **Replication** cần monitor Distributor queue
4. **Log Shipping** standby DB ở trạng thái RESTORING hoặc STANDBY
5. Cross-version hoặc cross-platform → **CDC + message queue** linh hoạt nhất

---

*Tài liệu được chuẩn bị cho phỏng vấn vị trí Senior Database Developer với 4+ năm kinh nghiệm SQL Server.*
