# Item: mech status (機械狀態)

mechStatus 裡存放sesame5各個硬體的狀態。

## mechStatus 的結構內容:
```c
#pragma pack(1)
typedef struct mech_status_s {
    uint16_t battery;
    int16_t target;                     // 馬達想到的地方
    int16_t position;                   // 感測器同步到的最新角度
    uint8_t is_clutch_failed: 1;        // 電磁鐵作棟是否成功(沒用到)
    uint8_t is_lock_range: 1;           // 在關鎖位置
    uint8_t is_unlock_range: 1;         // 在開鎖位置
    uint8_t is_critical: 1;             // 開關鎖時間超時，馬達停轉
    uint8_t is_stop: 1;                 // 把手角度沒有變化
    uint8_t is_low_battery: 1;          // 低電量(<5V)
    uint8_t is_clockwise: 1;            // 馬達轉動方向
} mech_status_t;
#pragma pack()
```
為了防止客戶端頻繁跟 sesame5 請求機械狀態，只能由 sesame5 在機械狀態改變時主動發送。



## 手機與 ssm5 傳輸 mechStatus 互動循序圖

<p align="left" >
  <img src="../src/mechStatus/SSM5_to_APP.png" alt="" title="">
</p>

## ssm5 推送內容
| Byte | N ~ 2   | 1         | 0    |
|-------|:------:|:---------:|:----:|
| Data  | payload | item_code | type |
| 說明    | 送給手機的資料 | 指令編號      | 推送類型 |

type : SSM2_OP_CODE_PUBLISH (0x08)

item code : SSM2_ITEM_CODE_INITIAL (81)

payload : 詳見以下表格

### payload
| Byte | 6 ~ 0               |
|------|:-------------------:|
| Data | mechStatus          |

## mech status
|Byte|6|5 ~ 4|3 ~ 2|1 ~ 0|
|:---:|:---:|:---:|:---:|:---:|
|Data|other data|position|target|battery|

## other data
|Bit|7|6|5|4|3|2|1|0|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Data|NULL|is_clockwise|is_low_battery|is_stop|is_critical|is_unlock_range|is_lock_range|is_clutch_failed|


## android 範例
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
``` java
class CHSesame5MechStatus(override val data: ByteArray) : CHSesameProtocolMechStatus {
    private val battery = bytesToShort(data[0], data[1])
    override val position: Short = bytesToShort(data[4], data[5])
    override val target: Short? = if ((bytesToShort(data[2], data[3]).toInt() == -32768)) null else bytesToShort(data[2], data[3])
    private val flags = data[6].toInt()
    override var isInLockRange: Boolean = flags and 2 > 0
    override var isBatteryCritical: Boolean = flags and 32 > 0
    override var isStop: Boolean? = flags and 16 > 0
    override fun getBatteryVoltage(): Float {
//        L.d("hcia", "[ss5][volt]" + battery * 2f / 1000f)
        return battery * 2f / 1000f
    }
}
```
