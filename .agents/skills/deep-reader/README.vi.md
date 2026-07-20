# Deep Reader Skill

**Language / Ngôn ngữ / 语言:** [English](./README.md) | [Tiếng Việt](./README.vi.md) | [中文](./README.zh.md)

Skill này giúp agent đọc sâu (deep-read) những cuốn sách và bài báo dài — theo
cách một nhà nghiên cứu cẩn thận vẫn làm, chứ không phải nhồi cả văn bản vào
một prompt.

## "Đọc Sâu" Ở Đây Nghĩa Là Gì

Sự chú ý (attention) của LLM bị loãng dần trên các context dài và dễ đánh mất
thông tin nằm giữa một prompt lớn (hiện tượng "lost in the middle"). Nạp cả
một cuốn sách 500 trang vào một context rồi hỏi về nó sẽ cho ra câu trả lời
âm thầm quên mất chương 3 vào lúc đang bàn tới chương 12.

Skill này khắc phục điều đó bằng chính phương pháp các nhà nghiên cứu vẫn
dùng: không bao giờ giữ cả cuốn sách trong bộ nhớ làm việc cùng lúc. Nó kết
hợp ba phương pháp đọc đã được kiểm chứng thành một pipeline:

- **Phương pháp của Mortimer Adler trong *How to Read a Book*** — đọc kiểm
  tra (inspectional), đọc phân tích (analytical), và đọc so sánh nhiều nguồn
  (syntopical), áp dụng theo đúng thứ tự đó.
- **Bước Recite của SQ3R** — sau khi viết xong ghi chú của một chương, đọc
  lướt lại chương đó và đối chiếu từng luận điểm, từng trích dẫn với văn bản
  gốc trước khi chuyển sang chương tiếp theo.
- **Phương pháp ba lượt đọc (three-pass) của S. Keshav** cho bài báo, luận
  văn, và bài khảo sát (survey).

Cuốn sách được đọc theo từng lượt — trước tiên để nắm cấu trúc, sau đó đọc
từng chương để nắm nội dung — và mọi thứ học được đều được ghi ra ngoài vào
một **workspace ghi chú neo theo số trang** (page-anchored notes workspace)
ngay trong lúc đọc. Ghi chú trở thành bộ nhớ bền vững; văn bản gốc chỉ cần
được đọc lại khi đang xử lý một chương cụ thể hoặc cần xác minh một luận điểm
cụ thể. Số trang chính là hệ tọa độ giúp mọi luận điểm có thể truy ngược về
nguồn, dù trong phiên làm việc này hay một phiên sau.

## Hai Chế Độ

- **overview** — chỉ chạy lượt đọc kiểm tra (inspectional) của Adler: một bản
  đồ cuốn sách (book map) cộng với bản tóm tắt bám sát mục đích đọc, tập
  trung vào những chương quan trọng nhất. Nhanh, không viết ghi chú theo
  từng chương.
- **study** — pipeline đầy đủ: xác định mục đích đọc → bản đồ kiểm tra →
  ghi chú phân tích theo từng chương kèm xác minh Recite → tổng hợp phân cấp
  trả lời bốn câu hỏi của Adler → xác minh trích dẫn bằng máy.

Bạn chọn chế độ (hoặc agent đề xuất một chế độ dựa trên độ dài cuốn sách và
mục đích đọc bạn nêu ra, rồi bạn xác nhận hoặc chỉnh lại).

## Khi Nào Nên Dùng Skill Này

Dùng skill này bất cứ khi nào bạn yêu cầu agent đọc, nghiên cứu, tóm tắt,
phân tích, review, hoặc trả lời câu hỏi về một cuốn sách dài, giáo trình,
PDF, EPUB, luận văn, luận án, bài khảo sát, hay bất kỳ tài liệu nào khoảng
50+ trang trở lên. Nó cũng áp dụng khi một workspace ghi chú từ phiên trước
đã tồn tại cạnh file nguồn và bạn hỏi tiếp một câu về cuốn sách đó — agent sẽ
tìm trong ghi chú có sẵn thay vì đọc lại từ đầu.

Đừng kỳ vọng skill này kích hoạt với tài liệu ngắn (dưới khoảng 30 trang):
những tài liệu đó vừa đủ trong context, và cả pipeline này sẽ chỉ là overhead
thừa thãi.

**Câu kích hoạt:** "đọc cuốn sách này giúp tôi", "nghiên cứu file PDF này",
"tóm tắt cuốn giáo trình này", "phân tích bài paper này", "đọc kỹ luận án
này", "tóm tắt sách", "phân tích luận án", "nghiên cứu paper này"

## Yêu Cầu

- **bash** — bắt buộc, mọi script đều là shell kiểu POSIX (coreutils, awk,
  grep, sed).
- **pdftotext** (thuộc gói **poppler**) — chỉ bắt buộc với nguồn PDF.
- **pandoc** — chỉ bắt buộc với nguồn EPUB/DOCX. Nguồn TXT/MD không cần cả
  hai.

Cài trên macOS:

```bash
brew install poppler pandoc
```

Cài trên Debian/Ubuntu:

```bash
apt-get install poppler-utils pandoc
```

Nếu thiếu dependency, script chuẩn bị văn bản sẽ thoát với thông báo lỗi rõ
ràng thay vì âm thầm lỗi, và skill sẽ chuyển sang đọc trực tiếp file
PDF/EPUB bằng Read tool của agent, theo từng đợt trang. Khi rơi vào nhánh dự
phòng này, bạn mất bước xác minh trích dẫn bằng máy, nên agent sẽ bù lại
bằng cách đọc lại thủ công thường xuyên hơn mỗi khi trích dẫn điều gì.

## Cách Dùng

Trong một agent hỗ trợ skills, hãy yêu cầu các việc như:

```text
Đọc sâu cuốn "Domain-Driven Design" và cho tôi biết nó áp dụng thế nào vào
ranh giới service hiện tại của bọn tôi — tôi muốn bản study đầy đủ, không
chỉ tóm tắt.
```

```text
Tôi chỉ cần nắm sơ bộ cuốn sổ tay 400 trang này trước cuộc họp ngày mai —
cho tôi một overview: các phần chính là gì và chương nào thực sự quan trọng
với người đang xây dựng BI pipeline?
```

```text
(sau đó, cùng cuốn sách) Sách nói gì về aggregate root so với bounded
context? Kiểm tra ghi chú có sẵn trước đã.
```

Với ví dụ thứ ba, agent sẽ tìm trong `terms.md` và các file `notes/` đã xây
dựng sẵn trong workspace thay vì đọc lại cuốn sách — đó chính là toàn bộ mục
đích của việc ghi chú ra ngoài ngay trong lúc đọc.

## Workspace Ghi Chú

Mọi thứ pipeline tạo ra đều nằm trong một thư mục workspace, mặc định đặt
cạnh file nguồn, để nó tồn tại xuyên suốt nhiều phiên làm việc:

```text
<slug>-notes/
├── source.txt          # toàn bộ văn bản kèm marker [[page N]] — hệ tọa độ
├── chapters.tsv         # cấu trúc đã xác nhận: chNN<TAB>from<TAB>to<TAB>title
├── chapters/            # từng file chương được cắt bởi split-chapters.sh
│   └── ch01-<slug>.txt
├── map.md               # kết quả lượt đọc kiểm tra
├── terms.md             # sổ thuật ngữ chính, xuyên suốt các chương
├── notes/
│   └── ch01-<slug>.md   # mỗi chương một ghi chú phân tích
└── synthesis.md         # bản tổng hợp cuối cùng + nhật ký xác minh
```

Số trang neo mỗi ghi chú trở lại `source.txt`, đó là điều cho phép một phiên
sau trả lời câu hỏi tiếp theo bằng cách tìm trong ghi chú thay vì đọc lại
cuốn sách.

## Các Script

Bảy script đảm nhiệm phần cơ học của pipeline; agent đảm nhiệm phần phán
đoán (xác nhận cấu trúc, viết ghi chú, tổng hợp) xoay quanh chúng.

| Script | Chức năng |
| --- | --- |
| `prepare-text.sh` | Chuyển nguồn thành `source.txt` đã đánh số trang, tạo workspace (idempotent) |
| `extract-structure.sh` | Bản phác thảo cấu trúc theo heuristic — chỉ là dự đoán ban đầu, cần đối chiếu với mục lục thật |
| `read-pages.sh` | In một trang hoặc một khoảng trang, giữ nguyên marker `[[page N]]` |
| `search-book.sh` | Grep toàn bộ cuốn sách, kết quả có gắn số trang |
| `split-chapters.sh` | Cắt các chương đã xác nhận thành file riêng |
| `build-diagram.sh` | Sơ đồ Mermaid mindmap cơ học theo cấu trúc đã xác nhận |
| `verify-quotes.sh` | Bắt trích dẫn bịa đặt và trích dẫn sai số trang |

Một script thứ tám, `self-test.sh`, không thuộc pipeline đọc — nó là công cụ
"bác sĩ cài đặt" (install doctor). Chạy `bash ./scripts/self-test.sh` bất cứ
khi nào các script hoạt động bất thường; nó tự tạo dữ liệu thử tổng hợp, chạy
thử toàn bộ script, và báo cáo dependency tùy chọn nào (`pdftotext`,
`pandoc`) đang có sẵn, giúp bạn phân biệt lỗi môi trường với lỗi cách dùng.

## Chống Bịa Đặt (Anti-Hallucination) Ngay Từ Thiết Kế

Mọi trích dẫn trong ghi chú ở chế độ study đều được kiểm tra bằng máy:
`verify-quotes.sh` chuẩn hóa và đối chiếu từng chuỗi trích dẫn với trang được
ghi (± 1 trang, để chấp nhận trường hợp câu văn vắt qua hai trang) và gắn cờ
kết quả `FAIL`/`NEAR` cho bất cứ trích dẫn nào bị bịa đặt hoặc ghi sai trang.
Trong thử nghiệm thực tế, pipeline đã được dùng để đọc sâu ba cuốn sách thật
dài 370–600 trang — *Domain-Driven Design*, *Inspired*, và một cuốn sổ tay
ERP/BI — và ở hai lần chạy chế độ study, bộ xác minh đã pass lần lượt
458/458 và 120/120 trích dẫn.

## Cài Đặt

### 1. Dùng CLI (Khuyến nghị)

```bash
npx skills add tronghieu/agent-skills --skill deep-reader
```

### 2. Cài Đặt Thủ Công (Cho người dùng cơ bản)

1. **Tải về:** Truy cập thư mục `skills/` trong kho lưu trữ này và tải file `deep-reader.zip`.
2. **Giải nén & Copy:** Giải nén file `deep-reader.zip` và copy thư mục `deep-reader` vào một trong các vị trí sau:

**Cho một dự án cụ thể:**
Copy thư mục `deep-reader` vào `.agents/skills/` hoặc `.claude/skills/` trong thư mục gốc dự án của bạn.

**Cài đặt toàn cục (Dùng cho mọi dự án):**
* **Mac / Linux:** `~/.agents/skills/` hoặc `~/.claude/skills/`
* **Windows:** `%USERPROFILE%\.agents\skills\` hoặc `%USERPROFILE%\.claude\skills\` (thường là `C:\Users\<Tên_Của_Bạn>`)
