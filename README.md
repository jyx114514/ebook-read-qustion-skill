### 推荐用deepseek-v4-flash
### 阶段 0 — 触发
你只需要说一句**触发词**即可，无需记命令：
- 「电子书阅读 / 读电子书 / 总结这本书 / 按章节读PDF / 输出每章思维导图 / 这本书讲了什么 / 关于这本书提问 / 书中图看不清 / 扫描版PDF…」
- 或直接 `@` 引用一个 PDF 文件路径

### 阶段 1 — Step 1·2：提取与验证（两种模式共用）
**Step 1** 运行 `extract_pdf.py`，把整本 PDF 解析成一份 `_pdf_extract.json`（包含 `num_pages`、`is_scanned`、`toc_usable`、`chapters[]`、`pages[]`）。这一份 JSON 全程复用——**总结和问答都读它，不重复解析**。

**Step 2** 校验切分质量：
- ✅ `is_scanned=false` → 正常文本层，继续
- ⚠️ `is_scanned=true` → **扫描版/无文字层** → 走 **Step 2.6**（AI 视觉逐页识读，无需外部 OCR；小书全读，大书先问范围）
- `toc_usable=false` → 书签只有页码，已自动回退标题启发式切分

### 阶段 2 — 模式 A：章节总结（产出 .md 文件）
1. **A1 先跑 `detect_math_code.py`**：把每章里的数学公式、代码示例、推导过程**先定位出来**，防止写总结时漏掉
2. **A2 逐章撰写**：每章输出 ①核心立意 ②关键要点 ③公式/代码/推导（**逐字保留，禁止"推导略"**）④开放问题
3. **每章配 Mermaid 思维导图**（root=章标题，≤2 层 ≤20 节点）
4. **遇到图/公式乱码 → Step 2.5 识图救援**：`export_images.py` 把那一页渲染成 PNG → 用 Read 工具**视觉识读**真实内容 → 引用格式 `（据图2.15，PDF p.59）`
5. 最后合成 `《书名》_章节总结.md`：全书概览 + 全书思维导图 + 目录锚点 + 每章详情

### 阶段 3 — 模式 B：书中问答（严格基于原文）
1. **`search_pdf.py` 关键词定位**：搜出命中章节 + 页码 + 上下文片段
2. **回读 JSON 原文上下文**（片段太短时）
3. **严格按书中内容回答 + 标注章节页码**（如「书中第 3 章…（p.45–46）提到…」）
4. **书中没有的内容明确说"书中没有直接讨论"**，绝不编造；公式/代码同样逐字复现
5. 后续追问**复用同一份 `_pdf_extract.json`**，不重新解析

### 文件构成
| 文件 | 作用 |
|------|------|
| `SKILL.md` | 完整工作流 + 边界情况处理（主文件） |
| `scripts/extract_pdf.py` | 文本/书签/章节切分/扫描件检测 |
| `scripts/search_pdf.py` | 问答定位（关键词 → 章节+页码） |
| `scripts/detect_math_code.py` | 定位公式/代码/推导 |
| `scripts/export_images.py` | 页面→PNG（Step 2.5/2.6 识图核心） |
| `references/qa_grounding.md` | 答案忠实于原书的规范 |

**一句话总结**：**一次提取 → 校验分流（正常文本 / 扫描版视觉读）→ 总结模式（公式先定位、逐字保留、图乱码走识图）或问答模式（搜索定位、原文作答）**，全程复用同一份提取结果。
[离散数学及其应用_第1章总结.md](https://github.com/user-attachments/files/30869362/_.1.md)
<img width="954" height="791" alt="屏幕截图 2026-08-09 155706" src="https://github.com/user-attachments/assets/ef658911-4461-45e9-bc37-08b8fae78fd5" />
<img width="938" height="695" alt="屏幕截图 2026-08-09 155712" src="https://github.com/user-attachments/assets/1dffefe3-efdc-4844-8b03-e069c6af49b6" />



