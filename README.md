# AI Candidate Screening & Shortlisting System

**A deployable n8n template for BPOs, recruiters and staffing agencies.**
Candidate applies → AI scores the PDF resume against your rubric, quoting the evidence → the recruiter decides Advance / Hold / Reject in a Google Sheet → strong candidates get a secure Stage 2 questionnaire → the recruiter requests an interview → the candidate books a free slot in Google Calendar with a Google Meet link. Every score, email and decision is logged.

**Design rule: the AI reads and scores, code checks, a person decides.** The AI never rejects anyone.

▶ **Case study page:** open [`index.html`](index.html) (or the GitHub Pages link, once enabled) to replay one candidate from application to booked interview.
📄 **Docs:** [Case Study & Runbook](docs/Case-Study-and-Runbook-AI-Candidate-Screening.pdf) · [System Documentation (public edition)](docs/System-Documentation-AI-Candidate-Screening.pdf) · [Owner SOP](docs/Owner-SOP-AI-Candidate-Screening.pdf) · [Setup Guide](docs/Setup-Guide-AI-Candidate-Screening.pdf)

> Demo employer: Halcyon Bay Contact Solutions (fictional), hiring a Customer Service Representative for an inbound telco voice account. All candidates, resumes and emails are test data, in line with the Data Privacy Act of 2012 (RA 10173).

---

## Results

| | |
|---|---|
| Tests passed on the final build | **41 / 41** (core screening 13, phase tests 16, final regression 12; 2 Oct 2026) |
| Application to acknowledgment email | **~20 seconds** (measured) |
| AI cost per candidate | **~US$0.01 per resume, ~US$0.02 per questionnaire** (estimate, GPT-4.1 via OpenRouter) |
| Recruiter time at 200 applications/month | **~38 h → ~5 h per month** (estimate; assumptions in the Case Study) |
| Application to booked interview | **~5–7 working days → ~1 day** (estimate) |

## How it works

```
Lane 1  Intake        Form (4 Yes/No knockouts + PDF) → Drive → AI scores rubric with quotes
                      → code: evidence check, weighted score, band, duplicates → Sheet + emails
        RECRUITER     reads evidence in the Sheet → Advance / Hold / Reject
Lane 2  Decisions     every 2 min: undo window → templated email → Stage 2 link on Advance
Lane 3  Stage 2       secure link → AI scores answers + claim check → combined score (60/40)
        RECRUITER     reads Stage 2 summary → Schedule Interview
Lane 5  Booking       secure link → only free calendar slots → re-check → Calendar event + Meet
Lane 4  Error alerts  watches every lane: 3 retries, then alert email + ERROR row
```

| Check | Rule |
|---|---|
| Knockouts | Candidate's own Yes/No answers; a "No" is flagged for review, never rejected |
| AI output | Temperature 0, JSON only, every score must quote the resume or answer |
| Evidence | Code confirms each quote exists; unmatched quotes get "please verify" |
| Years of experience | Taken only from dates on the resume; missing dates reported as missing |
| Fairness | Name, age, sex, civil status, religion, ethnicity and location ignored |
| Duplicates | Same email within 90 days is linked to the first application, not re-scored |
| Undo window | Decision emails wait 10 min (Config); release time is locked when first seen |
| Claim check | Stage 2 claims the resume does not support are flagged; scores never change |
| Secure links | Random 40-character token, single use, expires (7 days Stage 2, 5 days booking) |
| Errors | Retries ×3, then an alert email naming the failed step and an ERROR row |

## Test cases (final regression)

| # | Scenario | Expected → Actual |
|---|---|---|
| R-01 | Strong candidate | 100, Shortlist ✅ |
| R-02 | Weak candidate | 13, Low match, still sent to recruiter ✅ |
| R-03 | Knockout "No" | 86, Shortlist + KNOCKOUT FAIL flag, not rejected ✅ |
| R-04 | Same email re-applies | Linked to first application, not re-scored ✅ |
| R-05 | Advance | Email with Stage 2 link ✅ |
| R-06 | Reject | Polite email, no scores or AI mentioned ✅ |
| R-07 | Hold | No email ✅ |
| R-08 | Undo window, Config changed mid-window | No early send; cleared = cancelled ✅ (after fix) |
| R-09 | Stage 2, honest answers | 100 / combined 100, 0 claim flags ✅ (after fix) |
| R-10 | Interview booking | Booked; another candidate's slot + buffer hidden ✅ |
| R-11 | Reused or tampered links | "Already submitted" / "Not valid" / "Already booked" ✅ |
| R-12 | Real failure | Alert email + Audit Log ERROR row ✅ |

Full results: [`workflow/Test-Results-v1.0.xlsx`](workflow/Test-Results-v1.0.xlsx). Test resumes: [`test-resumes/`](test-resumes/).

## Repository

| Folder | Contents |
|---|---|
| `index.html` | Case study page with an interactive replay and the screenshots |
| `docs/` | Case Study & Runbook, System Documentation, Owner SOP, Setup Guide (PDF) |
| `screenshots/` | S1–S14 from the test runs, plus LinkedIn and Upwork cover images |
| `test-resumes/` | 12 fictional PDF resumes (strong, average, weak, knockout, duplicate, missing info) |
| `workflow/` | n8n workflow JSON (credentials removed), Google Sheet template, test results |

## Stack

n8n (self-hosted) · n8n Forms · OpenRouter (GPT-4.1; GPT-4.1 mini as low-cost option) · Google Sheets · Google Drive · Gmail · Google Calendar

## Roadmap

- **v1.3** Interview rescheduling and reminders
- **v2.0** Several jobs at once, scanned-resume OCR, redaction before AI, retention auto-delete, recruiter dashboard

---

<sub>Prepared by Khael Mendoza · mjmendoza.workph@gmail.com</sub>
