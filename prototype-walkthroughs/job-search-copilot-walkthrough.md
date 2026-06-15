# Job Search Copilot — Walkthrough Guide

**GitHub:** https://github.com/siddharthjaiswal1993-spec/job_search

---

## Demo Goal

Show a working React application that manages job applications as a structured pipeline, with AI-assisted resume tailoring, cover letter drafting, and application tracking across stages.

---

## What Problem This Demonstrates

Job search is a sales pipeline that most people manage in a spreadsheet. There is no structured CRM for job applications, no AI layer to tailor application materials to specific JDs, and no systematic way to track what works. This is a working application — not a concept — that demonstrates applied React skills alongside product thinking about a real personal workflow.

---

## Primary Persona

**Experienced professional actively searching for a new role.** Has 5-15 active applications at any time. Currently tracks applications in a spreadsheet. Spends 45 minutes tailoring each resume and cover letter from scratch. Has no visibility into which application stage is the bottleneck.

---

## Loom Script (~3 minutes)

"Let me show you Job Search Copilot — a working React application I built for structured job search management.

This one is different from my other portfolio projects: it's a working app I built for my own use, not just a concept or a strategy document. It solves a real problem I was experiencing.

Job search is fundamentally a sales pipeline. You have a target list, you have outreach stages, you have a win/loss rate, and you have a feedback loop to improve. But most people manage it in a spreadsheet with no structure and no AI assistance.

Job Search Copilot is a React application with three core features.

First, the application pipeline: a Kanban board with stages — Saved, Applied, Phone Screen, Technical, Final Round, Offer, Rejected. Each card shows the role, company, priority score, and next action. Color-coded by urgency.

Second, AI-assisted tailoring: when you add a job, you paste the JD. The app generates a tailored resume summary and cover letter draft keyed to the specific role and company. You edit and use. What took 45 minutes takes 10.

Third, analytics: track your funnel conversion rates. Where are you losing? If you have a 60% phone screen rate but a 20% technical conversion, the problem is interview performance, not sourcing. This tells you where to invest practice time.

The app runs locally and uses Claude API for the AI features."

---

## Click Path (Application)

1. Run locally: `npm install && npm run dev` → http://localhost:5173
2. Show Kanban pipeline — applications across stages
3. Click "Add Application" — show JD input and AI tailoring flow
4. Show tailored resume summary + cover letter draft output
5. Navigate to Analytics — show funnel conversion rates by stage
6. Show Notes/Timeline on an individual application card

---

## Key Screens

**Screen 1 — Application Pipeline (Kanban)**
Highlight: All active applications visible at once with stage, priority, and next action. Filter by stage, company size, or role type. This replaces the spreadsheet as the source of truth.

**Screen 2 — AI Tailoring (Resume Summary)**
Highlight: JD input → AI-generated tailored resume summary that mirrors the job's language and requirements. The PM editing this is doing high-level review, not writing from scratch. 5:1 time savings.

**Screen 3 — Cover Letter Draft**
Highlight: Structured draft with a hook, a match paragraph, and a forward-looking close. Personalized to the company's product area. Editable in-app before copying out.

**Screen 4 — Analytics**
Highlight: Funnel view: Applications → Phone Screens → Technicals → Finals → Offers. Conversion rates per stage. This is the only way to know where the bottleneck is. No spreadsheet gives you this automatically.

---

## If Asked "Is This a Real Product?"

"Yes — Job Search Copilot is a working React application. It uses a Claude API integration for AI features and LocalStorage for state persistence (no backend). I built it for my own use during my job search. The demo data is synthetic."

---

## Interview Talking Points

**For "Show me something you built for yourself":**
"Job Search Copilot is the clearest answer: I was managing my job search in a spreadsheet and it wasn't working. So I built a React app with an AI tailoring layer. The product insight was that job search is a sales pipeline — and every sales pipeline problem has been solved with CRM tooling. I adapted those patterns to the personal job search context, added an AI layer for the high-friction step (tailoring materials), and got a 5× reduction in the time I was spending per application. I still use it."

**For "How do you validate product ideas?":**
"Job Search Copilot is a direct example: I was the user. I knew exactly what the problem was because I was experiencing it. I didn't do customer discovery — I did a 10-day build instead. The test was whether I would use it myself. I did. The next validation step would be to see whether other job seekers have the same friction pattern before investing further."
