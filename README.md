# STM32F103C8T6 giao tiep RFID RC522 qua SPI

Project nay minh hoa cach su dung module RFID RC522 voi vi dieu khien STM32F103C8T6 thong qua giao tiep SPI. Chuong trinh khoi tao module MFRC522, lien tuc quet the RFID va bat/tat LED PC13 khi doc duoc UID hop le.

## Noi dung project

```text
.
|-- code/
|   |-- main.c       # Phan code can chen vao cac vung USER CODE cua STM32CubeIDE
|   |-- rc522.c      # Driver giao tiep MFRC522 qua SPI HAL
|   `-- rc522.h      # Khai bao thanh ghi, lenh va ham driver
|-- schematic/
|   |-- schematic.jpg
|   `-- schematic_easyeda.json
`-- BAO CAO GIAO TIEP SPI GIUA MODULE RFID RC522 VA STM32F103C8T6.pdf
```

## Phan cung su dung

- STM32F103C8T6 Blue Pill
- Module RFID RC522
- The/tag RFID Mifare 13.56 MHz
- Breadboard va day cam
- Nguon 3.3V

> Luu y: RC522 hoat dong muc logic 3.3V. Khong cap 5V truc tiep cho module RC522.

## Ket noi phan cung

| RC522 | STM32F103C8T6 | Ghi chu |
| --- | --- | --- |
| SDA / NSS | PA4 | SPI1 CS |
| SCK | PA5 | SPI1 SCK |
| MOSI | PA7 | SPI1 MOSI |
| MISO | PA6 | SPI1 MISO |
| RST | PB0 | Reset RC522 |
| 3.3V | 3.3V | Nguon module |
| GND | GND | Mass chung |

LED on-board PC13 duoc dung de bao hieu khi phat hien the.

## Cau hinh STM32CubeIDE

1. Tao project cho chip `STM32F103C8T6`.
2. Bat `SPI1` o che do `Full-Duplex Master`.
3. Cau hinh cac chan:
   - `PA4`: GPIO Output, dung lam CS/NSS cho RC522.
   - `PB0`: GPIO Output, dung lam RST cho RC522.
   - `PC13`: GPIO Output, dieu khien LED on-board.
4. Generate code bang STM32CubeMX/STM32CubeIDE.
5. Them `rc522.c` vao thu muc source va `rc522.h` vao thu muc include cua project.
6. Chen cac phan trong `code/main.c` vao dung vung `USER CODE` cua file `main.c` do CubeIDE tao.

## Cach su dung driver

Khai bao header va bien luu UID:

```c
#include "rc522.h"

uint8_t CardID[5];
uint8_t status;
```

Khoi tao RC522 sau khi HAL va SPI da duoc khoi tao:

```c
MFRC522_Init();
```

Trong vong lap chinh, kiem tra the RFID:

```c
status = MFRC522_Check(CardID);

if (status == MI_OK) {
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
  HAL_Delay(500);
  HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
}
```

Khi co the hop le trong vung doc, ham `MFRC522_Check()` tra ve `MI_OK` va UID 5 byte duoc luu trong mang `CardID`.

## Cac ham chinh

| Ham | Chuc nang |
| --- | --- |
| `MFRC522_Init()` | Reset va cau hinh module RC522 |
| `MFRC522_Check(uint8_t *id)` | Quet the va doc UID |
| `MFRC522_Request()` | Kiem tra co the trong vung doc hay khong |
| `MFRC522_Anticoll()` | Doc UID va kiem tra checksum |
| `MFRC522_Halt()` | Dua the ve trang thai halt |
| `MFRC522_ReadRegister()` | Doc thanh ghi MFRC522 qua SPI |
| `MFRC522_WriteRegister()` | Ghi thanh ghi MFRC522 qua SPI |

## Kiem tra hoat dong

1. Nap chuong trinh vao STM32F103C8T6.
2. Dat the RFID gan module RC522.
3. Neu doc the thanh cong, LED PC13 se sang/tat trong 500 ms.

Neu module khong doc duoc the, hay kiem tra lai:

- RC522 da duoc cap dung 3.3V.
- Chan SPI1 dung voi bang ket noi.
- Chan CS va RST trong `rc522.h` trung voi cau hinh CubeMX.
- Day GND cua STM32 va RC522 da noi chung.

## Tai lieu kem theo

- So do mach: `schematic/schematic.jpg`
- File EasyEDA: `schematic/schematic_easyeda.json`
- Bao cao project: `BAO CAO GIAO TIEP SPI GIUA MODULE RFID RC522 VA STM32F103C8T6.pdf`
