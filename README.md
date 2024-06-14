# ESP32-CAM with DS1307 RTC

## A. Giới thiệu

- Chụp ảnh và lưu trữ vào thẻ SD
- Sử dụng module DS1307 RTC để gắn timestamp vào ảnh
- Điều khiển LED bật khi chụp ảnh
- Nhấn reset để chụp ngay lập tức

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
   - Kết nối camera với ESP32-CAM
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

### Trạng thái của nút chụp ảnh được xác định thông qua cơ chế ngắt ở GPIO.
     ```cpp


### Thời gian hiện tại từ DS1307 RTC được lấy và gắn vào tên file ảnh.
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

### LED được bật khi bắt đầu chụp ảnh và tắt khi quá trình chụp hoàn tất.
    ```cpp
    pinMode(4, OUTPUT);
    digitalWrite(4, LOW);
    rtc_gpio_hold_en(GPIO_NUM_4);

### Ảnh được lưu vào thẻ SD với tên file bao gồm timestamp.
    ```cpp
    void takeSavePhoto() {

    // Take Picture with Camera
    camera_fb_t * fb = esp_camera_fb_get();
    if (!fb) {
    Serial.println("Camera capture failed");
    delay(1000);
    ESP.restart();
    }

    // Path where new picture will be saved in SD Card
    String path = getPictureFilename();
    Serial.printf("Picture file name: %s\n", path.c_str());

    // Save picture to microSD card
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

    // Turn off the LED
    digitalWrite(LED_PIN, LOW);
    }


    
## F. Cảm ơn
