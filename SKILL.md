---
name: paper-note-extractor
description: This skill should be used when the user wants to turn a physics/physical-science paper (PDF file, arXiv link, or DOI) into structured close-reading notes, or needs to digest a paper before a group meeting / course presentation. Triggers include phrases like "帮我把这篇论文整理成精读笔记", "读一下这篇 arXiv 文章，提取核心方程和物理图像", "我要在组会上讲这篇 paper，帮我做阅读笔记并列出可能被问到的问题". The skill extracts the physical picture, core equations with symbol-by-symbol explanations, key data, method limitations, and discussion-ready questions.
agent_created: true
---

# Paper Note Extractor（论文要点提取）

## Overview

Convert one physics paper into a structured close-reading note so the reader grasps the paper's physical picture, core equations, key data, and limitations without re-reading the original. Notes are written in Markdown with LaTeX equations and always end with a question list suitable for group-meeting discussion or further exploration.

## When to Use

- User provides a paper PDF path, arXiv URL, or DOI and asks for reading notes / a summary / close-reading extraction.
- User is preparing a group meeting (组会) or course presentation about a specific paper.
- Upstream skill "文献检索去噪" hands over a paper that must be digested.

Do not use for: general literature search (use the retrieval skill), solving homework problems, or summarizing non-physics business documents.

## Input Parameters

| Parameter | Type | Required | Meaning |
|-----------|------|----------|---------|
| paper_file | file path / URL | Yes | PDF file path, arXiv link, or DOI link |
| focus | text | No | Reading purpose or focus, e.g. "为组会汇报做准备", "关注数值方法部分", "与我的课题做关联". Default: general close-reading mode |
| depth | enum: quick / full | No | quick = speed-read version (≤3 sentences per section); full = complete version. Default: full |

## Execution Flow

1. **Acquire and parse the paper**: read title, abstract, main text, equations, figures, and references. If input is a URL, download first. If the PDF cannot be parsed (unsearchable scan, failed formula OCR), report the failure reason and the estimated proportion of unreadable content, then stop — never fabricate a note from fragmentary content.
2. **Classify the paper type**: theory derivation / experimental measurement / numerical simulation / review. Adjust emphasis accordingly (theory → derivation assumptions; experiment → error sources and apparatus; simulation → convergence and model validity; review → classification framework and open problems).
3. **Extract the physical picture and core equations**: describe the physical story with one everyday analogy plus one rigorous statement. For each core equation, explain every symbol's physical meaning, the equation's validity conditions, and its approximation assumptions. Transcribe equations exactly as in the paper; any simplification or symbol change must be declared explicitly.
4. **Extract key data**: important numerical results, measured values, fitted parameters, and error ranges, each annotated with the figure/table/section it comes from.
5. **Critically analyze limitations**: separate what the authors admit from what they omit — approximation validity range, experimental error sources, numerical convergence, sample size, unstated assumptions. Write only concrete, checkable criticisms.
6. **Generate questions in two categories**: questions to ask the authors (method details, robustness) and questions for further exploration (connections to related work, generalization). At least 3 each. If `focus` was given, tailor the questions to that purpose. If focus mentions 组会/汇报, additionally append a "汇报逻辑建议" section suggesting how to present the paper in ~10 minutes.
7. **Assemble and output** the note in the fixed six-part structure below.

## Output Format

Markdown document, with `##`/`###` headings, equations in LaTeX (`$...$` inline, `$$...$$` display). Fixed structure:

1. **一句话总结** — one sentence: what the paper did and found.
2. **物理图像** — 通俗类比 + 严格表述。
3. **核心方程与各项含义** — each equation with a symbol table and validity conditions. Example (excerpt):

   > ### 3. 核心方程
   > $$\chi(T) = \chi_0 + \frac{C}{T - T_c}$$
   > - $\chi$：磁化率，实验中通过 SQUID 测量得到
   > - $C$：Curie 常数，与有效磁矩平方成正比
   > - $T_c$：临界温度，拟合得 92.1 ± 0.3 K（图 3b）
   > - 适用条件：仅低于 $T_c$ 的顺磁区，忽略抗磁背景

4. **关键数据** — table with columns: 数据项 / 数值 / 来源图表 / 物理意义。
5. **方法局限** — two lists: 作者自认的局限；批判性补充的局限。
6. **可追问的问题** — two lists: 向作者提问类；延伸探究类（各不少于 3 个）。

Length limit: ≤1500 Chinese characters per paper. For very long papers, prioritize quality of sections 1–4. `depth=quick` compresses every section to ≤3 sentences.

## Constraints and Rules

1. Never fabricate content not present in the paper. Any inference must be tagged `[推断]` with its basis stated.
2. Equation transcription must match the original; do not silently simplify or rename symbols.
3. If parsing fails (OCR failure, unreadable formulas), state the failure reason and proportion explicitly; do not produce a note from incomplete content.
4. No generic praise ("这是一篇重要的工作"). Criticism must be concrete and verifiable.
5. For preprints, note at the top: "预印本，内容未经同行评审".
6. Write notes in the user's language (default Simplified Chinese); keep technical terms and equation symbols in their original form.

## Dependencies

- Tools: web search/fetch (to download arXiv/DOI papers), file reading (PDF parsing + OCR), LaTeX rendering in output.
- Related skills: optional chaining with "文献检索去噪" (upstream: its literature list feeds this skill) and "笔记体系重构" (downstream: archive notes into the knowledge map). No hard dependency.
