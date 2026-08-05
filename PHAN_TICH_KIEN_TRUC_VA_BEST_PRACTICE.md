# Phân Tích Kiến Trúc Hệ Thống Camera AI & Best Practice

> Tài liệu phân tích toàn diện: cách hệ thống hoạt động, bản chất của bước "train model",
> vai trò thực sự của card RTX, quy trình đưa ảnh vào huấn luyện, và đối chiếu với
> mặt bằng công nghệ hiện nay.
>
> Phạm vi: `face_worker_single/`, `model_trainer/`, `face_worker_monitor/`, `photo_recognition_web/`

---

## Mục Lục

1. [Camera AI làm việc thế nào](#1-camera-ai-làm-việc-thế-nào)
2. ["Train model" thực chất là gì](#2-train-model-thực-chất-là-gì)
3. [Setup model AI với card RTX trên Linux](#3-setup-model-ai-với-card-rtx-trên-linux)
4. [Cách lấy hình vào để train](#4-cách-lấy-hình-vào-để-train)
5. [Vai trò thực sự của card RTX](#5-vai-trò-thực-sự-của-card-rtx)
6. [Công nghệ chính của dự án](#6-công-nghệ-chính-của-dự-án)
7. [Best practice hiện nay cho camera AI](#7-best-practice-hiện-nay-cho-camera-ai)
8. [Lộ trình cải tiến đề xuất](#8-lộ-trình-cải-tiến-đề-xuất)
9. [Danh sách vấn đề cần sửa](#9-danh-sách-vấn-đề-cần-sửa)

---

## 1. Camera AI làm việc thế nào

Hệ thống **không** stream RTSP realtime — nó lấy **file video đã ghi trên thẻ nhớ camera
Dahua** rồi xử lý theo lô (batch).

### 1.1. Luồng xử lý 7 bước

| Bước | Thành phần | Việc làm |
|---|---|---|
| 1 | `CameraPoller`<br>`face_worker_single/workers/camera_poller.py` | Mỗi 15s gọi Dahua HTTP CGI (`mediaFileFind.cgi`) trên 4 IP camera, liệt kê video hôm nay, lọc khung giờ 6:00–10:00, dedup theo `camera_name + start_time`, đẩy thành **job** vào MongoDB (`status: pending`) |
| 2 | `VideoProcessor` – Phase 1<br>`workers/video_processor.py:714` | Scan queue mỗi 5s, lấy tối đa 30 job, **download song song** 4 thread về `/tmp` |
| 3 | Phase 2 – GPU<br>`workers/video_processor.py:784` | `multiprocessing Pool` spawn 3 tiến trình (mỗi tiến trình cô lập VRAM riêng). Decode video bằng PyAV, **lấy mẫu 1 frame/giây** |
| 4 | Detect | InsightFace `buffalo_l` (SCRFD) trên CUDA, `det_size = 1600×1600`. Lọc: `det_score ≥ 0.70`, mặt ≥ 30px. Sharpness (Laplacian) chỉ lưu metadata, **không dùng để lọc** |
| 5 | Recognize<br>`workers/video_processor.py:156` | Embedding 512-D → **cosine similarity** với ma trận trong `model.pkl` (FAISS `IndexFlatIP`, fallback numpy). Lấy **top-3**, ngưỡng `0.52`, rồi **Greedy Assignment**: 1 người chỉ được gán cho 1 khuôn mặt trong 1 frame |
| 6 | Crop + Dedup 2 lớp<br>`workers/video_processor.py:817` | Crop cả mặt + thân (padding 155% ngang, 80% trên, 3.5× xuống).<br>**Lớp 1**: giữ frame có `rec_confidence` cao nhất/người trong 1 video.<br>**Lớp 2**: 1 người chỉ 1 record mỗi buổi (MORNING/AFTERNOON) mỗi ngày.<br>→ Chỉ ảnh sống sót mới upload R2 (tiết kiệm băng thông) |
| 7 | Lưu + Sync | Ảnh JPG → Cloudflare R2; record → MongoDB `recognition_history`; `SyncService` POST lên `cameraai.vnaisoft.net` với header `X-Ma-Truong: MD5(ma_truong + "asava")` |

### 1.2. Sơ đồ luồng dữ liệu

```
[Camera Dahua - SD Card]
     │  HTTP CGI API (findFile / RPC_Loadfile)
     ▼
[CameraPoller - mỗi 15s]
  - Quét video hôm nay 00:00 → now-2min
  - Lọc giờ: chỉ giữ 06:00–10:00
  - Dedup: camera_name + start_time
     │
     ▼ tạo job
[MongoDB: collection jobs]
  status: pending → processing → completed/failed
     │
     ▼ scan mỗi 5s
[VideoProcessor]
  Phase 1: Download song song 4 thread → /tmp
  Phase 2: GPU Pool 3 tiến trình
     - Decode frame @ 1fps (PyAV)
     - InsightFace detect (det_size 1600×1600)
     - Filter: confidence ≥ 0.70, size ≥ 30px
     - Recognize: cosine vs model.pkl, threshold 0.52
     - Greedy Assignment (1 người/1 frame)
     - Dedup 2 lớp → crop mặt + thân
     │                │
     ▼                ▼
[MongoDB:           [Cloudflare R2]
 recognition_        ảnh crop JPG
 history]            {school}/{date}/{person}/
     │
     ▼ sau mỗi batch
[SyncService]
  POST → https://cameraai.vnaisoft.net/api/sync/recognition-history
```

### 1.3. Cơ chế tự phục hồi

| Cơ chế | Ngưỡng | Hành vi |
|---|---|---|
| **Watchdog thread** | `watchdog_timeout_minutes` | Không có heartbeat → `os._exit(1)` → Docker `restart: always` khởi động lại |
| **Batch timeout** | 20 phút | Batch chạy quá lâu → force restart container |
| **Retry lỗi tạm thời** | 10 phút | Lỗi mạng/timeout → requeue job thay vì `mark_failed` |
| **Stuck job** | `stuck_job_timeout_minutes: 15` | Job kẹt `processing` → reset về `pending` |
| **Cô lập GPU** | mỗi job 1 subprocess | 1 job crash không kéo sập cả worker |

---

## 2. "Train model" thực chất là gì

> **Quan trọng:** Trong toàn bộ dự án **không có backpropagation, không có gradient,
> không có PyTorch training loop**. Mạng neural là model **đã được InsightFace train sẵn**,
> tải về dùng nguyên trạng.

Bước gọi là "train" thực chất là **Enrollment / Gallery Building** — xây danh bạ vector đặc trưng.

### 2.1. Quy trình 6 bước

1. **Tải ảnh** chân dung học sinh & giáo viên từ R2 (song song 10 thread, retry 3 lần/ảnh)
2. **Auto-rotate** ảnh về đúng chiều — `services/face_trainer.py:59`
   - Thử 4 góc `0° / 90° / 180° / 270°`
   - Chọn góc có **mũi nằm dưới mắt** + **2 mắt ngang nhất**
   - Ưu tiên giữ nguyên 0° nếu đã đạt (`eye_y_diff / eye_dist < 0.15`)
3. **Lọc chất lượng** — `services/face_trainer.py:215`
   - `det_score ≥ 0.70` (env `MIN_CONFIDENCE`)
   - Đủ 5 landmark (ngũ quan)
   - `eye_y_diff / eye_dist ≤ 0.30` → loại mặt nghiêng
   - `nose_offset ≤ 0.40` → loại mặt quay ngang
4. **Rút embedding** 512-D bằng ArcFace (`buffalo_l`), chuẩn hoá L2
5. **Chọn ảnh tốt nhất** — mỗi người **chỉ giữ 1 embedding** của ảnh có confidence cao nhất (`num_images: 1`)
6. **Đóng gói + upload**
   - `model.pkl` (pickle protocol 4): `{school_code, persons[{person_id, person_type, mean_embedding}], trained_at, version}`
   - Upload R2 → `{ma_truong}/models/face_model_v{timestamp}.pkl`

### 2.2. Chính sách retrain

`model_trainer/main.py:108`

```
Nếu có ≥ 1 người (học sinh HOẶC giáo viên) với is_train = false/null
    → TRAIN LẠI TOÀN BỘ trường đó
Nếu tất cả is_train = true
    → BỎ QUA trường đó
```

Sau khi train xong, đánh dấu `is_train = true` cho **cả người train thành công lẫn người bị
skip** (để lần sau không lặp lại vô ích).

**Lịch chạy:** chạy ngay khi khởi động + mỗi ngày **02:00**.

### 2.3. Cơ chế cập nhật model xuống worker

`workers/video_processor.py:544`

```
model_trainer  →  upload R2: {ma_truong}/models/face_model_v{ts}.pkl
                       │
                       ▼ (worker poll định kỳ theo model_check_interval_minutes)
face_worker_single  →  list_objects_v2, so LastModified
                    →  nếu key mới ≠ key đang dùng → download
                    →  reset _model_cache = None → subprocess reload
```

> ⚠️ **Đính chính tài liệu:** `LOCAL_DEPLOY_README.md` ghi đường dẫn R2 là
> `models/{ma_truong}/model.pkl` — **không chính xác**. Đó là đường dẫn **local trong container**.
> Key R2 thật là `{ma_truong}/models/face_model_v{timestamp}.pkl`
> (khớp giữa `model_trainer/services/r2_service.py:45` và `face_worker_single/services/s3_service.py:67`).

---

## 3. Setup model AI với card RTX trên Linux

### 3.1. Chuẩn bị host (một lần)

```bash
# Driver NVIDIA ≥ 525
nvidia-smi

# NVIDIA Container Toolkit
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Xác nhận Docker thấy GPU
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu22.04 nvidia-smi

# MongoDB trên host
sudo systemctl start mongod && sudo systemctl enable mongod
mongosh --eval "db.adminCommand('ping')"
```

### 3.2. Worker nhận diện (`face_worker_single`) — ✅ đã cấu hình đúng

| Cấu hình | Giá trị | Lý do |
|---|---|---|
| Base image | `nvidia/cuda:12.4.1-cudnn-runtime-ubuntu22.04` | có sẵn CUDA + cuDNN runtime |
| GPU | `deploy.resources.reservations.devices: driver nvidia` | cấp GPU cho container |
| Env | `CUDA_VISIBLE_DEVICES=0` | chỉ định RTX 3060 |
| **`shm_size`** | **`8gb`** | **bắt buộc** — `spawn` Pool 3 tiến trình cần shared memory |
| Volume `trt_cache` | `/root/.config/insightface` | tránh rebuild TensorRT engine mỗi lần restart |
| Volume `insightface_models` | `/root/.insightface` | cache model buffalo_l |
| Network | `network_mode: host` | để với tới camera trong LAN |

```bash
cd face_worker_single
cp config.yaml.example config.yaml   # điền IP cam, mật khẩu, R2, mã trường
python3 test_local.py                # kiểm tra môi trường
docker compose up -d --build
docker exec face-worker-single nvidia-smi   # xác nhận GPU trong container
docker logs -f face-worker-single
```

**Tuning VRAM cho RTX 3060 12GB:**

| Tham số | Giá trị hiện tại | Khi OOM |
|---|---|---|
| `det_size` | `[1600, 1600]` | hạ xuống `[1280, 1280]` |
| `concurrent_jobs` | `3` | hạ xuống `2` |
| FAISS | CPU `IndexFlatIP` | giữ nguyên — cố ý để CPU tiết kiệm VRAM |

### 3.3. Trainer (`model_trainer`) — ⚠️ **hiện chưa chạy được GPU trong Docker**

Có **4 điểm lệch** cần xử lý trước khi deploy trainer bằng Docker trên Linux:

| # | Vấn đề | File | Hiện tại | Cần sửa |
|---|---|---|---|---|
| 1 | Base image không có CUDA | `model_trainer/Dockerfile:1` | `python:3.11-slim` | `nvidia/cuda:12.4.1-cudnn-runtime-ubuntu22.04` |
| 2 | Không cấp GPU cho container | `model_trainer/docker-compose.yml:17-24` | khối `deploy.devices` đang **comment** | bỏ comment |
| 3 | Code ép GPU, không fallback | `services/face_trainer.py:52` | `providers=['CUDAExecutionProvider']` | giữ nguyên **nếu** đã sửa 1+2 |
| 4 | `setup_gpu()` chỉ dành cho Windows | `model_trainer/main.py:35` | `os.add_dll_directory`, path `C:/Program Files/...` | trên Linux là no-op vô hại, có thể bỏ qua |

**Hệ quả:** với cấu hình hiện tại, `docker compose up` trên trainer sẽ **crash ngay** ở
`_load_model()` vì `onnxruntime-gpu` không tìm thấy CUDA provider.

**Phương án thay thế — chạy trainer bằng CPU:**

```diff
# model_trainer/requirements.txt
- onnxruntime-gpu>=1.16.0
+ onnxruntime>=1.16.0
```
```diff
# model_trainer/services/face_trainer.py:52
- providers=['CUDAExecutionProvider'],
+ providers=['CPUExecutionProvider'],
```

Chấp nhận được vì trainer dùng `det_size=(640,640)` và chỉ chạy 1 lần/ngày. Nhưng với vài
nghìn học sinh sẽ chậm hơn nhiều lần (xem [mục 5.1](#51-chỗ-1--model_trainer-bước-gọi-là-train)
về việc auto-rotate nhân 4–5 lần tải).

### 3.4. Khởi động trainer

```bash
cd model_trainer
cp .env.example .env      # MONGO_URI, SCHOOL_CODE, R2_*
nano schools_config.json  # danh sách trường + mongodb_uri
docker compose up -d --build
docker logs -f model_trainer

# Chạy ngay không chờ lịch 02:00:
docker exec model_trainer python -u main.py --now
```

---

## 4. Cách lấy hình vào để train

**Ảnh không nằm trong repo** — chúng đến từ MongoDB + R2.

### 4.1. Luồng đưa ảnh vào

```
Người dùng upload ảnh chân dung (web/app)
        ↓
Upload lên Cloudflare R2  →  uploads/{ma_truong}/faces/{person_id}/xxx.jpg
        ↓
POST /api/check-face-quality  (photo_recognition_web)
   → kiểm tra det_score ≥ 0.9 trước khi chấp nhận
        ↓
Lưu URL/key vào MongoDB, DB = mã trường:
   camera_ai_hoc_sinh_col.face_hoc_sinh   = [url1, url2, ...]   (học sinh)
   camera_ai_giao_vien_col.face_giao_vien = [url1, url2, ...]   (giáo viên)
   + đặt is_train = false   ← cờ kích hoạt train lại
        ↓
model_trainer (02:00 hàng ngày) đọc 2 collection này
```

### 4.2. Cấu trúc dữ liệu

**Mỗi trường = 1 database MongoDB riêng**, tên database chính là `ma_truong`.
Xem `model_trainer/schools_config.json`:

```json
{
  "mongodb_uri": "mongodb://202.78.227.119:27017/",
  "schools": [
    { "ma_truong": "79778408", "ten_truong": "Trường TH Lê Văn Tám", "enabled": true },
    { "ma_truong": "74724417", "ten_truong": "Trường TH Đoàn Thị Điểm", "enabled": true }
  ],
  "collection_name": "camera_ai_hoc_sinh_col",
  "models_collection": "camera_ai_models_col"
}
```

| Loại người | Collection | Field chứa ảnh | Field ID |
|---|---|---|---|
| Học sinh | `camera_ai_hoc_sinh_col` | `face_hoc_sinh[]` | `ma_hoc_sinh` |
| Giáo viên | `camera_ai_giao_vien_col` | `face_giao_vien[]` | `ma_giao_vien` |

Điều kiện lọc chung: `status_del = 1` (đang hoạt động) và mảng ảnh không rỗng.

### 4.3. Xử lý ảnh khi train

- Tải **song song 10 thread**, **retry 3 lần/ảnh**
- Ảnh nhỏ hơn 640px được **upscale `INTER_CUBIC`** trước khi detect — `services/face_trainer.py:192`
- Ảnh gốc lưu lại vào `model_trainer/data/downloads/{ma_truong}/{person_id}/` để kiểm tra thủ công
- Thư mục cũ bị **xoá sạch** trước mỗi lần train lại (full retrain)

### 4.4. Báo cáo lỗi — file cần xem để biết ai phải chụp lại ảnh

| File | Nội dung |
|---|---|
| `no_faces_{ma_truong}.csv` | Người không train được + **lý do cụ thể** |
| `no_faces_{ma_truong}.json` | Bản JSON của trên |
| `failed_images_{ma_truong}.csv` | Từng ảnh lỗi + confidence + góc xoay đã thử |

Vị trí: `model_trainer/data/downloads/models/{ma_truong}/`

### 4.5. Thêm ảnh cho học sinh mới / chụp lại

1. Upload ảnh lên R2 (nên qua `/api/check-face-quality` để đảm bảo `det_score ≥ 0.9`)
2. Push URL vào mảng `face_hoc_sinh` của document học sinh đó
3. **Đặt `is_train: false`** cho document đó
4. Chờ 02:00 hoặc chạy `--now` → trường đó được train lại toàn bộ, model mới lên R2
5. Worker tự phát hiện model mới và reload

### 4.6. Yêu cầu ảnh để pass filter

| Tiêu chí | Ngưỡng |
|---|---|
| Detection score | `≥ 0.70` (trainer) / `≥ 0.9` (API kiểm tra trước) |
| Landmark | đủ 5 điểm (2 mắt, mũi, 2 khoé miệng) |
| Độ nghiêng mặt | `eye_y_diff / eye_dist ≤ 0.30` |
| Mũi lệch tâm | `nose_offset ≤ 0.40` |

**Ảnh sẽ bị loại:** nghiêng nhiều, đeo khẩu trang, mờ, chụp từ xa, thiếu sáng, che khuất ngũ quan.

---

## 5. Vai trò thực sự của card RTX

> **Trả lời ngắn gọn:** RTX **không train** gì cả — nó chỉ chạy **inference (suy luận)**.

Mạng neural (`buffalo_l` — SCRFD detect + ArcFace embedding) đã được huấn luyện sẵn bởi
InsightFace. RTX chỉ làm một việc: **đẩy ảnh qua mạng neural đó thật nhanh** để lấy ra
vector 512 chiều.

### 5.1. Chỗ 1 — `model_trainer` (bước gọi là "train")

**Tải nhẹ, chạy 1 lần/ngày.** GPU dùng để rút embedding từ ảnh chân dung
(`services/face_trainer.py:52`, `det_size=(640,640)`).

Điểm ít ai để ý: hàm `auto_rotate_face` gọi `self.app.get()` **tối đa 4 lần mỗi ảnh**
(thử 0°/90°/180°/270°), rồi `extract_embedding` gọi thêm 1 lần nữa
→ **tới 5 lượt inference cho 1 tấm ảnh**.

Với vài nghìn học sinh × nhiều ảnh/người, **đây mới là lý do trainer cần GPU**, chứ không
phải vì "training" nặng.

Kết quả cuối cùng chỉ là gom các vector lại thành `model.pkl` — thao tác **pickle thuần CPU**,
không dính GPU.

### 5.2. Chỗ 2 — `face_worker_single` (nhận diện video)

**Đây mới là lý do thật sự phải mua RTX 3060.** Tải nặng gấp hàng trăm lần:

| Yếu tố | Giá trị | Ý nghĩa với GPU |
|---|---|---|
| `det_size` | **1600×1600** | gấp ~6× diện tích so với 640×640 của trainer |
| Tần suất | **1 frame/giây** × mỗi video | hàng nghìn lượt detect mỗi ngày |
| Song song | **3 tiến trình** (`concurrent_jobs`) | 3 bản InsightFace cùng nằm trong VRAM |
| Nguồn | 4 camera × khung 6:00–10:00 | chạy liên tục cả buổi sáng |

Chính vì vậy compose phải đặt `shm_size: 8gb` (do `spawn` Pool) và mount `trt_cache` để khỏi
build lại engine mỗi lần restart.

### 5.3. Bảng phân chia GPU / CPU

| Công đoạn | Chạy ở đâu | Ghi chú |
|---|---|---|
| Detect khuôn mặt (SCRFD) | 🟢 **GPU** | phần tốn nhất |
| Rút embedding 512-D (ArcFace) | 🟢 **GPU** | |
| Auto-rotate ảnh khi train | 🟢 **GPU** | ×4 lượt/ảnh |
| Decode video `.dav` | 🔴 CPU | PyAV, **không dùng NVDEC** trong luồng worker |
| So khớp cosine (FAISS) | 🔴 CPU | `IndexFlatIP` **cố ý** để CPU tiết kiệm VRAM |
| Crop ảnh, tính sharpness (OpenCV) | 🔴 CPU | |
| Dedup, upload R2, ghi MongoDB | 🔴 CPU / mạng | |
| Đóng gói `model.pkl` | 🔴 CPU | |

### 5.4. Hệ quả thực tế

- Máy chạy **trainer** có thể dùng GPU yếu hơn, hoặc chuyển hẳn sang CPU — chấp nhận được
  vì chỉ chạy 02:00 mỗi ngày.
- Máy chạy **worker** thì GPU là **bắt buộc** — code chủ động kiểm tra và **báo lỗi ngay nếu
  thiếu CUDA**, cố tình không cho fallback CPU (`workers/video_processor.py:281-287`).
- Muốn tăng throughput video: tăng `concurrent_jobs` (giới hạn bởi VRAM), hoặc dùng **NVDEC**
  để đẩy phần decode sang GPU — code mẫu đã có sẵn ở `step1_nvcodec_gpu.py` nhưng
  **chưa được tích hợp** vào worker.

---

## 6. Công nghệ chính của dự án

Gọi tên cho đúng: đây là **1:N Face Identification trên video pipeline dạng batch**.

### 6.1. Bốn khối lõi

| Khối | Công nghệ cụ thể | Vai trò |
|---|---|---|
| **Detection** | SCRFD-10GF (trong `buffalo_l`) | tìm bbox + 5 landmark |
| **Feature extraction** | ArcFace ResNet50 (`w600k_r50`) | ảnh mặt → vector 512-D chuẩn hoá L2 |
| **Matching** | Cosine similarity + FAISS `IndexFlatIP` | so 1 vector với toàn bộ gallery, ngưỡng 0.52 |
| **Enrollment** | trích embedding ảnh chân dung → `model.pkl` | "train" = xây gallery, **không phải học máy** |

### 6.2. Hạ tầng bao quanh

- **ONNX Runtime + CUDA Execution Provider** — engine inference
- **`multiprocessing spawn` Pool** — cô lập VRAM từng job
- **MongoDB** — hàng đợi job + lưu lịch sử nhận diện
- **Cloudflare R2** — object storage cho model và ảnh crop
- **Docker + watchdog + auto-restart** — tự phục hồi

### 6.3. Đặc điểm kiến trúc quyết định

Hệ thống **pull file video đã ghi từ thẻ nhớ camera** rồi lấy mẫu **1 frame/giây**, chứ không
xử lý luồng RTSP realtime.

Đây là lựa chọn **có chủ đích**:

| Ưu điểm | Nhược điểm |
|---|---|
| Bền với mạng chập chờn | Độ trễ cao (không realtime) |
| Xử lý lại được khi cần | Mỗi người chỉ lọt 2–3 frame → không tổng hợp theo thời gian được |
| Không mất dữ liệu khi worker chết | Phụ thuộc dung lượng thẻ nhớ camera |
| Kiểm soát tải GPU dễ | Không cảnh báo tức thời được |

### 6.4. Đánh giá thẳng

Stack này là **chuẩn mực của giai đoạn ~2021–2022**. Nó hoạt động được, đã chạy production,
và các con số cấu hình (det_size 1600, threshold 0.52, dedup 2 lớp) cho thấy đã qua tinh chỉnh
thực tế nghiêm túc.

Nhưng so với mặt bằng hiện nay, nó thiếu **3 lớp quan trọng**:
1. **Tracking** (tổng hợp theo thời gian)
2. **Quality-aware aggregation** (đo chất lượng ảnh mặt)
3. **Anti-spoofing** (chống giả mạo)

---

## 7. Best practice hiện nay cho camera AI

### 7.1. Thay đổi lớn nhất: từ "nhận diện theo frame" sang "nhận diện theo tracklet"

Đây là khác biệt căn bản giữa hệ thống cũ và hệ thống hiện đại, và cũng là **khoảng cách lớn
nhất** của dự án này.

```
Cách hiện tại:
  frame → detect → embed → match ngay → 1 quyết định/frame
  (mỗi frame là một canh bạc độc lập)

Best practice:
  frame → detect → TRACK (ByteTrack/BoT-SORT) → gom tracklet
        → chấm điểm chất lượng từng khuôn mặt trong tracklet
        → gộp embedding có trọng số chất lượng
        → MỘT quyết định cho cả tracklet (bỏ phiếu theo thời gian)
```

**Lợi ích cụ thể với bài toán cổng trường:** một học sinh đi qua camera xuất hiện trong 20–30
frame. Hệ thống hiện tại lấy mẫu 1fps nên chỉ thấy 2–3 frame và mỗi frame quyết định riêng lẻ;
nếu 1 frame nhận nhầm thì sinh ra record sai. Với tracklet, 30 frame cùng bỏ phiếu — nhận nhầm
gần như bị triệt tiêu, và trường hợp "đi nhanh quá chỉ lọt 1 frame mờ" cũng được cứu.

> Cơ chế **Greedy Assignment** hiện có (`workers/video_processor.py:350`) thực chất là một
> phiên bản thu nhỏ, thô sơ của ý tưởng này — chỉ ràng buộc trong phạm vi 1 frame chứ **chưa
> có chiều thời gian**.

### 7.2. Bảng đối chiếu theo từng hạng mục

| Hạng mục | Dự án hiện tại | Best practice hiện nay | Mức chênh |
|---|---|---|---|
| Backbone nhận dạng | ArcFace R50 (`buffalo_l`) | **AdaFace** hoặc ArcFace huấn luyện trên WebFace12M/Partial-FC | 🔴 Lớn |
| Tổng hợp theo thời gian | **Không có** | Tracking + quality-weighted fusion | 🔴 Lớn |
| Đo chất lượng ảnh mặt | Laplacian variance (chỉ ghi log) | **CR-FIQA / SER-FIQ / MagFace norm** | 🔴 Lớn |
| Chống giả mạo (PAD) | **Không có** | Silent-face anti-spoofing, hoặc camera IR/depth | 🔴 Lớn |
| Enrollment | **1 embedding/người** | Multi-template (5–10 vector), match = max | 🟠 Vừa |
| Ngưỡng quyết định | Cố định `0.52` toàn hệ thống | Hiệu chỉnh theo **FMR mục tiêu** trên chính gallery + score normalization | 🟠 Vừa |
| Decode video | PyAV (CPU) | NVDEC / DeepStream (GPU) | 🟠 Vừa |
| Inference engine | ONNX Runtime CUDA EP | **TensorRT FP16** + batching | 🟠 Vừa |
| Vector search | FAISS Flat CPU | Vẫn đúng ở quy mô vài nghìn người | 🟢 Ổn |
| Kiến trúc triển khai | Docker + watchdog + queue | Thiết kế tốt | 🟢 Ổn |

### 7.3. Bốn điểm cần nói rõ thêm

#### a) AdaFace đáng chú ý nhất với bài toán này

Nó được thiết kế riêng cho ảnh **chất lượng thấp** — đúng đặc thù camera an ninh: mặt nhỏ,
mờ do chuyển động, ngược sáng, góc nghiêng. `buffalo_l` được huấn luyện thiên về ảnh chất
lượng khá, nên khi gặp mặt 30–50px ở sân trường thì độ chính xác **tụt nhanh**.

Đây là thứ đổi được mà **không cần sửa kiến trúc** — chỉ thay model và train lại gallery.

#### b) Anti-spoofing là lỗ hổng đáng lo với hệ thống điểm danh

Hiện tại **giơ ảnh in trên điện thoại trước camera là hệ thống chấm công bình thường**. Với
môi trường trường học rủi ro này thấp, nhưng cần biết là nó tồn tại và cần được ghi nhận
trong đánh giá rủi ro.

#### c) Hiệu chỉnh ngưỡng theo FMR thay vì chọn số cảm tính

Best practice là: lấy chính gallery của trường, tính phân phối điểm giữa các cặp *khác người*,
rồi đặt ngưỡng tại mức sai số chấp nhận được (ví dụ **FMR = 1e-5**).

Ngưỡng `0.52` có thể **quá lỏng với trường 2000 học sinh** và **quá chặt với trường 200** —
vì xác suất trùng khớp nhầm **tăng theo kích thước gallery**. Hiện tại tất cả 6 trường trong
`schools_config.json` đều dùng chung một ngưỡng.

#### d) DeepStream là chuẩn công nghiệp cho video analytics đa camera trên NVIDIA

Gộp cả decode NVDEC + inference TensorRT + tracking vào một pipeline. Nhưng nó **nặng và phức
tạp**; với 4 camera và mô hình batch như hiện tại thì **chưa cần thiết**.

### 7.4. Về pháp lý và quyền riêng tư

Dữ liệu sinh trắc học của **trẻ em** thuộc nhóm **dữ liệu cá nhân nhạy cảm** theo
**Nghị định 13/2023/NĐ-CP** về bảo vệ dữ liệu cá nhân.

| Khía cạnh | Trạng thái hiện tại | Ghi chú |
|---|---|---|
| Xử lý on-premise | ✅ Điểm cộng | GPU + MongoDB đặt tại trường |
| Ảnh crop trên R2 | ⚠️ Cần rà lại | chính sách mã hoá + phân quyền truy cập |
| Embedding template | ⚠️ Cần rà lại | `model.pkl` là pickle thuần, không mã hoá |
| Thời hạn lưu trữ | ⚠️ Chưa thấy định nghĩa | cần chính sách xoá dữ liệu cũ |
| Credentials trong repo | 🔴 **Rủi ro** | R2 key hardcode trong `config.yaml.example` và `config/settings.py` |

---

## 8. Lộ trình cải tiến đề xuất

Xếp theo **hiệu quả trên công sức bỏ ra**:

### Ưu tiên 1 — Multi-template enrollment 🟢 dễ, lợi ích ngay

Sửa `face_trainer.py` để giữ **5–10 embedding/người** thay vì 1, match lấy **điểm cao nhất**.

- Vài chục dòng code, **không đụng kiến trúc**
- Ăn ngay độ chính xác — xử lý được thay đổi kiểu tóc, kính, góc chụp
- Tên field `mean_embedding` cho thấy đây **vốn đã là ý định thiết kế ban đầu** nhưng chưa
  được hiện thực
- Liên quan trực tiếp tới bài toán sinh đôi (xem `TWIN_DETECTION_SOLUTION.md`)

### Ưu tiên 2 — Hiệu chỉnh lại ngưỡng theo dữ liệu thật 🟢 dễ

Không viết code mới, chỉ cần một script phân tích phân phối điểm trên gallery hiện có, tính
ngưỡng riêng cho từng trường theo quy mô.

### Ưu tiên 3 — Đổi sang AdaFace 🟠 vừa

Thay model + train lại gallery. Hiệu quả rõ rệt với mặt nhỏ/mờ ở khoảng cách xa.

### Ưu tiên 4 — Thêm tracking + tổng hợp tracklet 🔴 khó, lợi ích lớn nhất

- Phải sửa vòng lặp frame trong `video_processor.py`
- Cần **tăng tần suất lấy mẫu** — 1fps hiện tại quá thưa để track ổn định (cần ≥ 5fps)
- Kéo theo tăng tải GPU đáng kể → có thể phải làm sau ưu tiên 5

### Ưu tiên 5 — NVDEC + TensorRT 🟠 vừa

Thuần tăng tốc, làm khi hàng đợi bắt đầu nghẽn. Code mẫu NVDEC đã có sẵn ở
`step1_nvcodec_gpu.py`.

---

> **Khuyến nghị:** Ba việc đầu (ưu tiên 1–3) có thể làm được mà **không phá vỡ kiến trúc batch
> hiện tại**. Nên bắt đầu từ đó, đo lại độ chính xác, rồi mới quyết định có đầu tư vào tracking
> (ưu tiên 4) hay không.

---

## 9. Danh sách vấn đề cần sửa

Tổng hợp các điểm phát hiện trong quá trình phân tích:

| # | Mức độ | Vấn đề | Vị trí |
|---|---|---|---|
| 1 | 🔴 Chặn | Trainer Dockerfile dùng `python:3.11-slim` nhưng code ép `CUDAExecutionProvider` → crash khi chạy Docker | `model_trainer/Dockerfile:1` |
| 2 | 🔴 Chặn | Khối GPU trong compose của trainer đang bị comment | `model_trainer/docker-compose.yml:17-24` |
| 3 | 🟠 Rủi ro | R2 credentials hardcode làm giá trị mặc định trong source | `model_trainer/config/settings.py:60-64`<br>`face_worker_single/config.yaml.example` |
| 4 | 🟠 Thiết kế | Mỗi người chỉ 1 embedding dù upload nhiều ảnh; field tên `mean_embedding` gây hiểu nhầm | `services/face_trainer.py:570` |
| 5 | 🟡 Tài liệu | `LOCAL_DEPLOY_README.md` ghi sai đường dẫn R2 của model | `LOCAL_DEPLOY_README.md:244` |
| 6 | 🟡 Code | Code chết: 2 dòng `return None` không bao giờ chạy tới sau vòng retry | `services/face_trainer.py:211-213` |
| 7 | 🟡 Vận hành | Ngưỡng `0.52` dùng chung cho cả 6 trường bất kể quy mô gallery | `config.yaml.example` |
| 8 | 🟡 Chưa dùng | Tham số được truyền vào subprocess nhưng **không hề được đọc**: `max_yaw_angle`, `max_pitch_angle`, `min_face_sharpness`, `margin_threshold`, `body_output_size`, `body_height_ratio`. Worker chỉ dùng `det_size`, `face_confidence`, `recognition_threshold`, `min_face_size`, `jpeg_quality` | `workers/video_processor.py:688-699` vs `:213-219` |
| 9 | 🟠 Tài liệu sai | `LOCAL_DEPLOY_README.md` mô tả bộ lọc `yaw ≤ 45°, pitch ≤ 35°` — **thực tế không được cài đặt** trong code nhận diện | `LOCAL_DEPLOY_README.md:334` |
| 10 | 🟡 Cấu hình | `use_clothing_check`, `temporal_window_seconds` có trong `config.yaml` nhưng không được truyền xuống subprocess xử lý | `config.yaml.example` |

---

*Tài liệu được tạo từ phân tích mã nguồn thực tế của repository.*
