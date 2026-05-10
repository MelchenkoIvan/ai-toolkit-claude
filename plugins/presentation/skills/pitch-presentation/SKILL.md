---
name: pitch-presentation
description: >
  Builds a polished, audience-aware pitch deck as a structured Markdown file —
  cover, problem, solution, features, proof, demo, pricing, ask. Interviews
  the user for product details, picks a 12-slide template by audience type
  (investor / sales / internal), writes exact copy with speaker notes and
  visual directives. Use when the user says "create a pitch", "make a pitch
  deck", "build a presentation for <product>", invokes
  `/pitch-presentation`, or asks for slide content for a product or feature.
  Always use this skill when the user wants a pitch presentation.
---

# Pitch Presentation

Builds a complete pitch deck as a single Markdown file — slide-by-slide content with headlines, body, speaker notes, and visual directives. The Markdown can be pasted into any slide tool (Slides, Keynote, PowerPoint, Pitch, Slidev, Marp).

---

## Accepted Inputs

- **Topic / product name** — e.g., "sales pitch for Acme Tasks"
- **Brief description** — e.g., "pitch for investors for a B2B SaaS targeting small retailers"
- **Audience hint** — e.g., "for VCs", "for enterprise clients", "for internal stakeholders"

---

## Phase 1 — Information Gathering

Collect what's needed to write a compelling pitch. Ask **one or two questions at a time**, not all at once.

### Required information

| Field | Question |
|---|---|
| **Presentation language** | "What language should the presentation be in? (default: English)" — ask this **first**; it determines the language of every headline, body line, and speaker note. |
| **Product name** | "What's the name of the product or company?" |
| **One-liner** | "In one sentence, what does it do?" |
| **Target audience** | "Who is this pitch for — investors, prospective customers, internal stakeholders, or end users?" |
| **The problem** | "What pain point or problem does it solve?" |
| **The solution** | "How does your product solve it — what's the core mechanism or differentiator?" |
| **Key metrics / traction** | "Any numbers? (users, revenue, growth, retention — even early indicators)" |
| **Market size** | "Sense of market size? (TAM/SAM/SOM or a rough figure)" |
| **Competitors** | "Main competitors, and what makes you different?" |
| **Business model** | "How does it make money?" |
| **Ask / CTA** | "What do you want the audience to do after the deck? (invest, sign up, schedule a call, approve budget)" |
| **Brand colors** | "Brand colors? (hex codes or descriptions like 'deep blue and white')" |
| **Tone** | "Tone — bold & confident, clean & minimal, warm & human, or technical & data-driven?" |
| **Slide count** | "How many slides? (default: 12)" |
| **Output path** | "Where should I write the Markdown file? (default: `./pitch-<product>.md`)" |

> **Defaults if not provided:** 12 slides, clean & minimal tone, English, no brand colors specified (suggest a modern dark palette).

After gathering, summarize and confirm before generating:
> "Here's what I have: [summary]. Does this look right before I build the deck?"

---

## Phase 2 — Visual Asset Inventory (Optional)

Pitch decks are dramatically more persuasive with real product screenshots. Ask the user **if** they have visuals to include.

### Step 1 — Ask for assets

> "Do you have product screenshots, mockups, or logos I should reference in the deck? If yes, share file paths or descriptions. If no, I'll mark visual placeholders for each slide so you can drop assets in later."

### Step 2 — If the user has assets

For each one, record:

| Field | Description |
|---|---|
| **File path or URL** | Where the asset lives |
| **Caption** | Short human-readable label (e.g., "Manager Dashboard") |
| **Best slide use** | Which slide it fits |
| **Notes** | Cropped? In a specific language? Light/dark mode? |

### Step 3 — Build a visual asset map

```
Slide 1 — Cover            → <asset path or placeholder>
Slide 2 — Problem          → illustration / icon (no screenshot needed)
Slide 3 — Solution         → <asset path or placeholder>
...
```

Mark any unsupplied slot as `[PLACEHOLDER: <description of needed visual>]` so the user knows exactly what to add later.

### Step 4 — Screenshot presentation guidelines (for the user, included in the output)

When the user adds screenshots to a slide tool, suggest:
- Place mobile screenshots in a clean phone frame.
- Slight tilt (5–8°) for a dynamic look.
- Screenshots fill 35–50% of slide width — don't dominate.
- Soft drop shadow.
- For two-up layouts (e.g., admin + end-user views), use opposing tilts (+6° / -6°).

---

## Phase 3 — Slide Architecture

Pick the structure based on audience.

### Investor Pitch (VC / Angel)
1. **Cover** — Product name + tagline + presenter
2. **The Problem** — Pain with emotional hook
3. **The Solution** — Product as the answer
4. **Product Demo** — Screenshots / mockups
5. **Market Opportunity** — TAM / SAM / SOM
6. **Traction** — Key metrics, milestones, logos
7. **Business Model** — Revenue streams, pricing
8. **Competitive Landscape** — 2×2 matrix or comparison
9. **Go-to-Market** — Acquisition channels + strategy
10. **Financials** — 3-year revenue projection
11. **Team** — Founders + key hires with credibility signals
12. **The Ask** — Investment amount + use of funds + CTA

### Sales / Customer Pitch
1. **Cover** — Product name + tagline
2. **The Problem We Solve** — Their pain, not generic
3. **The Solution** — Product overview
4. **Key Features** — 3 hero features with visuals
5. **How It Works** — 3-step process / workflow
6. **Social Proof** — Testimonials / case study / logos
7. **Results You Can Expect** — ROI / outcomes
8. **Pricing Overview** — Tiers or "let's talk"
9. **Why Us vs. Alternatives** — Differentiation table
10. **Onboarding** — "Getting started is easy"
11. **Team / About Us** — Credibility
12. **Next Steps / CTA** — Clear action item

### Internal / Product Demo Pitch
1. **Cover** — Feature/product name
2. **Background / Context** — Why we're building this
3. **User Problem** — Real user quote or data
4. **Solution Overview** — What we built
5. **Demo Walkthrough** — Step-by-step screens
6. **Technical Architecture** *(optional)*
7. **Metrics / Success Criteria** — How we'll measure success
8. **Roadmap** — What's next
9. **Q&A** — Open discussion

If the user requested a different slide count, expand or collapse: keep cover + problem + solution + ask as anchors; cut/duplicate the middle.

---

## Phase 4 — Slide-by-Slide Content

For each slide, generate this exact block. **Write everything in the language selected in Phase 1.**

```
---
### Slide N: [TITLE]

**Headline:** [exact headline text — ≤8 words, punchy]
**Subheadline:** [supporting text — 1 line, optional]
**Body copy:** [bullet points or paragraph — exact text to display]
**Visual directive:** [what image/icon/chart/mockup goes here — reference asset map slot or placeholder]
**Data point:** [any number/stat to highlight in large type]
**Speaker notes:** [what the presenter says — 3-5 sentences]
**Animation cue:** [entrance effect, build sequence, or transition]
---
```

### Content writing rules

**Storytelling techniques:**
- **Problem slides:** "Before → After" or "Pain → Cost → Nightmare" framework.
- **Solution slides:** Lead with outcome, not features ("Your team ships 2× faster" not "Our platform has CI/CD").
- **Data slides:** One hero number per slide, displayed at 100pt+ in the slide tool.
- **Traction slides:** Show trajectory, not just snapshots ("10% MoM growth for 6 months").
- **CTA slides:** Make the ask specific and time-bound ("Schedule a 30-min call this week").

**Persuasion techniques:**
- **Social proof first** — credibility signals before claims.
- **Rule of 3** — group benefits, steps, features in threes.
- **The contrast principle** — show "without us" before "with us".
- **Loss aversion framing** — "Every month you wait costs you [X]" on urgency slides.
- **Brevity rule** — max 30 words of body copy per slide (Guy Kawasaki's 30-point rule).

**Language rule:**
All headlines, body, data callouts, and speaker notes must be written in the language selected in Phase 1. Do not mix languages.

---

## Phase 5 — Visual Design Brief

Generate a one-section design brief at the top of the Markdown so the user can apply it consistently in their slide tool.

### Color system

Suggest hex values based on tone + brand colors:

| Variable | Usage |
|---|---|
| `--primary` | Headlines, CTAs, accent |
| `--secondary` | Supporting elements |
| `--background` | Slide background |
| `--surface` | Cards, section backgrounds |
| `--text-primary` | Body copy |
| `--text-muted` | Subtext |
| `--accent` | Highlight boxes, data callouts |

**Palette recommendations by tone:**

| Tone | Suggested Palette |
|---|---|
| Bold & Confident | Deep charcoal + vivid electric blue + neon accent |
| Clean & Minimal | Pure white + slate gray + single brand-color pop |
| Warm & Human | Off-white + warm terracotta + forest green accents |
| Technical / Data | Dark navy/black + cyan or green monochrome + data-viz colors |
| Premium / Enterprise | Deep midnight blue + gold + platinum |

If the user supplied brand colors, use those for `--primary` and `--accent`; pick complementary neutrals for the rest.

### Typography

| Role | Recommendation | Size |
|---|---|---|
| Hero headline | Bold geometric (Inter Bold, Plus Jakarta Sans Bold, Syne ExtraBold) | 64-80pt |
| Section headline | Same family, SemiBold | 40-48pt |
| Body copy | Inter Regular, DM Sans | 18-22pt |
| Data callout | Bold display (Syne ExtraBold, Space Grotesk Bold) | 80-120pt |
| Caption | Inter Light | 14pt |

Use **maximum 2 font families** per deck.

### Layout principles

1. One idea per slide.
2. ~60% visual / 40% text.
3. Generous whitespace — minimum 15% margin.
4. Z-pattern reading flow — key info top-left, CTA bottom-right.
5. Data callouts — one big number per data slide.
6. Full-bleed imagery — edge-to-edge photos with gradient overlay for text.
7. Consistent grid — 12-column, content never breaks the grid.

### Animation directives

| Slide type | Animation |
|---|---|
| Cover / hero | Pan transition + Rise on headline |
| Problem | Fade + line-by-line build (Typewriter) |
| Solution reveal | Wipe — creates "reveal" drama |
| Data / metrics | Pop on the hero number |
| Feature slides | Slide in from left, one feature at a time |
| Team | Fade — professional |
| CTA / close | Subtle Breathe on the CTA button |

**Animation rules:**
- 0.4–0.6s duration max.
- Maximum 2 animation types per deck.
- No spinning, bouncing, spiral.
- Sound off.

---

## Phase 6 — Output

Write a single Markdown file to the path the user chose in Phase 1.

### File structure

```markdown
# <Product Name> — <Pitch Type>

> Generated by `/pitch-presentation`. Paste into Slides / Keynote / PowerPoint / Pitch / Slidev / Marp.

## Design System
<color palette + typography + animation summary from Phase 5>

## Visual Asset Map
<the asset map from Phase 2>

## Slides
<full slide-by-slide content from Phase 4>

## Power Techniques Reference
<short cheatsheet — see below>

## Notes
- Output language: <language>
- Audience: <audience>
- Tone: <tone>
- Estimated presentation time: <N> minutes (target ~90s per slide)
```

### Power techniques cheatsheet (include at the end of the output)

| Technique | Description | Apply to |
|---|---|---|
| The Hook | Open with a provocative stat or question | Slide 2 (Problem) |
| The Villain | Name the enemy clearly — inefficiency, waste, manual work | Problem slide |
| The Hero Moment | One slide where the product is revealed as the answer | Solution slide |
| Social Proof Cascade | Stack 3 proof types: metric + logo + quote | Traction slide |
| The 10× Frame | Show 10× better, not 10% better | Differentiation slide |
| Urgency Stack | "Why now" + "cost of inaction" + "momentum" | Market / timing slide |
| The Reverse Demo | Show outcome first, then how | Product slide |
| Name Drop Early | Put the most impressive customer/partner logo on slide 1 if you have one | Cover |
| The Cliffhanger | End each section with a question the next slide answers | All transitions |
| Ask Specificity | "$2M seed to hire 3 engineers and reach $1M ARR by Q4" beats "raise money" | CTA slide |

---

## Phase 7 — Delivery Summary

After writing the file, print a summary in the conversation:

```
🎯 <Product Name> Pitch Deck — Ready

### Overview
- File: <path>
- Slides: <N>
- Audience: <type>
- Tone: <selected>
- Language: <selected>
- Estimated presentation time: <N> minutes

### Quick Reference: Slide Titles
1. Cover — "<headline>"
2. The Problem — "<headline>"
...

### Visual Placeholders Remaining
- Slide N: <description> — drop your screenshot/mockup here
- Slide M: <description>

### Next Steps
- [ ] Import the Markdown into your slide tool of choice
- [ ] Replace each [PLACEHOLDER: ...] with the matching asset
- [ ] Apply the design system (colors, fonts) from the Design System section
- [ ] Practice: target ~90 seconds per slide
```

---

## Rules

1. **Always ask for the presentation language first.** Every line of slide copy is generated in that language.
2. **Confirm key details with the user** before generating full content (after Phase 1).
3. **Default to Markdown output** — write a single self-contained `.md` file. Do not generate HTML / Google Slides API calls / other binary formats unless the user explicitly asks for a different format.
4. **Mark visual placeholders explicitly** when the user has not supplied an asset for a slide.
5. **Apply the brevity rule** — max ~30 words of body copy per slide.
6. **Use audience-appropriate structure** — investor vs. sales vs. internal templates differ; pick the right one based on the user's stated audience.
7. **Never invent metrics / customer logos / claims** — if the user hasn't supplied them, mark slides as `[PLACEHOLDER: insert real number]` rather than fabricating.
8. **One file in, one file out** — the user gets a single Markdown file plus the conversational summary.
