# Application Security: Control-Flow Hijacking & Exploitation
這是一個在 32-bit x86 Ubuntu 虛擬機環境下進行的系統安全實作專案，專注於分析與利用 C 語言應用程式中的底層記憶體漏洞。


## 技術與環境棧
- 系統架構： 32-bit x86 (i386)、Linux 系統呼叫 (System Calls, 如 execve)。
- 分析與開發工具： GNU Debugger (gdb)、x86 Assembly (AT&T 語法)、Python (處理 Little-endian 轉換與 Payload 構造)。
- 攻擊手法 : Buffer Overflow, Integer Confusion, Bypassing DEP, Format String Attack, Returned-Oriented Programming

## 核心實作項目
### Buffer Overflow & Shellcode Injection
- 透過分析 C 語言程式的 Stack Frame 與暫存器 (ebp, esp) 狀態，精準計算 Return Address 的偏移量，並注入 Shellcode 以取得系統 Shell。
- 針對具有動態記憶體偏移 (每次執行偏移 0xC-0x10C bytes) 的目標程式，利用 NOP Sled 技術提高 Exploit 的穩定性與成功率。

### 記憶體防護繞過 (Bypassing DEP & ROP)
- 在目標程式開啟 DEP 防護（禁止 Stack 執行代碼）的情況下，不依賴 pwntools 或 ROPgadget 等自動化測試工具，純透過 gdb 與 objdump 分析現有程式碼片段。
- 成功建構 Return-Oriented Programming (ROP) 攻擊鏈，透過串接合適的 Gadgets 執行任意指令。

### 進階記憶體漏洞利用
- Format String Attack： 利用 printf() 的 %n 轉換指定字 (Conversion specifier) 與直接參數存取 (Direct Parameter Access) 特性，達成精準的任意記憶體位置寫入。
- Integer Confusion： 利用 C 語言 size_t 型別 (無號整數) 在 32-bit 系統上超過 2^32 - 1 會發生溢位的特性，成功繞過 alloca 的緩衝區大小安全檢查。


## 開發挑戰與解決方案
- Null Byte 截斷問題： 在建構 Shellcode Payload 時，遇到 Null character (\0) 導致字串被提早截斷的問題。透過調整 Filler bytes 或改變 Shellcode 在 Buffer 中的放置位置來改變跳轉位址，成功繞過限制。
- 純手動建構 ROP Chain： 由於限制無法使用自動化 Exploit 工具，必須手動從 Assembly 中尋找合適的 Gadgets。透過靈活運用暫存器運算（例如利用 XOR 將暫存器清零）來拼湊出完整的系統呼叫參數，加深了對 x86 指令集的理解。
- 嚴謹的數學計算 : 算錯或少算任何一點，就失敗，因此在攻擊前須詳細且完整的推演過。