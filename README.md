# ESP32-CAM with DS1307 RTC

## A. Giới thiệu

- Chụp ảnh và lưu trữ vào thẻ SD
- Sử dụng module DS1307 RTC để gắn timestamp vào ảnh
- Điều khiển LED bật khi chụp ảnh
- Nhấn reset để chụp ngay lập tức

## B. Hướng dẫn sử dụng

1. **Kết nối phần cứng:**
   - Kết nối module DS1307 RTC với ESP32-CAM:
     - SDA -> GPIO 16
     - SCL -> GPIO 0
   - Kết nối thẻ SD với ESP32-CAM:
     - CLK -> GPIO 14
     - CMD -> GPIO 15
     - D0 -> GPIO 2
     - D1 -> GPIO 4
     - D2 -> GPIO 12
     - D3 -> GPIO 13

2. **Upload chương trình:**
   - Kết nối ESP32-CAM với máy tính qua cổng USB.
   - Mở Arduino IDE hoặc PlatformIO.
   - Cài đặt board ESP32 và chọn đúng cổng COM.
   - Upload chương trình lên ESP32-CAM.

3. **Sử dụng:**
   - Sau khi upload chương trình, ESP32-CAM sẽ bắt đầu chụp ảnh và lưu vào thẻ SD với timestamp từ DS1307.
   - Mở Serial Monitor với baud rate 115200 để xem thời gian thực.

![ESP32-CAM Setup](link-to-your-image)
![434845888_735890945296436_2923035924733321724_n](https://github.com/Tdha2605/Embedded-System/assets/109448448/0964cd01-b5c2-44f5-8060-a408da88a407)


[Link video hướng dẫn sử dụng](https://www.youtube.com/watch?v=your-video-link)

## C. Danh sách linh kiện (Bill of Material)

| Tên linh kiện         | Số lượng | Link mua hàng                                        |
|-----------------------|----------|-----------------------------------------------------|
| ESP32-CAM             | 1        | [Link mua hàng](https://bit.ly/ESP32-CAMMB)  |
| DS1307 RTC Module     | 1        | [Link mua hàng](https://nshopvn.com/product/module-thoi-gian-thuc-rtc-ds1307/)     |
| Dây nối               | 10       | [Link mua hàng](https://bit.ly/Daynoi) |

## D. Sơ đồ nguyên lý - Hardware Schematic

![Hardware Schematic](link-to-your-schematic-image)

## E. Thiết kế phần mềm - Software Concept

- Trạng thái của nút chụp ảnh được xác định thông qua cơ chế ngắt ở GPIO.
- Thời gian hiện tại từ DS1307 RTC được lấy và gắn vào tên file ảnh.
- LED được bật khi bắt đầu chụp ảnh và tắt khi quá trình chụp hoàn tất.
- Ảnh được lưu vào thẻ SD với tên file bao gồm timestamp.

```cpp
#define SDA_PIN 16  // GPIO 16 for SDA
#define SCL_PIN 0   // GPIO 0 for SCL

void setup() {
  Serial.begin(115200);
  Wire.begin(SDA_PIN, SCL_PIN);
  rtc.begin();
  if (!rtc.isrunning()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }
}

void loop() {
  DateTime now = rtc.now();
  Serial.print(now.year(), DEC);
  Serial.print('/');
  Serial.print(now.month(), DEC);
  Serial.print('/');
  Serial.print(now.day(), DEC);
  Serial.print(" ");
  Serial.print(now.hour(), DEC);
  Serial.print(':');
  Serial.print(now.minute(), DEC);
  Serial.print(':');
  Serial.print(now.second(), DEC);
  Serial.print("\n");
  delay(1000);
}
