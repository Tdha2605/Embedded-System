# ESP32-CAM with DS1307 RTC

## A. Giới thiệu

- Chụp ảnh và lưu trữ vào thẻ SD
- Sử dụng module DS1307 RTC để gắn timestamp vào ảnh
- Điều khiển LED bật khi chụp ảnh
- Nhấn reset để chụp ngay lập tức
- ![image](https://github.com/Tdha2605/Embedded-System/assets/109448448/fc867ed4-004f-4887-8865-d69f4ed06a67)

## B. Hướng dẫn sử dụng

1. **Kết nối phần cứng:**
   - Kết nối module DS1307 RTC với ESP32-CAM:
     - SDA -> GPIO 3
     - SCL -> GPIO 1
   - Kết nối thẻ SD với ESP32-CAM:
     - CLK -> GPIO 14
     - CMD -> GPIO 15
     - D0 -> GPIO 2
     - D1 -> GPIO 4
     - D2 -> GPIO 12
     - D3 -> GPIO 13
   - Kết nối camera với ESP32-CAM:
     -  PWDN_GPIO_NUM     32
     -  RESET_GPIO_NUM    -1
     -  XCLK_GPIO_NUM      0
     -  IOD_GPIO_NUM     26
     -  SIOC_GPIO_NUM     27
     -  Y9_GPIO_NUM       35
     -  Y8_GPIO_NUM       34
     -  Y7_GPIO_NUM       39
     -  Y6_GPIO_NUM       36
     -  Y5_GPIO_NUM       21
     -  Y4_GPIO_NUM       19
     -  Y3_GPIO_NUM       18
     -  Y2_GPIO_NUM        5
     -  SYNC_GPIO_NUM    25
     -  HREF_GPIO_NUM     23
     -  PCLK_GPIO_NUM     22

2. **Upload chương trình:**
   - Kết nối ESP32-CAM với máy tính qua cổng USB.
   - Mở Arduino IDE hoặc PlatformIO.
   - Cài đặt thư viện RTCLib
   - Cài đặt board ESP32 và chọn đúng cổng COM.
   - Upload chương trình lên ESP32-CAM.

3. **Sử dụng:**
   - Sau khi upload chương trình, ESP32-CAM sẽ bắt đầu chụp ảnh và lưu vào thẻ SD với timestamp từ DS1307.
   - Mở Serial Monitor với baud rate 115200 để xem thời gian thực.


[Link video demo](https://www.youtube.com/watch?v=your-video-link)

## C. Danh sách linh kiện (Bill of Material)

| Tên linh kiện         | Số lượng | Link mua hàng                                        |
|-----------------------|----------|-----------------------------------------------------|
| ESP32-CAM             | 1        | [Link mua hàng](https://bit.ly/ESP32-CAMMB)  |
| DS1307 RTC Module     | 1        | [Link mua hàng](https://nshopvn.com/product/module-thoi-gian-thuc-rtc-ds1307/)     |
| Dây nối               | 10       | [Link mua hàng](https://bit.ly/Daynoi) |

## D. Sơ đồ nguyên lý - Hardware Schematic

![image](https://github.com/Tdha2605/Embedded-System/assets/109448448/d8f4a7ad-63a7-4424-8785-bf738f323330)
![image](https://github.com/Tdha2605/Embedded-System/assets/109448448/9c082ea2-ff8d-455e-b207-d14637998ac7)
![image](https://github.com/Tdha2605/Embedded-System/assets/109448448/e8610968-4f38-4059-bdd4-ac5f40534a51)



## E. Thiết kế phần mềm - Software Concept

### Trạng thái của nút chụp ảnh được xác định thông qua nút reset GPIO 13
   - Cài đặt hàm chụp ảnh vào trong setup(), khi nhấn reset hệ thống sẽ chụp ảnh một lần và cập nhật lại timestamp để lấy cho những lần chụp định kỳ tiếp theo


### Thời gian hiện tại từ DS1307 RTC được lấy qua I2C (SDA = GPIO3, SCL = GPIO1) và gắn vào tên file ảnh.
   - Something
     ```cpp
    String getPictureFilename() {
    DateTime now = rtc.now();
    char timeString[20];
    snprintf(timeString, sizeof(timeString), "%04d-%02d-%02d_%02d-%02d-%02d",
           now.year(), now.month(), now.day(), now.hour(), now.minute(), now.second());
    Serial.println(timeString);
    String filename = "/picture_" + String(timeString) + ".jpg";
    return filename;
    }

### Ảnh được lưu vào thẻ SD với tên file bao gồm timestamp.
   - Khởi tạo thẻ SD: Trước khi đọc hoặc ghi, thẻ SD cần được khởi tạo. Quá trình khởi tạo bao gồm việc kiểm tra và xác định loại thẻ SD, sau đó thiết lập giao tiếp SPI với thẻ.
     ```cpp
     void initMicroSDCard() {
     Serial.println("Starting SD Card");
    if (!SD_MMC.begin()) {
    Serial.println("SD Card Mount Failed");
    return;
    }
    delay(1000); // Đảm bảo thẻ SD được khởi động đúng cách
    uint8_t cardType = SD_MMC.cardType();
    if (cardType == CARD_NONE) {
    Serial.println("No SD Card attached");
    return;
    }
    Serial.println("SD Card initialized successfully.");
    }
  - Hệ thống sẽ chụp ảnh và lưu vào thẻ SD với tên file bao gồm timestamp. Trước khi lưu, hệ thống sẽ kiểm tra nếu file mở thành công để ghi dữ liệu.
    ```cpp
    void takeSavePhoto() {
    // Bật LED
    digitalWrite(LED_PIN, HIGH);

    // Chụp ảnh bằng camera
    camera_fb_t * fb = esp_camera_fb_get();
    if (!fb) {
    Serial.println("Camera capture failed");
    delay(1000);
    ESP.restart();
    }

    // Đường dẫn nơi lưu ảnh trên thẻ SD
    String path = getPictureFilename();
    Serial.printf("Picture file name: %s\n", path.c_str());

    // Lưu ảnh vào thẻ SD
    fs::FS &fs = SD_MMC;
    File file = fs.open(path.c_str(), FILE_WRITE);
    if (!file) {
    Serial.printf("Failed to open file in writing mode");
    } else {
    file.write(fb->buf, fb->len); // payload (image), payload length
    Serial.printf("Saved: %s\n", path.c_str());
    }
    file.close();
    esp_camera_fb_return(fb);

    // Tắt LED
    digitalWrite(LED_PIN, LOW);
    }

### Chụp ảnh 30 giây một lần
   - Đợi 30 giây trước khi chụp ảnh tiếp theo
      ```cpp
     void loop() 
     {
     takeSavePhoto(); 
     delay(30000); 
     }
     
## F. Tác giả
### Thành viên trong nhóm
   - Trần Đức Hoàng Anh - Link github:
   - Nguyễn Văn Minh - Link github:
   - Mai Thế Tuấn - Link github: 
   - Trịnh Hà Trung - Link github: https://github.com/Trunng5703
### Lời cảm ơn đến: 
    - https://randomnerdtutorials.com/esp32-cam-take-photo-save-microsd-card/
    - https://www.instructables.com/ESP32-CAM-Take-Photo-and-Save-to-MicroSD-Card-With/
    
