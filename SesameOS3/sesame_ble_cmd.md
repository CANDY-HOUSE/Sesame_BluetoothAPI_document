[TOC]



手機跟 Sesame 有兩種溝通方式。第一種是 Publish，Sesame 主動將訂閱訊息傳送給手機，如 `8_mechStatus`。第二種是 Response，手機要 Sesame 幹活，Sesame 回覆，如 `82_lock`。

# 3. Application layer 指令對照表


## 3.1、Publish【Sesame->手机】

NotifyValue=True的时候启用，Sesame 主動將訂閱訊息傳送給手機，如 `81_mechStatus`。

| Byte |   0  |      1     |  2 \~ N |
| ---- | :--: | :--------: | :-----: |
| Data | type | item_code | payload |
| 說明   | 推送類型 |    指令編號    | 送給手機的資料 |

```c
#pragma pack(1)
typedef struct {
    uint8_t type;               /// 推送類型 : publish
    uint8_t item_code;          /// 指令(詳見 item_details)
    union ss5_payload payload;  /// 送給手機的資料
} ss5_publish;
#pragma pack()
```

## 3.2、 Request 資料格式

手機要 Sesame 幹活，Sesame 回覆，如 `82_lock`。

| Byte    |      0     |     1 ~ N    |   N+    |
| ----    | :--------: | :-----------: | :-----------: |
| Data    | item_code |    payload    |    others    |
| 說明    | 指令編號| 送給 Sesame 的資料 |其它資料 |
| 注册    |    1    | 1-64【publicKeyA】 | 65-68【timestamp】 |
| 登入    |    2    | 1-4【ccmkeyA】 |
| 历史    |    4    | 1【is peek】 | ispeek=true不删除 |
| 版本    |    5    | none |
| 时间更新|    8    | 1-4【timestamp】 |
| 自动上锁|    11    | 1-2【秒数】 |
| megnet  |    17    | none |
| mechSetting   |    80    | 1-5【autolock+开关锁角度】 |
| mechStatus   |    81    | none |
| lock   |    82    | 1-7【历史标签】 |
| unlock   |    83    | 1-7【历史标签】 |
| ops door open   |    90    | none |
| ops door close   |    91    | none |
| ops timmer   |    92    | none |
|  reset   |    104    | none |
| touch add ssm   |    101    | 1-16【Device Name】 | 17-32【secretKey】 |
| touch remove ssm  |    103    | 1-16【Device Name】 |
| touch card change   |    107    | 0【Card ID Len】1-id_len【CardID】 | CardIDLen+1【CardNameLen】etc | 
| touch card delete   |    108    | 1-16【Card ID】 |
| toch card get   |    113    | none |
| card mode set   |    114    | none |
| finger change   |    115    | 0【FingerIDLen】 |1-Len【FingerID】】 |
| finger delete   |    116    | 1【FingerID】 |
| finger get   |    117    | none |
| finger mode get   |    121    | none |
| finger mode set   |    114    | none |
| pw change   |    123    | 0【PwIDlen】 | 1-pwLen【Password】】 |
| pw delete   |    124    | none |
| pw get   |    125    | none |
| pw mode get   |    121    | none |
| pw mode set    |    130    | 0验证1新增 |


### type 推送類型

```c
typedef enum {
    SSM2_OP_CODE_RESPONSE = 0x07,
    SSM2_OP_CODE_PUBLISH = 0x08,
} ssm2_op_code_e;
```


```c
#pragma pack(1)
typedef struct {
    uint8_t item_code;              /// 指令(詳見 item_details)
    union ss5_payload payload;      /// 送給 Sesame5 的資料
} ss5_request;
#pragma pack()
```

### 3 Response & Publish資料格式

| Byte |   0  |      1     |    2   |   3～N   |  2N   | 3N   |
| ---- | :--: | :--------: | :----: | :-----: |:-----: |:-----: |
| Data | type | item_code |   res  | payload |payload |payload |
| 說明   | 推送類型 |    指令編號    | 命令處裡狀態 | 送給手機的資料1 |送給手機的資料2 |送給手機的資料3 |
| 初始化   | 8 |    14    | 0 | 0-3【random code】 | none  |
| 注册     | 7 |     1    | 0 | 0-6【mechStatus】 | 7-12【mechSetting】】 | 13~76【publicKeyS】】 |
|  登入    | 7 |    2   | 0 | 0-3【timestamp】 |
|  历史   | 7 |    4    | 0 | 0-3【id】】 | 4【type】 | 5-8【timestamp】 |
|  版本   | 7 |    5    | 0 | 0-11【version】 |
| mechSetting   | 8 |    80    | 0 | 0-1【lock】 | 2-3【unlock】】 | 4-5【autolock_second】 |
| mechStatus   | 8 |    81    | 0 | 0-3【random code】 |
| touch pub ssm key   | 8 |    102    | 0 | 0-21【ssm0-name】 |22【ssm0-status】 |23-44【ssm1_name】etc |
| toch card get   | 7 |    109    | 0 | 0-3【CardID】 |
| finger get   | 7 |    117    | 0 | 0-3【FingerID】|
| finger mode get   | 8 |    14    | 0 | 0-3【random code】 |
| pw notify   | 8 |    118    | 0 | 0【PW Type】 | 1【PWIDLen】 |



```c
#pragma pack(1)
typedef struct {
    uint8_t type;                   /// 推送類型 : Response
    uint8_t item_code;              /// 指令(詳見 item_details)
    uint8_t res;                    /// 命令處理狀態 (都回 success)
    union ss5_payload payload;      /// 送給手機的資料
} ss5_response;
#pragma pack()
```


### item_code

各指令互動細節詳見 `item_details` 說明。

