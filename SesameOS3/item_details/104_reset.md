# Item: reset

手機發送重置指令，sesame5 回覆指令成功，並清除資料和重新啟動。

## 循序圖

<p align="left" >
  <img src="../src/reset/reset.png" alt="" title="">
</p>

## 手機送出資料

| Byte |     0     |
|------|:---------:|
| Data | item code |

item code : SSM3_ITEM_RESET (104)

## ssm5 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM3_ITEM_RESET (104)

res : CMD_RESULT_SUCCESS (0x00)

## android 範例

``` java
    open fun reset(result: CHResult<CHEmpty>) {
        sendCommand(SesameOS3Payload(SesameItemCode.Reset.value, byteArrayOf()), DeviceSegmentType.cipher) { res ->
            if (res.cmdResultCode == SesameResultCode.success.value) {
                dropKey(result)
            } else {
                result.invoke(Result.failure(NSError(res.cmdResultCode.toString(), "CBCentralManager", res.cmdResultCode.toInt())))
            }
        }
    }
```
