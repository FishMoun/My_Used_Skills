# Alibaba AI Application Development Interview Transcript Structuring Notes

Session pattern that motivated this reference:

- User provided a Windows folder path containing:
  - `阿里三面文字版.docx` — ASR/text transcript.
  - `430阿里三面录音.m4a` — source recording.
  - `秦岭5.0.pdf` — resume.
- WSL path translation: `D:\研究生\实习准备\录音` → `/mnt/d/研究生/实习准备/录音`.
- `python` was unavailable and `python3` lacked `python-docx`; fallback `.docx` extraction through `zipfile` + `word/document.xml` worked.
- `pdftotext -layout` was available and useful for extracting resume context from the PDF.

Recommended output shape used successfully:

1. Metadata block with source files, duration, date, interview type, target team/role, and ASR correction note.
2. Speaker mapping table (`说话人1` HR, `说话人2` candidate, `说话人3` technical interviewer, others likely noise/misrecognition).
3. Overall timeline table by timestamp and module.
4. Topic sections:
   - 开场与自我介绍
   - 商飞项目
   - 缓存理论与最终一致性
   - 现场 coding：线程安全内存缓存
   - 灵旅 AI 智能体 / RAG
   - Agent / AI Coding 工具
   - 个人标签与复盘能力
   - HR 挫败经历
   - 学生与职场人的区别
   - 岗位与业务理解
   - offer 状态 / 到岗时间 / 意愿度
   - 候选人反问与结束
5. Consolidated question list by theme.
6. Explicit interviewer feedback section.
7. Suggested follow-up documents for deep review.

Common ASR corrections in this session:

- `Note.JS` / `noe JS` → `Node.js`
- `view` → `Vue`
- `radus` / `red` in cache context → `Redis`
- `Jason` → `JSON`
- `IEG` / `IG` / `1G` in context → `RAG` or `Agent`
- `BML5` / `M25` / `编码5` → `BM25`
- `queen问` → `通义千问` / `Qwen`
- `TDL` → `TTL`

Important handling note:

The user wanted the transcript organized first, not an immediate performance critique. For similar tasks, produce the structured record before doing deep diagnosis, unless the user explicitly asks to analyze at the same time.

## Deep diagnosis pattern from the follow-up request

After the factual record, the user asked to combine the original transcript with the structured document and judge improvement points question by question. The useful output was a separate file named `阿里三面_逐题诊断与改进点.md` with these elements:

- Overall diagnosis: the failure pattern was a “credibility chain” break, not one isolated weak answer. Claims in the resume/self-introduction were repeatedly tested by mechanism questions or artifacts.
- For each question: original question, what the candidate answered, current problem, interviewer intent, improvement points, and a suggested answer template.
- High-signal negative moments to diagnose explicitly:
  - `缓存/手搓缓存` claim → hand-write thread-safe TTL/LRU cache without AI; failure hurts project credibility.
  - `RAG/entity expansion/small model` claim → interviewer wanted mechanism, schema, validation, evaluation, not “called a fast API”.
  - `BM25` mention → candidate did not know core sparse-retrieval concept; provide a concise TF/IDF/document-length-normalization answer.
  - `Agent self-evolution/memory/skills` claim → distinguish memory from procedural skills/workflows that affect future behavior.
  - `复盘者` label → interviewer requested actual review artifacts; advise preparing a sanitized review sample.
  - `想来淘天/自营技术` claim → interviewer asked concrete supported business scenarios; answer should start from Taobao/Tmall/Tmall Supermarket/Tmall Global business context before data/AI functions.
- Recommended tone: direct but non-destructive. Translate “humiliation” into repairable categories: knowledge gap, expression gap, preparation/artifact gap, business understanding gap, and interviewer pressure.
- Recommended final action list: hand-code cache, learn BM25/hybrid/RRF/rerank, rewrite RAG pipeline with evaluation, prepare demo/GitHub/artifacts, prepare Taotian business understanding, and train answer rhythm (`先结论 → 分点 → 例子/指标 → 边界`).
