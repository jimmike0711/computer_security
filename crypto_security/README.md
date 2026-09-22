# Applied Cryptography & Cryptanalysis: Vulnerability Exploitation
這是一個專注於密碼學演算法缺陷分析與漏洞利用的實作專案，範圍涵蓋對稱式/非對稱式加密破解、雜湊函數碰撞，以及數位憑證偽造等進階攻擊手法。

## 技術與環境棧
- 加密演算法與協定： AES (CBC Mode), RSA, MD5, SHA1/SHA256, X.509 Certificates。
- 分析與開發工具： Python 3 (pyca/cryptography, PyCryptodome, urllib), OpenSSL, fastcoll。
- 攻擊手法 (Cryptanalysis)： Padding Oracle Attack, Length Extension Attack, MD5 Collision, Lenstra's Attack。

## 核心實作項目
### Block Cipher 破解與 Padding Oracle Attack
- 實作 AES-CBC 模式的解密與弱金鑰 (Weak Key) 暴力破解，在 $2^5$ 的極小金鑰空間內還原明文。
- Padding Oracle Attack： 針對具有漏洞的 Web API，利用 AES-CBC 模式解密時的 XOR 特性以及伺服器回傳的 HTTP 狀態碼 (404/500) 作為 Oracle。在完全不知道 AES 密鑰的情況下，透過操縱前一個 Ciphertext Block 逐步推導出正確的 Padding，成功逐字節 (Byte-by-byte) 還原出完整明文。

### Hash Function 缺陷利用與 MD5 碰撞
- Length Extension Attack： 針對基於 Merkle-Damgård 結構的 MD5 演算法進行攻擊。在伺服器錯誤實作 MD5(password || URL) 作為驗證機制的情境下，不需知道原始密碼，即可透過擴展 Hash 的內部狀態 (Internal State) 偽造出具有合法簽章的惡意 API 請求 (例如注入刪除指令)。
- MD5 Collision (程式碼偽造)： 利用 fastcoll 工具與 MD5 的長度擴展特性，刻意構造出兩個 MD5 雜湊值完全相同，但執行行為截然不同的 Python 腳本 (Benign vs. Malicious)。

### 公鑰密碼學與 X.509 憑證偽造
- 利用快速模指數 (Fast Modular Exponentiation) 實作基礎 RSA 解密。
- 憑證碰撞攻擊 (Colliding Certificates)： 針對使用 md5WithRSAEncryption 簽章演算法的憑證機構 (CA)，成功偽造出兩張具有相同 CA 簽章，但擁有不同 RSA 公鑰的合法 X.509 憑證。利用 fastcoll 對 tbsCertificate (to-be-signed) 欄位製造 MD5 碰撞，並結合 Lenstra 的攻擊手法與中國剩餘定理 (Chinese Remainder Theorem, CRT)，計算出兩組獨立且合法的 2047-bit RSA 質數因數。


## 開發挑戰與解決方案
- 密文區塊操縱與 Padding 推導： 在執行 Padding Oracle Attack 時，最大的挑戰在於理解 CBC 模式中密文與明文的 XOR 關係。透過迭代測試 $C1'[15]$ 的值 (0 到 255) 並觀察 Web Application 的 HTTP 404 (Padding 正確但明文錯誤) 回應，成功定位出使解密區塊最後一個位元組變為 0x10 的值，進而逆推出原始明文。
- 憑證欄位對齊與大數運算： 在構造 MD5 憑證碰撞時，必須確保 RSA Modulus 在 ASN.1 結構中的起始位置完美對齊 MD5 的 64-byte 區塊邊界。透過精確調整憑證中 pseudonym 欄位的長度完成對齊，並使用 PyCryptodome 函式庫處理 400-bit 質數的生成與龐大的 CRT 計算，最終成功計算出符合條件的 RSA 私鑰。