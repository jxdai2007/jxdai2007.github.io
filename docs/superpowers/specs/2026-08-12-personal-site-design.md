# Personal Site Redesign — Design Spec

Date: 2026-08-12
Status: Approved (design approved in brainstorming session; this doc pending user review)

## Goal

Deepen the existing minimal personal site for Jollen Dai (CS undergrad, UCLA, AI safety — unlearning robustness / intervention evaluation). Current site is structurally sound but content-thin. Bring it to parity with field-standard AI safety researcher sites without a visual redesign.

## Research basis

Surveyed 15 AI safety researcher personal sites (Olah, Nanda, Perez, Evans, Bowman, Christiano, Hubinger, Shlegeris, Shah, Leike, Hendrycks, Steinhardt, Krakovna, Hobbhahn, Casper). Findings:

- Minimal static sites dominate: plain HTML/Jekyll on GitHub Pages is the majority stack. No JS frameworks anywhere.
- Standard sections: first-person mission-driven bio, papers with links and one-line context, writing (usually links out to LessWrong/Substack), contact + profile links (Scholar, GitHub, X, LessWrong/Alignment Forum).
- Standard extras: photo (near-universal), CV PDF link, problem statement up top.
- Empty placeholders ("coming soon") appear on none of the surveyed sites.

Conclusion: current site already matches the structural standard. Gap is content depth — no photo, no papers section, empty Writing placeholder, no CV, missing X/LessWrong links.

## Decisions

- **Direction:** deepen minimal (keep style, add content). Rejected: designed portfolio, blog-centric.
- **Stack:** hand-written single `index.html` + inline CSS. No static site generator (YAGNI until a real blog exists).
- **Hosting:** GitHub Pages + custom domain (user purchases domain; CNAME + DNS steps provided).
- **Admonymous / anonymous feedback:** included as optional contact line.

## Page structure (single page, anchor sections)

1. **Header** — name, tagline, small photo. Links: GitHub, Scholar, LinkedIn, Email, CV (PDF), X, LessWrong.
2. **Intro** — current mission paragraph, unchanged.
3. **Now** — unchanged.
4. **Papers** — NEW. Two IEEE MIT URTC 2024 papers (linked) + REU group-bias paper marked "in preparation". One-line context per paper, not bare citations.
5. **Background** — kept, trimmed of anything that moved to Papers.
6. **Writing** — placeholder removed. One line linking LessWrong profile (+ Substack if exists).
7. **Contact** — kept; optional anonymous feedback link.
8. **Footer** — quiet "last updated" date.

## Styling

Current palette, typography, 40rem column, dark-mode media query all stay. Additions limited to: photo styling, papers list formatting, footer.

## Files

- `index.html` — edit in place
- `assets/` — photo, CV PDF (new)
- `CNAME` — new, custom domain
- `README.md` — new, one-liner

## Deployment

`git init` → GitHub repo → GitHub Pages (deploy from main) → custom domain via CNAME file + registrar DNS records (A records to GitHub Pages IPs, or CNAME to `<user>.github.io`).

## Testing

- Local browser: light mode, dark mode, ~375px mobile width
- Every link resolves (papers, profiles, CV, mailto)
- Post-deploy: live URL check, HTTPS enforced

## Inputs required from user at build time

- Photo file
- CV PDF
- Two IEEE URTC 2024 paper links (IEEE Xplore or arXiv)
- X profile URL, LessWrong profile URL (Substack if exists)
- Chosen domain name

## Out of scope

- Blog engine / static site generator
- Visual redesign, custom fonts, analytics
- Multi-page structure
