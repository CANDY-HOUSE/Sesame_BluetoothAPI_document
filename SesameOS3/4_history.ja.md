---
nav: サンプル
group: SESAME API
type:
  title: Bluetooth
  order: 0
title: 4_history (履歴)
order: 4
---

# 4 履歴

携帯電話から履歴リクエストコマンドを送信し、Sesame5 はフラッシュ内の最も古い履歴を返し、その履歴をフラッシュから削除します。

<!-- Sesame5のブロードキャストには、履歴タグを読み取る必要があるかどうかのフラグが含まれています。詳細は広告欄の説明をご覧ください。 -->

## シーケンス図

<p align="left" >
  <img src="./src/history/history.png" alt="" title="">
</p>

## 携帯電話からのデータ送信

| Byte |    1    |     0     |
| ---- | :-----: | :-------: |
| Data | is peek | item code |

item code : SSM2_ITEM_CODE_HISTORY (4)

is peek == true : 最新の履歴を表示し、履歴を削除しない

is peek == false : 最も古い履歴を 1 つポップアップし、その履歴を削除する

## ssm5 返信内容

### if (is peek == true || (is peek == true && flash に履歴がある場合))

| Byte |          N ~ 3           |    2     |       1        |       0        |
| ---- | :----------------------: | :------: | :------------: | :------------: |
| Data |        ペイロード        |   res    | アイテムコード |     タイプ     |
| 説明 | 携帯電話に送信するデータ | 処理状態 |  コマンド番号  | プッシュタイプ |

タイプ : SSM2_OP_CODE_RESPONSE (0x07)

アイテムコード : SSM2_ITEM_CODE_HISTORY(4)

res : CMD_RESULT_SUCCESS (0x00)

ペイロード (History): 以下は履歴データの形式です。

#### payload

| Byte |     N ~ 16     |   15 ~ 9    |     8 ~ 5      |     4      |         3 ~ 0          |
| ---- | :------------: | :---------: | :------------: | :--------: | :--------------------: |
| Data |     param      | mech_status |       ts       |    type    |           id           |
| 説明 | 履歴タグと長さ |  機械状態   | タイムスタンプ | 履歴タイプ | データが何個目の履歴か |

#### param

| Byte |  32 ~ 1  |      0      |
| ---- | :------: | :---------: |
| Data |   data   | data_length |
| 説明 | 履歴タグ | タグの長さ  |

### if (is peek == true && flash に履歴がない場合)

| Byte |    2     |       1        |       0        |
| ---- | :------: | :------------: | :------------: |
| Data |   res    | アイテムコード |     タイプ     |
| 説明 | 処理状態 |  コマンド番号  | プッシュタイプ |

タイプ : SSM2_OP_CODE_RESPONSE (0x07)

アイテムコード : SSM2_ITEM_CODE_HISTORY(4)

res : CMD_RESULT_NOT_FOUND (0x05)

```c

#pragma pack(1)
union ss5_his_param {
    int8_t data_length;
    int8_t data[32];
};
#pragma pack()

#pragma pack(1)
typedef struct {
    uint32_t id;                    /// 4 Bytes
    uint8_t type;                   /// 1 Bytes
    uint32_t ts;                    /// 4 Bytes
    mech_status_t mech_status;      /// 7 Bytes
    union ss5_his_param param;      /// 32 Bytes
} ssm_history;                      /// 4+1+4+7+32 = 48 Bytes
#pragma pack()

```

## iOS、Android、ESP32 の例

 <CustomBashOSPlatformHistory ios='true' android='true'  esp32='true'/>

<!-- ## Androidの例

```jsx | pure
    private fun readHistoryCommand(result: CHResult<CHEmpty>) {
        if (checkBle(result)) return

        sendEncryptCommand(SSM2Payload(SSM2OpCode.read, SesameItemCode.history, if (isInternetAvailable()) byteArrayOf(0x01) else byteArrayOf(0x00))) { res ->
            if (res.cmdResultCode == SesameResultCode.success.value) {
                if (isInternetAvailable()) {
//                    L.d("hcia", "deviceId.toString().uppercase():" + deviceId.toString().uppercase())
                    CHAccountManager.postSS2History(deviceId.toString().uppercase(), res.payload.toHexString()) {}
                }
                val recordId = res.payload.sliceArray(0..3).toBigLong().toInt()
                var historyType = Sesame2HistoryTypeEnum.getByValue(res.payload[4]) ?: Sesame2HistoryTypeEnum.NONE
                val newTime = res.payload.sliceArray(5..12).toBigLong() //4
//                L.d("hcia", "newTime:" + newTime)
                val historyContent = res.payload.sliceArray(13..res.payload.count() - 1)

                if (historyType == Sesame2HistoryTypeEnum.BLE_LOCK) {
                    val payload22 = historyContent.sliceArray(18..39)
                    val locktype = payload22[0] / 30
                    if (locktype == 1) {
                        historyType = Sesame2HistoryTypeEnum.WEB_LOCK
                    }
                    if (locktype == 2) {
                        historyType = Sesame2HistoryTypeEnum.WEB_LOCK
                    }
                    historyContent[18] = (payload22[0] % 30).toByte()

                }
                if (historyType == Sesame2HistoryTypeEnum.BLE_UNLOCK) {
                    val payload22 = historyContent.sliceArray(18..39)
                    val locktype = payload22[0] / 30
                    if (locktype == 1) {
                        historyType = Sesame2HistoryTypeEnum.WEB_UNLOCK
                    }
                    if (locktype == 2) {
                        historyType = Sesame2HistoryTypeEnum.WEB_UNLOCK
                    }
                    historyContent[18] = (payload22[0] % 30).toByte()
                }

                val chHistoryEvent: CHHistoryEvent = parseHistoryContent(historyType, historyContent, newTime, recordId)
                val historyEventToUpload: ArrayList<CHHistoryEvent> = ArrayList()

                historyEventToUpload.add(chHistoryEvent)
                val chHistorysToUI = ArrayList<CHSesame2History>()
                historyEventToUpload.forEach {
                    val ss2historyType = Sesame2HistoryTypeEnum.getByValue(it.type) ?: Sesame2HistoryTypeEnum.NONE
                    val ts = it.timeStamp
                    val recordID = it.recordID
                    val histag = it.historyTag?.base64decodeByteArray()
                    val tmphis = eventToHistory(ss2historyType, ts, recordID, histag)
                    if (tmphis != null) {
                        chHistorysToUI.add(tmphis)
                    }
                }

                historyCallback?.invoke(Result.success(CHResultState.CHResultStateBLE(Pair(chHistorysToUI.toList(), null))))
                if (isInternetAvailable()) {
                    this.readHistoryCommand {}
                }
            } else {
                historyCallback?.invoke(Result.failure(NSError(res.cmdResultCode.toString(), "CBCentralManager", res.cmdResultCode.toInt())))
            }
        }
    }

```

## iOSの例
```jsx | pure
    func readHistoryCommand(_ result: @escaping (CHResult<CHEmpty>))  {
        if (self.checkBle(result)) { return }
//        L.d("🌇 履歴を読む")
        URLSession.isInternetReachable { isInternetReachable in
            let deleteHistory = isInternetReachable == true ? "01":"00"

            self.sendCommand(.init(.read, .history, deleteHistory.hexStringtoData())) { (result) in

                if result.cmdResultCode == .success {

                    let histitem = result.data.copyData

                    guard let recordId = histitem[safeBound: 0...3]?.copyData,
                          let type = histitem[safeBound: 4...4]?.copyData,
                          let timeData = histitem[safeBound: 5...12]?.copyData else {
                        return
                    }
                    let hisContent = histitem[13...].copyData

                    let record_id_Int32: Int32 = recordId.withUnsafeBytes({ $0.bindMemory(to: Int32.self).first! })
                    let timestampInt64: UInt64 = timeData.withUnsafeBytes({ $0.bindMemory(to: UInt64.self).first! })

                    guard var historyType: Sesame2HistoryTypeEnum = Sesame2HistoryTypeEnum(rawValue: type.bytes[0]) else {
                        return
                    }

                    var historyContent = hisContent

                    if historyType == .BLE_LOCK || historyType == .BLE_UNLOCK {
                        let histag = hisContent[18...]
                        let tagcount_historyTag = histag.copyData
                        let tagcount = UInt8(tagcount_historyTag[0])

                        // ロックタイプを解析する
                        let originalTagCount = tagcount % Sesame2HistoryLockOpType.BASE.rawValue
                        let historyOpType = Sesame2HistoryLockOpType(rawValue: tagcount / Sesame2HistoryLockOpType.BASE.rawValue)
                        if historyType == .BLE_LOCK, historyOpType == .WM2 {
                            historyType = Sesame2HistoryTypeEnum.WM2_LOCK
                        } else if historyType == .BLE_LOCK, historyOpType == .WEB {
                            historyType = Sesame2HistoryTypeEnum.WEB_LOCK
                        } else if historyType == .BLE_UNLOCK, historyOpType == .WM2 {
                            historyType = Sesame2HistoryTypeEnum.WM2_UNLOCK
                        } else if historyType == .BLE_UNLOCK, historyOpType == .WEB {
                            historyType = Sesame2HistoryTypeEnum.WEB_UNLOCK
                        }

                        if historyOpType == .WEB || historyOpType == .WM2 {
                            if let type = Sesame2HistoryTypeEnum(rawValue: tagcount / 30) {
                                historyType = type
                            } else {
                                historyType = .NONE
                            }
                        }

                        historyContent = hisContent[...17].copyData + originalTagCount.data + hisContent[19...].copyData
                    }
                    if isInternetReachable == true {
                        self.readHistoryCommand() { _ in }
                    }
                } else {
                }
            }
        }
    }
```

## ESPサンプル

```jsx | pure
void send_read_history_cmd_to_ssm(sesame * ssm) {
    ESP_LOGI(TAG, "[send_read_history_cmd_to_ssm]");
    ssm->c_offset = 2;
    ssm->b_buf[0] = SSM_ITEM_CODE_HISTORY;
    ssm->b_buf[1] = 1;
    talk_to_ssm(ssm, SSM_SEG_PARSING_TYPE_CIPHERTEXT);
}
```
