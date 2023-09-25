# Item: Version detail

手機主動送出Version detail(item_code:0x05)指令，ssm5回傳ssm5的版本號。

## 循序圖
<p align="left" >
  <img src="../src/version/version_循序圖.png" alt="" title="">
</p>

## 手機送出資料
| Byte | 1         | 0       |
|------|:---------:|:-------:|
| Data | item code | op code |

item code : SSM2_ITEM_CODE_VERSION_DETAIL (0x05)

## ssm5 回傳內容
| Byte | 15 ~ 4  | 3   | 2  | 1    | 0  |
|------|:-------:|:---:|:--:|:----:|:--:|
| Data | payload | res | item code | type |op code|

type : SSM2_OP_CODE_RESPONSE(0x07)

item code : SSM2_ITEM_CODE_VERSION_DETAIL (0x05)

res : CMD_RESULT_SUCCESS (0x00)

## android 範例
``` java
    override fun getVersionTag(result: CHResult<String>) {
        if (checkBle(result)) return
        sendEncryptCommand(SSM2Payload(SSM2OpCode.read, SesameItemCode.versionTag, byteArrayOf())) { res ->
            val gitTag = res.payload.sliceArray(4..15)
            CHAccountManager.putSesameInfor(this, String(gitTag)) {}
            result.invoke(Result.success(CHResultState.CHResultStateBLE(String(gitTag))))
        }
    }
```
