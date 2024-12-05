---
nav: 例
group: SESAME API
type:
  title: Bluetooth
  order: 0
title: 101_セサミ追加（新規）
order: 101
---

# 101 セサミ追加（新規）

携帯電話から新規命令とトークンを送信し、sesame5 が命令成功を返答し、ssm_touch が主動的にセサミのリストを携帯電話に送信します。（セサミのリストの詳細は`102_pub_ssm_key`をご覧ください）

## アクティビティ図（新規セサミ 4 を追加）

Sesame4 の Bluetooth アドレスは送信情報で解析できないため、周囲の Bluetooth デバイスをスキャンして Sesame4 の Bluetooth アドレスを見つけ出します。

<p align="left" >
  <img src="./src/add_sesame/add_sesame4_act.png" alt="" title="">
</p>

## アクティビティ図（新規セサミ 5 を追加）

Sesame5 の Bluetooth アドレスは送信情報で解析できるため、周囲の Bluetooth デバイスをスキャンする必要はありません。

sessionKey = AES_CMAC(key:device_name, input:"candy")

address = sessionKey[0〜5]

address[5] |= 0xC0

<p align="left" >
  <img src="./src/add_sesame/add_sesame5_act.png" alt="" title="">
</p>

## シーケンス図

<p align="left" >
  <img src="./src/add_sesame/add_sesame.png" alt="" title="">
</p>

## 携帯が送信するデータ

| Byte |  32 ~ 17  |   16 ~ 1    |     0     |
| ---- | :-------: | :---------: | :-------: |
| Data | secretKey | device_name | item code |

item code : SSM3_ITEM_ADD_SESAME (101)

## ssm_touch の返信内容

| Byte |      2       |     1     |       0        |
| ---- | :----------: | :-------: | :------------: |
| Data |     res      | item_code |      type      |
| 説明 | 命令処理状態 | 指令番号  | プッシュの種類 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM3_ITEM_ADD_SESAME (101)

res : CMD_RESULT_SUCCESS (0x00)

## iOS、Android、ESP32 の例

<CustomBashOSPlatformAddSesame ios='true' android='true'  esp32='true'/>
