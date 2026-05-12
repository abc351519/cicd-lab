# 2026 NTU CI/CD 作業 — 檢查結果與繳交步驟

## 一、目前 `ci_b12705037.yaml` 與要求對照

| 要求                                    | 狀態                   | 說明                                                                                                                                                                                         |
| --------------------------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 檔名 `.github/workflows/ci_{學號}.yaml` | 符合                   | `ci_b12705037.yaml`（請確認學號是否為 **b12705037**）                                                                                                                                        |
| push 時自動執行                         | 大致符合，建議留意     | 目前僅在 **`push` 到 `main`** 時觸發。若希望「任何分支 push 都跑 CI」，請把 `on.push` 改成不限制 `branches`，或加上你會用到的分支（例如 `feature/**`）                                       |
| TypeScript typecheck                    | 符合                   | `npm run typecheck`（`tsc --noEmit`）                                                                                                                                                        |
| Prettier check                          | 符合                   | `npm run format:check`                                                                                                                                                                       |
| Test                                    | 符合                   | `npm run test`，並輸出 JUnit                                                                                                                                                                 |
| 任一步失敗 → Workflow 顯示失敗          | 符合                   | 各 step 非 0 結束時，整個 job 會失敗；`publish-unit-test-result-action` 設了 `action_fail: true`（v2 輸入名稱）                                                                              |
| 測試結果顯示在 Actions 結果頁           | 符合（需實際跑過確認） | `EnricoMi/publish-unit-test-result-action` 會把 JUnit 發佈到 **Checks / 測試結果**；若 GitHub 上沒出現，請在 workflow 頂層加上 `permissions: { checks: write }`（依倉庫預設 token 權限而定） |
| 使用 Marketplace Actions                | 符合                   | `actions/checkout`、`setup-node`、`upload-artifact`、`EnricoMi/publish-unit-test-result-action`                                                                                              |

**結論：** 以作業文字來看，設計已涵蓋主要得分點；建議你實際 push 一次到 GitHub，確認 Actions 綠燈且測試結果區塊有出現。若學號不是 b12705037，請重新命名檔案並調整內容敘述。

---

## 二、你接下來要做的「動作」清單

### 1. 確保檔案在正確分支並推上 GitHub

- 將 `ci_b12705037.yaml` commit 後 **push** 到會觸發 CI 的分支（依你目前設定為 **`main`**）。
- 到 GitHub：`https://github.com/<你的帳號>/cicd-lab/actions` 打開最新一次 **CI** workflow run。

### 2. 成功執行 — 截圖（報告至少一張）

建議截兩種畫面（可擇一或都放）：

1. **Actions 列表頁**：顯示 workflow 名稱 **CI**、綠色勾勾、觸發分支與 commit。
2. **單次 run 詳情頁**：左側 steps 全部綠色；若有 **Annotations** / **Test results** / **Checks** 區塊，一併截進去，以呼應「測試結果顯示於結果頁面」。

### 3. 失敗案例 — 故意製造錯誤再截圖

任選一種（做完記得 **還原** 並再 push 一次讓 main 綠燈）：

| 類型       | 簡單做法範例                              | 預期                             |
| ---------- | ----------------------------------------- | -------------------------------- |
| TypeScript | 在某 `.ts` 故意寫錯型別或引用不存在的變數 | `TypeScript typecheck` step 失敗 |
| Prettier   | 故意少縮排、多加空行不存檔 format         | `Prettier check` step 失敗       |
| 測試       | 暫時改某測試 `expect(1).toBe(2)`          | `Run tests...` step 失敗         |

- 截圖：**紅色失敗**的 workflow run，並可點進失敗的 step 看 log 首段錯誤訊息。
- 報告文字：寫「**錯誤原因**」（例如哪個檔案、什麼規則）與「**如何修正**」（例如跑 `npm run format:fix` 或還原程式）。

### 4. 產出 PDF 報告

- 用 Word / Google Docs / Markdown 轉 PDF 皆可。
- **檔名**：`學號_姓名_CICD_作業.pdf`（請替換成你的學號與姓名）。

---

## 三、報告建議章節與「要打什麼字」

以下可直接當大綱，用你自己的話改寫即可。

### 1. CI Pipeline 說明

- **目的**：在每次 push（你目前寫在 `main`）時自動驗證程式可編譯、格式一致、測試通過，並將測試報告呈現在 GitHub。
- **觸發條件**：`on: push`（說明你限制在 `main` 的原因，或若已改成全分支一併說明）。
- **執行環境**：`ubuntu-latest`、Node **22**、`npm ci` 安裝依賴。

### 2. 主要內容（貼上 YAML）

- 貼上 `.github/workflows/ci_b12705037.yaml` **全文或核心片段**（作業要求「主要內容」即可，不必硬塞 500 行 log）。
- 可用程式碼區塊排版，PDF 裡字體別太小。

### 3. Pipeline 設計說明（對應報告 20%「工具與策略」）

建議分段寫：

- **Typecheck**：使用專案 `package.json` 的 `typecheck`，與本機開發一致，避免「只在 CI 才過編譯」的落差。
- **Prettier**：`format:check` 只做檢查不寫檔，確保 repo 風格一致。
- **測試**：Vitest `run` 模式；`--reporter=junit --outputFile=...` 產生 **JUnit XML**，供後續上傳與彙總。
- **測試結果可視化**：`actions/upload-artifact` 保留 `junit.xml` 備查；`EnricoMi/publish-unit-test-result-action` 將結果發佈到 GitHub，方便在 Actions / Checks 檢視通過與失敗案例數。
- **失敗行為**：任一步驟 exit code 非 0 即整體 failed；與作業「任一檢查失敗應顯示失敗」一致。

### 4. CI 執行結果截圖

- **成功**：至少一張（見上文第二節）。
- 可補一句：commit SHA、執行日期、觸發方式（push）。

### 5. 失敗案例說明

- **你故意改了什麼**（檔案路徑 + 一行話）。
- **Pipeline 哪個 step 紅了**、log 關鍵一行（可貼文字不必全貼）。
- **如何修正**（還原或跑哪個 npm script）。

### 6.（選）延伸練習

- 未計分可略寫「未實作」或一兩句未來想接公有雲的方向。

---

## 失敗案例說明（報告可直接改寫）

以下對應本 repo 已推送的 **故意失敗** commit：`chore: intentional test failure for homework demo`（短 SHA **`ceda8b3`**）。GitHub Actions 會永久保留該次的 **紅色 Failure**；修復 commit 為 **`af10c7a`**（`fix: restore passing tests; document…`）。到 **Actions → CI** 在列表中找 **ceda8b3** 或訊息含 _intentional test failure_ 的那次 run 截圖即可（目前 `main` 尖端已是綠燈，不影響繳交）。

### 故意製造的錯誤

- **類型**：單元測試失敗（Vitest）。
- **檔案**：`test/app.test.ts`，案例「GET / returns app message and version」。
- **內容**：將 `expect(response.json().message).toBe('CI/CD Lab Fastify app is running')` 改為預期錯誤字串（例如 `'WRONG_MESSAGE_INTENTIONAL_FAILURE'`），使斷言與實際回傳不一致。

### Pipeline 現象

- **失敗的 step**：`Run tests and generate JUnit report`（其後步驟可能因 job 已失敗而略過或一併失敗，依 runner 行為而定）。
- **原因**：Vitest 斷言失敗，程序以 **非零 exit code** 結束，整個 workflow 顯示 **Failure**。
- **截圖建議**：進入該次 run → 點 **`build-and-test`** → 展開失敗 step 的 log，保留含 `expected … to be …` 的片段；若有測試結果摘要，可一併截到失敗測試筆數。

### 修正方式

- 將該 `expect` **還原**為正確字串 `'CI/CD Lab Fastify app is running'`。
- 本機執行 `npm run test` 確認通過後再 push；CI 應恢復成功。

---

## 四、繳交前快速檢查清單

- [ ] Fork 的 repo 網址正確、助教可開啟。
- [ ] `ci_{學號}.yaml` 學號與檔名一致。
- [ ] GitHub Actions 有至少一次 **成功** run。
- [ ] 有 **失敗** run 截圖 + 文字說明原因與修正。
- [ ] PDF 檔名：`學號_姓名_CICD_作業.pdf`。
- [ ] 截止：**2026/5/19 0:00** 前上傳至課程指定位置。

---

## 五、可選的 workflow 強化（非必填）

若助教或你本人希望「任意分支 push 都跑」：

```yaml
on:
  push:
```

若 `publish-unit-test-result-action` 在網頁上看不到測試摘要，可在 workflow 頂層加入：

```yaml
permissions:
  checks: write
```

（依組織 / 倉庫預設權限調整；多數個人 fork 預設即可，但若失敗再補。）
