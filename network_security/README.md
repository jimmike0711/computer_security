# Network Security: Protocol Exploitation & Man-in-the-Middle Attacks
這是一個專注於網路協定安全與底層封包操作的實作專案，範圍涵蓋網路探勘 (Reconnaissance)、中間人攻擊 (Man-in-the-Middle) 以及進階的 TCP 連線偽造 (TCP Connection Spoofing) 手法。

## 技術與環境棧
- 網路協定與架構： TCP/IP, ARP, DNS, HTTP, RSH。
- 開發與分析工具： Python (Scapy), Wireshark, tcpdump, Nmap, curl, dig。
- 攻擊手法： ARP Spoofing, TCP Session Hijacking / Injection, TCP ISN Prediction (Mitnick Attack), Port Scanning。

## 核心實作項目
### 網路探勘與客製化掃描器 (Network Reconnaissance)
- 利用 Nmap 進行網路拓樸分析，並完全使用 Python 的 Scapy 函式庫從零實作一個 TCP SYN Scanner。
- 透過收發 SYN/ACK 與 RST 封包，精準探測目標主機的開放埠口，並確保掃描過程後釋放目標機器的連線資源。

### ARP 欺騙與封包監聽 (Passive MitM)
- 實作 ARP Spoofing，將攻擊者主機安插於受害者與 DNS Server、Web Server 的通訊路徑中。
- 在不中斷原有網路連線的隱蔽前提下，即時解析並攔截明文傳輸的 DNS 查詢紀錄、HTTP Session Cookie 以及 Base64 編碼的 HTTP Basic Auth 認證資訊。

### TCP 封包動態注入 (Active MitM Script Injection)
- 延伸 ARP Spoofing 的場景，針對未加密的 HTTP 流量進行動態攔截與修改。
- 在 Web Server 回傳的 HTML 封包抵達受害者前，精準定位並於 </ body> 標籤前動態注入自訂的 JavaScript Payload (< script>SRC</ script>)。

### TCP 連線偽造 (Kevin Mitnick Attack)
- 針對具有可預測初始序號 (Predictable ISN) 漏洞的早期 TCP/IP 實作 (如 NetBSD 1.0) 進行 Off-Path 攻擊。
- 在無法接收到 SYN-ACK 封包的「盲打」狀態下，成功偽造受信任 IP 的 TCP 三方交握，並利用 RSH 協定注入指令，將攻擊者 IP 加入系統的最高權限信任名單 (/root/.rhosts)。


## 開發挑戰與解決方案
- TCP 狀態機的同步與修復 (State Synchronization)： 在進行 HTTP Script Injection 時，最大的技術挑戰在於注入 Payload 會改變原有 TCP 資料流的長度。為了解決這個問題，必須在攔截封包的當下，接管並動態修正後續所有雙向封包的 Sequence Number 與 Acknowledgment Number (同時處理模 $2^{32}$ 的溢位運算)。這確保了連線雙方不會因為接收到非預期的序號而觸發 TCP RST 中斷連線，成功維持了連線的穩定性與隱蔽性。 
- 無回饋狀態下的連線偽造 (Blind TCP Spoofing)： 在實作 Mitnick 攻擊時，由於攻擊者不在正確的路由路徑上，無法看見目標主機回應的 SYN-ACK 封包。解決方案為先向目標發送多個合法探測封包以歸納其 ISN 的遞增規律，進而精準預測下一次的 ISN 以完成盲目的三方交握。同時，針對舊版 TCP Stack 無法處理突發流量的特性，在發送偽造的 RSH 封包時實作了微小的傳輸延遲 (Pacing) 以確保攻擊成功率。
- Injection 超過 payload 乘載上限 : 利用 Scapy 提供的 IP 碎片化來解決。