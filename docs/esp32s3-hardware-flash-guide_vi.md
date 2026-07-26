# Hướng dẫn kết nối phần cứng & nạp firmware cho ESP32-S3

Tài liệu này hướng dẫn bạn **kết nối board ESP32-S3 với máy tính** và **nạp firmware nhỏ trí (xiaozhi)** để board kết nối tới `xiaozhi-esp32-server` mà bạn đã triển khai.

> Firmware nằm ở dự án phần cứng [78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32). Repo này (`xiaozhi-esp32-server`) chỉ là **backend**. Board sẽ tìm tới server thông qua **địa chỉ OTA** mà bạn cấu hình vào firmware.

Có **2 cách nạp**:

- **Cách A — Nạp firmware biên dịch sẵn qua trình duyệt (không cần cài IDF).** Đơn giản nhất, khuyên dùng cho người mới.
- **Cách B — Tự biên dịch từ mã nguồn bằng ESP-IDF.** Dùng khi bạn cần tùy chỉnh board type, wake word, hoặc gắn cứng địa chỉ OTA vào firmware.

---

## 0. Đi thẳng cho cấu hình của bạn (ESP32-S3 N16R8 + INMP441 + MAX98357A, KHÔNG màn, KHÔNG nút, tiếng Việt, nạp qua trình duyệt)

Đây là quy trình đầy đủ, đúng thứ tự cho đúng cấu hình bạn mô tả. Các mục 1–4 bên dưới là bản giải thích chi tiết hơn.

**Điểm mấu chốt cần hiểu trước:**
- **Ngôn ngữ hội thoại (nghe/nói tiếng Việt) do SERVER quyết định**, KHÔNG phải firmware. Firmware chỉ chuyển âm thanh; việc nhận dạng (ASR), suy nghĩ (LLM) và đọc (TTS) tiếng Việt là do bạn cấu hình ở `xiaozhi-esp32-server`. → Bạn KHÔNG cần một firmware "tiếng Việt" riêng; firmware biên dịch sẵn (mặc định) là đủ.
- **Không có màn hình**: chọn board type `bread-compact-wifi` khi nạp. Không cắm OLED thì firmware vẫn chạy bình thường (chỉ là không hiển thị gì).
- **Không có nút ngoài**: vẫn ổn — đánh thức bằng **từ khoá giọng nói (wake word)**, không cần nút. Riêng nút **BOOT/RST có sẵn trên chính board DevKit** dùng để reset/vào chế độ cấu hình khi cần (bạn không phải hàn thêm nút).
- **Từ khoá đánh thức**: firmware biên dịch sẵn dùng wake word tiếng Trung/tiếng Anh có sẵn (ví dụ "你好小智" / "Hi, ESP"). Chưa có wake word tiếng Việt trong bản dựng sẵn. Sau khi đánh thức, bạn cứ **nói tiếng Việt bình thường** và server sẽ trả lời bằng tiếng Việt.

**Các bước:**

1. **Đấu mạch** theo đúng chân ở [mục 1B](#1b-sơ-đồ-đấu-nối-mạch-board-diy-bread-compact-wifi): INMP441 (WS=4, SCK=5, SD=6) và MAX98357A (DIN=7, BCLK=15, LRC=16), chung GND. Bỏ qua phần OLED và nút âm lượng.

2. **Cài & chạy server `xiaozhi-esp32-server`** trước (bạn cần có **địa chỉ OTA** hoạt động). Xem [Deployment.md](./Deployment.md). Có 2 kiểu:
   - *Triển khai đơn giản (chỉ Server)*: cấu hình bằng file `data/.config.yaml`.
   - *Triển khai đầy đủ (có Web 智控台/bảng điều khiển)*: cấu hình bằng giao diện web, dễ hơn cho việc đặt tên/nhân vật/giọng.

3. **Cấu hình tiếng Việt trên server** (xem [mục 5](#5-cấu-hình-tiếng-việt-tên-trợ-lý-trên-server)):
   - **TTS (giọng đọc tiếng Việt)**: dùng `EdgeTTS` (miễn phí), đổi `voice` thành giọng Việt, ví dụ `vi-VN-HoaiMyNeural` (nữ) hoặc `vi-VN-NamMinhNeural` (nam).
   - **ASR (nhận dạng tiếng Việt)**: model mặc định `FunASR/SenseVoice` **không có tiếng Việt** (chỉ zh/en/ja/ko/yue). Muốn nghe tiếng Việt chuẩn, chọn một ASR hỗ trợ tiếng Việt (ví dụ dịch vụ đám mây Aliyun/Doubao/Volcano có tham số `language`, hoặc một ASR nền Whisper).
   - **Tên trợ lý + tính cách + trả lời tiếng Việt**: sửa `prompt` (persona) sang tiếng Việt, đặt tên trong đó (hoặc đặt trong bảng điều khiển web nếu dùng triển khai đầy đủ).

4. **Nạp firmware biên dịch sẵn qua trình duyệt** (xem [Cách A](#cách-a--nạp-firmware-biên-dịch-sẵn-qua-trình-duyệt)):
   - Cắm USB (cáp dữ liệu), mở Chrome/Edge vào https://espressif.github.io/esp-launchpad/ , Connect → chọn cổng → chọn firmware `bread-compact-wifi` (bản 1.6.1+), Flash.

5. **Thiết lập WiFi + trỏ về server của bạn** (xem [mục 3B](#3b-cấu-hình-wifi-và-địa-chỉ-server-lần-đầu-bật)):
   - Board bật lần đầu (chưa có WiFi) sẽ **tự phát WiFi cấu hình** tên kiểu `Xiaozhi-XXXX`. Kết nối điện thoại/PC vào WiFi đó → trang cấu hình tự mở.
   - Chọn WiFi nhà bạn (2.4GHz), nhập mật khẩu. Vào **"Tùy chọn nâng cao / 高级选项"** → nhập **địa chỉ OTA** server của bạn (vd `http://192.168.1.25:8002/xiaozhi/ota/`) → Lưu → board khởi động lại.

6. **Ghép thiết bị vào server (nếu dùng triển khai đầy đủ)**: lần đầu kết nối, server báo một **mã 6 số** (đọc qua loa / hiện trong log). Đăng nhập bảng điều khiển web → thêm thiết bị bằng mã đó → gán vào "agent" đã cấu hình tiếng Việt ở bước 3.

7. **Kiểm tra**: nói wake word → nói một câu tiếng Việt → nghe trợ lời trả lời tiếng Việt; đồng thời xem log ở server.

---

## 1. Chuẩn bị

### Phần cứng
- 1 board ESP32-S3 (ví dụ: `lichuang-dev`, `bread-compact-wifi`, `esp-box-3`, `waveshare` ...). Ghi nhớ **đúng model board** của bạn — sẽ cần khi cấu hình `BOARD_TYPE`.
- 1 **cáp USB truyền dữ liệu** (KHÔNG dùng cáp chỉ sạc). Đây là lỗi phổ biến nhất khiến máy không nhận cổng.
- Mạng WiFi 2.4GHz (ESP32-S3 không hỗ trợ WiFi 5GHz).

### Server
- Bạn đã triển khai xong `xiaozhi-esp32-server` và có **địa chỉ OTA** hoạt động, ví dụ:
  - Triển khai đơn giản (Server): `http://192.168.1.25:8003/xiaozhi/ota/`
  - Triển khai đầy đủ (all modules): `http://192.168.1.25:8002/xiaozhi/ota/`
- Mở địa chỉ OTA đó bằng trình duyệt: phải hiện thông báo *"OTA接口运行正常 / OTA interface running normally"*. Nếu chưa, xem [ota-upgrade-guide.md](./ota-upgrade-guide.md) và [firmware-build.md](./firmware-build.md) để cấu hình `server.websocket` trước.

> Ghi chú: `192.168.1.25` là ví dụ IP LAN của máy chạy server. Thay bằng IP thật của bạn. Board và máy chạy server phải cùng mạng LAN.

---

## 1B. Sơ đồ đấu nối mạch (board DIY `bread-compact-wifi`)

Phần này dành cho ai **tự lắp mạch trên breadboard** với ESP32-S3 + micro I2S + loa I2S. Các số chân dưới đây lấy **trực tiếp từ mã nguồn firmware** `main/boards/bread-compact-wifi/config.h` (chế độ I2S Simplex). Nếu bạn dùng board khác, số chân sẽ khác — xem mục ["Board khác thì tra chân ở đâu"](#board-khác-thì-tra-chân-ở-đâu) bên dưới.

### Linh kiện
- 1× ESP32-S3 DevKit (ví dụ ESP32-S3-DevKitC / N16R8).
- 1× Micro I2S MEMS **INMP441** (ngõ vào âm thanh).
- 1× Module khuếch đại I2S **MAX98357A** + 1 loa 4–8Ω (ngõ ra âm thanh).
- (Tuỳ chọn) 1× màn hình OLED **SSD1306** I2C 128×32 hoặc 128×64.
- Dây cắm breadboard.

### Bảng đấu nối chân

**Micro INMP441 → ESP32-S3** (I2S vào):

| INMP441 | ESP32-S3 | Ghi chú |
|---|---|---|
| VDD | 3V3 | Cấp nguồn 3.3V |
| GND | GND | |
| SD  | GPIO6 | Dữ liệu mic (MIC DIN) |
| WS / LRCL | GPIO4 | Word select |
| SCK / BCLK | GPIO5 | Bit clock |
| L/R | GND | Nối GND = kênh trái |

**Khuếch đại MAX98357A → ESP32-S3** (I2S ra):

| MAX98357A | ESP32-S3 | Ghi chú |
|---|---|---|
| Vin | 5V (VBUS) | Cấp nguồn (3.3V cũng chạy, 5V kêu to hơn) |
| GND | GND | |
| DIN | GPIO7 | Dữ liệu loa (SPK DOUT) |
| BCLK | GPIO15 | Bit clock |
| LRC | GPIO16 | Word select (LRCK) |
| Speaker + / − | Loa | Nối 2 cực loa |

**Màn hình OLED SSD1306 (I2C — tuỳ chọn) → ESP32-S3:**

| OLED | ESP32-S3 | Ghi chú |
|---|---|---|
| VCC | 3V3 | |
| GND | GND | |
| SDA | GPIO41 | |
| SCL | GPIO42 | |

**Nút bấm / LED tích hợp (đã định nghĩa sẵn trong firmware, lắp nếu cần):**

| Chức năng | ESP32-S3 | Ghi chú |
|---|---|---|
| Nút BOOT | GPIO0 | Thường có sẵn trên DevKit |
| Nút cảm ứng (Touch) | GPIO47 | Tuỳ chọn |
| Tăng âm lượng | GPIO40 | Nút nhấn xuống GND |
| Giảm âm lượng | GPIO39 | Nút nhấn xuống GND |
| LED RGB tích hợp | GPIO48 | WS2812 trên DevKit |

### Sơ đồ khối

```
                         ESP32-S3 DevKit
                       ┌────────────────────┐
   INMP441 (MIC)       │                    │      MAX98357A (AMP) ── Loa
   ┌──────────┐        │                    │      ┌──────────┐
   │ VDD ─────┼── 3V3 ─┤ 3V3           5V  ─┼── Vin┤          │
   │ GND ─────┼── GND ─┤ GND          GND  ─┼── GND┤          │
   │ SD  ─────┼─ GPIO6 ┤ 6                7 ┼─ DIN─┤ DIN      │
   │ WS  ─────┼─ GPIO4 ┤ 4               15 ┼─BCLK─┤ BCLK  +──┼── Loa+
   │ SCK ─────┼─ GPIO5 ┤ 5               16 ┼─LRC ─┤ LRC   −──┼── Loa−
   │ L/R ─────┼── GND  │                    │      └──────────┘
   └──────────┘        │ 41(SDA) 42(SCL)    │
                       │   │        │       │
                       └───┼────────┼───────┘
                           │        │
                      OLED SDA   OLED SCL  (3V3/GND cấp riêng)

   USB ── cáp dữ liệu ── PC   (vừa cấp nguồn 5V vừa để nạp/monitor)
```

Lưu ý đấu nối:
- **Chung GND**: tất cả GND (ESP32, mic, amp, OLED, loa) phải nối chung.
- **Không cấp nhầm điện áp**: INMP441 và OLED dùng **3V3**; MAX98357A nên dùng **5V** (chân 5V/VBUS của DevKit) để loa kêu to, nhưng 3V3 vẫn chạy.
- Khi mới lắp, có thể bỏ OLED và các nút âm lượng — chỉ cần mic + loa là đã nói chuyện được (chọn cấu hình không màn hình khi build nếu muốn).

### Board khác thì tra chân ở đâu
Mỗi loại board có file định nghĩa chân riêng trong repo firmware [78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32):
```
main/boards/<tên-board>/config.h
```
Mở đúng thư mục board của bạn (ví dụ `lichuang-dev`, `esp-box-3`, `waveshare`...) để xem `AUDIO_I2S_*_GPIO_*`, `DISPLAY_*_PIN`, `*_BUTTON_GPIO`. **Tuyệt đối không tự đổi chân trong firmware** để khớp mạch của bạn (theo `AGENTS.md`, việc đó ảnh hưởng tương thích OTA) — hãy đấu mạch theo đúng chân mà board đã định nghĩa.

---

## 2. Kết nối phần cứng với máy tính

1. **Cắm cáp USB** từ board vào máy tính.
   - Nhiều board ESP32-S3 có **2 cổng USB**:
     - Cổng ghi *UART* (đi qua chip CH340/CH343/CP210x) — dùng để nạp/monitor.
     - Cổng *USB* native (USB-OTG/USB-JTAG tích hợp trong chip S3).
   - Nếu không chắc, thử lần lượt từng cổng và xem cổng nào máy nhận (bước 3).

2. **Cài driver USB-UART** (nếu board dùng chip UART ngoài):
   - CP210x: driver Silicon Labs.
   - CH340/CH343: driver WCH.
   - Cổng USB native của ESP32-S3 thường **không cần** driver.

3. **Kiểm tra máy đã nhận cổng chưa:**
   - **Windows:** mở *Device Manager → Ports (COM & LPT)*, tìm mục `COMx`.
   - **Linux:** chạy `ls /dev/ttyUSB* /dev/ttyACM*` (thường là `/dev/ttyUSB0` hoặc `/dev/ttyACM0`). Nếu bị lỗi quyền: `sudo usermod -aG dialout $USER` rồi đăng xuất/đăng nhập lại.
   - **macOS:** `ls /dev/cu.*` (ví dụ `/dev/cu.usbserial-xxx`).

4. **Chế độ nạp (download/boot mode)** — hầu hết board tự vào chế độ nạp nên bạn **không cần** làm gì. Nếu nạp thất bại, vào thủ công:
   - Giữ nút **BOOT** (đôi khi ghi `IO0`), nhấn rồi thả nút **RST/EN**, sau đó thả nút **BOOT**.
   - Nạp xong nhấn **RST** một lần để board chạy firmware.

---

## Cách A — Nạp firmware biên dịch sẵn qua trình duyệt

Phù hợp nếu bạn dùng firmware đã build sẵn của tác giả (khuyên dùng bản **1.6.1 trở lên**): https://github.com/78/xiaozhi-esp32/releases

1. Dùng **Google Chrome hoặc Microsoft Edge** (Web Serial chỉ chạy trên các trình duyệt này, không dùng Firefox/Safari).
2. Mở trình nạp web: https://espressif.github.io/esp-launchpad/
3. Nhấn **Connect**, chọn đúng cổng `COMx` / `/dev/ttyUSBx` của board.
4. Chọn chip target **ESP32-S3**, chọn/tải firmware `.bin` phù hợp với board của bạn, rồi nhấn **Flash / Program**.
   - Tham khảo chi tiết trong [firmware-build.md](./firmware-build.md) (mục "第7步 烧录固件" — nạp qua ESP-Launchpad).
5. Sau khi nạp xong, **cấu hình server cho board** (theo [firmware-setting.md](./firmware-setting.md)):
   - Board bật lên lần đầu sẽ tạo WiFi phát ra (chế độ **cấu hình mạng / 配网**). Kết nối điện thoại/máy tính vào WiFi đó, trang cấu hình sẽ tự mở.
   - Vào **"Tùy chọn nâng cao / 高级选项"**, nhập **địa chỉ OTA** của server bạn (ví dụ `http://192.168.1.25:8002/xiaozhi/ota/`), chọn WiFi nhà bạn, nhập mật khẩu, rồi **Lưu** và khởi động lại board.

---

## Cách B — Tự biên dịch từ mã nguồn bằng ESP-IDF

Dùng khi cần gắn cứng địa chỉ OTA hoặc đổi board type/wake word.

### B1. Cài môi trường ESP-IDF
Cài **ESP-IDF 5.3.2 trở lên**. Hướng dẫn chi tiết (tiếng Trung): [Windows搭建 ESP IDF 5.3.2开发环境](https://icnynnzcwou8.feishu.cn/wiki/JEYDwTTALi5s2zkGlFGcDiRknXf). Hoặc theo hướng dẫn chính thức của Espressif.

### B2. Tải mã nguồn firmware
```bash
git clone https://github.com/78/xiaozhi-esp32.git
cd xiaozhi-esp32
```

### B3. Sửa địa chỉ OTA
Mở file `main/Kconfig.projbuild`, tìm `config OTA_URL` và đổi giá trị `default` thành địa chỉ OTA server của bạn:
```
config OTA_URL
    string "Default OTA URL"
    default "http://192.168.1.25:8002/xiaozhi/ota/"
```

### B4. Đặt target & cấu hình board
```bash
# Đặt chip target là esp32s3
idf.py set-target esp32s3

# Mở menu cấu hình
idf.py menuconfig
```
Trong menu: vào **`Xiaozhi Assistant`** → đặt **`BOARD_TYPE`** đúng model board của bạn. Lưu và thoát.

### B5. Biên dịch
```bash
idf.py build
```

### B6. Nạp firmware

**Cách nạp trực tiếp qua cổng (nhanh nhất khi đã có IDF):**
```bash
# Thay PORT bằng cổng thật, ví dụ Windows: -p COM5 ; Linux: -p /dev/ttyUSB0
idf.py -p PORT flash monitor
```
`monitor` sẽ mở log serial ngay sau khi nạp (thoát bằng `Ctrl+]`).

**Hoặc đóng gói ra file `.bin` để nạp qua web (Cách A):**
```bash
cd scripts
python release.py
```
File `build/merged-binary.bin` sinh ra chính là firmware để nạp qua ESP-Launchpad.
> Nếu `release.py` báo lỗi liên quan tới "zip", cứ bỏ qua — miễn là `build/merged-binary.bin` đã được tạo.

---

## 3B. Cấu hình WiFi và địa chỉ server (lần đầu bật)

Áp dụng khi bạn nạp firmware biên dịch sẵn (Cách A) và muốn board trỏ về server tự host của mình.

1. Sau khi nạp xong, cấp nguồn cho board (cắm USB). Vì **chưa có WiFi lưu sẵn**, board tự vào **chế độ cấu hình mạng** và phát ra một WiFi tên kiểu `Xiaozhi-XXXX`.
2. Dùng điện thoại/máy tính **kết nối vào WiFi `Xiaozhi-XXXX`** đó. Trang cấu hình (captive portal) thường tự bật lên; nếu không, mở trình duyệt vào `http://192.168.4.1`.
3. Trong trang cấu hình:
   - Chọn **WiFi nhà bạn** (bắt buộc băng tần **2.4GHz**) và nhập mật khẩu.
   - Bấm **"Tùy chọn nâng cao / 高级选项"**, nhập **địa chỉ OTA** của server bạn vào ô địa chỉ, ví dụ:
     ```
     http://192.168.1.25:8002/xiaozhi/ota/
     ```
     (thay bằng IP:cổng thật của server bạn — xem [firmware-setting.md](./firmware-setting.md), ảnh minh hoạ `docs/images/firmware-setting-ota.png`).
   - Bấm **Lưu / Save**, board sẽ khởi động lại và tự kết nối WiFi + server.
4. **Muốn đổi lại WiFi/địa chỉ sau này** (không có nút ngoài): dùng nút **BOOT** có sẵn trên DevKit để đưa board về chế độ cấu hình, hoặc xoá WiFi đã lưu rồi khởi động lại — board lại phát `Xiaozhi-XXXX`.

---

## 3. Kiểm tra hoạt động

1. Sau khi nạp + kết nối WiFi thành công, đọc **log serial** trên board (`idf.py monitor` hoặc terminal serial): board sẽ báo kết nối WiFi và kết nối tới server.
2. Nói **từ đánh thức (wake word)** để đánh thức nhỏ trí.
3. Quan sát **log ở phía server** (`xiaozhi-esp32-server`) — bạn sẽ thấy thiết bị kết nối và dữ liệu hội thoại đi qua.

---

## 4. Xử lý sự cố thường gặp

| Triệu chứng | Nguyên nhân / Cách xử lý |
|---|---|
| Máy không thấy cổng COM/ttyUSB | Dùng cáp chỉ sạc → đổi cáp truyền dữ liệu; chưa cài driver CH340/CP210x; thử cổng USB còn lại trên board |
| Nạp báo lỗi timeout / "Failed to connect" | Vào chế độ download thủ công: giữ **BOOT**, nhấn thả **RST**, thả **BOOT**; giảm baud; đổi cáp/cổng |
| Nút Connect trên web bị mờ | Dùng Chrome/Edge (Web Serial); đóng các chương trình đang chiếm cổng (Arduino IDE, serial monitor khác) |
| OTA báo không bình thường | Chưa cấu hình `server.websocket` — xem [firmware-build.md](./firmware-build.md) / [ota-upgrade-guide.md](./ota-upgrade-guide.md) |
| Board không kết nối server | Sai địa chỉ OTA; board và server khác mạng LAN; WiFi 5GHz (S3 chỉ hỗ trợ 2.4GHz) |
| Nạp xong màn hình/log trắng | Nhấn nút **RST** một lần để board chạy firmware |

Các câu hỏi khác xem thêm [FAQ.md](./FAQ.md).

---

## 5. Cấu hình tiếng Việt & tên trợ lý (trên server)

Nhắc lại: **tiếng Việt được quyết định ở server**, không phải firmware. Bạn cần chỉnh 3 nhóm: TTS (giọng đọc), ASR (nhận dạng), và prompt/tên.

### Nếu dùng "triển khai đơn giản" (sửa file `data/.config.yaml`)

File cấu hình nằm ở `main/xiaozhi-server/data/.config.yaml` (copy từ `config.yaml` rồi ghi đè các mục cần đổi). Ví dụ:

```yaml
selected_module:
  TTS: EdgeTTS        # giọng đọc
  # ASR: ...          # xem lưu ý ASR tiếng Việt bên dưới

TTS:
  EdgeTTS:
    type: edge
    voice: vi-VN-HoaiMyNeural   # giọng nữ Việt (hoặc vi-VN-NamMinhNeural = nam)

# Tên + tính cách + yêu cầu trả lời tiếng Việt
prompt: |
  Bạn là một trợ lý ảo tên là "Bé Na", nói chuyện thân thiện, tự nhiên bằng tiếng Việt.
  Luôn trả lời ngắn gọn, lịch sự và bằng tiếng Việt.
```
Lưu file rồi **khởi động lại** `xiaozhi-server`.

> Đổi **tên trợ lý** = đổi tên trong `prompt`. Đổi **giọng** = đổi `voice`. Đổi **giọng nói/độ dài trả lời** = sửa nội dung `prompt`.

### Nếu dùng "triển khai đầy đủ" (Web 智控台 / bảng điều khiển)

1. Đăng nhập bảng điều khiển bằng tài khoản **super admin**.
2. Vào phần **Agent/Vai trò (角色)** → tạo/sửa một agent:
   - **Tên trợ lý**: đặt tên tuỳ ý.
   - **Prompt/nhân vật**: viết bằng tiếng Việt như ví dụ trên.
   - **TTS**: chọn `EdgeTTS`, đặt giọng `vi-VN-HoaiMyNeural` / `vi-VN-NamMinhNeural`.
   - **ASR / LLM**: chọn model (xem lưu ý ASR bên dưới).
3. Ở **参数管理 / Quản lý tham số** đảm bảo `server.websocket` đã đúng.
4. Sau khi ghép thiết bị (mã 6 số ở bước 6 mục 0), **gán thiết bị vào agent** này.

### Lưu ý quan trọng về ASR tiếng Việt
- Model mặc định **`FunASR`/`SenseVoice` KHÔNG hỗ trợ tiếng Việt** (chỉ zh, en, ja, ko, yue). Nếu để mặc định, tiếng Việt của bạn sẽ bị nhận dạng sai/ra ký tự lạ.
- Để nghe hiểu tiếng Việt, chọn một ASR có tiếng Việt, ví dụ:
  - ASR đám mây có tham số `language` đặt về tiếng Việt (Aliyun / Doubao / Volcano...), hoặc
  - ASR nền **Whisper** (hỗ trợ đa ngôn ngữ gồm tiếng Việt).
- Các ASR đám mây cần API key — bạn khai báo key trong cùng file `.config.yaml` (hoặc trong bảng điều khiển) ở mục ASR tương ứng.

### TTS EdgeTTS — vì sao khuyên dùng
- Miễn phí, không cần API key, có sẵn nhiều giọng Việt tự nhiên (`vi-VN-HoaiMyNeural`, `vi-VN-NamMinhNeural`).
- Chỉ cần đổi `voice` là ra tiếng Việt ngay.

---

## 6. Dựng server đơn giản (Docker — chỉ chạy Server)

Đây là cách nhẹ nhất: chỉ chạy module `xiaozhi-server` bằng Docker, cấu hình bằng 1 file `.config.yaml`, **không cần Web/CSDL**. Chi tiết gốc: [Deployment.md](./Deployment.md) (方式一).

> **Kiến trúc CPU**: image Docker phát hành (0.8.2+) chỉ hỗ trợ **x86**. Nếu máy bạn là **ARM64**, phải tự build image theo [docker-build.md](./docker-build.md).

### B1. Cài Docker
Cài Docker + Docker Compose cho hệ điều hành của bạn (Windows/macOS: Docker Desktop; Linux: docker engine).

### B2. Tạo thư mục
```
xiaozhi-server/
  ├─ data/
  └─ models/
      └─ SenseVoiceSmall/
```

### B3. Tải model nhận dạng giọng nói (ASR mặc định — offline)
Tải file `model.pt` của **SenseVoiceSmall** rồi đặt vào `models/SenseVoiceSmall/model.pt`:
- ModelScope: https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt

> Lưu ý: SenseVoice **không nhận dạng tiếng Việt** (chỉ zh/en/ja/ko/yue). Vẫn cần file này để server khởi động được; muốn nghe hiểu tiếng Việt thì đổi ASR sang loại hỗ trợ tiếng Việt (xem [mục 5](#5-cấu-hình-tiếng-việt--tên-trợ-lý-trên-server)).

### B4. Tải file cấu hình
1. Tải `docker-compose.yml` (từ `main/xiaozhi-server/docker-compose.yml` trong repo, bấm **RAW → download**) đặt vào `xiaozhi-server/`.
2. Tải `config.yaml` (từ `main/xiaozhi-server/config.yaml`) đặt vào `xiaozhi-server/data/`, rồi **đổi tên thành `.config.yaml`**.

Cấu trúc cuối cùng:
```
xiaozhi-server/
  ├─ docker-compose.yml
  ├─ data/
  │   └─ .config.yaml
  └─ models/
      └─ SenseVoiceSmall/
          └─ model.pt
```

### B5. Sửa `.config.yaml` tối thiểu (đã kèm tiếng Việt)
```yaml
server:
  websocket: ws://192.168.1.25:8000/xiaozhi/v1/   # thay bằng IP LAN máy chạy server

selected_module:
  LLM: ChatGLMLLM      # LLM mặc định (miễn phí, nhưng cần đăng ký key)
  TTS: EdgeTTS         # giọng đọc miễn phí, có tiếng Việt

LLM:
  ChatGLMLLM:
    api_key: xxxxxxxxxxxxxxxx.xxxxxx   # lấy key ở https://bigmodel.cn/usercenter/proj-mgmt/apikeys

TTS:
  EdgeTTS:
    type: edge
    voice: vi-VN-HoaiMyNeural

prompt: |
  Bạn là một trợ lý ảo tên là "Bé Na", nói chuyện thân thiện, tự nhiên bằng tiếng Việt.
  Luôn trả lời ngắn gọn, lịch sự và bằng tiếng Việt.
```
Nguyên tắc: `.config.yaml` chỉ cần ghi những mục bạn muốn ghi đè; mục nào thiếu, hệ thống tự đọc từ `config.yaml` gốc.

### B6. Chạy server
Từ trong thư mục `xiaozhi-server/`:
```bash
docker compose up -d
docker logs -f xiaozhi-esp32-server
```

### B7. Xác định địa chỉ (RẤT quan trọng cho việc cấu hình board)
Với Docker, địa chỉ in trong log **không phải** địa chỉ thật. Hãy tính theo **IP LAN** của máy chạy server. Ví dụ IP là `192.168.1.25`:
- WebSocket: `ws://192.168.1.25:8000/xiaozhi/v1/`
- **Địa chỉ OTA (điền vào board ở [mục 3B](#3b-cấu-hình-wifi-và-địa-chỉ-server-lần-đầu-bật))**: `http://192.168.1.25:8003/xiaozhi/ota/`

> Với triển khai đơn giản này, cổng OTA là **8003** (khác với triển khai đầy đủ dùng 8002).

Mở `http://192.168.1.25:8003/xiaozhi/ota/` bằng trình duyệt: nếu báo "OTA接口运行正常 / OTA interface running normally" là server đã sẵn sàng. Giờ quay lại [mục 3B](#3b-cấu-hình-wifi-và-địa-chỉ-server-lần-đầu-bật) để trỏ board về địa chỉ OTA này.
