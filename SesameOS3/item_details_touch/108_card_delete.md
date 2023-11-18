# Item: Card Delete

手機發送刪除指令及卡片ID，ssm_touch 回覆指令成功並刪除指定卡片。

## 循序圖

<p align="left" >
  <img src="../src/card_delete/card_delete.png" alt="" title="">
</p>

## 手機送出資料

| Byte | 16   ~    1 |     0     |
|------|:-----------:|:---------:|
| Data |   Card ID   | item code |

item code : SSM_OS3_CARD_DELETE (108)

## ssm_touch 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM_OS3_CARD_DELETE (108)

res : CMD_RESULT_SUCCESS (0x00)