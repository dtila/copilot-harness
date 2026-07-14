---
name: marketing-landing-pages
description: Integrated marketing skill for public-facing landing pages - combines fact-based copywriting, SEO strategy, and conversion-optimized UI patterns using MudBlazor + Bootstrap.
---

This skill guides creation of marketing-effective landing pages that convert visitors through credible messaging and strategic UI design. Use for all public-facing pages (homepage, pricing, features, use cases).

## Core Principles

**Persuasion Philosophy**: Meek persuasion through verifiable facts, not hype
- Lead with measurable differentiators (98% accuracy, <1s response time, 200k companies monitored)
- Acknowledge pain points without fear-mongering ("Găsește licitații relevante mai rapid" vs "Nu mai pierde oportunități!")
- Understated confidence ("Platformă pentru achiziții publice" vs "Platforma #1 în România")
- No unverified claims (#1, "best", "revolutionary", "transform your business NOW")

**Target Audience**: Romanian companies participating in public procurement (200k registered, 20k active)
- SEAP participants (from first-time registrants to frequent winners)
- Procurement managers seeking competitive intelligence
- SMBs to enterprise needing procurement monitoring automation

**Product Context**: Achizito - SaaS for Romanian public procurement intelligence
- Data: SEAP + additional sources
- Core value: 98% accuracy vs 60-70% market average, <1s response time
- Freemium model: Free → Basic (500 RON/year + VAT) → Premium (1300 RON/year + VAT) → Enterprise (3000 RON/year)

**Reference Documentation**:
- [Marketing.md](c:\work\LicitatiiPublice\Marketing.md) - Complete product details, features, pricing, segmentation strategies, email campaign templates
- [SALES.md](c:\work\LicitatiiPublice\SALES.md) - Sales strategies, partnership models, email automation examples, customer objections and solutions

Consult these files for detailed context on product features, customer segments, conversion triggers, and messaging templates when creating landing page content.

## Content & Copywriting Guidelines

### Headline Patterns

**Hero Headlines** (Problem → Solution):
```
❌ "Transformă-ți afacerea cu licitații publice!"
❌ "Platforma #1 pentru achiziții publice"
✅ "Monitorizează 200,000 de companii înregistrate în SEAP"
✅ "Acuratețe de 98% în datele de achiziții publice"
✅ "Notificări automate pentru licitații relevante afacerii tale"
```

**Section Headlines** (Benefit-focused, specific):
```
❌ "Caracteristici incredibile"
❌ "Cea mai bună platformă"
✅ "Căutare avansată după autoritate contractantă sau ofertant"
✅ "Rapoarte despre evoluția contractelor atribuite"
✅ "Acces instant la documentația decriptată SEAP"
```

**Feature Headlines** (Functional, clear):
- State what it does: "Notificări automate când apar licitații noi"
- Include constraints honestly: "Filtrare după cuvinte cheie, CPV sau NUTS (maximum 2 filtre pentru planul gratuit)"
- Quantify where possible: "Răspuns sub 1 secundă pentru orice cerere"

### Value Proposition Structure

**Format**: [Specific benefit] + [Verifiable metric/fact]

Examples:
- "Identifică licitații relevante cu precizie de 98%, comparativ cu 60-70% media pieței"
- "Primește notificări când autoritățile selectate publică anunțuri noi"
- "Acces la documente SEAP decriptate direct în browser"
- "Monitorizează activitatea celor 200,000 de companii înregistrate"

**Avoid**:
- Vague benefits: "Creșteți șansele de câștig" → Replace with specific: "Urmărește contractele câștigate de competitori în industria ta"
- Unverifiable claims: "Cea mai rapidă" → Replace with: "Răspuns sub 1 secundă"
- Hype words: "Revoluționar", "Incredibil", "Transformă-ți afacerea"

### Call-to-Action (CTA) Hierarchy

**Primary CTA**: Free signup (freemium activation)
```
✅ "Începe gratuit" / "Crează cont gratuit"
✅ "Activează notificări gratuite"
❌ "ÎNCEARCĂ ACUM!" / "NU PIERDE ACEASTĂ OPORTUNITATE!"
```

**Secondary CTA**: Upgrade prompts (after establishing value)
```
✅ "Vezi planuri de abonament"
✅ "Compară planuri"
❌ "UPGRADE ACUM!" / "CUMPĂRĂ PREMIUM!"
```

**Tertiary CTA**: Enterprise consultation
```
✅ "Contactează-ne pentru soluții personalizate"
✅ "Solicită prezentare API"
```

**CTA Placement**:
- Hero section: Primary CTA prominent, secondary CTA subtle (text link)
- After feature sections: Contextual CTAs ("Vezi toate rapoartele disponibile")
- Before FAQ: Primary CTA repeat (trust established)
- Footer: All CTAs available

### Romanian Language Guidelines

**Tone**: Professional yet approachable
- Use "tu" form (informal) for familiarity, NOT "dumneavoastră" (too formal for SaaS)
- SEAP terminology acceptable (target audience knows it)
- Explain acronyms on first use: "CPV (Common Procurement Vocabulary)"

**Writing Style**:
- Short sentences (max 20-25 words)
- Active voice: "Primești notificări" vs "Notificări vor fi primite"
- Conversational but precise: "Găsește licitații" vs "Identifică oportunități de achiziții publice"

### SEO Strategy

**Long-tail Keyword Targeting**:
- Primary: "platformă monitorizare licitații publice SEAP"
- Secondary: "notificări licitații publice România", "rapoarte achiziții publice", "căutare autorități contractante"
- Intent-based: "cum monitorizez licitații SEAP", "analiza competitorilor licitații publice"

**Semantic Clusters** (group related terms):
- Procurement core: licitații publice, achiziții publice, SEAP, autoritate contractantă, ofertant
- Features: notificări, rapoarte, căutare avansată, CPV, NUTS
- Benefits: acuratețe, timp real, documentație decriptată

**On-Page SEO Requirements**:
- H1: One per page, include primary keyword naturally
- H2-H3: Semantic keyword variations, not repetitive stuffing
- Meta description: 150-160 chars, include primary keyword + benefit + CTA
- Image alt text: Descriptive + context (not keyword stuffing)
- Internal links: Link to feature pages, pricing, use cases
- Structured data: Use JSON-LD for Organization, SoftwareApplication, FAQPage

**Content Depth**:
- Homepage: 1500-2500 words (comprehensive but scannable)
- Feature pages: 800-1200 words (specific, detailed)
- FAQ: 10-15 questions covering objections + search intent

## UI & Conversion Patterns

### Layout Structure (MudBlazor + Bootstrap)

**Hero Section** (First screen, above fold):
```
Pattern:
- Left/Center: H1 headline + subheadline (1-2 sentences) + primary CTA + secondary CTA
- Right: Hero visual (screenshot, dashboard preview, data visualization)
- Background: Subtle gradient or light pattern (avoid solid white)

Bootstrap classes:
- Container: `container-lg py-5`
- Layout: `row align-items-center g-4`
- Headline: `display-4 fw-bold mb-3 lh-sm`
- Subheadline: `lead mb-4 text-muted`
- CTA group: `d-flex gap-3 flex-wrap`

MudBlazor:
- Primary CTA: `<MudButton Variant="Filled" Color="Color.Primary" Size="Size.Large">`
- Secondary CTA: `<MudButton Variant="Text" Color="Color.Default">`
```

**Social Proof Band** (Below hero):
```
Pattern:
- "Folosit de [X] companii" or "Monitorizăm [200k] companii înregistrate"
- Client logos (if available) OR key metrics (98% accuracy, <1s response)

Bootstrap: `bg-light py-4 border-top border-bottom`
Layout: `d-flex justify-content-center gap-5 flex-wrap align-items-center`
Typography: `text-muted small text-uppercase ls-wider`
```

**Feature Grid** (Icon + headline + description):
```
Pattern:
- 3-column grid on desktop, 1-column mobile
- Icon (MudIcon or custom SVG) + headline (H3) + 2-3 sentence description
- No "Learn more" links unless genuinely needed

Bootstrap:
- Container: `container py-5`
- Grid: `row g-4`
- Column: `col-md-6 col-lg-4`

Card structure:
<div class="text-center p-4">
  <MudIcon Icon="@Icons.Material.Filled.Notifications" Size="Size.Large" Color="Color.Primary" Class="mb-3" />
  <h3 class="h5 mb-3">Notificări automate</h3>
  <p class="text-muted mb-0">Primești alerte când apar licitații noi, filtrate după cuvinte cheie sau coduri CPV.</p>
</div>

Avoid:
- Heavy borders on cards (creates visual clutter)
- Background colors on every card (use subtle hover effects instead)
- Icons with no semantic meaning
```

**Comparison Table** (Free vs Paid plans):
```
Pattern:
- Sticky header on scroll
- Highlight recommended plan (subtle border or badge, NOT aggressive)
- Clearly mark limitations ("Max 2 filtre" vs "Nelimitat")

Use MudTable or custom HTML table:
- Bootstrap: `table table-borderless`
- Header: `bg-light text-center`
- Feature rows: Left-aligned feature name, center-aligned checkmarks/values
- CTA row: Bottom of each column

Avoid:
- "Most popular" badges in bright colors
- Strikethrough on features (confusing)
- Too many features listed (focus on differentiators)
```

**Trust Signal Section** (Testimonials, if available):
```
Pattern:
- Quote (max 100 words) + Name + Company + Role
- Headshot optional (adds credibility but not required)
- Focus on specific results: "Am identificat 15 licitații relevante în prima săptămână"

Bootstrap:
- Layout: `row g-4` with `col-md-6` or `col-lg-4`
- Card: `card border-0 shadow-sm h-100`
- Quote: `card-body` with `blockquote` element

Avoid:
- Generic praise ("Great platform!")
- Long testimonials (>100 words, users won't read)
- Testimonials without attribution (reduces credibility)
```

**FAQ Section** (Objection handling):
```
Pattern:
- Accordion (MudExpansionPanels) for scannability
- 10-15 questions covering: pricing, features, technical setup, security, support
- Questions in user's language: "Cum activez notificările?" not "What is notification setup?"

MudBlazor:
<MudExpansionPanels MultiExpansion="true">
  <MudExpansionPanel Text="Ce este SEAP și cum îl monitorizează Achizito?">
    <p>SEAP (Sistemul Electronic al Achizițiilor Publice) este platforma oficială...</p>
  </MudExpansionPanel>
</MudExpansionPanels>

Bootstrap alternative:
- Use `accordion` with `accordion-flush` for clean look
```

**Final CTA Section** (Before footer):
```
Pattern:
- Restate core value prop (different from hero, reinforcement)
- One clear CTA (primary from hero)
- Optional: Trust signal (security, privacy, support)

Background: `bg-primary text-white` or `bg-light` (not both on same page)
Layout: `text-center py-5`

Example:
<section class="bg-primary text-white py-5">
  <div class="container">
    <h2 class="display-5 mb-3">Începe să monitorizezi licitații gratuit</h2>
    <p class="lead mb-4">Acuratețe de 98%. Răspuns sub 1 secundă. 200,000 de companii monitorizate.</p>
    <MudButton Variant="Filled" Color="Color.Surface" Size="Size.Large">
      Crează cont gratuit
    </MudButton>
  </div>
</section>
```

### Spacing & Visual Hierarchy

**Section Spacing**:
- Between major sections: `py-5` (3rem top/bottom)
- Within sections: `mb-4` (1.5rem) between elements
- Tight groups: `mb-2` or `mb-3`

**Typography Scale**:
- H1 (Hero): `display-4` or `display-5` (large, attention-grabbing)
- H2 (Sections): `h2` or `h3` (clear hierarchy)
- H3 (Features): `h5` or `h6` (readable, not overpowering)
- Body: `p` with `lead` for important text, `text-muted` for secondary

**Color Usage**:
- Primary CTA: MudBlazor Primary color
- Backgrounds: `bg-light`, `bg-white`, subtle `bg-primary` for key sections
- Text: Default black, `text-muted` for secondary, NO custom grays
- Avoid: Heavy use of colored backgrounds (creates visual noise)

### Conversion Optimization Rules

**Above-Fold Checklist**:
- [ ] Clear value prop (What is it? Who is it for?)
- [ ] Primary benefit visible (98% accuracy, <1s response, specific feature)
- [ ] Primary CTA prominent (Free signup)
- [ ] Hero visual shows actual product (not stock photo)

**Trust Signal Placement**:
- Below hero: Metric band ("98% accuracy", "200k companies")
- Mid-page: Testimonials or case studies (if available)
- Before pricing: Security/privacy mention
- Footer: Support, privacy policy, terms links

**CTA Repetition Strategy**:
- Hero: 1 primary CTA
- After features: 1 contextual CTA ("Vezi toate funcțiile")
- Before FAQ: 1 primary CTA repeat
- Footer: All CTAs available
- NO aggressive pop-ups or sticky CTAs on mobile

**Mobile-First Considerations**:
- Stack hero image below headline on mobile (`order-2` class)
- Single-column feature grids (`col-12 col-md-6`)
- Simplify tables on mobile (consider card layout alternative)
- Touch-friendly CTA buttons (`Size.Large` on mobile)

### Accessibility & Performance

**Required**:
- Semantic HTML: Use `<section>`, `<article>`, `<aside>` appropriately
- Heading hierarchy: No skipping levels (H1 → H2 → H3)
- Alt text: Descriptive for images, empty for decorative
- Color contrast: WCAG AA minimum (4.5:1 for body text)
- Keyboard navigation: All interactive elements focusable

**Performance**:
- Lazy load images below fold: `loading="lazy"`
- Optimize images: WebP format, responsive sizes
- Minimize custom CSS: Prefer Bootstrap utilities
- Defer non-critical scripts

## Implementation Workflow

1. **Content First**: Write headlines, value props, feature descriptions before implementation
2. **Structure**: Create semantic HTML structure with Bootstrap grid
3. **Components**: Add MudBlazor components (buttons, icons, expansion panels)
4. **Refinement**: Adjust spacing, typography, colors using utilities
5. **Validation**: Check against conversion checklist, accessibility, mobile

## Anti-Patterns to Avoid

**Content**:
- ❌ Hype words: "revolutionary", "incredible", "transform your business"
- ❌ Unverified claims: "#1 platform", "best solution", "guaranteed results"
- ❌ Fear-mongering: "Don't lose opportunities!", "Your competitors are winning!"
- ❌ Aggressive CTAs: "BUY NOW!", "LIMITED TIME OFFER!"

**UI**:
- ❌ Heavy left/right border accents (unprofessional)
- ❌ Solid background colors on every section (visual chaos)
- ❌ Too many CTAs per section (decision paralysis)
- ❌ Stock photos of people (generic, inauthentic)
- ❌ Auto-playing videos or animations (annoying)
- ❌ Pop-ups on page load (anti-pattern for SaaS)

## Integration with Existing Skills

**Use `marketing-landing-pages` for**:
- Homepage (public landing page)
- Pricing page
- Feature showcase pages
- Use case/industry pages
- Any unauthenticated marketing pages

**Use `frontend-design` for**:
- Authenticated user dashboard
- Settings pages
- Internal tools
- Admin interfaces

**Use `frontend-design-creativity` for**:
- One-off campaigns (holiday, special promotions)
- Experimental features (A/B tests)
- Brand refresh explorations

This skill takes precedence for public-facing marketing pages.

## Validation Checklist

Before finalizing a landing page implementation:

**Content**:
- [ ] No unverified claims (#1, best, revolutionary)
- [ ] All metrics are factual (98% accuracy, <1s response, 200k companies)
- [ ] Headlines state clear benefits, not generic promises
- [ ] CTAs are encouraging but not aggressive
- [ ] Romanian "tu" form used consistently

**SEO**:
- [ ] One H1 with primary long-tail keyword
- [ ] H2-H3 use semantic variations
- [ ] Meta description 150-160 chars
- [ ] Alt text on all images
- [ ] Internal links to feature/pricing pages

**UI/UX**:
- [ ] Hero section has clear value prop + CTA above fold
- [ ] Feature grid uses 3-column on desktop, 1-column mobile
- [ ] Trust signals placed strategically (not cluttered)
- [ ] Spacing consistent (py-5 between sections)
- [ ] Mobile-friendly (tested on small screens)

**Conversion**:
- [ ] Primary CTA visible above fold
- [ ] CTAs repeated strategically (3-4 times max)
- [ ] Free signup emphasized over paid plans initially
- [ ] FAQ addresses common objections
- [ ] No aggressive pop-ups or sticky CTAs

**Accessibility**:
- [ ] Semantic HTML structure
- [ ] Proper heading hierarchy
- [ ] Color contrast meets WCAG AA
- [ ] All interactive elements keyboard-accessible
- [ ] Images have descriptive alt text
