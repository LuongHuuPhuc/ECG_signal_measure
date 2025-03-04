# ECG Signal Measure
#### 2025/03/03 
#### Lương Hữu Phúc
*** 
# **Giới thiệu sơ lược dự án**
- Dự án nhỏ sử dụng cảm biến AD8232 để đo tín hiệu điện tâm đồ (ECG) thông qua bộ ADC 12-bit của MCU Esp32
- Dự án này sẽ tiếp tục được phát triển và tối ưu hơn về code + hiển thị kết quả ra màn hình OLED + sử dụng wifi để gửi thông tin lên Sever
# **Sơ lược về tín hiệu ECG(Electrocardiography) - Điện tâm đồ**
- Theo [Wikipedia](https://en.wikipedia.org/wiki/Electrocardiography), điện tâm đồ là tín hiệu điện được ghi lại từ hoạt động điện của tim. Nó thể hiện sự thay đổi điện thế trong tim khi các tế bào co bóp và thư giãn. Tín hiệu ECG có thể được dùng để chẩn đoán các bệnh lý tim mạch như rối loạn nhịp tim, huyết áp cao,...
![image](https://github.com/user-attachments/assets/38c37543-2de3-402f-b32c-7d06bfd1d03a)
## Quá trình hình thành các đỉnh trong ECG
1. P-wave (Đỉnh P)
   * Đại điện cho sự kích thích tâm nhĩ, là quá trình điện thế lan truyền qua các tâm nhĩ, khiễn chúng co lại
   * P-wave là 1 sóng nhỏ và dương
2. QRS complex (Phức hợp QSR)
   * Là quá trình kích thích tâm thất, là khi điện thế lan truyền qua các tâm thất, khiến chúng co lại và bơm máy ra khỏi tim
   * Phức hợp bao gồm 3 phần chính: Q-wave, R-wave, S-wave:
     * Q-wave là sóng nhỏ đầu tiên, âm, phản ánh sự kích thích của vách giữa 2 tâm thất.
     * R-wave: Sóng cao, dương, phản ánh sự kích thích chủ yếu của các tâm thất
     * S-wave: Sóng âm sau R-wave, phản ánh sự kích thích của vùng dưới đáy tâm thất.
3. T-wave
   * Đại diện cho quá trình tái phân cực của tâm thất, khi tâm thất trở về trạng thái điện lý chuẩn bị cho chu kỳ tim tiếp theo
   * T-wave là sóng dương, có hình dạng dài
5. U-wave
   * Là một sóng nhỏ thường xuất hiện sau T-wave, nhưng không phải lúc nào cũng thấy được trên tất cả các ECG. Đỉnh U có thể liên quan đến quá trình tái phân cực của cơ tim hoặc các yếu tố như nồng độ kali trong máu.
![image](https://github.com/user-attachments/assets/be0cb6b4-8795-422f-94d1-9be2bbb03a11)
## **Chi tiết phần cứng - phần mềm sử dụng**
## 1. Hardware
* ## **01 Esp32 - 38 pins, 4MB flash memory, 520KB SRAM, sử dụng nguồn cấp bằng cáp micro-USB** <br>
![image](https://github.com/user-attachments/assets/07d970ef-ad88-44d2-b055-c6992b4bbfbc)
* ## **01 Module cảm biến ECG AD8232** <br> 
![image](https://github.com/user-attachments/assets/7e074dd4-e20f-422f-9068-c062b013bce5)
### Thông số và pinouts của AD8232 <br>
* Thông số
   - Nguồn cấp tối ưu: 3.3V
   - Dòng điện tiêu thụ: 170uA (thấp, tiết kiệm năng lượng)
   - Điện áp đầu ra: 0.9V - 1.5V, trung bình khoảng 1.2V
   - Dải tần hoạt động: 0.5Hz - 40Hz
   - Tích hợp Low Pass Filter (~0.5Hz): Loại bỏ trôi nền
   - Tích hợp High Pass Filter (~40Hz): Loại bỏ nhiễu tần số cao

|         Pinouts          |                 Chức năng                |
---------------------------|-------------------------------------------
|       RA(Right Arm)      |          Kết nối điện cực tay phải       |
|       LA(Left Arm)       |          Kết nối điện cực tay trái       |
|       RL(Right Leg)      |         Kết nối điện cực chân phải       |
|         Output           |     Tín hiệu ECG đã được khuếch đại      |
|                          |                                          |
|         LO+/LO-          |      Phát hiện mất kết nối điện cực      |
|                          |           (HIGH = mất tín hiệu)          |
|                          |                                          |
|                          |       (Chế độ tiết kiệm năng lượng)      |
|      SDN (Shutdown)      |         Điều khiển tắt mở cảm biến       |
|                          |          HIGH = bật, LOW = tắt           |

* ## **01 Dây kết nối điểm cực - đầu ra Jack 3.5m** <br>
![image](https://github.com/user-attachments/assets/8e1e8e0a-d897-49c6-962c-8e426ac3abfb)
* ## **01 Mạch nguồn ổn áp AMS117 (7v-12V input, 3.3V-5V output)** <br>
![image](https://github.com/user-attachments/assets/4f27250f-4f5f-4308-a1d1-60224184b1d5)
> Nên cấp nguồn ngoài cho cảm biến vì cảm biến nhiễu với môi trường, nguồn điện trực tiếp từ MCU và laptop, gây méo dạng dữ liệu !
* ## **01 Cáp nguồn USB to USB có hỗ trợ chống nhiễu Ferrite Bead** <br>
 ![image](https://github.com/user-attachments/assets/68fe1ff4-7e74-4732-9820-9a13913619eb)
* ## **Dây Jumper male-male (đực - đực)**
![image](https://github.com/user-attachments/assets/ba09ff8a-18e9-4bbb-83cc-cea74f1418c5)
## Sơ dồ kết nối mạch (Components Diagram)
![image](https://github.com/user-attachments/assets/548c5a26-caeb-46bb-8cf6-d17bcdca2573)

|          AD8232          |           Esp32 MCU           | Mạch ổn áp ASM117  |
---------------------------|-------------------------------|---------------------
|       RA(Right Arm)      | (Nếu dùng jack thì không cần) |          x         | 
|       LA(Left Arm)       | (Nếu dùng jack thì không cần) |          x         |
|       RL(Right Leg)      | (Nếu dùng jack thì không cần) |          x         |
|         Output           |         GPIO34 (ADC1_6)       |          x         |
|          LO+             |            GPIO23             |          x         |
|          LO-             |            GPIO35             |          x         |
|      SDN (Shutdown)      |  GND hoặc bỏ không (optional) |          x         |
|          3.3V            |         Không kết nối         |         3.3V       |
|          GND             |         Không kết nối         |         GND        |

- Cấp nguồn từ laptop Msi GF63 Thin 11UC cho mạch ổn áp và MCU
## **Kết nối các điện cực đến cơ thể**
![image](https://github.com/user-attachments/assets/31809a90-f88f-43a9-beec-f5ba67188a67)
> Kết nối các cực càng gần tim thì tín hiệu thu được càng rõ
## 2. Software/Firmware
* ## **Visual Studio Code**
![image](https://github.com/user-attachments/assets/abdbcfe6-f191-48c9-b9f2-4fa6c3f02d83)
* ## **Framework ESP-IDF của Espressif**
  - Framework hỗ trợ đầy đủ các thư viện để lập trình firmware cho các dòng Esp32, viết bằng ngôn ngữ C, Python
  - Tích hợp trình CLI (Command Line Interface) để tương tác với người dùng
![image](https://github.com/user-attachments/assets/cb6e2dec-7bd7-4ebc-b4a4-b83cda8a1141)
* ## **CoolTerm64**
  - Phần mềm hỗ trợ hiển thị số liệu bằng UART
  - Có thể hiển thị được Plain Text, Chart,...
![image](https://github.com/user-attachments/assets/ac777887-2218-4d1d-892d-53ea8cae35e3)
# **Kết quả thu được (Result)**
- Kết quả hiện thị số liệu trực quan bằng đồ thị bằng CoolTerm64 và dạng dữ liệu text 
- Tần số lấy mẫu 100Hz
- Sử dụng thêm thuật toán bộ lọc thông thấp họ Butterworth (Thuật toán bộ lọc ở repo này của tôi [PCG_signal_measure](https://github.com/LuongHuuPhuc/PCG_signal_measure)) để làm suy giảm biên độ của các tần số > 45Hz để thu được tín hiệu mịn và rõ ràng hơn.
* Hình ảnh thu lại trực tiếp từ CoolTerm64
![image](https://github.com/user-attachments/assets/c8b8f74f-edb6-4e29-8132-d415c2ccfb72)
* Hình ảnh được vẽ lại bằng Python
![image](https://github.com/user-attachments/assets/02cd2c59-4efe-4380-878b-e3333bfe023d)

# **Đánh giá kết quả thu được** 


