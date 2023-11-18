# Item: mech setting (機械設定)

mechSetting 是 Sesame5 的機械設定，包含 autolock 秒數及 sesame5 開鎖及關鎖的角度。

傳送 mechsetting 的狀況又兩種:

- APP 端主動向 Sesame5 要 mechSetting。
- Sesame5主動向手機推送mechSetting。

## 手機與ssm5傳輸mechsetting互動循序圖

APP 端主動向 Sesame5 要 mechSetting。

<p align="left" >
  <img src="../src/mechSetting/APP_to_SSM5.png" alt="" title="">
</p>

Sesame5主動向手機推送mechSetting。

<p align="left" >
  <img src="../src/mechSetting/SSM5_to_APP.png" alt="" title="">
</p>

## APP 請求命令

| Byte |    5 ~ 1    |     0     |
|------|:-----------:|:---------:|
| Data | mechSetting | item code |

item code : SSM2_ITEM_CODE_MECH_SETTING (80)

mechSetting : 機械設定

## ssm5 回應訊息

| Byte |   2    |     1     |  0   |
|------|:------:|:---------:|:----:|
| Data |  res   | item_code | type |
| 說明   | 命令處裡狀態 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_RESPONSE (0x07)

item code : SSM2_ITEM_CODE_MECH_SETTING (80)

res : CMD_RESULT_SUCCESS (0x00)

## ssm5 推送內容

| Byte |  N ~ 2  |     1     |  0   |
|------|:-------:|:---------:|:----:|
| Data | payload | item_code | type |
| 說明   | 送給手機的資料 |   指令編號    | 推送類型 |

type : SSM2_OP_CODE_PUBLISH (0x08)

item code : SSM2_ITEM_CODE_MECH_SETTING (80)

payload : 詳見以下表格

### payload

| Bit  |    5 ~ 0    |
|------|:-----------:|
| Data | mechSetting |

## mechSetting 結構內容

mechSetting 裡存放 autolock 秒數及 sesame5 開鎖及關鎖的角度，以下為 mechSetting 的結構內容

| Byte |      5 ~ 4      | 3 ~ 2  | 1 ~ 0 |
|:----:|:---------------:|:------:|:-----:|
| Data | autolock_second | unlock | lock  |
|  說明  |     自動關鎖時間      |  開鎖角度  | 關鎖角度  |

```c
typedef struct door_lock_unlock_s {
    int16_t lock;
    int16_t unlock;
} door_lock_unlock_t;

typedef struct mech_setting_s {
    door_lock_unlock_t lock_unlock;
    uint16_t auto_lock_second;
} mech_setting_t;
```

## android 範例

```java
    override fun configureLockPosition(lockTarget: Short, unlockTarget: Short, result: CHResult<CHEmpty>) {
        val cmd = SesameOS3Payload(SesameItemCode.mechSetting.value, lockTarget.toReverseBytes() + unlockTarget.toReverseBytes())
        sendCommand(cmd, DeviceSegmentType.cipher) { res ->
            if (res.cmdResultCode == SesameResultCode.success.value) {
                mechSetting?.lockPosition = lockTarget
                mechSetting?.unlockPosition = unlockTarget
                result.invoke(Result.success(CHResultState.CHResultStateBLE(CHEmpty())))
            } else {
                result.invoke(Result.failure(NSError(res.cmdResultCode.toString(), "CBCentralManager", res.cmdResultCode.toInt())))
            }
        }
    }
```

``` java
    override fun onGattSesamePublish(receivePayload: SSM3PublishPayload) {
        super.onGattSesamePublish(receivePayload)
//        L.d("hcia", "[ss5] " + receivePayload.cmdItCode)
        if (receivePayload.cmdItCode == SesameItemCode.mechStatus.value) {
            mechStatus = CHSesame5MechStatus(receivePayload.payload)
            deviceStatus = if (mechStatus!!.isInLockRange) CHDeviceStatus.Locked else CHDeviceStatus.Unlocked
            readHistoryCommand()
        }
        if (receivePayload.cmdItCode == SesameItemCode.mechSetting.value) {
            mechSetting = CHSesame5MechSettings(receivePayload.payload)
        }
    }
```

```java
class CHSesame5MechSettings(data: ByteArray) {
    var lockPosition: Short = bytesToShort(data[0], data[1])
    var unlockPosition: Short = bytesToShort(data[2], data[3])
    var autoLockSecond: Short = bytesToShort(data[4], data[5])
}
```
