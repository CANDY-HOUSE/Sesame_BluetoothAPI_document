# Item: Time

手機主動送出更新時間(item_code:0x08)指令及手機 timestamp，ssm5更新時間戳並回傳更新成功。

## 循序圖
<p align="left" >
  <img src="../src/time/time循序圖.png" alt="" title="">
</p>

## 手機送出資料
| Byte | 4 ~ 1     | 0         |
|------|:---------:|:---------:|
| Data | timestamp | item code |

item code : SSM2_ITEM_CODE_TIME (0x08)

timestamp : 手機的 timestamp

## ssm5 回傳內容
| Byte | 2      | 1         | 0    |
|-------|:-----:|:---------:|:----:|
| Data  | res    | item_code | type |
| 說明   | 命令處裡狀態 | 指令編號      | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_TIME (0x08)

res : CMD_RESULT_SUCCESS (0x00)

## android範例
``` java
    override fun updateTime(result: CHResult<CHEmpty>) {
        if (checkBle(result)) return
        sendCommand(SesameOS3Payload(SesameItemCode.time.value, System.currentTimeMillis().toUInt32ByteArray()), DeviceSegmentType.cipher) { res ->
            result.invoke(Result.success(CHResultState.CHResultStateBLE(CHEmpty())))
        }
    }
```
