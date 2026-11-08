# Dựng server cho client Tân Thiên Long (VNG 3.71) trên nền Skynet

> Tài liệu kỹ thuật ghi lại toàn bộ quá trình: mục tiêu, kiến trúc, techstack, giao thức, các bước đã làm, **bước hiện tại** và **việc tiếp theo**.
> Cập nhật: 2026-11-08. Mã nguồn server và công cụ nằm ở repo `E:\games\372dragon` (commit `47e18476`, `f85d8ff0`).

**Trạng thái nhanh:**

| Giai đoạn | Trạng thái |
|---|---|
| Đăng nhập, tạo nhân vật, chọn nhân vật | ✅ Chạy trên server thật |
| Kết nối game server, vào bản đồ (CGConnect → GCEnterScene) | ✅ Chạy trên server thật |
| Hiện nhân vật và giữ client không crash sau khi vào bản đồ | ▶ **ĐANG LÀM**: gói `GCCharBaseAttrib` |
| Vật phẩm, túi đồ, trang bị (cấu trúc item 244 byte) | ⏳ Tiếp theo |
| Di chuyển, chat, NPC, nhiệm vụ, kỹ năng… | ⏳ Sau đó |

---

## Mục lục

1. [Mục tiêu và cách tiếp cận](#1-mục-tiêu-và-cách-tiếp-cận): kèm mục 1.1 về 3 nguồn mã và khi nào dùng
2. [Kiến trúc tổng thể](#2-kiến-trúc-tổng-thể): nguyên tắc thiết kế, các tầng, process/service, vòng đời phiên, lưu trữ, môi trường
3. [Techstack](#3-techstack): kèm phiên bản cụ thể (mục 3.5)
4. [Client Tân Thiên Long 3.71](#4-client-tân-thiên-long-371)
5. [Giao thức mạng](#5-giao-thức-mạng)
6. [Server Skynet: kiến trúc chi tiết](#6-server-skynet-kiến-trúc-chi-tiết)
7. [Cơ sở dữ liệu và hạ tầng](#7-cơ-sở-dữ-liệu-và-hạ-tầng)
8. [Bộ công cụ dịch ngược và kiểm thử](#8-bộ-công-cụ-dịch-ngược-và-kiểm-thử)
9. [Nhật ký các bước đã làm](#9-nhật-ký-các-bước-đã-làm)
10. [Bước hiện tại](#10-bước-hiện-tại)
11. [Việc tiếp theo](#11-việc-tiếp-theo)
12. [Quy trình làm việc hằng ngày](#12-quy-trình-làm-việc-hằng-ngày)
13. [Lưu ý và rủi ro](#13-lưu-ý-và-rủi-ro)
14. [Phụ lục](#14-phụ-lục)

---

## 1. Mục tiêu và cách tiếp cận

**Mục tiêu:** chạy client chính thức **Tân Thiên Long (TTL) của VNG, bản 3.71.5720** (`E:\games\TLBB\TanThienLong`) trên một server tự dựng. Sau đó sửa dần từng tính năng của server cho khớp client.

**Vì sao chọn server Skynet của 372dragon:**

- Client TTL chỉ có bản build, không có mã nguồn server đi kèm. Phải dùng một server có sẵn rồi sửa giao thức cho khớp.
- Đã xem xét ba nguồn: `E:\games\372dragon` (server Skynet + Lua), `E:\games\skynet` (chỉ có framework Skynet gốc), `E:\games\1_source_cpp_pap` (không có mã server game).
- Server Skynet của 372dragon gần nhất: cùng dòng TLBB, phục vụ client "GL" (CY 3.9xxx) có bố cục gói gần giống TTL. Logic game viết bằng Lua nên sửa nhanh, không phải biên dịch lại.

### 1.1 Ba nguồn mã: công dụng và khi nào dùng

| | `E:\games\372dragon` | `E:\games\skynet` | `E:\games\1_source_cpp_pap` |
|---|---|---|---|
| **Là gì** | Dự án TLBB đầy đủ: client Windows x64 (từ nền mã TLBB 0.85) + **server game Skynet/Lua** + công cụ, test, tài liệu | Framework **Skynet gốc** của cloudwu (bản mới, v1.8.0+, Lua 5.5 sửa đổi) | Dự án "PAP" (2013–2014): viết lại kiến trúc kiểu TLBB bằng C++ theo Google style, chú thích tiếng Trung |
| **Có server game chạy được?** | **Có**: login, gate, agent, 198 bản đồ, script nhiệm vụ, MongoDB | Không: chỉ là nền tảng actor + ví dụ | Không: `server/login`, `server/server`, `server/world` chỉ có README; mới có khung `billing`, `sharememory` (CMake) |
| **Nội dung chính** | `Server/GameServer` (Skynet v1.4.0 nhúng sẵn trong `framework/`), `Server/Game_2` (Script, Scene), `Src/Client` + `Src/Engine` (client C++), `Tools/re/ttl`, `Registry/protocol/ttl` | `skynet-src` (nhân C), `service-src`, `lualib` (`skynet.*`, socket, cluster, mongo/mysql/redis driver), `examples`, `test` | `src/client` (~720 file: khung client, vgui, CEGUI, CGAL), `src/common` (net/packet, factorymanager, base, file), `src/server` (~31 file), `tools/auto_code` (sinh mã gói) |
| **Ngôn ngữ / build** | Lua 5.4.2 + C (server, `make`); C++ MSVC x64 + CMake (client) | C + Lua, `make linux` | C++ (Visual Studio, CMake cho phần server) |
| **Vai trò trong dự án TTL** | **Nguồn chính**: server đang chạy cho client TTL, nơi sửa và commit mọi thay đổi | **Tài liệu tham khảo** cho API Skynet và driver | **Tài liệu tham khảo** cách tổ chức gói tin kiểu TLBB bằng C++ |

**Khi nào dùng cái nào:**

- **`372dragon`: dùng hằng ngày.**
  - Sửa giao thức, logic game, script, dữ liệu cấu hình cho client TTL.
  - Tra cứu codec gói GL hiện có (`packet_domains/*.lua`) để biết ý nghĩa trường khi map sang TTL.
  - Dùng công cụ RE (`Tools/re/ttl`) và lưu kết quả (`Registry/protocol/ttl`).
  - Tham khảo client x64 (`Src/Client`) khi cần hiểu một gói được client TLBB **dùng** thế nào: tên lớp gói giống TTL, nên đọc C++ ở đây nhanh hơn đọc mã máy của `Game.exe`.
- **`skynet`: chỉ dùng khi cần hiểu hoặc sửa tầng framework.**
  - Tra API: `skynet.call/send/fork`, `socket`, `cluster`, `snax`, `skynet.db.mongo`.
  - Đọc cách `netpack`/`gateserver` gốc tách frame khi sửa `lua-netpack.c`.
  - Xem bản vá lỗi framework mới hơn để cân nhắc vá ngược.
  - **Không** thay thẳng `framework/` của 372dragon bằng bản này: 372dragon chạy Skynet v1.4.0 + Lua 5.4.2. Bản mới đổi sang Lua 5.5 và đổi API, nên ~1.570 file Lua và các `luaclib/*.so` có thể hỏng.
- **`1_source_cpp_pap`: hiếm khi dùng.**
  - Tham khảo mẫu C++ cho lớp gói (`common/net/packet`: `base`, `factorymanager`, `client_tologin/asklogin`, `connect`, `heartbeat`). Bố cục Read/Write/Execute/GetPacketSize giống vtable gói trong `Game.exe`, giúp đọc mã máy dễ hơn.
  - Tham khảo `tools/auto_code` nếu muốn sinh codec gói tự động.
  - Không dùng làm server: phần server chưa được viết. Phần client chỉ có khung (theo `docs/README.md`: chưa có nhân vật, quái, bản đồ, UI).

**Quyết định của chủ dự án:**

- "Sửa thẳng": sửa trực tiếp server trong WSL, **không giữ tương thích với client GL** nữa.
- Mỗi lần client crash phải tự đóng bảng báo lỗi (CrashReport/WerFault) để máy không bị treo.

**Phương pháp:** không có tài liệu giao thức của TTL, nên mọi định dạng gói đều lấy từ việc **dịch ngược `Game.exe`** (đọc RTTI, vtable, mã máy x86). Kết quả được kiểm chứng bằng cách chạy client thật qua một relay ghi log.

---

## 2. Kiến trúc tổng thể

```
 Windows (máy dev)                                     WSL Ubuntu-22.04
┌──────────────────────────────────┐              ┌───────────────────────────────────────────┐
│ TanThienLong\Bin\Game.exe  (x86) │              │ ~/372dragon/Server/GameServer (Skynet)    │
│  - tEngine.dll / Fairy engine    │              │                                           │
│  - CEGUI (giao diện)             │              │  Process Game_tlbb_2  (zone 1)            │
│  - Patch\LoginServer.txt  ───────┼─┐            │   ├─ logind   :15733  (login)             │
└──────────────────────────────────┘ │            │   ├─ gamed    :15734  (game gate)         │
                                     │ TCP        │   ├─ msgagent (1 agent / người chơi)      │
┌──────────────────────────────────┐ │            │   ├─ world, scene, Playermanager, ...     │
│ relay.py (ghi log gói)           │◄┘            │   └─ char_db  ──► MongoDB :27018 (tlbb)   │
│  127.0.0.1:25733 ─► 15733        │──────────────►  Process Manager_tlbb_8                   │
│  127.0.0.1:25734 ─► 15734        │  localhost   │  Process DataCenter_tlbb_9                │
└──────────────────────────────────┘  forwarding  │  Process Tool_tlbb_6 (MySQL, tuỳ chọn)    │
                                                  │  Mosquitto (MQTT) :1884                   │
                                                  └───────────────────────────────────────────┘
```

**Luồng kết nối của một người chơi:**

1. Client đọc `LoginServer.txt` để lấy danh sách server, rồi kết nối **login** (qua relay 25733 → 15733).
2. Login kiểm tra tài khoản trong MongoDB, trả danh sách nhân vật, xử lý tạo/xoá/chọn nhân vật.
3. Khi chọn nhân vật, login trả `LCRetCharLogin` gồm IP, cổng game và **khoá phiên** (subid).
4. Client kết nối **game gate** (`gamed`, qua relay 25734 → 15734) và gửi `CGConnect` kèm khoá phiên + guid.
5. `gamed` kiểm tra khoá rồi giao kết nối cho một `msgagent` riêng của người chơi. Agent nạp dữ liệu nhân vật và đưa nhân vật vào scene.

### 2.1 Nguyên tắc thiết kế

| Nguyên tắc | Áp dụng trong dự án |
|---|---|
| **Client là "hộp đen" cố định** | Không sửa `Game.exe`, không patch bộ nhớ. Mọi thay đổi nằm ở phía server, client chỉ đổi file cấu hình `LoginServer.txt`. |
| **Server có thẩm quyền (authoritative)** | Server quyết định toàn bộ trạng thái: vị trí, HP, vật phẩm, kết quả chiến đấu. Client chỉ gửi yêu cầu (`CG*`/`CL*`) và hiển thị kết quả (`GC*`/`LC*`). |
| **Tách login khỏi game** | `logind` chỉ lo tài khoản và nhân vật. `gamed` + `msgagent` lo trò chơi. Hai bên nối với nhau bằng khoá phiên (subid) một lần. |
| **Actor, không chia sẻ bộ nhớ** | Mỗi service Skynet là một Lua VM độc lập, chỉ trao đổi bằng message. Một agent lỗi không kéo cả server sập theo. |
| **Codec khai báo** | Mỗi gói là một bảng Lua có `xy_id`, `bis` (đọc) và `bos` (ghi). Đổi bố cục gói chỉ là sửa một hàm, không đụng logic game. |
| **Tương thích từng bước** | Gói nào chưa map được thì chặn (`TTL_UNMAPPED`) hoặc đệm 0 kèm comment `shortcut:`. Nhờ vậy client vẫn chạy tiếp, rồi sửa dần cho đúng. |
| **Kiểm chứng bằng client thật** | Mọi bố cục suy ra từ mã máy đều phải được kiểm tra bằng relay + crash log, không tin hoàn toàn vào phân tích tĩnh. |

### 2.2 Các tầng của hệ thống

```
┌───────────────────────────────────────────────────────────────────────┐
│ Tầng 1. Trình diễn (client TTL)                                        │
│   Render 3D (Fairy/OGRE, D3D9), UI (CEGUI + LuaPlus), âm thanh (FMOD)  │
│   Bảng dữ liệu tĩnh trong *.axp (DBC)                                  │
├───────────────────────────────────────────────────────────────────────┤
│ Tầng 2. Truyền tải                                                     │
│   TCP + frame "TLBB01|id|size|idx|body"; phần thân có thể bọc key+len  │
│   (relay.py chen giữa ở môi trường dev)                                │
├───────────────────────────────────────────────────────────────────────┤
│ Tầng 3. Cổng (gate)                                                    │
│   logind (15733) · gamed (15734) · lua-netpack.c tách frame            │
├───────────────────────────────────────────────────────────────────────┤
│ Tầng 4. Codec gói                                                      │
│   services/login/packet.lua · services/game/packet_ids.lua             │
│   packet_types/*.lua · packet_domains/*.lua · lualib/net.lua           │
├───────────────────────────────────────────────────────────────────────┤
│ Tầng 5. Logic game                                                     │
│   msgagent (mỗi người chơi) · scene/scenemanager (bản đồ, quái, NPC)   │
│   world, playermanager, guild, ranking, activity, pay, ybexchange …    │
│   Script nhiệm vụ/NPC: Server/Game_2/Script (~3.900 file Lua)          │
├───────────────────────────────────────────────────────────────────────┤
│ Tầng 6. Dữ liệu                                                        │
│   dbproxy / simpledb → MongoDB 27018 (tài khoản, nhân vật)             │
│   cfgdb (bảng cấu hình game) · mysqldb → MySQL (log, công cụ)          │
│   mqttor → Mosquitto 1884 (thông báo, GM)                              │
└───────────────────────────────────────────────────────────────────────┘
```

### 2.3 Mô hình process và service

Server chạy **4 process Skynet**, mỗi process đọc một file config riêng (`config_2`, `config_6`, `config_8`, `config_9`). Mỗi file có `process_id` và `svrtype`:

| Process | `svrtype` | Vai trò |
|---|---|---|
| `Manager_tlbb_8` | `Manager` | `main.lua` khởi tạo `cluster_db_mgr`, `cluster_mgr`, `cfgdb`. Điều phối cụm, nắm danh sách node |
| `Game_tlbb_2` | `Game` | Zone chơi: login, gate, agent, scene (198 bản đồ, kịch bản trong `Game_2/Script`, dữ liệu bản đồ trong `Game_2/Scene`) |
| `DataCenter_tlbb_9` | DataCenter | Dữ liệu dùng chung giữa các zone (xếp hạng, bang hội, giao dịch…) |
| `Tool_tlbb_6` | Tool | GM tool, web admin, MySQL |

- Process không phải `Manager` chạy `cluster_agent`, đăng ký với `cluster_mgr`, rồi bật các service của mình. *(Danh sách service cụ thể theo từng process được điều phối qua cluster; chưa đọc hết, ghi theo tên service trong `services/`.)*
- Process nói chuyện với nhau qua `clusterd` của Skynet (RPC qua TCP).
- `Game_tlbb_2` dùng `thread = 64` (64 worker thread), `harbor = 0` (không dùng harbor cũ, chỉ dùng cluster).

**Service chính trong `services/`:**

| Nhóm | Service |
|---|---|
| Mạng | `login/` (logind), `game/gamed.lua`, `msgagent.lua`, `agent_poll.lua` |
| Bản đồ | `scene/`, `scenemanager.lua`, `copyscenemanager.lua` (phó bản), `dynamicscenemanager.lua`, `spancopyscenemanager.lua` (liên server) |
| Thế giới | `world/`, `playermanager.lua`, `obj.lua`, `guild/`, `ranking.lua`, `activity/`, `ybexchange/` |
| Dữ liệu | `dbproxy.lua`, `simpledb.lua`, `cfgdb.lua`, `mysqldb.lua`, `gen_serial.lua` (sinh ID) |
| Vận hành | `monitor.lua`, `xylogger.lua`, `gm_tool.lua`, `hs_admin.lua`, `web/`, `mqttor.lua`, `textfilter_mgr.lua`, `pay.lua` |
| Khác | `robot/` (bot thử tải), `register.lua`, `license_check.lua` (đã vô hiệu) |

### 2.4 Vòng đời một phiên chơi

```
Client            logind              MongoDB         gamed            msgagent          scene
  │ CLConnect ─────►│                                   │                 │                 │
  │◄──── LCRetConnect                                   │                 │                 │
  │ CLAskLogin ────►│── tìm account ──►│                │                 │                 │
  │◄──── LCRetLogin │◄─────────────────│                │                 │                 │
  │ CLAskCharList ─►│── char_list ────►│                │                 │                 │
  │◄─ LCRetCharList │                                   │                 │                 │
  │ CLAskCharLogin ►│── sinh subid, báo gamed ─────────►│ users[guid]     │                 │
  │◄ LCRetCharLogin (ip, port, key)                     │                 │                 │
  │ ════ đóng kết nối login, mở kết nối game ════        │                 │                 │
  │ CGConnect ─────────────────────────────────────────►│ so key/subid    │                 │
  │                                                     │── tạo agent ───►│── nạp nhân vật ►│
  │◄──────────────────────────── KeyExchange, GCConnect ────────────────── │                 │
  │ CGEnterScene ──────────────────────────────────────────────────────────►│── vào scene ──►│
  │◄──────────── GCEnterScene + ~220 gói trạng thái ───────────────────────│                 │
  │ CGCharMove / CGChat / ... ◄──────────► xử lý trong agent/scene ◄──────►│                 │
  │ (ngắt kết nối) ────────────────────────────────────►│── logout ──────►│── lưu Mongo     │
```

- **Trạng thái nằm ở agent:** dữ liệu nhân vật được nạp từ Mongo vào `msgagent` khi vào game và được lưu lại định kỳ hoặc khi thoát.
- **Scene sở hữu thế giới:** quái, NPC, vật rơi, tầm nhìn (AOI) và di chuyển do service scene của bản đồ đó quản lý (`lnavmesh.so` để tìm đường).
- **Phát tin (broadcast):** khi một nhân vật thay đổi, scene gửi gói `GC*` cho những người chơi trong tầm nhìn qua `net:send2client`.

### 2.5 Thiết kế lưu trữ

| Kho | Nội dung | Ghi chú |
|---|---|---|
| MongoDB `tlbb.account` | `uid`, `pwd` (MD5), `uuid`, `char_list` | Một tài liệu cho mỗi tài khoản |
| MongoDB `tlbb.character` | Toàn bộ nhân vật dưới dạng tài liệu lồng nhau (`attrib`, túi đồ, trang bị, kỹ năng, nhiệm vụ…) | Đọc/ghi qua `dbproxy`/`simpledb` |
| MongoDB (khác) | `chat_limit`, bang hội, xếp hạng… | Tuỳ service |
| Bảng cấu hình | `~/372dragon/Server/configs/<env>` (env = `publish_yu`) do `cfgdb` nạp | Vật phẩm, kỹ năng, quái, cửa hàng; nạp nóng bằng `reload_*.exp` |
| MySQL 8 | Log, dữ liệu công cụ GM | Không bắt buộc để chơi |
| Mosquitto | Không lưu, chỉ chuyển tin | |

MongoDB bật `authorization`, chỉ nghe `127.0.0.1:27018`, WiredTiger cache 0,5 GB, dữ liệu ở `~/372dragon/Server/db`.

### 2.6 Thiết kế môi trường triển khai (dev)

```
Windows 11 (host) ──────────────────────────────────────────────────────────
│  E:\games\TLBB\TanThienLong   client
│  E:\games\372dragon           repo (mã server + Tools/re/ttl + Registry)
│  scratchpad\re\               script tự động hoá PowerShell
│  python relay.py              127.0.0.1:25733/25734
│
└── WSL2 Ubuntu 22.04 (12 CPU, ~11 GB RAM) ─────────────────────────────────
    ~/372dragon/Server/
      GameServer/      mã Skynet đang chạy (bản sửa trực tiếp)
      Game_2/          Script + Scene của zone 2
      configs/         bảng cấu hình theo env
      db/              dữ liệu MongoDB
      log/<yyyymm>/    log server
```

- Luồng mã nguồn: **sửa trong WSL → test → chép về `E:\games\372dragon` → commit**. WSL là bản đang chạy, repo là bản lưu trữ.
- Khi triển khai thật: bỏ relay, trỏ `LoginServer.txt` vào IP server, bỏ biến `TTL_GAME_PORT`, đặt server Linux riêng (có thể dùng cùng bộ script `start_zone.sh`).

---

## 3. Techstack

### 3.1 Client (có sẵn, không sửa mã)

| Thành phần | Chi tiết |
|---|---|
| File chạy | `Bin\Game.exe`, PE **x86 (32-bit)**, MSVC 2012 (`MSVCR110.dll`); chạy bằng `Game.exe -fl` |
| Engine 3D | `tEngine.dll`, nền **Fairy engine** (FairyVersion 7.132, kế thừa OGRE) |
| Giao diện | **CEGUI**, layout XML trong gói tài nguyên (`*.layout.xml`) |
| Dữ liệu bảng | DBC, định dạng AXP + mã hoá XOR 128 byte |
| Cấu hình server | `Patch\LoginServer.txt`: header 96 byte + XOR 128 byte |
| Báo lỗi | `Bin\Crash\crashMM_DD_HH_MM.zip` chứa `info.txt` (biểu thức assert, call stack) và `dump.dmp` |
| Đường dẫn build gốc (lộ trong assert) | `D:\TLBBHW\Vietnam_371M\Prj\GameCode\Client\...` |

### 3.2 Server

| Thành phần | Chi tiết |
|---|---|
| Framework | **Skynet** (actor model, C + Lua), mỗi service là một Lua VM, giao tiếp bằng message |
| Ngôn ngữ logic | **Lua 5.4** (service, gói tin, script game) |
| Mã C tự viết | `lualib-src/lua-netpack.c` (tách frame TCP), build bằng `make luaclib/netpack.so` |
| Cụm process | `Game_tlbb_2` (zone), `Manager_tlbb_8`, `DataCenter_tlbb_9`, `Tool_tlbb_6`, nối với nhau qua `clusterd` |
| Điều khiển | `./start_zone.sh all | stop | status | zone N` |
| Mã ký tự | Client dùng GBK/CP1258; server chuyển qua lại bằng `wire_charset` (`gbk.fromutf8`) |
| Log | `~/372dragon/Server/log/<yyyymm>/Game2_*.log`, `Manager8_*.log`, … |

### 3.3 Hạ tầng

| Thành phần | Chi tiết |
|---|---|
| Hệ điều hành server | WSL2 `Ubuntu-22.04` trên Windows 11; cổng WSL dùng được từ Windows qua localhost forwarding |
| CSDL chính | **MongoDB**, cổng 27018, database `tlbb` (collection `account`, `character`, `chat_limit`, …) |
| Message bus | **Mosquitto (MQTT)**, cổng 1884 |
| CSDL phụ | MySQL cho service Tool (không bắt buộc để chơi) |
| License | Bước kiểm tra license của bản 372dragon đã được bỏ qua |

### 3.4 Công cụ phát triển

| Công cụ | Dùng để |
|---|---|
| Python 3 + `pefile` + `capstone` | Đọc PE, RTTI, vtable và dịch ngược mã x86 của `Game.exe` |
| PowerShell + Win32 API (`SetCursorPos`, `mouse_event`, `PrintWindow`) | Tự động điều khiển client và chụp màn hình |
| Python socket | Relay ghi log, server giả lập (mock) |
| Git | Repo `E:\games\372dragon` |

### 3.5 Phiên bản cụ thể (đo trên máy dev ngày 2026-11-08)

**Client:**

| Thành phần | Phiên bản |
|---|---|
| Tân Thiên Long (VNG) | **3.71.5720** (`Patch\patchinfo.txt`: `Latest=3.71.5720`) |
| `Launch.exe` | 1.0.5.96 (patchinfo ghi `NewLaunch=1.0.5.92`) |
| `Bin\Game.exe` | FileVersion `0, 85, 0, 0`, 15.877.664 byte, PE x86, MSVC 2012 (`MSVCR110.dll`) |
| `Bin\tEngine.dll` | 1.0.0.6 (Fairy engine, FairyVersion 7.132) |
| OGRE | `OgreMain.dll`, `OgrePlatform.dll` (không ghi version) |
| CEGUI | `CEGUIBase.dll`, `CEGUIFalagard.dll`, `CEGUIOgreRenderer.dll` (không ghi version) |
| Script UI | `LuaPlus.dll` (không ghi version; LuaPlus thường dựa trên Lua 5.1, chưa kiểm chứng) |
| DirectX | `d3dx9_43.dll`, `d3dx11_43.dll`, `D3DCompiler_43.dll` 9.29.952.3111 (DirectX SDK June 2010) |
| Âm thanh | `fmod.dll` 3.75 |
| Gói tài nguyên | `Data\*.axp` (Brushes, Config, Effect, Interface ~1,1 GB, Material 1–7, Model…), tổng cỡ hàng chục GB |

**Server:**

| Thành phần | Phiên bản |
|---|---|
| Hệ điều hành | Ubuntu **22.04.5 LTS** trên WSL **2.7.14.0**, kernel 6.18.33.2-microsoft-standard-WSL2 |
| Skynet | **v1.4.0** (2020-11-16), trong `GameServer/framework` |
| Lua (server) | **5.4.2** (đi kèm Skynet) |
| Trình biên dịch | gcc 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04.3) |
| Thư viện C của Skynet (`framework/3rd`) | jemalloc, lpeg, lua-cjson, lua-md5, lua-pbc (protobuf), lua-rsa, lua-textfilter, lua-zlib, lua-zset |
| Thư viện C riêng (`luaclib`) | `netpack.so` (đã sửa), `gbk.so`, `lnavmesh.so`, `laiscript.so`, `rax.so`, `map.so`, `lfs.so`, `ldump.so`, `lprofile.so` |
| Thư viện C++ (`3rd`) | BehaviorTree.CPP (AI quái) |
| Quy mô mã | ~1.570 file Lua trong `GameServer`, ~3.870 file script trong `Game_2/Script`, 13 domain gói, 489 dòng `packet_ids.lua` |

**Cơ sở dữ liệu và hạ tầng:**

| Thành phần | Phiên bản | Cấu hình |
|---|---|---|
| MongoDB | **4.4.31** (git `0fdaf654`) | `127.0.0.1:27018`, auth bật, WiredTiger cache 0,5 GB, `mongod.conf` trong `~/372dragon/Server` |
| Mosquitto | **2.0.11** | cổng 1884, `/etc/mosquitto/mosquitto.conf` |
| MySQL | **8.0.46**-0ubuntu0.22.04.4 | cho Tool/log |

**Công cụ phát triển (Windows):**

| Thành phần | Phiên bản |
|---|---|
| Windows | 11 Education (10.0.26300) |
| Python | **3.14.6** (Windows); 3.10.12 trong WSL |
| capstone | 5.0.7 |
| pefile | 2024.8.26 |
| PowerShell | 5.1.26100 |

---

## 4. Client Tân Thiên Long 3.71

### 4.1 Khởi động

- Chạy thẳng `Game.exe` sẽ báo "hãy chạy từ Launch.exe". Phải chạy `Bin\Game.exe -fl`.
- Màn hình đầu là danh sách server → bấm "Chọn" → màn đăng nhập.

### 4.2 `Patch\LoginServer.txt`

- 96 byte header: `SEPCIAL_FILE_HD\0`, `u32 type` (= 1), `u32 body_len` (offset 20), `u32 checksum` (offset 24, **client không kiểm tra**).
- Toàn bộ file được XOR với khoá 128 byte (`Tools/re/ttl/loginserver_key.bin`). Khoá này không nằm sẵn trong `Game.exe`; nó được khôi phục bằng thống kê tần suất theo từng cột rồi tinh chỉnh bằng mô hình bigram.
- Công cụ `lsfile.py`: `dec <in> <out>` để giải mã, `enc <template> <body> <out>` để mã hoá lại.
- Bản gốc được giữ ở `LoginServer.txt.orig`. Bản hiện tại trỏ mọi server về `127.0.0.1:25733` (relay).

### 4.3 Bảng dữ liệu (DBC)

- Định dạng AXP, mã hoá XOR 128 byte (khác khoá của LoginServer.txt).
- Là nguồn văn bản tiếng Việt và dữ liệu (vật phẩm, kỹ năng, bản đồ…). Sẽ cần khi đồng bộ dữ liệu server với client.

### 4.4 Cách lấy định dạng gói từ `Game.exe`

- Mỗi gói là một lớp C++ `Packets::<Tên>`. Tìm chuỗi RTTI `.?AV<Tên>@Packets@@`, từ đó ra Complete Object Locator rồi ra **vtable**.
- Factory của gói: `vtbl[3]` = `GetPacketID`.
- Lớp gói:

  | Slot vtable | Chức năng |
  |---|---|
  | `vtbl[1]` | Read |
  | `vtbl[2]` | Write (luồng body) |
  | `vtbl[3]` | Execute (gọi handler) |
  | `vtbl[4]` | ID |
  | `vtbl[5]` | GetPacketSize |

- Mỗi trường được ghi bằng mẫu lệnh `push len; push ptr; call [stream_vtbl+4]`. Đọc chuỗi lệnh đó ra được bố cục gói.
- Kết quả: **1.314 gói** (ID 1..9045). 750 gói ra bố cục chính xác, 56 gói kích thước cố định, 508 gói "phức tạp" (cờ bit, nén, vòng lặp) phải dịch ngược tay.

---

## 5. Giao thức mạng

### 5.1 Khung gói (frame)

```
Client → Server:  "TLBB01" (6 byte ASCII) | u16 id | u16 size | u16 idx | body[size]
Server → Client:                            u16 id | u16 size | u16 idx | body[size]
```

- Client gắn tiền tố `TLBB01` vào **mọi** frame. Server bỏ tiền tố này trong `lua-netpack.c`.
- `idx` của client tăng dần: 0x7B00, 0x7C00, 0x7D00, … Server login gửi `idx = 0`; server game gửi `idx = n * 0x100`.
- Little-endian.

### 5.2 Khác biệt lớn giữa TTL và GL (CY)

| Điểm | GL (server gốc) | TTL 3.71 |
|---|---|---|
| ID gói | Bộ ID riêng | Khác hẳn (VD CGConnect: GL 49 → TTL 281) |
| GUID nhân vật | 32-bit | **64-bit**; trên dây gửi nửa cao trước, nửa thấp sau |
| Gói "mã hoá" | Không có | Một số gói bọc dạng `u8 key, u32 len, body[len]`, client giải mã `byte -= key` (server gửi key = 0) |
| Bản ghi nhân vật | 332 byte | 420 byte |
| Cấu trúc vật phẩm | 188 byte | 244 byte |
| GCCharBaseAttrib | 14 byte cờ | Byte header + lớp mã hoá + mặt nạ cờ **128 bit** |
| Gói lạ | — | Client ném lỗi `Invalid Packet[id]` rồi thoát |
| Sai kích thước gói cố định | — | Client ném lỗi `Read Packet Size error[id]` |

### 5.3 Chuỗi gói đăng nhập (đã chạy)

| Bước | Client → Server | Server → Client | Ghi chú |
|---|---|---|---|
| 1 | `CLConnect` (355, 124 B) | `LCRetConnect` (311, 74 B) | Cuối gói có `u32` phiên bản mã hoá 3474034634 và cờ WeGame = 0 |
| 2 | `CLAskLogin` (299, độ dài thay đổi, len+927) | `LCRetLogin` (223, 83 B) + `LCStatus` (554, 6 B) | Mật khẩu: gõ `tàikhoản%mậtkhẩu` vào ô tài khoản để server so MD5 trực tiếp |
| 3 | `CLAskCharList` (711, 50 B) | `LCRetCharList` (319) | Mỗi nhân vật 420 B, cuối gói thêm 17 B |
| 4 | `CLCheckAllowCreateChar` (1250) | `LCRetCheckAllowCreateChar` (1251) | GL dùng ID 776/777 |
| 5 | `CLAskCreateChar` (522, 148 B) | `LCRetCreateChar` (440) + danh sách mới | |
| 6 | `CLAskCharLogin` (426, ~58–62 B) | `LCRetCharLogin` (38) | Thân gói bọc mã hoá: 62 B, thêm 8 B nếu result = 15 |

### 5.4 Bản ghi nhân vật 420 byte

| Offset TTL | Nội dung |
|---|---|
| 0x00 | GUID u64 |
| 0x08 | Giới tính / race (client assert phải < 2) |
| 0x0C | Tên (30 B) |
| 0x0C … 0x12C | Giống GL nhưng lùi +4 (cấp, tóc, mặt, scene, môn phái, avatar, 19 trang bị, 19 visual, 19 gem, …) |
| 0x12C … 0x188 | 92 B chưa rõ (3 int tại 0x138 đưa vào một setter của char data) |
| 0x188 | Khối ngoại trang (vũ khí, áo lưng, mũ, màu) |
| 0x198 | Marker 0x62F631C4 |
| 0x19C … 0x1A4 | 8 B đuôi |

Bảng trên lấy từ hàm thêm nhân vật vào màn chọn (`0x924120` trong `Game.exe`).

### 5.5 Chuỗi gói vào game (đã chạy tới GCEnterScene)

```
C: CGConnect (281, 140 B)   key, guid_hi, guid_lo, …, version[32], …
S: KeyExchange (67)         u16 len + 8 byte 0
S: GCConnect (643, 18 B)
C: CGEnterScene (641, 11 B)
S: GCEnterScene (410, 52 B)
S: ~220 gói trạng thái: GCMissionModify, GCUICommand, GCCharBaseAttrib, GCDetailAttrib, GCMyBagList, …
```

---

## 6. Server Skynet: kiến trúc chi tiết

Thư mục gốc trong WSL: `~/372dragon/Server/GameServer` (bản sao trong repo: `E:\games\372dragon\Server\GameServer`).

### 6.1 Tầng mạng

| File | Vai trò |
|---|---|
| `lualib-src/lua-netpack.c` | Tách frame TCP thành gói. **Đã sửa:** bỏ tiền tố `TLBB01` (`skip_ttl_prefix`) |
| `lualib/net.lua` | `net:send` / `net:send2client`: đóng gói rồi gửi xuống socket. **Đã sửa:** chặn gói ID ≥ 50000 và các gói chưa map (`TTL_UNMAPPED`) |
| `services/login/server.lua` | Gửi gói phía login |

### 6.2 Login

| File | Vai trò |
|---|---|
| `services/login/logind.lua` | Xử lý theo `xy_id`: đăng nhập, danh sách nhân vật, tạo/xoá/chọn nhân vật. **Đã sửa:** biến môi trường `TTL_GAME_PORT` để đẩy kết nối game qua relay |
| `services/login/packet.lua` | Định nghĩa gói login (`bis` = đọc từ client, `bos` = ghi ra client). **Đã sửa** theo bố cục TTL |
| `login/tlbb_decrypt.lua` | Giải mã mật khẩu CRT + RC4 (khi không dùng cú pháp `tk%mk`) |

### 6.3 Game

| File | Vai trò |
|---|---|
| `services/game/gamed.lua` | Game gate: nhận kết nối, kiểm tra `CGConnect` (khoá + guid), tạo `msgagent` |
| `services/msgagent.lua` | Agent của từng người chơi: nạp dữ liệu, xử lý `request:<TênGói>()` |
| `services/game/packet.lua` | Nạp ID + kiểu + domain gói |
| `services/game/packet_ids.lua` | Hằng `XYID_*`. **Đã ghi lại** theo ID của TTL bằng `remap_ids.py` |
| `services/game/packet_types/*.lua` | Kiểu dùng chung (guid, item, equip, team, pet…) |
| `services/game/packet_domains/*.lua` | Gói theo nhóm: `core`, `movement`, `combat`, `item`, `shop`, `team`, `mission`, `social`, `pet`, `city`, `misc` |
| `lualib/cy_four_combat/*` | Logic chiến đấu bổ sung (gói GCCharDoAction) |

**Cách gói được dispatch:** `net:register_packets` duyệt `pairs(game.packet)` và đánh chỉ mục theo giá trị `xy_id`. Vì vậy **mỗi `xy_id` phải duy nhất**: `cys_mount_flight.lua` assert khi trùng ID.

### 6.4 Cụm process

- `Game_tlbb_2`: zone chính, nạp **198 bản đồ** (mất khoảng 3–4 phút đến khi cổng 15733 mở).
- `Manager_tlbb_8`, `DataCenter_tlbb_9`, `Tool_tlbb_6`: quản lý, dữ liệu dùng chung, công cụ. Nối với nhau qua `clusterd`/`clusteragent`.
- Lúc khởi động có thể thấy lỗi `changenode … failed` hoặc `failed to connect to mysql`. Đây là lỗi tạm khi các node chưa lên đủ, không ảnh hưởng đăng nhập.

---

## 7. Cơ sở dữ liệu và hạ tầng

- MongoDB `tlbb`:
  - `account`: các trường `uid`, `pwd`, `uuid`, `char_list`.
  - `character`: tài liệu nhân vật (`attrib.guid`, `attrib.sceneid`, `attrib.world_pos`, …).
- Tài khoản test: `abc123` / `123456` (uuid 100017). Nhân vật test: `lan123`, guid 200002.
- Mật khẩu MongoDB nằm trong `confs/publish_yu/server_conf.lua`. **Không chép mật khẩu vào tài liệu.**
- Mosquitto 1884 dùng cho thông báo nội bộ.

---

## 8. Bộ công cụ dịch ngược và kiểm thử

Tất cả nằm trong `E:\games\372dragon\Tools\re\ttl\`. Dữ liệu đầu ra nằm trong `Registry\protocol\ttl\`.

| Công cụ | Chức năng |
|---|---|
| `ttl_packets.py` | Quét `Game.exe`, xuất `ttl_packets.tsv` (tên, ID, kích thước, chất lượng) và `ttl_layout.json` (danh sách trường kèm offset) |
| `skynet_compare.py` | So bố cục TTL với codec Skynet, xuất `cmp_skynet.tsv` (same / samesize / diff / complex) |
| `remap_ids.py` | Ghi lại `packet_ids.lua` theo ID TTL, khớp qua tên lớp gói. Gói không có trong TTL nhận ID từ 50000 trở lên |
| `lsfile.py` + `loginserver_key.bin` | Giải mã và mã hoá `LoginServer.txt` |
| `relay.py` | Relay TCP ghi log đầy đủ từng frame (chiều, ID, size, idx, hex body) |
| `mock_login.py` | Server login giả lập phát lại bản ghi, dùng để thử bố cục bản ghi nhân vật trong vài giây |
| `mock_world.py` | Phát lại **cả phiên** login + game. Tham số: danh sách ID cần bỏ, số frame tối đa |
| `sniff.py` | Listener chỉ ghi log (dùng ở lần dò đầu tiên) |

Script PowerShell (thư mục scratchpad của phiên làm việc, nên chép vào repo nếu dùng lâu dài):

| Script | Chức năng |
|---|---|
| `ttl_login.ps1` | Mở client, chọn server, gõ tài khoản, bấm vào game. Tuỳ chọn `-NoLaunch` để không mở lại client |
| `ttl_do.ps1` | Chuỗi thao tác: `"x,y"` = click, `"t:chữ"` = gõ phím, `"w:ms"` = chờ |
| `shot.ps1` | Chụp cửa sổ client ra PNG |
| `ttl_clean.ps1` | Đóng Game.exe, CrashReport.exe, WerFault và báo file crash mới nhất |
| `ttl_iter.ps1` | Vòng lặp: phát lại → đọc crash → thêm ID vào danh sách bỏ → lặp lại |
| `ttl_bisect.ps1` | Chia đôi số frame để tìm đúng gói gây crash khi lỗi không ghi ID (ví dụ access violation) |
| `restart.sh` | Khởi động lại server trong WSL (đặt `TTL_GAME_PORT=25734`) và chờ cổng mở |

---

## 9. Nhật ký các bước đã làm

### Bước 0: Khảo sát ✅
- Xác định client TTL 3.71.5720 chỉ có bản build, khác client tlbb129.
- So ba nguồn server và chọn Skynet của 372dragon.

### Bước 1: Dịch ngược bảng gói ✅
- Viết `ttl_packets.py`, lấy được 1.314 gói.
- Viết `skynet_compare.py`: trong 417 tên gói chung có 117 gói giống hệt, 12 cùng kích thước, 64 khác, 224 phức tạp.

### Bước 2: Trỏ client về server tự dựng ✅
- Khôi phục khoá XOR của `LoginServer.txt`, viết `lsfile.py`.
- Phát hiện cần tham số `-fl`.
- Dò bằng listener và xác nhận tiền tố `TLBB01` + header `id/size/idx`.

### Bước 3: Đăng nhập ✅
- `lua-netpack.c`: bỏ tiền tố `TLBB01`.
- `CLConnect`, `LCRetConnect` (74 B), `CLAskLogin` (len+927), `LCRetLogin` (83 B; mã cờ lỗi khác GL, ví dụ GL 31 → TTL 47).

### Bước 4: Danh sách và tạo nhân vật ✅
- `LCRetCharList`: thêm 17 B đuôi. `CLCheckAllowCreateChar`/`LCRet…` đổi sang ID 1250/1251. `CLAskCreateChar` đọc đủ 148 B.
- Commit `47e18476`.

### Bước 5: Màn chọn nhân vật ✅
- Client treo ở "Đang vào trò chơi…". Dùng `mock_login.py` thử nhiều bố cục; crash `nRaceID < 2` cho thấy GUID là 64-bit.
- Dịch ngược hàm `0x924120` để lập bảng bản ghi 420 B (mục 5.4).
- Màn chọn nhân vật hiện đúng tên, model, cấp.

### Bước 6: Chọn nhân vật → game server ✅
- `CLAskCharLogin`: guid là u32 thứ hai.
- `LCRetCharLogin`: bọc `u8 key + u32 len + body` (thân 62 B).
- Thêm `TTL_GAME_PORT` để relay ghi được cả kết nối game.

### Bước 7: Đổi toàn bộ ID gói game ✅
- `remap_ids.py`: 431 hằng khớp TTL, 51 hằng không có trong TTL (nhận 50000+), không có xung đột.
- Sửa 2 ID viết cứng: `CGFlyOperation` 2242 → 1477, `GCCharDoAction` 786 → 286.

### Bước 8: Vào bản đồ ✅
- `CGConnect` đọc guid 64-bit. `GCConnect` 18 B, `GCEnterScene` 52 B.
- Client nhận được `KeyExchange` → `GCConnect` → `GCEnterScene`.

### Bước 9: Dọn các gói làm client crash ✅ (một phần)
- Chặn gói ID ≥ 50000 (`Invalid Packet[50001]`).
- `GCNotifyEquip` đệm cho đủ 246 B.
- Tạm chặn `GCExteriorInfo` (2349) và `GCItemInfo` (1205).
- Bằng `mock_world.py`, client **vào được thế giới game** (Đại Lý 165,170: minimap, thanh kỹ năng, khung chat).
- Commit `f85d8ff0`.

### Bước 10: Gói cố định sai kích thước ⏳ (đã sửa, chưa test)
- `GCDoubleExpInfo` → 20 B; `GCAskServerTime` → 99 B; `GCComboSkillOperation` → 52 B; `GCMissionList` → 4544 B (20 nhiệm vụ × 16 tham số, 256 cờ, 512 data; **suy ra từ kích thước**, cần kiểm chứng).
- Các sửa này mới nằm trong WSL, chưa commit.

---

## 10. Bước hiện tại

▶ **Bước 11: gói `GCCharBaseAttrib` (ID 213)**, thuộc tính của chính nhân vật.

**Hiện tượng:** trên server thật, client crash do access violation (ghi vào địa chỉ 0x6B011000). Chia đôi bằng `ttl_bisect.ps1` cho thấy gói gây lỗi là frame thứ 98 sau `CGEnterScene`, tức `GCCharBaseAttrib` dài 273 B.

**Đã biết về định dạng TTL** (hàm Read tại `0xb03400`):

```
u8  header      bit7 = 1 → thân bọc mã hoá; bit6 + 6 bit thấp: cờ phụ
--- nếu bit7 = 1:
u8  key
u32 len
body[len]       client giải mã byte -= key
    u32  objID
    u8   data_id
    u8[16] mặt nạ cờ (128 bit, offset +0x15..+0x24)
    u16  ?
    ... mỗi bit được bật → một trường, theo thứ tự client đọc
--- nếu bit7 = 0: nhánh khác (0xb03d5c), chưa phân tích
```

- Đã trích bảng **bit → kích thước trường** (khoảng 100 bit): bit 0 = 4 B (model), bit 1 = chuỗi (tên), bit 2 = chuỗi + 4 B (danh hiệu), bit 3–6 = 1 B, … Thứ tự đọc không tăng dần (ví dụ 0..7, rồi 109, 110, 8, …).
- Server hiện gửi định dạng GL: `objID, data_id, 14 byte cờ`, rồi các trường theo bit GL (`combat.lua`, `packet.GCCharBaseAttrib.bos`).

**Việc cần làm trong bước này:**

1. Hoàn thiện bảng bit TTL. Bổ sung các bit đọc qua thanh ghi khác hoặc qua hàm con (bit 96–99, 106, 108, 113–127).
2. Ghép **ý nghĩa** từng bit TTL với bit GL. Dựa vào kích thước, thứ tự, và cách client dùng giá trị (setter nào được gọi).
3. Viết lại `GCCharBaseAttrib.bos`: thêm byte header, bọc key 0 + len, mặt nạ 128 bit, rồi ghi trường theo thứ tự TTL. Tạm thời chỉ gửi các bit chắc chắn (model, tên, cấp, HP/MP %, tốc độ, môn phái…).
4. Kiểm tra nhanh bằng `mock_world.py` (sửa thẳng frame trong bản ghi), rồi mới khởi động lại server thật.

---

## 11. Việc tiếp theo

Thứ tự ưu tiên:

1. **Kiểm tra bước 10** trên server thật (4 gói cố định đã sửa) rồi commit.
2. **Cấu trúc vật phẩm TTL 244 B** (GL 188 B). Dùng trong `GCItemInfo`, `GCNotifyEquip`, `GCMyBagList`, `GCDetailEquipList`, `GCCharEquipment`… Phải map xong mới mở chặn `GCItemInfo`.
3. **`GCExteriorInfo` (2349)**: gói phức tạp, có các lệnh gọi `0xb211a0`.
4. **Các gói độ dài thay đổi khác trong luồng vào game:** `GCDetailAttrib` (1233), `GCMyBagList`, `GCRetSetting`, `GCAskFashionDepotData`, `GCSectDetail`, `GCUICommand`, `GCMissionModify`… Chạy `ttl_iter.ps1`/`ttl_bisect.ps1` sau mỗi lần sửa để tìm gói lỗi kế tiếp.
5. **Gói từ client lên:** `CGCharMove`, `CGChat`, `CGHeartBeat` (phức tạp), `CGEnvRequest`, `CGAskDetailAttrib` (TTL thêm 4 B)… Kiểm tra server đọc đúng, nhất là các trường GUID 64-bit.
6. **Gói di chuyển và nhìn thấy người/quái khác:** `GCNewPlayer` (TTL dùng guid 8 B), `GCNewMonster_Move`, `GCCharMove`…
7. **Dữ liệu game:** đối chiếu bảng DBC của client với dữ liệu server (vật phẩm, kỹ năng, bản đồ, NPC, script nhiệm vụ).
8. **Dọn dẹp lâu dài:**
   - Chép các script PowerShell vào repo.
   - Thay các chỗ `shortcut:` (đệm 0, chặn gói) bằng bố cục thật.
   - Trả `LoginServer.txt` về bản gốc khi phát hành, hoặc trỏ tới IP thật.
   - Bỏ `TTL_GAME_PORT` khi không dùng relay.

---

## 12. Quy trình làm việc hằng ngày

```text
1. Sửa Lua trong WSL:  \\wsl.localhost\Ubuntu-22.04\home\<user>\372dragon\Server\GameServer\...
2. Khởi động lại server: bash restart.sh   (stop → all → chờ cổng 15733, khoảng 3–4 phút)
   Lưu ý: service nạp Lua lúc khởi động. Sửa file trong lúc đang restart thì có thể phải restart lại.
3. Chạy relay:  python -I Tools/re/ttl/relay.py 25733 127.0.0.1 15733 <thư_mục_log> 25734:15734
4. Chạy client tự động:  ttl_login.ps1  (nếu kẹt ở danh sách server thì chạy lại với -NoLaunch)
   Vào game:  ttl_do.ps1 "1805,671"   (nút "Vào trò chơi" ở độ phân giải 1920x1080)
5. Nếu crash:
   - ttl_clean.ps1 (đóng bảng lỗi, BẮT BUỘC)
   - Đọc info.txt trong Bin\Crash\crash*.zip → "Read Packet Size error[ID]" / "Invalid Packet[ID]" / assert
   - Lỗi không ghi ID → ttl_bisect.ps1 với bản ghi relay
6. Thử nhanh không cần restart: mock_world.py phát lại bản ghi, sửa byte trong Python, chạy ttl_iter.ps1
7. Chép file đã sửa từ WSL về E:\games\372dragon, commit.
```

**Mẹo:**
- Bộ gõ tiếng Việt (Telex) làm sai chữ gõ tự động ("test" → "tét", "hero" → "hẻo"). Dùng tên không có chuỗi Telex như `abc123`, `lan123`.
- Click đầu tiên sau khi client mở hay bị bỏ qua. Script đã click 2 lần.
- Lệnh `bash -c '...'` gọi từ Git Bash bị lỗi trích dẫn. Hãy ghi lệnh ra file `.sh` rồi chạy `wsl ... bash file.sh` với `MSYS_NO_PATHCONV=1`.

---

## 13. Lưu ý và rủi ro

- **Các chỗ tạm (shortcut)** đều có comment `shortcut:` trong mã: đệm 0 cho đủ kích thước, chặn gói chưa map, trường không rõ nghĩa gửi 0. Client có thể hiển thị sai dù không crash.
- **Bỏ tương thích GL:** server hiện tại không còn phục vụ client GL được nữa (ID và bố cục đã đổi).
- **Tài nguyên dịch ngược:** chỉ có mã máy, không có symbol. Bố cục "phức tạp" phải đọc tay. Nếu cài Ghidra/IDA để decompile thì sẽ nhanh hơn nhiều.
- **Bảo mật:** không đưa mật khẩu MongoDB, IP thật hay dữ liệu tài khoản vào repo công khai. Bản ghi relay chứa dữ liệu phiên đăng nhập.
- **Bản quyền:** client và tài nguyên thuộc VNG. Dự án chỉ dùng để nghiên cứu và chạy nội bộ.

---

## 14. Phụ lục

### 14.1 File server đã sửa (so với bản 372dragon gốc)

| File | Nội dung sửa |
|---|---|
| `lualib-src/lua-netpack.c` | Bỏ tiền tố `TLBB01` |
| `services/login/packet.lua` | CLConnect, LCRetConnect, CLAskLogin, LCRetLogin, LCRetCharList (420 B), CLAskCreateChar, CLAskCharLogin, LCRetCharLogin, ID 1250/1251 |
| `services/login/logind.lua` | `TTL_GAME_PORT` |
| `services/game/packet_ids.lua` | ID theo TTL (bản cũ: `.bak-ttl-20261108`) |
| `services/game/packet_domains/core.lua` | CGConnect 140 B, GCConnect 18 B |
| `services/game/packet_domains/movement.lua` | GCEnterScene 52 B |
| `services/game/packet_domains/item.lua` | GCNotifyEquip 246 B (tạm đệm 0) |
| `services/game/packet_domains/misc.lua` | GCDoubleExpInfo, GCAskServerTime, GCComboSkillOperation (chưa commit) |
| `services/game/packet_domains/combat.lua` | GCMissionList 4544 B (chưa commit) |
| `services/game/packet_domains/cys_mount_flight.lua`, `cy_combat.lua`, `lualib/cy_four_combat/action_packet.lua` | ID viết cứng → ID TTL |
| `lualib/net.lua` | Chặn ID ≥ 50000 và `TTL_UNMAPPED` |

Mỗi file sửa trong WSL đều có bản sao lưu `*.bak-ttl-20261108` bên cạnh.

### 14.2 Địa chỉ quan trọng trong `Game.exe` (image base 0x400000)

| Địa chỉ | Hàm |
|---|---|
| `0xafd690` / `0xafd7a0` | LCRetCharLogin Read / Write (bọc key + len) |
| `0xa2dcf0` | Giải mã `byte -= key` |
| `0xaf8ca0` | LCRetCharList Read |
| `0x8b66c0` | Handler LCRetCharList |
| `0x924120` | Thêm nhân vật vào màn chọn (bản ghi 420 B) |
| `0x523410` | `SetRaceID` (assert `nRaceID < 2`) |
| `0xb03400` | GCCharBaseAttrib Read (mặt nạ 128 bit) |

### 14.3 Cổng

| Cổng | Dịch vụ |
|---|---|
| 15733 | logind (WSL) |
| 15734 | gamed (WSL) |
| 25733 / 25734 | relay ghi log trên Windows |
| 25735 | game giả lập (`mock_world.py`) |
| 27018 | MongoDB |
| 1884 | Mosquitto |
