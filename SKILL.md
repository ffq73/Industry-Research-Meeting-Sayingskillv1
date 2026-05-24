---
name: meeting-minutes-generator
description: Generate formal Chinese investment-research meeting minutes (会议纪要) from transcript files or meeting recordings. Use when the user provides audio transcripts (.txt, .pdf, .docx), meeting audio (.m4a, .mp3, .wav), or asks to create Q&A-style corporate meeting minutes for chemical-industry research internship workflows, especially in the Dongwu Securities style.
---

# Meeting Minutes Generator

Create formal Chinese investment-research meeting minutes from source transcripts. Produce an information-dense Q&A document that matches the provided Dongwu Securities chemical-industry examples.

## Core Rules

1. Base the minutes on the provided transcript content. Do not add outside facts to the answers.
2. Use outside search only to verify names, ticker/company spellings, product terms, or industry-event names. Keep verified terms only when they clarify source content.
3. Preserve all numbers, dates, percentages, prices, capacities, and units exactly unless correcting an obvious transcription typo after verification.
4. Replace vague time expressions such as "今年" or "明年" with specific quarters or months only when the transcript context makes the date clear.
5. Do not invent questions. Summarize the real question or discussion topic behind each answer.
6. Favor important, decision-useful information over transcript completeness. Remove filler, greetings, timestamps, speaker labels, repeated confirmations, and low-value chatter.
7. Never mix in information from other companies or prior tasks. Treat each meeting as a closed source set unless the user explicitly provides comparison materials.

## Resources To Load

- Read `references/meeting-minutes-guideline.md` before drafting.
- Read `references/meeting-minutes-example.md` when matching the Q&A style or title/body format.
- Read `references/transcript-cleaning-notes.md` when converting noisy transcription output into final minutes.
- Use `assets/meeting-minutes-template.docx` as the DOCX base whenever the user wants a Word file.

## Workflow

### 1. Prepare Source Text

For transcript files, run the extractor when useful:

```bash
python scripts/extract_transcripts.py input1.txt input2.pdf input3.docx -o combined-transcript.txt
```

For audio files, transcribe with the available transcription tool first, save the transcript, then treat it as source text. If no transcription tool is available, ask the user for a transcript or permission to use a specific service.

When multiple sources are provided, combine them in chronological order when known; otherwise preserve the user-provided order. Keep source labels in the working transcript for traceability.

Before drafting, delete or ignore transcript-service artifacts such as `关键词`, `全文概要`, automatic summaries, speaker timestamps, and repeated `讲话人` labels. These are not meeting evidence unless the user explicitly says to rely on them.

Build a meeting-specific terminology list from the transcript before drafting. For chemical-industry meetings, include company names, subsidiaries, competitors, product names, product grades, raw materials, intermediates, catalysts/additives, equipment, process routes, project names, policies, units, and acronyms. Verify uncertain entries online or against user-provided company materials before finalizing names. Use search for unclear chemical terms, product abbreviations, equipment names, project names, and units; do not use search results to add facts that were not discussed.

Use this general chemical terminology framework:

- **Company/entity**: listed company name, subsidiary, plant/site, customer, supplier, competitor, overseas company, institute, regulator.
- **Product chain**: upstream raw material, intermediate, monomer, polymer/resin/fiber/rubber, formulation, downstream application.
- **Process/equipment**: reactor, separation/purification, polymerization, spinning, chlorination/fluorination/hydrogenation/oxidation, distillation, cracking, slurry/solution/melt process, key device names.
- **Business data**: capacity, output, sales volume, inventory, price/spread, gross margin, operating rate, yield, energy consumption, depreciation, capex, project approval/construction/commissioning status.
- **Units and notation**: 万吨、吨、kg、元/吨、万元/吨、美元/桶、亿元、%、成、D、ppm、kWh、GWh、Q1/Q2/2026Q1.
- **Common confusion types**: one-character Chinese product differences, similar English abbreviations, upstream/downstream direction, domestic/overseas company names, product grade vs product category, capacity vs output vs sales.

### 2. Draft The Minutes

Use this drafting instruction:

```text
仅基于提供的会议录音转文字内容，生成一份会议纪要，问答形式，6000字以内。内容必须详细、全面，不要漏掉任何要点；问题概括要完整；答案要信息密度高、少废话；格式与范例保持一致。
```

Required structure:

```text
YYYYMMDD 公司名称 会议名称

Q：问题或讨论主题概括
A：详细回答。可分段，每段聚焦一个子要点。

Q：问题或讨论主题概括
A：详细回答。
```

Use unnumbered `Q：` / `A：` unless the user asks for numbered Qs. If the transcript is long, first outline all topics, then merge repeated questions into the same Q&A while preserving every substantive point.

Keep high-value content:

- Product demand, pricing, profitability, capacity, utilization, expansion plans, raw materials, customer structure, competitor landscape, process/equipment differences, and management judgement.
- Quantified statements with units and time points.
- Short causal explanations that help interpret operations or earnings.

Remove low-value content:

- Greetings, transitions, acknowledgements, unclear fragments with no recoverable meaning, repeated oral filler, transcript-generated keyword lists, and generic industry background not tied to the meeting.
- External facts that were not discussed in the meeting, even if they are true.

### 3. Verify Content

Before creating the final file, check:

- Every important topic in the transcript appears in the minutes.
- No content from another company, earlier meeting, automatic summary, or model memory appears in the minutes.
- All data has complete units, such as 亿元、万吨、元/吨、美元/桶、%。
- Company names, products, raw materials, equipment names, technologies, and policies are spelled consistently after verification.
- Time references are specific enough for year-boundary contexts.
- Answers contain no unsupported external analysis or subjective commentary.
- Every `Q：` has an `A：`.

### 4. Create DOCX Output

When the user needs a Word document, save the drafted body to a text file and run:

```bash
python scripts/build_minutes_docx.py --title "YYYYMMDD 公司名称 会议名称" --content minutes.txt --output "YYYYMMDD 公司名称 会议名称.docx"
```

The script uses `assets/meeting-minutes-template.docx` by default, applies the title/body format, and sets Chinese text to 楷体_GB2312 with Times New Roman for Latin text where Word supports it.

Pass only the final minutes body to `--content`; do not pass raw Markdown examples, source transcripts, or notes with extra headings unless those headings should appear in the document.

If the template is unavailable, the script creates a plain DOCX and adds a best-effort header watermark text: `内部资料 严禁外传 追究责任`.

Target DOCX format:

- Page margins: top/bottom 2.54cm, left/right 3.17cm.
- Font: Chinese 楷体_GB2312, English/numbers Times New Roman.
- Font size: 10.5pt for title, questions, and body unless the user provides a newer template.
- Title: centered and bold.
- Questions: each `Q：...` paragraph bold.
- Answers and follow-up paragraphs: regular weight.
- Watermark: keep the template watermark `内部资料 严禁外传 追究责任`.

## Final Delivery

Deliver the `.docx` file when requested. Also mention any unresolved uncertainties, such as unclear audio terms, missing units, inaudible passages, or dates that could not be safely normalized.
