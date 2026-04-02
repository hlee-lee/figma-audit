---
name: figma-audit
description: Audit a live website page and post findings as Figma comments on a screenshot in a Figma file. Use when the user says "audit", "site audit", "page audit", "UX audit", "review this page", "audit this site", or provides a URL alongside a Figma link for review purposes.
disable-model-invocation: true
argument-hint: "[site URL]"
---

# Figma Site Audit

Audit a live website page across four categories and post findings as **Figma comments** pinned to a screenshot in a Figma file.

---

## Core Principle

Every observation must tie back to the **page's user goal**. Don't describe what a module is — evaluate whether it helps or hinders the goal. Write like a senior product designer leaving notes for a colleague.

---

## Audit Categories

### Brand TOV
- Overall tone expression and consistency
- Recognition and trust signals
- Key messaging clarity
- Visual asset quality

### Design
- Brand system consistency
- Color, typography, art direction
- Layout variety and visual rhythm
- Motion and graphic elements

### Usability
- User friction and decision-making ease
- Navigation and flow clarity
- CTA affordance and visual cues
- Whether users can self-select or self-qualify without extra clicks

### Content
- Relevance to audience needs
- Educational vs promotional balance
- Terminology accessibility
- Asset diversity (text, image, video, UGC, interactive)

---

## Comment Delivery: Figma REST API

All audit notes MUST be posted as **Figma comments** via the REST API.

- Use `POST https://api.figma.com/v1/files/:file_key/comments`
- Use `curl` via Bash with the user's Figma personal access token
- The Plugin API (`use_figma`) does NOT support comments — do not use it for this skill
- If you do not have a token, **ask for it before proceeding**

---

## Step 0 — Gather Inputs (Blocking)

Collect all three before proceeding:

1. **Site URL** — the live page to audit
2. **Figma file URL** — the board containing the page screenshot (extract `fileKey` and `nodeId`)
3. **Figma personal access token** — required for posting comments

If any are missing, ask. Do NOT proceed without all three.

---

## Step 1 — Identify Page Goal (Blocking)

Ask:

"What is the primary goal of this page? For example: build trust and drive purchase, generate leads, educate and retain users, etc."

### Rules
- This goal anchors every observation in the audit
- If the user provides it upfront, confirm it back
- If unclear, suggest a goal based on the page type (homepage, PDP, landing page, etc.) and confirm
- Do NOT proceed until the goal is agreed upon

---

## Step 2 — Discover the Figma Layout

Before auditing, understand the screenshot placement:

1. Call `get_metadata` on the provided `nodeId` to find the screenshot image node (its ID, position, and dimensions)
2. Call `get_screenshot` on the image node to visually identify all modules and estimate their Y-positions within the image

Record:
- Parent frame ID (for page-level comments placed outside the screenshot)
- Screenshot image node ID (for module-level comments pinned on the screenshot)
- Image dimensions (width, height) for coordinate mapping

---

## Step 3 — Audit the Page

Fetch the live page URL using `WebFetch` with targeted prompts. Run multiple fetches in parallel to cover:

1. **Brand voice, tone, copy, trust signals**
2. **Design system, color, typography, layout, motion**
3. **Usability, navigation, CTAs, friction points**
4. **Content, SEO, educational value, asset types**

Use the page goal from Step 1 to frame analysis. Every finding should answer: "Does this help or hinder the user goal?"

---

## Step 4 — Map Modules to Coordinates

From the screenshot in Step 2, estimate the Y-position of each module within the image. Place comments at meaningful locations:

- Pin observations to the relevant area of the screenshot (use `node_offset` with x/y relative to the image node)
- Vary x-positions (left/center/right of image) to avoid comment overlap
- For page-level notes, pin to the parent frame with an x-offset to the right of the screenshot

---

## Step 5 — Write and Post Comments

### Module-Level Comments (pinned on the screenshot)

Each comment is **one focused observation, 1-2 sentences max**.

Use sentiment markers:
- `✅` — working well, supports the goal
- `⚠️` — opportunity, could better serve the goal
- `❌` — issue, actively hinders the goal

**Multiple comments per module are expected.** One positive, one opportunity, etc. — each its own pin.

#### Writing Rules
- One observation per comment — never combine multiple points
- Tie every note back to the page goal and the user's experience
- Don't describe what the module is — evaluate what it does for the user
- Don't make assumptions about the industry, competitors, or time period
- Don't reference technical implementation (HTML, CSS, ARIA, tabindex, schema)
- Write in plain, direct language — no jargon, no hedging
- Suggest specific improvements where relevant (e.g., "a benefit line like 'our #1 cream for dry skin' would help")

### Page-Level Comments (pinned on the parent frame, right of screenshot)

Three separate comments:

1. **Brand TOV** — One holistic note on the page's voice. Where it works, where it drops off, and how consistency would serve the goal. NOT per-module.

2. **Page Summary** — 2-3 short paragraphs. What the page does well, where it falls short, and the biggest structural observation (e.g., trust content buried too low, page assumes familiarity, etc.)

3. **Top Priorities** — 5 ranked, actionable recommendations. Each is 1-2 sentences. Framed around impact on the page goal.

---

## Step 6 — Confirm

After posting, summarize what was posted:
- Count of module-level comments by sentiment (✅ / ⚠️ / ❌)
- List of page-level notes posted
- Invite the user to review and adjust

---

## Comment Format Reference

### Module-level (on screenshot image node)
```
curl -s -X POST "https://api.figma.com/v1/files/:file_key/comments" \
  -H "X-Figma-Token: TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "⚠️ Observation text here — one point, 1-2 sentences.",
    "client_meta": {
      "node_id": "IMAGE_NODE_ID",
      "node_offset": {"x": 580, "y": 300}
    }
  }'
```

### Page-level (on parent frame, offset right)
```
curl -s -X POST "https://api.figma.com/v1/files/:file_key/comments" \
  -H "X-Figma-Token: TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "📋 Page Summary\n\nParagraph text here.",
    "client_meta": {
      "node_id": "PARENT_FRAME_ID",
      "node_offset": {"x": 1800, "y": 700}
    }
  }'
```

---

## Batch Posting

Post comments in parallel batches of 3-4 using `curl ... &` and `wait` to avoid rate limits while staying efficient. Run module batches first, page-level notes last.

---

## Editing Existing Audits

If the user asks to revise or redo:

1. Fetch all existing comments via `GET /v1/files/:file_key/comments`
2. Delete relevant comments via `DELETE /v1/files/:file_key/comments/:comment_id`
3. Repost revised comments

---

## Anti-Patterns

- Don't write multi-paragraph module comments — one point per pin
- Don't use category headers (Design, Usability, Content) in module comments
- Don't reference code, markup, or technical implementation
- Don't assume what competitors do or industry norms
- Don't describe what a module looks like — evaluate what it does for the user
- Don't add observations that don't connect to the page goal
- Don't post all comments as one batch — group into manageable parallel batches
