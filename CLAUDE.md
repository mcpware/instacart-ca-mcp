

---

## 由 Claude memory 搬入（2026-08-21）— `project_instacart_ca_mcp.md`

原本住喺 `~/.claude/projects/-home-nicole/memory/`，每個 session 都載入。
呢啲係 project-specific，應該 cd 入呢個 repo 先 load。

---
name: project-instacart-ca-mcp
description: instacart-ca-mcp repo — AI 跨店比價 MCP，自己擁有 Chromium profile 連 instacart.ca 內部 GraphQL
metadata: 
  node_type: memory
  type: project
  originSessionId: 1d13e80d-dd48-4621-bdc1-440aa4665a71
---

`~/MyGithub/instacart-ca-mcp`（public GitHub: **github.com/mcpware/instacart-ca-mcp** — transfer 咗去 mcpware org，唔再喺個人 ithiria894 下面）。README 拆兩個檔：`README.md`（英）+ `README.zh-HK.md`（廣東話），互相 link。Demo GIF 喺 assets/，跟 chatbotlite 套路整（HTML scene → Playwright screencap → ffmpeg）。MCP server 畀 AI 自己去 instacart.ca 搜尋商品 + 跨店比價，唔使人手撳 browser。配合 [[project_meal_prep]]（買餸前比價）。

Tools: `instacart_compare <query>`、`instacart_search`、`instacart_check`。

**架構（reusable lesson）**：唔好連現有 Chrome — modern Chrome 鎖死 CDP `/json` + `/json/version`（404），連 Playwright `connectOverCDP` 都死，browser-level ws 有 origin 限制。正解 = 自己擁有 headless Chromium profile（`launchPersistentContext`，profile `~/.instacart-ca-mcp/profile`），登入一次 session 寫落 disk。

**三個坑**：(1) product 錨點係 `"id":"items_<shop>-<pid>"` 唔係 productId，price 喺後面 5-6KB，window 要 9KB。(2) search endpoint 要 `x-client-identifier: web` header 否則 401。(3) `__Host-instacart_sid` 係 session cookie，import 時要 pin expiry 先 persist；Playwright addCookies 用 url（唔好 url+path 一齊）。

**登入**：`INSTACART_HEADLESS=false npm run login`（開窗），或 `npm run import-cookies <file>`（抄 cookie header）。Session 幾星期到幾個月過期，`instacart_check` 報 loggedIn:false 就重做。

常數（zone-specific，溫哥華 V6B6H4/755）：已 capture 晒全 zone **33 間去重店**嘅 shopId（Walmart 9057、Superstore 3702、Costco 5780、T&T 42632、Save-On 25116、Whole Foods、IGA、Wholesale Club、London Drugs… 全部喺 src/instacart.ts 個 STORES）。persistedQuery hash 都喺嗰度。`comparePrices` 預設比晒所有店，用 bounded concurrency（batch 4，mapLimit helper）。攞店名方法：對每個 shopId 跑 SearchResultsPlacements query，response 入面個大寫 `"name"`（唔係 lowercase aisle slug）就係店名。

**待辦**：Claude Code MCP host 仲 load 緊舊 Strider server；`~/.mcp.json` 已指去新 server 但要 restart 先 reload。
