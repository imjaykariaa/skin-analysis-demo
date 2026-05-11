# Asaya × GlamAR — Recommendation Engine PRD
**Final Screen Product Recommendations | AI Skin Analysis Flow**

**Owner:** Jay Karia (GlamAR)
**For:** Asaya product, content, and tech team
**Status:** v1 — Draft for review
**Repo:** https://github.com/imjaykariaa/skin-analysis-demo-asaya

---

## 1. Context & Positioning

Asaya is a **hyperpigmentation-led skincare brand**. Every recommendation surfaced at the end of the AI Skin Analysis must reinforce this positioning. The AI scan detects 14+ parameters, but the recommendation engine must always anchor the routine in a product from the [Hyperpigmentation collection](https://worldofasaya.com/collections/hyperpigmentation), even when the user's primary scan concern is acne, dullness, or scars — because all of those concerns either cause or compound pigmentation in melanin-rich skin.

**Core principle:**
> *No matter what the AI detects, the routine sells the user one truth — "Asaya solves pigmentation, in every form, on every part of the body."*

---

## 2. Objectives

1. Build a clickable, scientifically-defensible routine that the user trusts.
2. **Always lead with a hyperpigmentation hero product** as the primary recommendation.
3. Surface **body-specific SKUs** (underarm, knee/neck/elbow, inner thigh, face) when the conversation extends past the face.
4. Prefer **Routine Sets** over individual SKUs whenever 2+ concerns are detected — higher AOV, simpler decision.
5. Filter out SKUs that conflict with the user's skin type (e.g. no salicylic acid for dry/sensitive).

---

## 3. Inputs to the Engine

| Source | What it provides |
|---|---|
| **AI Skin Scan** (GlamAR SDK) | **Score basics:** Overall Skin Score, Skin Type, Skin Tone, Skin Age. <br>**Key Conditions:** Acne, Wrinkles, Pores. <br>**Eye Area:** Eye Bags, Dark Circles. <br>**Specific Issues:** Post Acne Scars, Whiteheads, Pigmentation, Hydration. <br>*(8 concerns scored 0–100. Melasma is NOT detected — it is inferred from quiz only.)* |
| **Quiz answers** | Gender, pregnancy status, self-reported skin type, top goal (Fade pigmentation / Reduce acne / Hydration / Anti-ageing / Maintenance), breakout frequency, **+ new "Beyond the Face" question (§3.A)**. |

> ⚠️ **Reality check:** Many users (especially those with Asaya's target demographic) will have their **top 3 concerns score in the 60–80 range across Pores, Hydration, Wrinkles, or Acne** — none of which directly map to the Hyperpigmentation collection. In those cases the engine **MUST fall back to the bestseller pigmentation routine** (§6.A) so the user always sees an on-brand Asaya recommendation. We never want a user to leave screen 6 without a clear hyperpigmentation-first routine to buy.

---

### 3.A New Quiz Question: "Beyond the Face"

Add this as the **final question in the quiz** (after acne frequency). It's the only input to the Body Extra slot on the routine screen.

> **Question:** "Beyond your face, what's the one area you'd love to brighten?"
> *Asaya's body-care range tackles pigmentation head-to-toe.*
>
> **Options (single-select):**
> - 🩷 **Dark underarms** → Even Tone Underarm Mist (Asaya's #1 bestseller)
> - 🦵 **Dark knees, neck & elbows** → Tone Restore Knee-Neck-Elbow Cream
> - 👙 **Bikini line & inner thighs** → Face Forward *(SKU TBC by Asaya)*
> - ✨ **Just my face for now** → Default to Even Tone Underarm Mist as an upsell (bestseller-led)

**Why single-select:** users won't read or commit to multi-select on mobile. One clear pick = one clear add-to-cart card. Higher attach rate.

**Why we default to Underarm Mist if user skips:** it's the brand's #1 best-seller; even non-buyers will see it as social proof. The card label changes from "Recommended for your dark underarms" → "Asaya's #1 bestseller — try it on your underarms."

---

## 4. Concern → Product Category Map

The scan returns 8 concern scores 0–100. Lower = more severe. **A concern is "active" when its score is below the threshold below.** We classify each concern as either **Pigmentation-family (on-brand)** or **Adjacent (still solvable with Asaya, but not the hero category).**

| # | Detected Concern | Threshold | Family | Primary Job | Asaya SKU |
|---|---|---|---|---|---|
| 1 | **Pigmentation** | <70 | 🟫 Pigmentation | Brightening serum + corrective SPF | Advanced Spot Targeting Serum + Spot Light SPF |
| 2 | **Post Acne Scars** | <70 | 🟫 Pigmentation | Niacinamide + Alpha Arbutin serum + Spot Gel | 10% Niacinamide & 1% Alpha Arbutin Serum + Advanced Spot Targeting Gel |
| 3 | **Dark Circles** | <65 | 🟫 Pigmentation | Multi-Peptide serum + SPF | 5% Multi Peptide & HA Serum + Spot Light SPF |
| 4 | **Acne** | <70 | 🟨 Adjacent | Salicylic cleanser + spot gel + oil-free moisturizer | Salicylic Cleanser + Advanced Spot Targeting Gel + Oil-Free Barrier+ Crème |
| 5 | **Whiteheads** | <70 | 🟨 Adjacent | Salicylic serum + cleanser | 2% Salicylic Acid & 5% Niacinamide Serum + Salicylic Cleanser |
| 6 | **Pores** | <75 | 🟨 Adjacent | 10% Niacinamide + 1% Alpha Arbutin Serum | 10% Niacinamide & 1% Alpha Arbutin Serum |
| 7 | **Hydration** | <60 | ⬜ Neutral | Hydrating moisturizer + HA serum | Intense Moisture Gel + 5% Multi Peptide & HA Serum |
| 8 | **Wrinkles** | <70 | ⬜ Neutral | Multi Peptide serum + night cream | 5% Multi Peptide & HA Serum + Overnight Revive Crème |
| 9 | **Eye Bags** | <70 | ⬜ Neutral | (No targeted Asaya SKU — Multi Peptide as proxy) | 5% Multi Peptide & HA Serum |

**Family rules:**
- 🟫 **Pigmentation family** — top 3 concerns include at least one of these → use scan-driven routine (§6.B).
- 🟨 **Adjacent** — top 3 are all adjacent → still build a routine, but **inject the Hero Serum (Advanced Spot Targeting Serum) as the upsell** card on top.
- ⬜ **Neutral / no pigmentation flag at all** — fire the **Bestseller Fallback Routine** (§6.A).

---

## 5. Skin-Type Filter (Hard Gate)

Each SKU in the catalogue has a Yes/No flag for Oily, Dry, Normal, Combination, Sensitive, Acne-prone (per the spreadsheet). The engine **must drop** any SKU where the user's skin type = `No`.

**Hard exclusions to enforce in code:**
- **Dry skin** → exclude Salicylic Face Cleanser, Spot Light SPF, Sheer Milk SPF, Oil-Free Barrier+ Crème, Even Evermore Crème, Glow & Lift Crème, Overnight Revive Crème
- **Sensitive skin** → exclude Sheerscreen Mist, Sheer Milk Sunscreen, Aqua Dew Sunscreen, Intense Moisture Gel, 10% Vit C Serum, Moisture Cocoon, Salicylic Cleanser, 10% Niacinamide+AA, Salicylic Serum
- **Pregnant / postpartum** (from quiz) → exclude any serum containing **Salicylic Acid, Kojic Acid, or Retinoids**. Show the existing care banner. Default the brightening hero to **Radiance Serum** instead of Kojic Acid + Alpha Arbutin.

---

## 6. Routine Construction Logic

The final screen always renders a **4-step AM + 3-step PM routine**, plus optional body extras. The routine source depends on whether the scan turned up any pigmentation-family concerns.

### 6.0 Master Decision Flow

```
Run the scan + quiz.
Take the user's TOP 3 concerns (lowest 3 scores under threshold).

  ┌─────────────────────────────────────────────────────────┐
  │ Q1: Does ANY of the top 3 sit in the Pigmentation       │
  │     family? (Pigmentation, Post Acne Scars, Dark Circles)│
  └─────────────────────────────────────────────────────────┘
        │                                    │
        │ YES                                │ NO
        ▼                                    ▼
  ┌────────────────────────┐    ┌─────────────────────────────┐
  │ Q2: Does the quiz say  │    │ Q3: Quiz priority =         │
  │ priority = Fade        │    │ "Fade pigmentation"?        │
  │ pigmentation?          │    └─────────────────────────────┘
  └────────────────────────┘            │              │
        │                                │ YES          │ NO
        ▼                                ▼              ▼
  → §6.B Scan-Driven Routine    → §6.A Bestseller   → §6.A Bestseller
    (hero = top pigment concern)   Fallback +         Fallback
                                   inject scan
                                   concern as Step 2
```

**Bottom line:** the Bestseller Fallback (§6.A) is the **default** unless the scan explicitly returns a pigmentation-family concern in the top 3. This guarantees every user leaves screen 6 with a routine — even when the AI says "your skin is great" or detects only Hydration + Wrinkles + Pores.

---

### 6.A 🌟 Bestseller Fallback Routine *(default — always on-brand)*

Use this when no pigmentation concern is flagged, OR when the user's scan is "all green / good." This is **Asaya's hero starter routine** — the one a customer would buy if they walked into a store and said "give me your bestselling pigmentation set."

| Step | SKU | Why |
|---|---|---|
| **AM 1. Cleanse** | Even Tone Face Cleanser | Daily brightening cleanser — universally compatible |
| **AM 2. Treat** ⭐ | **Advanced Spot Targeting Serum** | Asaya's #1 face serum, hero of the hyperpigmentation collection — works as prevention even without active pigmentation |
| **AM 3. Moisturize** | Glow & Lift Crème | All-skin-types peptide moisturizer |
| **AM 4. Protect** | **Spot Light Sunscreen SPF 50** | Best-selling SPF in the hyperpigmentation category. Pigmentation prevention is the brand promise. |
| **PM 1. Cleanse** | Even Tone Face Cleanser | Repeat |
| **PM 2. Treat** | 5% Multi Peptide & HA Serum | Hydration + collagen, complements AM serum |
| **PM 3. Repair** | Overnight Revive Crème | Night repair |

**Always pin the Underarm Mist** as an upsell card at the bottom: *"Pair with Asaya's #1 best-seller — Even Tone Underarm Mist."*

---

### 6.B Scan-Driven Routine *(when pigmentation-family concern detected)*

Builds the same 4+3 scaffold but replaces specific slots based on the user's scan:

| Step | Default | Replace with… | When |
|---|---|---|---|
| **AM 1. Cleanse** | Even Tone Face Cleanser | Salicylic Face Cleanser | Acne <50 AND skin ≠ Dry/Sensitive |
| **AM 2. Treat (Hero)** | *from §7 decision tree* | — | — |
| **AM 3. Moisturize** | Glow & Lift Crème | Oil-Free Barrier+ Crème | Oily/Acne-prone |
| **AM 3. Moisturize** | Glow & Lift Crème | Intense Moisture Gel | Hydration <60 AND skin ≠ Sensitive |
| **AM 4. Protect** | Spot Light Sunscreen | Aqua Dew Sunscreen | Dry/Sensitive |
| **AM 4. Protect** | Spot Light Sunscreen | Hint of Tint | Quiz goal = Anti-ageing |
| **PM 2. Treat** | 5% Multi Peptide & HA | 15% Vitamin C Serum | Pigmentation 60–75 + skin ≠ Sensitive |
| **PM 2. Treat** | 5% Multi Peptide & HA | Repeat AM Hero | Single severe concern (one concern <50) |
| **PM 3. Repair** | Overnight Revive Crème | Moisture Cocoon | Dry skin |

If user has 2+ active pigmentation concerns → **upgrade the recommendation to a Routine Set** (see §9).

---

## 7. The "Hero Serum" Decision Tree *(only fires inside §6.B Scan-Driven Routine)*

This is the **single most important product** in the routine — it's the pigmentation anchor. Used only when the scan flagged a pigmentation-family concern. Otherwise the Bestseller Fallback (§6.A) hard-codes Advanced Spot Targeting Serum.

```
IF Pigmentation <60 OR (quiz: priority = Fade pigmentation AND Pigmentation <70):
   → Advanced Spot Targeting Serum  ⭐ hero (Asaya bestseller)

ELSE IF Post Acne Scars <70 AND skin type ∈ {Oily, Combination, Acne-prone}:
   → 10% Niacinamide & 1% Alpha Arbutin Serum

ELSE IF Pigmentation 60–80 AND skin type ∈ {Dry, Sensitive}:
   → 2% Kojic Acid & 1% Alpha Arbutin Serum  (gentler actives)

ELSE IF Dark Circles <65 (primary concern):
   → 5% Multi Peptide Complex & HA Serum

ELSE IF Skin age > biological age + 2 OR Wrinkles <70:
   → Radiance Serum (Korean brightening)

ELSE:
   → Advanced Spot Targeting Serum (always-on brand fallback)
```

**Pregnant users** (from quiz): hero is always **Radiance Serum** (no Kojic Acid, no Salicylic Acid). Show the existing pregnancy care banner.

---

## 8. Beyond the Face — Single Product Slot

Every user gets exactly **one body product** card on screen 6, picked from the answer to the new quiz question (§3.A). One card → one decision → higher attach rate.

| Quiz Answer | Recommended SKU | Headline | Subline |
|---|---|---|---|
| Dark underarms | **Even Tone Underarm Mist** | "For your dark underarms" | Asaya's #1 best-seller. Fades dark patches & controls odour. |
| Dark knees, neck & elbows | **Tone Restore Knee-Neck-Elbow Cream** | "For your knees, neck & elbows" | Targeted body pigmentation cream. |
| Bikini line & inner thighs | **Face Forward** *(SKU TBC)* | "For your bikini line & inner thighs" | Sensitive-zone brightening. |
| Just my face for now | **Even Tone Underarm Mist** (fallback) | "Asaya's #1 best-seller" | Try our bestselling body brightening spray. |

**Render position on screen 6:**
Sit below the AM/PM routine cards, before the "Shop My Routine" CTA, with the section header: **✨ Beyond the Face — Asaya goes head-to-toe.**

**Card style:**
Same `pc` card style as the face products, but with a small "BODY CARE" eyebrow label in gold instead of "Step X."

**Why we don't show 3 random products:**
We considered randomly picking 3 body SKUs but rejected it — it dilutes the decision and lowers attach rate. One targeted recommendation, tied to a user answer, will convert ~3× higher.

---

## 9. Set vs Individual SKUs — When to Bundle

A Routine Set always sits as the **primary CTA** on screen 6, because it's higher AOV and a cleaner decision. Pick the set based on the user's profile (scan + quiz):

| User Profile (after scan + quiz) | Recommended Set |
|---|---|
| **DEFAULT — no pigmentation concern flagged** | **Even Tone Restore Set** (3-step beginner routine, on-brand fallback) |
| Pigmentation <60 (severe) + quiz priority = Fade pigmentation | **Advanced Anti-Pigmentation Kit** ⭐ |
| Pigmentation <70 AND Post Acne Scars <70 | **3-Step Spot Diminishing Set** |
| Pigmentation OR Scars flagged + body opt-in (underarm/knee) | **Head to Toe Anti Pigmentation Set** ⭐ |
| Acne <60 + quiz priority = Reduce acne | **Anti-Acne Set** (then upsell pigmentation hero) |
| Quiz priority = Anti-ageing + Wrinkles flagged | **Festive Glow Set** |
| Quiz priority = Hydration + Hydration <60 | **Even Tone Restore Set** |
| Single severe concern (one score <50) | **Rapid Dark Spot Correction Set** |
| Travel intent / first-time / wants minis | **Travel Kit** |

**Display logic:** Show the set card as the primary CTA, with the same 4-step routine below labelled *"Or build your own routine →"*.

---

## 10. Scoring & Ranking (When Multiple SKUs Qualify)

When more than one SKU could fill a slot, rank by:

```
final_score =
    0.40 * concernMatch    // does it solve the detected concern?
  + 0.20 * skinTypeMatch   // skin-type compatibility (binary, 0 or 1)
  + 0.15 * hyperpigBoost   // +15 if SKU sits in the Hyperpigmentation collection
  + 0.10 * bestSellerBoost // +10 if marked "best seller" (Underarm Mist, Spot Targeting Serum)
  + 0.10 * priceFit        // user's stated budget vs SRP
  + 0.05 * noveltyBoost    // newly launched SKUs
```

Min display threshold: **60/100**. If no SKU clears 60 for a slot, hide the slot (don't show a weak rec).

---

## 11. Screen 6 (Routine) — UI Specification

```
┌─────────────────────────────────────┐
│  [← Back]                     [✕]   │   frame-bar
│                                     │
│       Your Asaya Routine            │   Playfair Display
│  Personalized for combination skin  │
│   targeting fade pigmentation       │
├─────────────────────────────────────┤
│                                     │
│  💎  RECOMMENDED SET                 │   eyebrow
│  ┌────────────────────────────┐    │
│  │  Anti Pigmentation Set      │    │   primary CTA
│  │  4 products • ₹4,499        │    │
│  │  [Add to Cart →]            │    │
│  └────────────────────────────┘    │
│                                     │
│  ── or build your own ──            │
│                                     │
│  ☀️ MORNING ROUTINE                  │
│  Step 1: Cleanse                    │
│  Even Tone Face Cleanser  ₹__  [▸] │
│  Step 2: Treat                      │
│  Advanced Spot Targeting Serum ⭐   │
│  Step 3: Moisturize                 │
│  Glow & Lift Crème                  │
│  Step 4: Protect                    │
│  Spot Light Sunscreen SPF 50        │
│                                     │
│  🌙 NIGHT ROUTINE                    │
│  ... (3 cards)                      │
│                                     │
│  ✨ BEYOND THE FACE                   │  (always shown — one card)
│  ┌────────────────────────────┐    │
│  │ BODY CARE                   │    │
│  │ Even Tone Underarm Mist     │    │
│  │ For your dark underarms     │    │
│  │ ₹__              [Shop Now] │    │
│  └────────────────────────────┘    │
│                                     │
│  [Shop My Full Routine →]           │
│  ₹X,XXX total                       │
└─────────────────────────────────────┘
```

**Every product card must show:**
1. Step number + role (Cleanse / Treat / Moisturize / Protect) — or "BODY CARE" for the Beyond the Face slot
2. Product name
3. Why it was picked — *one sentence tied to the user's scan or quiz answer* (e.g., "Picked because Pigmentation scored 35/100 on your scan." or "You said dark underarms are your top body concern.")
4. Price + Shop Now button (Shopify add-to-cart)

---

## 12. Data Pipeline / Tech Requirements

| Layer | Owner | What needs to happen |
|---|---|---|
| **Product catalog** | Asaya | Maintain master sheet (this one) as source of truth. Add `hyperpig_collection_member` (Y/N), `best_seller` (Y/N), `actives` (comma list), `srp_inr`, `shopify_handle`. |
| **Skin-type tags** | Asaya | Already done in spreadsheet. Push to `metafields` on Shopify so engine can read at runtime. |
| **API contract** | GlamAR | `GET /asaya/recommendations?scan_id=...&quiz=...` returns: `{ set: {...}, am_routine: [...], pm_routine: [...], body_extras: [...] }`. |
| **Fallback rules** | GlamAR | If catalog API is down, ship the demo with hardcoded JSON of the 4-step default routine. |
| **Analytics** | Asaya + GlamAR | Track: set CTR, individual product CTR, total cart value when routine is added, drop-off between screen 6 and checkout. |

---

## 13. Out of Scope (v1)

- Real-time inventory check (assume in stock; show out-of-stock state from Shopify on click)
- User account history / repeat-customer logic
- Multi-language / international SKU swaps
- Doctor-in-loop validation (the pregnancy banner is the only safety overlay)

---

## 14. Open Questions for Asaya

1. **"Face Forward"** for inner thigh / bikini line — confirm exact SKU name + Shopify handle. Not yet in the master sheet.
2. Confirm "best seller" tags — are **Even Tone Underarm Mist** and **Advanced Spot Targeting Serum** the official top two we should anchor the fallback routine on?
3. **Default fallback set** — agree that the **Even Tone Restore Set** is the right "no concerns detected" default? Or should it be **3-Step Spot Diminishing Set** (more aggressive sell)?
4. Budget bands — should we offer a *"value routine"* (3 products under ₹2,500) vs *"complete routine"* (4-step ₹4,500+)?
5. For pregnant users — is **Radiance Serum** confirmed pregnancy-safe by the Asaya formulation team?
6. **Set bundling pricing** — when we recommend a set vs individual products, what's the discount delta we can surface as "Save ₹X with the set"?
7. **Melasma confidence** — GlamAR does NOT detect melasma directly. We infer it from quiz response ("dark patches on cheeks"). Comfortable with this, or should we wait until GlamAR ships a melasma classifier?

---

## 15. Success Metrics

- ≥35% of users on screen 6 click "Shop My Routine" or "Add Set to Cart"
- ≥15% conversion from screen 6 to checkout
- Average order value ≥ ₹3,500 (vs ₹1,800 baseline on browse traffic)
- **"Beyond the Face" body card attach rate ≥25%** (always shown, single SKU, one-click add)
- **Quiz completion rate ≥80%** — adding one extra question (Beyond the Face) must not tank completion. Monitor drop-off on Q6.
