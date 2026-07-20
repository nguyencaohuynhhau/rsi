# Diataxis Writer Skill

**Language / Ngôn ngữ / 语言:** [English](./README.md) | [Tiếng Việt](./README.vi.md) | [中文](./README.zh.md)

Skill này giúp agent viết, tái cấu trúc và review tài liệu theo framework
Diataxis.

## Diataxis là gì?

Diataxis là một framework tổ chức tài liệu dựa trên nhu cầu thật sự của người
đọc. Thay vì gom mọi thông tin liên quan vào cùng một trang, Diataxis chia tài
liệu thành 4 loại chính:

| Loại tài liệu | Khi người đọc muốn | Mục tiêu |
| --- | --- | --- |
| Tutorial | Học qua thực hành | Dẫn người đọc đi theo một lộ trình an toàn để đạt năng lực ban đầu |
| How-to guide | Hoàn thành một việc cụ thể | Đưa ra các bước rõ ràng để đạt một kết quả |
| Reference | Tra cứu thông tin chính xác | Cung cấp dữ liệu đầy đủ, nhất quán, dễ quét |
| Explanation | Hiểu bối cảnh hoặc lý do | Giải thích khái niệm, quyết định, tradeoff và mô hình tư duy |

Điểm cốt lõi của Diataxis là: mỗi tài liệu nên có một lời hứa rõ ràng với người
đọc. Một trang tutorial không nên cố trở thành API reference. Một trang how-to
không nên bị kéo dài bởi giải thích kiến trúc. Một trang reference không nên bắt
người đọc đi qua một câu chuyện dài trước khi tìm thấy field hoặc option họ cần.

## Ưu điểm

- **Giảm tài liệu rối và dài quá mức**: Diataxis giúp tách "học", "làm", "tra
  cứu" và "hiểu" thành các mục tiêu riêng.
- **Cải thiện trải nghiệm người đọc**: người đọc tìm đúng loại tài liệu cho việc
  họ đang làm, thay vì phải tự lọc thông tin.
- **Dễ review chất lượng hơn**: có thể hỏi "trang này đang phục vụ reader job
  nào?" trước khi sửa nội dung chi tiết.
- **Dễ bảo trì hơn**: mỗi trang có phạm vi và lời hứa rõ ràng, nên việc cập nhật
  ít kéo theo thay đổi lan man.
- **Phù hợp với nhiều loại knowledge work**: không chỉ technical docs, mà cả
  onboarding, knowledge base, process docs, manuals, runbooks, product docs và
  operational guides.

## Tại sao nên dùng Diataxis?

Nhiều tài liệu gây khó chịu không phải vì thiếu thông tin, mà vì trộn nhiều mục
đích trong cùng một chỗ. Một người mới cần được dẫn từng bước. Một người đang xử
lý task cần chỉ dẫn ngắn và chính xác. Một người tra cứu cần bảng, field,
default, constraint. Một người muốn hiểu cần bối cảnh và lý do.

Diataxis buộc người viết chọn mục tiêu chính trước khi viết. Nhờ vậy tài liệu có
cấu trúc tự nhiên hơn, dễ đọc hơn và ít mâu thuẫn hơn.

## Khi nào nên dùng skill này?

Dùng skill này khi cần:

- viết tài liệu mới theo Diataxis;
- review một tài liệu đang rối hoặc quá dài;
- tách một trang "getting started" thành tutorial, how-to, reference và
  explanation;
- thiết kế lại knowledge base hoặc docs site;
- viết onboarding docs, process docs, runbooks, manuals, product docs hoặc API
  docs.

Không nên áp dụng máy móc cho marketing copy, sales proposal, legal contract,
press release, fiction, hoặc nội dung chủ yếu nhằm thuyết phục cảm xúc. Các loại
tài liệu đó có thể mượn một phần tư duy phân loại của Diataxis, nhưng không nên
bị ép vào framework này.

## Cách dùng nhanh

Trong một agent hỗ trợ skills, hãy yêu cầu các việc như:

```text
Review tài liệu này theo Diataxis và đề xuất cách tách lại.
```

```text
Viết một tutorial getting started cho công cụ nội bộ này.
```

```text
Tạo reference docs cho CLI command này, giữ nó dễ tra cứu.
```

Skill cũng có script heuristic để phân loại tín hiệu Diataxis trong tài liệu:

Chạy lệnh từ thư mục của skill:

```bash
bash ./scripts/classify-doc.sh path/to/doc.md
```

Script này chỉ hỗ trợ chẩn đoán nhanh. Quyết định cuối cùng vẫn nên dựa trên
reader job và ngữ cảnh tài liệu.

## Cài Đặt

### 1. Dùng CLI (Khuyến nghị)

```bash
npx skills add tronghieu/agent-skills --skill diataxis-writer
```

### 2. Cài Đặt Thủ Công (Cho người dùng cơ bản)

1. **Tải về:** Truy cập thư mục `skills/` trong kho lưu trữ này và tải file `diataxis-writer.zip`.
2. **Giải nén & Copy:** Giải nén file `diataxis-writer.zip` và copy thư mục `diataxis-writer` vào một trong các vị trí sau:

**Cho một dự án cụ thể:**
Copy thư mục `diataxis-writer` vào `.agents/skills/` hoặc `.claude/skills/` trong thư mục gốc dự án của bạn.

**Cài đặt toàn cục (Dùng cho mọi dự án):**
* **Mac / Linux:** `~/.agents/skills/` hoặc `~/.claude/skills/`
* **Windows:** `%USERPROFILE%\.agents\skills\` hoặc `%USERPROFILE%\.claude\skills\` (thường là `C:\Users\<Tên_Của_Bạn>`)
