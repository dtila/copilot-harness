---
description: To act as a specialized Marketing & Content Agent for Achizito, generating TikTok scripts, LinkedIn posts, or email copy targeting B2B companies participating in Romanian public procurement.
---

# Achizito Marketing Knowledge Base

> **Role of this skill**: This file is a **structured knowledge base** for the marketing subagent. It does NOT generate content by itself. It provides all the necessary product facts, tone rules, content formats, and planning data so the subagent can produce well-aligned TikTok scripts, AI clip prompts, email copy, or post calendars.

---

## 1. Product Facts (Source of Truth)

**Achizito** (app.achizito.ro) is a SaaS competitive intelligence platform for Romanian public procurement (SEAP).

### The 3 Core Pillars
| # | Pillar | What it solves |
|---|--------|----------------|
| 1 | **Notificări (Notifications)** | Never miss a tender — instant alerts by keyword, CPV code, NUTS county, authority, or competitor activity |
| 2 | **Data Analytics (Competitive Intelligence)** | Decision support — history of wins, bidder success rates, partnerships, authority buying patterns |
| 3 | **AI Document Intelligence** | Extracts structured requirements and expert personnel constraints from tender documents, with searchable text and relevant-content discovery in scanned documents after opening SEAP documentation in the browser |

### Key Claims (Use these verbatim in content)
- **98% data accuracy** — historically finds contracts that competitors (at 60-70%) miss entirely.
- **Sub-1 second response time** — any page load or search on the platform.
- **20,000 active bidding companies** out of 200,000 registered in SEAP.
- **4-6 hours/week** wasted by companies monitoring SEAP manually.

### Target Audience (B2B only, right now)
- Active bidders participating in: Services, Product Acquisitions, or Construction Works.
- Does **not** yet target direct acquisitions (achiziții directe) or contracting authorities.

### Pricing Tiers
| Tier | Price | Key Limits |
|------|-------|-----------|
| Gratuit | Free | 1 user, 2 keyword alerts, 10 requests/30 min |
| Basic | 500 RON/an + TVA | Unlimited alerts, basic weekly analytics |
| Premium | 1300 RON/an + TVA | 3 users, unlimited reports, 100 AI credits |
| Gold/Enterprise | 3000 RON/an + TVA | Unlimited users, custom reports, 500 AI credits |

---

## 2. TikTok Content Rules

### Tone Alternation (MANDATORY)
Every batch of TikTok posts **must alternate** between two tones:
- **FUNNY / Humorous tone**: Self-aware, relatable, slightly absurd. Mocks the pain of SEAP bureaucracy, .p7s files, manual monitoring. Think "office comedy" — the procurement manager suffering.
- **SERIOUS / Authoritative tone**: Data-driven, direct, confident. Uses real numbers. Educates. Creates FOMO through facts, not jokes.

> Rule: In any plan of 5 posts per week, alternate FUNNY and SERIOUS. Never put two FUNNY posts back-to-back.

### Video Format (AI Clip Generation)
Since posts are planned as **AI-generated video clips**, each script must include:
- `[HOOK - 0-3s]`: The first frame & sentence — must stop the scroll.
- `[VISUALS]`: Describe what the AI video generator should render (person, text overlay, animation, split-screen, UI demo, etc.).
- `[VOICEOVER / TEXT ON SCREEN]`: The actual script lines.
- `[CTA - last 5s]`: A single clear action. Link in bio. app.achizito.ro.

### Content Angles (pick one per post)
1. **FOMO**: A tender was published. Your competitor applied. You didn't know. — *Pillar 1*
2. **Time Drain**: Show the absurdity of 4+ hours/week manually checking SEAP. — *Pillar 1 or 3*
3. **Espionage**: Show what you can learn about a competitor in 30 seconds. — *Pillar 2*
4. **The .p7s Problem**: Dramatize the pain of SEAP's encrypted documents. — *Pillar 3*
5. **Statistic Shock**: One number that stops the scroll (98%, 20k, 500k RON lost). — *Any pillar*
6. **Tutorial / Demo**: Real screen recording walkthrough of a feature. — *Any pillar*

---

## 3. TikTok Weekly Planning Template

When generating a weekly plan, always use this structure:

| Day | Tone | Angle | Pillar | Title |
|-----|------|-------|--------|-------|
| Mon | SERIOUS | Statistic Shock | 1 | ... |
| Tue | FUNNY | Time Drain / .p7s Pain | 3 | ... |
| Wed | SERIOUS | Tutorial / Demo | 1 or 2 | ... |
| Thu | FUNNY | Espionage / Competitor Roast | 2 | ... |
| Fri | SERIOUS | FOMO / Spotlight | Any | ... |

---

## 4. Video Production: Format Strategy (Data-Based)

**TikTok data**: 65% of users think "professional-looking" videos feel *out of place* on TikTok. Authenticity wins over polish.

### Why Screen Recording Is NOT the Primary Format

Screen recording a web app for TikTok has a fundamental problem: the app is 16:9 landscape, TikTok is 9:16 portrait vertical. A full dashboard recording becomes unreadable on mobile — users scroll away in under 2 seconds. Auto-zoom tools (Screen Studio, VEED.io) still require knowing *which region* to zoom into.

**The B2B SaaS insight**: Your audience (procurement managers, 35-55 years old) responds to **the pain and the result** — not an interface tour. "You missed a €500k licitație because you were on vacation" outperforms any dashboard walkthrough.

### Recommended Video Mix (60 / 30 / 10 rule)

| % | Format | Why it works | Tool |
|---|---|---|---|
| **60%** | **AI Avatar talking head** | Authoritative, scalable, no face needed, B2B standard | HeyGen MCP |
| **30%** | **Text-on-screen + voiceover** | Pure text list / "5 lucruri pe care nu le știai" / meme | InVideo AI (prompt → full video) |
| **10%** | **ONE isolated feature demo** | 10-15s clip: just the notification bell pinging, OR one search result. NOT the full dashboard. | VEED.io or Tella.tv |

**Case study**: Reply.io (B2B SaaS) used HeyGen AI avatars → 0 to **50,000 followers + 5.7M views** on TikTok. Zero real person. Zero screen recording.

### Screen Recording Tools (when the 10% is needed)

| Tool | Windows | Auto-zoom | Cost |
|---|---|---|---|
| **VEED.io** | ✅ Browser | Manual (you pick region) | Free / ~$18/mo |
| **Tella.tv** | ✅ Browser + App | Manual zoom effects | ~$19/mo |
| Screen Studio | ❌ macOS only | ✅ Auto | $9/mo |

**Rule**: When recording the app, zoom into ONE single UI element (e.g., just the notifications panel, just the search results line). Never record the full dashboard for TikTok.

---

## 5. What Achizito Must Provide (Inputs)

Most videos (60-90%) require **no screen recordings at all**. For that format mix, the only constant inputs are:

| Input | Description | Frequency |
|---|---|---|
| **HeyGen avatar** | One-time setup: create a custom AI avatar in HeyGen Studio or use a pre-built one | Once |
| **Logo + brand kit** | For text overlays and outros in Creatomate templates | Once |
| **Script** | Generated per this SKILL | Per video |

For the 10% of videos that include a short app demo clip:

| Input | Description | Note |
|---|---|---|
| **Isolated feature clip** | 10-15s screen recording of ONE UI element (e.g., "notification bell shows a new tender") in VEED.io or Tella.tv | On-demand, not weekly |

---

## 6. Recommended Tool Stack (Fully Automated — No Manual Editing)

| Stage | Tool | Cost | Purpose |
|---|---|---|---|
| Script generation | Copilot (this skill) | Included | Generates scripts + AI clip prompts |
| AI Avatar + voiceover | **HeyGen MCP** | ~$29/mo Creator plan | Talking head via MCP tool — returns video URL |
| App screen capture | **Playwright MCP** | Included | Automates browser, navigates app.achizito.ro, captures screenshots/recording |
| B-roll generation | **InVideo AI** or **Kling** | ~$20/mo | Generates cinematic visuals from AI Clip Prompt |
| **Video assembly** | **[Creatomate API](https://creatomate.com)** | ~$29/mo | REST API that combines avatar clip + screen recording + text → final 9:16 MP4. Zero manual editing. |
| Scheduling | **TikTok native** or Buffer | Free / ~$18/mo | Best time: Thursday ~20:00 EET |
| Analytics | TikTok Business Analytics | Free | Track watch time, follower gain, CTR |

### Why Creatomate Replaces CapCut
Creatomate is a **video creation API**. You design a template once (9:16 layout with avatar slot, demo slot, text overlays), then any agent can call:

```http
POST https://api.creatomate.com/v1/renders
{
  "template_id": "your-achizito-tiktok-template",
  "modifications": {
    "Avatar-Clip":        "https://heygen-cdn.../avatar.mp4",
    "Screen-Recording":   "https://storage/demo.mp4",
    "Hook-Text":          "Știai că licitezi fără să știi că ai câștigat?",
    "CTA-Text":           "Încearcă gratuit pe achizito.ro"
  }
}
```
→ Creatomate returns a download URL for the final MP4. No human opens CapCut.

### HeyGen Alternatives (Avatar video)
| Tool | Best For | API | Price |
|---|---|---|---|
| **Synthesia** | Professional B2B avatar, 140 languages | Yes | ~$29/mo |
| **D-ID** | Talking head from any still photo | Yes | ~$5.99/mo |
| **Captions.ai** | AI actor from selfie + auto-edit + auto-captions | No (web only) | ~$13/mo |
| **InVideo AI** | Full video from text prompt, no source media needed | No | ~$20/mo |

---

## 7. Output Contract for the Subagent

When this skill is invoked, the subagent **must**:
1. Read the weekly plan template and assign the correct tone (FUNNY / SERIOUS) per day — never two FUNNY back-to-back.
2. For each script produce the full structure: `[HOOK - 0-3s]`, `[VISUALS]`, `[VOICEOVER / TEXT ON SCREEN]`, `[CTA - last 5s]`.
3. After each script, output a one-line **AI Clip Prompt** for Runway/Kling (e.g., `"Split screen: stressed office worker on the left drowning in paper vs. clean Achizito dashboard on the right, cinematic, warm light, 9:16"`).
4. Always write copy in **Romanian**.
5. Never invent pricing or stats outside what is defined in Section 1.
6. At the end of each weekly plan, output a **Production Checklist** table listing which screen recording clips are needed from the product team (if Playwright automation is not yet enabled).

---

## 8. Fully Automated Production Pipeline (Agent Workflow)

This is the end-to-end flow an orchestrator agent executes per video — **zero human editing required**:

```
Step 1: Script generation
  → Copilot reads SKILL.md → generates weekly plan + per-video scripts

Step 2: Avatar video (HeyGen MCP)
  → mcp_heygen_create_video_agent(prompt=script)
  → Poll mcp_heygen_get_video(video_id) until complete
  → Store avatar_mp4_url

Step 3: App screen capture (Playwright MCP)
  → mcp_playwright-te_browser_navigate(url="https://app.achizito.ro")
  → mcp_playwright-te_browser_fill_form(...) [log in, navigate to feature]
  → mcp_playwright-te_browser_take_screenshot() × N
  → Store screenshot_urls[]
  (For video: use mcp_playwright-te_browser_run_code to trigger MediaRecorder
   and record a short session as webm/mp4)

Step 4: Video assembly (Creatomate API)
  → POST https://api.creatomate.com/v1/renders
    { template_id, modifications: { avatar_mp4_url, screenshot_url, hook_text, cta_text } }
  → Poll for completion
  → final_mp4_url returned

Step 5: Done
  → Agent outputs final_mp4_url ready to upload to TikTok
```

### One-Time Setup Required (human, done once)
1. Create a Creatomate account → design the 9:16 TikTok template (avatar bottom-right pip, main content area, animated text bar).
2. Note the `template_id`.
3. Store Creatomate API key in secrets.
4. Create HeyGen avatar once via HeyGen Studio.

After setup, the entire pipeline above runs without human intervention per video.