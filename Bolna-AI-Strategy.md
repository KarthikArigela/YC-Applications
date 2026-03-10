# Bolna AI — Strategic Analysis & Prototype Play
> Goal: Land a role at Bolna AI not with a resume, but by solving a real problem they haven't solved publicly yet.

---

## 1. What Bolna AI Is Actually Building (Decoded from JDs)

Bolna is not just a "voice AI company." They are building the **AWS of Voice AI** for Indian enterprises — a full-stack infrastructure layer that abstracts away:

- ASR (Automatic Speech Recognition) provider selection
- TTS (Text-to-Speech) voice selection
- LLM orchestration and prompt routing
- Real-time telephony infrastructure
- Multi-language handling (10+ Indian languages)

Their core value prop: **Route every call dynamically across the best ASR/LLM/TTS combo based on language, latency, and cost** — so enterprises don't have to manage any of it.

**Scale today:**
- 250K+ daily calls
- 1,500+ paying customers
- 3M+ minutes/month
- 15% MoM growth
- $6.3M raised (General Catalyst led)
- Profitable at $300K+ ARR

---

## 2. What These 6 Hires Signal — The Real Strategic Picture

### Hire 1: Full-Stack Engineer (Zero to One team)
**What it actually means:** Bolna knows their current product has a ceiling. They are spinning up an **internal startup** to find the *next* business to build on top of their Voice AI infrastructure. They need a technical co-founder equivalent to identify the problem AND build the solution.

**Hidden signal:** They don't know what to build next. They're looking for someone with enough founder instinct to go figure it out. This is a company that's admitting it needs to explore its own adjacencies.

### Hire 2: Forward Deployed Engineer (FDE) — Business
**What it actually means:** POC-to-production conversion is their biggest bottleneck. Customers say yes, run pilots, but getting them to go fully live and expand is leaky. The FDE plugs this gap manually right now — they're essentially deploying human labor to compensate for missing product.

**Hidden signal:** Their self-serve product flow is broken or missing. Customers can't configure and launch voice agents without hand-holding. They haven't productized their own onboarding.

### Hire 3: GTM Lead
**What it actually means:** Revenue is still founder-led. They're hitting the ceiling where the founders can't personally close every deal. They need someone to build the repeatable GTM motion — playbooks, pipeline, and a scaled FDE team.

**Hidden signal:** They have no repeatable sales process yet. Every deal is still somewhat custom. The product-market fit exists but the go-to-market fit is still being discovered.

### Hire 4: Product Designer (Graphics + UI/UX)
**What it actually means:** Their UI is functional but ugly or confusing. They are building an "AI agent builder" interface that needs to be good enough for non-technical enterprise buyers to use or at least understand. They also need brand assets for market presence.

**Hidden signal:** Their dashboard/console UX is an internal tool masquerading as a product. No design system, no coherent visual language, no way for customers to feel confident without calling an FDE.

### Hire 5: Senior Software Engineer (Frontend)
**What it actually means:** The frontend of their platform — agent builder, real-time monitoring, customer workflows — is under-built. They mention "real-time monitoring dashboards" which suggests they either don't have them or they're primitive.

**Hidden signal:** Customers using Bolna can't observe what their voice agents are doing in production. No good live dashboards, no call analytics, no visibility into failure modes without jumping on a call with Bolna's team.

### Hire 6: EIR (Entrepreneur in Residence)
**What it actually means:** Bolna wants to incubate ideas adjacent to their platform using their own alumni network or founders-in-waiting. This is a signal they're thinking platform ecosystem, not just product.

---

## 3. The Hidden Pain Points (Not Visible to the Open World)

These are the problems you WON'T find on their website, blog, or G2 reviews — but that fall directly out of reading these JDs carefully:

### Pain Point #1: Voice Agent Observability Gap
**The problem:** Once a voice agent is live, neither Bolna's team nor the customer has a good way to understand *why* calls fail. An agent that handles 10K calls/day will have hundreds of edge-case failures — wrong language detection, interruption handling failures, hallucinated responses, context drops. Right now, debugging this means listening to call recordings manually or eyeballing transcripts.

**Evidence from JDs:**
- FDE role: "Debug failures in real usage, not staging environments"
- FDE role: "Test live calls, monitor outcomes, debug failures, and refine behavior"
- Frontend role: "real-time monitoring dashboards"
- Frontend role: "real-time Voice AI conversations in parallel"

**Nobody is solving this well.** Existing call analytics tools (Gong, Chorus) are built for human sales calls. None are built for AI agent calls where the failure modes are completely different.

### Pain Point #2: The POC-to-Production Chasm
**The problem:** Customers run a POC. It works. Then months go by and they never fully deploy. Why? Because going from "this works in testing" to "I trust this with my 10K customers" requires confidence they don't have. They need proof that the agent handles edge cases, speaks the right languages at the right moments, doesn't hallucinate, and fails gracefully.

**Evidence from JDs:**
- FDE role: "taking a customer from 'can this work for us?' to 'this is live and critical to our business'"
- FDE role: "Move customers from pilot → rollout → expansion by proving ROI"
- Zero to One: "Get working prototypes in front of customers quickly"

**The missing piece:** There's no standardized "production readiness scorecard" for a voice AI agent. Nobody can tell a customer objectively: "your agent is 87% ready for production, and here's what to fix."

### Pain Point #3: No Self-Serve Agent Quality Testing
**The problem:** Before launching a voice agent, how do you know it works? Right now, you have to make real calls manually or run staging calls with the Bolna team. There's no automated way to stress-test a voice agent against 100 simulated customer scenarios before going live.

**Evidence from JDs:**
- FDE role: "Ship fast, in production... Launch MVP agents in days"
- FDE role: "Test live calls, monitor outcomes"
- Zero to One: "Debug failures in real usage, not staging environments"

**The gap:** No voice AI equivalent of unit tests or load tests. The industry is missing a "red-teaming tool" for voice agents.

### Pain Point #4: Enterprise Integration Friction
**The problem:** Enterprise customers in BFSI, E-commerce, Edtech, Healthcare all have existing CRMs, ticketing systems, and data pipelines. Every Bolna deployment requires custom integration work. There are no pre-built connectors or templates.

**Evidence from JDs:**
- GTM role: "clients across sectors like BFSI, E-commerce, Edtech, and Healthcare"
- FDE role: "Configure voices, languages, transcribers, interruption handling, and integrations"
- Zero to One: "identify the biggest unsolved problems in India's enterprise Voice AI market"

### Pain Point #5: No Multi-Language Agent Quality Benchmarking
**The problem:** Bolna supports 10+ Indian languages. But is their Hindi ASR as good as their Tamil ASR? Which TTS voice sounds most natural for Marathi? Which LLM handles code-switching (mixing Hindi + English) best? There's no public or internal benchmark — which means FDEs and customers are flying blind when choosing language configurations.

---

## 4. The Prototype to Build — The "Voice Agent Intelligence Dashboard"

### What to Build
**An open-source Voice Agent Observability & Production Readiness Tool** that:

1. **Ingests call transcripts/recordings** (from Bolna's API or uploaded CSVs)
2. **Auto-analyzes every call** using LLMs to detect:
   - Failure patterns (confusion loops, wrong language, hallucinations, dropped context)
   - Customer sentiment shifts
   - Agent interruption handling quality
   - Silence/latency spikes
   - Successful resolution vs. failed resolution classification
3. **Generates a Production Readiness Score** (0–100) with specific recommendations:
   - "Your agent fails 34% of the time when customers switch to Hindi mid-conversation"
   - "Interruption handling causes 18% of calls to drop early"
   - "Your fallback prompt is triggering for valid inputs — here are the edge cases"
4. **Simulates synthetic test calls** using an LLM to generate 50+ realistic customer personas and conversation scenarios, then runs them through a configurable voice agent prompt — exposing failures before they hit production
5. **Tracks improvement over time** — when an agent is updated, shows before/after quality deltas

### Why This Is the Right Prototype

| Reason | Detail |
|--------|--------|
| Solves Pain Point #1 | Directly addresses the observability gap |
| Solves Pain Point #2 | The readiness score is what unlocks POC → production |
| Solves Pain Point #3 | Synthetic test simulation replaces manual call testing |
| Relevant to 3 of 6 roles | Frontend Engineer, FDE, Zero to One team |
| Buildable in 2 weeks | Core MVP is feasible solo |
| Directly uses Bolna's stack | Built on top of Bolna's platform — shows you understand it |
| Open source = press + credibility | GitHub stars, a blog post, and an HN submission = inbound |

---

## 5. 2-Week Build Plan

### Week 1: Core Engine (Days 1–7)

**Day 1–2: Setup & Data Pipeline**
- Set up a Next.js frontend + FastAPI/Node backend
- Build transcript ingestion: accept JSON/CSV upload or Bolna webhook payload
- Parse transcripts into turn-by-turn conversation objects
- Store in SQLite or Supabase

**Day 3–4: LLM Analysis Engine**
- Build a prompt pipeline using Claude API (or GPT-4o) that analyzes each call
- Extract: resolution status, failure points, language switches, sentiment, confusion indicators
- Generate structured JSON output per call with scores

**Day 5–6: Production Readiness Scorecard**
- Aggregate analysis across 10+ calls to compute weighted scores
- Build the scoring rubric:
  - Language handling (20%)
  - Interruption handling (20%)
  - Context retention (20%)
  - Fallback quality (20%)
  - Resolution rate (20%)
- Generate natural-language recommendations per category

**Day 7: Synthetic Call Simulator (MVP)**
- Build a prompt that generates N synthetic customer personas for a given use case (e.g., "loan EMI reminder calls for BFSI")
- Simulate the conversation turn-by-turn using an LLM playing the "customer"
- Run the agent prompt against each synthetic customer and score the outcomes

### Week 2: Polish & Ship (Days 8–14)

**Day 8–9: Frontend Dashboard**
- Clean, Bolna-branded (dark theme, minimal) dashboard
- Call list with scores, filterable by failure type
- Per-call drill-down: conversation timeline, highlighted failure moments, fix suggestions
- Overall readiness gauge chart

**Day 10–11: Before/After Comparison**
- Allow uploading two versions of an agent prompt
- Show side-by-side quality delta
- "Version A resolved 67% of calls. Version B resolves 84%. Here's what changed."

**Day 12: Integration Demo**
- Record a 5-minute Loom demo showing:
  1. Uploading real (anonymized) call transcripts
  2. Seeing failure analysis
  3. Readiness score and recommendations
  4. Fixing a prompt and seeing the score improve
- Make it specific to an Indian BFSI use case (EMI reminder calls)

**Day 13: Write the Blog Post**
- Title: *"We analyzed 10,000 Voice AI calls and found 5 failure patterns nobody talks about"*
- Publish on Medium/Substack and submit to HN
- Tag Bolna, reference their platform

**Day 14: Outreach**
- DM Prateek Sachan (CTO, mentioned in Zero to One JD) on LinkedIn with the GitHub link
- DM the founders
- Apply to the role, but lead with: "I built this while studying your platform. I think it solves a real gap in your FDE workflow."

---

## 6. Tech Stack Recommendation

| Layer | Choice | Reason |
|-------|--------|--------|
| Frontend | Next.js + TailwindCSS + shadcn/ui | Fast to build, clean output |
| Backend | FastAPI (Python) | LLM integrations, transcript parsing |
| LLM | Claude API (claude-sonnet-4-6) | Best for structured analysis tasks |
| Database | Supabase (Postgres) | Free tier, real-time, auth built-in |
| Charts | Recharts or Tremor | React-native, no D3 overhead |
| Deployment | Vercel (frontend) + Railway (backend) | Free tiers, instant deploy |
| Auth | Supabase Auth or none for MVP | Skip auth for demo purposes |

---

## 7. What Role to Apply For

Based on your ability to build this prototype in 2 weeks, the strongest fit is:

### Primary: Full-Stack Engineer (Zero to One Team)
**Why:** You will have demonstrated exactly what this role requires — identifying an unsolved problem in the Voice AI space and building a working prototype solo, fast. The JD says "Your technical intuition will shape what's feasible" — you'll have proven that before your first interview.

### Secondary: Forward Deployed Engineer (Business)
**Why:** If you also record demos and frame this around customer value (POC-to-production conversion rate), you show FDE instincts too.

---

## 8. The Pitch (What to Say to Bolna)

> "I spent 2 weeks studying your platform and the problems your FDE team faces daily. I noticed that there's no good tool for voice AI agents to understand *why* their calls fail in production — and no automated way to test an agent before launch. I built a prototype that solves both. It ingests call transcripts, auto-detects failure patterns using LLMs, generates a Production Readiness Score, and simulates synthetic test calls against your agent prompt. I'd love to show you how it works and contribute to the Zero to One team or FDE motion. Here's the GitHub: [link]. Here's a 5-min demo: [link]."

---

## 9. Why This Works

1. **You're not asking for a job. You're delivering value.** This reframes the entire interaction.
2. **You've already done the work of an FDE** — you studied their customer pain, built a solution, and can demo it.
3. **It's directly relevant to 3 of their 6 open roles** — you're not applying for one role, you're showing you understand the whole company.
4. **Open source + blog post = credibility signal** that's hard to fake. It shows you ship.
5. **You identified a pain point they haven't solved publicly** — that's the highest-value signal you can send to an early-stage startup.
6. **It doubles as a portfolio piece** forever, regardless of the outcome.

---

## 10. Risk Mitigation

| Risk | Mitigation |
|------|------------|
| They already have this internally | "I'd love to see how your internal version compares and where you'd take it next" |
| They ignore the DM | Post publicly on LinkedIn, tag them — social proof from community engagement brings them to you |
| They say "cool but we're not hiring for this" | You've still built a portfolio piece and an open-source tool that attracts attention from other voice AI companies |
| You don't finish in 2 weeks | Ship a working demo of the analysis engine only — the synthetic simulator can be V2 |

---

*Document created: March 2026 | Strategy for landing a role at Bolna AI through demonstrated value*
