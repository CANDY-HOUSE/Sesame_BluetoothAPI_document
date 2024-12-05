---
nav: 例示
group: SESAME API
type:
  title: Bluetooth
  order: 0
title: 1_registor（註冊設備）
order: 1
---

# 1 レジスト（註冊設備）

レジスト（註冊）は APP と Sesame5 がペアリングする行為で、ペアリングした APP だけがその Sesame5 を制御することができます。

暗号化の詳細は `セキュリティレイヤー`を参照してください。

## Sesame(APP)が Sesame5 に登録するシーケンス図

<p align="left" >
  <img src="./src/register/ssm5註冊_循序圖.png" alt="" title="">
</p>

## Sesame(APP)が Sesame5 に登録するアクティビティ図

<p align="left" >
  <img src="./src/register/ssm5註冊_活動圖.png" alt="" title="">
</p>

## モバイルからのデータ送信

| バイト |         68 ~ 65          |   64 ~ 1   |     0     |
| ------ | :----------------------: | :--------: | :-------: |
| データ | モバイルのタイムスタンプ | publicKeyA | item_code |

## Sesame5 からのレスポンスデータ（既に登録済み）

| バイト |        2         |     1     |   0    |
| ------ | :--------------: | :-------: | :----: |
| データ |       res        | item_code |  type  |
| 説明   | コマンド処理状態 | 指令番号  | 押送型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_LOGIN (1)

res : CMD_RESULT_INVALID_ACTION (0x09)

## Sesame5 からのレスポンスデータ（未登録)

| バイト |       N ~ 3        |        2         |     1     |   0    |
| ------ | :----------------: | :--------------: | :-------: | :----: |
| データ |      payload       |       res        | item_code |  type  |
| 説明   | モバイルへのデータ | コマンド処理状態 | 指令番号  | 押送型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_LOGIN (1)

res : CMD_RESULT_SUCCESS (0x00)

payload : 詳細は以下のテーブルを参照

### Payload

| バイト |  76 ~ 13   |   12 ~ 7    |   6 ~ 0    |
| ------ | :--------: | :---------: | :--------: |
| データ | publicKeyS | mechSetting | mechStatus |

## iOS、Android、ESP32 の例

<CustomBashOSPlatformRegistor 
  ios='true'
  android='true' 
  esp32='true'
/>
