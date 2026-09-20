# RGB 像素混色密碼產生器 (RGB Pixel Blending Cipher Tool)

![License](https://img.shields.io/badge/License-MIT-emerald.svg)
![Type](https://img.shields.io/badge/Algorithm-Pixel--CBC%20Mode-cyan.svg)
![Frontend](https://img.shields.io/badge/Frontend-HTML5%20%2B%20TailwindCSS%20%2B%20JS-blue.svg)

一套結合**現代對稱式分組密碼學（Block Cipher）**、**CBC 像素反饋連鎖擴散機制**與**可視化資訊隱藏（Visual Steganography）**概念的前端密碼產生器與審計分析工具。

🌐 **線上即時體驗（GitHub Pages）**：[https://ericlin241.github.io/rgb-pixel-blending-cipher/](https://ericlin241.github.io/rgb-pixel-blending-cipher/)

---

## 🌟 核心特色

0. **支援深淺色主題切換（Dark / Light Mode）**
   - 支援系統偏好預設偵測與 `localStorage` 狀態持久化。
   - 頂部導航列一鍵切換日光淺色 / 極客深色模式，色塊與畫布自動適配對比度。
1. **Pixel-CBC 密碼連鎖混色演算法**
   - **字元與數值映射**：英文字母 `A~Z` 精確映射至數值 `0~25`。
   - **通道置換與非線性混淆**：將明文 3 字母映射為 $(R, G, B)$ 通道，並在加密時進行輪替置換 ($G \to R'$, $B \to G'$, 補數 $R \to B'$)。
   - **連鎖反饋機制（Cipher Block Chaining）**：自第 2 組起，將前一區塊的密文值作為偏移反饋向量，達成強大的雪崩效應（Avalanche Effect）與色彩擴散。
2. **視覺化色彩長條（Pixel Palette Canvas & Swatches）**
   - **明文與加密雙調色盤並列**：直觀展示混色前後色彩的劇烈變化。
   - **色彩數值線性映射**：區塊數值轉換為螢幕色彩 $\text{RGB}(\text{值} \times 10, \text{值} \times 10, \text{值} \times 10)$。
   - **區塊色票互動**：支援點擊複製 HEX 色碼、RGB 數值分解。
   - **PNG 色條繪圖匯出**：使用 HTML5 Canvas API 一鍵下載像素條圖片。
3. **完整的運算步驟清單（Step-by-step Audit Table）**
   - 即時揭露每一區塊的輸入、通道值、CBC 前饋偏移向量、代數展開公式、模 26 運算過程與輸出字母與色彩。
4. **雙模式與錯誤防護**
   - **加密模式**：即時過濾非英文字元，非 3 的倍數時自動使用字元 `'X'` 補齊（Padding）。
   - **解密模式**：嚴格驗證密文需為 3 的倍數，代數逆運算還原明文，並智慧提示末端填充字元 `'X'`。
   - **一鍵快速互轉**：支援將加密密文一鍵轉換為解密輸入進行往返驗證（Round-trip verification）。

---

## 📐 密碼演算法數學規格

### 1. 初始參數與金鑰
- **字母映射**：$A=0, B=1, \dots, Z=25$
- **固定濾鏡金鑰**：$K_R = 3, K_G = 7, K_B = 12$
- **長文本分組與填充**：每 3 字母一組 $(R_n, G_n, B_n)$。若長度非 3 的倍數，使用 `'X'` 補齊。

### 2. 像素連鎖加密模式 (Pixel-CBC Encryption)
- **第 1 組區塊 ($n = 1$)**：
  $$\begin{aligned}
  R'_1 &= (G_1 + K_R) \pmod{26} \\
  G'_1 &= (B_1 + K_G) \pmod{26} \\
  B'_1 &= (25 - R_1 + K_B) \pmod{26}
  \end{aligned}$$
- **第 2 組及後續區塊 ($n > 1$)**，引入前一組密文 $(R'_{n-1}, G'_{n-1}, B'_{n-1})$：
  $$\begin{aligned}
  R'_n &= (G_n + K_R + R'_{n-1}) \pmod{26} \\
  G'_n &= (B_n + K_G + G'_{n-1}) \pmod{26} \\
  B'_n &= (25 - R_n + K_B + B'_{n-1}) \pmod{26}
  \end{aligned}$$

### 3. 逆向連鎖解密模式 (Pixel-CBC Decryption)
- 對於第 $n$ 組密文 $(R'_n, G'_n, B'_n)$：
  - 若 $n = 1$：前饋偏移向量 $\text{offset} = (0, 0, 0)$
  - 若 $n > 1$：前饋偏移向量 $\text{offset} = (R'_{n-1}, G'_{n-1}, B'_{n-1})$
  $$\begin{aligned}
  R_n &= (25 - B'_n + K_B + \text{offset}_B) \pmod{26} \\
  G_n &= (R'_n - K_R - \text{offset}_R + 52) \pmod{26} \\
  B_n &= (G'_n - K_G - \text{offset}_G + 52) \pmod{26}
  \end{aligned}$$

---

## 🧪 預設測試範例

| 範例名稱 | 明文輸入 (Plaintext) | 填充後字串 (Padded) | 加密輸出 (Ciphertext) | 解密結果 (Decrypted) |
|:---:|:---:|:---:|:---:|:---:|
| **範例 1** (短單字) | `SPY` | `SPY` | `SFT` | `SPY` |
| **範例 2** (標準 6 字) | `ATTACK` | `ATTACK` | `WALBRW` | `ATTACK` |
| **範例 3** (需填充 8 字) | `SECURITY` | `SECURITYX` | `HJTBYKCCC` | `SECURITYX` (提示含 X) |

---

## 🚀 快速開始

本專案採用純前端技術架構（Pure Static Web），無須配置 Node.js 或建置編譯工具：

1. **Clone 專案**：
   ```bash
   git clone https://github.com/ericlin241/rgb-pixel-blending-cipher.git
   cd rgb-pixel-blending-cipher
   ```

2. **直接執行**：
   - 雙擊 `index.html` 即可在任一瀏覽器（Chrome, Firefox, Safari, Edge）中開啟。
   - 或使用任何本地伺服器：
     ```bash
     python3 -m http.server 8000
     # 瀏覽器開啟 http://localhost:8000
     ```

---

## 🛠️ 技術堆疊

- **HTML5 & Vanilla JavaScript (ES6+)**：模運算邏輯、動態 DOM 算繪、剪貼簿與下載 API。
- **Tailwind CSS (CDN)**：現代暗色 Cyberpunk 介面設計、Glassmorphism 玻璃擬態。
- **HTML5 Canvas API**：像素長條與資訊圖卡即時合成繪製與 PNG 匯出。
- **Lucide Icons**：清晰簡潔的 UI 圖示。

---

## 📄 授權條款

本專案基於 [MIT License](LICENSE) 條款開源發布。
