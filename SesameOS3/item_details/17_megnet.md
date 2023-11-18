# Item: megnet

手機發送 sesame5 角度校正指令，校正完成後回覆成功，sesame5 主動發送 mechstatus(機械狀態)

mechStatus 細節詳見 80_mechStatus 說明

## 循序圖

<p align="left" >
  <img src="../src/megnet/megnet.png" alt="" title="">
</p>

## 手機送出資料

| Byte |     0     |
|------|:---------:|
| Data | item code |

item code : SSM2_ITEM_CODE_MAGNET (17)

## ssm5 回傳內容

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_MAGNET (17)

res : CMD_RESULT_SUCCESS (0x00)

## android 範例

``` java
    override fun magnet(result: CHResult<CHEmpty>) {
        if (checkBle(result)) return
        sendCommand(SesameOS3Payload(SesameItemCode.magnet.value, byteArrayOf()), DeviceSegmentType.cipher) { res ->
            result.invoke(Result.success(CHResultState.CHResultStateBLE(CHEmpty())))
        }
    }
```
