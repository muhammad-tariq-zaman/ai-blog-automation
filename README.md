# Case Study: An Automated AI-Tools Blog (5 Articles a Day, $0 Budget)

*A side project by Muhammad Tariq Zaman — IT support executive by day, automation tinkerer by night.*

## What this is

I run a Blogger blog that reviews newly launched AI tools. The entire pipeline — finding a tool, researching it, writing a 1,000–1,500 word review, generating its featured image, quality-checking it, and publishing it — runs **automatically, five times a day, on a $0 budget**, including while my laptop is switched off.

This repo is the **case study and sanitized documentation** of that system. The live pipeline code stays private (it is tied to my own accounts), but the architecture, the workflow, the failures, and the lessons are all here — because the failures turned out to be the most valuable part.

## Why I built it

1. **Skills:** I wanted hands-on practice with automation, scheduling, content pipelines, and API thinking — the same skills I'm building for IT support work (PowerShell, Python).
2. **A real system, not a tutorial project.** A blog that must publish 5 times a day, every day, forces real engineering habits: state tracking, dedup, QC gates, failure handling, and logs you actually read.
3. **Zero budget.** Blogger is free, scheduling is free, images and writing are generated in-house. If it needs money, it doesn't ship.

## The pipeline

```
 ┌────────────┐   ┌──────────────┐   ┌───────────────┐   ┌────────────┐
 │  DISCOVER  │ → │   RESEARCH   │ → │     WRITE     │ → │  QC GATE   │
 │ futuretools│   │ official site│   │ template +    │   │ checklist: │
 │ .io newest │   │ + 2 indepen- │   │ SEO rules     │   │ facts, SEO,│
 │ tools      │   │ dent sources │   │ (1,000–1,500w)│   │ no copying │
 └────────────┘   └──────────────┘   └───────────────┘   └─────┬──────┘
                                                             ↓
 ┌────────────┐   ┌──────────────┐   ┌───────────────┐   ┌────────────┐
 │   RECORD   │ ← │    IMAGE     │ ← │    PUBLISH    │ ← │  (pass)    │
 │ covered-   │   │ 16:9 thematic│   │ Blogger       │   │            │
 │ tools.json │   │ art, no fake │   │ (see "Status" │   │            │
 │ + run log  │   │ screenshots  │   │  below!)      │   │            │
 └────────────┘   └──────────────┘   └───────────────┘   └────────────┘
```

**Schedule:** 5 runs/day — 09:00, 12:30, 16:00, 19:30, 23:00 (PKT), server-side cron.

### Key components

- **Discovery with memory.** Each run picks the newest tool from futuretools.io that is *not* already in `covered-tools.json`. That state file is the pipeline's memory: even across restarts, a tool is never written about twice. (~33 tools covered in the first week.)
- **Research rule: 3 sources minimum.** The tool's official site for facts (features, pricing), plus at least two independent sources for confirmation and criticism. If a claim can't be verified twice, it's phrased carefully ("the site claims…") or dropped. Nothing is copied — every article is synthesized fresh.
- **Template + SEO checklist.** Articles are built from an HTML template with enforced SEO: keyword-led title ≤60 chars, meta-length first paragraph, keyword placement in the first 100 words, H2/H3 structure, alt text, and a CTA button (with an affiliate-link substitution map for the day I join vendor programs).
- **A real QC gate.** A written checklist (facts sourced? no copied sentences? no fake "we tested it" claims? image not a fake screenshot?) must pass before anything publishes. Articles are framed honestly as first-look overviews from public information — never as hands-on testing that didn't happen.
- **Featured images.** Generated 16:9 thematic art per category (video tools, coding agents, audio…) — deliberately *not* fake UI screenshots, because inventing a product's interface would be dishonest to readers.
- **Run log.** One line per run: timestamp, tool, title, result. When something fails, the log is where the story is.

## Status — the honest part

**The publishing route broke, and the blog side is currently backed up.**

- The original design posted through Blogger's **Mail-to-Blogger** feature (email the article to a secret address; Blogger publishes it).
- On **15 Oct 2026** Blogger's mail servers started **hard-rejecting** those emails (bounces at 09:00, 12:30 and 16:00). Worse, emails Gmail reported as "sent" in the days around the failure didn't all appear on the blog — lesson one: *a sent receipt is not a published post. Verify at the destination.*
- The pipeline now runs in **staging mode**: articles and images are still produced 5×/day and saved locally with the run logged as "staged for resend", and covered-tools state treats staged slugs as covered so nothing is ever duplicated. Probing the email route to "test if it's back" is banned — every probe just lands another bounce in my inbox.
- **Plan B (built, awaiting deployment):** a Google Apps Script web app that receives the article over HTTPS and publishes it through the **official Blogger API** — no email involved. It includes a non-publishing `ping` self-test so the token, blog ID, and API connection can be verified *before* the first real post, and a resend queue that republishes the staged articles oldest-first with publish-verification and two retries.

## Lessons learned (the real output of this project)

1. **Verify outcomes where they land, not where you sent them.** "Sent" ≠ "published", "backup completed" ≠ "restore works". This applies to IT support exactly as much as to blogging.
2. **Design for the failure of your favourite dependency.** Mail-to-Blogger was simple and free — and a single point of failure. The recovery (API route) should have existed from day one.
3. **State files are unglamorous and essential.** One JSON file of covered tools prevents the entire class of "duplicate content" bugs, across crashes and restarts.
4. **QC checklists beat good intentions.** At 5 posts a day, nobody "just remembers" to check facts. A written gate does.
5. **Never send probe traffic into a system that's already bouncing.** Test harnesses (the `ping` action) exist so production doesn't become the test.
6. **Honesty scales better than hype.** No fake hands-on claims, no copied blurbs, no fake screenshots — slower, but the blog can survive any reader checking.

## Roadmap

- [ ] Deploy Plan B (Apps Script + Blogger API) and clear the staged backlog oldest-first
- [ ] Add publish-verification to every run (fetch the post URL, assert the title)
- [ ] Join affiliate programs for reviewed tools and wire the substitution map
- [ ] AdSense application once the blog passes 30+ published posts with real traffic
- [ ] Monthly "what the automation got wrong" review post — written by hand

---

*Stack: scheduled server-side jobs · a state file and a log file doing more work than any framework · Blogger · Gmail (retired route) · Google Apps Script + Blogger API (Plan B) · generated images · an HTML template and a checklist.*

*Questions or ideas? Open an issue — I read everything, and corrections are genuinely welcome.*
