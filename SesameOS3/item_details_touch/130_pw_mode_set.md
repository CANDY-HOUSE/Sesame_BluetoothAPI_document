# Item: Password Mode Set

手機發送新增指令獲取ssm_touch現在處於新增或是驗證密碼模式，sesame5 回覆指令成功及模式。

## 循序圖

<p align="left" >
  <img src="../src/pw_mode_set/pw_mode_set.png" alt="" title="">
</p>

## 手機送出資料

| Byte |    1    |     0     |
|------|:-------:|:---------:|
| Data | pw_mode | item code |

item code : SSM_OS3_PASSCODE_MODE_SET (130)

## ssm_touch 回傳內容

| Byte |    3    |  2  |     1     |  0   |
|------|:-------:|:---:|:---------:|:----:|
| Data | pw mode | res | item code | type |

type : SSM2_OP_CODE_RESPONSE(0x07)

item code : SSM_OS3_PASSCODE_MODE_SET (130)

res : CMD_RESULT_SUCCESS (0x00)

## pw mode

0x00->驗證模式

0x01->新增模式