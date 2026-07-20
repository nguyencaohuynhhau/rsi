# SQL Server Interview Questions - Senior Level

---

## Cau 1: Index la gi?

### Dinh nghia

Index la mot **cau truc du lieu** duoc tao tren mot hoac nhieu columns cua table, giup SQL Server **tim kiem du lieu nhanh hon** ma khong can quet toan bo bang. Tuong tu nhu **muc luc cua mot cuon sach** -- thay vi doc tu trang 1 den trang 500 de tim mot chu de, ban chi can mo muc luc, tra cuu ten chu de, va nhay thang den so trang tuong ung.

### Cau truc ben trong: B-Tree (Balanced Tree)

SQL Server su dung cau truc **B-Tree** cho hau het cac loai index:

```
            [Root Node]
           /     |     \
    [Inter-1] [Inter-2] [Inter-3]     ← Intermediate Nodes
    /   \      /   \      /   \
  [L1] [L2] [L3] [L4]  [L5] [L6]    ← Leaf Nodes
```

- **Root Node**: diem bat dau, chua cac key range de dinh huong tim kiem
- **Intermediate Nodes**: cac tang trung gian, thu hep pham vi tim kiem
- **Leaf Nodes**: tang la, chua du lieu thuc te hoac con tro den du lieu

Voi B-Tree, moi lan tim kiem chi can duyet **O(log n)** nodes thay vi **O(n)** rows. Mot bang 1 trieu rows chi can khoang 3-4 lan nhay (levels) la tim duoc row can thiet.

### Clustered Index vs Non-Clustered Index

| Dac diem | Clustered Index | Non-Clustered Index |
|---|---|---|
| Leaf node chua | **Du lieu thuc te** (data rows) | **Con tro** (RID hoac Clustered Key) |
| So luong / table | **Chi 1** | **Toi da 999** |
| Thu tu vat ly | Quyet dinh thu tu luu tru vat ly cua data | Khong anh huong thu tu vat ly |
| Kich thuoc | Chinh la bang du lieu | Cau truc rieng biet, nho hon |

```
-- Clustered Index (leaf = data rows)
Leaf: [1, "Nguyen Van A", "HN"] → [2, "Tran Thi B", "HCM"] → [3, "Le Van C", "DN"]

-- Non-Clustered Index (leaf = pointers)
Leaf: ["HCM" → Row 2] → ["DN" → Row 3] → ["HN" → Row 1]
            ↓                    ↓                  ↓
     (quay lai Clustered Index hoac RID de lay full row)
```

### Trade-offs

**Loi ich:**
- Tang toc **SELECT** dang ke -- tu quet toan bo bang xuong chi vai IO operations
- Ho tro **ORDER BY**, **GROUP BY**, **JOIN** hieu qua hon

**Chi phi:**
- **INSERT**: phai chen them entry vao moi index lien quan
- **UPDATE**: neu update column nam trong index, phai cap nhat ca data lan index
- **DELETE**: phai xoa entry tuong ung trong tat ca indexes
- **Storage**: moi index chiem them dung luong disk (co the len toi 20-40% kich thuoc bang goc)

**Vi du thuc te:** Mot bang `Orders` 10 trieu rows co 8 indexes. Moi lan INSERT 1 row, SQL Server phai ghi vao 9 noi (1 data + 8 indexes). Neu batch insert 100k rows/ngay, chi phi maintain index la dang ke.

> **Quy tac**: Index giong nhu "danh doi thoi gian ghi de lay thoi gian doc". Phu hop khi he thong **doc nhieu hon ghi** (OLTP thong thuong: 80% read, 20% write).

---

## Cau 2: Co bao nhieu loai index?

SQL Server ho tro nhieu loai index, moi loai phuc vu muc dich khac nhau:

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

- **Chi 1 per table** vi no quyet dinh thu tu luu tru vat ly cua du lieu
- **Nen chon column**: narrow (nho), unique, ever-increasing (IDENTITY, NEWSEQUENTIALID)
- **Tranh dung GUID (NEWID())** lam clustered key vi random → page splits → fragmentation

### 2. Non-Clustered Index

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Email
ON Employees(Email);
```

- Toi da **999** non-clustered indexes per table
- Tao cau truc B-Tree rieng biet, leaf node chua:
  - **Clustered Key** (neu bang co clustered index) de quay lai lay data
  - **RID** - Row Identifier (neu bang la heap, khong co clustered index)
- Moi key lookup tu non-clustered ve clustered index la **1 random I/O** → dat neu nhieu rows

### 3. Unique Index

```sql
CREATE UNIQUE INDEX UX_Employees_Email
ON Employees(Email);
```

- Dam bao **khong co gia tri trung lap** trong column(s)
- SQL Server cho phep **chi 1 NULL** trong Unique Index (vi coi NULL = NULL khi so sanh uniqueness)
- Muon nhieu NULLs + unique cho non-null → dung Filtered Index (xem ben duoi)
- `UNIQUE CONSTRAINT` thuc chat tao Unique Index phia sau

### 4. Composite Index (Multi-column Index)

```sql
CREATE INDEX IX_Orders_Status_Date
ON Orders(Status, OrderDate DESC, CustomerId);
```

**Leftmost Prefix Rule** -- rat quan trong:

```
Index: (Status, OrderDate, CustomerId)

-- SU DUNG DUOC index:
WHERE Status = 'Active'                                    -- dung cot 1
WHERE Status = 'Active' AND OrderDate > '2024-01-01'      -- dung cot 1, 2
WHERE Status = 'Active' AND OrderDate > '2024-01-01'
      AND CustomerId = 5                                   -- dung ca 3 cot

-- KHONG SU DUNG DUOC index:
WHERE OrderDate > '2024-01-01'                  -- thieu cot 1 (Status)
WHERE CustomerId = 5                            -- thieu cot 1, 2
WHERE OrderDate > '2024-01-01' AND CustomerId = 5  -- thieu cot 1
```

**Thu tu columns trong composite index rat quan trong:**
- Dat column co **equality filter** (=) truoc
- Dat column co **range filter** (>, <, BETWEEN) sau
- Dat column dung cho **ORDER BY** cuoi cung

### 5. Covering Index / INCLUDE Columns

```sql
CREATE INDEX IX_Orders_CustomerId
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount, Status);
```

- **INCLUDE columns** duoc luu o **leaf level** cua index nhung **khong nam trong key**
- Giup **tranh Key Lookup** hoan toan -- query chi can doc index ma khong can quay lai table
- INCLUDE columns **khong anh huong thu tu sap xep** cua index
- Khi nao dung: query can them vai columns ngoai key columns de SELECT

```sql
-- Query nay duoc "cover" hoan toan boi index tren:
SELECT OrderDate, TotalAmount, Status
FROM Orders
WHERE CustomerId = 123;
-- Execution plan se hien: Index Seek (khong co Key Lookup)
```

### 6. Filtered Index

```sql
-- Chi index nhung orders dang active
CREATE INDEX IX_Orders_Active
ON Orders(OrderDate, CustomerId)
WHERE Status = 'Active';

-- Unique index cho phep nhieu NULLs
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN)
WHERE SSN IS NOT NULL;
```

- Chi index **mot subset** cua rows thoa dieu kien WHERE
- **Nho hon**, **nhanh hon** maintain, **it ton storage**
- Rat huu ich khi phan lon queries chi quan tam mot phan du lieu (VD: chi records active, chi don hang chua xu ly)

### 7. Columnstore Index

```sql
-- Clustered Columnstore (thay the toan bo storage cua table)
CREATE CLUSTERED COLUMNSTORE INDEX CCIX_FactSales
ON FactSales;

-- Non-Clustered Columnstore (them vao table co san)
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCIX_Orders_Analytics
ON Orders(OrderDate, ProductId, Quantity, TotalAmount);
```

- Luu tru du lieu **theo cot** thay vi theo hang
- **Nen du lieu** rat tot (10x compression ratio la binh thuong)
- **Batch mode execution**: xu ly hang ngan rows cung luc thay vi tung row
- Phu hop cho **analytics, reporting, data warehouse** (OLAP)
- Khong phu hop cho **point lookups** (tim 1 row cu the)

### 8. Full-Text Index

```sql
CREATE FULLTEXT CATALOG ftCatalog AS DEFAULT;

CREATE FULLTEXT INDEX ON Articles(Title, Content)
KEY INDEX PK_Articles ON ftCatalog;

-- Su dung:
SELECT * FROM Articles
WHERE CONTAINS(Content, '"SQL Server" NEAR "performance"');

SELECT * FROM Articles
WHERE FREETEXT(Content, N'toi uu hieu suat truy van');
```

- Tim kiem **noi dung van ban** phuc tap: tu dong, cum tu, gan nhau, dong nghia
- SQL Server dung **inverted index** rieng biet, khong phai B-Tree

### 9. Spatial Index

```sql
CREATE SPATIAL INDEX SIX_Stores_Location
ON Stores(GeoLocation);
```

- Danh cho du lieu **geography/geometry**: toa do GPS, vung, duong
- Su dung cau truc grid hierarchy de chia khong gian

### 10. XML Index

```sql
CREATE PRIMARY XML INDEX PIX_Config
ON Settings(XmlData);

CREATE XML INDEX SIX_Config_Path
ON Settings(XmlData)
USING XML INDEX PIX_Config FOR PATH;
```

- Primary XML Index: shred XML thanh relational format noi bo
- Secondary XML Index: toi uu cho PATH, VALUE, hoac PROPERTY queries

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

Gender:  2 / 1,000,000 = 0.000002  → Cuc ky thap
Email:   1,000,000 / 1,000,000 = 1.0  → Cuc ky cao (ly tuong)
Status:  5 / 1,000,000 = 0.000005  → Thap
```

Khi selectivity thap, index tra ve **qua nhieu rows** (50% neu Male/Female dong deu). SQL Server Query Optimizer se **tu dong bo qua index** va chon **Table Scan** (hoac Clustered Index Scan) vi:

```
-- Gia su 1 trieu rows, 50% Male, 50% Female
-- Dung index: 500,000 Index Seeks + 500,000 Key Lookups = 1,000,000 random I/O
-- Table scan: 1 sequential scan qua toan bo table = nhanh hon nhieu
```

**Nguong selectivity**: SQL Server thuong chon index khi query tra ve khoang **< 15-30%** tong rows (con tuy vao nhieu yeu to khac). Gender voi 50/50 se khong bao gio dat nguong nay.

### Khi nao CO THE danh index cho Gender?

**Truong hop 1: Filtered Index khi du lieu lech (skewed data)**

```sql
-- 95% Male, chi 5% Female → filter Female co selectivity cao
CREATE INDEX IX_Employees_Gender_Female
ON Employees(Gender, HireDate)
WHERE Gender = 'Female';

-- Query nay se dung index:
SELECT * FROM Employees
WHERE Gender = 'Female' AND HireDate > '2024-01-01';
```

**Truong hop 2: Composite Index ket hop voi columns khac**

```sql
-- Gender ket hop voi columns co selectivity cao
CREATE INDEX IX_Employees_Gender_DeptId_Salary
ON Employees(DepartmentId, Gender, Salary);

-- Query nay co the dung index hieu qua:
SELECT * FROM Employees
WHERE DepartmentId = 10 AND Gender = 'Male' AND Salary > 50000;
```

**Truong hop 3: Covering Index**

```sql
CREATE INDEX IX_Employees_Gender_Cover
ON Employees(Gender)
INCLUDE (FullName, Email);

-- Neu query chi can cac columns trong index → Index-only scan, khong Key Lookup
SELECT FullName, Email FROM Employees WHERE Gender = 'Female';
-- Van scan nhieu rows nhung tranh duoc random I/O ve clustered index
```

### bit vs string cho Gender

```sql
-- bit: 1 byte cho moi 8 bit columns (tiet kiem)
Gender BIT -- 0 hoac 1

-- string: ton hon nhung doc duoc
Gender VARCHAR(10) -- 'Male', 'Female'
Gender CHAR(1) -- 'M', 'F'
```

Du dung kieu du lieu nao, **van de cot loi la selectivity thap** → index don le khong hieu qua. Tuy nhien, `BIT` hoac `CHAR(1)` giup **Composite Index nho hon** (vi key size nho), nen uu tien kieu du lieu nho.

### Ket luan

| Tinh huong | Danh index? |
|---|---|
| Index don le tren Gender | **Khong** |
| Du lieu lech (5/95%) + Filtered Index | **Co the** |
| Composite Index voi columns khac | **Co** |
| Covering Index tranh Key Lookup | **Xem xet** |

---

## Cau 4: Co danh duoc index tren truong Nullable khong?

### Cau tra loi: CO - SQL Server hoan toan ho tro

SQL Server **luu tru gia tri NULL trong index B-Tree** binh thuong. NULL duoc coi la gia tri nho nhat, nen:
- Voi index **ASC**: cac NULL entries nam o **dau** cua index
- Voi index **DESC**: cac NULL entries nam o **cuoi** cua index

### NULL va Index Seek

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
-- CA HAI query deu CO THE su dung Index Seek:
SELECT * FROM Products WHERE DiscontinuedDate IS NULL;
-- → Index Seek tren phan dau cua index (vi NULL nam dau voi ASC)

SELECT * FROM Products WHERE DiscontinuedDate IS NOT NULL;
-- → Index Seek (range scan phan khong NULL)

SELECT * FROM Products WHERE DiscontinuedDate = '2024-06-15';
-- → Index Seek binh thuong
```

### Filtered Index toi uu cho Nullable columns

Neu phan lon rows co gia tri NULL va query thuong chi quan tam non-null:

```sql
-- Index day du: 1 trieu rows, 900k NULL, 100k co gia tri
CREATE INDEX IX_Products_DiscontinuedDate
ON Products(DiscontinuedDate);
-- Kich thuoc: lon, chua ca 900k NULL entries khong can thiet

-- Filtered Index: chi 100k rows
CREATE INDEX IX_Products_DiscontinuedDate_NotNull
ON Products(DiscontinuedDate)
WHERE DiscontinuedDate IS NOT NULL;
-- Kich thuoc: nho hon 10x, nhanh hon maintain
```

### Unique Index va NULL - Dac biet quan trong

```sql
-- Unique Index chi cho phep TOI DA 1 NULL
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN);

INSERT INTO Employees (SSN) VALUES (NULL);  -- OK
INSERT INTO Employees (SSN) VALUES (NULL);  -- LOI! Duplicate key
```

SQL Server coi **NULL = NULL** trong ngur canh Unique Index (khac voi phep so sanh thong thuong noi NULL = NULL tra ve UNKNOWN).

**Giai phap: Filtered Unique Index**

```sql
-- Cho phep nhieu NULLs, nhung dam bao unique cho cac gia tri non-null
CREATE UNIQUE INDEX UX_Employees_SSN
ON Employees(SSN)
WHERE SSN IS NOT NULL;

INSERT INTO Employees (SSN) VALUES (NULL);           -- OK
INSERT INTO Employees (SSN) VALUES (NULL);           -- OK (khong bi kiem tra)
INSERT INTO Employees (SSN) VALUES ('123-45-6789');  -- OK
INSERT INTO Employees (SSN) VALUES ('123-45-6789');  -- LOI! Duplicate
```

### ANSI_NULLS va anh huong den queries

```sql
SET ANSI_NULLS ON; -- Mac dinh, tuan thu chuan SQL
-- WHERE Column = NULL  → khong tra ve gi (sai)
-- WHERE Column IS NULL → dung

SET ANSI_NULLS OFF; -- Khong khuyen khich, se bi loai bo trong tuong lai
-- WHERE Column = NULL  → hoat dong (nhung khong nen dung)
```

### Best Practices

| Tinh huong | Khuyen nghi |
|---|---|
| Column phan lon NULL, query filter IS NOT NULL | **Filtered Index WHERE col IS NOT NULL** |
| Can unique nhung cho phep nhieu NULLs | **Filtered Unique Index WHERE col IS NOT NULL** |
| Column it NULL, query ca NULL lan non-null | **Index binh thuong** |
| Column trong composite index co the NULL | **Dat nullable column sau** trong index key |

---

## Cau 5: WHERE cham thi danh index - SELECT nhanh hon dung khong?

### Cau tra loi: KHONG PHAI LUC NAO CUNG DUNG

Day la **misconception pho bien nhat** ve indexing. Danh index khong phai "vien dan bac" -- co nhieu truong hop index ton tai nhung **hoan toan vo dung**.

### 1. Non-SARGable Queries (Search ARGument-able)

**SARGable** la kha nang SQL Server dung index de tim kiem. Cac truong hop **KHONG SARGable**:

#### a) Function tren column

```sql
-- KHONG dung index (scan toan bo)
SELECT * FROM Orders
WHERE YEAR(OrderDate) = 2024;
-- SQL Server phai tinh YEAR() cho TUNG ROW de so sanh

-- FIX: viet lai thanh range query
SELECT * FROM Orders
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
-- Index Seek tren OrderDate

-- Tuong tu:
WHERE UPPER(Name) = 'NGUYEN VAN A'     -- KHONG SARGable
WHERE Name = 'Nguyen Van A'             -- SARGable (dung collation)

WHERE DATEDIFF(DAY, CreateDate, GETDATE()) < 30  -- KHONG
WHERE CreateDate > DATEADD(DAY, -30, GETDATE())  -- SARGable
```

#### b) LIKE voi wildcard o dau

```sql
WHERE Name LIKE '%abc'     -- Full scan, KHONG dung index
WHERE Name LIKE '%abc%'    -- Full scan, KHONG dung index
WHERE Name LIKE 'abc%'     -- Index Seek (SARGable)
```

#### c) Implicit Conversion (chuyen doi ngam dinh)

```sql
-- Column la VARCHAR, nhung so sanh voi INT
-- PhoneNumber VARCHAR(20), co index
SELECT * FROM Customers
WHERE PhoneNumber = 0912345678;
-- SQL Server phai CONVERT toan bo PhoneNumber sang INT de so sanh → SCAN

-- FIX: truyen dung kieu du lieu
SELECT * FROM Customers
WHERE PhoneNumber = '0912345678';
-- Index Seek
```

**Quy tac Data Type Precedence**: SQL Server convert kieu co do uu tien thap sang kieu co do uu tien cao. VARCHAR < INT, nen VARCHAR bi convert → scan toan bo column.

#### d) Phep phu dinh

```sql
WHERE Status != 'Active'       -- Thuong scan (tra ve nhieu rows)
WHERE Status NOT IN ('A','B')   -- Thuong scan
WHERE NOT EXISTS (...)          -- Tuy truong hop
```

#### e) OR conditions

```sql
-- Co the khong dung index hieu qua
SELECT * FROM Orders
WHERE CustomerId = 123 OR ProductId = 456;

-- FIX: dung UNION
SELECT * FROM Orders WHERE CustomerId = 123
UNION ALL
SELECT * FROM Orders WHERE ProductId = 456;
-- Moi query dung duoc index rieng
```

### 2. Selectivity thap - Optimizer chon Scan

```sql
-- Index ton tai tren Status, nhung chi co 3 gia tri phan biet
-- Status = 'Active' tra ve 80% rows
SELECT * FROM Orders WHERE Status = 'Active';
-- Optimizer: "Index Seek 800k rows + 800k Key Lookups > Table Scan"
-- → Chon Table Scan (dung)
```

### 3. Key Lookup qua dat

```sql
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);

SELECT CustomerId, OrderDate, TotalAmount, ShipAddress, Notes
FROM Orders
WHERE CustomerId = 123;
-- Tim duoc 500 rows qua index → 500 Key Lookups ve Clustered Index
-- Moi Key Lookup la 1 random I/O → cham

-- FIX: Covering Index
CREATE INDEX IX_Orders_CustomerId
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount, ShipAddress, Notes);
-- 0 Key Lookups → nhanh hon nhieu
```

### 4. Parameter Sniffing

```sql
CREATE PROCEDURE GetOrders @Status VARCHAR(20)
AS
    SELECT * FROM Orders WHERE Status = @Status;

-- Lan dau goi voi 'Cancelled' (1% rows) → plan dung Index Seek
EXEC GetOrders 'Cancelled';

-- Lan sau goi voi 'Active' (80% rows) → REUSE plan cu → Index Seek + 800k Key Lookups
EXEC GetOrders 'Active';
-- Cuc ky cham! Plan khong phu hop voi gia tri nay
```

### 5. Qua nhieu Indexes → Write cham

```sql
-- Bang Orders co 15 indexes
INSERT INTO Orders (...) VALUES (...);
-- SQL Server phai ghi vao: 1 Clustered Index + 15 Non-Clustered Indexes = 16 write operations

-- Moi UPDATE tren column nam trong index phai update ca index
UPDATE Orders SET Status = 'Shipped' WHERE OrderId = 123;
-- Neu Status nam trong 5 indexes → 5 index updates
```

### Ket luan: Luon xem Execution Plan

```sql
-- Bat Actual Execution Plan truoc va sau khi danh index
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Xem Execution Plan
-- Kiem tra: Scan vs Seek, Key Lookup co khong, Estimated vs Actual rows
-- So sanh logical reads truoc/sau
```

**Quy trinh dung:**
1. Xac dinh query cham (qua monitoring hoac pg_stat_statements tuong duong la Query Store trong SQL Server)
2. Xem **Execution Plan** hien tai
3. Phan tich **WHERE, JOIN, ORDER BY** dang dung columns nao
4. Kiem tra query co **SARGable** khong -- fix query truoc
5. Tao index phu hop (composite, covering, filtered)
6. So sanh Execution Plan moi -- xac nhan cai thien
7. Monitor **write performance** de dam bao khong bi anh huong qua nhieu

---

## Cau 6: Cau lenh SELECT va thu tu xu ly logic

### Thu tu VIET vs Thu tu XU LY LOGIC

Day la **mot trong nhung kien thuc nen tang quan trong nhat** cua SQL. Thu tu viet va thu tu xu ly **hoan toan khac nhau**:

```
THU TU VIET (Syntax)          THU TU XU LY LOGIC (Execution)
─────────────────              ─────────────────────────────
SELECT          (5)    ←──    1. FROM + JOINs
DISTINCT        (6)    ←──    2. ON (dieu kien JOIN)
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

### Chi tiet tung buoc

| Buoc | Menh de | Chuc nang |
|------|---------|-----------|
| 1 | **FROM + JOINs** | Xac dinh nguon du lieu, thuc hien phep JOIN tao Virtual Table |
| 2 | **ON** | Ap dung dieu kien JOIN, loc cac rows match |
| 3 | **WHERE** | Loc rows theo dieu kien (truoc khi group) |
| 4 | **GROUP BY** | Nhom cac rows co cung gia tri thanh cac groups |
| 5 | **HAVING** | Loc groups theo dieu kien aggregate |
| 6 | **SELECT** | Chon columns, tinh toan expressions, alias |
| 7 | **DISTINCT** | Loai bo cac rows trung lap |
| 8 | **ORDER BY** | Sap xep ket qua |
| 9 | **TOP / OFFSET-FETCH** | Gioi han so luong rows tra ve |

### Vi du thuc te: Bao cao doanh thu theo khach hang

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

**Walk-through tung buoc:**

**Buoc 1 - FROM + JOIN:** Ket hop bang `Customers` voi `Orders` qua `CustomerId`. Tao virtual table chua tat ca cac cap (customer, order) match.

**Buoc 2 - ON:** Chi giu cac rows co `c.CustomerId = o.CustomerId`.

**Buoc 3 - WHERE:** Loc chi nhung orders tu 2024 tro di va co Status = 'Completed'. Rows khong thoa bi loai.

**Buoc 4 - GROUP BY:** Nhom cac rows con lai theo `c.CustomerName`. Moi group la mot khach hang voi tat ca orders cua ho.

**Buoc 5 - HAVING:** Chi giu nhung groups co `SUM(TotalAmount) > 1,000,000`. Khach hang doanh thu thap bi loai.

**Buoc 6 - SELECT:** Tinh `COUNT(o.OrderId)`, `SUM(o.TotalAmount)`, lay `c.CustomerName`. Dat alias `TotalOrders`, `Revenue`.

**Buoc 7 - DISTINCT:** Loai bo cac rows trung lap (neu co).

**Buoc 8 - ORDER BY:** Sap xep theo `Revenue DESC`. Luu y: ORDER BY co the dung alias vi SELECT da chay truoc.

**Buoc 9 - TOP 10:** Lay 10 rows dau tien sau khi sap xep.

### Tai sao thu tu nay quan trong?

**Cau hoi thuong gap trong interview:**

```sql
-- Tai sao query nay LOI?
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE OrderYear = 2024;
-- Loi: "Invalid column name 'OrderYear'"
-- Vi WHERE (buoc 3) chay TRUOC SELECT (buoc 6)
-- Alias 'OrderYear' chua ton tai khi WHERE thuc thi

-- FIX:
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE YEAR(OrderDate) = 2024;

-- Hoac tot hon (SARGable):
SELECT FullName, YEAR(OrderDate) AS OrderYear
FROM Orders
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
```

```sql
-- Tai sao ORDER BY co the dung alias nhung WHERE thi khong?
SELECT FullName, SUM(Amount) AS TotalAmount
FROM Orders
GROUP BY FullName
ORDER BY TotalAmount DESC;  -- OK! ORDER BY (buoc 8) chay SAU SELECT (buoc 6)
```

```sql
-- Tai sao HAVING dung duoc aggregate nhung WHERE thi khong?
-- WHERE (buoc 3) chay TRUOC GROUP BY (buoc 4) → chua co groups de aggregate
-- HAVING (buoc 5) chay SAU GROUP BY (buoc 4) → da co groups
SELECT DepartmentId, AVG(Salary)
FROM Employees
WHERE AVG(Salary) > 5000  -- LOI! Chua co groups
GROUP BY DepartmentId;

SELECT DepartmentId, AVG(Salary)
FROM Employees
GROUP BY DepartmentId
HAVING AVG(Salary) > 5000;  -- OK! Da group xong
```

---

## Cau 7: Co bao nhieu kieu JOIN?

SQL Server ho tro **6 kieu JOIN** chinh:

### 1. INNER JOIN

```
Customers         Orders            Result (INNER JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Binh  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cuong │   │ 2  │ 150$ │     │ Binh  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     └───────┴──────┘
                  └────┴──────┘
-- Cuong (ID=3) khong co order → bi loai
-- Order cua CustomerId=4 khong co customer → bi loai
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Chi tra ve rows co match O CA HAI bang
```

### 2. LEFT JOIN (LEFT OUTER JOIN)

```
Customers         Orders            Result (LEFT JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Binh  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cuong │   │ 2  │ 150$ │     │ Binh  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     │ Cuong │ NULL │
                  └────┴──────┘     └───────┴──────┘
-- Cuong khong co order → van giu, Orders columns = NULL
-- Order CustomerId=4 khong co customer → bi loai (vi khong nam o LEFT)
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Tat ca customers, ke ca nhung nguoi chua co order

-- Tim customers CHUA co order:
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
-- Tat ca orders, ke ca nhung orders khong co customer match
-- Tuong duong voi LEFT JOIN nhung dao 2 bang
```

Trong thuc te, **it khi dung RIGHT JOIN** -- thuong viet lai thanh LEFT JOIN cho de doc.

### 4. FULL OUTER JOIN

```
Customers         Orders            Result (FULL OUTER JOIN)
┌────┬───────┐   ┌────┬──────┐     ┌───────┬──────┐
│ 1  │ An    │   │ 1  │ 100$ │     │ An    │ 100$ │
│ 2  │ Binh  │   │ 1  │ 200$ │     │ An    │ 200$ │
│ 3  │ Cuong │   │ 2  │ 150$ │     │ Binh  │ 150$ │
└────┴───────┘   │ 4  │ 300$ │     │ Cuong │ NULL │
                  └────┴──────┘     │ NULL  │ 300$ │
                                    └───────┴──────┘
-- Cuong khong co order → giu, order = NULL
-- Order CustomerId=4 khong co customer → giu, customer = NULL
```

```sql
SELECT c.CustomerName, o.TotalAmount
FROM Customers c
FULL OUTER JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Tat ca rows tu CA HAI bang, NULL cho phia khong match

-- Use case: doi chieu du lieu giua 2 he thong
SELECT
    COALESCE(a.Id, b.Id) AS Id,
    a.Amount AS SystemA_Amount,
    b.Amount AS SystemB_Amount
FROM SystemA a
FULL OUTER JOIN SystemB b ON a.Id = b.Id
WHERE a.Id IS NULL OR b.Id IS NULL;  -- Tim records chi co 1 phia
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
-- Tich Descartes: moi row bang A ket hop voi MOI row bang B
SELECT c.Color, s.Size
FROM Colors c
CROSS JOIN Sizes s;

-- Use case: tao lich
SELECT d.Date, s.ShiftName
FROM Calendar d
CROSS JOIN Shifts s
WHERE d.Date BETWEEN '2024-01-01' AND '2024-12-31';

-- CANH BAO: 10,000 rows x 10,000 rows = 100 TRIEU rows!
```

### 6. SELF JOIN

```sql
-- Bang Employees co cot ManagerId tro ve chinh bang do
CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    FullName NVARCHAR(200),
    ManagerId INT REFERENCES Employees(EmployeeId)
);

-- Tim nhan vien va ten manager cua ho
SELECT
    e.FullName AS Employee,
    m.FullName AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;

-- CEO/Director khong co manager → ManagerId = NULL → LEFT JOIN giu lai
```

### Performance Tips

```sql
-- 1. INNER JOIN nhanh nhat (loai nhieu rows nhat)
-- Thu tu uu tien: INNER > LEFT/RIGHT > FULL > CROSS

-- 2. Dam bao co index tren JOIN columns
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);
-- LEFT JOIN Customers c → Orders o: index tren o.CustomerId rat quan trong

-- 3. Tranh Implicit Conversion trong JOIN
-- BAD: CustomerId la INT o bang A nhung VARCHAR o bang B
FROM TableA a JOIN TableB b ON a.CustomerId = b.CustomerId
-- Neu khac data type → convert toan bo 1 phia → khong dung index

-- 4. SQL Server Optimizer tu chon JOIN algorithm:
-- Nested Loop: tot cho bang nho INNER + bang lon co index OUTER
-- Merge Join: tot khi ca 2 input da sorted (VD: 2 clustered index)
-- Hash Join: tot khi khong co index, bang lon, khong sort
-- Thuong khong can dung JOIN hints, de optimizer tu quyet dinh
```

---

## Cau 8: WHERE vs HAVING?

### Su khac biet cot loi

| Dac diem | WHERE | HAVING |
|----------|-------|--------|
| Thoi diem thuc thi | **Truoc GROUP BY** (buoc 3) | **Sau GROUP BY** (buoc 5) |
| Loc | **Tung row** rieng le | **Tung group** |
| Aggregate functions | **KHONG** dung duoc | **CO** dung duoc |
| Hieu suat | **Tot hon** (giam du lieu som) | Phai xu ly nhieu du lieu hon |

### Vi du minh hoa

```sql
-- Bai toan: Tim phong ban co hon 10 nhan vien dang Active

-- CACH SAI: filter trong HAVING (cham)
SELECT DepartmentId, COUNT(*) AS EmpCount
FROM Employees
GROUP BY DepartmentId
HAVING COUNT(*) > 10 AND DepartmentId != 5;
-- Buoc 3 (WHERE): khong co → xu ly TAT CA rows (ke ca Dept 5)
-- Buoc 4 (GROUP BY): nhom tat ca rows, ke ca Dept 5
-- Buoc 5 (HAVING): loai Dept 5 va groups <= 10
-- → Lang phi: da group Dept 5 roi moi loai

-- CACH DUNG: filter trong WHERE (nhanh)
SELECT DepartmentId, COUNT(*) AS EmpCount
FROM Employees
WHERE DepartmentId != 5        -- Loai Dept 5 TRUOC khi group
GROUP BY DepartmentId
HAVING COUNT(*) > 10;
-- Buoc 3 (WHERE): loai tat ca rows Dept 5 → it rows hon
-- Buoc 4 (GROUP BY): nhom it rows hon → nhanh hon
-- Buoc 5 (HAVING): chi kiem tra aggregate condition
```

### Quy tac vang

> **Neu dieu kien KHONG can aggregate function → dung WHERE.**
> **Neu dieu kien CAN aggregate function (SUM, COUNT, AVG, MIN, MAX) → dung HAVING.**

```sql
-- WHERE: loc tren column values cua tung row
WHERE Status = 'Active'
WHERE OrderDate >= '2024-01-01'
WHERE DepartmentId IN (1, 2, 3)

-- HAVING: loc tren ket qua aggregate cua group
HAVING COUNT(*) > 10
HAVING SUM(TotalAmount) > 1000000
HAVING AVG(Salary) BETWEEN 5000 AND 10000
```

### Truong hop dac biet: HAVING khong co GROUP BY

```sql
-- Treats toan bo result set nhu 1 group duy nhat
SELECT COUNT(*) AS TotalActive
FROM Employees
HAVING COUNT(*) > 100;
-- Neu co hon 100 employees → tra ve count
-- Neu khong → tra ve 0 rows (khong phai 0, ma la KHONG CO ROW nao)
```

### So sanh performance thuc te

```sql
-- Bang Orders: 10 trieu rows, 1 trieu rows co Status = 'Cancelled'

-- Query 1: HAVING filter (cham)
SELECT CustomerId, SUM(TotalAmount)
FROM Orders
GROUP BY CustomerId
HAVING CustomerId != 999;
-- Xu ly: 10 trieu rows → GROUP BY → HAVING loai 1 customer
-- Logical reads: ~50,000 pages

-- Query 2: WHERE filter (nhanh)
SELECT CustomerId, SUM(TotalAmount)
FROM Orders
WHERE CustomerId != 999
GROUP BY CustomerId;
-- Xu ly: loai rows cua Customer 999 truoc → GROUP BY it rows hon
-- Logical reads: ~49,950 pages (tiet kiem I/O)

-- Chenh lech tuy thuoc vao luong du lieu bi loc.
-- Cang nhieu rows bi loc boi WHERE → cang tiet kiem.
```

---

## Cau 9: UNION vs UNION ALL?

### Su khac biet chinh

| Dac diem | UNION | UNION ALL |
|----------|-------|-----------|
| Ket qua trung lap | **Loai bo** | **Giu tat ca** |
| Internal operation | Sort + Distinct (hoac Hash Match) | Chi noi (Concatenation) |
| Performance | **Cham hon** | **Nhanh hon** |
| Do phuc tap | O(n log n) - phai sort/hash | O(n) - chi append |

### Vi du minh hoa

```sql
-- Du lieu:
-- TableA: 1, 2, 3
-- TableB: 2, 3, 4

SELECT Id FROM TableA
UNION
SELECT Id FROM TableB;
-- Ket qua: 1, 2, 3, 4 (loai duplicate 2 va 3)

SELECT Id FROM TableA
UNION ALL
SELECT Id FROM TableB;
-- Ket qua: 1, 2, 3, 2, 3, 4 (giu tat ca 6 rows)
```

### Execution Plan so sanh

```
UNION:                              UNION ALL:
┌──────────┐                        ┌──────────────┐
│ Sort     │  ← Chi phi cao         │ Concatenation │  ← Chi phi thap
│ Distinct │  O(n log n)            │              │  O(n)
├──────────┤                        ├──────────────┤
│ Concat   │                        │ Table A      │
├──────────┤                        │ Table B      │
│ Table A  │                        └──────────────┘
│ Table B  │
└──────────┘
```

UNION phai **doc toan bo du lieu, sort, va loai duplicates** -- voi du lieu lon, chi phi nay rat dang ke (tempdb spills, CPU).

### Khi nao dung cai nao?

**Dung UNION ALL khi:**

```sql
-- 1. Biet chac khong co duplicate (moi query tu bang/partition khac nhau)
SELECT OrderId, Amount FROM Orders_2023
UNION ALL
SELECT OrderId, Amount FROM Orders_2024;
-- Moi partition co du lieu rieng biet → khong trung

-- 2. Chap nhan duplicate (khong anh huong logic)
SELECT ProductId FROM RecentlyViewed
UNION ALL
SELECT ProductId FROM Recommendations;
-- Trung khong sao, chi can danh sach de hien thi

-- 3. Performance la uu tien
-- Du lieu lon, can ket qua nhanh
```

**Dung UNION khi:**

```sql
-- Can dam bao unique (VD: bao cao tong hop khong trung)
SELECT Email FROM Newsletter_Subscribers
UNION
SELECT Email FROM Active_Customers;
-- Mot nguoi vua subscribe vua la customer → chi hien 1 lan
```

### Cac quy tac quan trong

```sql
-- 1. SO LUONG COLUMNS phai bang nhau
SELECT Id, Name FROM TableA
UNION ALL
SELECT Id FROM TableB;        -- LOI! Khac so columns

-- 2. DATA TYPES phai tuong thich (compatible)
SELECT Id, Name FROM TableA         -- Id: INT, Name: VARCHAR
UNION ALL
SELECT Code, Description FROM TableB -- Code: INT, Description: VARCHAR
-- OK neu data types tuong thich. SQL Server se implicit convert neu can.

-- 3. Column names lay tu query DAU TIEN
SELECT Id AS MaKH, Name AS TenKH FROM TableA
UNION ALL
SELECT Code, Description FROM TableB;
-- Ket qua co columns: MaKH, TenKH (lay tu query dau)

-- 4. ORDER BY chi dung duoc o CUOI CUNG
SELECT Id FROM TableA
UNION ALL
SELECT Id FROM TableB
ORDER BY Id;    -- ORDER BY ap dung cho TOAN BO ket qua

-- Muon sort tung phan → dung subquery hoac CTE
```

### Vi du thuc te: Combine du lieu tu nhieu nguon

```sql
-- Bao cao tong hop doanh thu tu nhieu kenh ban hang
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
-- UNION ALL vi moi bang chua du lieu rieng biet, khong trung
-- Performance: chi Concatenation, khong Sort Distinct
```

### Meo nho

| Cau hoi | Tra loi |
|---------|---------|
| Mac dinh nen dung gi? | **UNION ALL** (nhanh hon, chi dung UNION khi that su can loai trung) |
| Co bao nhieu UNION/UNION ALL? | Khong gioi han, nhung cang nhieu → cang cham |
| Co the ket hop UNION va UNION ALL? | **Co** |
| UNION co dung index khong? | Moi SELECT con co the dung index rieng, nhung Sort Distinct o ngoai thi khong |

```sql
-- Ket hop ca UNION va UNION ALL
SELECT Id FROM TableA
UNION ALL          -- Giu duplicate giua A va B
SELECT Id FROM TableB
UNION              -- Loai duplicate voi C
SELECT Id FROM TableC;
-- Luu y: thu tu uu tien co the gay nhap nham → dung parentheses neu can
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
| READ UNCOMMITTED | Co | Co | Co | Không shared lock |
| READ COMMITTED (default) | Khong | Co | Co | Shared lock giữ trong lúc đọc |
| REPEATABLE READ | Khong | Khong | Co | Shared lock giữ đến hết transaction |
| SERIALIZABLE | Khong | Khong | Khong | Range locks |
| SNAPSHOT | Khong | Khong | Khong | Row versioning (TempDB) |

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

**Outbox Pattern (recommended cho microservices)**:
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
| **Microservices event sync** | Outbox Pattern + Message Queue |
| **Bi-directional (legacy)** | Merge Replication (cân nhắc alternatives) |

### Lưu ý quan trọng

1. **Always On AG** yêu cầu Enterprise Edition (Standard chỉ hỗ trợ Basic AG)
2. **CDC** tăng I/O trên transaction log, monitor disk performance
3. **Replication** cần monitor Distributor queue
4. **Log Shipping** standby DB ở trạng thái RESTORING hoặc STANDBY
5. Cross-version hoặc cross-platform → **CDC + message queue** linh hoạt nhất

---

*Tài liệu được chuẩn bị cho phỏng vấn vị trí Senior Database Developer với 4+ năm kinh nghiệm SQL Server.*
