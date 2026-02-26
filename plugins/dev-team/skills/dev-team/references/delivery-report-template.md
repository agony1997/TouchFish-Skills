# Delivery Report

> Project: {name} | Date: {date}
> Based on: {list of source spec paths}

## Summary

{2-3 sentences describing what was delivered}

## Implemented Items

### Backend

- [x] T-{ID}: {description} — {files changed}

### Frontend

- [x] T-{ID}: {description} — {files changed}

## QA Results

| Task | Result | Rounds | Notes |
|------|--------|--------|-------|

## Agent Metrics

### Team Composition

| Agent | Model | Role | Tasks Completed |
|-------|-------|------|-----------------|
| TL | opus | Planner + QA gate | — |
| challenger | sonnet | Devil's advocate | — |
| {worker} | {model} | {role} | {count} |

### Resource Usage

| Agent | Tokens (in/out) | Duration | Est. Cost (USD) | Source |
|-------|-----------------|----------|-----------------|--------|
| TL | — | {duration} | — | tracked |
| challenger | — | {duration} | — | tracked |
| {worker} | — | {duration} | — | tracked |
| QA ({n} reviews) | {in} / {out} | {duration} | ${cost} | exact |
| **Total** | | **{total}** | **~${total}** | |

<!-- Source: "exact" = from Task tool usage return, "tracked" = TL timestamp tracking -->

### Cost Breakdown

| Role | Count | Est. Cost | % of Total |
|------|-------|-----------|------------|
| TL (Opus) | 1 | ${cost} | {pct}% |
| Workers (Sonnet) | {n} | ${cost} | {pct}% |
| Challenger (Sonnet) | 1 | ${cost} | {pct}% |
| QA Sub-agents (Sonnet) | {n} | ${cost} | {pct}% |

<!-- Pricing: Opus in=$15/MTok out=$75/MTok | Sonnet in=$3/MTok out=$15/MTok | Haiku in=$0.80/MTok out=$4/MTok -->

## Contract Verification

- Backend <-> Contract: {pass/fail}
- Frontend <-> Contract: {pass/fail}

## Deferred / Manual Items

- [ ] {item} — {reason}

## Issues Summary

- Total: 0 | Resolved: 0 | Deferred: 0
- Details: see {date}-ISSUES.md

## Cross-references

- {date}-TRACE.md | {date}-PROCESS_LOG.md | {date}-ISSUES.md | {date}-API_CONTRACT.md
