# Figma Site Audit — Claude Code Skill

A Claude Code skill that audits a live website page and posts actionable UX findings directly as Figma comments on a screenshot in a Figma file. Focuses on user goals, clarity, and product thinking rather than surface-level critique.

## What It Does

- Walks you through a structured audit process to evaluate a live webpage against its primary user goal
- Audits across four categories: Brand TOV, Design, Usability, and Content
- Fetches and analyzes a live webpage
- Maps insights to specific areas of a screenshot in Figma
- Posts granular, pinned comments directly in Figma via the REST API
- Produces both module-level feedback and page-level summaries

## Requirements

- Claude Code
- Figma file with a full-page screenshot of the site
- Figma personal access token

## Installation

Copy the `.claude/skills/` folder into your project:

    # Clone this repo
    git clone https://github.com/hlee-lee/figma-audit.git

    # Copy the skill into your project
    cp -r figma-audit/.claude/skills/figma-audit your-project/.claude/skills/

Your project should look like:

    your-project/
    └── .claude/
        └── skills/
            └── figma-audit/
                └── SKILL.md

## Usage

In Claude Code:

    /figma-audit [site URL]

You'll be prompted to provide:

- Site URL (page to audit)
- Figma file URL (with screenshot)
- Figma personal access token

## Output

### Module-Level Comments

- Pinned directly on the screenshot
- One observation per comment
- Uses sentiment markers:
  - ✅ Working well
  - ⚠️ Opportunity
  - ❌ Issue

### Page-Level Comments

Posted to the right of the frame:

- Brand TOV — overall voice and messaging consistency
- Page Summary — strengths, weaknesses, structural insight
- Top Priorities — 5 ranked, actionable recommendations

## Core Principle

Every observation must answer:

> Does this help or hinder the page’s user goal?

## License

MIT
```

