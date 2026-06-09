# @plotdb/quill

Fork 自 [slab/quill](https://github.com/slab/quill)，修正上游未處理的安全性漏洞。

## 與上游的差異

### XSS 修正（基於 slab/quill@2.0.3）

**參考：** [Fluid Attacks Advisory - Diomedes](https://fluidattacks.com/advisories/diomedes)

`getSemanticHTML()` 輸出路徑中，三個模組的 `html()` 方法將使用者可控的值直接插入 HTML template literal，未經 HTML 跳脫，導致 XSS 漏洞。

| 檔案 | 問題 | 攻擊方式 |
|---|---|---|
| `formats/video.ts` | video URL 直接插入 `href` 屬性與連結文字 | 含特殊字元的 URL 可注入額外屬性或 HTML |
| `formats/formula.ts` | formula 字串直接插入 `<span>` 內容 | 注入任意 HTML 標籤 |
| `modules/syntax.ts` | language 值直接插入 `data-language` 屬性 | 透過 `"` 跳脫屬性，注入事件處理器 |

**修法：** 對三個值均套用既有的 `escapeText()` 函式（轉義 `&`, `<`, `>`, `"`, `'`）。

**影響範圍：** 僅在呼叫 `getSemanticHTML()` 時觸發（包含使用者 Ctrl+C 複製內容時）。`getContents()` / `setContents()` 的儲存與渲染流程不受影響。

## 版本策略

版本號獨立於上游，從 `1.0.0` 開始。對應的上游版本記錄在 `package.json` 的 `upstream` 欄位。

| @plotdb/quill | 上游 base |
|---|---|
| 1.0.0 | slab/quill@2.0.3 |

## Release 流程

```bash
# 1. 同步上游（有新版本時）
git fetch upstream
git merge upstream/main
# 更新 package.json 的 upstream 欄位

# 2. 改版號
vi packages/quill/package.json   # 修改 version

# 3. Build
npm run build:quill

# 4. Commit + Tag
git add packages/quill/package.json
git commit -m "Release x.y.z"
git tag vx.y.z

# 5. Publish
cd packages/quill/dist
npm publish --access public --otp=<otp>

# 6. Push
cd ../../..
git push origin main --tags
```

## 安全性監控

上游漏洞通報來源：
- [slab/quill GitHub Security Advisories](https://github.com/slab/quill/security/advisories)
- Watch slab/quill → Custom → Security alerts

有新漏洞時：確認是否影響本 fork → 修補 → 發新版。
