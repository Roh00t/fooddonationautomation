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