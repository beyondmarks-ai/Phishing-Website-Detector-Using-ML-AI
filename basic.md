# How this project works (simple guide)

*Copyright © Beyond Marks AI Academy. This page explains the system in plain language anyone can follow—including students who are new to security tools.*

---

## 1. What is this, in one sentence?

This project is a **link safety checker**: you give it a web address (URL), and it looks at many different clues to guess whether the link is **probably safe**, **suspicious**, or **risky**—and it tells you **why**, not just a single yes/no.

---

## 2. The big idea (picture it like a check-up)

Think of a URL like a person walking into a clinic:

- **Nurses** run different small tests at the same time (one checks the heart, one checks temperature, one checks papers…).
- Each test writes a note: “good sign,” “bad sign,” or “nothing special.”
- At the end, a **doctor** adds up the notes and gives a **health score** and a **label** (Safe / Suspicious / Risky).

In our software, the “tests” are **small programs** (we call them **tasks** or **analyzers**). They all run **in parallel** (many at once) so the answer comes back in **a fraction of a second to about a second**, depending on the network and your setup.

---

## 3. How it works, step by step

### Step A — You send a link

- The app receives something like `https://example.com/path`.
- It **cleans and checks** the text so it is a real web address. Very broken input is rejected.

### Step B — The system collects facts about that link

Many checks run at the same time. Each one answers a *different* question. Examples of questions the system tries to answer:

| Kind of question | Simple explanation |
|------------------|--------------------|
| **How popular is this site?** | Uses a **traffic ranking** list (Tranco) so “unknown” or “very popular” can influence the result. |
| **Does the name look like a famous brand, but with typos?** | Compares the domain to well-known site names. |
| **Is this domain name random gibberish?** | Looks at **randomness** in the name (see “Algorithms” below). |
| **Is the link a shortener?** | Short links hide the real destination. |
| **Does the link use weird characters to look like a real site?** | **Punycode** / **homoglyph** checks. |
| **Where does the link *actually* go?** | **Redirect chain** (hop by hop) until a final page. |
| **Is there HTTPS and a normal certificate?** | **SSL/TLS** checks. |
| **Is the site known as phishing?** | Looks up **PhishTank** (a public list of reported phishing links). |
| **What is on the page?** | Fetches the page (when possible) to see **forms** (login, payment), **hidden iframes**, **where forms send data**, and **brand** words vs the real domain. |
| **How old is the domain?** | **WHOIS / RDAP** (registration data), when available. |
| **Are DNS records sensible?** | **Nameservers** and **email (MX)** records, as signals. |

Not every check succeeds every time (some sites block robots, or time out). The system is built to still return a result when **some** checks fail, and it may mark the analysis as **incomplete** in those cases.

### Step C — Scores and a verdict (the “decision”)

The code uses **heuristic scoring** (rules + points), **not** a neural-network “AI model” that learned from millions of images.

- **Trust points** and **risk points** are added up from many small rules (see the next section).
- These are combined into a **final score** (0–100) and a **verdict**:
  - **Risky** — very low final score
  - **Suspicious** — in the middle
  - **Safe** — high enough final score

You also get **lists of reasons**: good, bad, and neutral, so a human can read *why* the system decided what it did.

---

## 4. “Algorithms” and methods (explained for beginners)

**Heuristic** means: *we use clear “if this, then add points” rules* that security people understand and can argue about. The main scoring lives in the **analyzer** (for example `server/internal/analyzer/result.go` in the code).

### 4.1 Trust score, risk score, and final score

- Many signals add to **trust** (good) or **risk** (bad).
- Both are **capped** so they never go above 100.
- A **final score** is calculated from the difference between trust and risk (with a simple formula) so the result stays in a sensible 0–100 range.
- **Verdict** uses **score bands** (thresholds): low = Risky, middle = Suspicious, high = Safe.

This is a **scoring function**, not machine learning. You can read the exact rules in the code if you want to audit them.

### 4.2 String distance — **Levenshtein** (typosquatting)

**Typosquatting** = making a domain that looks *almost* like a famous one (`paypall` vs `paypal`).

- The system compares the **second-level name** in the domain (e.g. `paypal` in `www.paypal.com`) to names of **top well-known** sites (from a list loaded at startup, based on the same ranking data file).
- It uses **Levenshtein distance** (a classic computer-science “edit distance”): the smallest number of single-letter **insert, delete, or change** steps to turn one string into another.
- If the distance is **1 or 2** and a few other conditions hold, the system flags a possible **typosquat** or **combo-squat** (a brand name hidden inside a longer name).

Code reference: `server/internal/service/typosquat/typosquat.go`.

### 4.3 Information theory — **Shannon entropy** (random-looking names)

Some phishing sites use **long random** sub-domains. Random strings **look** messy. One way to measure that mess is **Shannon entropy** (from information theory), applied to the letters in a domain name.

- High randomness is **suspicious**; the code combines entropy with a few other simple letter-pattern checks to build a **randomness** signal.

Code reference: `server/internal/service/checks/entropy.go` (function names like `shannonEntropy`, `AnalyzeDomainRandomness`).

### 4.4 Threat intelligence — **PhishTank**

- The software can ask the **PhishTank** service whether a URL has been **reported** as phishing.
- The score treats **verified, confirmed** phishing as **very serious** compared to a report that is still unverified.  
  (Exact values are in the scoring rules in `result.go`.)

### 4.5 Rules for URLs, pages, and infrastructure (no fancy math, just clear checks)

These include things like: URL **length** and **path depth** limits, **subdomain** count, **sensitive words** in the path, **IP address in the host** (instead of a domain), **HSTS** support, **redirect** count, **HTTP status**, **NS/MX** DNS sanity, **domain age** from registration data, and **form** behavior (e.g. password on non-HTTPS, form posting to a **different** domain).

### 4.6 Optional: screenshots

If **headless browser** (Chrome) is available in the deployment, a **screenshot** path can be used for the UI. If Chrome is not connected, that feature is skipped or limited—**the rest of the engine still works**.

### 4.7 Caching (speed)

- **Valkey/Redis** (when running) can remember recent answers for a while so the same link is not fully re-scanned on every request.
- If the cache is down, the program **continues**; it may be slower and **rate limits** may be disabled.

---

## 5. How the software was built (the “recipe”)

This is a **client–server** design:

- **Backend** — **Go** (Golang) language, **Gin** web framework, **HTTP API** for analysis and health checks.
- **Frontend (website)** — **SvelteKit** + **Vite**, **TypeScript**, **Tailwind CSS** for a modern page where you paste a URL and see the report.
- **Browser extension** (optional) — a small **Chrome** extension in `web/chrome-extension` that can talk to your local or hosted API.
- **Containers (optional)** — **Docker Compose** can run the backend, web front, **Valkey**, and a **headless Chrome** image together for a full production-like setup.
- **Metrics** — **Prometheus**-style endpoint for operators who monitor the service.

**Important:** The **module path** in Go may still use a `github.com/...` string inside imports; that is a **technical name** the Go tool expects. It does *not* mean the running product is “claiming to be a public GitHub project.” The behavior is defined by **your** deployment and this codebase.

---

## 6. What this tool does **not** do (honest limits)

- It is **not 100% accurate**. Bad sites can look okay for a while; good sites can look odd. **Always** combine tools with your own judgment and your organization’s policies.
- It is **not a replacement** for full enterprise security products or a human analyst.
- It relies on **public data, DNS, the live web**, and **third-party lists**; any of these can be wrong, late, or temporarily unreachable.

---

## 7. Where to read more in this repository

- **Run and test** — `docs/setup.md`, `docs/testing.md`
- **API details** — `docs/api.md`, and when the server runs: `/swagger/index.html`
- **Configuration** — `docs/configuration.md`

---

## 8. Glossary (quick)

| Word | Simple meaning |
|------|----------------|
| **URL** | A web link. |
| **Domain** | The “home address” of a site, e.g. `example.com`. |
| **Heuristic** | A simple, explained rule, not a black-box AI. |
| **TLS / SSL** | Encrypted (HTTPS) connection and certificate checks. |
| **Phishing** | Tricking you into giving secrets or money using fake sites or links. |
| **Punycode** | Encoding that can make foreign letters look like normal Latin letters in a link. |
| **Homoglyph** | Look-alike characters (e.g. `rn` vs `m`). |
| **Redirect** | A link that bounces the browser to another address before the final page. |
| **RDAP / WHOIS** | Public registration info about who registered a domain and when. |

---

*If you are a student: read sections 2–4 first, then open the code in `server/internal/analyzer` with a teacher or mentor. That is where the “reasoning” of the tool becomes concrete.*
