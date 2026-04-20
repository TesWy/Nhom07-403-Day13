# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Team Metadata
- [GROUP_NAME]: 07
- [REPO_URL]: https://github.com/TesWy/Nhom07-403-Day13
- [MEMBERS]:
  - Member A: Huỳnh Lê Xuân Ánh | Role: Logging & PII
  - Member B: Huỳnh Nhựt Huy | Role: Tracing & Enrichment
  - Member C: Nguyễn Ngọc Khánh Duy | Role: SLO & Alerts
  - Member D: Nguyễn Ngọc Hưng | Role: Load Test & Dashboard
  - Member E: Huỳnh Khải Huy | Role: Demo & Report

---

## 2. Group Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 100/100
- [TOTAL_TRACES_COUNT]: 197 (Langfuse dashboard; 175 llm-generation + 22 chat-response spans)
- [PII_LEAKS_FOUND]: 0

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: docs/evidence/logs-correlation-id.png
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: docs/evidence/logs-pii-redaction.png
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: docs/evidence/langfuse/1.png
- [TRACE_WATERFALL_EXPLANATION]: Langfuse span detail for `llm-generation`: latency 0.15s, model `claude-sonnet-4-5`, 37 prompt + 178 completion tokens ($0.002781). Input query contains `[REDACTED_CREDIT_CARD]` confirming PII scrubbing reaches the trace layer. Metadata includes `feature: qa`, `query_preview`, and `doc_count: 1`. Under `rag_slow` incident, P99 spiked to 2.66s (docs/evidence/langfuse/4.png), confirming RAG retrieval as bottleneck.

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: docs/evidence/dashboard-layer2-6-panels.png
- [SLO_TABLE]:
| SLI | Target | Window | Current Value |
|---|---:|---|---:|
| Latency P95 | < 3000ms | 28d | 150ms (dashboard live snapshot) |
| Error Rate | < 2% | 28d | 0.00% (dashboard live snapshot) |
| Cost Budget | < $2.5/day | 1d | $0.0820 total observed; forecast 1h = $0.5467 |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: docs/evidence/alerts-rules-with-runbook.png
- [SAMPLE_RUNBOOK_LINK]: docs/alerts.md#1-high-latency-p95

---

## 4. Incident Response (Group)
- [SCENARIO_NAME]: rag_slow
- [SYMPTOMS_OBSERVED]: Under `rag_slow` incident with concurrency 5, P95 latency spiked to **2,651ms** (P50=2,650ms, P99=2,651ms) — exceeding the `high_latency_p95` alert threshold of 1,500ms. Error rate remained 0.00% and cost forecast stayed at $1.52/hr, isolating latency as the sole symptom.
- [ROOT_CAUSE_PROVED_BY]: Incident toggle response confirms `rag_slow = True` (`python scripts/inject_incident.py --scenario rag_slow` -> `200 {'ok': True, 'incidents': {'rag_slow': True, ...}}`), and retrieval path in `app/mock_rag.py` applies delay when `STATE["rag_slow"]` is enabled.
- [FIX_ACTION]: Corrected incident command path to `python scripts/inject_incident.py --scenario rag_slow`, enabled scenario successfully, and used dashboard + load runs to observe impact.
- [PREVENTIVE_MEASURE]: Add command checklist for incident scripts, keep a runbook mapping scenario -> expected symptom, and define latency alert thresholds with immediate rollback/disable flow.


---

## 5. Individual Contributions & Evidence

### Huỳnh Lê Xuân Ánh
- [TASKS_COMPLETED]: Implemented PII handling module (`app/pii.py`) used for redaction in the observability pipeline (Logging & PII scope).
- [EVIDENCE_LINK]: https://github.com/TesWy/Nhom07-403-Day13/commit/d14086f27b1efb807b8ad576cf219005abcb7375

### Huỳnh Nhựt Huy
- [TASKS_COMPLETED]: Updated runtime log dataset (`data/logs.jsonl`) for demo/validation flow and dashboard evidence preparation.
- [EVIDENCE_LINK]: https://github.com/TesWy/Nhom07-403-Day13/commit/0595dd6c900e374875dda81e7c8cf3e36fe878e5

### Nguyễn Ngọc Khánh Duy
- [TASKS_COMPLETED]: Enriched logs with request context and improved correlation ID handling in middleware/application flow (`app/main.py`, `app/middleware.py`).
- [EVIDENCE_LINK]: https://github.com/TesWy/Nhom07-403-Day13/commit/3f23a50b08f091b3a75c6f2cf45c46728e7eddef

### Nguyễn Ngọc Hưng
- [TASKS_COMPLETED]: Designed and implemented dashboard artifacts plus alert rules/runbook integration (`dashboard.html`, `docs/dashboard-spec.md`, `config/alert_rules.yaml`, `docs/alerts.md`, `app/main.py`).
- [EVIDENCE_LINK]: https://github.com/TesWy/Nhom07-403-Day13/commit/4009bdbd9be8ee36badcbeebda537a20c43449cd ; https://github.com/TesWy/Nhom07-403-Day13/commit/d3924c04fee5a77a9a789fbb60c9e00299fa0e4e ; https://github.com/TesWy/Nhom07-403-Day13/commit/c023a77e8a14f8931efe3f4bddf4203800ebd771 ; https://github.com/TesWy/Nhom07-403-Day13/commit/438356047184fb963d36e2baf6feea671a064184

### Huỳnh Khải Huy
- [TASKS_COMPLETED]: Fixed and refined dashboard presentation, added/organized evidence artifacts, and completed group final report content for submission.
- [EVIDENCE_LINK]: https://github.com/TesWy/Nhom07-403-Day13/commit/9327948cb590f13095817576ed732908211c6f77 ; https://github.com/TesWy/Nhom07-403-Day13/commit/3cc0ed42a8d4143d0c80dc5399bf5e88793b0292

---

## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: (Description + Evidence)
- [BONUS_AUDIT_LOGS]: (Description + Evidence)
- [BONUS_CUSTOM_METRIC]: (Description + Evidence)
