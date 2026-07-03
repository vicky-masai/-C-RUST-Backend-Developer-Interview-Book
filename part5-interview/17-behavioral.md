# Chapter 17 — Behavioral Questions & Company Research

> Technical skill gets you shortlisted; the behavioral round decides between two equally-skilled candidates. This chapter: the STAR method, prepared stories, questions to ask *them*, and how to research Rieter specifically.

## 17.1 The STAR method (use it for every "tell me about a time" question)

- **S**ituation — one sentence of context ("Our telemetry service started dropping readings under peak load").
- **T**ask — your responsibility ("I owned the consumer side and had to fix it without downtime").
- **A**ction — what *you* did, technically specific, "I" not "we" ("I profiled with flamegraphs, found per-message DB inserts, batched them and made writes idempotent").
- **R**esult — measurable outcome + lesson ("Throughput went from 2k to 40k msg/s; we added lag alerts so it can't silently regress").

Rules: 90 seconds max per story, lead with the situation not your childhood, always land on a result with a number if possible, and end with what you learned or changed permanently.

## 17.2 Prepare SIX stories (they cover ~90% of behavioral questions)

Write your own versions of these before the interview — bullet points, then rehearse out loud:

| # | Story to prepare | Questions it answers |
|---|---|---|
| 1 | **Hardest technical bug** you solved (use a Ch 14 war-story shape) | "hardest problem", "debugging story", "proudest moment" |
| 2 | A time you **disagreed** with a teammate/lead and how it resolved | "conflict", "disagreement with manager", "pushback on design" |
| 3 | A **mistake/failure** you owned (bad deploy, missed edge case) | "biggest mistake", "a time you failed", "weakness in action" |
| 4 | A **deadline crunch** and how you scoped/prioritized | "pressure", "tight deadline", "how do you prioritize" |
| 5 | Something you **learned fast** (e.g. picking up Rust, Kafka) | "learning agility", "new technology", "outside comfort zone" |
| 6 | A time you **improved a process** (CI, testing, code review, docs) | "initiative", "impact beyond code", "leadership without title" |

For each: one line S, one line T, three lines A, one line R. Keep a printed cheat sheet for the morning of the interview.

### Worked example (story 3 — the mistake)

> "I once shipped a config change that assumed RPM values fit in a 16-bit field; a new machine model exceeded it and readings silently wrapped (S). I was on support that week and owned the fix (T). I reproduced it in a test first, fixed the type, then audited the codebase for other narrow integer assumptions and added a range-check at ingestion with an alert (A). Fix shipped same day; the audit found two more latent cases, and validation-at-the-boundary became a team convention (R). Lesson: never let bad data in silently — fail loudly at the edge."

Notice the shape: own it fast → fix → *systematize so it can't recur*. That last step is what separates junior from senior answers.

## 17.3 Standard behavioral questions — answer sketches

**"Tell me about yourself." (the opener — script this!)**
60–90 seconds, present → past → future: "I'm a backend developer working mainly in C++/Rust — currently I build X at Y, where I own Z. Before that I did A, which taught me B. I'm looking for a role with more systems-level work close to real machines, which is why Rieter caught my attention." Do NOT recite your CV chronologically.

**"Why do you want to leave your current job?" / "Why this role?"**
Always forward-looking, never trash-talking: growth, technology match (C++/Rust/systems), domain interest (industrial/manufacturing tech). "I want to work closer to physical systems where software quality has visible real-world impact" lands very well at a machinery company.

**"What's your greatest weakness?"**
Real weakness + active mitigation, not a humblebrag: "I used to go too deep polishing code before sharing it; I now open draft PRs early to get direction feedback before investing in polish." Pick one that isn't disqualifying for a backend role.

**"Where do you see yourself in 5 years?"**
Ambitious but aligned: growing into a senior/lead engineer who owns system design, mentors, and is a domain expert in [their space]. Avoid "management" unless it's a lead role, and avoid "I don't know".

**"How do you handle disagreement about a technical decision?"**
Process answer: understand their reasoning first → argue with data/prototypes not opinions → agree on decision criteria → commit fully once decided even if it's not my choice → revisit with evidence if reality disproves it. Have story 2 ready as the example.

**"How do you handle a task you don't know how to do?"**
Timebox research → read docs/code before asking → ask a *specific* question with what I've tried → build a small spike to de-risk → keep the ticket updated. Signals independence *and* communication.

**"Tell me about working with non-technical stakeholders / requirements." (JD-relevant)**
Use Ch 13.3 language: clarifying questions, restating in my own words, acceptance criteria before code, demoing early. Machinery domain bonus: "I'd expect to work with mechanical/process engineers — I enjoy translating domain constraints into software requirements."

## 17.4 Questions to ask THEM (prepare 5, ask 3)

Asking nothing reads as low interest. Good ones for this role:

**About the work:**
- "What does the C++ vs Rust split look like today, and where is Rust adoption heading in the team?"
- "Is the software on the machines themselves, in the plant (edge), in the cloud — or all three? Where does this role sit?"
- "What does the data pipeline from a spinning machine to an operator dashboard look like today?"

**About the team & process:**
- "How does the team do code review and testing? What's the path from merge to running in a customer's plant?"
- "What does success look like for this role after 6 months?"

**Strategic (for a senior interviewer/manager):**
- "Rieter talks about digitization of spinning mills — how central is this team to that strategy?"

Avoid asking about salary/leave in the technical rounds; save for HR.

## 17.5 Researching Rieter (do this the week before — verify current facts)

> The facts below are the stable outline — **verify current numbers/news on rieter.com and recent press releases before the interview**, since leadership, results and product names change.

**What the company is:** Rieter is a Swiss company (HQ Winterthur, founded 1795) and the world's leading supplier of systems for **short-staple fiber spinning** — the machines that turn cotton/man-made fibers into yarn. Listed on the SIX Swiss Exchange. Operates worldwide with strong presence in Asia (major markets: China, India, Turkey).

**Business areas (typical structure):** machines & systems (complete spinning lines: bale opening → carding → drawing → ring/rotor/air-jet spinning), components (spare/wear parts), and after-sales/services — plus **digital products** connecting machines in a mill.

**Why they hire backend C++/Rust developers:** modern spinning machines are packed with sensors and controllers; mills want plant-wide monitoring, quality analytics, predictive maintenance, remote diagnostics. That means: embedded/edge software talking to machine PLCs, data pipelines (hello Kafka), APIs and dashboards (hello REST/WebSocket), time-series storage (hello databases chapter) — exactly this book's stack. Their digital platform for mill monitoring (branded ESSENTIAL in recent years) is the kind of product this role likely supports.

**Domain vocabulary worth knowing (drop one, naturally):**
- **Short-staple spinning**: processing fibers (cotton) up to ~50 mm into yarn.
- Process stages: **blowroom** (opening/cleaning bales) → **carding** (aligning fibers into sliver) → **drawing** → **roving** → **spinning** (ring, **rotor/open-end**, or **air-jet**) → winding.
- **Ring spinning** = highest yarn quality, most spindles; **rotor** = high productivity for coarser yarn; **air-jet** = fast, specific yarn character.
- KPIs a mill cares about: yarn quality (evenness, imperfections), **ends down** (thread breaks), spindle/rotor speed (RPM!), energy per kg of yarn, machine utilization/OEE.

**How to use the research:** one sentence in "why Rieter" ("the combination of 200+ years of machinery expertise with a modern digitization push"), one domain term in a design answer ("say a carding machine streams quality metrics…"), and one informed question from 17.4. Don't lecture them about their own company.

**Checklist the week before:**
- [ ] Read the latest annual/half-year report summary (investor page) — revenue trend, current strategy keywords.
- [ ] Read 2–3 recent press releases (new products, digital announcements).
- [ ] Look up your interviewers on LinkedIn — their background shapes their questions.
- [ ] Re-read the job description; map every bullet to a chapter of this book and one story of yours.

## 17.6 Logistics & final-day checklist

- **Salary question:** give a researched range, not a single number; or deflect once: "I'd first like to understand the full scope — what range is budgeted?" Know your walk-away number beforehand.
- **Remote interview setup:** wired connection if possible, close Slack/notifications, pen + paper for design sketches, water, your six-story cheat sheet, the JD, and 3 questions printed.
- **On the day:** arrive/log in 10 min early; in the first minute smile and get names; when you don't know something say so and reason out loud; when you finish an answer — *stop talking*.
- **After:** send a short thank-you note referencing one specific discussion point; note down every question you were asked (for round 2 and for future you).

---

## 🎯 Chapter 17 Interview Q&A (meta-round)

**Q1. "Tell me about yourself" — what's the structure?**
Present → past → future in 60–90 s: current role and ownership, one relevant past achievement, why this role is the logical next step. Scripted and rehearsed, but delivered naturally.

**Q2. How do you answer a question you have no story for?**
Bridge to the nearest prepared story: "I haven't faced exactly that, but a similar situation was…" — six stories cover almost everything if you bridge deliberately.

**Q3. They ask about a technology you don't know. Response?**
Honesty + adjacency + speed: "I haven't used SOAP in production, but I know it's contract-first XML with WSDL; given my REST experience the concepts map, and I pick these up fast — like when I learned Kafka in X weeks."

**Q4. How do you show seniority signals at 3–7 yrs level?**
Own outcomes not tasks; mention prevention (tests, alerts, conventions) after every fix; talk trade-offs unprompted; scope things ("I'd timebox that spike to a day"); ask about NFRs in design questions.

**Q5. The interviewer is silent after your answer. What do you do?**
Don't ramble to fill silence — it's often deliberate. Ask "Would you like me to go deeper on any part?" or simply hold the pause.

---

*You've reached the end of the book. Go back to the [study plan](../README.md) — and remember: type the code, say the answers out loud, and sleep before the interview. Good luck!*
