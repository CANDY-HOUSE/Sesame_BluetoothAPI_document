# Bluetooth LE Advertising

以下為 Sesame5 廣播中的欄位及各欄位中每個位元所代表的含意。

## CANDY HOUSE, Inc.

| CANDY HOUSE, Inc. | Decimal |  Hex   |
|:-----------------:|:-------:|:------:|
|   Service UUID    |  64897  | 0xFD81 |
|    Company ID     |  1370   | 0x055A |

## Service UUID (0x03)

Sesame5 服務 UUID : 0xFD81

## Manufacturer Data (0xFF)

| Byte | 20 ~ 5 |  4   | 3  |  2   | 1 ~ 0 |
|------|:------:|:----:|:--:|:----:|:-----:|
| Data | IC 序號  | 是否註冊 | 保留 | 產品編號 | 公司代碼  |

公司代碼 : 0x055A

| 產品編號(Decimal) |        機型        |
|:----:|:----------------:|
|  0   |     Sesame 3     |
|  1   |  WiFi Module 2   |
|  2   |   Sesame Bot 1   |
|  3   |  Sesame Bike 1   |
|  4   |     Sesame 4     |
|  5   |     Sesame 5     |
|  6   |  Sesame Bike 2   |
|  7   |    Sesame Pro    |
|  8   |   Open Sensor    |
|  9   | Sesame Touch Pro |
|  10  |   Sesame Touch   |
|  11  |   BLE Connector  |
|  12  |   APAMAN Locker  |
|  13  |   Hub 3          |
|  14  |   Remote         |
|  15  |   Remote Nano    |
|  16  |   Sesame 5 US    |
|  17  |   Sesame Bot 2   |
|  18  |   F2             |
|  19  |   F1             |
|  20  |Bluetooth半導体光リレー|

