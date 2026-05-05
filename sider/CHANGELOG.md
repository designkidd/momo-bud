# Changelog

All notable changes to this project will be documented in this file.

---

## [2.21.22] - 2026-05-05
### Added — Hermes Setup Guide
- Hermes (beta) 現在和 OpenClaw (beta) 一樣，在 Connect row 右側顯示 `Setup Guide` 按鈕。
- Setup Guide modal 會依照目前 provider 切換內容；Hermes 會顯示 `.env` 範例與重啟 Hermes gateway 的步驟，包含 `API_SERVER_CORS_ORIGINS=*` 說明與 `API_SERVER_KEY=<your-secret-key>` 佔位。

---

## [2.21.21] - 2026-05-05
### Docs — Hermes `.env` 設定說明
- 設定頁與中英文 AI Providers 文件補充 Hermes Agent `.env` 範例，說明需像 OpenClaw gateway 一樣修改 `API_SERVER_ENABLED`、`API_SERVER_HOST`、`API_SERVER_PORT`、`API_SERVER_KEY` 與 `API_SERVER_CORS_ORIGINS`。
- 文件範例使用 `API_SERVER_KEY=<your-secret-key>` 佔位，不再使用實際密碼；並標明 `API_SERVER_CORS_ORIGINS=*` 是 Chrome extension / 遠端連線最簡單可用設定。

---

## [2.21.20] - 2026-05-05
### Fixed — Hermes Chat 403
- Hermes chat request 改由 background `proxy_fetch` 代發，避免 sidepanel 直接 `fetch` 時與 curl 不一致的 browser Origin / CORS / preflight 差異造成 `HTTP 403`。
- Hermes `/v1/chat/completions` 改用官方 curl 對齊的最小 OpenAI-compatible payload：`model: "hermes-agent"`、`messages`、`stream: false`，不再帶通用 provider 的 `temperature` / thinking 參數。

---

## [2.21.19] - 2026-05-05
### Fixed — Hermes 連線與切換行為
- Hermes 發生 `HTTP 401/403` 或 browser fetch 失敗時，錯誤訊息會直接提示檢查 `API_SERVER_KEY`，並顯示目前 Chrome extension origin，方便在 Hermes host 設定 `API_SERVER_CORS_ORIGINS=chrome-extension://<extension-id>` 後重啟 gateway。
- 從其他模型切換到 Hermes agent provider 時會自動開新對話、清掉目前畫面的舊對話與圖片／頁面引用狀態，避免把其他模型的上下文送入 Hermes。
- Hermes / agent provider 送出 API 前會過濾本地 system message，確保自訂 system prompt 真正不會送到 agent provider。

---

## [2.21.18] - 2026-05-05
### Fixed — OpenClaw 重覆上一輪回覆
- 對齊 OpenClaw Gateway 最新協議與官方 Copilot 行為，`chat.send` 改回 `deliver: true`，避免訊息只被記錄但未正常派送給 agent。
- OpenClaw 串流事件改用 `runId` 關聯本次發送；後備 `chat.history` 輪詢會先建立發送前 history snapshot，只接受本次新增 user 之後的新 assistant 回覆，避免誤抓上一輪回覆並在畫面重覆顯示。

---

## [2.21.17] - 2026-05-05
### Added — Hermes Provider
- 新增 **Hermes (beta)** AI Service Provider，預設 Base URL 為 `http://127.0.0.1:8642/v1`，預設模型為 `hermes-agent`。
- Hermes 採用官方 OpenAI-compatible HTTP API Server 接入，沿用現有 Chat Completions 串流路徑，不使用 OpenClaw 的 WebSocket RPC。
- 設定頁新增 Hermes (beta) provider 選項、官方 favicon 風格 icon、API Server 設定提示與官方文件連結；Hermes 改為跟 OpenClaw 一樣需要先 Connect，成功後才自動啟用 `hermes-agent` 模型，避免未連線或 key/CORS 錯誤時側欄直接送出造成 HTTP 403。
- Hermes Connect 會優先檢查 `/v1/models` 以驗證 `API_SERVER_KEY`，再 fallback 到 `/health` / chat completion，錯誤訊息會提示檢查 `API_SERVER_KEY` 與 `API_SERVER_CORS_ORIGINS`。
- 選取 Hermes (beta) 時，側邊欄會像 OpenClaw 一樣禁用自訂 system prompt、聊天歷史/新對話、聯網搜尋與引用頁面；即使切換前已有 pending page reference，送出時也不會附加到 Hermes。
- README 與中英文 AI Providers 文件同步更新為 12+ providers，並補充 Hermes Agent 啟用 `API_SERVER_ENABLED` / `API_SERVER_KEY` 的設定說明。

---

## [2.21.16] - 2026-05-05
### Fixed — 串流回覆中 Enter 不再停止生成
- 修正 AI 回覆尚未完成時，在輸入框按 `Enter` 或 `Cmd/Ctrl+Enter` 會誤觸發停止生成，造成 `BodyStreamBuffer was aborted` 的問題。
- 現在串流中輸入框的送出快捷鍵會被忽略；使用者可先輸入下一句，待上一輪回覆完成後再按 `Enter` 送出。
- 停止生成仍保留在送出按鈕切換出的停止按鈕上。

---

## [2.21.15] - 2026-05-05
### Changed — 截圖框選交互優化
- 截圖框選遮罩新增操作提示，說明可拖曳選區、雙擊截取整個可見頁面、按 `Esc` 取消。
- 雙擊遮罩會直接選取整個目前可見頁面並自動送出，方便快速模擬全屏截圖翻譯。
- 單擊或過小拖曳不再直接取消遮罩，避免誤觸；仍可用 `Esc` 或右鍵取消。

---

## [2.21.14] - 2026-05-05
### Changed — 截圖上傳支援框選區域
- 「截圖上傳」按鈕改為先在目前網頁顯示框選遮罩，拖選區域後只裁剪該區域並自動送出，避免每次都上傳整個可見分頁。
- 支援按 `Esc` 或右鍵取消框選；過小選區會視為取消。
- 裁剪時依照實際截圖尺寸與 viewport 比例換算座標，保留高 DPI 螢幕下的清晰度。

---

## [2.21.13] - 2026-05-05
### Added — 截圖上傳自動送出
- 在輸入區工具列新增「截圖上傳」按鈕，可截取目前 Chrome 活動分頁的可見畫面，加入圖片訊息並自動送出，方便用視覺模型快速 OCR／翻譯畫面內容。
- 截圖功能會在首次使用時請求 `<all_urls>` 可選權限，以符合 `chrome.tabs.captureVisibleTab` 對截圖權限的要求；PNG 過大時會自動改用 JPEG，仍超過 5MB 則提示縮小視窗後重試。
- 新增繁中、簡中、英文的截圖按鈕與錯誤提示文案。

### Changed — Qwen 預設模型
- Qwen 預設模型改為 `qwen3.5-plus` 與 `qwen3.5-flash`，對齊目前 Qwen API 平台顯示的旗艦模型名稱。

---

## [2.21.12] - 2026-05-05
### Fixed — 引用頁面不再改動原網站 DOM
- 修正 Readability 引用頁面抓取流程會在原頁插入 `<base>` 並移除 hidden 元素，可能導致部分網站（例如 `linux.do` / Discourse 類站點）抓取後排版變成未套樣式狀態的問題。
- 內容提取改為先 clone `document.documentElement`，只在 clone 內加入 `<base>`、清理 script/style/hidden 元素並回傳 HTML；不再修改 live page DOM。

---

## [2.21.11] - 2026-05-05
### Fixed — 聊天畫面抖動與聯網搜尋穩定性
- 修正輸入框 auto-grow、聊天區 scrollbar 與串流自動跟隨互相影響時，造成打字或 AI 串流輸出期間介面抖動的問題；聊天區改為保留穩定 scrollbar gutter，並限制串流自動捲動每個 animation frame 最多執行一次。
- 側邊欄「聯網搜尋」維持智慧觸發：開啟時僅在訊息看起來需要即時網路資訊時搜尋；未開啟時仍保留明確搜尋語的自動觸發。
- 搜尋結果同時注入 system prompt 與最後一則 user message，降低模型忽略搜尋上下文後回答「我沒有連網」的機率。
- Brave / Tavily 搜尋失敗、未設定 API key 或回傳 0 筆時，會自動 fallback 到 DuckDuckGo；OpenRouter `:online` 只作為 client-side 搜尋失敗時的後備。
- 設定頁與繁中／簡中／英文 i18n 說明同步更新為新的聯網搜尋行為。

---

## [2.21.10] - 2026-05-04
### Changed — 引用頁面預設查詢不顯示在對話
- 調整 2.21.9 的 LM Studio 相容修正：空白引用頁面送出時，預設 user query 只在組 API payload 時補入，不再寫進 session，也不會顯示在聊天畫面。
- UI 仍只顯示「Page Referenced / 引用頁面」標籤；LM Studio / Qwen 仍能收到非空 user query 以避免 Jinja template 報錯。

---

## [2.21.9] - 2026-05-04
### Fixed — LM Studio 引用頁面空訊息
- 修正只附加「引用頁面」但輸入框空白時，仍送出空白 `user` 訊息，導致 LM Studio / Qwen Jinja prompt template 報錯 `No user query found in messages` 的問題。
- 現在空白引用送出會自動補上一句「請根據引用頁面內容回答。」作為 user query，引用頁面內容仍保留為獨立 system context。
- 送出按鈕只會依照等待送出的 `_pendingPageContext` 啟用，避免已使用過的舊頁面引用誤判為可送出。

---

## [2.21.8] - 2026-04-12
### Changed — 聯網搜尋觸發邏輯
- 側邊欄「聯網搜尋」開啟時改為**智慧觸發**：仍須通過 `shouldSearch`（明確搜尋語、價格／新聞／天氣等訊號、訊息內 URL 等）才會執行搜尋，**不再每則使用者訊息都搜**，以減少延遲、token 與搜尋 API 用量。
- `year-ref`（今年／去年）判斷移至「創作／翻譯」等略過規則之後，避免「幫我寫一篇 2026 年的故事」誤觸發搜尋；「總結 2025 年…」等仍以年份觸發搜尋。
- 設定頁與 i18n 說明已同步為上述行為。

---

## [2.21.7] - 2026-04-12
### Fixed — 聯網搜尋與簡短追問
- 開啟聯網搜尋時，若使用者只輸入承接上一輪的短句（例如先前談 iPhone、接著問「中國的價錢」），先前僅用**當則訊息**組搜尋關鍵字，容易變成籠統的「中國／價錢」而脫離主題。
- **修正**：從**目前訊息之前**的對話文字擷取主題關鍵字（常見產品／型號等，如 `iPhone`），在判斷為「價格／地點類簡短追問」且字面上尚未帶出商品時，將關鍵字**併入實際搜尋字串**。
- **送 API**：在上述條件下於最後一則純文字 user 訊息前加上一行英文脈絡（`Related topic from earlier in the conversation: …`），讓 OpenRouter `:online` 等由伺服器決定搜尋詞時也能對齊上一輪主題。
- 實作位置：`sider/sidepanel.js`（`buildContextualSearchQuery`、`extractTopicHintsFromText`、`shouldAugmentSearchQuery` 等）。

---

## [2.21.6] - 2026-04-08
### Changed
- 將設計系統預覽文件由 `dev-notes/ui-preview.html` 重新命名為 `dev-notes/design-system-preview.html`，名稱更貼近用途
- 同步更新 Design System Preview 文件內版本字樣至 `v2.21.6`

---

## [2.21.5] - 2026-04-08
### Changed
- `dev-notes/design-system-preview.html` 補齊按鍵交互說明：新增 Primary / Outline / Danger / Disabled 的行為規範，以及分段按鈕（single-select、active 狀態、Auto/Light/Dark 映射）說明
- Design System Preview 文件版本字樣同步更新至 `v2.21.5`

---

## [2.21.4] - 2026-04-06
### Changed
- 產品名稱統一為 **Momo AI Bud**：`manifest.json` 的 `name`、工具列／快捷鍵提示、懸浮球與 iframe `title`、側邊欄與設定頁預設 `<title>`
### Fixed
- 設定頁瀏覽器分頁標題改為依介面語系更新（先前 `options.html` 的 `<title>` 寫死為「設定 - AI Sidebar」，選英文時仍顯示中文）；於 i18n 套用時設定 `document.title`（如英文 `Settings - Momo AI Bud`、繁中 `設定 - Momo AI Bud`）

---

## [2.21.3] - 2026-04-06
### Changed
- README / README.zh-TW：開頭賣點與功能列表對齊（含 Groq、OpenClaw、系統 TTS、字體大小／字粗、多語言、浮球等）；移除已不主打之「深度思考／think deeper」描述

---

## [2.21.2] - 2026-04-06
### Changed
- 同名模型下拉選單不再加 Provider 文字標籤（如 `(Ollama)`），僅靠 Provider icon 區分

---

## [2.21.1] - 2026-04-06
### Fixed — 本地 Provider 同名模型被誤刪
- Ollama / LM Studio 等本地 Provider 手動新增的模型若與其他雲端 Provider 預設模型同名（如在 Ollama 加 `kimi-k2.5`），會被 `sanitizeModels` 和載入時的跨 provider 過濾邏輯誤刪
- 修正：本地 Provider（ollama、lmstudio、無預設模型的 provider）跳過跨 provider 模型歸屬檢查，允許任意模型名稱

---

## [2.21.0] - 2026-04-06
### Added — Groq Provider
- 新增 Groq AI 供應商，預設模型 `openai/gpt-oss-120b` 和 `meta-llama/llama-4-scout-17b-16e-instruct`
- API Key 申請連結指向 https://console.groq.com/keys
- 設定頁顯示 Groq 免費方案說明：無須綁卡、每分鐘約 30 次請求、每日最高 14,400 次、全球頂尖推理速度
- `host_permissions` 加入 `api.groq.com`

---

## [2.20.1] - 2026-04-06
### Changed
- Google AI 預設模型更新為 `gemini-3.1-flash-lite-preview` 和 `gemini-3-flash-preview`

---

## [2.20.0] - 2026-04-06
### Fixed — 同名模型跨 Provider 共存
- 模型唯一識別改用 `provider::name` 組合 uid，不同 Provider 下的同名模型可同時啟用並出現在 sidebar 下拉選單
- 同名模型在下拉選單中自動加上 Provider 名稱標籤（如 `gemini-3.1-flash (Google AI)` / `gemini-3.1-flash (OpenRouter)`）
- 向下相容：舊格式的 model 選擇會自動遷移為新 uid 格式

---

## [2.19.5] - 2026-04-06
### Fixed — 測試連線不再彈出權限請求
- 將所有內建 AI Provider 的 API 域名加入 `manifest.json` 的 `host_permissions`，包括 DeepSeek、Google AI、MiniMax、Moonshot、NVIDIA、OpenRouter、Qwen
- 使用者按 Start Test 時不再觸發 Chrome「has requested additional permissions」彈窗
- 自訂 URL 或 localhost（Ollama / LM Studio）仍會透過 `optional_host_permissions` 動態請求

---

## [2.19.4] - 2026-04-05
### Changed — OpenClaw 設定頁 UX 重構
- **Start Test → Connect / Disconnect**：按鈕改為 Connect，連線成功後變為 Disconnect（hover 變紅），點擊可斷開連線
- **Connected 後自動載入 Session**：Connect 成功後自動呼叫 Load Sessions，不需手動操作
- **Session 切換自動載入對話記錄**：選擇不同 Session 後，側邊欄自動偵測 `providerConfigs.sessionKey` 變更並載入 `chat.history`
- **Enabled Models 預設 off**：未 Connected 前 toggle 為 off + disabled，Connected 後自動開啟
- **Load 按鈕改為 Refresh icon**：Session 旁的 Load 按鈕改為 🔄 refresh icon（`btn btn-outline small`），loading 時旋轉動畫
- **Setup Guide 移到右側**：與 Connect 按鈕同行，左邊 Connect，右邊 Setup Guide
- **Enabled Models 標題改為二級標題**：與 Session 一致的 `field-label` 樣式，左對齊
- **Session 切換提示**：切換 Session 時顯示「切換中…」→「✓ 已切換」狀態提示
- **Model name 簡化**：`openclaw:main` → `openclaw`（HTML、JS、utils.js）

---

## [2.18.1] - 2026-04-05
### Changed
- **OpenClaw 設定頁簡化**：OpenClaw provider 不再顯示完整的 "Enabled Models" 列表和 "Add Model" 按鈕，改為單一 `openclaw:main` 開關；切換回其他 provider 時恢復完整模型列表

---

## [2.17.9] - 2026-04-05
### Fixed — Provider Hint i18n 完整修復（2.17.5 ~ 2.17.9）

此問題經歷多次迭代才完全修復，記錄根因和陷阱以防再犯：

**問題**：設定頁 Provider Configuration 的 hint（"Leave empty to use default: ..." + "Apply for API Key here: ..."）在初始載入或切換語言時顯示不正確。

**根因分析（共 4 層）**：

1. **`data-i18n` 覆蓋動態內容（2.17.6）**
   - `providerBaseUrlHint` 有 `data-i18n="useDefaultUrl"`，i18n-loader 的 `__applyTranslations()` 會把 JS 動態生成的 innerHTML（含 URL + `<a>` 連結）覆蓋回簡單的翻譯字串
   - ✅ 修復：移除 HTML 中的 `data-i18n`，讓 JS 完全控制 hint 內容

2. **`currentLang` 設定時序（2.17.7）**
   - `loadProviderConfig` 的 i18n 重新呼叫在 `currentLang` 設定之前執行，`t()` 使用 fallback `'en'`
   - ✅ 修復：把重新呼叫移到 `await __applyTranslations(lang)` 完成且 `currentLang` 已設定之後

3. **語言切換未觸發 hint 更新（2.17.8）**
   - 切換語言時只呼叫了 `applyLanguageConversion()`，沒有重新呼叫 `loadProviderConfig`
   - ✅ 修復：語言切換 callback 中加入 `await applyLanguageConversion()` + `loadProviderConfig()`

4. **`applyLanguageConversion` 的 async 陷阱（2.17.9）**
   - 函數標記為 `async` 但內部用 `chrome.storage.local.get(key, callback)` — callback 模式不會被 `await` 等待，Promise 立即 resolve
   - ✅ 修復：改用 `await chrome.storage.local.get(key)`（MV3 Promise API），讓 `await` 真正等待完成

**設計原則（避免再犯）**：
- 動態生成的 UI 內容不要加 `data-i18n`，否則 i18n-loader 會覆蓋
- 依賴 i18n 翻譯的 UI 更新必須在 `__applyTranslations()` 完成後執行
- MV3 中 `chrome.storage` API 優先用 Promise 版本，避免 callback + async 的陷阱
- 語言切換時，所有 JS 動態生成的 i18n 內容都需要手動重新渲染

---

## [2.17.3] - 2026-04-05
### Added
- **所有雲端 Provider 加入 API Key 申請連結**：新增 Moonshot、NVIDIA、MiniMax、OpenRouter 的 API Key 申請頁面連結（LM Studio、Ollama、OpenClaw 為本地服務，不需要）

---

## [2.17.2] - 2026-04-05
### Changed
- **所有 AI Provider 預設模型精簡為兩個主流模型**（全部預設關閉）：
  - DeepSeek: `deepseek-chat`, `deepseek-reasoner`
  - Google AI: `gemini-2.5-flash`, `gemini-2.5-pro`
  - MiniMax: `MiniMax-Text-01`, `abab6.5s-chat`
  - Moonshot: `kimi-k2.5`, `kimi-k2`
  - NVIDIA: `nemotron-3-super-120b-a12b`, `nemotron-3-nano-30b-a3b`
  - OpenAI: `gpt-5.4-mini`, `gpt-5.4`
  - OpenRouter: `anthropic/claude-sonnet-4.6`, `deepseek/deepseek-chat`
  - Qwen: `qwen3-max`, `qwen-plus`
  - Ollama: 清空預設（本地模型由使用者自行新增）
  - LM Studio / OpenClaw: 不變

---

## [2.17.1] - 2026-04-05
### Fixed
- **「載入」按鈕中文未 i18n**：`options.html` 中 OpenClaw Session 的「載入」按鈕和 placeholder 文字改為英文預設並加上 `data-i18n="loadSessions"`，中文系統由 i18n-loader 自動替換

---

## [2.17.0] - 2026-04-05
### Changed
- **全面 i18n 國際化**：移除所有 JS 檔案中的 hardcoded 中文字串（約 50+ 處），改用 i18n key 或英文 fallback
  - `js/openclaw.js`：5 處錯誤訊息改為英文
  - `sidepanel.js`：OpenClaw 錯誤、頁面引用 UI、session 標題、圖片 alt、搜尋結果、格式化錯誤訊息等
  - `options.js`：按鈕文字、WebSocket 錯誤、TTS 預覽 fallback 等
  - `content-floatball.js`：3 個 tooltip 改為英文
- **i18n JSON 新增 25 個 key**（`en.json`、`hant.json`、`hans.json` 三語同步）

### Fixed
- **英文系統不再顯示中文錯誤訊息**：如「載入失敗」「連線逾時」「無可用 Session」等

---

## [2.16.5] - 2026-04-05
### Fixed
- **OpenClaw 回覆太慢導致「未回應」問題**：
  - `chat.send` RPC 超時從 30 秒增加到 60 秒
  - `chat.send` 超時時若事件流已有回應，不再直接結束，繼續等待回覆
  - 新增輪詢穩定檢查：備用輪詢取得回覆後若 15 秒內無新內容變化，自動視為完成

---

## [2.16.4] - 2026-04-05
### Changed
- **移除 hardcoded API Key**：NVIDIA、Ollama、Qwen 的 `defaultApiKey` 和 `enforceDefaultEnabled` 已移除，所有模型預設為關閉

---

## [2.16.3] - 2026-04-05
### Changed
- **模型選擇器固定寬度**：`#modelSelector` 寬度固定為 160px，不再隨模型名稱長度變動

---

## [2.16.2] - 2026-04-05
### Changed
- **按鈕順序調整**：Upload Image 和 Reference Page 按鈕位置互換（圖片上傳在前）

---

## [2.16.1] - 2026-04-05
### Fixed
- **OpenClaw 模式禁用聯網搜尋與引用頁面按鈕**：選擇 OpenClaw 模型時，Web Search 和 Reference Page 按鈕現在會被灰掉並禁用（opacity 0.35、pointer-events none），避免使用者誤觸無效功能；切換回其他模型時自動恢復

---

## [2.16.0] - 2026-04-05
### Added
- **UI Design System Specification** (`sider/UI-SPEC.md`): formal document covering all design tokens (light/dark), typography, spacing, border radius, component interaction states (4-state: default/hover/focus/disabled), dark mode rules, animation standards, responsive breakpoints, iconography, and accessibility guidelines
- **Cursor Rule** (`.cursor/rules/ui-design-system.mdc`): AI-readable condensed spec — auto-applies when editing CSS/HTML/JS under `sider/`
- **Design System Preview Page** (`dev-notes/design-system-preview.html`): standalone visual reference with live light/dark toggle, color swatches, type scale, spacing bars, radius grid, and interactive component demos
- **Cross-file token aliases**: `options.css` now defines `--accent`/`--accent-hover`; `sidepanel.css` now defines `--bg-page`/`--bg-card`/`--bg-subtle`/`--bg-active`/`--bg-active-hover`/`--focus` — both naming conventions work in both files

### Changed
- **iframe-sidebar.css dark mode**: migrated from `@media (prefers-color-scheme: dark)` to `[data-theme="dark"]` attribute on sidebar container, driven by `chrome.storage` theme setting via `content-floatball.js`; dark background unified to `#202223` (was `#1a1a1a`)
- **Form element width alignment**: removed `max-width: 600px` from `.provider-select-wrapper`; added `box-sizing: border-box` to `.input` and `.provider-select-button`; `.inline-row > .input` now uses `flex:1` — all form elements align to card padding edges consistently

---

## [2.15.0] - 2026-04-04
### Added
- **OpenRouter server-side web search**: when web search is enabled and provider is OpenRouter, automatically appends `:online` suffix to model name — search is handled server-side via Exa/native engine, giving much better results than client-side DDG scraping
- **OpenRouter annotations**: parses `url_citation` annotations from OpenRouter responses and displays them in the search sources modal
- **Auto-trigger search on explicit intent**: messages starting with "查一下", "搜尋", "search for" etc. now auto-trigger web search even when the toggle is OFF
- **DuckDuckGo proxy fallback**: when direct HTML fetch fails, retries through the background service worker `proxy_fetch` to bypass CORS/anti-scraping blocks

### Changed
- **Search always fires when toggle is ON**: previously, `shouldSearch()` could still skip queries even when the user explicitly enabled search — now the toggle means "always search"
- **PROMPT_BUDGET tripled** from 2000 → 6000 characters — search results no longer get aggressively truncated
- **Per-result snippet limit** increased (≤3 results: 1200 chars each; >3 results: 600 chars each)
- **Improved search injection prompt**: follows OpenRouter-style format — simpler, more natural, explicitly tells model not to claim it lacks internet access
- **Removed OpenClaw web search injection**: OpenClaw has native search capability, no longer needs client-side injection

---

## [2.14.3] - 2026-04-04
### Added
- **Prompt visibility toggle**: each system prompt card now has an on/off switch — hidden prompts won't appear in the sidebar's prompt dropdown (data stored as `visible` property, defaults to `true`)

### Changed
- **Smaller toggle switches** across the entire options page: 46×26px → 36×20px (knob 18px → 14px) for a cleaner, more modern look
- **Field labels** enlarged from 12px/secondary color to 14px/primary color — second-level headings now clearly stand out from body text
- **Web search toggle layout**: "Simple Search Mode", "Visit Website in Message", and "Internet Search ON by Default" toggles moved from left-of-hint to right-aligned (new `.toggle-row` layout matching Float Ball style)

---

## [2.14.2] - 2026-04-04
### Fixed
- **Options anchor scroll**: added `scroll-margin-top: 100px` to all sections so nav anchor jumps don't hide content behind the viewport top
- **Streaming auto-scroll**: reworked auto-follow logic using `requestAnimationFrame` and `_programmaticScroll` flag — user scroll-up now properly pauses auto-follow; removed `scroll-behavior: smooth` from chat scroller where it caused jank

---

## [2.14.1] - 2026-04-04
### Added
- **First-install language detection**: `background.js` now writes `zhVariant` to both local and sync storage on install, based on Chrome UI language (`chrome.i18n.getUILanguage()`)
- **`__detectBrowserLanguage()`** function in i18n-loader.js — all fallback defaults now use detected browser language instead of hardcoded `'hant'`

### Changed
- **Dark mode provider icons**: black SVG icons get `filter: brightness(0) invert(1)` in dark theme; colored brand SVGs (DeepSeek, Google, OpenRouter, Qwen, OpenClaw) excluded from inversion

---

## [2.14.0] - 2026-04-04
### Changed
- **Options sidebar redesign**: 7 nav items with SVG icons (General, Sidebar, AI Models, Web Search, Capture, TTS, Prompts); merged related sections per nav item via `data-sections`
- **TTS section** moved after Page Capture for better flow
- **Scroll spy** updated to use `data-sections` attribute for multi-section nav highlighting
- New i18n keys: `navGeneral`, `navSidebar`, `navAiModels`, `navWebSearch`, `navCapture`, `navTts`, `navPrompts`

---

## [2.13.3] - 2026-04-04
### Fixed
- **`shouldSearch` heuristics**: fixed `\b` word boundary failing with CJK characters; improved entity / alphanumeric query detection; added explicit triggers like `你查一下`

---

## [2.13.2] - 2026-04-04
### Added
- **Conservative `shouldSearch`**: default behavior is now no-search unless explicit signals detected (search keywords, question patterns, entity queries)

### Changed
- **Chat history panel**: width set to `min(240px, 72%)` with responsive tweaks
- **Custom dialog** dark mode styles improved; slightly narrower `max-width`

---

## [2.13.1] - 2026-04-04
### Fixed
- **Theme `auto` mode**: `applyTheme('auto')` now resolves to `dark`/`light` using `prefers-color-scheme` media query + live listener for system theme changes (both `options.js` and `sidepanel.js`)

---

## [2.13.0] - 2026-04-04
### Added (inspired by [page-assist](https://github.com/n4ze3m/page-assist))
- **Google HTML scraping**: New default search engine — scrapes Google search pages directly from Chrome (most stable, free, no API key needed)
- **DuckDuckGo HTML scraping**: Re-added as option via html.duckduckgo.com
- **Simple Internet Search toggle**: ON = use search snippets only (fast); OFF = visit result pages for full content (slower but more accurate RAG-like experience)
- **Total Search Results**: Configurable number of results (1-20, default 5)
- **Visit Website in Message**: Automatically detects URLs in user messages and fetches page content for the AI
- **Internet Search ON by Default**: Option to auto-enable web search for every new chat
- **SearXNG**: Open-source meta-search with 10 fallback instances (JSON + HTML fallback)

### Changed
- Default search provider changed from Tavily to Google (no API key needed)
- Redesigned web search settings page with all new options
- `performWebSearch` now combines URL-visited content with search results

---

## [2.10.7] - 2026-04-04
### Added
- **Search sources viewer**: 🔍 button appears in AI response action bar when web search was used; clicking it opens a modal showing all search result sources with titles, URLs, and snippets (similar to page reference viewer)
- **Search query optimization**: `extractSearchQuery()` strips instruction prefixes like "幫我查", "上網搜索", "請告訴我" etc. to get cleaner search keywords

### Changed
- **Search result injection**: Now uses a dedicated system message with strong Chinese instructions telling the AI to prioritize and cite search results (previously appended to user message, which models often ignored)

---

## [2.10.6] - 2026-04-04
### Fixed
- **Search query optimization**: Extracts keywords from user message instead of searching the entire sentence (strips "幫我查", "搜索", "上網找" etc.)
- **Search result injection**: Changed from appending to user message to a dedicated system message with strong Chinese instructions telling the AI to prioritize and cite search results
- OpenClaw path also uses improved injection format

---

## [2.10.5] - 2026-04-04
### Fixed
- **Provider switching**: Settings changes in options page now take effect immediately in side panel (added `storage.onChanged` listener)
- **DuckDuckGo**: Routes requests through background service worker (proxy_fetch) to bypass anti-bot detection; falls back to direct fetch if proxy fails
- **Default provider**: Changed from DuckDuckGo to Tavily (more reliable); DDG marked as "unstable" in UI
- **Toggle feedback**: Web search toggle now shows ON/OFF toast notification

---

## [2.10.4] - 2026-04-04
### Fixed
- **Tavily API**: Updated authentication from deprecated `api_key` in request body to `Authorization: Bearer` header (Tavily API 2026 format change)
- **DuckDuckGo**: Added 3 retry strategies (HTML POST, Lite POST, HTML GET), regex fallback parser, and User-Agent header for better compatibility
- **Test button**: Now shows detailed DDG debug info (raw HTML snippet, attempt log) when search returns no results, making diagnosis much easier

---

## [2.10.0] - 2026-04-04
### Added
- **Web Search**: new toggle button (globe icon) in the composer area enables live web search
  - When enabled, each message automatically searches the web and injects results as context before sending to the AI model
  - Default provider: **DuckDuckGo** (free, no API key required)
  - Optional provider: **Tavily** (higher quality results, requires free API key from tavily.com)
  - Works with both standard OpenAI-compatible API and OpenClaw WebSocket paths
  - Search state persists across sessions
  - New settings section in Options page for search provider configuration
  - Full i18n support (繁中 / 简中 / English)

---

## [2.9.33] - 2026-03-31
### Changed
- Settings page: all hardcoded Chinese strings replaced with English equivalents
  - `模型名稱` label → `Model Name`
  - `顯示思路` (CSS content) → `Show Thinking`
- Provider thinking hints (`THINKING_HINTS`) and Prefix hint now fully localised — hant / hans / en versions; hint re-renders after `currentLang` is resolved from storage (was always defaulting to `hant`)
- UI hover effects toned down across settings page — `.model-row`, `.icon-btn`, `.sp-card` hover now uses `var(--bg-card)` instead of hardcoded `#fff`; border no longer strengthens on hover
- System Prompt cards: active/selected state no longer shows coloured border or box-shadow — selection indicated by radio dot only; background stays `var(--bg-subtle)` in both light and dark mode (consistent interaction)
- Translator default prompt: Terminology Notes section updated to specify explanation language direction (Chinese explanation for English source, and vice-versa)

---

## [2.9.16] - 2026-03-30
### Changed
- Thinking toggle now aware of each model's actual capability (3 states):
  - **toggleable** — Qwen, Gemini 2.5, Claude, Kimi, MiniMax: button works normally
  - **always_on** — DeepSeek Reasoner, o1, o3: button locked + accent colour, tooltip explains
  - **unsupported** — GPT-4o, Ollama generic, LM Studio etc: button greyed out, tooltip explains
- Button auto-updates when user switches models
- Correct API params per provider when thinking is on:
  - Qwen / Kimi / MiniMax / Moonshot → `enable_thinking: true`
  - Gemini 2.5 → `thinkingConfig: { thinkingBudget: 8000 }`
  - Claude → `thinking: { type: "enabled", budget_tokens: 10000 }` + `temperature: 1`
  - DeepSeek Reasoner → `response_format: { type: "text" }` only (always on)
- Added i18n keys: `thinkingAlwaysOn`, `thinkingUnsupported`

---

## [2.9.15] - 2026-03-30
### Added
- Global thinking mode toggle button in the composer toolbar (lightbulb icon, next to page-context button)
- One click turns thinking on/off for **all** providers and models universally
- State persists via `chrome.storage.sync` (`globalThinking` key)
- When enabled: `enable_thinking: true` is sent in every API request, and `<think>` blocks are rendered regardless of model name or per-provider setting
- Per-provider thinking setting in options still works as before; global toggle is an OR override

---

## [2.9.14] - 2026-03-30
### Fixed
- Language switching broken after sync migration — `zhVariant` was removed from the local read, so first load after update always defaulted to `hant`
- Language/theme/font switches now write to **both** local (for instant reload) and sync (for cross-device)
- `sidepanel.js` `loadTheme()` and `awaitGetZhVariant()` now fall back to local if sync value is absent
- Added all UI settings back to local read as migration fallback (`zhVariant`, `theme`, `messageSize`, `messageWeight`, `showFloatBall`, capture settings)

---

## [2.9.13] - 2026-03-30
### Changed
- NVIDIA icon replaced with official LobeHub SVG (sourced from `@lobehub/icons-static-svg`) — uses `currentColor`, single-path design matching the real NVIDIA wordmark/eye logo

---

## [2.9.12] - 2026-03-30
### Changed
- All settings now sync across Google accounts via `chrome.storage.sync`
- Synced keys: provider configs (API keys, base URLs, models), `activeProvider`, `model`, `theme`, `zhVariant`, `messageSize`, `messageWeight`, `showFloatBall`, all page capture settings, `pageContextLimit`
- Chat sessions and computed aggregates (`customModels`, `providerConfigs`) remain in local storage
- `content-floatball.js` and `background.js` updated to read/write floatball state from sync
- `loadAll()` reads sync first with local fallback for seamless migration of existing users

---

## [2.9.11] - 2026-03-30
### Changed
- All provider models are now fully editable — users can delete or rename any default model
- Removed readonly lock and hidden delete button on default models in the model list
- Removed `ensureDefaultModels` enforcement from save, load, and persist flows (defaults only apply on first-time init)

---

## [2.9.10] - 2026-03-30
### Fixed
- Provider select button showed wrong icon (momo) for newly added providers — replaced hardcoded icon map with `PROVIDER_ICONS` from utils.js so all providers resolve correctly

---

## [2.9.9] - 2026-03-30
### Changed
- Provider list sorted alphabetically: DeepSeek → Google AI → LM Studio → MiniMax → Moonshot → NVIDIA → Ollama → OpenClaw → OpenAI → OpenRouter → Qwen

---

## [2.9.8] - 2026-03-30
### Added
- MiniMax provider (`api.minimaxi.chat/v1`), models: MiniMax-Text-01, abab6.5s-chat, abab6.5-chat
- Moonshot provider (`api.moonshot.cn/v1`), models: moonshot-v1-8k/32k/128k, kimi-k2
- OpenRouter provider (`openrouter.ai/api/v1`), preset models: GPT-4o, Claude 3.5 Sonnet, Gemini 2.5 Pro, DeepSeek
- Icons for all three providers sourced from LobeHub icons CDN

---

## [2.9.7] - 2026-03-30
### Changed
- NVIDIA icon replaced with proper eye/swirl logo SVG (traced from official PNG)

---

## [2.9.6] - 2026-03-30
### Changed
- Replaced all native browser `confirm()` / `alert()` dialogs with custom-styled modals matching the extension UI (teal accent, rounded corners, dark mode support)
- Affects: delete session, delete all/selected sessions, export, import, reset prompts, image upload warnings, streaming warnings, TTS, paste errors (17 call sites total)

---

## [2.9.5] - 2026-03-29
### Fixed
- Empty sessions (no messages sent) no longer appear in chat history
- Clicking "New Chat" while already on an empty session no longer creates a duplicate empty entry
- Empty sessions are cleaned up from storage on every save

---

## [2.9.4] - 2026-03-29
### Changed
- Welcome screen suggestions updated (方案 D):
  - 隨機給我一個沒用但有趣的冷知識 🎲
  - 幫我想一個今天說出去能唬人的話 🎭
  - 給我一句讓人覺得我很有深度的話 ✨

---

## [2.9.2] - 2026-03-29
### Added
- NVIDIA provider: OpenAI-compatible endpoint (`integrate.api.nvidia.com/v1`), model `moonshotai/kimi-k2.5`

---

## [2.9.1] - 2026-03-29
### Added
- Settings page: left sidebar navigation with anchor links for quick section jumping
- Scroll spy — current section is highlighted as you scroll
- Sidebar auto-hides on screens ≤ 820px (responsive)

---

## [2.9.0] - 2026-03-29
### Added
- **OpenClaw provider (beta)**: WebSocket-based AI gateway integration
  - Session selector: load and switch between OpenClaw gateway sessions
  - Auto-reconnect with backoff strategy
  - RPC protocol aligned with Copilot architecture
  - Tutorial modal with step-by-step setup instructions (Server setup + Device Auth)
- **Google AI provider**: Gemini 2.5 Flash / Pro / Pro MaxThinking models
- **Thinking mode toggle**: enable `enable_thinking` parameter for supported providers (Qwen, etc.)
- **Image upload**: attach multiple images to messages (base64 encoded, inline preview)
- **Page content capture**:
  - Expand/collapse preview card in the composer area
  - Cancel capture mid-flight
  - Tracks last captured URL to avoid redundant re-fetches
- **Adaptive streaming renderer**: throttles DOM updates by content length (80ms / 300ms / 800ms intervals) to prevent jank on long responses
- **Character counter** in the message input
- **Scroll-to-bottom button** with intelligent auto-follow (pauses when user scrolls up)
- **i18n system**: full Traditional Chinese / Simplified Chinese / English UI localization

### Changed
- Shared code extracted into dedicated modules: `js/utils.js`, `js/storage.js`, `js/openclaw.js`, `js/markdown.js`
- Provider defaults (models, base URLs, icons) now centralized in `js/utils.js`
- Default prompts moved to `prompt-defaults.js` (shared by sidepanel + background)
- Background service worker uses `declarativeNetRequest` to rewrite WebSocket `Origin` header for OpenClaw (static rules for 127.0.0.1 + dynamic rule for custom gateway URLs)
- Qwen models updated: added `qwen3-max`; default model is `qwen-flash`
- Ollama models updated: `qwen3-vl:235b` added

### Fixed
- Side panel now uses global configuration (not per-tab), ensuring consistent state across all tabs
- Provider icon path resolution uses `chrome.runtime.getURL` to avoid CSP issues

---

## [2.8.0] - 2026-03-28
### Added
- `floatball-frame.html` for the floating ball iframe

### Changed
- Background service worker refactored: auto-injects floatball content script on install/update to all tabs
- Browser language auto-detected on first install (English / Traditional Chinese / Simplified Chinese)

---

## [pre-2.8.0] - (no git history before this point)
### Notes
- Baseline feature set: streaming chat, multi-provider support (Qwen, OpenAI, DeepSeek), Markdown rendering, chat session history, page content capture (Readability + Turndown), system prompt management, dark/light theme
