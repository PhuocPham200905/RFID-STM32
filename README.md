# STM32F103C8T6 giao tiếp RFID RC522 qua SPI

Project này minh họa cách sử dụng module RFID RC522 với vi điều khiển STM32F103C8T6 thông qua giao tiếp SPI. Chương trình khởi tạo module MFRC522, liên tục quét thẻ RFID và bật/tắt LED PC13 khi đọc được UID hợp lệ.

## Nội dung project

```text
.
|-- code/
|   |-- main.c       # Phần code cần chèn vào các vùng USER CODE của STM32CubeIDE
|   |-- rc522.c      # Driver giao tiếp MFRC522 qua SPI HAL
|   `-- rc522.h      # Khai báo thanh ghi, lệnh và hàm driver
|-- schematic/
|   |-- schematic.jpg
|   `-- schematic_easyeda.json
`-- BÁO CÁO GIAO TIẾP SPI GIỮA MODULE RFID RC522 VÀ STM32F103C8T6.pdf
```

## Phần cứng sử dụng

- STM32F103C8T6 Blue Pill
- Module RFID RC522
- Thẻ/tag RFID Mifare 13.56 MHz
- Breadboard và dây cắm
- Nguồn 3.3V

> Lưu ý: RC522 hoạt động ở mức logic 3.3V. Không cấp 5V trực tiếp cho module RC522.

## Kết nối phần cứng

| RC522 | STM32F103C8T6 | Ghi chú |
| --- | --- | --- |
| SDA / NSS | PA4 | SPI1 CS |
| SCK | PA5 | SPI1 SCK |
| MOSI | PA7 | SPI1 MOSI |
| MISO | PA6 | SPI1 MISO |
| RST | PB0 | Reset RC522 |
| 3.3V | 3.3V | Nguồn module |
| GND | GND | Mass chung |

LED on-board PC13 được dùng để báo hiệu khi phát hiện thẻ.

## Cấu hình STM32CubeIDE

1. Tạo project cho chip `STM32F103C8T6`.
2. Bật `SPI1` ở chế độ `Full-Duplex Master`.
3. Cấu hình các chân:
   - `PA4`: GPIO Output, dùng làm CS/NSS cho RC522.
   - `PB0`: GPIO Output, dùng làm RST cho RC522.
   - `PC13`: GPIO Output, điều khiển LED on-board.
4. Generate code bằng STM32CubeMX/STM32CubeIDE.
5. Thêm `rc522.c` vào thư mục source và `rc522.h` vào thư mục include của project.
6. Chèn các phần trong `code/main.c` vào đúng vùng `USER CODE` của file `main.c` do CubeIDE tạo.

## Cách sử dụng driver

Khai báo header và biến lưu UID:

```c
#include "rc522.h"

uint8_t CardID[5];
uint8_t status;
```

Khởi tạo RC522 sau khi HAL và SPI đã được khởi tạo:

```c
MFRC522_Init();
```

Trong vòng lặp chính, kiểm tra thẻ RFID:

```c
status = MFRC522_Check(CardID);

if (status == MI_OK) {
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
  HAL_Delay(500);
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
}
```

Khi có thẻ hợp lệ trong vùng đọc, hàm `MFRC522_Check()` trả về `MI_OK` và UID 5 byte được lưu trong mảng `CardID`.

## Các hàm chính

| Hàm | Chức năng |
| --- | --- |
| `MFRC522_Init()` | Reset và cấu hình module RC522 |
| `MFRC522_Check(uint8_t *id)` | Quét thẻ và đọc UID |
| `MFRC522_Request()` | Kiểm tra có thẻ trong vùng đọc hay không |
| `MFRC522_Anticoll()` | Đọc UID và kiểm tra checksum |
| `MFRC522_Halt()` | Đưa thẻ về trạng thái halt |
| `MFRC522_ReadRegister()` | Đọc thanh ghi MFRC522 qua SPI |
| `MFRC522_WriteRegister()` | Ghi thanh ghi MFRC522 qua SPI |

## Kiểm tra hoạt động

1. Nạp chương trình vào STM32F103C8T6.
2. Đặt thẻ RFID gần module RC522.
3. Nếu đọc thẻ thành công, LED PC13 sẽ sáng/tắt trong 500 ms.

Nếu module không đọc được thẻ, hãy kiểm tra lại:

- RC522 đã được cấp đúng 3.3V.
- Chân SPI1 đúng với bảng kết nối.
- Chân CS và RST trong `rc522.h` trùng với cấu hình CubeMX.
- Dây GND của STM32 và RC522 đã nối chung.

## Tài liệu kèm theo

- Sơ đồ mạch: `schematic/schematic.jpg`
- File EasyEDA: `schematic/schematic_easyeda.json`
- Báo cáo project: `BÁO CÁO GIAO TIẾP SPI GIỮA MODULE RFID RC522 VÀ STM32F103C8T6.pdf`
