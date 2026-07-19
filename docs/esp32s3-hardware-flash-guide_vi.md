# Hướng dẫn kết nối phần cứng & nạp firmware cho ESP32-S3

Tài liệu này hướng dẫn bạn **kết nối board ESP32-S3 với máy tính** và **nạp firmware nhỏ trí (xiaozhi)** để board kết nối tới `xiaozhi-esp32-server` mà bạn đã triển khai.

> Firmware nằm ở dự án phần cứng [78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32). Repo này (`xiaozhi-esp32-server`) chỉ là **backend**. Board sẽ tìm tới server thông qua **địa chỉ OTA** mà bạn cấu hình vào firmware.

Có **2 cách nạp**:

- **Cách A — Nạp firmware biên dịch sẵn qua trình duyệt (không cần cài IDF).** Đơn giản nhất, khuyên dùng cho người mới.
- **Cách B — Tự biên dịch từ mã nguồn bằng ESP-IDF.** Dùng khi bạn cần tùy chỉnh board type, wake word, hoặc gắn cứng địa chỉ OTA vào firmware.

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
