[TOC]

# 1、蓝牙广播BLE Advertising

以下為 Sesame5 廣播中的欄位及各欄位中每個位元所代表的含意。

## CANDY HOUSE, Inc.

| CANDY HOUSE, Inc. | Decimal |   Hex  |
| :---------------: | :-----: | :----: |
|    Service UUID   |  64897  | 0xFD81 |
|     Company ID    |   1370  | 0x055A |

## Service UUID (0x03)

Sesame5 服務 UUID : 0xFD81

## Manufacturer Data (0xFF)

| Byte | 0 \~ 1 |   2  |  3  |   4  | 5 \~ 20 |
| ---- | :----: | :--: | :-: | :--: | :-----: |
| Data |  公司代碼  | 產品編號 |  保留 | 是否註冊 |  IC 序號  |

公司代碼 : 0x055A

| 產品編號 |        機型        |
| :--: | :--------------: |
|   0  |     Sesame 3     |
|   1  |   WiFi Module 2  |
|   2  |   Sesame Bot 1   |
|   3  |   Sesame Bike 1  |
|   4  |     Sesame 4     |
|   5  |     Sesame 5     |
|   6  |   Sesame Bike 2  |
|   7  |    Sesame Pro    |
|   8  |    Open Sensor   |
|   9  | Sesame Touch Pro |
|  10  |   Sesame Touch   |

# 2、Application layer

手機跟 Sesame 有兩種溝通方式。第一種是 Publish，Sesame 主動將訂閱訊息傳送給手機，如 `81_mechStatus`。第二種是 Response，手機要 Sesame 幹活，Sesame 回覆，如 `82_lock`。

## 1. Publish【Sesame->手机】

NotifyValue=True的时候启用，Sesame 主動將訂閱訊息傳送給手機，如 `81_mechStatus`。

| Byte |   0  |      1     |  2 \~ N |
| ---- | :--: | :--------: | :-----: |
| Data | type | item\_code | payload |
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

## 2. Request

手機要 Sesame 幹活，Sesame 回覆，如 `82_lock`。

| Byte |      0     |     1 \~ N    |
| ---- | :--------: | :-----------: |
| Data | item\_code |    payload    |
| 說明   |    指令編號    | 送給 Sesame 的資料 |

```c
#pragma pack(1)
typedef struct {
    uint8_t item_code;              /// 指令(詳見 item_details)
    union ss5_payload payload;      /// 送給 Sesame5 的資料
} ss5_request;
#pragma pack()
```

### 3 Response 資料格式

| Byte |   0  |      1     |    2   |   3～N   |
| ---- | :--: | :--------: | :----: | :-----: |
| Data | type | item_code |   res  | payload |
| 說明   | 推送類型 |    指令編號    | 命令處裡狀態 | 送給手機的資料 |

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

# 3. Application layer 指令對照表

### type 推送類型

```c
typedef enum {
    SSM2_OP_CODE_RESPONSE = 0x07,
    SSM2_OP_CODE_PUBLISH = 0x08,
} ssm2_op_code_e;
```

### item\_code

各指令互動細節詳見 `item_details` 說明。

```c
typedef enum {
    SSM2_ITEM_CODE_NONE = 0,
    SSM2_ITEM_CODE_REGISTRATION = 1,
    SSM2_ITEM_CODE_LOGIN = 2,
    SSM2_ITEM_CODE_HISTORY = 4,
    SSM2_ITEM_CODE_VERSION_DETAIL = 5,
    SSM2_ITEM_CODE_TIME = 8,
    SSM2_ITEM_CODE_AUTOLOCK = 11,
    SSM2_ITEM_CODE_INITIAL = 14,
    SSM2_ITEM_CODE_IRER = 15,
    SSM2_ITEM_CODE_MAGNET = 17,

    SSM2_ITEM_CODE_MECH_SETTING = 80,
    SSM2_ITEM_CODE_MECH_STATUS = 81,
    SSM2_ITEM_CODE_LOCK = 82,
    SSM2_ITEM_CODE_UNLOCK = 83,

    SSM3_ITEM_RESET = 104
} ssm2_item_code_e;
```

### res 命令處理狀態

```c
typedef enum {
    CMD_RESULT_SUCCESS = 0,         /// 手機下命令 Sesame5 會直接回 success
    CMD_RESULT_INVALID_ACTION = 9,  /// 已經註冊還下註冊命令 
} cmd_result_e;
```

# 4、Security layer

安全層是 Sesame5 及手機間資料透傳實作的加密傳輸協定。

## 1. 4Bytes Random code

搜尋到 Sesame5 並連線開啟 notify，這時 Sesame5 主動發送 4Bytes Random Code。
(詳見 `14_inital`)

## 2. Device secret

註冊時，APP 及 Sesame5 雙方都會透過 secp256r1 產生 public key 及 private key，交換 public key 後使用自己的 private key 及對方的 public key 生成 secret，雙方會產生一樣的 secret，將 secret 前 16 Bytes 作為 Device secret。

publicKey : 64 Bytes

privateKey : 32 Bytes

secret : 32 Bytes

### 步驟 :

1.  sesame5 使用 secp256r1 產生 publicKeyS 及 privateKeyS

2.  APP 使用 secp256r1 產生 publicKeyA 及 privateKeyA

3.  雙方交換 publicKey

4.  sesame5 使用 privateKeyS 及 publicKeyA 產生 secret S

5.  APP 使用 privateKeyA 及 publicKeyS 產生 secret A

6.  secret S 跟 secret A 相等

7.  雙方將 secret 前 16 Bytes 作為 Device secret。

### C 範例程式

```c
    /// 雙方各自產生自己的 public key 及 private key
    uint8_t ecc_private_alice[32], ecc_public_alice[64];
    uint8_t ecc_private_bob[32], ecc_public_bob[64];
    uECC_make_key_lit(ecc_public_alice, ecc_private_alice, uECC_secp256r1());
    uECC_make_key_lit(ecc_public_bob, ecc_private_bob, uECC_secp256r1());

    log_info_array_ex("ecc_public_alice ", ecc_public_alice, sizeof(ecc_public_alice))
    log_info_array_ex("ecc_private_alice ", ecc_private_alice, sizeof(ecc_private_alice))

    log_info_array_ex("ecc_public_bob ", ecc_public_bob, sizeof(ecc_public_bob))
    log_info_array_ex("ecc_private_bob ", ecc_private_bob, sizeof(ecc_private_bob))

    /// 交換 public key 使用自己的 private key 及對方的 public key，雙方產生共同的 secret
    uint8_t ecdh_secret_alice[32];
    uint8_t ecdh_secret_bob[32];
    uECC_shared_secret_lit(ecc_public_bob, ecc_private_alice, ecdh_secret_alice, uECC_secp256r1());
    uECC_shared_secret_lit(ecc_public_alice, ecc_private_bob, ecdh_secret_bob, uECC_secp256r1());

    log_info_array_ex("ecdh_secret_alice ", ecdh_secret_alice, sizeof(ecdh_secret_alice))
    log_info_array_ex("ecdh_secret_bob ", ecdh_secret_bob, sizeof(ecdh_secret_bob))
```

## 3. Token

### 用途

1.  登入驗證 (詳見 : `2_登入`)

2.  數據透傳時透過 aes\_ccm 加密資料，使用 Token 作為加密時用到的 key。

### 產生方式

以 AES\_CMAC 演算法，使用 Device secret 對 4Bytes Random code 簽章，就能產生Token。

device\_secret : 16 Bytes

random\_code : 4 Bytes

Token : 16 Bytes

```c
AES_CMAC(key:device_secret, input:random_code, input_size:4, output:Token);
```

## 4. 資料加密

對於需要加密的資料使用 aes\_ccm 對資料進行加密。

開機後 CCM\_IV.count 初始化為 0，跟設備連線後更新 random\_code，每做一次加密 count 計數加一。

```c
typedef struct {
    int64_t count;///8 byte
    uint8_t nouse;///1 byte = 0
    uint8_t random_code[4];///4 byte
} CCM_IV;
```

### aes\_ccm 參數

輸入:

1.  key : ccm\_key(token)
2.  iv : CCM\_IV
3.  iv length : 13
4.  add : 0x00
5.  add length : 1
6.  input : p\_data 要加密的資料
7.  input length : length 資料長度
8.  tag length : 4

輸出:

1.  tag : ccm\_tag
2.  output : ccm\_ecrrypt\_data 加密後的資料

### C 加密範例

```c
aes_ccm_encrypt_and_tag(
                        key:ccm_key,
                        iv:CCM_IV, iv_length:13, 
                        add:0x00, add_length:1, 
                        input:p_data, add_length:length, 
                        output:ccm_ecrrypt_data, 
                        tag:ccm_tag, tag_length:4
                        );
CCM_IV.count ++;
```

## 5. 資料解密

輸入:

1.  key : ccm\_key(token)
2.  iv : CCM\_IV
3.  iv length : 13
4.  add : 0x00
5.  add length : 1
6.  input : p\_data 要解密的資料
7.  input length : length 資料長度
8.  tag length : 4
9.  tag : ccm\_tag

輸出:

1.  output : ans\_data 解密後的資料

### C 解密範例

```c
aes_ccm_auth_decrypt(
                    key:ccm_key,
                    iv:CCM_IV, iv_length:13, 
                    add:0x00, add_length:1, 
                    input:ccm_ecrrypt_data, add_length:length, 
                    output:ans_data, 
                    tag:ccm_tag, tag_length:4
                    );
CCM_IV.count ++;
```

## 注意 !!!

資料加解密使用的 CCM\_IV 內有個 count 紀錄加密次數，若是加密與解密端的 count 計數不匹配會導致解密失敗。

# 5、Segment layer

Segment layer 要做兩件事情。

1.  Sesame5 與 手機通訊時，限制封包最大長度 20Bytes，送長訊息需要對資料進行拆解，傳到接收端再組合。
2.  標記傳送的資料是否有加密

## 對照表

Segment layer 在封包頭中的標記此封包是否被切割或加密，接收端可以按照標記做相應的處理，將封包還原成原始資料。以下為 Segment layer 標記對照表。

Segment layer 標記有 8bits，其中 bit 7 \~ 1 表示是否為結束封包及資料是否加密，bit 0 表示該封包是否為一筆資料的起始封包。

| bit 7 \~ 1 | 結束   | bit 0 | 開始  |
| ---------- | ---- | ----- | --- |
| b0000000   | 沒結束  | 0     | 非開始 |
| b0000000   | 沒結束  | 1     | 開始  |
| b0000001   | 明文結束 | 0     | 非開始 |
| b0000001   | 明文結束 | 1     | 開始  |
| b0000010   | 加密結束 | 0     | 非開始 |
| b0000010   | 加密結束 | 1     | 開始  |

## 標記以 16 進位解讀

0 表示該封包不是開始封包也不是結束封包

1 表示該封包是開始封包不是結束封包

2 表示該封包不是開始封包是結束封包，並且組合起來的資料是明文

3 表示該封包是開始封包也是結束封包，並且資料是明文，資料沒有被切割，不需要組合

4 表示該封包不是開始封包是結束封包，並且組合起來的資料是密文

5 表示該封包是開始封包也是結束封包，並且資料是密文，資料沒有被切割，不需要組合

| 標記   | 開始  | 結束   |
| ---- | --- | ---- |
| 0x00 | 非開始 | 沒結束  |
| 0x01 | 開始  | 沒結束  |
| 0x02 | 非開始 | 明文結束 |
| 0x03 | 開始  | 明文結束 |
| 0x04 | 非開始 | 加密結束 |
| 0x05 | 開始  | 加密結束 |

## 範例

假設一個封包能放 4bytes 的資料，超過4bytes就需要進行分割(實際為 20bytes)。

### 短明文封包

資料: A A B B

<p align="left">
  <img src alt title>
</p>

發送:

    1. 收到資料 : A A B B
    2. 資料 : A A B B
    3. 加上封包頭 : 3 A A B B

接收:

    4. 收到 : 3 A A B B
    5. 去除封包頭還原資料 : A A B B

### 中明文封包

資料: A A A A B B B B

<p align="left">
  <img src alt title>
</p>

發送:

    1. 收到資料 : A A A A B B B B
    2. 拆解資料 : A A A A 
    3. 加上封包頭 : 1 A A A A
    6. 拆解資料 : B B B B
    7. 加上封包頭 : 2 B B B B

接收:

    4. 收到 : 1 A A A A
    5. 判斷資料還沒送完，等下一個封包
    8. 收到 : 2 B B B B
    9. 資料接收完成，組合封包
    10. 去除封包頭還原資料 : A A A A B B B B

### 長明文封包

資料 : A A A A B B B B C C C C

<p align="left">
  <img src alt title>
</p>

發送:

    1. 收到資料 : A A A A B B B B C C C C
    2. 拆解資料 : A A A A 
    3. 加上封包頭 : 1 A A A A
    6. 拆解資料 : B B B B
    7. 加上封包頭 : 0 B B B B
    10. 拆解資料 : C C C C
    11. 加上封包頭 : 2 C C C C

接收:

    4. 收到 : 1 A A A A
    5. 判斷資料還沒送完，等下一個封包
    8. 收到 : 0 B B B B
    9. 判斷資料還沒送完，等下一個封包
    12. 收到 : 2 C C C C
    13. 資料接收完成，組合封包
    14. 去除封包頭還原資料 : A A A A B B B B C C C C

### 短密文封包

資料 : A A B B

資料加密後 : X X Y Y

<p align="left">
  <img src alt title>
</p>

發送:

    1. 收到資料 : A A B B
    2. 資料加密後 : X X Y Y
    3. 加上封包頭 : 5 X X Y Y

接收:

    4. 收到 : 5 X X Y Y
    5. 去除封包頭還原資料 : X X Y Y
    6. 解密 : A A B B

### 中密文封包

資料: A A A A B B B B

資料加密後 : X X X X Y Y Y Y

發送:

    1. 收到資料 : A A A A B B B B
    2. 資料加密後 : X X X X Y Y Y Y
    3. 拆解資料 : X X X X 
    4. 加上封包頭 : 1 X X X X
    7. 拆解資料 : Y Y Y Y
    8. 加上封包頭 : 4 Y Y Y Y

接收:

    5. 收到 : 1 X X X X
    6. 判斷資料還沒送完，等下一個封包
    9. 收到 : 4 Y Y Y Y
    10. 資料接收完成，組合封包
    11. 去除封包頭還原資料 : X X X X Y Y Y Y
    12. 資料解密 : A A A A B B B B

### 長密文封包

資料 : A A A A B B B B C C C C

資料加密後 : X X X X Y Y Y Y Z Z Z Z

<p align="left">
  <img src alt title>
</p>

發送:

    1. 收到資料 : A A A A B B B B C C C C
    2. 資料加密後 : X X X X Y Y Y Y Z Z Z Z
    3. 拆解資料 : X X X X 
    4. 加上封包頭 : 1 X X X X
    7. 拆解資料 : Y Y Y Y
    8. 加上封包頭 : 0 Y Y Y Y
    11. 拆解資料 : Z Z Z Z
    12. 加上封包頭 : 4 Z Z Z Z

接收:

    5. 收到 : 1 X X X X
    6. 判斷資料還沒送完，等下一個封包
    9. 收到 : 0 Y Y Y Y
    10. 判斷資料還沒送完，等下一個封包
    13. 收到 : 4 Z Z Z Z
    14. 資料接收完成，組合封包
    15. 去除封包頭還原資料 : X X X X Y Y Y Y Z Z Z Z
    16. 資料解密 : A A A A B B B B C C C C

## 注意 !!!

資料經過 AES\_CCM 加密後除了密文外還會產生 4Bytes 的 ccm\_tag 用於解密，密文及 ccm\_tag 需要一起傳給接收端，因此加密後的資料會增加 4Bytes，詳見 `security_layer` 說明。
