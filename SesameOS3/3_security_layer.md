# Security layer
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

1. sesame5 使用 secp256r1 產生 publicKeyS 及 privateKeyS

2. APP 使用 secp256r1 產生 publicKeyA 及 privateKeyA

3. 雙方交換 publicKey

4. sesame5 使用 privateKeyS 及 publicKeyA 產生 secret S

5. APP 使用 privateKeyA 及 publicKeyS 產生 secret A

6. secret S 跟 secret A 相等

7. 雙方將 secret 前 16 Bytes 作為 Device secret。

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
1. 登入驗證 (詳見 : `2_登入`)

2. 數據透傳時透過 aes_ccm 加密資料，使用 Token 作為加密時用到的 key。
   
### 產生方式
以 AES_CMAC 演算法，使用 Device secret 對 4Bytes Random code 簽章，就能產生Token。

device_secret : 16 Bytes

random_code : 4 Bytes

Token : 16 Bytes

```c
AES_CMAC(key:device_secret, input:random_code, input_size:4, output:Token);
```

## 4. 資料加密
對於需要加密的資料使用 aes_ccm 對資料進行加密。

開機後 CCM_IV.count 初始化為 0，跟設備連線後更新 random_code，每做一次加密 count 計數加一。
```c
typedef struct {
    int64_t count;///8 byte
    uint8_t nouse;///1 byte = 0
    uint8_t random_code[4];///4 byte
} CCM_IV;
```

### aes_ccm 參數
輸入:
1. key : ccm_key(token)
2. iv : CCM_IV
3. iv length : 13
4. add : 0x00
5. add length : 1
6. input : p_data 要加密的資料
7. input length : length 資料長度
8. tag length : 4
   
輸出:
1. tag : ccm_tag
2. output : ccm_ecrrypt_data 加密後的資料
   
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
1. key : ccm_key(token)
2. iv : CCM_IV
3. iv length : 13
4. add : 0x00
5. add length : 1
6. input : p_data 要解密的資料
7. input length : length 資料長度
8. tag length : 4
9. tag : ccm_tag
   
輸出:
1. output : ans_data 解密後的資料
   
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
資料加解密使用的 CCM_IV 內有個 count 紀錄加密次數，若是加密與解密端的 count 計數不匹配會導致解密失敗。