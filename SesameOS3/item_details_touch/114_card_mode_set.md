# Item: Card Mode Set

手機發送新增指令獲取ssm_touch現在處於新增或是驗證卡片模式，sesame5 回覆指令成功及模式。

## 循序圖

<p align="left" >
  <img src="../src/card_mode_set/card_mode_set.png" alt="" title="">
</p>

## 手機送出資料

| Byte |     1     |     0     |
|------|:---------:|:---------:|
| Data | card_mode | item code |

item code : SSM_OS3_CARD_MODE_SET (114)

## ssm_touch 回傳內容

| Byte |     3     |  2  |     1     |  0   |
|------|:---------:|:---:|:---------:|:----:|
| Data | card mode | res | item code | type |

type : SSM2_OP_CODE_RESPONSE(0x07)

item code : SSM_OS3_CARD_MODE_SET (114)

res : CMD_RESULT_SUCCESS (0x00)

## card mode

0x00->驗證模式

0x01->新增模式