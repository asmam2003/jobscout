# JobScout

An LLM-powered job triage system that scores 50+ daily job listings against canonical resume profiles and surfaces a ranked queue with gap analysis. Built with Flask, the Anthropic API, and Claude as a coding partner.

## Problem

Job aggregators return dozens of new listings per day across the roles I was targeting, and none of them rank postings by actual fit to a candidate's background. Manually reading each posting to decide whether to apply was consuming 2-3 hours every morning, with most of that time spent on listings I would ultimately skip. The bottleneck was triage, not application quality.

## What JobScout does

JobScout pulls job listings from public APIs, applies fast keyword filters to cut obvious mismatches, fetches the full posting from the source URL to avoid truncated descriptions, then scores each survivor against my canonical resume profiles using the Anthropic API. The result is a ranked queue with:

- A fit score from 0 to 10
- The best-matching profile from my library
- Three specific gaps between the posting and my background
- Three verbatim phrases from the posting not addressed by my current resume

The interface allows filtering by track, setting a minimum score, marking listings as applied, or dismissing bad fits.

A live snapshot from the deployed app showing 62 listings filtered to score ≥ 4 across all tracks:

![JobScout interface](jobscout_screenshot.png)

The full 23-page output is in `JobScout.pdf` in this repo.

## AI tools used

**Anthropic API (Claude)** is the scoring engine. Each posting goes to Claude with a structured prompt against my canonical resume profiles. Claude returns scored JSON: a fit score, the best-matching track, specific gaps, and verbatim JD phrases not addressed by the resume. The JSON schema is enforced so downstream filtering and ranking are deterministic rather than reliant on parsing freeform text.

**Claude as a coding partner** during development. I designed the system architecture, the prompt structure, and the decision to enforce structured JSON output. The code implementation was a collaboration with Claude through Claude.ai.

## Two design decisions worth flagging

**Structured output over freeform text.** Early prototypes had Claude return narrative recommendations. I switched to enforced JSON because freeform output cannot be filtered, sorted, or quantified. Structured output turned the LLM from a recommendation tool into a deterministic scoring layer.

**Full-JD fetch before scoring.** The Adzuna API truncates job descriptions mid-sentence. Claude was scoring against incomplete information, producing inflated fit scores. I redesigned the pipeline to fetch full postings from the source URL before scoring. The fix was at the data-quality layer, not the prompt layer.

## Tech stack

- Flask (web framework)
- PostgreSQL via SQLAlchemy (storage)
- Anthropic API (scoring)
- Render (deployment with twice-daily scheduled cron)
- Adzuna and Greenhouse APIs (listing sources)

## Outcome

JobScout runs daily without intervention. The triage queue takes 20-30 minutes to review each morning, down from 2-3 hours of unstructured screening. Applications go out to listings the system identified as fit score ≥ 7 and where the gap analysis showed addressable resume tailoring rather than fundamental mismatches.
