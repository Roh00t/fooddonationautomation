# Food Donation Matching — Value Case & UiPath Activity Mapping

## AI Automation Module | 6 members | Deadline 18 September 2026

> **Correction to earlier plans.** Previous drafts specified `Digitize Document`,
`Data Extraction Scope` and `Generative Extractor` for WF2. Those belong to the **IntelligentOCR**
package, which is **not available in Studio Web**. The Studio Web equivalent is
**`Extract Document Data`** from the **Document Understanding** package. See §6.3.
> 

---

# PART A — THE VALUE CASE

---

## 1. Problem statement

### 1.1 The organisation

A food rescue coordinator sits between food suppliers (supermarkets, distributors, F&B outlets) with near-expiry stock they cannot sell, and recipient organisations (shelters, soup kitchens, family service centres) that need food. Singapore has real operators here — Food from the Heart, The Food Bank Singapore — so the client is credible without inventing one.

### 1.2 The manual process today

1. Log into each supplier's portal and download the day's delivery notes
2. Read each note; record item, quantity and expiry into a stock list
3. Work out which items are close to expiring and still safe to donate
4. Recall or look up which recipients want what, and which can physically accept it — halal-only, no cold storage, limited collection capacity
5. Decide who gets what, and how much, when supply cannot cover demand
6. Pick a collection slot that works for the recipient and beats the expiry date
7. Message each recipient, usually WhatsApp or email, with what they are getting and when
8. Update the stock list so the same item is not promised twice

### 1.3 Why it breaks

- **Steps 1–3 are clerical repetition.** Same actions daily, no judgement involved.
- **Step 4 is invisible knowledge.** Recipient constraints live in a coordinator's head or a scattered set of notes. A new coordinator cannot reproduce the decisions.
- **Step 5 degrades under pressure.** The observed real-world pattern is first-come-first-served — whoever messages the coordinator first gets the stock, regardless of need. That is an *allocation failure*, not a communication failure.
- **Steps 7 and 8 are separate manual actions from step 5.** Any of them can be delayed or skipped, which is how stock gets double-promised and recipients are left without status.
- **The deadline is absolute.** Unlike an invoice or a job application, the work item physically spoils. A missed window is not a delay — it is total loss, and the food goes to waste.

### 1.4 Trigger and end state

**Trigger:** the scheduled daily run — new delivery notes are available on the supplier portal.

**End state:** every recipient with a satisfied allocation has received an email stating exactly what they are collecting, in what quantity, and when and where. The inventory sheet reflects allocated and remaining quantities. Anything expiring with no eligible recipient is flagged at-risk rather than left silently unallocated.

### 1.5 Scope guardrail

5–6 delivery notes, 3–4 recipient organisations, one run. Small enough for a four-minute demo, large enough that the allocation logic must actually decide something.

### 1.6 Data declaration

All delivery notes, recipient profiles and demand records are **created by the team**. No real supplier, charity, or personal data is used. State this in the video and in the GenAI declaration.

---

## 2. What we are automating

| # | Bot / workflow | Manual action it replaces | Brief's feature category |
| --- | --- | --- | --- |
| **WF0** | `Portal_DownloadDeliveryNotes` | Coordinator logs into the supplier portal and downloads today's delivery notes | UI automation |
| **WF1** | `Intake_RegisterDeliveries` | Coordinator opens each note and creates a stock record; checks the demand inbox | Data automation |
| **WF2** | `Extract_DeliveryNoteData` | Coordinator reads each note and transcribes item, quantity, expiry, category | PDF extraction (Document Understanding) |
| **WF3** | `Agent_AllocateSupply` | Coordinator compares expiring stock against every eligible recipient and decides who gets what, how much, and when they collect | **Agentic automation** |
| **WF4** | `Notify_Recipients` | Coordinator hand-writes a message to each recipient | Email automation |
| **WF5** | `Reconcile_UpdateInventory` | Coordinator updates the stock list, tracks who has received what, and remembers who is still waiting | Data automation |

### What stays human — say this on camera

- Setting the matching weights and thresholds (a one-off business decision, not per-item work)
- Reviewing anything routed to Manual Review — low extraction confidence, no eligible recipient, or an allocation the agent declines to force
- Overriding any allocation. The automation proposes; the coordinator can veto.

**This is a strength, not a hedge.** "The agent handles clear cases and escalates unclear ones" is a more defensible position than claiming full autonomy over food distribution, and it directly answers the rubric's demand that you justify design choices.

---

## 3. Where the value comes from

Four distinct sources. Naming them separately beats one vague "it saves time."

### 3.1 Removing repeated comparison work

Comparing one item against a recipient pool is the same cognitive operation repeated N times. It is not that a coordinator *can't* do it — it is that comparison quality degrades with repetition and pool size. The bot compares item 20 against every recipient with the same rigour it applied to item 1.

### 3.2 Changing the allocation rule itself

This is the biggest value item and the easiest to undersell. The manual process allocates by *who asked first*. The automation allocates by a weighted combination of expiry urgency, need fit, and **fairness debt** — how much each organisation has already received. That is not a faster version of the manual process; it is a *different and better* one.

### 3.3 Consistency and auditability

Every allocation is scored against published weights, and every decision — including every declined one — carries a written reason stored against the record. Manual matching produces no equivalent trail unless someone writes one by hand, which under load nobody does.

### 3.4 Closing the decide → record → notify gap

Manually these are three separate actions. Automated, they are one transaction. The inventory can never drift out of sync with the actual decision, which structurally eliminates double-promising — you cannot forget to update a record that updates itself.

---

## 4. Estimated human-effort reduction

> **State these as assumptions, not findings.** We have no operational data from a real food rescue organisation. A grader who asks "where did that number come from?" must get a real answer. Showing the assumption and the arithmetic beats quoting a confident percentage, and it directly avoids the rubric's "manual cost stated in general terms without detail" trap.
> 

### Assumed daily baseline

| Manual step | Assumed time |
| --- | --- |
| Portal login and download | 5 min |
| Reading notes, recording stock | 15 min |
| Identifying near-expiry items | 8 min |
| Matching supply to recipient needs | 20 min |
| Deciding collection slots | 10 min |
| Messaging recipients | 12 min |
| Updating records | 8 min |
| **Total** | **~78 min/day** |

### At illustrative volumes

| Period | Manual effort | After automation* | Reduction |
| --- | --- | --- | --- |
| Per day | ~78 min | ~12 min | ~66 min |
| Per week (5 days) | ~6.5 hrs | ~1 hr | ~5.5 hrs |
| Per month (22 days) | ~28.6 hrs | ~4.4 hrs | **~24 hrs** |
- Residual effort assumes roughly **15%** of items are routed to human review — low extraction confidence, no eligible recipient, or a declined marginal allocation.

**Headline framing:** on these assumptions, roughly **80–85%** of daily manual effort is removed, with the remainder concentrated on cases that genuinely need a person. Say *"on our assumptions"* every time.

### The non-linear point worth making

Manual comparison effort grows with pool size — every new recipient adds comparisons for every item. Automated comparison does not meaningfully grow in *human* time at all. **The saving widens as the service scales.** That reads as considered reasoning rather than a plucked percentage.

### An honest caveat to include

Setup is not free. Defining the criteria, tuning the weights, and reviewing agent decisions in early operation all cost time. The saving is ongoing; the cost is upfront. Saying so makes the rest of your numbers more credible.

---

## 5. Gaps addressed and how

| # | Gap in the manual process | How the automation bridges it |
| --- | --- | --- |
| 1 | **Allocation tracks who asked first, not who needs most** | Fairness-debt term in the scoring function — organisations that received less in prior cycles get priority. This is the headline design decision. |
| 2 | **Comparison quality degrades with pool size and fatigue** | Deterministic weighted scoring applies identical criteria to every pairing regardless of pool size or time of day |
| 3 | **Recipient constraints live in someone's head** | `RecipientProfile` tab holds cold-storage capability, dietary rules, capacity and pickup windows as explicit, queryable data |
| 4 | **No record of why an allocation was made or declined** | Every decision writes a reasoning string against the record — an audit trail that does not exist manually |
| 5 | **Records drift out of sync with decisions** | Status update happens in the same automated transaction as the decision |
| 6 | **Stock double-promised** | Quantity re-read immediately before each allocation; sum-check guard before any write |
| 7 | **Unmatched stock silently forgotten** | WF5 flags near-expiry items with no eligible recipient as `Unallocated-AtRisk` |
| 8 | **Recipients wait for a response** | Notification fires on decision, not on coordinator availability |
| 9 | **Criteria are implicit and unexplainable** | Weights and thresholds live in a `MatchingRules` config tab — visible, adjustable, explainable to a recipient who asks |

### Gaps we do *not* claim to close

- We do not decide what food is safe to donate — expiry rules come from the supplier and regulation.
- We do not remove human oversight of the criteria themselves.
- We do not solve food waste. We reduce the operational cost of one organisation's rescue workflow.

---

# PART B — UIPATH ACTIVITY MAPPING

---

## 6. What is actually available in Studio Web

### 6.1 Package availability — the three that change your build

Read straight off the activity-packages table:

| Package | Studio Desktop (Windows) | Studio Web | Consequence for this project |
| --- | --- | --- | --- |
| **Mail** (Productivity) | ✅ | **❌** | **No** `Send SMTP Mail Message`, `Use Gmail`, `Use Outlook 365`. Email must come from Google Workspace or Microsoft 365 |
| **PDF** (Document Understanding) | ✅ | **❌** | **No** `Read PDF Text` from this package. Use Document Understanding's `Extract PDF Text` instead |
| **IntelligentOCR** (Document Understanding) | ✅ | **❌** | **No** `Digitize Document`, `Data Extraction Scope`, `Generative Extractor`, `Present Validation Station`. Use the Document Understanding package instead |
| **Excel** (Productivity) | ✅ | **❌** | Irrelevant here — we use Google Sheets |
| **Form** (Workflow) | ✅ | **❌** | No UiPath Forms. Google Forms is our intake |
| **Database** (Developer) | ✅ | **❌** | Not needed |

### 6.2 Packages available in Studio Web that this project uses

| Package | Category | Studio Web | Used in |
| --- | --- | --- | --- |
| **UI Automation** | UI Automation | ✅ | WF0 |
| **Google Workspace** | Productivity | ✅ | WF1, WF2, WF4, WF5 |
| **Document Understanding** | Document Understanding | ✅ | WF2 |
| **System** | Workflow | ✅ | All — control flow, DataTables, logging |
| **UiPath GenAI Activities** | Integration Service | ✅ *(preview)* | WF3 |
| **WebAPI** | Developer | ✅ | Optional — JSON handling, HTTP |
| **Anthropic Claude / OpenAI / Azure OpenAI / Google Vertex** | Integration Service | ✅ | Alternative LLM path for WF3 |
| **Persistence** | Workflow | ✅ | Optional — long-wait patterns |
| **Testing** | Workflow | ✅ | Optional — verification activities |

---

## 7. Activity list by workflow

**Confidence key:** [Verified] = confirmed in UiPath docs during research. [Check] = expected to exist but verify the exact name in your own Studio Web palette, since cloud releases change naming.

---

### 7.1 WF0 — `Portal_DownloadDeliveryNotes` · UI automation · Leo

**Package: UI Automation** [Verified available in Studio Web]

| Activity | Purpose in WF0 |
| --- | --- |
| `Use Application/Browser` [Check] | Open and scope the supplier portal browser session |
| `Navigate To` / `Go to URL` [Check] | Load the portal login page |
| `Type Into` [Check] | Enter username and password |
| `Click` [Check] | Submit login; click each delivery-note download link |
| `Check App State` [Check] | Confirm login succeeded before proceeding — do not assume |
| `Get Text` [Check] | Read the list of available notes, or confirm a success banner |
| `Extract Table Data` [Check] | If the portal lists notes in a table, pull filenames in one go |
| `Take Screenshot` [Check] | Capture failure state for the catch block |
| `Highlight` [Check] | Optional — visually mark elements during the demo recording |

**Then verify the download landed** (System package, §7.6):
`Path Exists` → `If` → `Throw` a BusinessRuleException if the file is missing.

**Alternative/fallback if browser download proves unreliable:**
Google Workspace `Download File` (Drive) — see §7.2. Decide this fallback in week 1.

---

### 7.2 WF1 — `Intake_RegisterDeliveries` · Data automation · Karthik

**Package: Google Workspace** [Verified — all activities below confirmed in UiPath docs]

**Google Forms activities — this improves the original design:**

| Activity | Purpose |
| --- | --- |
| `Get Form Response List` | Read all recipient demand submissions |
| `Get Form Info` | Read the form's question structure |
| `Form Response Created` *(trigger)* | **Fire the automation on submission** rather than polling the responses sheet |
| `Form Response Updated` *(trigger)* | Handle a recipient amending their request |

**Google Sheets activities:**

| Activity | Purpose |
| --- | --- |
| `Read Range` | Load `Demand`, `RecipientProfile`, `MatchingRules` into DataTables |
| `Write Range` | Bulk-write new Inventory rows |
| `Write Row` | Append a single inventory record |
| `Write Cell` | Set `Processed = TRUE` on a demand row (idempotency guard) |
| `Read Cell` | Read a single config value |
| `Read Column` / `Read Row` | Targeted reads |
| `For Each Row in Spreadsheet` | Iterate demand rows |
| `Add Sheet` | Optional — create a per-run log tab |
| `Create Spreadsheet` | Optional — one-time setup |

**Google Drive activities:**

| Activity | Purpose |
| --- | --- |
| `Download File` | Pull the delivery-note PDF into the local working folder |
| `Get File List` | Enumerate PDFs in the intake folder |
| `For Each File or Folder` | Iterate downloaded notes |
| `File or Folder Exists` | Guard before reading |
| `Get File or Folder Info` | Filename, size, modified date |
| `Move File` | Move processed PDFs to an archive folder |
| `Create Folder` | Set up the archive folder |
| `Upload Files` | Optional — push artifacts back to Drive |
| `Share File or Folder` | Optional — grant marker access |

**Useful triggers/persistence** (available if you want event-driven rather than scheduled):
`File Created`, `Row Added to the Bottom of a Sheet`, `Cell in Sheet Updated`,
`Wait for File Created and Resume`, `Wait for Row Added to the Bottom of a Sheet`.

---

### 7.3 WF2 — `Extract_DeliveryNoteData` · PDF extraction · Karthik

**Package: Document Understanding** [Verified — this is the corrected activity set]

| Activity | Purpose |
| --- | --- |
| **`Extract Document Data`** | **The core activity.** Points at a Document Understanding project configured in Automation Cloud; returns extracted fields with confidence scores. This replaces the whole Digitize → Data Extraction Scope → Extractor chain from Studio Desktop |
| `Classify Document` | Confirm the PDF really is a delivery note before extracting |
| `Extract PDF Text` | Raw text fallback if structured extraction underperforms |
| `Get PDF Page Count` | Sanity check — a multi-page note may hold several items |
| `Extract PDF Page Range` | Split a multi-item note if needed |
| `Extract PDF Images` | Not needed here, but available |
| `Merge PDFs` | Not needed here |
| `Set PDF Password` | Not needed here |

**Human-in-the-loop — available in Studio Web:**

| Activity | Purpose |
| --- | --- |
| `Create Validation Task and Wait` | Route low-confidence extractions to a human, block until resolved |
| `Create Validation Task` | Create the task without blocking |
| `Wait for Validation Task and Resume` | Long-running pattern |
| `Create Classification Validation Task` / `...and Wait` | Human check on document type |

**[Certain] Design note.** The confidence gate is the interesting part for Part 4. Read `ExtractionConfidenceThreshold` from the config tab; anything below it goes to `Create Validation Task and Wait` or gets flagged `Extraction Review` and excluded from matching. Never let low-confidence data reach the agent silently.

**[Verified] Known limitation from the docs:** if you use any "Wait for" activity that suspends the workflow while holding a `DataTable` variable, that DataTable must be serializable. A DataTable initialised as `new System.Data.DataTable` is **not** serializable and will fail. Either leave the default value empty or name it: `new System.Data.DataTable("MyTable")`. This will catch you out if you use validation tasks — worth knowing before you hit it.

---

### 7.4 WF3 — `Agent_AllocateSupply` · Agentic automation · Rohit

Two viable paths. Confirm with your module lead which counts as "agentic automation" under the brief, since the brief specifies *"an agent with tools, context, and a prompt."*

**Path A — UiPath Agents / Agent Builder** [Check — verify availability in your tenant]
The brief's wording maps most directly onto a UiPath Agent with defined tools. Build the tools as separate workflows and register them with the agent. **Verify this is enabled in your tenant during week 1** — if it is not, Path B is your answer.

**Path B — UiPath GenAI Activities** [Verified available in Studio Web, marked *preview*]
Prompt-based LLM calls with your deterministic tools implemented as ordinary workflows the LLM's output is validated against. Still defensible as agentic if the agent has tools, context and a prompt you can explain — but be honest in Part 2 that the orchestration is yours, not a managed agent runtime.

**[Certain] Preview-package risk.** UiPath GenAI Activities is flagged preview in the docs. Preview packages change without the stability guarantees of GA releases. Test early; do not discover a breaking change during integration week.

**Alternative LLM connectors, all ✅ Studio Web:** Anthropic Claude, Open AI, Microsoft Azure OpenAI, Google Vertex, Amazon Bedrock. Any of these can serve as the reasoning call if the GenAI pack misbehaves.

**Deterministic tools — build these as plain workflows, not LLM calls:**

| Tool workflow | Activities used |
| --- | --- |
| `GetExpiringInventory` | Sheets `Read Range` → `Filter Data Table` → `Sort Data Table` |
| `GetEligibleRecipients` | Sheets `Read Range` → `Filter Data Table` (hard constraints) → `For Each Row` |
| `ScoreAllocation` | `Assign` with the weighted expression, weights from `MatchingRules` |
| `CheckPickupFeasibility` | `Assign` / date arithmetic against pickup windows and expiry |

**Output handling:** `Deserialize JSON` (WebAPI package, ✅ Studio Web) [Check] to parse the agent's structured response, then the six validation guards before any write.

---

### 7.5 WF4 — `Notify_Recipients` · Email automation · Kyle

**Package: Google Workspace — Gmail activities** [Verified]
**Remember: the Mail package is ❌ in Studio Web. These are your email activities.**

| Activity | Purpose |
| --- | --- |
| `Send Email` | The introduction/collection email to each recipient |
| `Get Email List` | Optional — read replies or confirmations |
| `For Each Email` | Optional — process confirmations |
| `Reply to Email` | Optional — thread confirmations |
| `Download Email Attachments` | Optional — if suppliers email notes instead of using the portal |
| `Apply Gmail Labels` | Tag sent notifications for traceability |
| `Mark Email as Read/Unread` | Inbox hygiene |
| `Move Email` | Archive processed mail |
| `Get Newest Email` | Optional |

**Microsoft 365 equivalent** (also ✅ Studio Web) if your team is on Outlook: `Send Email`, `Get Email List`, `For Each Email`, `Reply to Email`, `Download Email Attachments`.

**Guard:** check `EmailSent = FALSE` before sending; set it `TRUE` in the same iteration. A re-run that re-emails every recipient is an embarrassing thing to have happen on camera.

---

### 7.6 WF5 — `Reconcile_UpdateInventory` · Data automation · Kyle

**Google Sheets** `Write Cell`, `Write Range`, `Write Row`, `Read Range`, `For Each Row in Spreadsheet` — decrement quantities, update statuses, increment `CumulativeReceivedKg`, write the `AuditLog` row.

**System package** [Verified ✅ Studio Web] — used across every workflow:

| Category | Activities |
| --- | --- |
| Control flow | `Sequence`, `If`, `Switch`, `While`, `Do While`, `For Each`, `Break`, `Continue`, `Parallel` [Check] |
| Error handling | `Try Catch`, `Throw`, `Rethrow`, `Retry Scope` [Check] |
| Variables | `Assign`, `Multiple Assign` [Check] |
| DataTables | `Build Data Table`, `Add Data Row`, `Filter Data Table`, `Sort Data Table`, `Merge Data Table`, `For Each Row in Data Table`, `Get Row Item`, `Output Data Table`, `Clear Data Table`, `Remove Data Row`, `Lookup Data Table`, `Join Data Tables` [Check] |
| Files | `Path Exists`, `Read Text File`, `Write Text File`, `Copy File`, `Move File`, `Delete`, `Create Folder` [Check] |
| Logging | `Log Message`, `Comment`, `Add Log Fields` [Check] |
| Orchestration | **`Invoke Workflow File`** — how the master project assembles WF0–WF5 |
| Timing | `Delay` [Check] |
| Text | `Modify Text`, `Text to Left/Right`, `Matches`, `Replace` [Check] |
| Cloud | `Get Asset` / `Get Credential` [Check] — Orchestrator assets, if you use them for portal credentials |

---

## 8. Activity → feature → owner summary

| Feature (brief) | Workflow | Primary package | Anchor activities | Owner |
| --- | --- | --- | --- | --- |
| UI automation | WF0 | UI Automation | `Use Application/Browser`, `Type Into`, `Click`, `Check App State` | Leo |
| Data automation | WF1, WF5 | Google Workspace | `Read Range`, `Write Range`, `Write Cell`, `Download File`, `Get Form Response List` | Karthik, Kyle |
| PDF extraction | WF2 | Document Understanding | **`Extract Document Data`**, `Classify Document`, `Create Validation Task and Wait` | Karthik |
| Agentic automation | WF3 | GenAI Activities / Agents | Agent or GenAI call + four deterministic tool workflows | Rohit |
| Email automation | WF4 | Google Workspace (Gmail) | `Send Email`, `Apply Gmail Labels` | Kyle |
| *(orchestration)* | Main | System | `Invoke Workflow File`, `Try Catch`, `Log Message` | Alex |

---

## 9. Verify these in week 1 — before anyone builds

| # | Item | Why | Owner | Done ? (Yes/No) |
| --- | --- | --- | --- | --- |
| 1 | Is **UiPath Agents / Agent Builder** enabled in your tenant? | Decides Path A vs Path B for WF3 | Rohit |  |
| 2 | Can you create a **Document Understanding project** in Automation Cloud, and does `Extract Document Data` reach it? | The entire WF2 design depends on this | Karthik |  |
| 3 | Do **browser file downloads** work reliably in Studio Web, and where do they land? | Highest-risk item in the whole build | Leo — **by 31 Aug** |  |
| 4 | Has **every member** authorised their own Google Workspace connection? | Connections do not transfer with a shared project | Alex |  |
| 5 | Confirm the **GenAI Activities** preview package is stable in your tenant | Preview packages change without notice | Rohit |  |
| 6 | Ask the module lead: does the 5-feature cap count **categories or instances**? | WF1 + WF5 are both Data automation | Alex |  |

---

## 10. Things to say on camera that earn marks

- **"We had to redesign our extraction step."** Studio Web does not carry IntelligentOCR, so the Digitize → Extract chain we planned did not exist. We moved to `Extract Document Data` against a Document Understanding project. That is a real, specific difficulty with a real fix — exactly what Part 4 asks for.
- **"The Mail package isn't available in Studio Web,"** so email runs through Google Workspace's Gmail activities. Shows you understand the platform, not just the happy path.
- **The fairness rule.** "Our problem statement says whoever messages first gets the food. So we put a fairness-debt term in the allocation score." A design decision that answers the problem you named.
- **Config-driven weights.** Change `FairnessWeight` in the sheet, re-run one item, show a different allocation. Proves the build is configurable, not hardcoded.
- **Rejected alternatives.** "We considered Orchestrator queues and rejected them — they add failure surface without serving a single-run process."

---

## 11. Sources

- UiPath Activities overview (package availability matrix, incl. the Studio Web column):
`https://docs.uipath.com/overview/other/latest/overview/activities-overview`
- Google Workspace activities package:
`https://docs.uipath.com/activities/other/latest/productivity/about-google-gsuite-activities`
- Document Understanding activities package (incl. the serialisable-DataTable limitation):
`https://docs.uipath.com/activities/other/latest/document-understanding/about-the-documentunderstanding-package`

Everything marked **[Check]** should be confirmed against your own Studio Web palette before you rely on it. Package availability is documented; individual activity naming shifts between cloud releases.

Adversarial Review — Food Donation Matching Plans

## Master Build Plan | UiPath Studio Web

**Module:** Enterprise Automation Build · **Team:** 6 · **Deadline:** Friday 18 September 2026, 2359

> This document supersedes the earlier expanded-design note. Take it to the 29 August meeting as the
thing being voted on. Every date in §6 assumes the build starts 30 August.
> 

> **Verify activity names against your own Studio Web palette.** Cloud releases change connector and
activity naming. Where behaviour is version-dependent it is flagged inline.
> 

---

## 1. Process framing

### 1.1 The organisation and the process

A food rescue coordinator sits between food suppliers (supermarkets, distributors, F&B outlets) with near-expiry stock they cannot sell, and recipient organisations (shelters, soup kitchens, family service centres) that need food. Singapore has real operators in this space — Food from the Heart, The Food Bank Singapore — which makes the client credible without inventing one.

The coordinator's daily process today, done by hand:

1. Log into each supplier's portal and download the day's delivery notes
2. Read each note and record item, quantity and expiry into a stock list
3. Work out which items are close to expiring and still safe to donate
4. Recall or look up which recipients want what, and which can physically accept it (halal-only, no cold storage, limited collection capacity)
5. Decide who gets what, and how much, when supply cannot cover demand
6. Pick a collection slot that works for the recipient and beats the expiry date
7. Message each recipient — usually WhatsApp or email — with what they are getting and when
8. Update the stock list so the same item is not promised twice

### 1.2 Why it is worth automating

- **Steps 1-3 are pure clerical repetition.** Same actions, every day, no judgement.
- **Step 5 degrades under time pressure.** The observed real-world pattern is first-come-first-served: whoever messages the coordinator first gets the stock, regardless of need. That is an allocation failure, not a communication failure.
- **Steps 7 and 8 are separate manual actions from step 5.** Any of them can be delayed or skipped, which is how stock gets double-promised and how recipients are left without status.
- **The deadline is absolute.** Unlike an invoice or a job application, the work item physically spoils. A missed window is not a delay, it is a total loss.

### 1.3 Trigger and end state

**Trigger:** the scheduled daily run — new delivery notes are available on the supplier portal.

**End state:** every recipient with a satisfied allocation has received an email stating exactly what they are collecting, in what quantity, and when and where. The inventory sheet reflects allocated and remaining quantities. Anything expiring with no eligible recipient is flagged at-risk rather than left silently unallocated.

### 1.4 Manual cost — state assumptions, not conclusions

Present the arithmetic on camera and label it as an assumption. A grader who asks where the number
came from must get a real answer.

| Manual step | Assumed daily time |
| --- | --- |
| Portal login and download | 5 min |
| Reading notes and recording stock | 15 min |
| Identifying near-expiry items | 8 min |
| Matching to recipient needs | 20 min |
| Deciding collection slots | 10 min |
| Messaging recipients | 12 min |
| Updating records | 8 min |
| **Total** | **~78 min/day ≈ 6.5 hrs/week** |

Add the qualitative point that matters more than the hours: allocation currently tracks *who asked first*, not *who needs most*. The automation changes the allocation rule itself, not just its speed.

### 1.5 Scope guardrail

5-6 delivery notes, 3-4 recipient organisations, one run. Small enough for a four-minute demo, large enough that the allocation logic must actually decide something.

### 1.6 Data declaration

All delivery notes, recipient profiles and demand records are **created by the team**. No real supplier, charity, or personal data is used. State this in the video and in the GenAI declaration.

---

## 2. Architecture

### 2.1 Feature mapping (5 categories — at the cap)

| Brief's feature | Where used |
| --- | --- |
| UI automation | WF0 |
| PDF extraction (Document Understanding) | WF2 |
| Data automation | WF1, WF5 |
| Agentic automation | WF3 |
| Email automation | WF4 |

WF1 and WF5 are the same category used twice. **Confirm with the module lead whether the cap counts categories or instances** — if instances, fold WF5's writes back into WF4 and WF1.

### 2.2 End-to-end flow

```
                    ┌─────────────────────────────────┐
                    │  Main.xaml  (orchestration)     │
                    │  Try / Catch / Finally          │
                    └─────────────────────────────────┘
                                   │
  ┌────────────────────────────────┼────────────────────────────────┐
  ▼                                                                 │
┌───────────────────────────────────────────────────────────────┐   │
│ WF0  Portal_DownloadDeliveryNotes          [UI AUTOMATION]    │   │
│  • Open supplier portal in browser                            │   │
│  • Type Into → username, password; Click → Login              │   │
│  • Click each delivery-note download link                     │   │
│  • VERIFY each file exists on disk before returning           │   │
│  out_lstPDFPaths : List<String>                               │   │
│  out_intDownloadCount : Int32                                 │   │
└───────────────────────────────────────────────────────────────┘   │
                                   ▼                                │
┌───────────────────────────────────────────────────────────────┐   │
│ WF1  Intake_RegisterDeliveries             [DATA AUTOMATION]  │   │
│  • For each PDF → create Inventory placeholder row            │   │
│    ItemID = INV-{yyyyMMdd}-{seq}, Status = Pending Extraction │   │
│  • Read Demand tab where Processed = FALSE                    │   │
│  • Set Processed = TRUE  ← idempotency guard                  │   │
│  in_lstPDFPaths / out_dtPendingItems / out_dtOpenDemand       │   │
└───────────────────────────────────────────────────────────────┘   │
                                   ▼                                │
┌───────────────────────────────────────────────────────────────┐   │
│ WF2  Extract_DeliveryNoteData              [PDF EXTRACTION]   │   │
│  • Digitize Document → Data Extraction Scope                  │   │
│  • Fields: ItemName, Category, QuantityKg, ExpiryDate,        │   │
│            DietaryTags, DeliveryNoteRef                       │   │
│  • Confidence < threshold → Status = Extraction Review,       │   │
│    excluded from matching (NOT silently passed through)       │   │
│  • Write fields to the item's Inventory row                   │   │
│  in_dtPendingItems / out_dtExtractedInventory                 │   │
└───────────────────────────────────────────────────────────────┘   │
                                   ▼                                │
┌───────────────────────────────────────────────────────────────┐   │
│ WF3  Agent_AllocateSupply                  [AGENTIC]          │   │
│  • Filter inventory to DaysToExpiry <= ExpiryWindowDays       │   │
│  • Sort ascending by DaysToExpiry (most urgent first)         │   │
│  • Per item: hard-constraint filter → score → agent decides   │   │
│  • Validate output, then write Allocations rows               │   │
│  in_dtExtractedInventory / in_dtOpenDemand                    │   │
│  out_dtAllocations / out_dtUnallocated                        │   │
└───────────────────────────────────────────────────────────────┘   │
                                   ▼                                │
┌───────────────────────────────────────────────────────────────┐   │
│ WF4  Notify_Recipients                     [EMAIL AUTOMATION] │   │
│  • One email per recipient, grouping all their allocations    │   │
│  • Body: items, quantities, pickup slot, location, reasoning  │   │
│  • Guard on EmailSent = FALSE; set TRUE in same iteration     │   │
│  in_dtAllocations / out_intEmailsSent                         │   │
└───────────────────────────────────────────────────────────────┘   │
                                   ▼                                │
┌───────────────────────────────────────────────────────────────┐   │
│ WF5  Reconcile_UpdateInventory             [DATA AUTOMATION]  │◄──┘
│  • Decrement QuantityRemainingKg per allocation               │
│  • Update Status: Available / Partially / Fully Allocated     │
│  • Increment RecipientProfile.CumulativeReceivedKg            │
│  • Flag unallocated near-expiry stock as Unallocated-AtRisk   │
│  • Write AuditLog row                                         │
│  in_dtAllocations / in_dtUnallocated / out_strRunSummary      │
└───────────────────────────────────────────────────────────────┘
```

### 2.3 Why not Orchestrator queues

Queues are the right pattern for production dispatcher/performer separation at volume. **For this project they are not.** The brief asks for one process running end to end in a four-minute demo; queue infrastructure adds moving parts, adds a second thing that can fail on camera, and demonstrates nothing the rubric asks for. Mentioning in Part 2 that you *considered and rejected* queues, with that reasoning, is worth marks under "what other options you considered."

### 2.4 Main.xaml orchestration

```
Main.xaml
  1. Read Config          → MatchingRules tab into Dictionary<String,String>
  2. Try
       Invoke WF0  → out_lstPDFPaths
       If out_intDownloadCount = 0 → Log "No deliveries today", exit clean
       Invoke WF1  → dtPendingItems, dtOpenDemand
       Invoke WF2  → dtExtractedInventory
       If dtExtractedInventory.Rows.Count = 0 → Log, exit clean
       Invoke WF3  → dtAllocations, dtUnallocated
       If dtAllocations.Rows.Count > 0 → Invoke WF4
       Invoke WF5  → strRunSummary
     Catch (BusinessRuleException)
       Log message, write AuditLog row with ErrorsCaught, exit gracefully
     Catch (Exception)
       Log full detail, screenshot if available, write AuditLog, rethrow
     Finally
       Close browser, write run-complete AuditLog entry
```

**Distinguish the two exception types deliberately** — a business exception (no eligible recipient, no stock expiring today) is a valid outcome and should exit cleanly. A system exception (portal down, Sheets connector fails) is a genuine error. Explaining this distinction in Part 4 demonstrates understanding rather than a canvas tour.

---

## 3. Google Sheet schema

One team-owned spreadsheet. Set link sharing to **"Anyone with the link can view"** before submission —
the brief states an automation pointing at an unopenable sheet cannot be marked.

### Tab: `Inventory`

| Column | Type | Written by | Notes |
| --- | --- | --- | --- |
| ItemID | Text | WF1 | `INV-20260918-001` |
| DeliveryNoteRef | Text | WF2 | Traceability to source PDF |
| SourcePDFName | Text | WF1 |  |
| ItemName | Text | WF2 |  |
| Category | Text | WF2 | Dry / Chilled / Frozen / Fresh |
| DietaryTags | Text | WF2 | Semicolon-delimited: `Halal;Vegetarian` |
| QuantityKg | Number | WF2 | As delivered |
| QuantityRemainingKg | Number | WF2 → WF5 | Decremented on allocation |
| ExpiryDate | Date | WF2 | **ISO `yyyy-MM-dd` only** |
| Status | Text | WF1 → WF5 | Pending Extraction / Extraction Review / Available / Partially Allocated / Fully Allocated / Unallocated-AtRisk |
| ExtractionConfidence | Number | WF2 | Min across extracted fields |
| RegisteredAt | DateTime | WF1 |  |

### Tab: `RecipientProfile` (standing attributes, registered once)

| Column | Type | Notes |
| --- | --- | --- |
| RecipientID | Text | `REC-001` |
| OrgName | Text |  |
| ContactEmail | Text |  |
| AcceptedCategories | Text | `Dry;Chilled;Fresh` |
| HasColdStorage | Boolean | **Hard constraint** for Chilled/Frozen |
| DietaryRequirements | Text | `Halal only` / `Vegetarian only` / `No restriction` |
| PickupWindowStart | Time | e.g. `09:00` |
| PickupWindowEnd | Time | e.g. `17:00` |
| MaxCapacityKgPerPickup | Number | Prevents overwhelming a small shelter |
| CumulativeReceivedKg | Number | **Drives the fairness rule.** Incremented by WF5 |
| Active | Boolean |  |

Separating these from per-request demand is the fix for the original proposal's biggest data flaw: cold-storage capability and dietary rules are properties of the organisation, not of a single request.

### Tab: `Demand` (Google Form responses)

| Column | Source |
| --- | --- |
| Timestamp | Form |
| RecipientID | Form |
| FoodTypeNeeded | Form |
| QuantityNeededKg | Form |
| NeededByDate | Form |
| Notes | Form |
| **Processed** | **Added manually.** WF1 sets TRUE — prevents reprocessing |

### Tab: `Allocations` (decision record)

| AllocationID | RunID | ItemID | ItemName | RecipientID | OrgName | QuantityAllocatedKg | PickupDateTime | PickupLocation | AgentReasoning | AgentConfidence | Status | EmailSent | SentTimestamp |

`AgentReasoning` is both your audit trail and the body text of the notification email.

### Tab: `MatchingRules` (config — zero magic numbers in the build)

| RuleKey | Value | Purpose |
| --- | --- | --- |
| ExpiryWindowDays | 7 | Only items expiring within this are matched |
| MinAllocationKg | 5 | Floor — smaller isn't worth a pickup trip |
| MinAllocationPercent | 20 | Floor as % of the request |
| UrgencyWeight | 50 |  |
| FairnessWeight | 30 |  |
| NeedFitWeight | 20 |  |
| MinDaysBeforeExpiryForPickup | 1 | Pickup must beat expiry |
| ExtractionConfidenceThreshold | 0.70 |  |
| PickupLocation | Central Warehouse, 12 Jurong East St 21 |  |
| WarehouseWindowStart / End | 08:00 / 18:00 |  |

Reading config from a sheet is a real best practice and a strong Part 4 moment — change `FairnessWeight` live on camera and re-run to show a different allocation.

### Tab: `AuditLog`

| RunID | RunTimestamp | PDFsDownloaded | ItemsExtracted | ExtractionFailures | ItemsExpiring | AllocationsMade | TotalKgAllocated | UnmetDemandCount | UnallocatedAtRisk | EmailsSent | ErrorsCaught | RunStatus |

---

## 4. Agent design

The brief singles this out as the feature teams most often underbuild. The failure mode is one free-text prompt with no tools. Avoid it.

### 4.1 Division of labour

**Deterministic code does arithmetic and hard constraints. The agent resolves competing objectives.**

Everything the agent could get wrong by hallucinating — eligibility, scores, capacity limits — is computed before it ever sees the data.

### 4.2 Tools

```
GetExpiringInventory(withinDays)
  → Items with DaysToExpiry <= withinDays AND Status = "Available"
     AND QuantityRemainingKg > 0
  → Sorted ascending by DaysToExpiry
  → Returns: ItemID, ItemName, Category, DietaryTags,
             QuantityRemainingKg, ExpiryDate, DaysToExpiry

GetEligibleRecipients(itemID)
  → Applies HARD constraints only — anything returned is already eligible:
      • item.Category IN recipient.AcceptedCategories
      • recipient.HasColdStorage = TRUE if Category IN (Chilled, Frozen)
      • item.DietaryTags compatible with recipient.DietaryRequirements
      • recipient has open, unfulfilled demand for this food type
      • recipient.Active = TRUE
  → Returns: RecipientID, OrgName, QuantityNeededKg,
             MaxCapacityKgPerPickup, CumulativeReceivedKg,
             PickupWindowStart, PickupWindowEnd

ScoreAllocation(itemID, recipientID)
  → Deterministic, weights read from MatchingRules:
      urgencyScore   = (ExpiryWindowDays - DaysToExpiry) / ExpiryWindowDays * UrgencyWeight
      needFitScore   = MIN(QuantityRemainingKg / QuantityNeededKg, 1) * NeedFitWeight
      fairnessScore  = (1 - recipient.CumulativeReceivedKg / MAX(all recipients' CumulativeReceivedKg)) * FairnessWeight
      total          = urgencyScore + needFitScore + fairnessScore
  → Returns the total AND all three components separately, so the agent
    can reason about *which* factor is driving a score

CheckPickupFeasibility(recipientID, expiryDate)
  → Valid slots satisfying ALL of:
      • inside recipient.PickupWindowStart–End
      • inside WarehouseWindowStart–End
      • at least MinDaysBeforeExpiryForPickup before expiry
  → Returns a list of valid datetime slots (empty if none exist)
```

Returning score *components* rather than a single number is the design choice that makes the agent's reasoning non-trivial. It can say "I prioritised this recipient because their fairness debt was high even though urgency favoured the other" — which a single scalar cannot support.

### 4.3 Agent prompt

```
ROLE
You allocate near-expiry food stock to recipient organisations on behalf
of a food rescue coordinator. Hard eligibility constraints are already
applied — every recipient you are shown can legally and physically
accept the item. Scores are computed for you; do not recalculate them.

INPUT (per item)
- The item: name, category, quantity remaining, expiry date, days to expiry
- Eligible recipients, each with: open demand quantity, max pickup capacity,
  cumulative kg received to date, and a score broken into urgency /
  need-fit / fairness components
- Valid pickup slots per recipient

YOUR DECISION
Allocate the available quantity across one or more eligible recipients,
or decline to allocate this item.

RULES — these are absolute
- Total allocated must never exceed QuantityRemainingKg.
- No recipient may receive more than MaxCapacityKgPerPickup.
- No allocation below MinAllocationKg or below MinAllocationPercent of
  that recipient's request. An allocation too small to justify a
  collection trip is waste, not fairness.
- Every pickupDateTime must be one of the slots supplied for that
  recipient. Do not invent a slot.

RULES — judgement
- Prefer allocating the full quantity over leaving stock unallocated.
  Expired stock helps nobody.
- Splitting one item across two recipients is allowed and often correct.
- When two recipients score within 10 points, prefer the one with the
  higher fairness debt (lower CumulativeReceivedKg).
- When an item expires in 1 day, prefer a single recipient who can take
  the whole quantity over a split that risks a missed collection.
- Decline only when no allocation can clear the minimum thresholds.

OUTPUT — return ONLY this JSON object, no prose, no markdown fences:
{
  "itemID": "<string>",
  "allocations": [
    {
      "recipientID": "<string>",
      "quantityKg": <number>,
      "pickupDateTime": "<yyyy-MM-dd HH:mm>",
      "reasoning": "<1-2 sentences naming the deciding factor>"
    }
  ],
  "unallocatedQuantityKg": <number>,
  "declineReason": "<string or null>",
  "confidence": "HIGH" | "MEDIUM" | "LOW"
}
```

### 4.4 Why this is genuinely agentic

Urgency and fairness pull against each other, and the scoring function cannot resolve that — the weights *encode* the tension, they do not settle it. The residual judgement is the agent's: split a batch or keep it whole, favour the urgent expiry or the underserved org, decline a marginal allocation or force it through.

**Seed your test data so this conflict happens on camera.** An agent that only ever sees clean 1-to-1 matches will look decorative no matter how well it is built.

### 4.5 Validation guards — do not skip these

```
After the agent returns, BEFORE writing anything:

1. SUM(allocations.quantityKg) <= item.QuantityRemainingKg
      → fail: log, retry once, then route to Manual Review
2. Every recipientID exists in the eligible list passed in
      → LLMs occasionally return plausible IDs that were not in the input
3. Every quantityKg <= that recipient's MaxCapacityKgPerPickup
4. Every quantityKg >= MinAllocationKg
5. Every pickupDateTime is in the slot list returned by CheckPickupFeasibility
6. JSON parses and contains every required key
      → fail: retry once with a stricter reminder, then Manual Review
```

Never branch a workflow on unvalidated LLM output. Say this in Part 4 — it is exactly the kind of design reasoning the rubric asks for.

---

## 5. Team split (6 members)

### Phase 1 — build (22 Aug – 6 Sept)

| Member | Owns | Feature | Why them |
| --- | --- | --- | --- |
| **Rohit** | WF3 Agent_AllocateSupply | Agentic automation | Hardest and most novel piece |
| **Alex** | **Integration owner** — master project, Main.xaml, naming contract, end-to-end testing, config tab | Orchestration | The role that protects the 25% Integration mark |
| **Leo** | WF0 Portal_DownloadDeliveryNotes + building the mock portal itself | UI automation | Highest-risk item, needs a dedicated owner |
| **Karthik** | WF1 Intake + WF2 Extraction | Data automation + PDF extraction | Two tightly coupled workflows, one owner avoids a handoff |
| **Kyle** | WF4 Notify + WF5 Reconcile | Email automation + Data automation | Two lighter workflows, balanced against Karthik's pair |
| **Harry** | Test data and test-case design, AuditLog, demo scenario scripting | — | See below |

### Why Harry's role is not filler

Someone must author the 5-6 delivery notes, 3-4 recipient profiles and the demand rows, **and design the cases that force interesting decisions**. Required scenarios:

1. A **contested item** — two eligible recipients, insufficient supply. Forces the fairness rule.
2. An item with **no eligible recipient** — proves the at-risk exception path.
3. An item that **must be split** — quantity exceeds one recipient's capacity.
4. An item expiring **tomorrow** — tests the pickup-feasibility constraint.
5. One **deliberately messy PDF** — tests the extraction confidence gate.

Without this, your demo will happen to contain only easy matches, and 25% of the marks evaporate.

### On the original role split

Alex's plan put four members on deck and video, two on UiPath. That places 65% of the rubric on two people and leaves four idle from 29 Aug to 7 Sep, since deck and video cannot start before the build is stable. Phase 2 below returns everyone to those roles — the work just moves later.

### Phase 2 — deck, video, submission (7 – 18 Sept)

| Member | Role |
| --- | --- |
| Leo | PPT deck lead (Canva) |
| Karthik | PPT deck, writes Part 1 framing |
| Kyle | Video edit lead |
| Harry | Video edit, delivers Part 5 reflection |
| Alex | Records the demo run, presents Part 2 design |
| Rohit | Presents Part 4 technical walkthrough |

All six must appear and speak. Suggested split:

| Part | Time | Speaker |
| --- | --- | --- |
| 1 — Process and problem | 1.5 min | Karthik |
| 2 — Design and diagram | 2 min | Alex, handing to Rohit for the agent rationale |
| 3 — Demonstration | 4 min | Leo (portal), Karthik (extraction), Rohit (allocation), Kyle (emails and reconcile) |
| 4 — Technical walkthrough | 1.5 min | Rohit |
| 5 — Reflection | 1 min | Harry |

### Naming contract — frozen 29 Aug, before anyone builds

```
Workflows
  Main.xaml
  WF0_PortalDownload.xaml      WF1_IntakeRegister.xaml
  WF2_ExtractDeliveryNote.xaml WF3_AgentAllocate.xaml
  WF4_NotifyRecipients.xaml    WF5_ReconcileInventory.xaml

Arguments — prefix in_ / out_ / io_, then type hint, then PascalCase
  in_dtOpenDemand, out_dtAllocations, in_strItemID,
  out_intEmailsSent, in_dictConfig

DataTable column names — exactly as §3. No renaming, no spaces.

Dates    ISO  yyyy-MM-dd
DateTime      yyyy-MM-dd HH:mm
Quantities    decimal kg, 1 dp
IDs           INV-{yyyyMMdd}-{000}, REC-{000}, ALLOC-{yyyyMMdd}-{000}

Config    every threshold read from MatchingRules. Zero magic numbers.
```

The brief names mismatched arguments as the single most common cause of breakage when parts are joined. Write this into a shared doc and treat it as frozen.

---

## 6. Build order and timeline

### 22 – 29 Aug — decide and set up

| Owner | Task |
| --- | --- |
| All | **29 Aug: lock the idea. No further pivots.** |
| Alex | Create master project, share with all; publish the naming contract |
| Alex | Create spreadsheet with all six tabs; create the Google Form; link responses to the `Demand` tab |
| Leo | Build the mock supplier portal (static page, GitHub Pages or Google Sites) with a login form and PDF download links |
| Harry | Draft the 5-6 delivery-note PDFs and 3-4 recipient profiles covering all five scenarios in §5 |
| All | Each member creates their own private Studio Web project and **authorises their own Google connections** |

**Gate:** naming contract frozen, sheet live, portal live. Nothing downstream works without these.

### 30 Aug – 6 Sept — individual builds

| Owner | Milestone | Due |
| --- | --- | --- |
| Leo | **WF0 logs in and downloads a PDF to a known path** | **31 Aug — hard deadline** |
| Karthik | WF1 creates Inventory rows and reads Demand with the Processed guard | 3 Sept |
| Karthik | WF2 extracts all six fields from 5 of 6 test PDFs | 5 Sept |
| Rohit | WF3 tools return correct values; agent returns valid JSON on 5 test items; all six validation guards pass | 6 Sept |
| Kyle | WF4 sends grouped emails; WF5 decrements correctly and writes AuditLog | 5 Sept |
| Alex | Main.xaml skeleton with try/catch and config reader | 3 Sept |
| Harry | All test data final and loaded; test cases documented | 2 Sept |

**Gate (6 Sept):** every workflow runs standalone against the shared sheet.

### 7 – 10 Sept — integration

| Day | Activity |
| --- | --- |
| 7 Sept | Alex adds WF0-WF2 to master, links via Invoke Workflow File, tests the chain |
| 8 Sept | Adds WF3-WF5; first full end-to-end run |
| 9 Sept | Fix name mismatches, data-type errors, date-format errors. **Budget more time here than the individual builds took.** |
| 10 Sept | **GATE: three consecutive clean end-to-end runs.** Recording does not start before this. |

### 11 – 18 Sept — record, edit, submit

| Day | Activity |
| --- | --- |
| 11–13 Sept | Record demo run and speaking parts. Record in segments so one fluffed line does not cost the take. Deck built in parallel by Leo and Karthik |
| 14–16 Sept | Edit to **9:30–10:30** (leave margin; outside 9-11 min is penalised) |
| 17 Sept | Upload YouTube as **Unlisted, not Private**. Export .uis and **open it once to confirm it works**. Zip supporting PDFs. Set sheet sharing to anyone-with-link-can-view. Put the YouTube link and sheet link in the text file. Each member signs their GenAI declaration |
| 18 Sept | **Submit by 2359** — not at 2350 |

### Studio Web lock protocol

Only Alex touches the master project outside booked slots. Anyone entering posts in the team chat and posts again on exit. Idle locks clear after 15 minutes. Alex downloads a dated backup at the end of every session into a shared folder.

---

## 7. Risks

| # | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| 1 | **Portal download unreliable in Studio Web** — file lands in an unpredictable path, click doesn't register, or the browser blocks it | High | Breaks the chain at step one | **Prove by 31 Aug.** Own the portal so selectors are stable. Verify file existence on disk before WF1 proceeds. Fallback: pre-place PDFs in Drive and have WF0 download from Drive instead — decide this fallback in week 1, not week 3 |
| 2 | **Over-allocation** — 30kg allocated from a 25kg item | Medium | Visible logic failure on camera | Deterministic sum check before any write. Re-read `QuantityRemainingKg` immediately before each allocation; never trust a cached value. Process items sequentially, never in parallel |
| 3 | **Extraction fails on layout variation** | Medium | Corrupts everything downstream | Standardise on two delivery-note layouts. Confidence gate at 0.70 routing to Extraction Review rather than passing bad data through |
| 4 | **Demo contains only easy 1-to-1 matches** — agent looks decorative | High if unmanaged | Loses the 25% feature mark | Harry owns test-case design. All five scenarios in §5 must appear in the run |
| 5 | **Date arithmetic errors** across expiry, days-to-expiry, pickup windows | Medium | Silent wrong allocations | One date format in the naming contract. Test explicitly with an item expiring tomorrow and one expiring today |
| 6 | **Agent returns invalid JSON or hallucinated IDs** | Medium | Silent bad data | All six validation guards in §4.5. Retry once, then Manual Review |
| 7 | **Studio Web lock contention** during 7-10 Sept | High | Schedule slip | Alex owns the master. Booking posted in chat. Nobody enters unbooked |
| 8 | **Per-person connections** — Sheets, Drive, Gmail do not transfer with a shared project | Certain | Workflow fails for everyone but the author | Every member authorises their own in week 1. This surprises teams during integration |
| 9 | **Duplicate emails on re-run** | Medium | Embarrassing on camera | Guard on `EmailSent = FALSE`; set TRUE in the same iteration as the send |
| 10 | **Video overruns 11 minutes** | Medium | Direct deduction | Target 10:00. Rehearse with a timer. Pre-cut any wait over 15 seconds and say on camera that you cut it |
| 11 | **Feature-count interpretation** — WF1 + WF5 might read as six instances | Low | Rubric risk | Ask the module lead in week 1. Folding WF5 into WF4 is small if done early |
| 12 | **A fourth pivot** | Real — three already considered | Fatal at this date | 29 Aug is the lock. After that, changes are to the build, not the idea |

---

## 8. What to say on camera that earns marks

- **The fairness rule.** "Our problem statement says whoever messages first gets the food. So we built a fairness-debt term into the allocation score — organisations that received less in prior cycles get priority." That is a design decision that answers the problem you named, which is exactly what the 15% design criterion wants.
- **The split allocation.** Show one batch divided across two recipients and let the agent explain why.
- **Live config change.** Change `FairnessWeight`, re-run one item, show a different allocation. Proves the build is configurable rather than hardcoded.
- **Rejected alternatives.** "We considered Orchestrator queues and rejected them — they add failure surface without serving a single-run process." Part 2 asks for what you considered.
- **The hard thing.** Portal download path handling, or the over-allocation guard. Both are real, both are explainable, neither is manufactured.
- **The honest reflection.** "We would have frozen the sheet schema and argument names before anyone opened Studio Web." Specific and credible — which is what the 5% reflection criterion rewards.