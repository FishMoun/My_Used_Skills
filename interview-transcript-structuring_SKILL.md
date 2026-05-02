---
name: interview-transcript-structuring
description: Convert interview recordings/transcripts plus resumes or JDs into structured Markdown records for later deep interview review, especially Chinese technical/HR interviews.
tags:
  - interview
  - transcript
  - markdown
  - career
  - chinese
---

# Interview Transcript Structuring

Use this skill when the user asks to organize an interview recording transcript, meeting transcript, or interview notes into a structured Markdown document, especially when they provide a folder containing a transcript (`.docx`, `.txt`, etc.), recording, resume, JD, or related materials.

## Goals

Produce a clean, factual, navigable Markdown record that can serve as the basis for later deep review. Do **not** jump straight into judging the candidate unless the user asks for analysis; first preserve and structure what happened.

When the user later asks for deep review, switch from factual structuring to **question-by-question diagnosis** grounded in the original transcript: what they answered, where it missed the interviewer’s intent, what capability was being tested, concrete improvement points, and rewritten answer templates.

## Workflow

1. **Locate source files.**
   - Translate Windows paths to WSL paths when needed, e.g. `D:\研究生\实习准备\录音` → `/mnt/d/研究生/实习准备/录音`.
   - List the folder and identify transcript, recording, resume, JD, and any existing notes.
2. **Extract transcript text.**
   - For `.txt` / `.md`, read directly.
   - For `.docx`, prefer `python-docx` if available; if not installed, extract `word/document.xml` via `zipfile` and parse `<w:t>` text. Preserve speaker/timestamp markers.
   - Save a raw extracted `.txt` next to the source for auditability, e.g. `*_原始提取.txt`.
3. **Use resume/JD as context, not as a replacement for the transcript.**
   - If a resume PDF is present and `pdftotext` is available, extract enough to confirm project names, technologies, dates, and role names.
   - Mark uncertain transcript corrections as “疑似转写错误” rather than silently rewriting meaning.
4. **Create the structured Markdown.**
   Include, where applicable:
   - metadata block: source files, duration, date/time, interview type, role/team;
   - speaker-role mapping;
   - overall timeline table;
   - sections by phase/topic with timestamps;
   - interviewer questions and candidate answer key points;
   - coding/task requirements and observed process;
   - explicit interviewer feedback/评价性表达;
   - consolidated question list by theme;
   - suggested follow-up documents for deeper review.
5. **Keep it factual.**
   - Lightly correct obvious ASR errors (`view`→`Vue`, `radus`→`Redis`, `Note.JS`→`Node.js`) only when context is clear.
   - Record sensitive or emotionally charged moments neutrally at this stage.
   - Avoid over-interpreting motives or outcomes in the structure document.
6. **Write and verify.**
   - Save as a new `.md` in the same folder with a clear name such as `公司轮次面试记录_结构化整理.md`.
   - Verify the file exists and has substantial line count/content before reporting completion.

## Deep review workflow

Use this after the structured record exists, or when the user explicitly asks to diagnose/improve the answers.

1. **Re-read the original transcript, not only the structured summary.** The wording, interruptions, hesitation markers, and interviewer redirects often reveal the actual failure mode.
2. **Work question by question.** For each interview question include:
   - original question or close quote;
   - what the candidate answered;
   - current diagnosis (knowledge gap, expression issue, preparation gap, evidence/demo gap, or interviewer-pressure/experience issue);
   - what the interviewer was likely testing;
   - concrete improvement points;
   - a rewritten answer or answer framework when useful.
3. **Preserve emotional safety while staying direct.** Acknowledge that a painful interview can feel humiliating, but translate it into repairable categories rather than global self-judgment.
4. **Identify cross-cutting patterns.** Common patterns include credibility-chain breaks (`I used X` → asked to hand-code/explain/demo X), vague business understanding, unsupported metrics, and inability to show artifacts.
5. **Save the diagnosis separately** from the factual record, e.g. `公司轮次_逐题诊断与改进点.md`, and verify line count/content.

## Useful extraction snippets

### `.docx` fallback without `python-docx`

```python
import zipfile, re, html
from pathlib import Path

p = Path('/mnt/d/.../transcript.docx')
with zipfile.ZipFile(p) as z:
    xml = z.read('word/document.xml').decode('utf-8')
paras = []
for para in re.findall(r'<w:p[\s\S]*?</w:p>', xml):
    para = re.sub(r'<w:br\s*/?>', '\n', para)
    texts = re.findall(r'<w:t[^>]*>([\s\S]*?)</w:t>', para)
    line = ''.join(html.unescape(t) for t in texts)
    if line.strip():
        paras.append(line)
text = '\n'.join(paras)
```

### Resume/JD PDF context

```bash
pdftotext -layout '/mnt/d/.../resume.pdf' - | sed -n '1,160p'
```

## Output template

```markdown
# <公司/岗位/轮次>面试记录（结构化整理）

> 来源文件：`...`  
> 辅助参考：`...`  
> 面试时长：...  
> 面试时间：...  
> 面试类型：...  
> 整理说明：本文对原始转写进行结构化归档和轻度错字校正，不做深入评价。

## 0. 说话人标注

| 说话人 | 初步角色判断 | 说明 |
|---|---|---|

## 1. 面试整体时间线

| 时间段 | 模块 | 主要内容 |
|---|---|---|

## 2. <阶段/主题>

### 2.1 <问题或子主题>

**时间：...**

**面试官问题：** ...

**候选人回答要点：**

- ...

## N. 本次面试问题清单（按主题归档）

## N+1. 面试官明确给出的反馈 / 评价性表达

## N+2. 后续复盘建议的材料索引
```

## References

- `references/alibaba-ai-interview-structuring.md` — concrete session notes for structuring and then deeply diagnosing a Chinese Alibaba AI application development interview transcript from `.docx` plus resume PDF in WSL.

## Pitfalls

- Do not treat an ASR transcript as perfectly reliable; flag likely recognition errors.
- Do not let resume content override what was actually said in the interview.
- Do not skip raw extraction output; it makes later correction and deeper review much easier.
- For emotionally difficult interviews, structure first, then analyze. This helps separate facts, candidate performance, and interviewer behavior.
- In deep review, do not diagnose from the polished structured document alone; compare against the raw transcript so you catch hesitation, interviewer redirects, failed demos, and exact feedback.
- If the candidate made a claim that invites proof (`cache`, `RAG`, `Agent`, `skills`, `review habit`, `business interest`), check whether the interview later demanded proof via hand-coding, mechanism explanation, demo, GitHub, or artifacts.
