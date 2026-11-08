# Đại Toàn Thư So Sánh Chỉ Số Trang Bị & 30 Build Mạnh Nhất Phái Tiêu Dao (30 Server TLBB 2007–2026)

> **Tài liệu chuẩn hóa End-Game** — Trích xuất trực tiếp từ cơ sở dữ liệu cấu hình thực tế (`EquipBase.txt`, `ItemSegValue.txt`, `GreenEquipBase.txt`, `EquipSetAttr.txt`, `GemInfo.txt`, `StandardImpact.txt`, và các bảng hệ thống phụ trợ) của **30 tổ hợp Server & Client** trong [DANH_SACH_SERVER_TLBB.md](file:///e:/games/TLBB/TLBB%20hoixua/docs/DANH_SACH_SERVER_TLBB.md).
> **Nguyên tắc lọc dữ liệu:** Tuyệt đối **loại bỏ đồ GM/Admin** (`GM`, `Admin`, `Test`, `Vô Địch`, `Hội Viên` +9999), **loại bỏ phôi rác/chưa mở** (`未使用`, `Chưa sử dụng`, `Tân Thủ`), chỉ giữ lại các trang bị End-Game thực tế đạt đỉnh sức mạnh hợp lệ của từng phiên bản.

---

## Phần I: Cơ Chế Sức Mạnh Phái Tiêu Dao (逍遥) & Khung Chuẩn Hóa Build

### 1. Bản chất Sát thương & Vị thế Meta của Phái Tiêu Dao

Trong toàn bộ lịch sử *Thiên Long Bát Bộ* (từ bản **0.26 năm 2007** đến bản **ChangYou 3.72 / Origin 2025**), **Tiêu Dao (逍遥)** luôn giữ vị thế độc tôn về khả năng **khuếch đại sát thương (Damage Amplification)**, **khống chế diện rộng / đơn mục tiêu** và **thả diều cơ động (Kiting)**:

1. **Kỹ năng linh hồn — *Khê Sơn Hành Lữ* (溪山行旅):**
   - Gây trạng thái **Hư Nhược (虚弱)**: **Phóng đại 100% (x2.0) toàn bộ sát thương** mục tiêu phải nhận trong thời gian hiệu lực (cả sát thương thuộc tính lẫn nội/ngoại công, xuyên thích).
   - Ở các bản có **Tâm Pháp / Tu Luyện / Tiến Cấp Kỹ Năng (Võ Đạo / Thiên Đạo)** (như STT 10..15, 18..21, 28, 29), *Khê Sơn Hành Lữ* còn được tăng xác suất trúng và rút ngắn thời gian hồi chiêu, biến **Chính xác (Hit)** và **Thân pháp** thành chỉ số sống còn để không bị trượt (*Miss*) nhịp *Khê Sơn*.
2. **Hệ số Thuộc tính Kép (Hỏa Công & Độc Công) + Nội Công:**
   - **Hệ số sát thương thuộc tính chuẩn của Tiêu Dao:**
     - **Hỏa công (Fire Attack - Chủ đạo):** Hệ số **`1.30`** (mỗi `1` điểm Hỏa công tăng `1.3` sát thương cơ bản trước khi nhân *Khê Sơn Hành Lữ*, tức **`2.6` sát thương thực** khi nổ *Khê Sơn*!). Đi kèm kỹ năng *Đạn Chỉ Thần Công* và bẫy *Bát Trận Đồ / Tru Tiên Trận*.
     - **Độc công (Poison Attack - Phụ trợ / Song tu):** Hệ số **`1.25`** (mỗi `1` điểm Độc công tăng `1.25` sát thương cơ bản; kỹ năng *ương sát / Đào Hoa Trận* cộng hưởng Độc công cực mạnh).
     - **Băng công & Huyền công:** Hệ số **`1.00`** — Khi mặc các trang bị **Chân·Trùng Lâu / Chí Tôn Trùng Lâu / Đồ Chế 4 thuộc tính**, Tiêu Dao trở thành môn phái **Tứ Tu (4 hệ)** ăn trọn 100% điểm Băng/Hỏa/Huyền/Độc trên trang bị.
   - **Nội công (Internal Attack):** Giai đoạn **2007 (0.26)** chưa có ngọc Giảm kháng, sát thương Nội công kết hợp Hội công là nguồn sát thương lớn. Từ **0.85 (2009) trở đi**, Thuộc tính + Giảm kháng vươn lên làm trụ cột số 1, nhưng Nội công vẫn quyết định sát thương nền của *Đạn Chỉ Thần Công* và nảy chí mạng.
3. **Cơ chế Giảm Kháng (Reduce Resist) & Trần Âm Kháng:**
   - Khi `Giảm kháng Hỏa/Độc` của Tiêu Dao vượt quá `Kháng Hỏa/Độc` của đối thủ, kháng của mục tiêu bị kéo về `0` (hoặc **âm kháng tối đa -100 điểm** ở các bản hỗ trợ trừ kháng âm, tăng thêm tới **+100% sát thương thuộc tính** trước cả khi nhân đôi từ *Khê Sơn Hành Lữ*!).
   - Do đó, trên 6 trang bị Công, **Lục Minh Thạch / Minh Tinh Thạch (Ngọc Kép Giảm Kháng)** luôn là điều kiện tiên quyết để Tiêu Dao xuyên thủng các mục tiêu kháng Hỏa/Độc.
4. **Bộ Khống Chế & Cơ Động Đỉnh Cao:**
   - *Họa Địa Vi Lao* (vây khốn 6s–15s) + hiệu ứng `[Trang bị] Họa Địa Vi Lao` từ **Trùng Lâu Giới / Chân·Trùng Lâu Giới** (nhốt 15s không thể giải trừ thông thường).
   - *Kinh Đào Hãi Lãng* (Tản công tập thể) + hiệu ứng `[Trang bị] Đại Bi Phú` từ **Trùng Lâu Ngọc / Chân·Trùng Lâu Ngọc** (Tản công cưỡng chế khi bị đánh).
   - *Lăng Ba Vi Bộ* (Dịch chuyển tức thời,クール다운 ngắn) kết hợp **Né tránh + Hội thủ + Thân pháp** cao giúp Tiêu Dao bất tử trong tay cao thủ.

---

### 2. Công Thức Giải Mã Chỉ Số Trang Bị Từ Database Server (`EquipBase` × `ItemSegValue`)

Trong mã nguồn TLBB (cả 32-bit lẫn 64-bit), chỉ số của một trang bị bất kỳ trong `EquipBase.txt` và `GreenEquipBase.txt` được xác định bởi:
1. **Cột Trọng số (`Attr Weights`):** Các cột từ `Col 32` đến `Col 89` (hoặc `Col 31..88` ở schema 101 cột / Vô Giới) lưu trọng số chọn dòng (`w > 0` nghĩa là dòng thuộc tính đó được kích hoạt).
2. **Đoạn Giá trị (`SegID`):** Cột `ValueSeg` (cột `87` ở bản 97 cột, hoặc `91 + shift` ở bản 101–104 cột) trỏ trực tiếp tới dòng `SegID` trong bảng **`ItemSegValue.txt`**, nơi lưu giá trị số thực tế của từng thuộc tính.
3. **Phẩm chất Sao (`QualityRule` / Số Sao 1★–9★):** Cột `QualityRule` trỏ vào `ItemQuality.txt` (mã `9` = 9★ Thần cấp Trùng Lâu, `18` = 8★ Chế/Xanh, `19` = 7★–8★ Chế cao cấp, `20` = 6★–7★ Full môn phái).

---

### 3. Thứ Tự Ưu Tiên Thuộc Tính & Công Thức Khảm Ngọc 4 Lỗ Chuẩn Phái Tiêu Dao

#### 3.1. Thứ tự ưu tiên dòng chỉ số trên Trang bị (Tiêu Dao Nội Công — Chủ Hỏa / Song Tu Hỏa-Độc)

| Hạng ưu tiên | Nhóm trang bị Công (Vũ khí, Uyển, Thủ, Liên, 2 Nhẫn, 2 Phù) | Nhóm trang bị Thủ (Mão, Y phục, Kiên, Yêu đai, Hài) |
| :--- | :--- | :--- |
| **Ưu tiên 1 (Tối thượng)** | **Hỏa công + Độc công** (hoặc **4 Thuộc tính Băng/Hỏa/Huyền/Độc**) + **Giảm kháng Hỏa/Độc** | **Giới hạn HP tối đa (+HP / +%HP)** + **Thể lực** |
| **Ưu tiên 2 (Trụ cột)** | **Chính xác** (đảm bảo trúng *Khê Sơn Hành Lữ*) + **Thể lực** | **Thân pháp** + **Né tránh** + **Hội thủ (Phòng ngự hội tâm)** |
| **Ưu tiên 3 (Khuếch đại)** | **Hội công (Tấn công hội tâm)** + **Thân pháp** + **Tất cả thuộc tính** | **Kháng Băng / Hỏa / Huyền / Độc** + **Tất cả thuộc tính** |
| **Ưu tiên 4 (Phụ trợ)** | **Nội công cơ bản & Nội công cộng thêm** + **Nội lực (Trí lực)** | **Ngoại thủ / Nội thủ** + **Kháng tất cả thuộc tính** |

#### 3.2. Công thức Khảm Ngọc 4 Lỗ Chuẩn (17 Vị Trí Trang Bị)

> [!IMPORTANT]
> **Quy tắc Engine TLBB:** Lỗ thứ 1, 2, 3 là khảm thường; **Lỗ thứ 4 (Cực Hạn Đả Khổng)** không được khảm trùng loại ngọc đã khảm ở 3 lỗ đầu (trừ các trang bị cho phép khảm đồng chất hoặc dùng **Ngọc Kép `Lục Minh Tinh Thạch` / `Minh Tinh Thạch`** ở lỗ 1 và **Thuần Tịnh Bảo Thạch** ở lỗ 2).

| Nhóm vị trí (Số món) | Lỗ 1 (Chủ lực) | Lỗ 2 (Phụ trợ 1) | Lỗ 3 (Phụ trợ 2) | Lỗ 4 (Cực hạn - Khác loại) |
| :--- | :--- | :--- | :--- | :--- |
| **Nhóm Công (8 món):** Vũ khí, Hộ uyển, Hộ thủ, 2 Giới chỉ, 2 Hộ phù, Ám khí | **Hồng Tinh Thạch / Thuần Tịnh Hồng Tinh Thạch** *(Hỏa công)* | **Lục Minh Thạch / Lục Minh Tinh Thạch** *(Giảm kháng Hỏa + Kháng)* | **Tử Ngọc** *(Chính xác)* hoặc **Lục Tinh Thạch** *(Độc công - Song tu)* | **Biến Thạch** *(Hội công)* hoặc **Lục Minh Thạch** *(Giảm kháng #2)* |
| **Nhóm Thủ (6 món):** Mão, Y phục, Hộ kiên, Yêu đai, Hài, Võ hồn | **Hồng Bảo Thạch** *(Thể lực / Máu)* | **Tổ Mẫu Lục** *(Né tránh)* | **Hoàng Ngọc / Hạo Thạch / Nguyệt Quang** *(Kháng thuộc tính chính của server)* | **Lục Bảo Thạch** *(Thân pháp)* hoặc **Khổng Tước Thạch / Hắc Bảo Thạch** *(Kháng #2)* |
| **Nhóm Trung gian (3 món):** Hạng liên, Long văn, Lệnh bài / Hào hiệp ấn | **Hồng Bảo Thạch** *(Thể lực - nếu thiên Thủ)* hoặc **Hồng Tinh Thạch** *(Hỏa công)* | **Hồng Tinh Thạch** *(Hỏa công)* | **Lục Minh Thạch / Lục Minh Tinh Thạch** *(Giảm kháng Hỏa)* | **Tử Ngọc** *(Chính xác)* hoặc **Biến Thạch** *(Hội công)* |

*(Lưu ý riêng cho **STT 01 - Bản 0.26 năm 2007**: Do chưa có ngọc Giảm kháng `Lục Minh Thạch` và mỗi món chỉ đục tối đa 3 lỗ ở bản chuẩn 2007 hoặc 4 lỗ qua tool, Lỗ 1–2–3 trên đồ Công khảm: **Hồng Tinh Thạch (Hỏa công) + Miêu Nhãn Thạch (Nội công) + Tử Ngọc (Chính xác) / Biến Thạch (Hội công)**).*

---

## Phần II: Bảng Đại Đối Soát 30 Server TLBB (Phân Nhánh Kiến Trúc, Trần Ngọc & Trần Sức Mạnh Tiêu Dao)

Bảng dưới đây tổng hợp toàn bộ **30 Server** theo đúng thứ tự trong [DANH_SACH_SERVER_TLBB.md](file:///e:/games/TLBB/TLBB%20hoixua/docs/DANH_SACH_SERVER_TLBB.md), đối chiếu trực tiếp **Trần Cấp Độ, Trần Ngọc, Hệ Trang Bị Đỉnh Nhất, Tổng Thuộc Tính & Máu Ước Tính Khi Full Build Tiêu Dao**:

| STT | Tên Server / Nhánh Phiên Bản | Năm | Hệ Phái & Kiến Trúc | Trần Ngọc Hợp Lệ | Bộ Trang Bị Đỉnh Nhất Phái Tiêu Dao (Không tính đồ GM) | Tổng Hỏa+4 Hệ (Ước tính Full Build) | Tổng HP Thực Chiến (Full Build) |
| :---: | :--- | :---: | :--- | :--- | :--- | :---: | :---: |
| **01** | **TLBB026_Offline** *(0.26 Nguyên Bản)* | 2006 – 2007 | 9 Phái (32-bit `0.26`) | **Ngọc C9** *(Hỏa +180, Thể +186, Nội +3300; chưa có Giảm kháng)* | **Thần Khí 98 `Quân Thần` / Quạt Chế 100 `Ngoạ Long`** + **Đồ Chế 9x 7 dòng (8★)** *(Lân Mộ / Huyền Thế / Thần Tế)* | ~1.900 – 2.200 *(Nội công ~65.000)* | ~195.000 – 230.000 |
| **02** | **0.85 Original** *(0858130 Chuẩn)* | 2008 – 2009 | 9 Phái (32-bit `0.85`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm`** + **2x `Trùng Lâu Giới` + 2x `Chân·Trùng Lâu Ngọc`** + **Đồ Chế 11x (`Vong Xuyên`, `Khấp Lộ`)** + `Băng Phách` | ~3.200 – 3.600 | ~290.000 – 340.000 |
| **03** | **TLBB hoixua** *(0.85 Custom)* | 2008 – 2010 | 9 Phái (32-bit `0.85`) | **Ngọc C9 Thuần Tịnh** *(+Siêu Cấp Bảo Thạch nếu mở Event)* | **TK 102 `Phù Sinh` (9★, 10 dòng)** + **Full 10 Món `(Thần) Trùng Lâu` / `Trùng Lâu` (kích Set 184/185)** + `Kim Sí Linh Vũ` + `Ngự Dao Bàn` | ~4.500 – 5.200 | ~450.000 – 550.000 |
| **04** | **FullQuest** *(0.85 Chuẩn Nhiệm Vụ)* | 2008 – 2010 | 9 Phái (32-bit `0.85`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `(Thần) Đại Tống Quân Sầm`** + **Bộ Chế `Tân - ...` (Lv96–102)** + **Trang sức Chế 115 (`Cát Tường Dạ`, `Ưng Tường`)** + `Băng Phách` | ~3.400 – 3.800 | ~300.000 – 350.000 |
| **05** | **GS-020** *(0.85 Custom Long Văn)* | 2009 – 2010 | 9 Phái (32-bit `0.85`) | **Ngọc Kép C9·C9** *(Giảm kháng +180, Kháng +400)* | **TK 102 `Thái Cổ Phiêu Linh` (16 dòng)** + **Full 10 Món `Chân·Trùng Lâu` (`10556720..39`, 11–16 dòng)** + `Long Văn +9` + `Phi Vũ Tiễn` + `Võ Hồn [Hỏa]` | ~8.500 – 10.500 | ~750.000 – 950.000 |
| **06** | **69DO** *(69 Dragon Oath)* | 2010 – 2012 | 9/10 Phái (32-bit EN) | **Ngọc Kép C7·C7** *(Giảm kháng +90, Kháng +290)* | **Quạt `Immortal Fan` (Lv119)** + **Full Bộ `[69 Dragon] Legendary (Fire)` (`10223300..26`, 13 dòng)** *(hoặc Bộ `Super Ardor` nếu khóa Lv69)* | ~4.200 – 4.800 | ~480.000 – 580.000 |
| **07** | **GS-468** *(Phục Cổ Đả Kim)* | 2011 – 2013 | 10 Phái (32-bit `2.0`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK `Thái Cổ Phiêu Linh` (15 dòng)** + **Full Bộ `Chân·Trùng Lâu` & `Trùng Lâu·Phản/Triệu/Quy` (16 dòng)** + `Kiều Phong Chi Hồn` + `Hào Hiệp Ấn` | ~7.800 – 9.200 | ~700.000 – 880.000 |
| **08** | **winner** *(0.85 Winner)* | 2012 – 2014 | 9 Phái (32-bit `0.85`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK `(Thần) Thái Cổ Phiêu Linh`** + **Full 10 Món `(Thần) Trùng Lâu` (`10309101..212`, 9★, 9 dòng)** + `Băng Phách Thần Châm` | ~3.600 – 4.200 | ~380.000 – 450.000 |
| **09** | **TLBB chuyen sinh** *(Chuyển Sinh Siêu Cấp)* | 2013 – 2015 | 9/10 Phái (Chuyển Sinh) | **Ngọc Kép C9·C9 Siêu Cấp** *(Kháng +32.000, Giảm kháng +250)* | **TK `Thái Cổ Phiêu Linh` (16 dòng)** + **Full 10 Món `Siêu Cấp.Song Long Trùng Lâu +15` (16 dòng, +13.000 4 hệ/món)** + Bộ `Hoàng Kim Bá Vương` | **~750.000 – 900.000+** *(Tứ tu)* | **~6.500.000 – 9.000.000+** |
| **10** | **TGS** *(Túy Giang Sơn Lv255)* | 2015 – 2018 | 10 Phái (32-bit `3.0`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +410)* | **TK `Phù Sinh` / `Phiến Cấp 150` (19★)** + **Full Bộ `Chân-Trùng Lâu` & `Phản/Triệu/Quy`** + `Long Văn` (+50% HP) + `Anh Đào Bố Đinh` + `Hào Hiệp Ấn` | ~5.800 – 6.800 | ~680.000 – 850.000 |
| **11** | **TLBB 3.6 OFFLINE** *(Cầm Long Thức 3.61)* | 2017 – 2019 | 11 Phái (32-bit `3.61`) | **Ngọc Kép C9·9 Siêu Buff** *(Hỏa +1.020, Thể +320, Giảm kháng +162 trên 1 viên! Hồng Bảo Thạch +528 Thể; Điểm Xuyết Thời Trang +528 Thể)* | **TK 108 `Mã Nhã Chi Linh` (`10558701`, +50% HP, +1.500 4 hệ, +500 Nội/Thể) / `Lôi Minh Ly Hỏa Phiến` (+85% HP, +240 Hỏa/Độc, +55 Giảm kháng)** + **5 Món `Chân·Trùng Lâu` (`Seg=4243`, +145 4 hệ, +25 4 Giảm kháng/món)** + **Đồ Chế 120–150 (`Seg=1112..1150`)** + `Long Văn` (+185 4 hệ, +75 Tất cả) + `Ngự Dao Bàn` (+300 Cường/Nội/Thể/Trí) | **~26.000 – 31.500** *(Tứ tu + Ngọc C9·9 +1.020)* | **~2.200.000 – 2.850.000** |
| **12** | **TLBB 3D OFFLINE 3.63** *(Thiên Long 3D Gate)* | 2017 – 2019 | 11 Phái (32-bit `3.63`) | **Ngọc Kép C7-7** *(Hỏa/Độc +290 & Giảm kháng +90; Hồng Bảo Thạch C7 +534 Thể lực, Huyết Tinh C7 +29.412 HP)* | **TK 102 `Phù Sinh` (9★, 19 dòng `Seg=3009`, +50% HP, +208 4 hệ, +40 4 Giảm kháng)** + **6 Món `Chân·Trùng Lâu` (`10553103..10553111`, `Seg=4321`, +200 4 hệ, +24 Giảm kháng/món, kích 2 Set `Binh Tuyệt` +20% HP & `Phá Giáp` +89 Hội)** + **Đồ Chế 150 (`Seg=1150`)** + **`Kim Lan Lệnh` (+66% HP)** + `Pháp Bảo·Ngự Dao Bàn` + `Long Văn +9` (+120 4 hệ) | ~10.800 – 12.800 | ~1.300.000 – 1.700.000 |
| **13** | **TLBB_manda** *(Mạn Đà Sơn Trang)* | 2018 – 2020 | 10 Phái (32-bit `3.3`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK `Phù Sinh` / `Vong Xuyên Phiến`** + **Full Bộ `Chân·Trùng Lâu` (16 dòng `Seg=2152`, +260 4 hệ/món)** + `Trùng Lâu·Phản/Triệu/Quy` (`Seg=2153`) | ~11.500 – 13.500 | ~920.000 – 1.150.000 |
| **14** | **TLBB 3.78** *(Tinh Kiếm 3.78 VH)* | 2019 | 11/12 Phái (32-bit `3.78`) | **Ngọc Kép C9-9** *(Hỏa/Độc +350 & Giảm kháng +150; Thuần Tịnh +300; Thể +342)* | **TK 105 `Thái Cổ Phiêu Vũ` (`10308155`, +50% HP, +136 Hỏa/Độc, +40 Giảm kháng) / `Lôi Minh Ly Hỏa Phiến` 16 dòng / `Quạt 150 Cấu Tạo`** + **6 Món `Chân·Trùng Lâu` & `Phản/Triệu` (`Seg=100`, Set 182/161)** + **Đồ Chế 150 (`Seg=1142..1150`)** + **`Long Văn Cấp 80 Max` (+629 Hỏa, +50% HP)** + **`Ngự Dao Bàn` (+50% HP/MP, +160 4 Giảm kháng)** | ~11.200 – 13.400 | ~1.250.000 – 1.600.000 |
| **15** | **TLBB 3.6** *(TLBB 3.6 Việt Hóa)* | 2019 – 2021 | 12 Phái (32-bit `3.60`) | **Ngọc C9 Buff Cao** *(Hỏa/Độc +420, Giảm kháng +150, Thể +513, Máu +33.080; khảm cả Nang/Rương/Thú)* | **TK 102 `Phù Sinh` (9★, 16 dòng `Seg=3009`, +20% HP, +178 4 hệ, +45 Giảm kháng)** + **Full 11 Món `Chân-Trùng Lâu` (`10553103..10553119`, 9★)** *(hoặc phối Đồ Chế 150 `Seg=1142..1150`)* + `Bách Bảo Nang/Rương` 20 dòng | ~13.200 – 15.500 | ~1.350.000 – 1.750.000 |
| **16** | **TLBB_hj_ko_pack** *(XJL7796 Hoài Cựu)* | 2020 – 2022 | 9 Phái (`0.05.3334`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm`** + **5 Món `Chân-Trùng Lâu` (`Liên/Ngọc` 13 dòng +83 4 hệ)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~4.200 – 4.800 | ~380.000 – 440.000 |
| **17** | **TLBB-HJ** *(2010 Origin VH)* | 2022 – 2023 | 9 Phái (`0.05.3334`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm`** + **5 Món `Chân-Trùng Lâu` (`Liên/Ngọc` 13 dòng +83 4 hệ)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~4.200 – 4.800 | ~380.000 – 440.000 |
| **18** | **暗黑群服HoaiNiem** *(Ám Hắc Quần Phục)* | 2022 – 2023 | 12 Phái (32-bit `3.x Mod`) | **Ngọc Kép C9·C9** *(Hỏa/Độc +350, Giảm kháng +150, Thể +342; khảm thêm vào Bách Bảo Nang/Rương/Tọa Kỵ)* | **TK 102 `Phù Sinh` (16 dòng `Seg=3010`) / `Phiến 150`** + **5 Món `Chí Tôn·Trùng Lâu` (`Seg=5004`, +493 4 hệ & +160 Giảm kháng/món)** + **5 Món `Chân·Trùng Lâu` (`Seg=2152`, +600 tiềm năng & +432 Tất cả/món)** + **6 Món `Thiên Giám Linh Vũ`** + `Bách Bảo Nang/Rương` 20 dòng | **~24.500 – 28.500** *(Tứ tu)* | **~2.100.000 – 2.650.000** |
| **19** | **tlbb_lee** *(1510 Docker / 3.72)* | 2022 – 2024 | 11 Phái (ChangYou `3.72`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK Luyện Hồn `Phù Sinh Quan Khích`** + **Bộ `Chân-Trùng Lâu` (`10553103..18`, Uyển/Thủ/Ngoa +320 4 hệ)** + `Siêu Cấp Tai Thỏ` + `Tụ Nghĩa Lệnh` (+50% HP) | ~12.500 – 14.800 | ~1.250.000 – 1.550.000 |
| **20** | **tlbb129** *(129 VM / 3.72)* | 2022 – 2024 | 11 Phái (ChangYou `3.72`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK Luyện Hồn `Phù Sinh Quan Khích`** + **6 Món `Chân·Trùng Lâu` (kích 2 Set)** + **Mũ `Siêu Cấp Tai Thỏ`** + Đồ Chế 12x + `Tụ Nghĩa Lệnh` (+50% HP) | ~9.800 – 11.500 | ~1.150.000 – 1.400.000 |
| **21** | **TLBB PB14P2024** *(14 Môn Phái 2024)* | 2024 | 14 Phái (32-bit `1.4.11`) | **Ngọc Kép C9·C9** *(Hỏa/Độc +350, Giảm kháng +150, Thể +342)* | **Quạt Chế 150 (`Seg=1150`) / TK `Phù Sinh`** + **5 Món `Chân·Trùng Lâu` (`Seg=4323`/`4324`, +221 4 hệ/món)** + **Bộ `Trùng Lâu·Phản/Triệu/Quy`** + **Đồ Chế 150 (`Seg=1142..1146`)** + `Tụ Nghĩa Lệnh` + `Chu Tước Hào Hiệp Ấn` | ~11.800 – 13.600 | ~1.100.000 – 1.380.000 |
| **22** | **4.TLBB HJ64** *(Client LL 2.7 64-bit)* | 2024 (10/2024) | 9 Phái (64-bit `v2.7`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm` / `Phiến Đúc 140`** + **5 Món `Chân-Trùng Lâu` (9★)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~3.900 – 4.500 | ~370.000 – 430.000 |
| **23** | **TLBB-HJ-10P-64bit** *(Hoài Cựu 10P Mộ Dung)* | 2024 (10/2024) | 10 Phái (64-bit `v2.7`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm` / `Phiến 150`** + **5 Món `Chân - Trùng Lâu` (9★)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~4.000 – 4.600 | ~375.000 – 435.000 |
| **24** | **tlbb2.7-vh** *(Hoài Cựu 2.7 VH 24.10)* | 2024 (10/2024) | 9/10 Phái (64-bit `v2.7`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm`** + **5 Món `Chân - Trùng Lâu` (9★)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~3.900 – 4.500 | ~370.000 – 430.000 |
| **25** | **tlbb-10p-vogioi** *(Hoài Cựu 10P Vô Giới)* | 2024 (10/2024) | 10 Phái (64-bit Vô Giới) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm`** + **5 Món `Trùng Lâu` 13 Dòng (`Seg=100` Buff: HP +32.221, 4 hệ +110/món)** + `Tịch Diệt` + **Bộ 3 Món `Thiên Giám`** | ~5.400 – 6.200 | ~560.000 – 660.000 |
| **26** | **tlbb-origin-skyway** *(2010 Origin Skyway)* | 2024 – 2025 | 9 Phái (`0.05.3334`) | **Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **TK 102 `Đại Tống Quân Sầm` / `Ngoạ Long`** + **5 Món `Chân-Trùng Lâu` (`Liên/Ngọc` +83 4 hệ)** + Đồ Chế 9x–11x + **Bộ 3 Món `Thiên Giám`** | ~4.300 – 4.900 | ~385.000 – 445.000 |
| **27** | **Skynet_bot_ai** *(Hoài Cựu 64-bit Bot AI)* | 2026 (04/11) | 10 Phái (64-bit Custom) | **Ngọc C9 Buff** *(Giảm kháng +220, Thể +300, CX +45)* | **TK `Đại Tống Quân Sầm · Thần Thoại` (16 dòng)** + **Bộ `Chân·Chí Tôn Trùng Lâu` (16 dòng, +326 4 hệ/món)** + `Chí Tôn Đới` (+16% HP) + `Thiên Giám` | ~10.800 – 12.600 | ~850.000 – 1.050.000 |
| **28** | **Skynet-bot** *(ChangYou 3.72 Bot)* | 2026 (06/11) | 11 Phái (ChangYou `3.72`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK `Phù Sinh` / `Phù Sinh Quan Khích`** + **Mũ `Siêu Cấp Tai Thỏ`** + **6 Món `Chân·Trùng Lâu` + 2 Món `Chân·Giới/Ngọc` (`Seg=100` +105 4 hệ)** + `Tụ Nghĩa Lệnh` | ~10.500 – 12.200 | ~1.180.000 – 1.420.000 |
| **29** | **Level Up Games** *(Thien Long Origin)* | 2026 (Mới nhất) | 11 Phái (ChangYou `3.72`) | **Ngọc Kép C9·C9** *(Giảm kháng +150, Kháng +350)* | **TK Luyện Hồn `Phù Sinh Quan Khích`** + **Bộ `Chân-Trùng Lâu` Mở Rộng (`10553103..18`, +320 4 hệ)** + `Siêu Cấp Tai Thỏ` + `Tụ Nghĩa Lệnh` (+50% HP) | ~12.500 – 14.800 | ~1.250.000 – 1.550.000 |
| **30** | **TanThienLong** *(VNG Official 3.71.5720)* | 2025 – 2026 | **15 Môn Phái** (Dual 32/64-bit VNG) | **Ngọc Kép C9-C9 & Thuần Tịnh C9** *(50921309 Hồng Minh Tinh Thạch: Hỏa +350 & Giảm kháng +150; Thuần Tịnh +350; Thể lực C9)* | **TK 9★ `Trường Tuyệt` (`10317043`, +18% HP, +169 Băng/Độc)** + **Full 10 Món `Chân-Trùng Lâu` (`Khôi, Ngoa, Thủ, Giáp, Uyển, Kiên, Đới, Liên, Giới, Ngọc` — Kích 2 Set 182 & 207 + Impact Bạo kích 10%, Sát thương +15%, Miễn CC 5s)** + **`Bí Bảo Trấn Phái` (Slot 23)** + **`Giang Hồ Lệnh` (Slot 20)** + **`Hào Hiệp Ấn` (Slot 21)** + **`Long Văn` (Slot 19)** + **`Thú Cưỡi Đỉnh Cấp` (+51.020 HP, +510 Hỏa)** | ~14.500 – 16.800 | ~1.450.000 – 1.850.000 |

---

## Phần III: Chi Tiết 30 Build Mạnh Nhất Phái Tiêu Dao Trên 30 Server (Xếp Theo Thời Gian)

Dưới đây là hồ sơ chi tiết từng vị trí trang bị, mã Item ID, phân đoạn `SegID`, chỉ số giải mã thực tế từ `ItemSegValue.txt`, bộ Ngọc khảm tối ưu và hệ thống phụ trợ của từng server theo đúng thứ tự thời gian phát triển (`STT 01..30`).

---

### Server 01: `TLBB026_Offline` (Phiên bản 0.26 Nguyên Bản — Năm 2007)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB026_Offline\tlbb026` | Client `E:\games\TLBB\TLBB026_Offline\Client`
- **Đặc trưng hệ thống:**
  - Bản `0.26` cổ điển nhất (năm 2007): Chỉ có **12 ô trang bị cơ bản** (Vũ khí, Mão, Y phục, Hộ kiên, Hộ uyển, Hộ thủ, Yêu đai, Hài, Hạng liên, 2 Giới chỉ, 2 Hộ phù — chưa có Võ hồn, Ám khí, Long văn).
  - **Không tồn tại đồ Trùng Lâu** trong `EquipBase.txt` / `GreenEquipBase.txt`.
  - **Không tồn tại ngọc Giảm kháng (`Lục Minh Thạch`)** và chưa có ngọc Thuần Tịnh. Ngọc cao nhất là **Bảo Thạch Cấp 9**:
    - `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +186**
    - `Miêu Nhãn Thạch (Cấp 9)` (`50901002`): **Nội công +3.300**
    - `Hồng Tinh Thạch (Cấp 9)` (`50902006`) / `Lục Tinh Thạch (Cấp 9)` (`50902008`): **Hỏa công +180** / **Độc công +180**
    - `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +1.932** | `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +28** | `Tổ Mẫu Lục (Cấp 9)` (`50914001`): **Né tránh +644**
- **Phân tích Lựa chọn Trang bị:**
  - Bộ **Full 100 Tiêu Dao** (`Thu Thủy` Set 94, `Vô Nhai` Set 93, `Hành Vân` Set 95) chỉ có 4 dòng/món và kích dòng ẩn tối đa ở mốc 5–6 món (+34 Thể lực, +50% Nội công cơ bản hoặc +17 Tất cả thuộc tính).
  - Trong khi đó, **Đồ Chế 9x–100 (7 dòng, `Quality=18` — 8★, `Seg=197..211`)** cộng đồng thời **3 hệ Băng/Huyền/Độc +22~23, Ngoại/Nội công +1.053~1.106, Chính xác +1.170~1.230, Thể lực +106~114, Nội lực +106~115, Thân pháp +58~61, Tất cả thuộc tính +20~21** trên mỗi món!
  - Do đó, **Build Mạnh Nhất Phái Tiêu Dao bản 0.26** là **Full Đồ Chế 9x–100 (8★, 7 dòng)** (hoặc phối 2 nhẫn + 2 phù chế + đồ chế 9x).

#### Bảng Build 12 Vị Trí Trang Bị Mạnh Nhất — Server 01 (`TLBB026_Offline`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Ngoạ Long** / **Phụng Sô** *(hoặc TK 98 **Quân Thần** `10404070`)* | `10204010` `10204020` | 100 | 8★ | Ngoại/Nội công CB: **756** | **7 dòng (`Seg=211`):** Băng/Huyền/Độc công **+23**, Ngoại/Nội công **+1.106**, Chính xác **+1.230**, Hội công **+10**, Cường/Nội lực **+115**, Thể/Trí lực **+114**, Thân pháp **+61**, Tất cả thuộc tính **+21** *(TK `Quân Thần` `10404070`: Nội công CB **884**, 4 dòng `Seg=70`)* |
| **2. Mão** | **Lân Mộ Xích Quán** | `10210039` | 92 | 8★ | Ngoại/Nội thủ CB: **482** | **7 dòng (`Seg=197`):** Kháng Băng/Hỏa/Huyền/Độc **+22**, HP **+11.559**, MP **+5.963**, Ngoại/Nội thủ **+1.016**, Né tránh **+379**, Thể/Trí/Nội lực **+105**, Thân pháp **+57**, Tất cả thuộc tính **+20** |
| **3. Y phục** | **Lân Mộ Xích Cầu** | `10213039` | 98 | 8★ | Ngoại/Nội thủ CB: **629** | **7 dòng (`Seg=200`):** Kháng Băng/Hỏa/Huyền/Độc **+23**, HP **+12.279**, MP **+6.321**, Ngoại/Nội thủ **+1.080**, Né tránh **+402**, Thể/Trí/Nội lực **+112**, Thân pháp **+60**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **Huyền Thế Ma Kiên** | `10215020` | 98 | 8★ | Ngoại/Nội thủ CB: **453** | **7 dòng (`Seg=206`):** Kháng Băng/Hỏa/Huyền/Độc **+23**, HP **+12.279**, MP **+6.321**, Ngoại/Nội thủ **+1.080**, Né tránh **+402**, Thể/Trí/Nội lực **+112**, Thân pháp **+60**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **Huyền Thế Ma Uyển** | `10214020` | 96 | 8★ | Chính xác CB: **510** | **7 dòng (`Seg=204`):** Ngoại/Nội công **+1.064**, Chính xác **+1.182**, Hội công **+10**, Cường/Nội lực **+111**, Thể/Trí lực **+110**, Thân pháp **+59**, Tất cả thuộc tính **+20** |
| **6. Hộ thủ** | **Lân Mộ Xích Thủ** | `10212039` | 96 | 8★ | Ngoại/Nội thủ CB: **502** | **7 dòng (`Seg=199`):** HP **+12.039**, MP **+6.202**, Ngoại/Nội thủ **+1.059**, Né tránh **+394**, Thể/Trí/Nội lực **+110**, Thân pháp **+59**, Tất cả thuộc tính **+20** |
| **7. Yêu đai** | **Bộ Nguyệt** | `10221020` | 92 | 8★ | Ngoại/Nội thủ CB: **423** | **7 dòng (`Seg=209`):** Kháng Băng/Hỏa/Huyền/Độc **+22**, HP **+11.559**, MP **+5.963**, Ngoại/Nội thủ **+1.016**, Né tránh **+379**, Thể/Trí/Nội lực **+105**, Thân pháp **+57**, Tất cả thuộc tính **+20** |
| **8. Hài** | **Lân Mộ Xích Lý** | `10211039` | 94 | 8★ | Né tránh CB: **163** | **7 dòng (`Seg=198`):** HP **+11.799**, MP **+6.082**, Ngoại/Nội thủ **+1.038**, Né tránh **+386**, Thể/Trí/Nội lực **+107**, Thân pháp **+58**, Tất cả thuộc tính **+20** |
| **9. Hạng liên** | **Điệp Luyến** | `10220020` | 98 | 8★ | HP/MP CB: **4.020** | **7 dòng (`Seg=208`):** Băng/Huyền/Độc công **+23**, Kháng 4 hệ **+23**, HP **+12.279**, Ngoại/Nội công **+1.085**, Ngoại/Nội thủ **+1.080**, Chính xác **+1.206**, Hội công **+10**, Thể/Trí/Nội lực **+112**, Thân pháp **+60**, Tất cả **+21** |
| **10. Giới chỉ 1** | **Thần Tế** | `10222010` | 95 | 8★ | Nội công CB: **577** | **7 dòng (`Seg=210`):** Băng/Huyền/Độc công **+22**, Ngoại/Nội công **+1.053**, Chính xác **+1.170**, Hội công **+10**, Cường/Nội lực **+110**, Thể/Trí lực **+109**, Thân pháp **+58**, Tất cả thuộc tính **+20** |
| **11. Giới chỉ 2** | **Phật Ngữ** | `10222020` | 95 | 8★ | Nội công CB: **577** | **7 dòng (`Seg=210`):** Băng/Huyền/Độc công **+22**, Ngoại/Nội công **+1.053**, Chính xác **+1.170**, Hội công **+10**, Cường/Nội lực **+110**, Thể/Trí lực **+109**, Thân pháp **+58**, Tất cả thuộc tính **+20** |
| **12. Hộ phù 1 & 2** | **Thanh Hồ** & **Phi Tuyết** | `10223010` `10223020` | 95 | 8★ | Nội thủ CB: **509** | **7 dòng (`Seg=210`):** Băng/Huyền/Độc công **+22**, Ngoại/Nội công **+1.053**, Chính xác **+1.170**, Hội công **+10**, Cường/Nội lực **+110**, Thể/Trí lực **+109**, Thân pháp **+58**, Tất cả thuộc tính **+20** |

---

### Server 02: `0.85 Original` (Phiên bản 0.858130 Chuẩn — Năm 2009)

- **Đường dẫn:** Server `E:\games\TLBB\0.85 Original\tlbb` | Client `E:\games\TLBB\0.85 Original\Client 0858130`
- **Đặc trưng hệ thống:**
  - Bản `0.85` chuẩn kinh điển (Mừng Tuổi / Truyền Thuyết Ảnh Tử): Mở cấp độ **119**, xuất hiện **Thần Khí 102 (`Đại Tống Quân Sầm`)**, **Ám Khí (`Băng Phách Thần Châm` `10155002`)**, **Đồ Chế 11x–120 (`Vong Xuyên Phiến`, `Khấp Lộ Liên`, `Cát Tường Dạ`, `Ưng Tường Văn Chương`)** và **2 cặp Nhẫn/Phù Trùng Lâu đầu tiên (`Trùng Lâu Giới` `10422016` & `Chân·Trùng Lâu Ngọc` `10423025`)**.
  - **Hệ thống Ngọc C9:** Xuất hiện **Ngọc Thuần Tịnh C9** (`Thuần Tịnh Hồng Tinh Thạch C9` `50902010`: **Hỏa công +220**; `Thuần Tịnh Lục Tinh Thạch C9` `50902012`: **Độc công +220**) và **Ngọc Giảm Kháng C9** (`Lục Minh Thạch C9` `50921002`: **Giảm kháng Hỏa +150**; `50921004`: **Giảm kháng Độc +150**).
  - **Hệ thống Cường hóa:** Tối đa **+9** (`EquipStrengthen.txt`).

#### Bảng Build 13 Vị Trí Trang Bị Mạnh Nhất — Server 02 (`0.85 Original`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Đại Tống Quân Sầm** *(hoặc Quạt Chế 120 **Vong Xuyên Phiến** `10204027`)* | `10304101` | 102 | 7★ | Nội công CB: **1.050** \| Impact `9350` | **7 dòng (`Seg=385`):** HP tối đa **+14%**, Hỏa công **+113**, Độc công **+113**, Nội công **+1.439**, Chính xác **+1.093**, Nội lực **+95**, **Giảm kháng Hỏa/Độc +41** *(Hoặc `Vong Xuyên Phiến` `10204027` Lv120, 8★, `Seg=320`: 4 hệ **+120**, Nội công **+1.319**, Chính xác **+1.470**, Thể/Trí/Nội **+136**, Thân pháp **+73**, Tất cả **+25**)* |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** *(hoặc Lân Mộ Xích Quán `10210039`)* | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Phệ Lân Huyết Thiên Y** | `10213046` | 118 | 8★ | Ngoại/Nội thủ CB: **752** | **7 dòng (`Seg=288`):** Kháng 4 hệ **+35**, HP **+14.689**, MP **+7.515**, Ngoại/Nội thủ **+1.292**, Né tránh **+481**, Thể/Trí/Nội lực **+134**, Thân pháp **+72**, Tất cả thuộc tính **+25** |
| **4. Hộ kiên** | **Nhật Nguyệt Kiên** | `10215027` | 118 | 8★ | Ngoại/Nội thủ CB: **542** | **7 dòng (`Seg=315`):** Kháng 4 hệ **+35**, HP **+14.689**, MP **+7.515**, Ngoại/Nội thủ **+1.292**, Né tránh **+481**, Thể/Trí/Nội lực **+134**, Thân pháp **+72**, Tất cả thuộc tính **+25** |
| **5. Hộ uyển** | **Quy Túc** | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** Băng/Hỏa/Huyền/Độc công **+117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, MP **+7.396**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Trích Tiên Đái** | `10221027` | 112 | 8★ | Ngoại/Nội thủ CB: **511** | **7 dòng (`Seg=318`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **8. Hài** | **Phệ Lân Thánh Lý** | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, MP **+7.276**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Khấp Lộ Liên** | `10220027` | 118 | 8★ | HP/MP CB: **4.812** | **7 dòng (`Seg=317`):** Băng/Hỏa/Huyền/Độc công **+119**, HP **+14.689**, Ngoại/Nội công **+1.297**, Ngoại/Nội thủ **+1.292**, Chính xác **+1.446**, Hội công **+12**, Thể/Trí/Nội **+134**, Thân pháp **+72**, Tất cả **+25** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu Giới** *(x2)* *(hoặc **Cát Tường Dạ** `10222027` +116 4 hệ)* | `10422016` | 90 | **9★** | Nội công CB: **1.155** \| `[Trang bị] Họa Địa Vi Lao` (`9401`) | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** *(Nếu cần thuần sát thương thuộc tính dùng `Cát Tường Dạ` `10222027` Lv115: 4 hệ **+116**, Nội công **+1.266**, Chính xác **+1.410**, Thể/Nội **+131**)* |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** *(x2)* *(hoặc **Ưng Tường Văn Chương** `10223027`)* | `10423025` | 90 | **9★** | Nội thủ CB: **1.019** \| `[Trang bị] Đại Bi Phú` (`9402`) | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** *(Hoặc `Ưng Tường Văn Chương` `10223027` Lv115: 4 hệ **+116**, Nội công **+1.266**, Chính xác **+1.410**, Thể/Nội **+131**)* |
| **12. Ám khí** | **Băng Phách Thần Châm** | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **481** | Đục 4 lỗ, ép phẩm chất & tẩy luyện dòng: **Máu % + Hỏa công + Thể lực + Thân pháp + Chính xác** |

---

### Server 03: `TLBB hoixua` (Phiên bản 0.85 Custom — Workspace Hiện Tại)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB hoixua\tlbb` | Client `E:\games\TLBB\TLBB hoixua\Client`
- **Đặc trưng hệ thống:**
  - Nâng cấp sâu từ nền `0.85`: Tích hợp **Full Bộ 10 Món `Trùng Lâu` (`10553114..10553124`, kích Set 185 `Ma Giới Thần Binh`)** và **Full Bộ 10 Món `(Thần) Trùng Lâu` (`10309101..10309212`, kích Set 184 `Thần Giới Trang Bị`)**, cùng Thần Khí 102 9★ **`Phù Sinh` (`10553170`, 10 dòng)**, **Võ Hồn `Ngự Dao Bàn` (`10308002`)** và **Ám Khí 90 `Kim Sí Linh Vũ` (`10155004`)**.
  - Đặc biệt có thêm dòng Hộ Phù 14 dòng `10223320` (Lv95, 4 hệ +95, Nội công +1.053, Chính xác +1.170, Thể/Trí/Nội +109) cho hướng build thuần thuộc tính.
  - **Hệ thống Ngọc:** Chuẩn cày cuốc dùng **Ngọc Thuần Tịnh C9** (+220 Hỏa/Độc) & **Lục Minh Thạch C9** (+150 Giảm kháng). Ngoài ra `GemInfo.txt` có định nghĩa dải `Siêu Cấp Bảo Thạch (Cấp 99-99)` (`50920001..50920004`, +20.000 chỉ số) dành cho sự kiện đặc biệt.

#### Bảng Build 14 Vị Trí Trang Bị Mạnh Nhất — Server 03 (`TLBB hoixua`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(hoặc **(Thần) Thái Cổ Phiêu Linh** `10304420`)* | `10553170` | 102 | **9★** | Nội công CB: **1.150** \| Impact `8118` | **10 dòng (`Seg=389`):** HP tối đa **+33%**, Băng/Hỏa/Huyền/Độc công **+208**, **Giảm kháng Hỏa/Độc/Băng/Huyền +40**, Nội công **+2.596**, Chính xác **+1.944**, Hội công **+17**, Thể lực **+194**, Thân pháp **+104** |
| **2. Mão** | **(Thần) Trùng Lâu Khôi** *(hoặc `Trùng Lâu Khôi` `10553119`)* | `10309101` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7515` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **3. Y phục** | **(Thần) Trùng Lâu Giáp** *(hoặc `Trùng Lâu Giáp` `10553122`)* | `10309102` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7511` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **(Thần) Trùng Lâu Kiên** *(hoặc `Trùng Lâu Kiên` `10553117`)* | `10309115` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7514` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **(Thần) Trùng Lâu Uyển** *(hoặc `Trùng Lâu Uyển` `10553123`)* | `10309114` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7522` | **9–10 dòng (`Seg=100`, Set 184/185):** HP **+12.520**, Ngoại/Nội công **+1.106**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **6. Hộ thủ** | **(Thần) Trùng Lâu Thủ** *(hoặc `Trùng Lâu Thủ` `10553121`)* | `10309103` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7516` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **7. Yêu đai** | **(Thần) Trùng Lâu Đới** *(hoặc `Trùng Lâu Đới` `10553118`)* | `10309105` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7513` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **8. Hài** | **(Thần) Trùng Lâu Ngoa** *(hoặc `Trùng Lâu Ngoa` `10553120`)* | `10309104` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7512` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **9. Hạng liên** | **Trùng Lâu-Phản** *(hoặc `(Thần) Trùng Lâu Liên` `10309107`)* | `10553124` `10309107` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7518` / `7508` | **9 dòng (`Seg=100`, Set 184/185):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **10. Giới chỉ 1 & 2** | **(Thần) Trùng Lâu Triệu** & **(Thần) Trùng Lâu Giới** | `10309206` `10309106` | 80 | **9★** | Nội công CB: **1.155** \| Impact `7519` & `7507` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **11. Hộ phù 1 & 2** | **(Thần) Trùng Lâu Quy** & **(Thần) Trùng Lâu Ngọc** *(hoặc Phù 14 dòng `10223320`)* | `10309212` `10309112` | 80 | **9★** | Nội thủ CB: **1.019** \| Impact `7520` & `7503` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** *(Hoặc `10223320` Lv95, 14 dòng `Seg=210`: 4 hệ **+95**, Nội công **+1.053**, Chính xác **+1.170**, Thể/Trí/Nội **+109**)* |
| **12. Võ hồn** | **Ngự Dao Bàn** *(Võ hồn Nội công)* | `10308002` | 65 | 5★ | Ngoại/Nội công + Ngoại/Nội thủ CB | Mở 4 lỗ ngọc, Lĩnh ngộ kỹ năng Hỏa công + Giảm kháng Hỏa + Kháng 4 hệ + Bạo kích |
| **13. Ám khí** | **Kim Sí Linh Vũ** | `10155004` | 90 | 6★ | Ngoại/Nội công CB: **481** \| Impact `7540` | Mở 4 lỗ ngọc, Đả thông huyệt đạo & Tẩy luyện: **Giới hạn Máu % + Hỏa công + Thể lực + Thân pháp** |

---

### Server 04: `FullQuest` (Phiên bản 0.85 Chuẩn Nhiệm Vụ — Năm 2010)

- **Đường dẫn:** Server `E:\games\tlbb_draft\FullQuest\GM_Server\tlbb` | Client `E:\games\tlbb_draft\FullQuest\FullQuest`
- **Đặc trưng hệ thống:**
  - Bản `0.85` giữ nguyên nhịp độ cày nhiệm vụ và phó bản chuẩn: Không phát tràn lan bộ 10 món Trùng Lâu mà tập trung vào **Thần Khí 102 `(Thần) Đại Tống Quân Sầm` (`10304106`)**, dòng trang bị tinh luyện **`Tân - ...` (`10150101..10150122`, Lv96–102, 7 dòng `Seg=197..209`)** và **Đồ Chế 11x (`Seg=285..320`, Lv112–120)**.
  - **Hệ thống Ngọc:** Tối đa **Ngọc C9 Thuần Tịnh** (Hỏa/Độc +220) và **Lục Minh Thạch C9** (Giảm kháng +150).

#### Bảng Build 13 Vị Trí Trang Bị Mạnh Nhất — Server 04 (`FullQuest`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **(Thần) Đại Tống Quân Sầm** *(hoặc Quạt 120 **Vong Xuyên Phiến** `10204027`)* | `10304106` | 102 | 7★ | Nội công CB: **1.050** \| Impact `9359` | **7 dòng (`Seg=385`):** HP tối đa **+14%**, Hỏa công **+113**, Độc công **+113**, Nội công **+1.439**, Chính xác **+1.093**, Nội lực **+95**, **Giảm kháng Hỏa/Độc +41** *(Hoặc `Vong Xuyên Phiến` `10204027` Lv120: 4 hệ **+120**, Nội công **+1.319**, Chính xác **+1.470**, Thể/Nội **+136**)* |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** *(hoặc `Tân - Phệ Lân Khuynh Đô Quán` `10150105`)* | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Phệ Lân Huyết Thiên Y** *(hoặc `Tân - Huyết Thiên Y` `10150116`)* | `10213046` | 118 | 8★ | Ngoại/Nội thủ CB: **752** | **7 dòng (`Seg=288`):** Kháng 4 hệ **+35**, HP **+14.689**, MP **+7.515**, Ngoại/Nội thủ **+1.292**, Né tránh **+481**, Thể/Trí/Nội lực **+134**, Thân pháp **+72**, Tất cả thuộc tính **+25** |
| **4. Hộ kiên** | **Nhật Nguyệt Kiên** *(hoặc `Tân - Huyền Thế Ma Kiên` `10150121`)* | `10215027` | 118 | 8★ | Ngoại/Nội thủ CB: **542** | **7 dòng (`Seg=315`):** Kháng 4 hệ **+35**, HP **+14.689**, MP **+7.515**, Ngoại/Nội thủ **+1.292**, Né tránh **+481**, Thể/Trí/Nội lực **+134**, Thân pháp **+72**, Tất cả thuộc tính **+25** |
| **5. Hộ uyển** | **Quy Túc** *(hoặc `Tân - Huyền Thế Ma Uyển` `10150114`)* | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** Băng/Hỏa/Huyền/Độc công **+117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** *(hoặc `Tân - Phệ Lân Huyền Thủ` `10150115`)* | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, MP **+7.396**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Trích Tiên Đái** *(hoặc `Tân - Bộ Nguyệt` `10150104`)* | `10221027` | 112 | 8★ | Ngoại/Nội thủ CB: **511** | **7 dòng (`Seg=318`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **8. Hài** | **Phệ Lân Thánh Lý** *(hoặc `Tân - Phệ Lân Thánh Lữ` `10150106`)* | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, MP **+7.276**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Khấp Lộ Liên** *(hoặc `Tân - Điệp Luyến` `10150122`)* | `10220027` | 118 | 8★ | HP/MP CB: **4.812** | **7 dòng (`Seg=317`):** Băng/Hỏa/Huyền/Độc công **+119**, HP **+14.689**, Ngoại/Nội công **+1.297**, Chính xác **+1.446**, Hội công **+12**, Thể/Trí/Nội **+134**, Thân pháp **+72**, Tất cả **+25** |
| **10. Giới chỉ 1 & 2** | **Cát Tường Dạ** *(x2)* *(hoặc **Trùng Lâu Giới** `10422016`)* | `10222027` `10422016` | 115 / 90 | 8★ / 9★ | Nội công CB: **689** / **1.155** | **7 dòng (`Seg=319`):** Băng/Hỏa/Huyền/Độc công **+116**, Ngoại/Nội công **+1.266**, Chính xác **+1.410**, Hội công **+12**, Cường/Nội/Thể/Trí **+131**, Thân pháp **+70**, Tất cả **+24** *(Phối 1 `Trùng Lâu Giới` `10422016` để lấy Phong huyệt 15s)* |
| **11. Hộ phù 1 & 2** | **Ưng Tường Văn Chương** *(x2)* *(hoặc **Chân·Trùng Lâu Ngọc** `10423025`)* | `10223027` `10423025` | 115 / 90 | 8★ / 9★ | Nội thủ CB: **608** / **1.019** | **7 dòng (`Seg=319`):** Băng/Hỏa/Huyền/Độc công **+116**, Ngoại/Nội công **+1.266**, Chính xác **+1.410**, Hội công **+12**, Cường/Nội/Thể/Trí **+131**, Thân pháp **+70**, Tất cả **+24** *(Phối 1 `Chân·Trùng Lâu Ngọc` `10423025` lấy Tản công)* |
| **12. Ám khí** | **Băng Phách Thần Châm** | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **481** | Đục 4 lỗ ngọc C9, tẩy luyện phẩm chất & hiệu ứng chuẩn 0.85 |

---

### Server 05: `GS-020` (Phiên bản 0.85 Custom Trùng Lâu 16 Dòng & Long Văn — Năm 2011)

- **Đường dẫn:** Server `E:\games\tlbb_draft\GS-020\tlbb` | Client `E:\games\tlbb_draft\GS-020\Client`
- **Đặc trưng hệ thống:**
  - Một trong những bản Custom `0.85` hoàn thiện và đồ sộ nhất: Mở rộng toàn bộ trang bị End-Game lên **11–16 dòng thuộc tính**, tích hợp **Long Văn +9 (`10157010`, Slot 10)**, **Ám Khí 120 `Phi Vũ Tiễn` (`10433325`, Slot 17)** và **Võ Hồn chuyên biệt hệ nguyên tố `Võ Hồn [Hỏa]` (`10433321`, Slot 18, Impact `25511`) / `Võ Hồn [Độc]` (`10433320`, Impact `25513`)**.
  - **Hệ thống Ngọc C9 Kép:** Có **`Lục Minh Tinh Thạch (Cấp 9·Cấp 9)` (`50921409`)**: **Kháng Huyền +400 & Giảm kháng Hỏa +150**; **`Lục Minh Thạch (Cấp 9)` (`50921004`)**: **Giảm kháng Hỏa +180**.

#### Bảng Build 15 Vị Trí Trang Bị Mạnh Nhất — Server 05 (`GS-020`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Thái Cổ Phiêu Linh** *(hoặc **Lôi Minh Ly Hỏa Phiến** `10553115` Lv106 / **Phù Sinh** `10304435`)* | `10553219` | 102 | **9★** | Nội công CB: **1.288** \| Impact `8125` | **16 dòng (`Seg=1118`):** Tích hợp **Băng/Hỏa/Huyền/Độc công + Giảm kháng 4 hệ + HP tối đa + Ngoại/Nội công + Chính xác + Hội công + Cường/Nội/Thể/Trí/Thân + Tất cả thuộc tính** *(Hoặc `Lôi Minh Ly Hỏa Phiến` `10553115` Lv106: Nội công CB **1.416**, 11 dòng)* |
| **2. Mão** | **Chân·Trùng Lâu Khôi** *(hoặc `Chí Tôn·Trùng Lâu Khôi` `10556006`)* | `10556735` | 105 | **9★** | Ngoại/Nội thủ CB: **1.163** \| Impact `7543` | **11–16 dòng (`Seg=5046`/`1118`):** HP tối đa + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Thể/Trí/Nội lực + Thân pháp + Hội thủ + Tất cả thuộc tính *(Bản `Chí Tôn` `10556006` có thêm **Giảm kháng 4 hệ** ngay trên Mũ!)* |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** *(hoặc `10553111` 16 dòng)* | `10556734` | 105 | **9★** | Ngoại/Nội thủ CB: **1.495** \| Impact `7541` | **11–16 dòng (`Seg=5046`/`1118`):** HP tối đa + Kháng Băng/Hỏa/Huyền/Độc + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10556727` | 105 | **9★** | Ngoại/Nội thủ CB: **1.079** \| Impact `7514` | **16 dòng (`Seg=1118`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **5. Hộ uyển** | **Chân·Trùng Lâu Uyển** | `10556736` | 105 | **9★** | Chính xác CB: **1.196** \| Impact `7546` | **12–16 dòng (`Seg=5046`/`10553121`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội công + Chính xác + Hội công + Thể/Trí/Thân pháp + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **6. Hộ thủ** | **Chân·Trùng Lâu Thủ** | `10556737` | 105 | **9★** | Ngoại/Nội thủ CB: **1.196** \| Impact `7544` | **12–16 dòng (`Seg=5046`/`10553117`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính |
| **7. Yêu đai** | **Chân·Trùng Lâu Đới** | `10556738` | 105 | **9★** | Ngoại/Nội thủ CB: **1.029** \| Impact `7545` | **12–16 dòng (`Seg=5046`/`10553119`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính |
| **8. Hài** | **Chân·Trùng Lâu Ngoa** | `10556739` | 105 | **9★** | Né tránh CB: **388** \| Impact `7542` | **12–16 dòng (`Seg=5046`/`10553113`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính |
| **9. Hạng liên** | **Chân·Trùng Lâu Liên** | `10556720` | 105 | **9★** | HP/MP CB: **8.040** \| Impact `7508` | **16 dòng (`Seg=1118`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội công + Chính xác + Né tránh + Hội công + Thể/Trí/Thân pháp + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **10. Giới chỉ 1 & 2** | **Chân·Trùng Lâu Giới** *(x2)* *(hoặc phối `10553104`)* | `10556721` `10553104` | 105 | **9★** | Nội công CB: **1.155** \| Impact `7507` | **16 dòng (`Seg=1118`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội công + Chính xác + Hội công + Thể/Trí/Thân pháp + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** *(x2)* *(hoặc phối `10553105`)* | `10556722` `10553105` | 105 | **9★** | Nội thủ CB: **1.019** \| Impact `7503` | **16 dòng (`Seg=1118`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội công + Chính xác + Hội công + Thể/Trí/Thân pháp + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **12. Long văn (Slot 10)** | **Long Văn +9** | `10157010` | 50 | 8★ | Ngoại/Nội công + Thủ CB \| Impact `25509` | **16 dòng (`Seg=1118`):** Cộng toàn diện 4 hệ công + Giảm kháng 4 hệ + Máu + 5 dòng tiềm năng + Tất cả thuộc tính |
| **13. Ám khí (Slot 17)** | **Phi Vũ Tiễn** *(hoặc **Đoạn Hồn Tiêu** `10155006`)* | `10433325` | 120 | **9★** | Ngoại/Nội công CB: **2.922** \| Impact `25517` | **16 dòng (`Seg=1118`):** Đỉnh cao Ám khí 120, cộng full 16 dòng công/thủ/thuộc tính/giảm kháng |
| **14. Võ hồn (Slot 18)** | **Võ Hồn [Hỏa]** *(hoặc **Võ Hồn [Độc]** `10433320`)* | `10433321` | 55 | **9★** | Ngoại/Nội công CB: **980** \| Impact `25511` | **16 dòng (`Seg=1118`):** Chuyên biệt hóa cho Tiêu Dao Hỏa công (kích hoạt hiệu ứng nguyên tố Hỏa `Impact 25511` + 16 dòng thuộc tính) |

---

### Server 06: `69DO` (69 Dragon Oath — Phiên Bản Quốc Tế PvP 69 & End-Game 2012)

- **Đường dẫn:** Server `E:\games\tlbb_draft\69DO\tlbb` | Client `E:\games\tlbb_draft\69DO\69DO_Full`
- **Đặc trưng hệ thống:**
  - Server tiếng Anh nổi tiếng *69 Dragon Oath*: Trần ngọc giới hạn ở **Gem Lv7 (`Bright Epidote Lv7-Lv7` `50721414`: Giảm kháng Hỏa +90 & Kháng Huyền +290; `Pure Ruby Lv7`: Hỏa công +160)**.
  - Có **2 mốc Build hoàn chỉnh** trong Database:
    1. **Mốc End-Game (Lv105–119):** Vũ khí **`Immortal Fan` (`10304435`, Lv119, Nội công CB 2.250)** đi cùng **Full Bộ `[69 Dragon] Legendary` (`10223300..10223326`, Lv105, 9★, `Seg=5000`, 12–13 dòng)**, trong đó Hạng liên, Nhẫn, Hộ phù được **chia riêng theo từng hệ nguyên tố (Ice / Fire / Light / Poison)** để tối đa hóa **+270 Hỏa công & +120 Giảm kháng Hỏa** trên mỗi món!
    2. **Mốc Khóa Cấp 69 (Giải đấu PvP 69):** Sử dụng trọn bộ **`Super - ... of Ardor` (`10420092..10420110`, Lv50, 9★, `Seg=100`, 9 dòng siêu cường: HP +17.520, Chính xác +7.230, Né tránh +4.410, Hội công/thủ +30, Thể/Trí +170, Thân pháp +120, Tất cả +40)** kết hợp Thần Khí 62 `Divine Flame Fan` (`10304422`, Nội công CB 1.244).

#### Bảng Build 15 Vị Trí Trang Bị Mạnh Nhất — Server 06 (`69DO`)

| Vị trí | Tên trang bị tối ưu (End-Game & Lv69) | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Immortal Fan** *(Khóa 69: **Divine Flame Fan** `10304422` Lv62)* | `10304435` | 119 | **9★** | Nội công CB: **2.250** \| Impact `8125` | **9 dòng (`Seg=385`):** HP tối đa + Hỏa/Độc công + Giảm kháng + Nội công + Chính xác + Hội công + Thể/Thân |
| **2. Mão** | **[69 Dragon] Legendary Crown** *(Lv69: `Super - Hat of Ardor` `10420105`)* | `10223300` | 105 | **9★** | Ngoại/Nội thủ CB: **685** | **12 dòng (`Seg=5000`):** Kháng Băng/Hỏa/Huyền/Độc **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **3. Y phục** | **[69 Dragon] Legendary Armor** *(Lv69: `Super - Armor of Ardor` `10420100`)* | `10223302` | 105 | **9★** | Ngoại/Nội thủ CB: **884** | **12 dòng (`Seg=5000`):** HP **+14.556**, **HP tối đa +20%**, Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Tất cả thuộc tính **+69** |
| **4. Hộ kiên** | **[69 Dragon] Legendary Shoulder** *(Lv69: `Super - Pauldron of Ardor` `10420101`)* | `10223305` | 105 | **9★** | Ngoại/Nội thủ CB: **638** | **12 dòng (`Seg=5000`):** Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **5. Hộ uyển** | **[69 Dragon] Legendary Bracer** *(Lv69: `Super - Bracer of Ardor` `10420104`)* | `10223304` | 105 | **9★** | Chính xác CB: **713** | **12 dòng (`Seg=5000`):** Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **6. Hộ thủ** | **[69 Dragon] Legendary Gloves** *(Lv69: `Super - Gloves of Ardor` `10420106`)* | `10223301` | 105 | **9★** | Ngoại/Nội thủ CB: **703** | **12 dòng (`Seg=5000`):** Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **7. Yêu đai** | **[69 Dragon] Legendary Belt** *(Lv69: `Super - Sash of Ardor` `10420102`)* | `10223306` | 105 | **9★** | Ngoại/Nội thủ CB: **598** | **12 dòng (`Seg=5000`):** Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **8. Hài** | **[69 Dragon] Legendary Boots** *(Lv69: `Super - Shoes of Ardor` `10420103`)* | `10223303` | 105 | **9★** | Né tránh CB: **229** | **12 dòng (`Seg=5000`):** Kháng 4 hệ **+100**, Ngoại/Nội thủ **+5.327**, Né tránh **+2.735**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Hội thủ **+32** |
| **9. Hạng liên** | **[69 Dragon] Legendary Necklace (Fire)** *(hoặc bản Poison `10223310`)* | `10223308` | 105 | **9★** | HP/MP CB: **5.524** | **13 dòng (`Seg=5000`):** HP **+14.556**, **HP tối đa +20%**, **Hỏa công +270**, Ngoại/Nội công **+5.327**, Chính xác **+5.327**, Hội công **+62**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Tất cả **+69**, **Giảm kháng Hỏa +120** |
| **10. Giới chỉ 1 & 2** | **[69 Dragon] Legendary Ring (Fire)** *(x2)* *(hoặc phối 1 Ring Poison `10223318`)* | `10223316` | 105 | **9★** | Nội công CB: **806** | **13 dòng (`Seg=5000`):** HP **+14.556**, **HP tối đa +20%**, **Hỏa công +270**, Ngoại/Nội công **+5.327**, Chính xác **+5.327**, Hội công **+62**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Tất cả **+69**, **Giảm kháng Hỏa +120** |
| **11. Hộ phù 1 & 2** | **[69 Dragon] Legendary Charm (Fire)** *(x2)* *(hoặc phối 1 Charm Poison `10223326`)* | `10223324` | 105 | **9★** | Nội thủ CB: **712** | **13 dòng (`Seg=5000`):** HP **+14.556**, **HP tối đa +20%**, **Hỏa công +270**, Ngoại/Nội công **+5.327**, Chính xác **+5.327**, Hội công **+62**, Cường/Nội/Thể lực **+270**, Thân pháp **+165**, Tất cả **+69**, **Giảm kháng Hỏa +120** |
| **12. Võ hồn (Slot 10)** | **Soul Of Magic** *(Ngự Dao Bàn)* | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc Lv7, mở rộng thuộc tính Hỏa/Độc + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Ice Needle** *(Băng Phách Thần Châm)* | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **288** | Đục 4 lỗ ngọc Lv7, tẩy luyện Máu % + Hỏa công + Thể lực |
| **14. Long văn (Slot 18)** | **Dragon Tatto** *(Long Văn)* | `10155010` | 80 | 5★ | Ngoại/Nội công CB: **1.540** \| Impact `12304` | Tăng % HP tối đa + Xuyên kháng Hỏa + Thuộc tính Hỏa công |

---

### Server 07: `GS-468` (Phiên bản Phục Cổ Đả Kim 10 Phái — Năm 2013)

- **Đường dẫn:** Server `E:\games\tlbb_draft\GS-468\tlbb` | Client `E:\games\tlbb_draft\GS-468\客户端\天龙八部468复古打金版`
- **Đặc trưng hệ thống:**
  - Bản *Phục Cổ Đả Kim* (Cày vàng cổ điển): Sở hữu hệ thống trang bị cực kỳ phong phú gồm **Thần Khí 15 dòng `Thái Cổ Phiêu Linh` (`10354419`)**, **Đồ Chế 130 (10★) `Nhiễm Vận` (`10204028`)**, **Full Bộ `Chân·Trùng Lâu` (`10553103..10553121`, 16 dòng)**, **`Chí Tôn·Trùng Lâu Khôi/Kiên` (`10556006`, `10565172`, 16 dòng)**, bộ 3 trang sức **`Trùng Lâu·Phản / Triệu / Quy` (`10159133..10159135`, 16 dòng)**, đặc biệt ở Slot 18 có **`Kiều Phong Chi Hồn` (`10556005`, Lv105, 9★, 14 dòng)** và Slot 21 có **`Chu Tước Hào Hiệp Ấn·Bát Cực` (`10159024`)**.
  - **Hệ thống Ngọc:** **Ngọc Kép C9·C9** (`Lục Minh Tinh Thạch C9·C9` `50921409`: **Kháng Huyền +350 & Giảm kháng Hỏa +150**).

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 07 (`GS-468`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Thái Cổ Phiêu Linh** *(hoặc Quạt 130 **Nhiễm Vận** `10204028` 10★)* | `10354419` | 102 | **9★** | Nội công CB: **1.246** \| Impact `8125` | **15 dòng (`Seg=385`):** HP + **Băng/Hỏa/Huyền/Độc công** + **Giảm kháng 4 hệ** + Ngoại/Nội công + Chính xác + Hội công + Cường/Nội/Thể/Trí/Thân + Tất cả thuộc tính |
| **2. Mão** | **Chân·Trùng Lâu Khôi** *(hoặc **Chí Tôn·Trùng Lâu Khôi** `10556006`)* | `10553115` `10556006` | 80 / 90 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7543` | **16 dòng (`Seg=100`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Hội công/thủ + Cường/Nội/Thể/Trí/Thân + Tất cả thuộc tính *(Bản `Chí Tôn` `10556006` có thêm **Giảm kháng 4 hệ**)* |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7511` | **16 dòng (`Seg=100`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Hội công/thủ + Cường/Nội/Thể/Trí/Thân + Tất cả thuộc tính |
| **4. Hộ kiên** | **Chí Tôn·Trùng Lâu Kiên** *(hoặc `Chân·Trùng Lâu Kiên` `10553109`)* | `10565172` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7514` | **16 dòng (`Seg=100`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Thể/Trí/Thân pháp + Hội thủ + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **5. Hộ uyển** | **Chân·Trùng Lâu Uyển** | `10553121` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7546` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Chính xác + Hội công + Cường/Nội/Thể/Trí/Thân + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **6. Hộ thủ** | **Chân·Trùng Lâu Thủ** | `10553117` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7544` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội thủ + Né tránh + Hội công/thủ + Thể/Trí/Thân + Tất cả thuộc tính |
| **7. Yêu đai** | **Chân·Trùng Lâu Đới** | `10553119` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7545` | **16 dòng (`Seg=100`):** HP + Kháng 4 hệ + Ngoại/Nội thủ + Né tránh + Hội công/thủ + Thể/Trí/Thân + Tất cả thuộc tính |
| **8. Hài** | **Chân·Trùng Lâu Ngoa** | `10553113` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7542` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Ngoại/Nội thủ + Né tránh + Hội công/thủ + Thể/Trí/Thân + Tất cả thuộc tính |
| **9. Hạng liên** | **Trùng Lâu·Phản** *(hoặc `Chân·Trùng Lâu Liên` `10553103`)* | `10159133` `10553103` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7518` / `7508` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Chính xác + Né tránh + Hội công + Thể/Trí/Thân + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu·Triệu** & **Chân·Trùng Lâu Giới** | `10159134` `10553104` | 80 | **9★** | Nội công CB: **1.155** \| Impact `7519` & `7507` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Chính xác + Né tránh + Hội công + Thể/Trí/Thân + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **11. Hộ phù 1 & 2** | **Trùng Lâu·Quy** & **Chân·Trùng Lâu Ngọc** | `10159135` `10553105` | 80 | **9★** | Nội thủ CB: **1.019** \| Impact `7520` & `7503` | **16 dòng (`Seg=100`):** HP + **Băng/Hỏa/Huyền/Độc công** + Chính xác + Né tránh + Hội công + Thể/Trí/Thân + Tất cả thuộc tính + **Giảm kháng 4 hệ** |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9·C9, lĩnh ngộ kỹ năng Hỏa/Độc + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Kim Sí Linh Vũ** | `10155008` | 90 | 8★ | Ngoại/Nội công CB: **2.180** \| Impact `7530` | Đục 4 lỗ ngọc C9·C9, tẩy luyện phẩm chất & đả thông huyệt đạo |
| **14. Hồn hiệp (Slot 18)** | **Kiều Phong Chi Hồn** *(hoặc Long Văn `10157001`)* | `10556005` | 105 | **9★** | Ngoại/Nội công CB: **288** | **14 dòng (`Seg=100`):** Cộng thêm **4 hệ công + Giảm kháng 4 hệ + Máu + Tiềm năng** |
| **15. Hào hiệp ấn (Slot 21)** | **Chu Tước Hào Hiệp Ấn·Bát Cực** | `10159024` | 85 | 7★ | Ngoại/Nội công CB: **1.410** | Tăng mạnh **HP + Thuộc tính +70 +7 dòng giảm khống chế** |

---

### Server 08: `winner` (Phiên bản 0.85 Winner — Năm 2013)

- **Đường dẫn:** Server `E:\games\tlbb_draft\winner\tlbb` | Client `E:\games\tlbb_draft\winner\Client`
- **Đặc trưng hệ thống:**
  - Nền tảng `0.85` tích hợp trọn bộ **10 món `(Thần) Trùng Lâu` (`10309101..10309212`, Lv80, 9★, 9 dòng, `Seg=100`, kích Set 184 `Thần Giới Trang Bị`)** và **Thần Khí 102 `(Thần) Thái Cổ Phiêu Linh` (`10304425`, Lv102, Nội công CB 1.150)**.
  - **Hệ thống Ngọc:** **Ngọc C9 Thuần Tịnh** (+220 Hỏa/Độc công) và **Lục Minh Thạch C9** (+150 Giảm kháng Hỏa/Độc).

#### Bảng Build 13 Vị Trí Trang Bị Mạnh Nhất — Server 08 (`winner`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **(Thần) Thái Cổ Phiêu Linh** *(hoặc Quạt 120 **Vong Xuyên Phiến** `10204027`)* | `10304425` | 102 | **9★** | Nội công CB: **1.150** \| Impact `8125` | **9 dòng (`Seg=385`):** HP tối đa **+14%**, Hỏa công **+113**, Độc công **+113**, **Giảm kháng Hỏa/Độc +41**, Nội công **+1.439**, Chính xác **+1.093**, Thể/Nội/Thân pháp *(Hoặc phối `Vong Xuyên Phiến` `10204027` Lv120: 4 hệ **+120**, Nội công **+1.319**, Chính xác **+1.470**)* |
| **2. Mão** | **(Thần) Trùng Lâu Khôi** | `10309101` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7515` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **3. Y phục** | **(Thần) Trùng Lâu Giáp** | `10309102` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7511` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **(Thần) Trùng Lâu Kiên** | `10309115` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7514` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **(Thần) Trùng Lâu Uyển** *(hoặc **Quy Túc** `10214027` +117 4 hệ)* | `10309114` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7522` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả **+21** *(Nếu cần thuần thuộc tính có thể dùng `Quy Túc` `10214027` Lv116: 4 hệ **+117**)* |
| **6. Hộ thủ** | **(Thần) Trùng Lâu Thủ** | `10309103` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7516` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **7. Yêu đai** | **(Thần) Trùng Lâu Đới** | `10309105` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7513` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **8. Hài** | **(Thần) Trùng Lâu Ngoa** | `10309104` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7512` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **9. Hạng liên** | **(Thần) Trùng Lâu Liên** *(hoặc **Khấp Lộ Liên** `10220027` +119 4 hệ)* | `10309107` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7508` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **10. Giới chỉ 1 & 2** | **(Thần) Trùng Lâu Triệu** & **(Thần) Trùng Lâu Giới** | `10309206` `10309106` | 80 | **9★** | Nội công CB: **1.155** \| Impact `7519` & `7507` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **11. Hộ phù 1 & 2** | **(Thần) Trùng Lâu Quy** & **(Thần) Trùng Lâu Ngọc** | `10309212` `10309112` | 80 | **9★** | Nội thủ CB: **1.019** \| Impact `7520` & `7503` | **9 dòng (`Seg=100`, Set 184):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **12. Ám khí** | **Băng Phách Thần Châm** | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **481** | Đục 4 lỗ ngọc C9, tẩy luyện phẩm chất & hiệu ứng chuẩn |

---

### Server 09: `TLBB chuyen sinh` (Phiên bản Chuyển Sinh Siêu Cấp — Năm 2013)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB chuyen sinh\tlbb` | Client `E:\games\TLBB\TLBB chuyen sinh\Client`
- **Đặc trưng hệ thống:**
  - **Đỉnh cao nhất về quy mô chỉ số (Hyper-Scaling Reborn Meta)** trong toàn bộ 23 Server: Thiết kế riêng cho gameplay **Chuyển Sinh (Rebirth)** nhiều lần nên bảng `ItemSegValue.txt` (`Seg=6000..6010`) và `GemInfo.txt` được nhân hệ số khổng lồ:
    - **Ngọc Kép C9-C9 (`Lục Minh Tinh Thạch C9-C9` `50921409`):** **Kháng Huyền +32.000 & Giảm kháng Hỏa +250**!
    - **Bộ 10 Món `Siêu Cấp.Song Long Trùng Lâu +15` (`10553530..10553539`, Lv120, 9★, 16 dòng, `Seg=6010`, Impact `32623..32632`):** Mỗi món cộng trực tiếp **HP +170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, **Ngoại/Nội công +12.000**, **Ngoại/Nội thủ +3.633**, **Cường lực/Nội lực/Thể lực +13.000**!
    - **Cặp Nhẫn/Phù `Trùng Lâu Giới / Ngọc` (`10570101..10570104`, `Seg=6000`):** Cộng **HP tối đa +100%**, **Nội công +50%**, **HP +18.500**, **Ngoại/Nội công +18.500**, **Chính xác +10.000**, **5 dòng tiềm năng +4.000**!
    - **Bộ 3 Pháp Bảo / Ám Khí / Long Văn `Hoàng Kim Bá Vương` (`10553236..10553238`, Lv120, 9★, 16 dòng, `Seg=5046`):** Mỗi món cộng **Nội công CB 15.000**, **HP +30.000**, **4 hệ thuộc tính +10.000**, **Nội công +24.000**, **Cường/Nội/Thể lực +10.000**!

#### Bảng Build 15 Vị Trí Trang Bị Mạnh Nhất — Server 09 (`TLBB chuyen sinh`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Thái Cổ Phiêu Linh** *(hoặc **Siêu Cấp.Lăng Ba Linh Lung Phiến** `10553359`)* | `10553576` `10553359` | 120 | **9★** | Nội công CB: **10.000** \| Impact `8125` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường lực **+13.000**, Nội lực **+13.000**, Thể lực **+13.000** |
| **2. Mão** | **Siêu Cấp.Song Long Trùng Lâu Khôi+15** | `10553530` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32623` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **3. Y phục** | **Siêu Cấp.Song Long Trùng Lâu Giáp+15** | `10553533` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32626` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **4. Hộ kiên** | **Siêu Cấp.Song Long Trùng Lâu Kiên+15** | `10553534` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32627` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **5. Hộ uyển** | **Siêu Cấp.Song Long Trùng Lâu Uyển+15** | `10553539` | 120 | **9★** | Ngoại/Nội công CB: **15.000** \| Impact `32632` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **6. Hộ thủ** | **Siêu Cấp.Song Long Trùng Lâu Thủ+15** | `10553532` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32625` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **7. Yêu đai** | **Siêu Cấp.Song Long Trùng Lâu Đới+15** | `10553536` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32629` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **8. Hài** | **Siêu Cấp.Song Long Trùng Lâu Ngoa+15** | `10553531` | 120 | **9★** | Ngoại/Nội thủ CB: **10.000** \| Impact `32624` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **9. Hạng liên** | **Siêu Cấp.Song Long Trùng Lâu Liên+15** | `10553535` | 120 | **9★** | Ngoại/Nội công CB: **15.000** \| Impact `32628` | **16 dòng (`Seg=6010`):** HP **+170.000**, **Băng/Hỏa/Huyền/Độc công +13.000**, Ngoại/Nội công **+12.000**, Ngoại/Nội thủ **+3.633**, Cường/Nội/Thể lực **+13.000** |
| **10. Giới chỉ 1 & 2** | **Siêu Cấp.Song Long Trùng Lâu Triệu+15** & **Trùng Lâu Giới** | `10553537` `10570101` | 120 | **9★** | Nội công CB: **15.000** & **28.000** | **Triệu (`Seg=6010`):** HP **+170.000**, 4 hệ **+13.000**, Thể/Nội **+13.000**. **Giới (`10570101`, `Seg=6000`):** **HP tối đa +100%**, **Nội công +50%**, HP **+18.500**, Nội công **+18.500**, Chính xác **+10.000**, 5 tiềm năng **+4.000** |
| **11. Hộ phù 1 & 2** | **Siêu Cấp.Song Long Trùng Lâu Quy+15** & **Trùng Lâu Ngọc** | `10553538` `10570104` | 120 | **9★** | Nội công CB: **15.000** & **28.000** | **Quy (`Seg=6010`):** HP **+170.000**, 4 hệ **+13.000**, Thể/Nội **+13.000**. **Ngọc (`10570104`, `Seg=6000`):** **HP tối đa +100%**, **Nội công +50%**, HP **+18.500**, Nội công **+18.500**, Chính xác **+10.000**, 5 tiềm năng **+4.000** |
| **12. Pháp bảo (Slot 10)** | **Hoàng Kim Bá Vương - Tuyệt** *(hoặc `Cuồng Phong Sát - Tuyệt` `10553213`)* | `10553237` | 120 | **9★** | Nội công CB: **15.000** \| Impact `25511` | **16 dòng (`Seg=5046`):** HP **+30.000**, **4 hệ công +10.000**, Ngoại/Nội công **+24.000**, Ngoại/Nội thủ **+2.593**, Cường/Nội/Thể lực **+10.000** *(Bản `Cuồng Phong Sát` `10553213` `Seg=6008`: HP **+140.000**, 4 hệ **+6.500**)* |
| **13. Ám khí (Slot 17)** | **Hoàng Kim Bá Vương - Tiễn** *(hoặc `Cuồng Phong Sát - Tiễn` `10553214`)* | `10553238` | 120 | **9★** | Nội công CB: **15.000** \| Impact `25517` | **16 dòng (`Seg=5046`):** HP **+30.000**, **4 hệ công +10.000**, Ngoại/Nội công **+24.000**, Ngoại/Nội thủ **+2.593**, Cường/Nội/Thể lực **+10.000** |
| **14. Long văn (Slot 18)** | **Hoàng Kim Bá Vương - Minh** *(hoặc `Long Văn +9` `10157009`)* | `10553236` | 120 | **9★** | Nội công CB: **15.000** \| Impact `25509` | **16 dòng (`Seg=5046`):** HP **+30.000**, **4 hệ công +10.000**, Ngoại/Nội công **+24.000**, Ngoại/Nội thủ **+2.593**, Cường/Nội/Thể lực **+10.000** |

---

### Server 10: `TGS` (Túy Giang Sơn MaxLv 255 — Năm 2014)

- **Đường dẫn:** Server `E:\games\TLBB\TGS\tlbb` | Client `E:\games\TLBB\TGS\CLIENT`
- **Đặc trưng hệ thống:**
  - Bản *Túy Giang Sơn* (nền 3.0 mở giới hạn cấp độ 255): Sở hữu **Quạt Chế Cấp 150 (`Phiến Cấp 150` `10204030`, 19★, `Seg=400`, Nội công CB 1.310)** cùng **Thần Khí 102 `Phù Sinh` (`10304435`)**, **Full Bộ `Chân-Trùng Lâu` (`10553104..10553118`, 9★, 9 dòng)** kết hợp **`Trùng Lâu-Phản / Triệu / Quy` (`10159133..10159135`)**.
  - Đặc biệt mạnh ở các ô trang bị mở rộng (Slot 10, 17, 18, 19, 21):
    - **Long Văn (`10157001`, Slot 18, `Seg=4150`):** Cộng **HP +22.630, HP tối đa +50%, Băng/Hỏa/Huyền/Độc công +292, Kháng 4 hệ +36**!
    - **Pháp Bảo / Slot 19 (`Anh Đào Bố Đinh` `10102221`, 8★, `Seg=3009`):** Cộng **HP +21.196, HP tối đa +20%, Băng/Hỏa/Huyền/Độc công +178, Kháng 4 hệ +37, Tất cả thuộc tính +18**!
    - **Hào Hiệp Ấn (`Chu Tước Hào Hiệp Ấn-Bát Cực` `10159024`, Slot 21):** Nội công CB **1.410**, cộng HP + 4 hệ thuộc tính + giảm khống chế.
  - **Hệ thống Ngọc:** **`Lục Minh Tinh Thạch-Thông Bảo (Cấp 9-Cấp 9)` (`50921418`)**: **Kháng Huyền +410 & Giảm kháng Hỏa +150**.

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 10 (`TGS`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(hoặc **Phiến Cấp 150** `10204030` 19★)* | `10304435` `10204030` | 102 / 150 | 8★ / 19★ | Nội công CB: **1.150** / **1.310** \| Impact `8125` | **Phù Sinh (`Seg=385`):** HP **+14%**, Hỏa/Độc **+113**, Giảm kháng **+41**. **Phiến 150 (`Seg=400`):** 4 hệ **+120**, Nội công **+1.638**, Chính xác **+1.830**, Hội công **+14**, Thể/Trí/Nội **+170**, Thân pháp **+91**, Tất cả **+31** |
| **2. Mão** | **Chân-Trùng Lâu Khôi** | `10553113` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7515` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **3. Y phục** | **Chân-Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7511` | **9 dòng (`Seg=100`):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **Trùng Lâu Kiên Siêu Cấp** | `10553109` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7514` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **Chân-Trùng Lâu Uyển** *(hoặc Uyển Chế 116 `Quy Túc` `10214027` +117 4 hệ)* | `10553106` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7522` | **9 dòng (`Seg=100`):** HP **+12.520**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **6. Hộ thủ** | **Chân-Trùng Lâu Thủ** | `10553115` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7516` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **7. Yêu đai** | **Trùng Lâu Đới** | `10553107` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7513` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **8. Hài** | **Chân-Trùng Lâu Ngoa** | `10553117` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7512` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **9. Hạng liên** | **Trùng Lâu-Phản** *(hoặc `Chân-Trùng Lâu Liên` `10553103`)* | `10159133` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7518` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu-Triệu** & **Chân-Trùng Lâu Giới** | `10159134` `10553104` | 80 | **9★** | Nội công CB: **1.155** \| Impact `7519` & `7507` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **11. Hộ phù 1 & 2** | **Trùng Lâu-Quy** & **Chân-Trùng Lâu Ngọc** | `10159135` `10553105` | 80 | **9★** | Nội thủ CB: **1.019** \| Impact `7520` & `7503` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9·C9, lĩnh ngộ kỹ năng Hỏa công + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Kim Sí Linh Vũ** | `10155008` | 90 | 8★ | Ngoại/Nội công CB: **2.180** \| Impact `7530` | Đục 4 lỗ ngọc C9·C9, tẩy luyện phẩm chất & hiệu ứng |
| **14. Long văn (Slot 18)** | **Long Văn** | `10157001` | 80 | 5★ | Ngoại/Nội công CB: **1.540** \| Impact `12304` | **11 dòng (`Seg=4150`):** HP **+22.630**, **HP tối đa +50%**, **Băng/Hỏa/Huyền/Độc công +292**, Kháng 4 hệ **+36** |
| **15. Linh sủng / Bố đinh (Slot 19)** | **Anh Đào Bố Đinh** | `10102221` | 1 | 8★ | Ngoại/Nội công CB: **2.180** | **11 dòng (`Seg=3009`):** HP **+21.196**, **HP tối đa +20%**, **Băng/Hỏa/Huyền/Độc công +178**, Kháng 4 hệ **+37**, Tất cả thuộc tính **+18** |
| **16. Hào hiệp ấn (Slot 21)** | **Chu Tước Hào Hiệp Ấn-Bát Cực** | `10159024` | 85 | 7★ | Ngoại/Nội công CB: **1.410** | Cộng **HP + Thuộc tính 4 hệ +70 +7 hiệu ứng giảm khống chế** |

---

### Server 11: `TLBB 3.6 OFFLINE` (Phiên bản Cầm Long Thức 3.61 — Thần Khí Mã Nhã +1.500 4 Hệ & Ngọc Kép C9·9 +1.020 Hỏa)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB 3.6 OFFLINE\tlbb` | Client `E:\games\TLBB\TLBB 3.6 OFFLINE\Client TLBB`
- **Đặc trưng hệ thống:**
  - Bản **Cầm Long Thức 3.61 (`3.61.6603`)**: Có cơ chế bảo vệ kép — Client mã hóa `.packed` **2.958 file** trong `Config.axp`, đồng thời Server đổi tên các bảng cấu hình gốc (`EquipBase.txt` thật nằm ở `ImpactCol.txt` với **21.201 dòng**, `EquipSetAttr.txt` nằm ở `ItemRule.txt.bak`).
  - **Hệ thống Ngọc Cấp 9·9 Siêu Buff 3 Dòng (`GemInfo.txt`):**
    - `Hồng Minh Tinh Thạch (Cấp 9·9)` (`50921309`) / `Lục Minh Tinh Thạch (Cấp 9·9)` (`50921409`): Cộng tới **3 chỉ số trên 1 viên ngọc** — **Hỏa/Độc công +1.020, Thể lực +320, Giảm kháng Hỏa/Độc +162**!
    - `Thuần Tịnh Hồng Tinh Thạch (Cấp 9)` (`50902007`): **Hỏa công +591** | `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +528** | `Huyết Tinh Thạch (Cấp 9)` (`50913006`): **HP +28.800**.
    - `Hoàng/Lam/Hắc Bảo Thạch (Cấp 9)` (`50913001/02/05`): **Cường/Nội/Trí lực +588** | `Lục Bảo Thạch (Cấp 9)` (`50913003`): **Thân pháp +209** | `Miêu Nhãn Thạch (Cấp 9)` (`50901001`): **Nội công +8.972** | `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +3.981** | `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +45** | `Thuần Tịnh Kháng·Thông Bảo (Cấp 9)` (`50912013..16`): **Kháng +175**.
    - Đặc biệt có **Ngọc Điểm Xuyết Thời Trang Cấp 9** (`50931000..50934008`): Khảm vào Thời trang cộng **Thể lực +528**!
  - **Hệ thống Trang bị Đỉnh Nhất Phái Tiêu Dao:**
    - **Vũ khí & Thần Khí:**
      - **`Mã Nhã Chi Linh` (`10558701`, Lv108, 9★, `Seg=3019`, Impact `10493`, 13 dòng):** Ngoại/Nội công CB **1.350** — cộng **HP tối đa +50%, Băng/Hỏa/Huyền/Độc công +1.500, Ngoại/Nội công +3.000, Chính xác +3.200, Hội công +35, Nội lực +500, Thể lực +500, Thân pháp +115, Tất cả thuộc tính +50**!
      - **`Lôi Minh Ly Hỏa Phiến` (`10553239` / `10553299`, Lv96, 9★, `Seg=3010`, Impact `9281`, 11 dòng):** Chuyên biệt cho Tiêu Dao Hỏa-Độc với **HP tối đa +85%, Hỏa công +240, Độc công +240, Nội công +2.700, Chính xác +3.000, Hội công +17, Nội lực +320, Thể lực +320, Thân pháp +120, Tất cả thuộc tính +75, Giảm kháng Hỏa +55**!
      - **`Thái Cổ Phiêu Linh` (`10127486..10127564` / `10558934`, Lv106, 9★, `Seg=3009`, 17 dòng):** Cộng **HP tối đa +50%, 4 Thuộc tính +350, Nội công +2.012, Chính xác +2.000, Hội công +13, Nội/Thể/Trí +360, Thân pháp +98, Tất cả +35, Giảm kháng 4 hệ +28** *(bản `10558754` `Seg=3018`: **HP +52%, 4 hệ +400, Nội/Thể/Trí +370, 4 Giảm kháng +32**)*.
    - **Bộ 5 Món `Chân·Trùng Lâu` (`Seg=4243`, 9★, Set `274..381`):**
      - **`Chân·Trùng Lâu Liên` (`10127505`)**, **`Chân·Trùng Lâu Giới` (`10126581`)**, **`Chân·Trùng Lâu Ngọc` (`10126580`)** (19 dòng): Mỗi món cộng **HP +18.154, MP +7.184, Băng/Hỏa/Huyền/Độc công +145, Ngoại/Nội công +1.700, Chính xác +1.600, Né tránh +600, Hội công +10, Thể lực +145, Trí lực +145, Thân pháp +60, Tất cả thuộc tính +36, Giảm kháng 4 hệ +25**!
      - **`Chân·Trùng Lâu Giáp` (`10125138`)** & **`Chân·Trùng Lâu Kiên` (`10125140`)** (13 dòng): Mỗi món cộng **HP +18.154, Kháng 4 hệ +50, Ngoại/Nội thủ +1.528, Né tránh +600, Thể lực +145, Trí lực +145, Thân pháp +60, Hội thủ +10, Tất cả thuộc tính +36**!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 11 (`TLBB 3.6 OFFLINE`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Mã Nhã Chi Linh** (`10558701`) *(hoặc **Lôi Minh Ly Hỏa Phiến** `10553239` / **Thái Cổ Phiêu Linh** `10558754`)* | `10558701` `10553239` | 108 / 96 | **9★** | Nội công CB: **1.350** \| Impact `10493` / `9281` | **Mã Nhã (`Seg=3019`, 13 dòng):** **HP +50%**, **4 hệ Băng/Hỏa/Huyền/Độc công +1.500**, Ngoại/Nội công **+3.000**, CX **+3.200**, Hội **+35**, **Nội/Thể +500**, Thân **+115**, Tất cả **+50**. **Lôi Minh (`Seg=3010`):** **HP +85%**, Hỏa/Độc **+240**, **Giảm kháng Hỏa +55**, Tất cả **+75** |
| **2, 5, 6, 7, 8. Phòng cụ Chế** | **Đồ Chế 112–120 Buff (`Seg=1112..1120`)** hoặc **Đồ Chế 150 (`Seg=1142..1150`)** *(Mão `Giảo Nguyệt Cân` `10210067`, Uyển, Thủ, Đai, Hài)* | `10210067` .. `10221040` | 112–150 | **9★** | Ngoại/Nội thủ & Chính xác CB cao | **Đồ Chế 112–120 (`Seg=1112..1120` được buff vượt cấp 140):** Ngoại/Nội thủ **+1.596**, Né tránh **+595**, **Thể/Trí/Nội +166..178**, Thân pháp **+98..111**, Tất cả **+33..37** *(Đồ Chế 150 `Seg=1142..1150`: Thể **+161..169**, Thân **+117..129**, Tất cả **+39..42**)* |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10125138` | 85 | **9★** | Ngoại/Nội thủ CB: **825** \| Impact `7520` | **13 dòng (`Seg=4243`, Set 274):** **HP +18.154**, **Kháng 4 hệ +50**, Ngoại/Nội thủ **+1.528**, Né tránh **+600**, **Thể/Trí +145**, Thân pháp **+60**, Hội thủ **+10**, **Tất cả +36** |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10125140` | 85 | **9★** | Ngoại/Nội thủ CB: **625** \| Impact `7518` | **13 dòng (`Seg=4243`, Set 274):** **HP +18.154**, **Kháng 4 hệ +50**, Ngoại/Nội thủ **+1.528**, Né tránh **+600**, **Thể/Trí +145**, Thân pháp **+60**, Hội thủ **+10**, **Tất cả +36** |
| **9. Hạng liên** | **Chân·Trùng Lâu Liên** | `10127505` | 85 | **9★** | Chính xác CB: **410**, Né CB: **136** \| Impact `14284` | **19 dòng (`Seg=4243`, Set 294):** **HP +18.154**, MP **+7.184**, **Băng/Hỏa/Huyền/Độc công +145**, Ngoại/Nội công **+1.700**, Chính xác **+1.600**, Né tránh **+600**, Hội công **+10**, **Thể/Trí +145**, Thân pháp **+60**, **Tất cả +36**, **Giảm kháng 4 hệ +25** |
| **10. Giới chỉ 1 & 2** | **Chân·Trùng Lâu Giới** *(x2 — hoặc phối `10127503`)* | `10126581` `10127503` | 85 | **9★** | Ngoại/Nội công CB: **515** \| Impact `14289` | **19 dòng (`Seg=4243`, Set 294):** **HP +18.154**, MP **+7.184**, **Băng/Hỏa/Huyền/Độc công +145**, Ngoại/Nội công **+1.700**, Chính xác **+1.600**, Né tránh **+600**, Hội công **+10**, **Thể/Trí +145**, Thân pháp **+60**, **Tất cả +36**, **Giảm kháng 4 hệ +25** |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** *(x2 — hoặc phối `10127504`)* | `10126580` `10127504` | 85 | **9★** | Ngoại/Nội công CB: **515** \| Impact `14288` | **19 dòng (`Seg=4243`, Set 294):** **HP +18.154**, MP **+7.184**, **Băng/Hỏa/Huyền/Độc công +145**, Ngoại/Nội công **+1.700**, Chính xác **+1.600**, Né tránh **+600**, Hội công **+10**, **Thể/Trí +145**, Thân pháp **+60**, **Tất cả +36**, **Giảm kháng 4 hệ +25** |
| **12. Ám khí** | **Long Ngâm (`龙呤`)** | `10125128` | 115 | **9★** | Ngoại/Nội công CB: **32** \| Set `274` | **5 dòng (`Seg=3020`):** **Cường lực +50, Nội lực +50, Thể lực +50, Trí lực +50, Thân pháp +50** + Luyện chuẩn + 4 lỗ Ngọc C9·9 |
| **13. Võ hồn** | **Ngự Dao Bàn (`御瑶盘`)** *(Hệ Hỏa)* | `10125130` | 65 | 8★ | Ngoại/Nội công CB: **383** \| Set `274` | **7 dòng (`Seg=1201`):** **Kháng Hỏa +70**, **Cường lực +300, Nội lực +300, Thể lực +300, Trí lực +300, Thân pháp +180**, **Giảm kháng Hỏa +24** |
| **14. Long văn** | **Long Văn (`龙纹`)** | `10125133` | 80 | **9★** | Chính xác CB: **380**, Né CB: **115** \| Set `274` | **16 dòng (`Seg=2209`):** **HP +12.500**, **Băng/Hỏa/Huyền/Độc công +185**, Ngoại/Nội công **+1.300**, Chính xác **+2.000**, Hội công **+20**, **Cường/Nội/Thể/Trí +210**, Thân pháp **+80**, Hội thủ **+18**, **Tất cả thuộc tính +75** |
| **15–16. Thời trang & Tọa kỵ** | **Thời Trang Môn Phái / Cao Cấp 9★** (`10124567..595`) | `10124567` | 1 | **9★** | Khảm **Ngọc Điểm Xuyết C9 (+528 Thể lực)** | **7 dòng (`Seg=4232`):** **HP +15.000**, **Cường/Nội/Thể/Trí +230**, **Thân pháp +170**, **Tất cả thuộc tính +55** |

---

### Server 12: `TLBB 3D OFFLINE 3.63` (Phiên bản Thiên Long 3D Gate — Trần Ngọc C7 Buff, 6 Món Chân·Trùng Lâu Kích 2 Đại Bộ & Kim Lan Lệnh +66% HP)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB 3D OFFLINE 3.63\tlbb` | Client `E:\games\TLBB\TLBB 3D OFFLINE 3.63\Thien Long 3D\Thien Long Gate`
- **Đặc trưng hệ thống:**
  - Bản **Thiên Long 3D Offline 3.63 (`3.63.0823`)**: `EquipBase.txt` 15.744 dòng (107 cột), `ItemSegValue.txt` 657 dòng, `GemInfo.txt` 364 dòng.
  - **Trần Ngọc Giới Hạn Ở Cấp 7 Nhưng Buff Thể Lực & Thuộc Tính Tương Đương Ngọc C9 (`GemInfo.txt`):**
    - Không có ngọc cấp 8–9; trần ngọc cao nhất là **Cấp 7 (`507...`)**:
      - `Hồng Minh Tinh Thạch (Cấp 7-7)` (`50721307`) / `Lục Minh Tinh Thạch (Cấp 7-7)` (`50721407`): **Hỏa/Độc công +290 & Giảm kháng Hỏa/Độc +90**.
      - `Thuần Tịnh Hồng Tinh Thạch (Cấp 7)` (`50702007`): **Hỏa công +230** | `Hồng Bảo Thạch (Cấp 7)` (`50713004`): **Thể lực +534** (cao hơn cả Hồng Bảo Thạch C9 của đa số server!) | `Huyết Tinh Thạch (Cấp 7)` (`50713006`): **HP +29.412**.
      - `Hoàng/Lam/Hắc Bảo Thạch (Cấp 7)` (`50713001/02/05`): **Cường/Nội/Trí lực +270** | `Lục Bảo Thạch (Cấp 7)` (`50713003`): **Thân pháp +145** | `Miêu Nhãn Thạch (Cấp 7)` (`50701001`): **Nội công +2.596** | `Tử Ngọc (Cấp 7)` (`50703001`): **Chính xác +2.889** | `Biến Thạch (Cấp 7)` (`50704002`): **Hội công +18**.
  - **Hệ thống Trang bị Đỉnh Nhất Phái Tiêu Dao:**
    - **Vũ khí & Thần Khí:**
      - **`Phù Sinh` (`10304427..10304435`, Lv102, 9★, `Seg=3009`, Impact `9218..9220`, 19 dòng)** & **Ảo sức Vũ khí `Lãnh Ngọc Hàn Phương / Noãn Hương Phấn Nhụy` (`10102201..202`, `Seg=3009`, Impact `9062`, 19 dòng)**: Ngoại/Nội công CB **1.150** — cộng **HP tối đa +50%, Băng/Hỏa/Huyền/Độc công +208, Ngoại/Nội công +1.236, Chính xác +1.592, Hội công +14, Cường/Nội/Thể/Trí +152, Thân pháp +96, Tất cả thuộc tính +33, Giảm kháng 4 hệ Băng/Hỏa/Huyền/Độc +40**!
      - **`Quạt Chế 150 (`150铸造`)` (`10204030`, Lv150, 9★, `Seg=1150`, 14 dòng):** MP **+9.536**, **Băng/Huyền/Độc công +120**, Ngoại/Nội công **+1.638**, Chính xác **+1.830**, Hội công **+20**, Cường/Nội **+170**, Thể/Trí **+169**, Thân pháp **+129**, Tất cả **+42**.
    - **Bộ 6 Món `Chân·Trùng Lâu` (`10553103..10553111`, Lv50, 9★, `Seg=4321`) Kích Hoạt 2 Đại Bộ Ẩn (`Set 155` & `Set 156`):**
      - **Bộ 3 Món Công — `Chân·Trùng Lâu Liên` (`10553103`), `Chân·Trùng Lâu Giới` (`10553104`), `Chân·Trùng Lâu Ngọc` (`10553105`)** (`Seg=4321`, 19 dòng, **Set `156 Trùng Lâu·Phá Giáp`**): Mỗi món cộng **HP +15.520, Băng/Hỏa/Huyền/Độc công +200, Ngoại/Nội công +1.506, Chính xác +1.530, Hội công +20, Cường/Nội +149, Thể/Trí +148, Thân pháp +88, Tất cả thuộc tính +40, Giảm kháng 4 hệ +24**! Mặc đủ 3 món kích hoạt **Set `156` (`重楼·破甲`): Chính xác +5.277 & Hội công +89**!
      - **Bộ 3 Món Thủ — `Chân·Trùng Lâu Đai` (`10553107`), `Chân·Trùng Lâu Kiên` (`10553109`), `Chân·Trùng Lâu Giáp` (`10553111`)** (`Seg=4321`, 15 dòng, **Set `155 Trùng Lâu·Binh Tuyệt`**): Mỗi món cộng **HP +15.520, Kháng 4 hệ +24, Ngoại/Nội thủ +1.301, Né tránh +710, Cường/Nội +149, Thể/Trí +148, Thân pháp +88, Hội thủ +15, Tất cả thuộc tính +40**! Mặc đủ 3 món kích hoạt **Set `155` (`重楼·兵绝`): Né tránh +4.799 & HP tối đa +20%**!
      - Ô Nhẫn #2 và Phù #2 đeo thêm **`Trùng Lâu Giới` (`10422016`)** & **`Trùng Lâu Ngọc` (`10423024`)** (`Seg=100`, 20 dòng: HP +12.520, 4 hệ công +23, Ngoại/Nội công +1.106, CX +1.230, Né +410, Hội +10, Thể/Trí +114, Thân +62, Tất cả +21).
    - **Hệ thống Pháp Bảo, Long Văn, Lệnh Bài & Thời Trang (`Seg=4243 / 4244`):**
      - **`Kim Lan Lệnh` (`10158002`, Lv80, 9★, `Seg=4314`):** Cộng trực tiếp **HP +15.000 & HP tối đa +66%**!
      - **`Pháp Bảo·Ngự Dao Bàn` (`10156002`, Lv65, `Seg=4243`, 9 dòng):** Nội công CB **500**, Ngoại/Nội thủ CB **500** — cộng **Nội công +2.012, Chính xác +2.500, Hội công +11, Nội/Thể/Trí +148, Thân pháp +78, Hội thủ +18, Tất cả +33**.
      - **`Long Văn +9` (`10157009`, Lv80, 9★, `Seg=4244`, 11 dòng):** Ngoại/Nội thủ CB **1.000** — cộng **HP +10.000, Băng/Hỏa/Huyền/Độc công +120, Chính xác +2.500, Cường/Nội/Thể/Trí +148, Thân pháp +78**.
      - **`Thời Trang Thuộc Tính` (`10553200..10553395`, `Seg=4244`, 15 dòng):** Cộng **HP +10.000, Băng/Hỏa/Huyền/Độc công +120, Ngoại/Nội công +2.012, Chính xác +2.500, Hội công +11, Cường/Nội/Thể/Trí +148, Thân pháp +78, Tất cả +33**!

#### Bảng Build 17 Vị Trí Trang Bị Mạnh Nhất — Server 12 (`TLBB 3D OFFLINE 3.63`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(TK 102 9★ 19 dòng)* + Ảo sức **Lãnh Ngọc Hàn Phương** (`10102201`) | `10304435` `10102201` | 102 | **9★** | Nội công CB: **1.150** \| Impact `9220` & `9062` | **19 dòng (`Seg=3009`):** **HP tối đa +50%**, **Băng/Hỏa/Huyền/Độc công +208**, Ngoại/Nội công **+1.236**, Chính xác **+1.592**, Hội công **+14**, **Cường/Nội/Thể/Trí +152**, Thân pháp **+96**, Tất cả **+33**, **Giảm kháng 4 hệ +40** |
| **2, 5, 6, 8. Mão, Uyển, Thủ, Hài** | **Đồ Chế Cao Cấp 112–150 (`Thận Lâu Mão` `10210062` / Đồ Chế 150 `Seg=1150`)** | `10210062` .. `10214040` | 112–150 | **9★** | Ngoại/Nội thủ & Chính xác CB cao | **10 dòng (`Seg=112..1150`):** Ngoại/Nội thủ **+1.228..1.590**, Né tránh **+458..592**, Cường/Nội **+129..166**, **Thể/Trí +128..165**, **Thân pháp +76..123**, Hội thủ **+12..19**, **Tất cả +26..40** |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10553111` | 50 | **9★** | Ngoại/Nội thủ CB: **886** \| Impact `7520` | **15 dòng (`Seg=4321`, Set 155):** **HP +15.520**, **Kháng 4 hệ +24**, Ngoại/Nội thủ **+1.301**, Né tránh **+710**, Cường/Nội **+149**, **Thể/Trí +148**, Thân pháp **+88**, Hội thủ **+15**, **Tất cả +40** |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10553109` | 50 | **9★** | Ngoại/Nội thủ CB: **886** \| Impact `7518` | **15 dòng (`Seg=4321`, Set 155):** **HP +15.520**, **Kháng 4 hệ +24**, Ngoại/Nội thủ **+1.301**, Né tránh **+710**, Cường/Nội **+149**, **Thể/Trí +148**, Thân pháp **+88**, Hội thủ **+15**, **Tất cả +40** |
| **7. Yêu đai** | **Chân·Trùng Lâu Đai** | `10553107` | 50 | **9★** | Ngoại/Nội thủ CB: **886** \| Impact `7507` | **15 dòng (`Seg=4321`, Set 155):** **HP +15.520**, **Kháng 4 hệ +24**, Ngoại/Nội thủ **+1.301**, Né tránh **+710**, Cường/Nội **+149**, **Thể/Trí +148**, Thân pháp **+88**, Hội thủ **+15**, **Tất cả +40** *(Kích **Set 155 `Trùng Lâu·Binh Tuyệt`: +4.799 Né tránh & +20% HP**)* |
| **9. Hạng liên** | **Chân·Trùng Lâu Liên** | `10553103` | 50 | **9★** | Chính xác CB: **921**, Né CB: **521** \| Impact `7515` | **19 dòng (`Seg=4321`, Set 156):** **HP +15.520**, **Băng/Hỏa/Huyền/Độc công +200**, Ngoại/Nội công **+1.506**, Chính xác **+1.530**, Hội công **+20**, Cường/Nội **+149**, **Thể/Trí +148**, Thân pháp **+88**, **Tất cả +40**, **Giảm kháng 4 hệ +24** |
| **10. Giới chỉ 1 & 2** | **Chân·Trùng Lâu Giới** (`10553104`) & **Trùng Lâu Giới** (`10422016`) | `10553104` `10422016` | 50 | **9★** | Nội công CB: **886** & **331** \| Impact `7511` & `5952` | **Chân·Giới (`Seg=4321`, Set 156):** **HP +15.520**, **4 hệ công +200**, **Giảm kháng 4 hệ +24**, Thể/Trí **+148**, Tất cả **+40**. **Trùng Lâu Giới (`Seg=100`, 20 dòng):** HP **+12.520**, 4 hệ **+23**, Thể/Trí **+114**, Tất cả **+21** |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** (`10553105`) & **Trùng Lâu Ngọc** (`10423024`) | `10553105` `10423024` | 50 | **9★** | Nội công CB: **886** & **331** \| Impact `7503` & `5953` | **Chân·Ngọc (`Seg=4321`, Set 156):** **HP +15.520**, **4 hệ công +200**, **Giảm kháng 4 hệ +24**, Thể/Trí **+148**, Tất cả **+40** *(Kích **Set 156 `Trùng Lâu·Phá Giáp`: +5.277 Chính xác & +89 Hội công**)*. **Trùng Lâu Ngọc (`Seg=100`):** HP **+12.520**, 4 hệ **+23**, Thể/Trí **+114**, Tất cả **+21** |
| **12. Ám khí** | **Kim Sí Linh Vũ** | `10155021` `10155139` | 90 | 6★ | Ngoại/Nội công CB: **32** \| Impact `7539..7579` | **5 dòng (`Seg=2001`):** Cường/Nội/Thể/Trí/Thân **+10** + Luyện chuẩn + 4 lỗ Ngọc C7-7 |
| **13. Võ hồn** | **Pháp Bảo·Ngự Dao Bàn** | `10156002` | 65 | 4★ | Nội công CB: **500**, Ngoại/Nội thủ CB: **500** | **9 dòng (`Seg=4243`):** Nội công **+2.012**, Chính xác **+2.500**, Hội công **+11**, **Nội/Thể/Trí +148**, Thân pháp **+78**, Hội thủ **+18**, **Tất cả +33** |
| **14. Long văn** | **Long Văn +9** | `10157009` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** | **11 dòng (`Seg=4244`):** **HP +10.000**, **Băng/Hỏa/Huyền/Độc công +120**, Chính xác **+2.500**, **Cường/Nội/Thể/Trí +148**, Thân pháp **+78** |
| **15. Lệnh bài** | **Kim Lan Lệnh** | `10158002` | 80 | **9★** | Bang hội Lệnh bài | **2 dòng (`Seg=4314`):** **HP +15.000**, **HP tối đa +66%** + 4 Châu Báu |
| **16. Hào hiệp ấn** | **Thanh Long Hào Hiệp Ấn·Ngũ Hành** | `10159005` | 85 | **9★** | Ngoại/Nội công CB: **945** | **Giảm miễn hiệu ứng khống chế (`Seg=4243`)** |
| **17. Thời trang** | **Thời Trang Thuộc Tính (`Tiên Lữ Tình Duyên` / `Long Phượng Trình Tường`)** | `10553200` `10553395` | 1 | 5★ | Chính xác CB: **410**, Né CB: **136** | **15 dòng (`Seg=4244`):** **HP +10.000**, **Băng/Hỏa/Huyền/Độc công +120**, Ngoại/Nội công **+2.012**, Chính xác **+2.500**, Hội công **+11**, **Cường/Nội/Thể/Trí +148**, Thân pháp **+78**, **Tất cả +33** |

---

### Server 13: `TLBB_manda` (Mạn Đà Sơn Trang — Phiên Bản 3.3 Custom 2015)

- **Đường dẫn:** Server `E:\games\tlbb_draft\TLBB_manda\tlbb` | Client `E:\games\tlbb_draft\TLBB_manda\Client`
- **Đặc trưng hệ thống:**
  - Bản `TLBB_manda` có bước đột phá lớn ở bảng `ItemSegValue.txt`: Các món **`Chân·Trùng Lâu` (`10553113..10553121`)** được gán **`Seg=2152` (16 dòng siêu cường)** và bộ 3 trang sức **`Trùng Lâu·Phản / Triệu / Quy` (`10159133..10159135`)** được gán **`Seg=2153` (16 dòng tối thượng)**!
  - Cụ thể ở `Seg=2152`: Mỗi món cộng **HP +23.888, Băng/Hỏa/Huyền/Độc công +260, Giảm kháng 4 hệ +80, Chính xác +25.660, Hội công +140, Cường/Nội/Thể/Trí +600, Thân pháp +320, Tất cả thuộc tính +100**!
  - Ở `Seg=2153` (`Phản / Triệu / Quy`): Mỗi món cộng **HP +25.888, Băng/Hỏa/Huyền/Độc công +280, Giảm kháng 4 hệ +90, Chính xác +28.660, Hội công +150, Thể/Trí +650, Thân pháp +350, Tất cả thuộc tính +110**!
  - **Hệ thống Ngọc:** **`Lục Minh Tinh Thạch (Cấp 9·Cấp 9)` (`51921418`)**: **Kháng Huyền +350 & Giảm kháng Hỏa +150**.

#### Bảng Build 15 Vị Trí Trang Bị Mạnh Nhất — Server 13 (`TLBB_manda`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(hoặc Quạt 120 **Vong Xuyên Phiến** `10204027`)* | `10304435` `10204027` | 102 / 120 | 8★ | Nội công CB: **1.150** / **1.054** \| Impact `8125` | **Phù Sinh (`Seg=385`):** HP **+14%**, Hỏa/Độc **+113**, Giảm kháng **+41**. **Vong Xuyên (`Seg=320`):** HP **+14.930**, 4 hệ **+120**, Nội công **+1.319**, Chính xác **+1.470**, Thể/Nội/Trí **+136** |
| **2. Mão** | **Chân·Trùng Lâu Khôi** | `10553115` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7543` | **16 dòng (`Seg=2152`):** HP **+23.888**, Kháng 4 hệ **+240**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, Hội công **+140**, Cường/Nội/Thể/Trí **+600**, Thân pháp **+320**, Tất cả thuộc tính **+100** |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7511` | **9 dòng (`Seg=100`):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể/Trí **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10553109` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7514` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể/Trí **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **Chân·Trùng Lâu Uyển** | `10553121` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7546` | **16 dòng (`Seg=2152`):** HP **+23.888**, **Băng/Hỏa/Huyền/Độc công +260**, Chính xác **+25.660**, Hội công **+140**, Cường/Nội/Thể/Trí **+600**, Thân pháp **+320**, Tất cả **+100**, **Giảm kháng 4 hệ +80** |
| **6. Hộ thủ** | **Chân·Trùng Lâu Thủ** | `10553117` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7544` | **16 dòng (`Seg=2152`):** HP **+23.888**, **Băng/Hỏa/Huyền/Độc công +260**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, Hội công **+140**, Cường/Nội/Thể/Trí **+600**, Thân pháp **+320**, Tất cả **+100** |
| **7. Yêu đai** | **Chân·Trùng Lâu Đới** | `10553119` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7545` | **16 dòng (`Seg=2152`):** HP **+23.888**, Kháng 4 hệ **+240**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, Hội công **+140**, Cường/Nội/Thể/Trí **+600**, Thân pháp **+320**, Tất cả thuộc tính **+100** |
| **8. Hài** | **Chân·Trùng Lâu Ngoa** | `10553113` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7542` | **16 dòng (`Seg=2152`):** HP **+23.888**, **Băng/Hỏa/Huyền/Độc công +260**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, Hội công **+140**, Cường/Nội/Thể/Trí **+600**, Thân pháp **+320**, Tất cả **+100** |
| **9. Hạng liên** | **Trùng Lâu·Phản** | `10159133` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7518` | **16 dòng (`Seg=2153`):** HP **+25.888**, **Băng/Hỏa/Huyền/Độc công +280**, Chính xác **+28.660**, Hội công **+150**, Cường/Nội/Thể/Trí **+650**, Thân pháp **+350**, Tất cả **+110**, **Giảm kháng 4 hệ +90** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu·Triệu** *(x2 hoặc phối `Chân·Trùng Lâu Giới` `10553104`)* | `10159134` `10553104` | 80 | **9★** | Nội công CB: **1.155** \| Impact `7519` & `7507` | **Triệu (`Seg=2153`, 16 dòng):** HP **+25.888**, **4 hệ công +280**, Chính xác **+28.660**, Hội công **+150**, Thể/Trí **+650**, Thân pháp **+350**, **Giảm kháng 4 hệ +90** |
| **11. Hộ phù 1 & 2** | **Trùng Lâu·Quy** *(x2 hoặc phối `Chân·Trùng Lâu Ngọc` `10553105`)* | `10159135` `10553105` | 80 | **9★** | Nội thủ CB: **1.019** \| Impact `7520` & `7503` | **Quy (`Seg=2153`, 16 dòng):** HP **+25.888**, **4 hệ công +280**, Chính xác **+28.660**, Hội công **+150**, Thể/Trí **+650**, Thân pháp **+350**, **Giảm kháng 4 hệ +90** |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9·C9, lĩnh ngộ kỹ năng Hỏa/Độc + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Kim Sí Linh Vũ** | `10155008` | 90 | 8★ | Ngoại/Nội công CB: **2.180** \| Impact `7530` | Đục 4 lỗ ngọc C9·C9, đả thông huyệt đạo |
| **14. Long văn (Slot 18)** | **Long Văn** | `10157001` | 80 | 5★ | Ngoại/Nội công CB: **1.540** \| Impact `12304` | Mở rộng HP tối đa % + Xuyên kháng Hỏa + Thuộc tính Hỏa |
| **15. Hào hiệp ấn (Slot 21)** | **Chu Tước Hào Hiệp Ấn·Bát Cực** | `10159024` | 85 | 7★ | Ngoại/Nội công CB: **1.410** | Cộng HP + 4 hệ thuộc tính +70 +7 hiệu ứng giảm khống chế |

---

### Server 14: `TLBB 3.78` (Phiên bản Tinh Kiếm 3.78 Việt Hóa — Long Văn +629 Hỏa, Ngự Dao Bàn +50% HP/MP & Đồ Chế 150)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB 3.78\tlbb` | Client `E:\games\TLBB\TLBB 3.78\Client`
- **Đặc trưng hệ thống:**
  - Bản **Tinh Kiếm 3.78 Việt Hóa (`3.78.5000`, `ClientVer=190125`)**: `EquipBase.txt` 15.582 dòng (102 cột), `ItemSegValue.txt` 3.311 dòng, `GemInfo.txt` 770 dòng. *(Đã loại bỏ vật phẩm VIP/GM `10573592 Âm Dương Phiến [Hội Viên]` +9.999 Ngoại/Nội công).*
  - **Hệ thống Ngọc Cấp 9 & Minh Tinh Thạch Kép C9-9 (`GemInfo.txt`):**
    - `Hồng Minh Tinh Thạch (Cấp 9-9)` (`50921309`) / `Lục Minh Tinh Thạch (Cấp 9-9)` (`50921409`): **Hỏa/Độc công +350 & Giảm kháng Hỏa/Độc +150**.
    - `Thuần Tịnh Hồng Tinh Thạch (Cấp 9)` (`50902007`): **Hỏa công +300** | `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +342** | `Huyết Tinh Thạch (Cấp 9)` (`50913006`): **HP +18.780** | `Miêu Nhãn Thạch (Cấp 9)` (`50901001`): **Nội công +3.315** | `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +3.690** | `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +27**.
  - **Hệ thống Trang bị Đỉnh Nhất Phái Tiêu Dao:**
    - **Vũ khí & Thần Khí:**
      - **`Thái Cổ Phiêu Vũ` (`10308153..10308155`, Lv105, 7★, `Seg=3009`, Impact `9422..9424`, 12 dòng):** Ngoại/Nội công CB **4.096** — cộng **HP tối đa +50%, Hỏa công +136, Độc công +136, Nội công +1.236, Chính xác +1.592, Nội/Thể/Trí +152, Thân pháp +96, Tất cả thuộc tính +33, Giảm kháng Hỏa +40, Giảm kháng Độc +40**.
      - **`Lôi Minh Ly Hỏa Phiến` (`10304006` [Ngoại] 16 dòng `Seg=122` / `10304007` [Nội] 14 dòng `Seg=112`, Lv108, 9★, Impact `9703`):** Cộng **HP tối đa +50%, 4 Thuộc tính +26..27, Nội/Ngoại công +1.234..1.340, Chính xác +1.374..1.494, Né tránh +458..498, Hội công +12..14, Thể/Trí +128..139, Thân pháp +76..89, Tất cả +26..30**.
      - **`Quạt 150 Cấu Tạo` (`10204030`, Lv150, 9★, `Seg=1150`, 14 dòng):** MP **+9.536**, **Băng/Huyền/Độc công +120**, Ngoại/Nội công **+1.638**, Chính xác **+1.830**, Hội công **+20**, Cường/Nội **+170**, Thể/Trí **+169**, Thân pháp **+129**, Tất cả **+42**.
    - **Hệ thống Phụ Kiện Đột Phá Chỉ Số (`Võ Hồn` & `Long Văn`):**
      - **`Ngự Dao Bàn` (`10156002`, Lv102, `Seg=200`):** Cộng **HP +18.545, HP tối đa +50%, MP +9.536, MP tối đa +50%, Băng công +320, Giảm kháng 4 hệ Băng/Hỏa/Huyền/Độc +160**!
      - **`Long Văn Cấp 80 Max` (`10157029`, Lv80, `Seg=4085`, 19 dòng):** Cộng **HP +14.798, HP tối đa +50%, Hỏa công +629, Ngoại/Nội công +2.612, Ngoại/Nội thủ +2.592, Chính xác +1.440, Né tránh +480, Hội công +18, Cường/Nội +137, Thể/Trí +135, Thân pháp +73, Hội thủ +15, Tất cả thuộc tính +25, Giảm kháng Hỏa +38**!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 14 (`TLBB 3.78`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Thái Cổ Phiêu Vũ** *(TK 105)* hoặc **Quạt 150 Cấu Tạo** (`10204030`) | `10308155` `10204030` | 105 / 150 | 7★ / **9★** | Nội công CB: **4.096** / **1.310** \| Impact `9424` | **Thái Cổ Phiêu Vũ (`Seg=3009`, 12 dòng):** **HP +50%**, **Hỏa/Độc công +136**, **Giảm kháng Hỏa/Độc +40**, Nội công **+1.236**, CX **+1.592**, Thể/Trí/Nội **+152**, Thân **+96**, Tất cả **+33**. **Quạt 150 (`Seg=1150`):** Băng/Huyền/Độc **+120**, Thể/Trí **+169**, Thân **+129**, Tất cả **+42** |
| **2–8. Phòng cụ & Hộ uyển** | **Trọn Bộ Đồ Chế 150 (`150 Cấu Tạo` — Mão, Y phục, Kiên, Uyển, Thủ, Đai, Hài)** | `10210060` .. `10221040` | 142–150 | 8★–**9★** | Ngoại/Nội thủ & Chính xác CB chuẩn cấp 150 | **7–10 dòng (`Seg=1142..1146`):** Kháng 4 hệ **+35**, HP **+17.569..18.545**, Ngoại/Nội thủ **+1.547..1.633**, Né tránh **+577..606**, **Thể/Trí/Nội +161..169**, **Thân pháp +117..129**, **Tất cả thuộc tính +39..42** |
| **9. Hạng liên** | **Chân.Trùng Lâu Liên** / **Trùng Lâu.Phản** *(hoặc **Liên 150 Cấu Tạo** `10220040`)* | `10420090` `10420091` | 50 / 147 | **9★** | HP/MP CB: **8.040** \| Set `182` | **Chân.Liên (`Seg=100`, Set 182):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21**. **Liên 150 (`Seg=1147`):** Băng/Huyền/Độc **+119**, HP **+18.289**, Thể **+167**, Tất cả **+41** |
| **10. Giới chỉ 1 & 2** | **Chân-Trùng Lâu Giới** (`10422149`) & **Trùng Lâu.Triệu** (`10422151`) *(hoặc **Giới 150** `10222040`)* | `10422149` `10422151` | 50 / 148 | **9★** | Nội công CB: **331** \| Set `182` \| *Họa Địa Vi Lao* | **Chân-Giới (`Seg=100`, Set 182):** HP **+12.520**, MP **+6.441**, CX **+1.230**, Né **+410**, Hội **+10**, Thể/Trí **+114**, Thân **+62**, Tất cả **+21**. **Giới 150 (`Seg=1148`):** **Băng/Huyền/Độc công +119**, Thể/Trí **+164**, Thân **+122**, Tất cả **+40** |
| **11. Hộ phù 1 & 2** | **Chân Trùng Lâu Ngọc** (`10423025`) & **Trùng Lâu Ngọc** (`10423026`) | `10423025` `10423026` | 50 | **9★** | Nội công CB: **331** \| Set `161` \| *Đại Bi Phú* | **15 dòng (`Seg=100`, Set 161):** **HP +12.520**, MP **+6.441**, Băng/Huyền công **+23**, Ngoại/Nội công **+1.106**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Cường/Nội **+115**, **Thể/Trí +114**, Thân pháp **+62**, Tất cả **+21** |
| **12. Ám khí** | **Đoạn Hồn Phiêu** *(Lv120 9★)* | `10553520` `10155006` | 120 | **9★** | Ngoại/Nội công CB: **512** | **10 dòng (`Seg=4002`):** HP **+4.796**, Ngoại/Nội thủ **+824**, Chính xác **+444**, Né tránh **+148**, Cường/Nội **+46**, Thể/Trí **+44**, Thân pháp **+24**, Tất cả **+9** *(Bản `10155006` `Seg=250` có Giảm kháng Băng/Hỏa +10)* |
| **13. Võ hồn** | **Ngự Dao Bàn** | `10156002` | 102 | 5★ | Nội công CB: **500** | **8 dòng (`Seg=200`):** **HP +18.545**, **HP tối đa +50%**, MP **+9.536**, **MP tối đa +50%**, Băng công **+320**, **Giảm kháng 4 hệ Băng/Hỏa/Huyền/Độc +160** |
| **14. Long văn** | **Long Văn Cấp 80 Max** | `10157029` | 80 | 5★ | Ngoại/Nội thủ CB: **1.500** | **19 dòng (`Seg=4085`):** **HP +14.798**, **HP tối đa +50%**, **Hỏa công +629**, Ngoại/Nội công **+2.612**, Ngoại/Nội thủ **+2.592**, Chính xác **+1.440**, Né tránh **+480**, Hội công **+18**, Cường/Nội **+137**, Thể/Trí **+135**, Thân pháp **+73**, Hội thủ **+15**, Tất cả **+25**, **Giảm kháng Hỏa +38** |
| **15–16. Thời trang & Tọa kỵ** | **Thời Trang Phối Phái** (`Seg=4005`) & **Tọa Kỵ Môn Phái 9★** (`10149001..042`) | `10149035` `10553097` | 1–60 | **9★** | Tốc độ di chuyển **+80%** | **Tọa kỵ (`Seg=4006`, 13 dòng):** HP **+5.278**, **Băng/Hỏa/Huyền/Độc công +70**, Ngoại/Nội công **+930**, Ngoại/Nội thủ **+910**, Cường/Nội **+50**, Thể/Trí **+48**, Thân **+26**, Tất cả **+9** |

---

### Server 15: `TLBB 3.6` (Phiên bản TLBB 3.6 Việt Hóa — Ngọc C9 +420 Thuộc Tính & 11 Món Chân-Trùng Lâu)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB 3.6\tlbb` | Client `E:\games\TLBB\TLBB 3.6\client`
- **Đặc trưng hệ thống:**
  - Bản **TLBB 3.6 Việt Hóa (12 Môn Phái)**: `EquipBase.txt` 16.580 dòng (107 cột), `ItemSegValue.txt` 4.581 dòng, `GemInfo.txt` 516 dòng.
  - **Hệ thống Ngọc Cấp 9 Được Buff Cao Vượt Chuẩn SOHU (`GemInfo.txt`):**
    - `Tinh Khiết Hồng Tinh Thạch (Cấp 9)` (`50902006`) / `Tinh Khiết Lục Tinh Thạch (Cấp 9)` (`50902008`): **Hỏa công +420** / **Độc công +420** (cao hơn +200 điểm/viên so với chuẩn +220!).
    - `Minh Tinh Thạch Kép (Cấp 9·9)` (`50921309` / `50921409`): **Hỏa/Độc công +420 & Giảm kháng Hỏa/Độc +150**!
    - `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +513** (gấp 2.75 lần chuẩn +186!) | `Huyết Tinh Thạch (Cấp 9)` (`50913006`): **HP +33.080**!
    - `Lục Bảo Thạch (Cấp 9)` (`50912004`): **Thân pháp +279** | `Miêu Nhãn Thạch (Cấp 9)` (`50901002`): **Nội công +4.973** | `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +5.933** | `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +42** | `Tổ Mẫu Lục (Cấp 9)` (`50914001`): **Né tránh +1.978**.
    - Cho phép **khảm ngọc vào cả `Bách Bảo Nang` (`10150028`), `Bách Bảo Rương` (`10150037`) và `Tọa Kỵ`**.
  - **Hệ thống Trang bị Đỉnh Nhất Phái Tiêu Dao:**
    - **Vũ khí:** Thần Khí 102 9★ **`Phù Sinh` (`10304435`, `Seg=3009`, Impact `9220`, 16 dòng: HP +21.196, HP tối đa +20%, Băng/Hỏa/Huyền/Độc công +178, Giảm kháng 4 hệ +45, Chính xác +1.892, Cường/Nội/Thể +182, Thân pháp +136, Tất cả thuộc tính +33)** — vượt trội hoàn toàn so với Quạt Chế 150 (`10204030`, `Seg=1150`)!
    - **Hệ thống 11 Món `Chân-Trùng Lâu` (`10553103..10553119`, Lv80, 9★) phối Đồ Chế 150 (`150 Cấu Tạo`, `Seg=1142..1148`):**
      - Nhóm `Seg=100` (Set `155` & `156`): `Chân-Trùng Lâu Liên` (`10553103`), `Chân-Trùng Lâu Giới` (`10553104`), `Chân-Trùng Lâu Ngọc` (`10553105`), `Trùng Lâu Kiên Siêu Cấp` (`10553109`), `Chân-Trùng Lâu Giáp` (`10553111`), `Trùng Lâu Đới` (`10553115`) — mỗi món cộng **HP +12.520, MP +6.441, Chính xác +1.230, Né tránh +410, Hội công +10, Thể lực +114, Trí lực +114, Thân pháp +62, Tất cả thuộc tính +21** và kích hoạt hiệu ứng Trùng Lâu (`Impact 7503..7514, 32707`).
      - Nhóm `Seg=151` (`10553116..10553119`): `Chân-Trùng Lâu Thủ` (`10553116`, Impact `32708`), `Chân-Trùng Lâu Ngoa` (`10553117`, Impact `32709`), `Chân-Trùng Lâu Khôi` (`10553118`, Impact `32710`), `Chân-Trùng Lâu Uyển` (`10553119`, Impact `32711`). *(Lưu ý: Do `Seg=151` thiên về phòng ngự, người chơi muốn tối đa hóa điểm Thuộc tính và Thể lực có thể phối **Đồ Chế 150 `150 Cấu Tạo`** ở các vị trí Mão/Hài/Thủ/Uyển/Nhẫn #2/Phù #2 với `Seg=1142..1148`: **Băng/Huyền/Độc công +119, Thể lực +161..167, Thân pháp +117..125, Tất cả thuộc tính +39..41**).*

#### Bảng Build 18 Vị Trí Trang Bị Mạnh Nhất — Server 15 (`TLBB 3.6`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(Thần Khí 102 9★ 16 dòng)* | `10304435` | 102 | **9★** | Nội công CB: **1.150** \| Impact `9220` | **16 dòng (`Seg=3009`):** **HP +21.196**, **HP tối đa +20%**, **Băng/Hỏa/Huyền/Độc công +178**, **Giảm kháng 4 hệ +45**, Chính xác **+1.892**, **Cường/Nội/Thể +182**, Thân pháp **+136**, Tất cả **+33** |
| **2. Mão** | **Chân-Trùng Lâu Khôi** *(hoặc **Mão 150 Cấu Tạo** `10210060`)* | `10553118` `10210060` | 80 / 150 | **9★** / 8★ | Ngoại/Nội thủ CB: **2.227** / **733** \| Impact `32710` | **Chân-Khôi (`Seg=151`, Set 156):** Kích Set 156 + Impact `32710`. **Mão 150 (`Seg=1142`):** Kháng 4 hệ **+35**, HP **+17.569**, Thể/Trí/Nội **+161**, Thân pháp **+117**, Tất cả **+39** |
| **3. Y phục** | **Chân-Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7511` | **9 dòng (`Seg=100`, Set 155):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** |
| **4. Hộ kiên** | **Trùng Lâu Kiên Siêu Cấp** | `10553109` | 80 | **9★** | Ngoại/Nội thủ CB: **1.608** \| Impact `7514` | **9 dòng (`Seg=100`, Set 156):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** |
| **5. Hộ uyển** | **Chân-Trùng Lâu Uyển** *(hoặc **Uyển 150 Cấu Tạo** `10214040`)* | `10553119` `10214040` | 80 / 150 | **9★** / 8★ | Chính xác CB: **904** / **756** \| Impact `32711` | **Chân-Uyển (`Seg=151`, Set 156):** Impact `32711`. **Uyển 150 (`Seg=1146`):** Ngoại/Nội công **+1.596**, Chính xác **+1.782**, Hội công **+19**, Cường/Nội **+166**, Thể/Trí **+165**, Thân pháp **+123**, Tất cả **+40** |
| **6. Hộ thủ** | **Chân-Trùng Lâu Thủ** *(hoặc **Thủ 150 Cấu Tạo** `10212060`)* | `10553116` `10212060` | 80 / 150 | **9★** / 8★ | Ngoại/Nội thủ CB: **2.227** / **753** \| Impact `32708` | **Chân-Thủ (`Seg=151`, Set 156):** Impact `32708`. **Thủ 150 (`Seg=1144`):** HP **+18.049**, Ngoại/Nội thủ **+1.590**, Né tránh **+592**, Thể/Trí/Nội **+165**, Thân pháp **+123**, Tất cả **+40** |
| **7. Yêu đai** | **Trùng Lâu Đới** | `10553115` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `32707` | **9 dòng (`Seg=100`, Set 156):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** |
| **8. Hài** | **Chân-Trùng Lâu Ngoa** *(hoặc **Hài 150 Cấu Tạo** `10211060`)* | `10553117` `10211060` | 80 / 150 | **9★** / 8★ | Né tránh CB: **719** / **247** \| Impact `32709` | **Chân-Ngoa (`Seg=151`, Set 156):** Impact `32709`. **Hài 150 (`Seg=1143`):** HP **+17.809**, Ngoại/Nội thủ **+1.569**, Né tránh **+584**, Thể/Trí/Nội **+163**, Thân pháp **+120**, Tất cả **+40** |
| **9. Hạng liên** | **Chân-Trùng Lâu Liên** *(hoặc **Liên 150 Cấu Tạo** `10220040`)* | `10553103` `10220040` | 80 / 150 | **9★** / 8★ | HP/MP CB: **8.040** \| Impact `7508` | **Chân-Liên (`Seg=100`, Set 156):** HP **+12.520**, CX **+1.230**, Thể/Trí **+114**, Tất cả **+21**. **Liên 150 (`Seg=1147`):** **Băng/Huyền/Độc +119**, Kháng 4 hệ **+35**, HP **+18.289**, Thể/Trí/Nội **+167**, Tất cả **+41** |
| **10. Giới chỉ 1 & 2** | **Chân-Trùng Lâu Giới** (`10553104`) & **Giới 150 Cấu Tạo** (`10222040`) | `10553104` `10222040` | 80 / 150 | **9★** / 8★ | Nội công CB: **1.662** / **852** \| Impact `7507` | **Chân-Giới (`Seg=100`):** Khống chế *Họa Địa Vi Lao* `Impact 7507`. **Giới 150 (`Seg=1148`):** **Băng/Huyền/Độc công +119**, Ngoại/Nội công **+1.585**, CX **+1.770**, Hội **+19**, Cường/Nội **+165**, Thể/Trí **+164**, Thân **+122**, Tất cả **+40** |
| **11. Hộ phù 1 & 2** | **Chân-Trùng Lâu Ngọc** (`10553105`) & **Hộ Phù 150 Cấu Tạo** (`10223040`) | `10553105` `10223040` | 80 / 150 | **9★** / 8★ | Nội thủ CB: **1.466** / **751** \| Impact `7503` | **Chân-Ngọc (`Seg=100`):** Khống chế *Đại Bi Phú* `Impact 7503`. **Phù 150 (`Seg=1148`):** **Băng/Huyền/Độc công +119**, Ngoại/Nội công **+1.585**, CX **+1.770**, Hội **+19**, Cường/Nội **+165**, Thể/Trí **+164**, Thân **+122**, Tất cả **+40** |
| **12–18. Phụ kiện** | **Kim Sí Linh Vũ** (`10155008`), **Ngự Dao Bàn** (`10156002`), **Long Văn** (`10157001`), **Tụ Nghĩa Lệnh** (`10158003`), **Chu Tước Hào Hiệp Ấn** (`10159024`), **Bách Bảo Nang** (`10150028` 20 dòng `Seg=1092`) & **Bách Bảo Rương** (`10150037` 20 dòng `Seg=1092`) | — | 80–100 | 5★–9★ | Khảm full dàn **Ngọc C9 Buff (+420 Hỏa/Độc, +513 Thể lực, +33.080 HP)** trên cả 18 món! | Tạo nên lượng máu và sát thương thuộc tính từ Ngọc lớn bậc nhất trong các bản 3.x |

---

### Server 16 & Server 17: `TLBB_hj_ko_pack` (XJL7796) & `TLBB-HJ` (Hoài Cựu 2010 Origin `0.05.3334` — Năm 2020)

- **Đường dẫn:**
  - **STT 16 (`TLBB_hj_ko_pack`):** Server `E:\games\tlbb_draft\TLBB_hj_ko_pack\TLBB\手工端\手工端\tlbb` | Client `E:\games\tlbb_draft\TLBB_hj_ko_pack\TLBB\客户端\怀旧源端客户端`
  - **STT 17 (`TLBB-HJ`):** Server `E:\games\TLBB\TLBB-HJ\tlbb` | Client `E:\games\TLBB\TLBB-HJ\Client`
- **Đặc trưng hệ thống:**
  - Hai server này chia sẻ chung nền tảng **Hoài Cựu Nguyên Bản 9 Phái (`0.05.3334`)** (STT 16 là bản gốc tiếng Trung XJL7796, STT 17 là bản đã Việt hóa hoàn chỉnh):
  - **Điểm đặc sắc nhất của nhánh Hoài Cựu `0.05.3334`:**
    1. Hệ thống **Thiên Giám 3 Món (Slot 19, 20, 21 — `10157000..10157008`)**:
       - Slot 19: `Lưu Quang / Mộ Vũ / Tinh Huy Thiên Giám Minh Kính` (`10157000`/`003`/`006`)
       - Slot 20: `Lưu Quang / Mộ Vũ / Tinh Huy Thiên Giám Song Thứ` (`10157001`/`004`/`007`)
       - Slot 21: `Lưu Quang / Mộ Vũ / Tinh Huy Thiên Giám Tỏa Giáp` (`10157002`/`005`/`008`)
    2. Trong 5 món `Chân-Trùng Lâu` cổ điển, **`Chân-Trùng Lâu Liên` (`10420089`)** và **`Chân Trùng Lâu Ngọc` (`10423025`)** được mở lên **13 dòng (`Seg=100`)**, cộng thêm trực tiếp **Băng/Hỏa/Huyền/Độc công +83** trên mỗi món!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 16 (`TLBB_hj_ko_pack`) & Server 17 (`TLBB-HJ`)

| Vị trí | Tên trang bị tối ưu (Việt hóa STT 17 / Gốc STT 16) | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Đại Tống Quân Sầm** *(hoặc Quạt Chế 120 **Vong Xuyên Phiến** `10204027`)* | `10304111` `10204027` | 102 / 120 | 7★ / 8★ | Nội công CB: **1.050** / **1.054** \| Impact `9359` | **Đại Tống Quân Sầm (`Seg=385`):** HP **+14%**, Hỏa/Độc công **+113**, **Giảm kháng Hỏa/Độc +41**, Nội công **+1.439**, Chính xác **+1.093** *(Hoặc `Vong Xuyên Phiến` `10204027` Lv120: 4 hệ **+120**, Nội công **+1.319**, Chính xác **+1.470**, Thể/Nội **+136**)* |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** *(hoặc **Lân Mộ Xích Quán** `10210039`)* | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Chân-Trùng Lâu Giáp** | `10413104` | 90 | **9★** | Ngoại/Nội thủ CB: **1.258** \| Impact `9406` | **9 dòng (`Seg=100`):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** *(Kèm hiệu ứng Ma Giới Hộ Thể)* |
| **4. Hộ kiên** | **Chân-Trùng Lâu Kiên** | `10415056` | 90 | **9★** | Ngoại/Nội thủ CB: **907** \| Impact `9404` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** *(Giảm 5% sát thương nhận vào)* |
| **5. Hộ uyển** | **Quy Túc** *(hoặc **Huyền Thế Ma Uyển** `10214020`)* | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** Băng/Hỏa/Huyền/Độc công **+117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** *(hoặc **Lân Mộ Xích Thủ** `10212039`)* | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, MP **+7.396**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Trích Tiên Đái** *(hoặc **Bộ Nguyệt** `10221020`)* | `10221027` | 112 | 8★ | Ngoại/Nội thủ CB: **511** | **7 dòng (`Seg=318`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **8. Hài** | **Phệ Lân Thánh Lý** *(hoặc **Lân Mộ Xích Lý** `10211039`)* | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, MP **+7.276**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Chân-Trùng Lâu Liên** | `10420089` | 90 | **9★** | HP/MP CB: **8.040** \| Impact `9408` | **13 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, **Băng/Hỏa/Huyền/Độc công +83**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả **+21** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu Giới** & **Cát Tường Dạ** *(hoặc 2x `Trùng Lâu Giới`)* | `10422016` `10222027` | 90 / 115 | **9★** / 8★ | Nội công CB: **1.155** / **689** \| Impact `9401` | **Trùng Lâu Giới (`Seg=100`):** HP **+12.520**, Chính xác **+1.230**, Thể/Trí **+114**, Phong huyệt 15s. **Cát Tường Dạ (`Seg=319`):** 4 hệ **+116**, Nội công **+1.266**, Chính xác **+1.410**, Thể/Nội **+131** |
| **11. Hộ phù 1 & 2** | **Chân Trùng Lâu Ngọc** *(x2)* | `10423025` | 90 | **9★** | Nội thủ CB: **1.019** \| Impact `9402` | **13 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, **Băng/Hỏa/Huyền/Độc công +83**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả **+21** |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9, mở rộng thuộc tính Hỏa/Độc + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Băng Phách Thần Châm** | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **481** | Đục 4 lỗ ngọc C9, tẩy luyện phẩm chất & hiệu ứng |
| **14–16. Thiên giám (Slot 19, 20, 21)** | **Tinh Huy Thiên Giám Minh Kính / Song Thứ / Tỏa Giáp** | `10157006` `10157007` `10157008` | 85 | 7★ | Bộ 3 món **Thiên Giám** đặc trưng Hoài Cựu | Tăng mạnh **Thuộc tính công + Giảm kháng + Giới hạn Sinh lực + Kháng khống chế** |

---

### Server 18: `暗黑群服HoaiNiem` (Phiên bản Ám Hắc Quần Phục 12 Phái — Chí Tôn Trùng Lâu & 6 Thiên Giám)

- **Đường dẫn:** Server `E:\games\TLBB\暗黑群服HoaiNiem\tlbb` | Client `E:\games\TLBB\暗黑群服HoaiNiem\暗黑群服`
- **Đặc trưng hệ thống:**
  - Bản Mod **Ám Hắc Quần Phục (暗黑群服)** trên nền **3.x 12 Môn Phái** (có Đường Môn, Quỷ Cốc): `EquipBase.txt` 15.082 dòng (103 cột), `ItemSegValue.txt` 5.274 dòng, `GemInfo.txt` 2.827 dòng.
  - **Mở rộng số lượng ô trang bị lên tới 21 vị trí** (ngoài 12 ô cơ bản + Võ Hồn + Ám Khí + Long Văn, còn mở thêm **`Bách Bảo Nang` (Hành Nang)**, **`Bách Bảo Rương` (Cách Hạp)** và **Bộ 6 Món `Thiên Giám Linh Vũ` ở các ô Point `19, 20, 21, 22, 23, 24`**).
  - **Hệ thống Ngọc Cấp 9 Tinh Khiết & Minh Tinh Thạch Kép (`GemInfo.txt`):**
    - `Thuần Tịnh Hồng Tinh Thạch (Cấp 9)` (`50902006`) / `Thuần Tịnh Lục Tinh Thạch (Cấp 9)` (`50902008`): **Hỏa công +350** / **Độc công +350**.
    - `Minh Tinh Thạch Kép (Cấp 9·9)` (`50921309` / `50921409`): **Hỏa/Độc công +350 & Giảm kháng Hỏa/Độc +150** trên cùng 1 viên ngọc!
    - `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +342** | `Huyết Tinh Thạch (Cấp 9)` (`50913006`): **Giới hạn HP +22.053**.
    - `Miêu Nhãn Thạch (Cấp 9)` (`50901002`): **Nội công +3.315** | `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +3.955** | `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +28**.
    - **Đặc biệt:** Cột `EquipPoint` trong `GemInfo.txt` cho phép **khảm ngọc trực tiếp vào cả `Bách Bảo Nang` (Hành Nang), `Bách Bảo Rương` (Cách Hạp) và `Tọa Kỵ` (Thú cưỡi)**!
  - **Hệ thống Trang bị Đỉnh Nhất — Sự Kết Hợp Giữa 5 Món `Chí Tôn·Trùng Lâu` (`Seg=5004`) & 5 Món `Chân·Trùng Lâu` (`Seg=2152`):**
    1. **Nhóm 5 Món `Chí Tôn·Trùng Lâu` (`10553172..10553176`, Lv80, 9★, `Seg=5004`, 16 dòng):**
       - **`Chí Tôn·Trùng Lâu Kiên` (`10553172`, Impact `5638`, Set `213`)** & **`Chí Tôn·Trùng Lâu Giáp` (`10553173`, Impact `5639`, Set `213`)**: Mỗi món cộng **HP +32.097, HP tối đa +11%, MP +6.441, Kháng Băng/Hỏa/Huyền/Độc +240, Chính xác +6.542, Né tránh +4.073, Hội công +91, Cường/Nội/Thể/Thân +508, Hội thủ +41, Tất cả thuộc tính +220**!
       - **`Chí Tôn·Trùng Lâu Liên` (`10553174`, Set `212`)**, **`Chí Tôn·Trùng Lâu Giới` (`10553175` x2, Set `212`)**, **`Chí Tôn·Trùng Lâu Ngọc` (`10553176` x2, Set `212`)**: Mỗi món cộng **HP +32.097, HP tối đa +11%, Băng/Hỏa/Huyền/Độc công +493, Hội công +91, Cường/Nội/Thể/Thân +508, Tất cả thuộc tính +220, Giảm kháng Băng/Hỏa/Huyền/Độc +160**!
    2. **Nhóm 5 Món `Chân·Trùng Lâu` (`10553113..10553121`, Lv80, 9★, `Seg=2152`, 16 dòng, Set `203`):**
       - **`Chân·Trùng Lâu Ngoa/Khôi/Thủ/Đái` (`10553113`/`10553115`/`10553117`/`10553119`)**: Mỗi món cộng **HP +23.208, Kháng 4 hệ +143, Ngoại/Nội thủ +1.399, Né tránh +2.522, Hội công +140, Cường/Nội/Thể/Trí +600, Thân pháp +297, Hội thủ +250, Tất cả thuộc tính +432**!
       - **`Chân·Trùng Lâu Uyển` (`10553121`, Impact `7546`)**: Cộng **HP +23.208, Băng/Hỏa/Huyền/Độc công +360, Chính xác +35.660, Hội công +140, Cường/Nội/Thể +600, Thân pháp +297, Tất cả thuộc tính +432, Giảm kháng 4 hệ +188**!

#### Bảng Build 21 Vị Trí Trang Bị Mạnh Nhất — Server 18 (`暗黑群服HoaiNiem`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** *(TK 102 16 dòng)* hoặc **Phiến 150 Chế** (`10204031`) | `10304435` `10204031` | 102 / 150 | 8★ | Nội công CB: **1.150** / **1.310** \| Impact `9558` / `9281` | **Phù Sinh (`Seg=3010`, 16 dòng):** HP + **HP%**, **4 hệ Băng/Hỏa/Huyền/Độc công**, **4 Giảm kháng**, Chính xác, Cường/Nội/Thể/Thân, Tất cả thuộc tính. **Phiến 150 (`Seg=1150`, 12 dòng):** MP **+9.536**, Băng/Huyền/Độc **+120**, Ngoại/Nội công **+1.638**, CX **+1.830**, Hội **+20**, Cường/Nội **+170**, Thể/Trí **+169**, Thân **+129**, Tất cả **+42** |
| **2. Mão** | **Chân·Trùng Lâu Khôi** | `10553115` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7543` | **16 dòng (`Seg=2152`, Set 203):** HP **+23.208**, Kháng 4 hệ **+143**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, **Hội công +140**, **Cường/Nội/Thể/Trí +600**, Thân pháp **+297**, **Hội thủ +250**, **Tất cả thuộc tính +432** |
| **3. Y phục** | **Chí Tôn·Trùng Lâu Giáp** | `10553173` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `5639` | **16 dòng (`Seg=5004`, Set 213):** **HP +32.097**, **HP tối đa +11%**, MP **+6.441**, **Kháng 4 hệ +240**, Chính xác **+6.542**, Né tránh **+4.073**, Hội công **+91**, **Cường/Nội/Thể/Thân +508**, Hội thủ **+41**, **Tất cả +220** |
| **4. Hộ kiên** | **Chí Tôn·Trùng Lâu Kiên** | `10553172` | 80 | **9★** | Ngoại/Nội thủ CB: **1.608** \| Impact `5638` | **16 dòng (`Seg=5004`, Set 213):** **HP +32.097**, **HP tối đa +11%**, MP **+6.441**, **Kháng 4 hệ +240**, Chính xác **+6.542**, Né tránh **+4.073**, Hội công **+91**, **Cường/Nội/Thể/Thân +508**, Hội thủ **+41**, **Tất cả +220** |
| **5. Hộ uyển** | **Chân·Trùng Lâu Uyển** | `10553121` | 80 | **9★** | Chính xác CB: **904** \| Impact `7546` | **16 dòng (`Seg=2152`, Set 203):** HP **+23.208**, **Băng/Hỏa/Huyền/Độc công +360**, **Chính xác +35.660**, **Hội công +140**, **Cường/Nội/Thể +600**, Thân pháp **+297**, **Tất cả +432**, **Giảm kháng 4 hệ +188** |
| **6. Hộ thủ** | **Chân·Trùng Lâu Thủ** | `10553117` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7544` | **16 dòng (`Seg=2152`, Set 203):** HP **+23.208**, Kháng 4 hệ **+143**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, **Hội công +140**, **Cường/Nội/Thể/Trí +600**, Thân pháp **+297**, **Hội thủ +250**, **Tất cả +432** |
| **7. Yêu đai** | **Chân·Trùng Lâu Đái** | `10553119` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7545` | **16 dòng (`Seg=2152`, Set 203):** HP **+23.208**, Kháng 4 hệ **+143**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, **Hội công +140**, **Cường/Nội/Thể/Trí +600**, Thân pháp **+297**, **Hội thủ +250**, **Tất cả +432** |
| **8. Hài** | **Chân·Trùng Lâu Ngoa** | `10553113` | 80 | **9★** | Né tránh CB: **719** \| Impact `7542` | **16 dòng (`Seg=2152`, Set 203):** HP **+23.208**, Kháng 4 hệ **+143**, Ngoại/Nội thủ **+1.399**, Né tránh **+2.522**, **Hội công +140**, **Cường/Nội/Thể/Trí +600**, Thân pháp **+297**, **Hội thủ +250**, **Tất cả +432** |
| **9. Hạng liên** | **Chí Tôn·Trùng Lâu Liên** | `10553174` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `5636` | **16 dòng (`Seg=5004`, Set 212):** **HP +32.097**, **HP tối đa +11%**, **Băng/Hỏa/Huyền/Độc công +493**, Hội công **+91**, **Cường/Nội/Thể/Thân +508**, **Tất cả +220**, **Giảm kháng 4 hệ +160** |
| **10. Giới chỉ 1 & 2** | **Chí Tôn·Trùng Lâu Giới** *(x2)* | `10553175` | 80 | **9★** | Nội công CB: **1.662** \| Impact `5634` | **16 dòng (`Seg=5004`, Set 212):** **HP +32.097**, **HP tối đa +11%**, **Băng/Hỏa/Huyền/Độc công +493**, Hội công **+91**, **Cường/Nội/Thể/Thân +508**, **Tất cả +220**, **Giảm kháng 4 hệ +160** |
| **11. Hộ phù 1 & 2** | **Chí Tôn·Trùng Lâu Ngọc** *(x2)* | `10553176` | 80 | **9★** | Nội thủ CB: **1.466** \| Impact `5632` | **16 dòng (`Seg=5004`, Set 212):** **HP +32.097**, **HP tối đa +11%**, **Băng/Hỏa/Huyền/Độc công +493**, Hội công **+91**, **Cường/Nội/Thể/Thân +508**, **Tất cả +220**, **Giảm kháng 4 hệ +160** |
| **12. Ám khí** | **Kim Sí Linh Vũ** | `10155008` | 90 | 5★ | Ngoại/Nội công CB: **2.180** \| Impact `26000` | Khảm 4 lỗ Ngọc C9·9 (+350 Hỏa, +150 Giảm kháng) + Luyện chuẩn + Kỹ năng Ám khí |
| **13. Võ hồn** | **Ngự Dao Bàn** / **Lưu Ly Diễm** | `10156002` `10156001` | 1 | 5★ | Nội/Ngoại công CB: **227** | Mở rộng thuộc tính Võ Hồn cấp 9 + 3 kỹ năng Võ Hồn + 4 lỗ Ngọc |
| **14. Long văn** | **Long Văn (+100)** | `10156602` | 1 | **9★** | Ngoại/Nội công CB: **414** \| Impact `9281` | **13 dòng (`Seg=4150`):** HP **+12.520**, Ngoại/Nội công **+1.688**, Chính xác **+1.230**, Hội công **+10**, Cường/Nội **+165**, Thể/Trí **+164**, Thân pháp **+122**, Tất cả **+40** |
| **15. Hành nang & Cách hạp** | **Bách Bảo Nang** (`10150028`) & **Bách Bảo Rương** (`10150037`) | `10150028` `10150037` | 100 | **9★** | Khảm thêm **4 lỗ Ngọc C9** mỗi món! | **20 dòng/món (`Seg=1092`):** HP **+11.556**, **4 hệ Băng/Hỏa/Huyền/Độc công +91**, Kháng 4 hệ **+22**, **Giảm kháng 4 hệ +11**, Ngoại/Nội công **+1.027**, CX **+1.146**, Hội **+10**, Thể/Trí/Nội **+106**, Thân **+57**, Tất cả **+20** |
| **16–21. Bộ 6 Thiên Giám** | **Lưu Quang / Mộ Vũ / Tinh Huy / Khinh Phong / Dạ Tuyết / Ngự Lôi Thiên Giám** | `10157000` .. `10157081` | 85 | 7★ | Trang bị vào 6 ô Point `19, 20, 21, 22, 23, 24` | Kích hoạt trọn bộ 6 món Thiên Giám Linh Vũ cộng dồn lượng máu, thuộc tính 4 hệ và xuyên kháng khổng lồ |

---

### Server 19 (`tlbb_lee`), Server 20 (`tlbb129`), Server 28 (`Skynet-bot`) & Server 29 (`Level Up Games`) — Hệ ChangYou 3.72 (11 Phái Quỷ Cốc — 2022–2026)

Bốn server **STT 19 (`tlbb_lee`)**, **STT 20 (`tlbb129`)**, **STT 28 (`Skynet-bot`)** và **STT 29 (`Level Up Games`)** đều thuộc đại gia đình **ChangYou 3.72 (11 Phái — có Đường Môn & Quỷ Cốc)** với đầy đủ các hệ thống hiện đại nhất: **Thần Khí Luyện Hồn 102 (`Phù Sinh Quan Khích` `10317041`), Điêu Văn Kép, Tinh Thông Trang Bị, Chân Nguyên Cấp 9, Long Văn, Tụ Nghĩa Lệnh (`10158003`), Hào Hiệp Ấn (`10159024`), Võ Đạo/Thiên Đạo, Con Cái, Thần Đỉnh**.

Tuy nhiên, khi soi sâu vào bảng `EquipBase.txt` và `ItemSegValue.txt`, **4 server này phân hóa thành 3 biến thể Build End-Game khác nhau rõ rệt**:

---

#### Server 19: `tlbb_lee` (1510 Docker / ChangYou 3.72 Mở Rộng — 2022–2024)

- **Đường dẫn:** Server `E:\games\tlbb_lee\tlbb-docker\tlbbsrc1510\tlbbsrc` | Client `E:\games\tlbb_lee\CLIENT`
- **Đã sửa 07/11/2026** (đối chiếu `Public/Config/EquipBase.txt` + `ItemSegValue.txt` của `tlbb_lee`): Seg/Cấp của Vũ khí, Mão, Giáp, Kiên, Đai và chỉ số Giáp/Kiên/Đai; 6 món `Chân-Trùng Lâu` (`10553103..113`) ở `tlbb_lee` có `SetID=-1` nên **không kích bộ ẩn** (khác `tlbb129`). Các hàng còn lại chưa đối chiếu lại.
- **Điểm khác biệt cốt lõi so với `tlbb129`:**
  1. Ngoài 6 món `Chân-Trùng Lâu` chính thống (`10553103..10553113`), `tlbb_lee` **mở thêm 4 món `Chân-Trùng Lâu` nối tiếp (`10553114..10553118`, `Seg=5007`)** gồm:
     - **`Chân-Trùng Lâu Uyển` (`10553114`):** **Băng/Hỏa/Huyền/Độc công +320**, HP **+12.520**, Thể lực **+114**, Chính xác **+1.230**, Hội công **+10**, Thân pháp **+62**, Tất cả **+21**!
     - **`Chân-Trùng Lâu Thủ` (`10553115`):** **Băng/Hỏa/Huyền/Độc công +320**, HP **+12.520**, Thể lực **+114**, Ngoại/Nội thủ **+1.101**, Hội thủ **+10**!
     - **`Chân-Trùng Lâu Ngoa` (`10553116`):** **Băng/Hỏa/Huyền/Độc công +320**, HP **+12.520**, Thể lực **+114**, Né tránh **+410**, Hội thủ **+10**!
     - **`Chân-Trùng Lâu Khôi` (`10553118`):** **Ngoại thủ cơ bản +50%**, HP **+12.520**, Kháng 4 hệ **+8**, Thể/Trí **+114** (hoặc dùng **`Siêu Cấp Tai Thỏ` `10410035`**: **HP +17%, 4 hệ công +215, Thể lực +368, Thân pháp +368, Tất cả +188**!).
  2. Ngoài ra trong `EquipBase.txt` của `tlbb_lee` (và `tlbb129`, `Level Up Games`) còn có dải trang bị sự kiện cao cấp `10555710..10555719` (`Seg=5037`, 9★, 10–20 dòng: mỗi món cộng **HP +120.000, HP% +17%, 4 hệ công +368, Giảm kháng 4 hệ +55, Thể/Thân +368, Tất cả +188**) nếu server bật shop sự kiện đặc biệt. Ở chế độ chuẩn, bộ **`Chân-Trùng Lâu` (`10553103..10553118`) + `Siêu Cấp Tai Thỏ` (`10410035`)** là chuẩn mực End-Game.

##### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 19 (`tlbb_lee`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh Quan Khích** *(Thần Khí Luyện Hồn 102)* | `10317041` | 102 | 8★ | Nội công CB: **1.150** \| Impact `8118` | **`Seg=3009`** *(bản cũ ghi `Seg=389`)*; `EquipBase` không có dòng trọng số — thuộc tính đến từ cơ chế Luyện Hồn/Impact, **chưa đối chiếu lại**. |
| **2. Mão** | **Siêu Cấp Tai Thỏ** *(hoặc `Chân-Trùng Lâu Khôi` `10553118`)* | `10410035` | 100 | **9★** | Ngoại/Nội thủ CB: **1.545** \| Impact `10663` | **9 dòng (`Seg=102`)** *(bản cũ ghi `Seg=5032`)*: **HP tối đa +17%**, **Băng/Hỏa/Huyền/Độc công +215**, Thể lực **+368**, Thân pháp **+368**, Hội thủ **+288**, Tất cả thuộc tính **+188** |
| **3. Y phục** | **Chân-Trùng Lâu Giáp** | `10553111` | 50 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7511` | **9 dòng (`Seg=5009`):** HP **+12.520**, **HP tối đa +8%**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả **+21** *(bản cũ ghi HP +22.000, Né +620, Thể +288, Tất cả +88 — không có trong `ItemSegValue`)* |
| **4. Hộ kiên** | **Chân-Trùng Lâu Kiên** | `10553109` | 50 | **9★** | Ngoại/Nội thủ CB: **1.608** \| Impact `7514` | **9 dòng (`Seg=5008`):** HP **+12.520**, **HP tối đa +8%**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **Chân-Trùng Lâu Uyển** | `10553114` | 80 | **9★** | Chính xác CB: **1.000** \| Impact `7522` | **9 dòng (`Seg=5007`):** HP **+12.520**, **Băng/Hỏa/Huyền/Độc công +320**, Chính xác **+1.230**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **6. Hộ thủ** | **Chân-Trùng Lâu Thủ** | `10553115` | 80 | **9★** | Ngoại/Nội thủ CB: **1.000** \| Impact `7516` | **9 dòng (`Seg=5007`):** HP **+12.520**, **Băng/Hỏa/Huyền/Độc công +320**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả **+21** |
| **7. Yêu đai** | **Chân-Trùng Lâu Đai** | `10553113` | 50 | **9★** | Ngoại/Nội thủ CB: **1.500** \| Impact `7513` | **9 dòng (`Seg=5007`):** HP **+12.520**, **HP tối đa +11%**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả **+21** *(trong `tlbb_lee` 6 món `Chân` này có `SetID=-1` — **không kích bộ ẩn nào**, khác với `tlbb129`)* |
| **8. Hài** | **Chân-Trùng Lâu Ngoa** | `10553116` | 80 | **9★** | Né tránh CB: **1.000** \| Impact `7512` | **9 dòng (`Seg=5007`):** HP **+12.520**, **Băng/Hỏa/Huyền/Độc công +320**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả **+21** |
| **9. Hạng liên** | **Chân-Trùng Lâu Liên** | `10553103` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7508` | **12 dòng (`Seg=5010`):** **HP tối đa +8%**, **Băng/Hỏa/Huyền/Độc công +215**, Chính xác **+1.600**, Né tránh **+480**, Hội công **+20**, Thể lực **+288**, Trí lực **+168**, Thân pháp **+128**, Tất cả **+88** |
| **10. Giới chỉ 1 & 2** | **Chân-Trùng Lâu Giới** & **Trùng Lâu Giới** | `10553104` `10422016` | 80 / 90 | **9★** | Nội công CB: **1.662** & **1.155** \| Impact `7507` & `7506` | **Chân-Giới (`Seg=5010`):** **HP +8%**, **4 hệ công +215**, Chính xác **+1.600**, Hội công **+20**, Thể lực **+288**, Tất cả **+88**. **Trùng Lâu Giới (`10422016`, `Seg=100`):** HP **+12.520**, Hội công **+45**, Thể/Trí/Thân **+114**, Tất cả **+68** |
| **11. Hộ phù 1 & 2** | **Chân-Trùng Lâu Ngọc** & **Trùng Lâu Ngọc** | `10553105` `10423024` | 80 / 90 | **9★** | Nội thủ CB: **1.466** & **1.019** \| Impact `7503` & `7502` | **Chân-Ngọc (`Seg=5010`):** **HP +8%**, **4 hệ công +215**, Chính xác **+1.600**, Thể lực **+288**, Tất cả **+88** *(trong `tlbb_lee` món này có `SetID=-1` — không kích bộ ẩn; bộ Thiên Thương Set 183 chỉ có ở `tlbb129`)*. **Trùng Lâu Ngọc (`Seg=100`):** Hội công **+45**, Tất cả **+68** |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9·C9, lĩnh ngộ kỹ năng Hỏa công + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Kim Sí Linh Vũ** | `10155008` | 90 | 8★ | Ngoại/Nội công CB: **2.180** \| Impact `7530` | Đục 4 lỗ ngọc C9·C9, đả thông huyệt đạo + Luyện chuẩn |
| **14. Long văn (Slot 18)** | **Long Văn** | `10157001` | 80 | 5★ | Ngoại/Nội công CB: **1.540** \| Impact `12304` | Tăng % HP tối đa + Xuyên kháng Hỏa + Thuộc tính Hỏa công |
| **15. Lệnh bài (Slot 20)** | **Tụ Nghĩa Lệnh** | `10158003` | 80 | 5★ | Chính xác CB: **840** | **12 dòng (`Seg=4001`):** HP **+4.676**, **HP tối đa +50%**, **Băng/Hỏa/Huyền/Độc công +64**, Kháng 4 hệ **+21**, khảm 4 Châu Báu Tứ Tượng |
| **16. Hào hiệp ấn (Slot 21)** | **Chu Tước Hào Hiệp Ấn - Bát Cực** | `10159024` | 85 | 7★ | Ngoại/Nội công CB: **1.410** | Cộng **HP + 4 hệ thuộc tính +70 +7 hiệu ứng giảm khống chế** |

---

#### Server 20: `tlbb129` (129 VM / ChangYou 3.72 Chuẩn 6 Chân·Trùng Lâu — 2022–2024)

- **Đường dẫn:** Server `E:\games\tlbb_draft\tlbb129\tlbb` | Client `E:\games\tlbb_draft\tlbb129\2.Client`
- **Đã đối chiếu lại ngày 07/11/2026** trực tiếp với `tlbb/Public/Config/EquipBase.txt`, `ItemSegValue.txt`, `EquipSetAttr.txt` của `tlbb129` (cột `ValueSeg` = cột 90 của `EquipBase`; mỗi dòng thuộc tính đang bật lấy giá trị thô ở cột tương ứng của `ItemSegValue`). Bản trước ghi sai ID/Seg của nhiều món và **lấy chỉ số Giáp/Kiên/Đai từ nguồn khác** — đã sửa bên dưới.
- **Đặc trưng hệ thống:**
  - Khác với `tlbb_lee`, `EquipBase.txt` của `tlbb129` **không có** 4 món `10553114..10553118` (Uyển/Thủ/Ngoa/Khôi `Seg=5007`) *(đã xác nhận: 5 ID này không tồn tại)*.
  - **2 Bộ Ẩn `Trùng Lâu Thiên Thương` / `Thiên Hựu` — mỗi bộ có 2 phiên bản (thường & Chân):**
    - **Set `183` = Thiên Thương (Công):** `Chân·Trùng Lâu Liên / Giới / Ngọc` (`10553103 / 04 / 05`, `Seg=5010`). Bản thường là Set `182` (`10553100 / 01 / 02`, `Seg=100`).
    - **Set `208` = Thiên Hựu (Thủ):** `Chân·Trùng Lâu Giáp / Kiên / Đai` (`10553111 / 09 / 13`). Bản thường là Set `207` (`10553110 / 08 / 12`).
    - Mỗi bộ cần đủ **3 món** (cột 2 của `EquipSetAttr` = `3`); hiệu ứng gắn qua Impact `5974` (Set 183) và `5997` (Set 208) — **giá trị cộng của hai Impact này chưa được đối chiếu lại** (bản cũ ghi “+40% HP, +400 Thể/Nội/Xuyên thích”; coi là chưa kiểm chứng).
    - *Bản cũ gọi nhầm Giáp/Kiên/Đai là Set 182 và Liên/Giới/Ngọc là Set 183 — Giáp/Kiên/Đai thực tế thuộc Set 208.*
  - Mão dùng **`Siêu Cấp Tai Thỏ` (`10410035`, 9★, Lv100, `Seg=102`)**; **Thần Khí** `Phù Sinh Quan Khích` (`10317041`, Lv102, `Seg=3009`, không có dòng trọng số trong `EquipBase` → thuộc tính đến từ cơ chế Luyện Hồn, **chưa đối chiếu**).
  - Ở Uyển/Thủ/Hài dùng **Đồ Chế 116–126**: `Quy Túc` (`10214027`, Lv116, `Seg=1116`), `Huyễn Thế Ma Oản` (`10214031`, Lv126, `Seg=1096`), `Phệ Lân Huyền Thủ` (`10212098`, Lv126, `Seg=1096`), `Thệ Lân Thánh Lý` (`10211092`, Lv124, `Seg=1094`). **Các món này KHÔNG có dòng Băng/Hỏa/Huyền/Độc công** (chỉ Ngoại/Nội công hoặc thủ + chính xác/né + 5 thuộc tính chính); bản cũ nói “4 hệ +120” là sai. Muốn có 4 hệ ở các ô này phải dùng dải sự kiện `10555710+` (`Seg=5037`).
  - `Ngự Dao Bàn` (`10156002`) có `Seg=-1` (chỉ số nằm ở Impact), `Tụ Nghĩa Lệnh` (`10158003`, `Seg=4001`) và `Hào Hiệp Ấn` (`10159024`, `Seg=2408`) **không có dòng trọng số** trong `EquipBase` → chỉ số nằm ở Impact/cơ chế riêng.

##### Bảng Build Trang Bị Tiêu Dao — Server 20 (`tlbb129`) — đối chiếu thực tế 07/11/2026

> Cột cuối là **giá trị thô** từ `ItemSegValue` cho các dòng đang bật (`(x2)` = trọng số dòng bằng 2). Tên là tên Việt hóa hiện tại trong `EquipBase.txt` (có chỗ khác tên Hán-Việt trong tài liệu cũ: `Chân - Trọng Lâu` = `Chân·Trùng Lâu`).

| Vị trí | Tên (trong EquipBase) | Item ID | Cấp | ValueSeg | Dòng thuộc tính (giá trị thô) |
| :--- | :--- | :---: | :---: | :---: | :--- |
| 1. Vũ khí | Phù Sinh Quan Khích | `10317041` | 102 | `3009` | — |
| 2. Mão | Siêu cấp Tai Thỏ | `10410035` | 100 | `102` | HP% 17, Băng công 215, Hỏa công 215, Huyền công 215, Độc công 215, Thể lực 368, Thân pháp 368, Hội thủ 288, Tất cả thuộc tính 188 |
| 3. Y phục | Chân - Trọng Lâu Giáp | `10553111` | 50 | `5009` | HP 12520, HP% 8, Ngoại thủ 1101, Nội thủ 1101, Né tránh 410, Thể lực 114, Thân pháp 62, Hội thủ 10, Tất cả thuộc tính 21 |
| 4. Hộ kiên | Chân - Trọng Lâu Kiên | `10553109` | 50 | `5008` | HP 12520, HP% 8, Ngoại thủ 1101, Nội thủ 1101, Né tránh 410, Thể lực 114, Thân pháp 62, Hội thủ 10, Tất cả thuộc tính 21 |
| 5. Hộ uyển | Quy Túc | `10214027` | 116 | `1116` | Ngoại công 1276, Nội công 1276, Chính xác 1422, Hội công 13, Cường lực 133, Linh khí 133, Thể lực 132, Định lực 132, Thân pháp 81, Tất cả thuộc tính 27 |
| 5b. Hộ uyển 126 | Huyễn Thế Ma Oản | `10214031` | 126 | `1096` | Ngoại công 1063, Nội công 1063, Chính xác 1182, Hội công 9, Cường lực 111, Linh khí 111, Thể lực 110, Định lực 110, Thân pháp 60, Tất cả thuộc tính 20 |
| 6. Hộ thủ | Phệ Lân Huyền Thủ | `10212098` | 126 | `1096` | Ngoại thủ 1058, Nội thủ 1058, Né tránh 394, Cường lực 111, Linh khí 111, Thể lực 110 (x2), Định lực 110, Thân pháp 60 (x2), Hội thủ 9, Tất cả thuộc tính 20 |
| 7. Yêu đai | Chân - Trọng Lâu Đai | `10553113` | 50 | `5007` | HP 12520, HP% 11, Ngoại thủ 1101, Nội thủ 1101, Né tránh 410, Thể lực 114, Thân pháp 62, Hội thủ 10, Tất cả thuộc tính 21 |
| 8. Hài | Thệ Lân Thánh Lý | `10211092` | 124 | `1094` | Ngoại thủ 1037, Nội thủ 1037, Né tránh 386, Cường lực 109, Linh khí 109, Thể lực 108 (x2), Định lực 108, Thân pháp 59 (x2), Hội thủ 9, Tất cả thuộc tính 20 |
| 9. Hạng liên | Chân - Trọng Lâu Liên | `10553103` | 50 | `5010` | HP% 8, Băng công 215, Hỏa công 215, Huyền công 215, Độc công 215, Chính xác 1600, Thể lực 288, Thân pháp 128, Tất cả thuộc tính 88 |
| 10a. Giới chỉ 1 | Chân - Trọng Lâu Giới | `10553104` | 50 | `5010` | HP% 8, Băng công 215, Hỏa công 215, Huyền công 215, Độc công 215, Chính xác 1600, Thể lực 288, Thân pháp 128, Tất cả thuộc tính 88 |
| 10b. Giới chỉ 2 | Trùng Lâu Giới | `10422016` | 50 | `100` | HP 12520, MP 6441, Chính xác 1230, Né tránh 488, Hội công 45, Thể lực 114, Định lực 114, Thân pháp 114, Tất cả thuộc tính 68 |
| 11a. Hộ phù 1 | Chân - Trọng Lâu Ngọc | `10553105` | 50 | `5010` | HP% 8, Băng công 215, Hỏa công 215, Huyền công 215, Độc công 215, Chính xác 1600, Thể lực 288, Thân pháp 128, Tất cả thuộc tính 88 |
| 11b. Hộ phù 2 | Trùng Lâu Ngọc | `10423024` | 50 | `100` | HP 12520, MP 6441, Chính xác 1230, Né tránh 488, Hội công 45, Thể lực 114, Định lực 114, Thân pháp 114, Tất cả thuộc tính 68 |
| 12. Ngự Dao Bàn | Ngự Dao Bàn | `10156002` | 65 | `-1` | — |
| 13. Kim Sí Linh Vũ | Kim Sí Lính Vũ | `10155008` | 90 | `2001` | Cường lực 10, Linh khí 10, Thể lực 10, Định lực 10, Thân pháp 10 |
| 14. Long Văn | Long văn | `10157001` | 80 | `4001` | HP 4676, MP 2421, Băng công 64, Hỏa công 64, Huyền công 64, Độc công 64, Ngoại công 824, Ngoại thủ 804, Nội công 824, Nội thủ 804, Chính xác 432, Né tránh 144, Hội công 10, Cường lực 45, Linh khí 45, Thể lực 43, Định lực 43, Thân pháp 23, Hội thủ 8, Tất cả thuộc tính 8 |
| 15. Tụ Nghĩa Lệnh | Tụ Nghĩa Lệnh | `10158003` | 80 | `4001` | — |
| 16. Hào Hiệp Ấn | Chu Tước - Hào Hiệp Ấn Bát Cực | `10159024` | 85 | `2408` | — |

> **Lưu ý về phương pháp:** nhiều con số trong các bảng build của server khác (STT 01–19, 21–29) được trích bằng cách giải mã `ItemSegValue` mà không ghi rõ cột đọc `ValueSeg`; khi đối chiếu `tlbb129` thấy một số `Seg` và hệ số khác dữ liệu thực. Các mục khác **chưa được đối chiếu lại** ngày 07/11/2026 — xem mục “Phạm vi kiểm chứng” trong [DANH_SACH_SERVER_TLBB.md](DANH_SACH_SERVER_TLBB.md).

---

### Server 21: `TLBB PB14P2024` (Phiên bản 14 Môn Phái 2024 — Chân Trùng Lâu `Seg=4323` & Đồ Chế 150)

- **Đường dẫn:** Server `E:\games\TLBB\TLBB PB14P2024\tlbb` | Client `E:\games\TLBB\TLBB PB14P2024\Client14P2024`
- **Đặc trưng hệ thống:**
  - Bản **14 Môn Phái Hoàn Chỉnh năm 2024** (`tlbb_launch 1.4.11.12.exe`): Bao gồm 9 phái gốc + Mộ Dung, Đường Môn, Quỷ Cốc, Đào Hoa Đảo, Tuyệt Tình Cốc / Mạn Đà / Ác Nhân Cốc. `EquipBase.txt` lên tới **21.820 dòng**, `ItemSegValue.txt` 5.263 dòng, `GemInfo.txt` 2.726 dòng.
  - **Hệ thống Ngọc Cấp 9 Tinh Khiết & Minh Tinh Thạch Kép (`GemInfo.txt`):**
    - `Thuần Tịnh Hồng Tinh Thạch (Cấp 9)` (`50902006`) / `Thuần Tịnh Lục Tinh Thạch (Cấp 9)` (`50902008`): **Hỏa công +350** / **Độc công +350**.
    - `Minh Tinh Thạch Kép (Cấp 9·9)` (`50921309` / `50921409`): **Hỏa/Độc công +350 & Giảm kháng Hỏa/Độc +150**.
    - `Hồng Bảo Thạch (Cấp 9)` (`50913004`): **Thể lực +342** | `Huyết Tinh Thạch (Cấp 9)` (`50913006`): **HP +22.053** | `Tử Ngọc (Cấp 9)` (`50903001`): **Chính xác +3.955**.
  - **Hệ thống Trang bị Đỉnh Nhất Phái Tiêu Dao:**
    - **5 Món `Chân·Trùng Lâu` (`10553103..10553111`, Lv80, 9★):**
      - **`Chân·Trùng Lâu Liên` (`10553103`)**, **`Chân·Trùng Lâu Giới` (`10553104`)**, **`Chân·Trùng Lâu Ngọc` (`10553105`)** (`Seg=4323`, 12 dòng, Set `182`): Mỗi món cộng **HP +24.742, MP +11.996, Băng/Hỏa/Huyền/Độc công +221, Chính xác +2.460, Né tránh +820, Hội công +20, Thể lực +228, Trí lực +228, Thân pháp +124, Tất cả thuộc tính +42**!
      - **`Chân·Trùng Lâu Kiên` (`10553109`)** & **`Chân·Trùng Lâu Giáp` (`10553111`)** (`Seg=4324`, 9 dòng, Set `234`): Mỗi món cộng **HP +24.742, Ngoại/Nội thủ +2.202, Né tránh +820, Thể lực +228, Trí lực +228, Thân pháp +124, Hội thủ +20, Tất cả thuộc tính +42**!
    - **Bộ 3 Món `Trùng Lâu·Phản / Triệu / Quy` (`10159133..10159135`, Lv80, 9★, `Seg=100`, Set `182`):** Đeo ở ô Nhẫn #2 và Hộ Phù #2, mỗi món cộng **HP +12.520, Băng/Hỏa/Huyền/Độc công +100, Chính xác +1.230, Né tránh +410, Hội công +10, Thể lực +114, Trí lực +114, Thân pháp +62, Tất cả thuộc tính +21**!
    - **Hệ Đồ Chế 150 (`150铸造`, `Seg=1142..1150`, 8★):** Lấp đầy các vị trí Vũ khí (`10204031`), Mão (`10210061`), Hài (`10211061`), Thủ (`10212061`), Uyển (`10214041`), Yêu đai (`10221041`) với **Băng/Huyền/Độc công +120, Thể lực +161..169, Thân pháp +117..129, Tất cả thuộc tính +39..42**.

#### Bảng Build 17 Vị Trí Trang Bị Mạnh Nhất — Server 21 (`TLBB PB14P2024`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Quạt Chế 150 (`150铸造`)** *(hoặc TK 102 **Phù Sinh** `10304435`)* | `10204031` `10304435` | 150 / 102 | 8★ | Nội công CB: **1.310** / **1.150** \| Impact `9281` / `9558` | **Quạt 150 (`Seg=1150`, 12 dòng):** MP **+9.536**, **Băng/Huyền/Độc công +120**, Ngoại/Nội công **+1.638**, Chính xác **+1.830**, Hội công **+20**, Cường/Nội **+170**, Thể/Trí **+169**, Thân pháp **+129**, Tất cả **+42** |
| **2. Mão** | **Mão Chế 150 (`150铸造`)** | `10210061` | 150 | 8★ | Ngoại/Nội thủ CB: **733** | **9 dòng (`Seg=1142`):** Kháng 4 hệ **+35**, HP **+17.569**, MP **+8.953**, Ngoại/Nội thủ **+1.547**, Né tránh **+577**, Thể/Trí/Nội **+161**, Thân pháp **+117**, Tất cả **+39** |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7511` | **9 dòng (`Seg=4324`, Set 234):** **HP +24.742**, MP **+11.996**, Ngoại/Nội thủ **+2.202**, Né tránh **+820**, **Thể lực +228**, Trí lực **+228**, **Thân pháp +124**, Hội thủ **+20**, **Tất cả +42** |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10553109` | 80 | **9★** | Ngoại/Nội thủ CB: **1.608** \| Impact `7514` | **9 dòng (`Seg=4324`, Set 234):** **HP +24.742**, MP **+11.996**, Ngoại/Nội thủ **+2.202**, Né tránh **+820**, **Thể lực +228**, Trí lực **+228**, **Thân pháp +124**, Hội thủ **+20**, **Tất cả +42** |
| **5. Hộ uyển** | **Hộ Uyển Chế 150 (`150铸造`)** | `10214041` | 150 | 8★ | Chính xác CB: **756** | **7 dòng (`Seg=1146`):** Ngoại/Nội công **+1.596**, Chính xác **+1.782**, Hội công **+19**, Cường/Nội **+166**, Thể/Trí **+165**, Thân pháp **+123**, Tất cả **+40** |
| **6. Hộ thủ** | **Hộ Thủ Chế 150 (`150铸造`)** | `10212061` | 150 | 8★ | Ngoại/Nội thủ CB: **753** | **7 dòng (`Seg=1144`):** HP **+18.049**, MP **+9.192**, Ngoại/Nội thủ **+1.590**, Né tránh **+592**, Thể/Trí/Nội **+165**, Thân pháp **+123**, Tất cả **+40** |
| **7. Yêu đai** | **Yêu Đai Chế 150 (`150铸造`)** | `10221041` | 150 | 8★ | Ngoại/Nội thủ CB: **634** | **9 dòng (`Seg=1142`):** Kháng 4 hệ **+35**, HP **+17.569**, MP **+8.953**, Ngoại/Nội thủ **+1.547**, Né tránh **+577**, Thể/Trí/Nội **+161**, Thân pháp **+117**, Tất cả **+39** |
| **8. Hài** | **Hài Chế 150 (`150铸造`)** | `10211061` | 150 | 8★ | Né tránh CB: **247** | **7 dòng (`Seg=1143`):** HP **+17.809**, MP **+9.072**, Ngoại/Nội thủ **+1.569**, Né tránh **+584**, Thể/Trí/Nội **+163**, Thân pháp **+120**, Tất cả **+40** |
| **9. Hạng liên** | **Chân·Trùng Lâu Liên** | `10553103` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7508` | **12 dòng (`Seg=4323`, Set 182):** **HP +24.742**, MP **+11.996**, **Băng/Hỏa/Huyền/Độc công +221**, Chính xác **+2.460**, Né tránh **+820**, Hội công **+20**, **Thể/Trí +228**, Thân pháp **+124**, Tất cả **+42** |
| **10. Giới chỉ 1 & 2** | **Chân·Trùng Lâu Giới** (`10553104`) & **Trùng Lâu·Triệu** (`10159134`) | `10553104` `10159134` | 80 | **9★** | Nội công CB: **1.662** & **1.155** \| Impact `7507` & `7519` | **Chân·Giới (`Seg=4323`):** **HP +24.742**, **4 hệ công +221**, CX **+2.460**, Hội **+20**, Thể/Trí **+228**, Tất cả **+42**. **Trùng Lâu·Triệu (`Seg=100`):** HP **+12.520**, **4 hệ công +100**, Thể/Trí **+114**, Tất cả **+21** |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** (`10553105`) & **Trùng Lâu·Quy** (`10159135`) | `10553105` `10159135` | 80 | **9★** | Nội thủ CB: **1.466** & **1.019** \| Impact `7503` & `7520` | **Chân·Ngọc (`Seg=4323`):** **HP +24.742**, **4 hệ công +221**, CX **+2.460**, Hội **+20**, Thể/Trí **+228**, Tất cả **+42**. **Trùng Lâu·Quy (`Seg=100`):** HP **+12.520**, **4 hệ công +100**, Thể/Trí **+114**, Tất cả **+21** |
| **12–17. Phụ kiện** | **Kim Sí Linh Vũ** (`10155008`), **Ngự Dao Bàn** (`10156002`), **Long Văn** (`10157001`), **Tụ Nghĩa Lệnh** (`10158003`, `Seg=4001`), **Chu Tước Hào Hiệp Ấn·Bát Cực** (`10159024`, `Seg=2408`) | — | 80–100 | 5★–9★ | Đầy đủ Chân Nguyên, Tinh Thông, Điêu Văn, Thần Đỉnh, Con Cái 14 Phái | Kích đồng thời Set 182 (6 món Trùng Lâu Công) và Set 234 (2 món Trùng Lâu Thủ) |

---

### Server 22 (`4.TLBB HJ64`), Server 23 (`TLBB-HJ-10P-64bit`) & Server 24 (`tlbb2.7-vh`) — Nhánh Hoài Cựu 64-bit v2.7 (10/2024)

- **Đường dẫn:**
  - **STT 22 (`4.TLBB HJ64`):** Server `E:\games\TLBB\4.TLBB HJ64\3. Source\1. tlbb1` | Client `E:\games\TLBB\4.TLBB HJ64\1. Client LL 2.7`
  - **STT 23 (`TLBB-HJ-10P-64bit`):** Server `E:\games\TLBB\TLBB-HJ-10P-64bit\tlbb\tlbb1` | Client `E:\games\TLBB\TLBB-HJ-10P-64bit\CLIENT`
  - **STT 24 (`tlbb2.7-vh`):** Server `E:\games\TLBB\tlbb2.7-vh\tlbb2.7-24.10\home\tlbb1` | Client `E:\games\TLBB\tlbb2.7-vh\CLIENT`
- **Đặc trưng hệ thống:**
  - Ba server này đại diện cho thế hệ **Hoài Cựu 64-bit v2.7** (STT 22 là bản 9 phái Lưu Quang, STT 23 mở rộng lên 10 phái có Mộ Dung, STT 24 là bản Việt hóa 24.10).
  - Trong nhánh v2.7 64-bit:
    1. Toàn bộ **5 vị trí Trùng Lâu kinh điển** đều có bản **`Chân - Trùng Lâu` (`Chân-Trùng Lâu Giáp` `10413104`, `Chân-Trùng Lâu Kiên` `10415056`, `Chân-Trùng Lâu Liên` `10420089`, `Chân-Trùng Lâu Giới` `10422149`, `Chân-Trùng Lâu Ngọc` `10423025`)** đạt 9★, 9 dòng (`Seg=100`).
    2. Ngoài `Vong Xuyên Phiến` Lv120 (`10204027`), trong `EquipBase.txt` có mở thêm **`Phiến Đúc Cấp 140` (`10204029`, Lv140, `Seg=399`: Nội công CB 1.224, 4 hệ +120, Nội công +1.532, Chính xác +1.710, Thể/Nội/Trí +158)** và ở STT 23 có **`150 Chú Tạo` (`10204030`, Lv150, `Seg=400`: Nội công CB 1.310, 4 hệ +120, Nội công +1.638, Chính xác +1.830, Thể/Nội/Trí +170)**.
    3. Sở hữu hệ thống **Thiên Giám 3 Món (`Lưu Quang / Mộ Vũ / Tinh Huy Thiên Giám`, Slot 19–21, `10157000..10157014`)** và hệ thống **Thú Hồn (`PetSoul`) / Võ Ý (`Wuyi`)**.

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 22 (`4.TLBB HJ64`), Server 23 (`TLBB-HJ-10P-64bit`) & Server 24 (`tlbb2.7-vh`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Đại Tống Quân Sầm** *(hoặc **Phiến Đúc 140/150** `10204029`/`10204030`)* | `10304106` `10204029` | 102 / 140 | 7★ / 8★ | Nội công CB: **1.050** / **1.224** \| Impact `9359` | **Đại Tống Quân Sầm (`Seg=385`):** HP **+14%**, Hỏa/Độc công **+113**, **Giảm kháng Hỏa/Độc +41**, Nội công **+1.439**, Chính xác **+1.093**. **Phiến 140 (`Seg=399`):** 4 hệ **+120**, Nội công **+1.532**, Chính xác **+1.710**, Thể/Trí/Nội **+158** |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Chân - Trùng Lâu Giáp** | `10413104` | 90 | **9★** | Ngoại/Nội thủ CB: **1.258** \| Impact `9406` | **9 dòng (`Seg=100`):** HP **+12.520**, Kháng 4 hệ **+8**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **4. Hộ kiên** | **Chân - Trùng Lâu Kiên** | `10415056` | 90 | **9★** | Ngoại/Nội thủ CB: **907** \| Impact `9404` | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại/Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **5. Hộ uyển** | **Quy Túc** *(hoặc **Tịch Diệt Ảo Phật Hoàn** `10514090`)* | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** **Băng/Hỏa/Huyền/Độc công +117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, MP **+7.396**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Trích Tiên Đái** *(hoặc **Tịch Diệt Ảo Phật Yêu Đai** `10521090`)* | `10221027` | 112 | 8★ | Ngoại/Nội thủ CB: **511** | **7 dòng (`Seg=318`):** Kháng 4 hệ **+35**, HP **+13.969**, MP **+7.157**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **8. Hài** | **Phệ Lân Thánh Lý** | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, MP **+7.276**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Chân - Trùng Lâu Liên** *(hoặc **Khấp Lộ Liên** `10220027` +119 4 hệ)* | `10420089` | 90 | **9★** | HP/MP CB: **8.040** \| Impact `9408` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **10. Giới chỉ 1 & 2** | **Chân - Trùng Lâu Giới** *(x2)* *(hoặc phối **Cát Tường Dạ** `10222027`)* | `10422149` `10222027` | 90 / 115 | **9★** / 8★ | Nội công CB: **1.155** \| Impact `9409` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** *(Kèm Phong huyệt 15s)* |
| **11. Hộ phù 1 & 2** | **Chân - Trùng Lâu Ngọc** *(x2)* *(hoặc phối **Ưng Tường Văn Chương** `10223027`)* | `10423025` `10223027` | 90 / 115 | **9★** / 8★ | Nội thủ CB: **1.019** \| Impact `9402` | **9 dòng (`Seg=100`):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể/Trí **+114**, Thân pháp **+62**, Tất cả **+21** *(Kèm Tản công)* |
| **12. Võ hồn (Slot 10)** | **Ngự Dao Bàn** | `10156002` | 1 | 5★ | Ngoại/Nội công + Thủ CB | Đục 4 lỗ ngọc C9, lĩnh ngộ kỹ năng Hỏa/Độc + Giảm kháng |
| **13. Ám khí (Slot 17)** | **Băng Phách Thần Châm** | `10155002` | 70 | 7★ | Ngoại/Nội công CB: **481** | Đục 4 lỗ ngọc C9, tẩy luyện phẩm chất & hiệu ứng |
| **14–16. Thiên giám (Slot 19, 20, 21)** | **Tinh Huy Thiên Giám Minh Kính / Song Thứ / Tỏa Giáp** | `10157012` `10157013` `10157014` | 85 | 7★ | Bộ 3 món **Thiên Giám Cấp Cao** | Tăng mạnh **Thuộc tính công + Giảm kháng + Giới hạn Sinh lực + Võ Ý** |

---

### Server 25: `tlbb-10p-vogioi` (Hoài Cựu 10 Phái Vô Giới — 10/2024)

- **Đường dẫn:** Server `E:\games\TLBB\tlbb-10p-vogioi\tlbb1\tlbb1` | Client `E:\games\TLBB\tlbb-10p-vogioi\CLIENT`
- **Đặc trưng hệ thống (Điểm đột phá cực kỳ thú vị khi giải mã `ItemSegValue.txt`):**
  - Mặc dù bề ngoài giống nhánh Hoài Cựu 10 phái (STT 23), nhưng trong `tlbb-10p-vogioi`, đội ngũ phát triển đã **nâng cấp toàn diện `SegID=100` trong `ItemSegValue.txt`** và mở rộng **5 món `Trùng Lâu` gốc (`10413102`, `10415055`, `10420088`, `10422016`, `10423024`) lên 13 dòng thuộc tính**!
  - **Sự đảo ngược thú vị giữa `Trùng Lâu` thường và `Chân - Trùng Lâu` ở Server 25:**
    - Trong `tlbb-10p-vogioi`, các món **`Trùng Lâu Liên` (`10420088`), `Trùng Lâu Giới` (`10422016`), `Trùng Lâu Ngọc` (`10423024`)** có **13 dòng** (`Seg=100`): cộng **HP +32.221, Băng/Hỏa/Huyền/Độc công +110, Chính xác +2.341, Né tránh +755, Hội công +18, Thể lực +175, Trí lực +175, Thân pháp +122, Tất cả thuộc tính +38**!
    - Trong khi đó, các món `Chân - Trùng Lâu Liên / Giới / Ngọc` (`10420089`, `10422149`, `10423025`) lại chỉ bật 9 trọng số dòng (thiếu 4 dòng thuộc tính Băng/Hỏa/Huyền/Độc +110)! Vì vậy ở Server 25, **đeo `Trùng Lâu Liên / Giới / Ngọc` 13 dòng mạnh hơn hẳn `Chân - Trùng Lâu` 9 dòng**!
    - Tương tự, **Bộ `Tịch Diệt Ảo Phật` (`10514090`, `10521090`)** cũng trỏ vào `Seg=100` nên chiếc Yêu Đai `Tịch Diệt Ảo Phật Yêu Đai` (`10521090`) cộng tới **HP +32.221, Ngoại/Nội thủ +1.955, Thể lực +175, Thân pháp +122, Tất cả +38**!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 25 (`tlbb-10p-vogioi`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Đại Tống Quân Sầm** *(hoặc **Phiến 150** `10204030`)* | `10304106` `10204030` | 102 / 150 | 7★ / 8★ | Nội công CB: **1.050** / **1.310** \| Impact `9359` | **Đại Tống Quân Sầm (`Seg=385`):** HP **+14%**, Hỏa/Độc công **+113**, **Giảm kháng Hỏa/Độc +41**, Nội công **+1.439**, Chính xác **+1.093** |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Trùng Lâu Giáp** *(13 dòng — mạnh hơn Chân)* | `10413102` | 90 | **9★** | Ngoại/Nội thủ CB: **1.258** \| Impact `9405` | **13 dòng (`Seg=100`):** **HP +32.221**, Kháng Băng/Hỏa/Huyền/Độc **+11**, **Ngoại/Nội thủ +1.955**, **Né tránh +755**, **Thể lực +175**, Trí lực **+175**, **Thân pháp +122**, **Hội thủ +20**, **Tất cả thuộc tính +38** |
| **4. Hộ kiên** | **Trùng Lâu Kiên** *(13 dòng — mạnh hơn Chân)* | `10415055` | 90 | **9★** | Ngoại/Nội thủ CB: **907** \| Impact `9403` | **13 dòng (`Seg=100`):** **HP +32.221**, Kháng Băng/Hỏa/Huyền/Độc **+11**, **Ngoại/Nội thủ +1.955**, **Né tránh +755**, **Thể lực +175**, Trí lực **+175**, **Thân pháp +122**, **Hội thủ +20**, **Tất cả thuộc tính +38** |
| **5. Hộ uyển** | **Tịch Diệt Ảo Phật Hoàn** *(hoặc **Quy Túc** `10214027` +117 4 hệ)* | `10514090` `10214027` | 96 / 116 | 7★ / 8★ | Chính xác CB: **509** / **610** | **Tịch Diệt (`Seg=100`):** Ngoại/Nội công **+1.946**, **Chính xác +2.341**, **Hội công +18**, **Thể/Trí +175**, **Thân pháp +122**. **Quy Túc (`Seg=313`):** **4 hệ công +117**, Chính xác **+1.422**, Thể/Nội **+132** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Tịch Diệt Ảo Phật Yêu Đai** | `10521090` | 94 | 7★ | Ngoại/Nội thủ CB: **423** | **6 dòng Siêu Cường (`Seg=100`):** **HP +32.221**, **Ngoại/Nội thủ +1.955**, **Thể lực +175**, **Thân pháp +122**, **Tất cả thuộc tính +38** *(Hưởng trọn buff `Seg=100`!)* |
| **8. Hài** | **Phệ Lân Thánh Lý** | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Trùng Lâu Liên** *(13 dòng — có 4 hệ +110)* | `10420088` | 90 | **9★** | HP/MP CB: **8.040** \| Impact `9407` | **13 dòng (`Seg=100`):** **HP +32.221**, MP **+6.441**, **Băng/Hỏa/Huyền/Độc công +110**, **Chính xác +2.341**, **Né tránh +755**, **Hội công +18**, **Thể lực +175**, Trí lực **+175**, **Thân pháp +122**, **Tất cả +38** |
| **10. Giới chỉ 1 & 2** | **Trùng Lâu Giới** *(x2 — 13 dòng, có 4 hệ +110)* | `10422016` | 90 | **9★** | Nội công CB: **1.155** \| Impact `9401` | **13 dòng (`Seg=100`):** **HP +32.221**, MP **+6.441**, **Băng/Hỏa/Huyền/Độc công +110**, **Chính xác +2.341**, **Né tránh +755**, **Hội công +18**, **Thể lực +175**, Trí lực **+175**, **Thân pháp +122**, **Tất cả +38** |
| **11. Hộ phù 1 & 2** | **Trùng Lâu Ngọc** *(x2 — 13 dòng, có 4 hệ +110)* | `10423024` | 90 | **9★** | Nội thủ CB: **1.019** \| Impact `9400` | **13 dòng (`Seg=100`):** **HP +32.221**, MP **+6.441**, **Băng/Hỏa/Huyền/Độc công +110**, **Chính xác +2.341**, **Né tránh +755**, **Hội công +18**, **Thể lực +175**, Trí lực **+175**, **Thân pháp +122**, **Tất cả +38** |
| **12–16. Phụ kiện** | **Ngự Dao Bàn** (`10156002`), **Băng Phách Thần Châm** (`10155002`), **Bộ 3 Món Tinh Huy Thiên Giám** (`10157012..10157014`) | — | 70–85 | 5★–7★ | Đầy đủ Võ Hồn, Ám Khí, Thiên Giám, Võ Ý, Thú Hồn | Cộng hưởng trực tiếp với lượng máu nền khổng lồ (+32.221 HP/món) từ dàn Trùng Lâu `Seg=100` |

---

### Server 26: `tlbb-origin-skyway` (Phiên bản 2010 Origin Skyway — 2024–2025)

- **Đường dẫn:** Server `E:\games\tlbb_draft\tlbb-origin-skyway\tlbb` | Client `E:\games\tlbb_draft\tlbb-origin-skyway\Client`
- **Đặc trưng hệ thống:**
  - Thuộc nhánh **Hoài Cựu `0.05.3334`** (cùng họ với STT 16 & STT 17), nhưng trong `tlbb-origin-skyway`, cây quạt chế 100 **`Ngoạ Long` (`10204010`)** được gán thẳng vào **`Seg=100` (10 dòng: Băng/Hỏa/Huyền/Độc công +83, Ngoại/Nội công +1.106, Chính xác +1.230, Hội công +10, Cường/Nội lực +115, Thể/Trí lực +114, Thân pháp +62, Tất cả +21)** bên cạnh Thần Khí 102 `Đại Tống Quân Sầm` (`10304111`) và Quạt 120 `Vong Xuyên Phiến` (`10204027`).
  - Hai món **`Chân-Trùng Lâu Liên` (`10420089`)** và **`Chân Trùng Lâu Ngọc` (`10423025`)** giữ nguyên cấu trúc **13 dòng (`Seg=100`, +83 cả 4 thuộc tính Băng/Hỏa/Huyền/Độc)**, kết hợp **Bộ 3 Món `Thiên Giám` (`10157000..10157008`)**.
  - *(Bảng trang bị 16 vị trí của Server 26 giống hệt **Bảng Build Server 16 & 17** ở trên, chỉ bổ sung thêm tùy chọn Quạt `Ngoạ Long` `10204010` `Seg=100` +83 4 hệ).*

---

### Server 27: `Skynet_bot_ai` (Hoài Cựu 64-bit Tích Hợp Bot AI & Chí Tôn Trùng Lâu — 04/11/2026)

- **Đường dẫn:** Server `E:\games\TLBB\Skynet_bot_ai\ubuntu` | Client `E:\games\TLBB\Skynet_bot_ai\CLIENT`
- **Đặc trưng hệ thống:**
  - Bản **Hoài Cựu 64-bit Custom AI Bot** (đã đối chiếu chi tiết trong phiên làm việc trước):
  - **Hệ thống Ngọc C9 được Buff vượt trội:**
    - `Lục Minh Thạch (Cấp 9)` (`50921002`/`004`): **Giảm kháng Hỏa / Độc +220** (cao hơn +70 điểm/viên so với bản chuẩn +150!).
    - `Hồng Bảo Thạch / Lục Bảo Thạch (Cấp 9)` (`50913004`/`50912005`): **Thể lực / Thân pháp +300** (bản chuẩn +186 / +109).
    - `Biến Thạch (Cấp 9)` (`50904002`): **Hội công +45** (bản chuẩn +28).
  - **Hệ thống Trang bị Độc Quyền (`Chân·Chí Tôn Trùng Lâu` & `Thần Khí Thần Thoại`):**
    - Vũ khí: **`Đại Tống Quân Sầm · Thần Thoại` (`10563207`, Lv102, 9★, 16 dòng, `Seg=178`, Nội công CB 3.969)**!
    - Phòng cụ: **`Chí Tôn·Trùng Lâu Giáp` (`10413106`, 14 dòng, `Seg=158`)**, **`Chí Tôn·Trùng Lâu Kiên` (`10415059`, 14 dòng, `Seg=158`)**, **`Chí Tôn·Trùng Lâu Đới` (`10421091`, 14 dòng, `Seg=159`, cộng +16% HP tối đa)**!
    - Trang sức công: **`Chân·Chí Tôn Trùng Lâu Liên` (`10420096`)**, **`Chân·Chí Tôn Trùng Lâu Giới` (`10422159`)**, **`Chân·Chí Tôn Trùng Lâu Ngọc` (`10423068`)** — cả 3 đều là **9★, 16 dòng (`Seg=161`)**, mỗi món cộng **HP +18.887, Băng/Hỏa/Huyền/Độc công +326, Chính xác +3.476, Hội công +26, Thể/Trí +210, Thân pháp +124, Tất cả thuộc tính +54**!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 27 (`Skynet_bot_ai`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Đại Tống Quân Sầm · Thần Thoại** *(hoặc `Đại Tống Quân Sầm · Truyền Thuyết` `10563107`)* | `10563207` | 102 | **9★** | Nội công CB: **3.969** \| Impact `9359` | **16 dòng (`Seg=178`):** Đỉnh cao Thần Khí 102 bậc Thần Thoại — cộng **HP% + 4 hệ Băng/Hỏa/Huyền/Độc công + Giảm kháng 4 hệ + Nội công + Chính xác + Hội công + 5 dòng tiềm năng + Tất cả thuộc tính** |
| **2. Mão** | **Phệ Lân Khuynh Đô Quán** *(hoặc **Lân Mộ Xích Quán** `10210039`)* | `10210046` | 112 | 8★ | Ngoại/Nội thủ CB: **582** | **7 dòng (`Seg=285`):** Kháng 4 hệ **+35**, HP **+13.969**, Ngoại/Nội thủ **+1.228**, Né tránh **+458**, Thể/Trí/Nội lực **+127**, Thân pháp **+69**, Tất cả thuộc tính **+24** |
| **3. Y phục** | **Chí Tôn·Trùng Lâu Giáp** | `10413106` | 85 | **9★** | Ngoại/Nội thủ CB: **1.886** \| Impact `9406` | **14 dòng (`Seg=158`):** HP **+20.647**, Kháng Băng/Hỏa/Huyền/Độc **+50**, Ngoại/Nội thủ **+1.656**, Né tránh **+1.108**, Thể lực **+242**, Trí lực **+242**, Thân pháp **+173**, Hội thủ **+35**, Tất cả **+76** |
| **4. Hộ kiên** | **Chí Tôn·Trùng Lâu Kiên** | `10415059` | 85 | **9★** | Ngoại/Nội thủ CB: **1.360** \| Impact `9404` | **14 dòng (`Seg=158`):** HP **+20.647**, Kháng Băng/Hỏa/Huyền/Độc **+50**, Ngoại/Nội thủ **+1.656**, Né tránh **+1.108**, Thể lực **+242**, Trí lực **+242**, Thân pháp **+173**, Hội thủ **+35**, Tất cả **+76** |
| **5. Hộ uyển** | **Quy Túc** *(hoặc **Huyền Thế Ma Uyển** `10214020`)* | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** **Băng/Hỏa/Huyền/Độc công +117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Chí Tôn·Trùng Lâu Đới** | `10421091` | 85 | **9★** | Ngoại/Nội thủ CB: **1.195** \| Impact `9415` | **14 dòng (`Seg=159`):** **HP tối đa +16%**, Kháng 4 hệ **+50**, Ngoại/Nội thủ **+1.656**, Né tránh **+1.108**, Hội công **+33**, Thể lực **+242**, Trí lực **+242**, Thân pháp **+173**, Hội thủ **+35**, Tất cả **+76** |
| **8. Hài** | **Phệ Lân Thánh Lý** | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Chân·Chí Tôn Trùng Lâu Liên** | `10420096` | 85 | **9★** | HP/MP CB: **12.060** \| Impact `9408` | **16 dòng (`Seg=161`):** HP **+18.887**, MP **+10.287**, **Băng/Hỏa/Huyền/Độc công +326**, Ngoại/Nội công **+1.841**, Chính xác **+3.476**, Né tránh **+809**, Hội công **+26**, Thể/Trí **+210**, Thân pháp **+124**, Tất cả **+54** |
| **10. Giới chỉ 1 & 2** | **Chân·Chí Tôn Trùng Lâu Giới** *(x2 hoặc phối `Chí Tôn Giới` `10422157`)* | `10422159` `10422157` | 85 | **9★** | Nội công CB: **1.732** \| Impact `9409` | **Chân·Chí Tôn (`Seg=161`, 16 dòng):** HP **+18.887**, **Băng/Hỏa/Huyền/Độc công +326**, Ngoại/Nội công **+1.841**, Chính xác **+3.476**, Hội công **+26**, Thể/Trí **+210**, Thân pháp **+124**, Tất cả **+54** *(Bản `10422157` `Seg=160`: 4 hệ **+215**)* |
| **11. Hộ phù 1 & 2** | **Chân·Chí Tôn Trùng Lâu Ngọc** *(x2 hoặc phối `Chí Tôn Ngọc` `10423067`)* | `10423068` `10423067` | 85 | **9★** | Nội thủ CB: **1.528** \| Impact `9402` | **Chân·Chí Tôn (`Seg=161`, 16 dòng):** HP **+18.887**, **Băng/Hỏa/Huyền/Độc công +326**, Ngoại/Nội công **+1.841**, Chính xác **+3.476**, Hội công **+26**, Thể/Trí **+210**, Thân pháp **+124**, Tất cả **+54** |
| **12–16. Phụ kiện** | **Ngự Dao Bàn** (`10156002`), **Băng Phách Thần Châm** (`10155002`), **Bộ 3 Món Tinh Huy Thiên Giám** (`10157012..10157014`) | — | 70–85 | 5★–7★ | Khảm full dàn Ngọc C9 Buff (+220 Giảm kháng, +300 Thể lực) | Tạo nên bản Tiêu Dao Hoài Cựu có lượng Giảm kháng và Tứ Tu cao nhất nhánh 64-bit |

---

### Server 28: `Skynet-bot` (Phiên bản ChangYou 3.72 Tích Hợp Bot — 06/11/2026)

- **Đường dẫn:** Server `E:\games\TLBB\Skynet-bot\tlbb` | Client `E:\games\TLBB\Skynet-bot\CLIENT`
- **Đặc trưng hệ thống:**
  - Cùng kiến trúc **ChangYou 3.72 (11 phái)** với `STT 20 (tlbb129)` nhưng có **điểm nâng cấp cực giá trị ở cặp Nhẫn #2 và Hộ Phù #2**:
    - Trong `Skynet-bot`, hai món **`Chân·Trùng Lâu Giới` (`10422149`)** và **`Chân·Trùng Lâu Ngọc` (`10423025`)** (mã ID nhánh `1042x`) được mở thành **13 dòng (`Seg=100`)**: cộng **HP +12.520, Băng/Hỏa/Huyền/Độc công +105, Chính xác +1.230, Né tránh +410, Hội công +45, Thể lực +114, Trí lực +114, Thân pháp +114, Tất cả thuộc tính +68**!
    - Nhờ vậy, Tiêu Dao ở `Skynet-bot` đeo **1 cặp `Chân·Trùng Lâu Giới / Ngọc` (`10553104` / `10553105`, `Seg=5010`, +8% HP, +215 4 hệ, kích Set 183)** ở ô Nhẫn #1 / Phù #1 VÀ đeo **1 cặp `Chân·Trùng Lâu Giới / Ngọc` (`10422149` / `10423025`, `Seg=100`, +105 4 hệ, +45 Hội công, +68 Tất cả)** ở ô Nhẫn #2 / Phù #2 — **không bị thọt bất kỳ dòng thuộc tính nào ở cả 4 ô Nhẫn/Phù**!

#### Bảng Build 16 Vị Trí Trang Bị Mạnh Nhất — Server 28 (`Skynet-bot`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Phù Sinh** / **Phù Sinh Quan Khích** | `10304435` `10317041` | 102 | 8★ | Nội công CB: **1.150** \| Impact `8125` / `8118` | **10 dòng (`Seg=389`):** HP tối đa **+15%**, **Băng/Hỏa/Huyền/Độc công +208**, **Giảm kháng 4 hệ +40**, Nội công **+2.596**, Chính xác **+1.944**, Hội công **+17**, Thể lực **+194**, Thân pháp **+104** |
| **2. Mão** | **Siêu Cấp Tai Thỏ** | `10410035` | 1 | **9★** | Ngoại/Nội thủ CB: **1.545** \| Impact `10663` | **9 dòng (`Seg=5032`):** **HP tối đa +17%**, **Băng/Hỏa/Huyền/Độc công +215**, Kháng 4 hệ **+55**, Thể lực **+368**, Thân pháp **+368**, Tất cả thuộc tính **+188** |
| **3. Y phục** | **Chân·Trùng Lâu Giáp** | `10553111` | 80 | **9★** | Ngoại/Nội thủ CB: **2.227** \| Impact `7511` | **9 dòng (`Seg=5009`, Set 182):** HP **+22.000**, **HP tối đa +8%**, Kháng 4 hệ **+35**, Né tránh **+620**, Thể lực **+288**, Trí lực **+168**, Thân pháp **+128**, Hội thủ **+18**, Tất cả **+88** |
| **4. Hộ kiên** | **Chân·Trùng Lâu Kiên** | `10553109` | 80 | **9★** | Ngoại/Nội thủ CB: **1.608** \| Impact `7514` | **9 dòng (`Seg=5006`, Set 182):** **HP tối đa +8%**, Ngoại/Nội thủ **+1.825**, Né tránh **+620**, Thể lực **+288**, Trí lực **+168**, Thân pháp **+128**, Hội thủ **+18**, Tất cả **+88** |
| **5. Hộ uyển** | **Quy Túc** *(Đồ Chế 116–126)* | `10214027` | 116 | 8★ | Chính xác CB: **610** | **7 dòng (`Seg=313`):** **Băng/Hỏa/Huyền/Độc công +117**, Ngoại/Nội công **+1.277**, Chính xác **+1.422**, Hội công **+12**, Cường/Nội/Thể/Trí **+132**, Thân pháp **+71**, Tất cả **+25** |
| **6. Hộ thủ** | **Phệ Lân Huyền Thủ** *(Đồ Chế 116–126)* | `10212046` | 116 | 8★ | Ngoại/Nội thủ CB: **601** | **7 dòng (`Seg=287`):** HP **+14.449**, Ngoại/Nội thủ **+1.271**, Né tránh **+474**, Thể/Trí/Nội lực **+132**, Thân pháp **+71**, Tất cả thuộc tính **+25** |
| **7. Yêu đai** | **Chân·Trùng Lâu Đới** | `10553113` | 80 | **9★** | Ngoại/Nội thủ CB: **1.500** \| Impact `7513` | **9 dòng (`Seg=5008`, Set 182):** **HP tối đa +11%**, Ngoại/Nội thủ **+1.825**, Né tránh **+620**, Thể lực **+288**, Trí lực **+168**, Thân pháp **+128**, Hội thủ **+18**, Tất cả **+88** *(Kích Set 182)* |
| **8. Hài** | **Phệ Lân Thánh Lý** *(Đồ Chế 114–124)* | `10211046` | 114 | 8★ | Né tránh CB: **196** | **7 dòng (`Seg=286`):** HP **+14.209**, Ngoại/Nội thủ **+1.250**, Né tránh **+466**, Thể/Trí/Nội lực **+129**, Thân pháp **+70**, Tất cả thuộc tính **+24** |
| **9. Hạng liên** | **Chân·Trùng Lâu Liên** | `10553103` | 80 | **9★** | HP/MP CB: **8.040** \| Impact `7508` | **12 dòng (`Seg=5010`, Set 183):** **HP tối đa +8%**, **Băng/Hỏa/Huyền/Độc công +215**, Chính xác **+1.600**, Né tránh **+480**, Hội công **+20**, Thể lực **+288**, Trí lực **+168**, Thân pháp **+128**, Tất cả **+88** |
| **10. Giới chỉ 1 & 2** | **Chân·Trùng Lâu Giới** (`10553104`) & **Chân·Trùng Lâu Giới** (`10422149`) | `10553104` `10422149` | 80 / 90 | **9★** | Nội công CB: **1.662** & **1.155** \| Impact `7507` & `7509` | **Giới #1 (`10553104`, `Seg=5010`):** **HP +8%**, **4 hệ công +215**, Thể lực **+288**, Tất cả **+88**. **Giới #2 (`10422149`, `Seg=100`, 13 dòng):** HP **+12.520**, **4 hệ công +105**, Hội công **+45**, Thể/Trí/Thân **+114**, Tất cả **+68** |
| **11. Hộ phù 1 & 2** | **Chân·Trùng Lâu Ngọc** (`10553105`) & **Chân·Trùng Lâu Ngọc** (`10423025`) | `10553105` `10423025` | 80 / 90 | **9★** | Nội thủ CB: **1.466** & **1.019** \| Impact `7503` & `7505` | **Ngọc #1 (`10553105`, `Seg=5010`):** **HP +8%**, **4 hệ công +215**, Thể lực **+288**, Tất cả **+88** *(Kích Set 183)*. **Ngọc #2 (`10423025`, `Seg=100`, 13 dòng):** HP **+12.520**, **4 hệ công +105**, Hội công **+45**, Thể/Trí/Thân **+114**, Tất cả **+68** |
| **12–16. Phụ kiện** | **Ngự Dao Bàn** (`10156002`), **Kim Sí Linh Vũ** (`10155008`), **Long Văn** (`10157001`), **Tụ Nghĩa Lệnh** (`10158003` +50% HP), **Chu Tước Hào Hiệp Ấn** (`10159024`) | — | 80–90 | 5★–8★ | Đầy đủ Chân Nguyên C9, Tinh Thông, Điêu Văn Kép, Con Cái, Thần Đỉnh | Song kiếm hợp bích 2 cặp Chân·Trùng Lâu Giới/Ngọc hoàn hảo nhất nhánh 3.72 |

---

### Server 29: `Level Up Games` (Thien Long Origin — ChangYou 3.72 Việt Hóa 2026)

- **Đường dẫn:** Server `E:\games\tlbb_lee\tlbb-docker\tlbbsrc1510\tlbbsrc` | Client `E:\games\TLBB\Level Up Games\Thien Long Origin`
- **Đặc trưng hệ thống:**
  - Bản thương mại Việt hóa hoàn chỉnh (*Thien Long Origin — Level Up Games*) phát triển trên nền **ChangYou 3.72 Mở Rộng** (cùng cấu trúc bảng `EquipBase.txt` và `ItemSegValue.txt` với **Server 19 `tlbb_lee`**).
  - Sở hữu đầy đủ **10 món `Chân-Trùng Lâu` (`10553103..10553118`)** trong đó **Uyển (`10553114`), Thủ (`10553115`), Ngoa (`10553116`)** đều cộng **Băng/Hỏa/Huyền/Độc công +320**, kết hợp **Mũ `Siêu Cấp Tai Thỏ` (`10410035`, HP +17%, 4 hệ +215, Thể/Thân +368, Tất cả +188)**, **Thần Khí Luyện Hồn 102 `Phù Sinh Quan Khích` (`10317041`)**, **`Tụ Nghĩa Lệnh` (`10158003`, HP +50%)** và **`Chu Tước Hào Hiệp Ấn - Bát Cực` (`10159024`)**.
  - *(Chi tiết từng vị trí trang bị, Item ID và chỉ số dòng của Server 29 đồng nhất 100% với **Bảng Build 16 Vị Trí của Server 19 `tlbb_lee`**).*

---

### Server 30: `TanThienLong` (Phiên Bản Tân Thiên Long 3D Chính Thức VNG — 15 Môn Phái 2026)

- **Đường dẫn:** Client `E:\games\TLBB\TanThienLong` | Server: Kết nối chính thức VNG (`ttl.zing.vn`) hoặc Máy chủ 3.7x
- **Đối chiếu lại 08/11/2026 (giải mã `Config.axp`, 701 bảng):** `Seg=100` = HP 12.520, MP 6.441, Né 410, Hội thủ 10, Thể/Trí 114, Thân 62, Tất cả 21 (**khớp** bảng bên dưới); `Seg=4151` Trường Tuyệt: HP 18%, 4 hệ công 169/dòng, Nội/Ngoại công 1.554, CX 1.960, Hội 17 (**khớp**); Set 182 `Trùng Lâu Thiên Thương` & 207 `Trùng Lâu Thiên Hựu` (3 món mỗi bộ, **khớp**). `Impact` chưa đối chiếu lại. **Không có server** — bảng này là chỉ số của *client* VNG. Ở bản VNG, dải `10553103..118` là ô trống và Set 183/208 mang tên khác (“Hải ngoại tân trình”/“Mai”), nên Set 183/208 của `tlbb129` là phần riêng của 3.72.
- **Đặc trưng hệ thống:**
  - **Client Chính Thức Bản Quyền VNG (Version `3.71.5720`)**: Hỗ trợ đồng thời cả 2 engine native 32-bit (`Bin\Game.exe`) và 64-bit (`Bin64\Game.exe`), sử dụng bảng mã `CodePage = 1258` (tiếng Việt chuẩn thương mại).
  - **Bản duy nhất trong toàn bộ 30 Server tích hợp 15 Môn Phái**: Bao gồm 9 phái cổ điển + **Mộ Dung, Đường Môn, Quỷ Cốc, Đào Hoa Đảo, Tuyệt Tình Cốc (`menpai14`), Ác Nhân Cốc (`menpai15`)**.
  - **Hệ thống Thần Khí 9★ `Trường Tuyệt` (`10317043..051`, Lv102, 9★, `Seg=4151`)**: Bước tiến hóa cao nhất của Thần Binh Hải Vực trên nền VNG, sở hữu **12 dòng chỉ số**: HP tối đa **+18%**, **Băng công +169 & Độc công +169** (hoặc Hỏa công), Nội công **+1.554**, Chính xác **+1.960**, Hội công **+17**, Thể lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21**.
  - **Trọn bộ 43 ID Trùng Lâu VNG Full 10 Vị Trí Kèm Hiệu Ứng Impact Cực Mạnh**: Toàn bộ trang bị giữ chuẩn 9 dòng `Seg=100` (`HP +12.520, Thể/Trí +114, Thân +62, Tất cả +21`) nhưng gắn kèm các bùa chú Impact đặc quyền không server lậu nào có (Chân-Khôi: **+10% Bạo kích**, Chân-Ngoa: **3% x5 Bạo kích**, Chân-Thủ: **+15% Sát thương**, Chân-Giáp: **+10% HP**, Chân-Uyển: **+10% Chính xác**, Chân-Đai: **Giải trừ khống chế & miễn dịch 5s**, Chân-Kiên: **Giảm 10–15% sát thương**).
  - **Đầy đủ 6 ô Phụ Kiện VNG Thương Mại**: **Bí Bảo Trấn Phái (Slot 23)** `Phạn Linh Châu-Ngũ Hành`, **Hào Hiệp Ấn (Slot 21)** `Thanh Long Hào Hiệp Ấn`, **Bang Hội Lệnh Bài (Slot 20)** `Giang Hồ Lệnh`, **Long Văn (Slot 19)**, **Võ Hồn (Slot 18)** `Lưu Ly Diệm`, **Ám Khí (Slot 17)** `Phi Hoàng Thạch`, cùng dàn **Thú Cưỡi Đỉnh Cấp / Thượng Cổ Thần Thú (`Seg=2207..2213`)** cộng tới **+51.020 HP & +510 Thuộc tính**.

#### Bảng Build 18 Vị Trí Trang Bị Mạnh Nhất — Server 30 (`TanThienLong`)

| Vị trí | Tên trang bị tối ưu | Item ID | Cấp | Sao | Chỉ số cơ bản & Hiệu ứng kỹ năng (`Impact`) | Chỉ số dòng thuộc tính (`ItemSegValue` thực tế) |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1. Vũ khí** | **Trường Tuyệt (Phiến 9★)** | `10317043` | 102 | **9★** | Nội công CB: **1.554** \| Impact `55090` | **12 dòng (`Seg=4151`):** **HP tối đa +18%**, **Băng công +169**, **Độc công +169** *(kết hợp Tiêu Dao Song Tu)*, Nội công **+1.554**, Chính xác **+1.960**, Hội công **+17**, Thể lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **2. Mão** | **Chân-Trùng Lâu Khôi** | `10410151` | 50 | **9★** | Ngoại/Nội thủ CB: **1.101** \| **Impact `38330`: Tỉ lệ bạo kích +10%** | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **3. Y phục** | **Chân-Trùng Lâu Giáp** | `10413104` | 50 | **9★** | Ngoại/Nội thủ CB: **1.101** \| **Impact `38336`: Giới hạn sinh lực +10%** | **9 dòng (`Seg=100`, Set 207):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** *(Kích Set 207 +5% HP)* |
| **4. Hộ kiên** | **Trùng Lâu Kiên Siêu Cấp** *(hoặc `Chân Trùng Lâu Kiên` `10415056`)* | `10415055` `10415056` | 50 | **9★** | Ngoại/Nội thủ CB: **1.101** \| **Impact `20337`: Giảm 15% sát thương nhận vào** | **9 dòng (`Seg=100`, Set 182):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** *(Kích Set 182)* |
| **5. Hộ uyển** | **Chân-Trùng Lâu Uyển** | `10414057` | 50 | **9★** | Chính xác CB: **1.230** \| **Impact `38338`: Chính xác +10%** | **9 dòng (`Seg=100`):** HP **+12.520**, **Ngoại công +1.106**, **Nội công +1.106**, Chính xác **+1.230**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **6. Hộ thủ** | **Chân-Trùng Lâu Thủ** | `10412099` | 50 | **9★** | Ngoại/Nội thủ CB: **1.101** \| **Impact `38334`: Toàn bộ sát thương tăng +15%** | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **7. Yêu đai** | **Chân-Trùng Lâu Đai** | `10421094` | 50 | **9★** | Ngoại/Nội thủ CB: **1.101** \| **Impact `38340`: Tự xóa khống chế + Miễn dịch khống chế 5s** | **9 dòng (`Seg=100`, Set 207):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** *(Kích Set 207)* |
| **8. Hài** | **Chân-Trùng Lâu Ngoa** | `10411103` | 50 | **9★** | Né tránh CB: **410** \| **Impact `38332`: 3% xác suất gây bạo kích x5 sát thương** | **9 dòng (`Seg=100`):** HP **+12.520**, Ngoại thủ **+1.101**, Nội thủ **+1.101**, Né tránh **+410**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Hội thủ **+10**, Tất cả thuộc tính **+21** |
| **9. Hạng liên** | **Chân.Trùng Lâu Liên** | `10420089` | 50 | **9★** | HP/MP CB: **6.441** \| **Impact `5984`: Thất Minh 25 giây** | **9 dòng (`Seg=100`, Set 182):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** *(Kích Set 182)* |
| **10. Giới chỉ 1 & 2** | **Chân-Trùng Lâu Giới** *(x2 hoặc `Chân-Trùng Lâu Giới` `10422149` & `10423076`)* | `10422149` `10423076` | 50 | **9★** | Chính xác CB: **1.230** \| **Impact `5967`: Phong Huyệt 20 giây** | **9 dòng (`Seg=100`, Set 182):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **11. Hộ phù 1 & 2** | **Chân-Trùng Lâu Ngọc** *(x2 hoặc `Chân-Trùng Lâu Ngọc` `10423025` & `10423077`)* | `10423025` `10423077` | 50 | **9★** | Né tránh CB: **410** \| **Impact `5965`: Tê Liệt 20 giây** | **9 dòng (`Seg=100`, Set 182):** HP **+12.520**, MP **+6.441**, Chính xác **+1.230**, Né tránh **+410**, Hội công **+10**, Thể lực **+114**, Trí lực **+114**, Thân pháp **+62**, Tất cả thuộc tính **+21** |
| **12. Ám khí** | **Phi Hoàng Thạch** *(hoặc `Kim Sí Linh Vũ`)* | `10155001` | 30 | 1★–8★ | Tu luyện 8 tầng | **5 dòng (`Seg=2001`):** Cường lực **+10**, Nội lực **+10**, Thể lực **+10**, Trí lực **+10**, Thân pháp **+10** |
| **13. Võ hồn** | **Lưu Ly Diệm** | `10156001` | 65 | 1★–8★ | Nâng cấp bằng Linh Hồn Toái Phiến | Hợp thành cấp cao, mở 3 kỹ năng thuộc tính Hỏa/Độc và 4 lỗ ngọc |
| **14. Long văn** | **Long Văn** | `10157001` | 80 | 1★–8★ | Trưởng thành, nâng sao, tẩy luyện | **20 dòng (`Seg=4001`):** HP **+4.676**, MP **+2.421**, **Băng/Hỏa/Huyền/Độc công +64**, Ngoại/Nội công **+824**, Ngoại/Nội thủ **+804**, Chính xác **+432**, Né **+144**, Hội công **+10**, Thể lực **+43**, Tất cả **+8** |
| **15. Lệnh bài** | **Giang Hồ Lệnh** | `10158001` | 80 | 1★–8★ | Bang Hội Lệnh Bài thương mại | Mở 4 bảo châu khảm thuộc tính Hỏa, Thể lực và Giảm kháng Hỏa |
| **16. Hào hiệp ấn** | **Thanh Long Hào Hiệp Ấn-Thái Nhất** | `10159001` | 85 | 1★–8★ | Hào Hiệp Ấn VNG | Giảm thời gian khống chế (Phong huyệt, Tản công, Tê liệt, Thất minh) |
| **17. Bí bảo trấn phái** | **Phạn Linh Châu-Ngũ Hành** | `10154005` | 60 | 1★–8★ | Bí bảo độc quyền VNG (Slot 23) | **7 dòng (`Seg=5005`):** HP **+5.778**, Né tránh **+189**, Thể lực **+53**, Trí lực **+53**, Thân pháp **+29**, Hội thủ **+5**, Tất cả thuộc tính **+10** |
| **18. Thú cưỡi thuộc tính** | **Thượng Cổ Thần Thú-Hỏa: Ám Viêm Minh Sư** *(hoặc `Dao Quang Khai Minh Thú` `10141905`)* | `10141907` `10141905` | 10 | 7★ | Slot Cưỡi (Slot 8) VNG | **6 dòng (`Seg=2209`):** **HP +51.020**, **Hỏa công +510**, Né tránh **+612**, Thể lực **+612**, Thân pháp **+612**, **Giảm kháng Hỏa +54** *(Dao Quang: HP +51.020, 5 Tiềm năng +1.020)* |

---

## Phần IV: Bảng Đối Chiếu Các Hệ Thống Phụ Trợ Gia Tăng Sức Mạnh Trên 30 Server

Ngoài 12–21 món trang bị mặc trực tiếp trên người, sức mạnh tổng thể (Max Power) của nhân vật **Tiêu Dao** còn được quyết định bởi **12 Hệ thống Phụ trợ** dưới đây. Bảng này tổng hợp chính xác sự hiện diện và trần sức mạnh của từng hệ thống trên 30 Server (theo STT thời gian mới `01..30`):

| Hệ thống Phụ trợ | Nhóm Cổ Điển 0.26 (`STT 01`) | Nhóm 0.85 Chuẩn & Custom (`STT 02, 03, 04, 08`) | Nhóm Custom Siêu Cấp & 3.0–3.78 (`STT 05, 06, 07, 09, 10, 11, 12, 13, 14, 15, 18`) | Nhóm Hoài Cựu 9P/10P 32 & 64-bit (`STT 16, 17, 22..27`) | Nhóm ChangYou 3.72 & 14 Phái Hiện Đại (`STT 19, 20, 21, 28, 29`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Cường Hóa (`EquipStrengthen`)** | Tối đa **+9** *(Tăng chỉ số cơ bản vũ khí/phòng cụ)* | Tối đa **+9** *(Riêng STT 03 mở rộng mốc Chí Tôn)* | Tối đa **+9 đến +99** *(STT 07, 10..15, 18 hỗ trợ tới lv 99)* | Chuẩn **+9** *(Tăng ~68%–110% chỉ số cơ bản)* | **Chí Tôn Cường Hóa Cấp 99** *(Tăng hàng ngàn Nội công/Phòng thủ CB)* |
| **2. Khảm Ngọc (`GemInfo`)** | **3–4 lỗ, Ngọc C9** *(Chưa có Giảm kháng & Thuần Tịnh)* | **4 lỗ, Ngọc C9 Thuần Tịnh** *(Hỏa +220, Giảm kháng +150)* | **4 lỗ, Ngọc Kép C7·C7 / C9·C9** *(STT 09: Kháng +32.000; **STT 11: +1.020 Hỏa, +320 Thể, +162 Giảm kháng**; **STT 12: Trần C7 nhưng Thể +534**; STT 15: +420 Hỏa)* | **4 lỗ, Ngọc C9 Thuần Tịnh** *(Riêng STT 27 buff Giảm kháng +220, Thể +300)* | **4 lỗ, Ngọc Minh Tinh C9·C9** *(STT 21: +350 Hỏa & +150 Giảm kháng, +342 Thể)* |
| **3. Điêu Văn (`DWConfig`)** | ❌ Chưa có | ❌ Chưa có *(Hoặc tích hợp qua Script)* | ✅ **Điêu Văn Cấp 10** *(Tăng +520 Thể lực/Thân pháp hoặc +700 Hỏa công/món)* | ✅ Hỗ trợ qua Lua / Thiên Giám | ✅ **Song Điêu Văn (Điêu Văn Kép Cấp 10)** trên cả 17 vị trí! |
| **4. Tinh Thông Trang Bị (`EquipJingTong`)** | ❌ Chưa có | ❌ Chưa có | ✅ Mở trên STT 11, 12, 14, 15, 18 | ❌ Chưa có | ✅ **Tinh Thông Cấp 100 (3 dòng/món):** 8 món Công ép **3 dòng Hỏa công**, 9 món Thủ ép **3 dòng Thể lực**! |
| **5. Chân Nguyên (`ZhenYuan`)** | ❌ Chưa có | ❌ Chưa có | ✅ Mở trên STT 10, 11, 12, 13, 14, 15, 18 *(12 ô Chân Nguyên Đỏ/Cam)* | ❌ Chưa có *(Thay bằng **Thú Hồn `PetSoul`** & **Võ Ý `Wuyi`**)* | ✅ **12 Ô Chân Nguyên Cam/Tím Cấp 9** *(6 ô Công tăng +% Thuộc tính trên trang bị, 6 ô Thủ tăng +% Thể lực)* |
| **6. Võ Hồn (`WuHun`)** | ❌ Chưa có | ✅ Mở trên STT 03 (`Ngự Dao Bàn` Lv65) | ✅ **Võ Hồn Hợp Thành Cấp 7–9** *(STT 11: +300 Cường/Nội/Thể/Trí; STT 12: `Pháp Bảo·Ngự Dao Bàn`; STT 14: +50% HP/MP & +160 Giảm kháng)* | ✅ **`Ngự Dao Bàn` Cấp Hợp Thành 7**, mở 3 kỹ năng + 4 lỗ ngọc | ✅ **`Ngự Dao Bàn` Cấp Hợp Thành 9**, tăng >600 Hỏa/Độc + Hạ kháng |
| **7. Ám Khí (`DarkItem`)** | ❌ Chưa có | ✅ `Băng Phách Thần Châm` (`10155002`) / `Kim Sí` (STT 03) | ✅ **`Kim Sí Linh Vũ` / `Long Ngâm` (STT 11) / `Đoạn Hồn Phiêu` (STT 14)** | ✅ **`Băng Phách Thần Châm` (`10155002`)** | ✅ **`Kim Sí Linh Vũ` (`10155008`, Lv120)** — Ngũ Độc/Hỏa Liên Hoa + Luyện chuẩn + 4 lỗ |
| **8. Long Văn (`LongWen`) / Thiên Giám** | ❌ Chưa có | ❌ Chưa có | ✅ **Long Văn +9 / Cấp 80–100** *(**STT 11: +185 4 hệ & +75 Tất cả**; **STT 14: +629 Hỏa & +50% HP**; STT 18 có cả Long Văn & 6 Thiên Giám)* | ✅ Thay thế bằng **Bộ 3 Món Thiên Giám (Slot 19, 20, 21)** | ✅ **Long Văn Cấp 100 (9★)** *(+20% HP, +Xuyên kháng Hỏa, +Sát thương Hỏa)* |
| **9. Lệnh Bài (`BangHoi`) & Hào Hiệp Ấn** | ❌ Chưa có | ❌ Chưa có | ✅ Mở trên STT 07, 10, 12, 13, 14, 15 *(Riêng **STT 12 có `Kim Lan Lệnh` +66% HP** & `Thanh Long Hào Hiệp Ấn`)* | ❌ Chưa có *(Dùng Thiên Giám)* | ✅ **`Tụ Nghĩa Lệnh` (+50% HP, 4 Châu Báu)** + **`Chu Tước Hào Hiệp Ấn` (Giảm khống chế)** |
| **10. Tu Luyện (`XiuLian`) & Bí Tịch** | ❌ Chưa có | ❌ Chưa có | ✅ **Tu Luyện Ngũ Hành + Bí Tịch** *(STT 07, 10, 11, 12, 13, 14, 15, 18)* | ✅ **Tu Luyện + Tiến Cấp Kỹ Năng** *(Cường hóa Khê Sơn Hành Lữ)* | ✅ **Tu Luyện 15 Quyển + Võ Đạo/Thiên Đạo + Bí Tịch (`Thần Công Lục` + `Long Khấp Quyết`)** |
| **11. Thần Đỉnh (`ShenDing`) & Con Cái (`Infant`)** | ❌ Chưa có | ❌ Chưa có | ✅ Mở trên STT 10, 11, 12, 13, 14, 15, 18 *(Càn Nguyên Kim Đỉnh + Con Cái + Bách Bảo Nang/Rương)* | ✅ Thay bằng **Võ Ý (`Wuyi`)** & **Thú Hồn (`PetSoul` — `Chu Tước/Huyền Vũ`)** | ✅ **Song Đỉnh 9 Phẩm (`Càn Nguyên` + `Khôn Giám`)** + **Con Cái Lv119 (4 Kỹ năng Anh Hùng)** |
| **12. Xuyên Thích (`Penetration`)** | ❌ Chưa có | ❌ Chưa có | ✅ Mở trên STT 11, 12, 14, 15, 18 | ❌ Chưa có | ✅ **Xuyên Thích Bạo Kích** *(Từ Cường hóa Chí Tôn, Tinh Thông, Chân·Trùng Lâu, Hào Hiệp Ấn)* |

---

## Phần V: Tổng Kết & Phân Loại 5 Trường Phái Build Tiêu Dao Trong 30 Server

Từ dữ liệu giải mã thực tế của cả **30 Server** (đã sắp xếp chuẩn theo dòng thời gian `STT 01..30`), người chơi và quản trị viên server có thể quy hoạch **Build Tiêu Dao Mạnh Nhất** theo **5 Trường Phái (Archetypes)** rõ ràng:

1. **Trường phái 1 — Cổ Điển Thuần Nội Công & Chế 9x (`STT 01 - TLBB026_Offline`):**
   - Chưa có Giảm kháng, chưa có Trùng Lâu. Sức mạnh nằm ở **Đồ Chế 9x–100 (7 dòng, 8★)** cộng song song **Nội công + Thể lực + Thân pháp + Tất cả thuộc tính**, khảm **Hồng Tinh Thạch + Miêu Nhãn Thạch + Tử Ngọc**.
2. **Trường phái 2 — Chuẩn 0.85 Kinh Điển (`STT 02, 03, 04, 08`):**
   - Sự lên ngôi của **Thần Khí 102 (`Đại Tống Quân Sầm` / `Phù Sinh` / `Thái Cổ Phiêu Linh`)**, **Ngọc Thuần Tịnh C9 (+220 Hỏa) + Lục Minh Thạch C9 (+150 Giảm kháng)**.
   - Ở bản chuẩn (`STT 02, 04`): Phối **Đồ Chế 11x (`Vong Xuyên`, `Khấp Lộ`, `Quy Túc`, `Cát Tường Dạ`, `Ưng Tường`)** với **cặp Nhẫn/Phù Trùng Lâu**.
   - Ở bản mở đủ Set Trùng Lâu (`STT 03, 08`): Mặc **Full 10 Món `(Thần) Trùng Lâu`** để lấy lượng máu, chính xác, né tránh, hội thủ vượt trội và kích hoạt dòng ẩn Set 184/185.
3. **Trường phái 3 — Hoài Cựu Hiện Đại 9–10 Phái (`STT 16, 17, 22, 23, 24, 25, 26, 27`):**
   - Đặc trưng bởi **Bộ 3 Món Thiên Giám (Slot 19, 20, 21)**, **5 Món Trùng Lâu / Chân-Trùng Lâu** phối cùng **Đồ Chế 9x–11x** ở các vị trí Mão, Uyển, Thủ, Đai, Hài.
   - **Lưu ý Vàng:**
     - Trên `STT 16, 17, 26`: Luôn dùng `Chân-Trùng Lâu Liên` (`10420089`) & `Chân Trùng Lâu Ngọc` (`10423025`) vì có **13 dòng (+83 4 hệ)**.
     - Trên `STT 25 (tlbb-10p-vogioi)`: Phải dùng **`Trùng Lâu Liên/Giới/Ngọc` thường (`10420088`/`10422016`/`10423024`)** vì được mở **13 dòng (`Seg=100`: +32.221 HP, +110 4 hệ)** mạnh hơn bản Chân 9 dòng!
     - Trên `STT 27 (Skynet_bot_ai)`: Dùng trọn bộ **`Chân·Chí Tôn Trùng Lâu` (`Seg=158..161`, +326 4 hệ/món)** và **`Đại Tống Quân Sầm · Thần Thoại` (`10563207`)**.
4. **Trường phái 4 — ChangYou 3.6 / 3.63 / 3.72 / 3.78, 14 Phái & VNG 15 Phái Đỉnh Cao (`STT 12, 14, 15, 19, 20, 21, 28, 29, 30`):**
   - Nền tảng hoàn chỉnh nhất với **Thần Khí 102–105 `Phù Sinh` / `Phù Sinh Quan Khích` / `Thái Cổ Phiêu Vũ` / `Trường Tuyệt`**, **Chân·Trùng Lâu kích 2 Set ẩn**, **`Tụ Nghĩa Lệnh` / `Kim Lan Lệnh` / `Giang Hồ Lệnh`**, và **`Hào Hiệp Ấn`**.
   - Trên `STT 12 (TLBB 3D OFFLINE 3.63)`: Khai thác tối đa **6 Món `Chân·Trùng Lâu` (`Seg=4321`, +200 4 hệ & +24 Giảm kháng/món)** để kích hoạt đồng thời **Set `155 Trùng Lâu·Binh Tuyệt` (+20% HP, +4.799 Né tránh)** và **Set `156 Trùng Lâu·Phá Giáp` (+5.277 Chính xác, +89 Hội công)**, kết hợp **`Kim Lan Lệnh` (+66% HP)**, **`Thời Trang Thuộc Tính` (+120 4 hệ)** và **Ngọc C7 (`Hồng Bảo Thạch C7` +534 Thể lực)**.
   - Trên `STT 14 (TLBB 3.78)`: Tận dụng 2 món phụ kiện đột phá là **`Long Văn Cấp 80 Max` (`10157029`, +629 Hỏa công, +50% HP, +38 Giảm kháng)** và **`Ngự Dao Bàn` (`10156002`, +50% HP, +50% MP, +160 4 Giảm kháng)** kết hợp **`Thái Cổ Phiêu Vũ` (`10308155`)** & **Đồ Chế 150 (`150 Cấu Tạo`)**.
   - Trên `STT 15 (TLBB 3.6)`: Dùng **TK 102 `Phù Sinh` 9★ 16 dòng (`Seg=3009`, +20% HP, +178 4 hệ, +45 Giảm kháng)** + **11 Món `Chân-Trùng Lâu` phối Đồ Chế 150** + **`Bách Bảo Nang/Rương` 20 dòng**, khảm full **Ngọc C9 Buff (+420 Hỏa/Độc, +513 Thể lực, +33.080 HP)**.
   - Trên `STT 19 (tlbb_lee)` & `STT 29 (Level Up Games)`: Mặc **Mũ `Siêu Cấp Tai Thỏ`** + 3 món **`Chân-Trùng Lâu Uyển/Thủ/Ngoa` (`10553114..10553116`, +320 4 hệ/món)**.
   - Trên `STT 20 (tlbb129)`: Uyển/Thủ/Hài dùng **Đồ Chế 116–126** (`Quy Túc`, `Phệ Lân Huyền Thủ`, `Thệ Lân Thánh Lý`) — các món này **không có dòng 4 hệ công**, chỉ cho Ngoại/Nội công/thủ, Chính xác/Né và 5 thuộc tính chính *(đã đối chiếu lại 07/11/2026)*.
   - Trên `STT 21 (TLBB PB14P2024)`: Phối **5 Món `Chân·Trùng Lâu` (`Seg=4323`/`4324`, +221 4 hệ)** + **Bộ `Trùng Lâu·Phản/Triệu/Quy` (`Seg=100`, +100 4 hệ)** + **Đồ Chế 150 (`Seg=1142..1150`)** + Ngọc Tinh Khiết C9 (+350 thuộc tính).
   - Trên `STT 28 (Skynet-bot)`: Đeo thêm cặp **`Chân·Trùng Lâu Giới/Ngọc` (`10422149`/`10423025`, 13 dòng, +105 4 hệ, +45 Hội công)** ở ô Nhẫn #2 và Phù #2.
   - Trên `STT 30 (TanThienLong - VNG Official)`: Đỉnh cao 15 môn phái với **Thần Khí 9★ `Trường Tuyệt` (`10317043`, +18% HP, +169 Băng/Độc)** + **Full 10 món `Chân-Trùng Lâu` VNG (`Seg=100`, kích Set 182 & Set 207: +5% HP)** bộc phá toàn diện với các Impact độc quyền (**Bạo kích +10%, 3% x5 Bạo kích, Sát thương +15%, Miễn dịch khống chế 5s, Giảm 10–15% sát thương**), kết hợp **Bí Bảo Trấn Phái (Slot 23)**, **Hào Hiệp Ấn (Slot 21)**, **Giang Hồ Lệnh (Slot 20)**, **Long Văn (Slot 19)** và **Thượng Cổ Thần Thú thuộc tính (`Seg=2209`: +51.020 HP & +510 Hỏa công)**.
5. **Trường phái 5 — Custom 16–19 Dòng, Cầm Long Thức, Ám Hắc Quần Phục & Siêu Cấp Chuyển Sinh (`STT 05, 06, 07, 09, 10, 11, 13, 18`):**
   - Các trang bị `Chân·Trùng Lâu` / `Chí Tôn·Trùng Lâu` được mở khóa tối đa **12–19 dòng thuộc tính** (cộng cả 4 hệ công lẫn Giảm kháng 4 hệ trên cùng 1 món đồ như ở `STT 05, 07, 11, 13, 18`), hoặc chia riêng trang sức theo hệ Hỏa (`[69 Dragon] Legendary Fire` ở `STT 06`).
   - Ở **`STT 11 (TLBB 3.6 OFFLINE - Cầm Long Thức)`**: Thần Khí **`Mã Nhã Chi Linh` (`10558701`)** cộng tới **+1.500 4 hệ & +500 Nội/Thể**, đi kèm viên **`Hồng Minh Tinh Thạch C9·9` (`50921309`)** cộng đồng thời **3 chỉ số (+1.020 Hỏa công, +320 Thể lực, +162 Giảm kháng Hỏa)** và bộ **5 Món `Chân·Trùng Lâu` (`Seg=4243`, 19 dòng)**.
   - Ở **`STT 18 (暗黑群服HoaiNiem)`**: Nhân vật mặc tới **21 món trang bị** gồm **5 Món `Chí Tôn·Trùng Lâu` (`Seg=5004`, +493 4 hệ & +160 Giảm kháng/món)** + **5 Món `Chân·Trùng Lâu` (`Seg=2152`, +600 tiềm năng & +432 Tất cả/món)** + **Bộ 6 Món `Thiên Giám Linh Vũ`** + **`Bách Bảo Nang/Rương` 20 dòng**; và đạt đỉnh điểm số học tuyệt đối ở **`STT 09 (TLBB chuyen sinh)`** với bộ **`Siêu Cấp.Song Long Trùng Lâu +15` (+170.000 HP & +13.000 4 hệ mỗi món)**.
