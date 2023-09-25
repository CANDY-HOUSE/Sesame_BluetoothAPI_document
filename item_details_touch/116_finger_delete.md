# Item: Finger Delete

手機端傳送要刪除的指紋ID，ssm_touch 回傳成功訊息，刪除指定指紋。

## 循序圖
<p align="left" >
  <img src="../src/finger_delete/finger_delete.png" alt="" title="">
</p>

## 手機送出資料
| Byte |     1     | 0         |
|------|:---------:|:---------:|
| Data |  finger_id  | item code |

item code : SSM_OS3_FINGERPRINT_DELETE (116)

finger_id : 要刪除的指紋ID

## ssm_touch 回傳內容
| Byte  | 2      | 1         | 0    |
|-------|:------:|:---------:|:----:|
| Data  | res    | item_code | type |
| 說明   | 命令處裡狀態 | 指令編號      | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM_OS3_FINGERPRINT_DELETE (115)

res : CMD_RESULT_SUCCESS (0x00)

