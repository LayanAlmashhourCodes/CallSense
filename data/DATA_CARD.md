# Dataset card: CallSense labeled calls

80 Arabic (Saudi-dialect) customer-service conversations for a telecom company, prepared by the team for this educational project. Each call is labeled with an **intent** and an **escalation** flag, and carries both a human reference transcript and the output of Whisper small.

> **Audio files are not included** in this repository (only transcripts and metadata).

## File

`Whisper_Small_Baseline_Results_with_Resolution.xlsx`

| Sheet | Content |
|---|---|
| `Results` | The 80 calls (main table, read by the notebook) |
| `Summary` | Summary of the Whisper baseline run |
| `شرح الأعمدة` | Column guide (Arabic) |

## Columns of `Results`

| Column | Description |
|---|---|
| `call_id` | `CS001` … `CS080` |
| `audio_filename` | Original recording name (audio not shipped) |
| `reference_transcript` | Human transcript, with `[Agent]` / `[Customer]` turns |
| `whisper_prediction` | Whisper small baseline output (plain text, no speaker tags) |
| `wer` | Word error rate of `whisper_prediction` against the reference, as stored in the file |
| `intent` | `billing`, `technical`, `cancellation`, `complaint`, `information`, `out_of_scope` |
| `escalation` | `1` if the call should be handed to a supervisor, else `0` |
| `status`, `error` | Baseline run status (all `success`) |
| `resolution`, `resolution_status`, `resolution_source`, `review_status` | **Empty by design.** They are filled through the human approval workflow (`approve()` in the notebook), which grows the knowledge base over time |

## Statistics

| Intent | Calls | Escalated |
|---|---:|---:|
| information | 19 | 0 |
| technical | 14 | 6 |
| cancellation | 13 | 6 |
| billing | 12 | 2 |
| complaint | 12 | 6 |
| out_of_scope | 10 | 0 |
| **Total** | **80** | **20** |

- Reference transcripts: median 132 words per call (min 31, max 217).
- Whisper small baseline WER (as stored in `wer`): mean **0.443**, median 0.447, range 0.124 to 1.006. Lowest on `complaint` (0.350), highest on `billing` (0.551).

## Observed escalation patterns

These are patterns seen in the data, not a formal labeling guideline: an issue that was reported before and never solved, a promise or commitment by the company that was not fulfilled, an obviously frustrated customer or a threat to cancel, or a fix that needs authority beyond the agent. In this dataset `information` and `out_of_scope` calls are never escalated.

## Intended use and limitations

- Few-shot examples and retrieval source for the CallSense pipeline, and a small evaluation set.
- **Small** (about 12 calls per intent) and from a single source, so it does not cover the variety of real call centers. Metrics measured on it can be optimistic (see `baseline_classifier/`).
- Do **not** add real customer data. Before publishing we scanned the repository for emails, phone numbers and access tokens and found none.
