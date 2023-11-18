# Item: Card Change

1. ssm_touch 加入新卡，主動推送新卡ID及名字給手機。

2. 手機將id及新名稱傳給 ssm_touch，ssm_touch 回傳命令接收成功後修改卡片名稱，並主動推送新卡ID及名字給手機(
   名字超過20Bytes，只取前20Bytes)。

## 循序圖 (新增卡片)

<p align="left" >
  <img src="../src/card_change/card_change.png" alt="" title="">
</p>

## 循序圖 (修改卡片名稱)

<p align="left" >
  <img src="../src/card_change/card_change_name.png" alt="" title="">
</p>

## 手機送出資料

| Byte | N   ~    1 |     0     |
|------|:----------:|:---------:|
| Data |  payload   | item code |

item code : SSM_OS3_CARD_CHANGE (107)

payload : 詳見以下表格

### payload

| Byte | (Card Name Len + Card ID Len + 1) ~ (Card ID Len + 2) | Card ID Len + 1 | Card ID Len ~ 1 |      0      |
|:----:|:-----------------------------------------------------:|:---------------:|:---------------:|:-----------:|
| Data |                       Card Name                       |  Card Name Len  |     Card ID     | Card ID Len |

#### 範例

id_len = 5

name_len = 4

| Byte |  10 ~ 7   |       6       |  5 ~ 1  |      0      |
|:----:|:---------:|:-------------:|:-------:|:-----------:|
| Data | Card Name | Card Name Len | Card ID | Card ID Len |

## ssm_touch 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM_OS3_CARD_CHANGE (107)

res : CMD_RESULT_SUCCESS (0x00)

## ssm_touch 推送內容

| Byte |  N ~ 2  |     1     |  0   |
|------|:-------:|:---------:|:----:|
| Data | payload | item_code | type |
| 說明   | 送給手機的資料 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_PUBLISH (0x08)

item code : SSM_OS3_CARD_CHANGE (107)

payload : 詳見以下表格

### payload

| Byte | (Card Name Len + Card ID Len + 1) ~ (Card ID Len + 2) | Card ID Len + 1 | Card ID Len ~ 1 |      0      |
|:----:|:-----------------------------------------------------:|:---------------:|:---------------:|:-----------:|
| Data |                       Card Name                       |  Card Name Len  |     Card ID     | Card ID Len |

#### 範例

id_len = 5

name_len = 4

| Byte |  10 ~ 7   |       6       |  5 ~ 1  |      0      |
|:----:|:---------:|:-------------:|:-------:|:-----------:|
| Data | Card Name | Card Name Len | Card ID | Card ID Len |

