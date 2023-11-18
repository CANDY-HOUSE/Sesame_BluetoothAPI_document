# Item: Remove Sesame

手機發送刪除指令及設備名稱，sesame5 回覆指令成功，ssm_touch 主動推送 Sesame 列表給手機。(Sesame 列表詳見 `102_pub_ssm_key`)

## 循序圖

<p align="left" >
  <img src="../src/rm_sesame/rm_sesame.png" alt="" title="">
</p>

## 手機送出資料

| Byte | 16   ~    1 |     0     |
|------|:-----------:|:---------:|
| Data | device_name | item code |

item code : SSM3_ITEM_REMOVE_SESAME (103)

## ssm_touch 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM3_ITEM_REMOVE_SESAME (103)

res : CMD_RESULT_SUCCESS (0x00)