# Item: unlock

手機發送 unlock 指令及歷史標籤，手機回傳指令成功，建立開鎖歷史並開鎖。

## 循序圖

<p align="left" >
  <img src="../src/unlock/unlock.png" alt="" title="">
</p>

## 手機送出資料

| Byte | 7 ~ 1 |     0     |
|------|:-----:|:---------:|
| Data | 歷史標籤  | item code |

item code : SSM2_ITEM_CODE_UNLOCK (83)

歷史標籤 : 顯示在 sesame5 歷史列表的標籤

## ssm5 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_UNLOCK (83)

res : CMD_RESULT_SUCCESS (0x00)

## android 範例

``` java
    override fun unlock(historytag: ByteArray?, result: CHResult<CHEmpty>) {
        if (deviceStatus.value == CHDeviceLoginStatus.UnLogin && isConnectedByWM2) {
            CHAccountManager.cmdSesame(SesameItemCode.unlock, this, sesame2KeyData!!.hisTagC(historytag), result)
        } else {
            if (checkBle(result)) return
//        L.d("hcia", "[ss5][unlock] historyTag:" + sesame2KeyData!!.createHistagV2(historyTag).toHexString())
            sendCommand(SesameOS3Payload(SesameItemCode.unlock.value, sesame2KeyData!!.createHistagV2(historytag)), DeviceSegmentType.cipher) { res ->
                if (res.cmdResultCode == SesameResultCode.success.value) {
                    result.invoke(Result.success(CHResultState.CHResultStateBLE(CHEmpty())))
                } else {
                    result.invoke(Result.failure(NSError(res.cmdResultCode.toString(), "CBCentralManager", res.cmdResultCode.toInt())))
                }
            }
        }
    }
```
