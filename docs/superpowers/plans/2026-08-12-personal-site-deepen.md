# Personal Site Deepen Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bring Jollen Dai's minimal personal site to AI-safety-field standard: photo, Papers section, CV link, real Writing section, X/LessWrong links, GitHub Pages deploy with custom domain.

**Architecture:** Single hand-written `index.html` with inline CSS, edited in place. Static assets in `assets/`. Git → GitHub repo `jxdai2007.github.io` → GitHub Pages → custom domain via CNAME.

**Tech Stack:** Plain HTML/CSS. No framework, no build step. `git`, `gh` CLI, `curl`, `python3` (stdlib only) for link checking.

**Spec:** `docs/superpowers/specs/2026-08-12-personal-site-design.md`

## Global Constraints

- No JS frameworks, no static site generator, no build step. One page.
- Keep existing CSS variables, system font stack, 40rem column, dark-mode media query untouched.
- Placeholder copy ("coming soon", "TBD") never ships.
- Commit messages: plain, no `Co-Authored-By` lines (user rule).
- All new hrefs must resolve (Task 5 link check gates deploy).
- `SITE-DATA` values below refer to `assets/site-data.md` written in Task 1. Tasks 2, 3, 4, 7 substitute real values from that file — never commit literal `{{...}}` tokens.

---

### Task 1: Collect assets and record site data

**Files:**
- Create: `assets/photo.jpg` (user-provided)
- Create: `assets/cv.pdf` (user-provided)
- Create: `assets/site-data.md`

**Interfaces:**
- Consumes: nothing (first task; blocks on user input)
- Produces: `assets/site-data.md` with keys `X_URL`, `LESSWRONG_URL`, `SUBSTACK_URL` (optional), `ADMONYMOUS_URL` (optional), `PAPER1_URL`, `PAPER2_URL`, `DOMAIN`. Files `assets/photo.jpg`, `assets/cv.pdf`.

- [ ] **Step 1: Request inputs from user** (this task cannot proceed without them)

Ask the user for:
1. Photo file → save/convert to `assets/photo.jpg` (any decent headshot; if PNG, `sips -s format jpeg photo.png --out assets/photo.jpg` on macOS)
2. CV PDF → save as `assets/cv.pdf`
3. Links for the two IEEE MIT URTC 2024 papers (IEEE Xplore preferred; Scholar citation pages acceptable):
   - Paper 1: "Analyzing factors influencing crime rates in communities by lasso regression"
   - Paper 2: "Enhancing Ultrasonic Thin Film Measurement Using PCA and DFT Driven Machine Learning Models"
4. X profile URL, LessWrong profile URL, Substack URL (if exists), Admonymous URL (if wanted)
5. Chosen custom domain (user purchases it at any registrar)

- [ ] **Step 2: Write `assets/site-data.md`**

```markdown
# Site data (single source of truth for substitutions)
X_URL: https://x.com/<handle>
LESSWRONG_URL: https://www.lesswrong.com/users/<handle>
SUBSTACK_URL: (optional, or "none")
ADMONYMOUS_URL: (optional, or "none")
PAPER1_URL: <link — crime rates / lasso regression paper>
PAPER2_URL: <link — ultrasonic thin film ML paper>
DOMAIN: <purchased domain, e.g. jollendai.com>
```

Fill every key with the user's actual values. If Substack/Admonymous don't exist, write `none` — downstream tasks then omit those lines entirely.

- [ ] **Step 3: Verify assets exist and URLs resolve**

Run: `test -f assets/photo.jpg && test -f assets/cv.pdf && echo OK`
Expected: `OK`

Run for each http(s) URL in site-data:
`curl -sIL -A "Mozilla/5.0" -o /dev/null -w "%{http_code} " <URL>; echo <URL>`
Expected: `200` (or `301/302`). Note: X and LinkedIn may return `403`/`999` to curl — for those two, verify by opening in a browser instead.

- [ ] **Step 4: Commit**

```bash
git add assets/
git commit -m "Add photo, CV, and site data"
```

---

### Task 2: Header — photo and expanded link row

**Files:**
- Modify: `index.html` (head `<style>` block and top of `<main>`)

**Interfaces:**
- Consumes: `assets/site-data.md` (`X_URL`, `LESSWRONG_URL`), `assets/photo.jpg`, `assets/cv.pdf`
- Produces: `.header` / `.headshot` CSS classes; link row used as-is by later tasks (no further changes to it)

- [ ] **Step 1: Verify current state (failing check)**

Run: `grep -c 'assets/photo.jpg' index.html || true`
Expected: `0`

- [ ] **Step 2: Add header CSS**

Read `index.html`. In the `<style>` block, directly after the `.tagline` rule, insert:

```css
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1.25rem;
  }
  .headshot {
    width: 84px;
    height: 84px;
    border-radius: 50%;
    object-fit: cover;
    flex-shrink: 0;
  }
```

- [ ] **Step 3: Restructure header HTML and extend links**

Replace:

```html
<main>
  <h1>Jollen Dai</h1>
  <p class="tagline">CS undergraduate at UCLA, working on AI safety.</p>
```

with:

```html
<main>
  <header class="header">
    <div>
      <h1>Jollen Dai</h1>
      <p class="tagline">CS undergraduate at UCLA, working on AI safety.</p>
    </div>
    <img class="headshot" src="assets/photo.jpg" alt="Photo of Jollen Dai">
  </header>
```

In the `<p class="links">` block, after the Email link, add (substituting SITE-DATA values):

```html
    <a href="assets/cv.pdf">CV</a>
    <a href="{{X_URL}}">X</a>
    <a href="{{LESSWRONG_URL}}">LessWrong</a>
```

- [ ] **Step 4: Verify (passing check)**

Run: `grep -c 'assets/photo.jpg' index.html && grep -c 'assets/cv.pdf' index.html && grep -c '{{' index.html || true`
Expected: `1`, `1`, then `0` (no unsubstituted tokens).

Open `index.html` in a browser: photo renders round, right of name; header doesn't wrap awkwardly at ~375px width (flex shrinks; if the tagline wraps under the photo acceptably, pass).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add headshot and CV/X/LessWrong links to header"
```

---

### Task 3: Papers section + Background trim

**Files:**
- Modify: `index.html` (between the Now and Background sections; one `<li>` removed from Background)

**Interfaces:**
- Consumes: `assets/site-data.md` (`PAPER1_URL`, `PAPER2_URL`)
- Produces: `<h2>Papers</h2>` section; Background no longer mentions the URTC papers

- [ ] **Step 1: Verify current state (failing check)**

Run: `grep -c '<h2>Papers</h2>' index.html || true`
Expected: `0`

- [ ] **Step 2: Insert Papers section**

Read `index.html`. Directly before `<h2>Background</h2>`, insert (substituting SITE-DATA values):

```html
  <h2>Papers</h2>
  <ul>
    <li><a href="{{PAPER1_URL}}">Analyzing Factors Influencing Crime Rates in Communities by Lasso Regression</a> — IEEE MIT URTC 2024, sole author. Regularized regression to identify which community-level factors actually predict crime rates.</li>
    <li><a href="{{PAPER2_URL}}">Enhancing Ultrasonic Thin Film Measurement Using PCA and DFT Driven Machine Learning Models</a> — IEEE MIT URTC 2024, sole author. Feature pipelines (PCA, DFT) for ML-based thin-film thickness measurement.</li>
    <li>Group bias in aligned LLMs — in preparation, from the NSF Trustworthy AI REU at RIT (advised by Prof. Ashique KhudaBukhsh). Finding: the bias wasn't removable — every method either damaged the model or quietly failed, and standard benchmarks couldn't see the difference.</li>
  </ul>
```

The one-line context sentences above are draft copy — flag them to the user for wording approval at task review; adjust if the user corrects the papers' claims.

- [ ] **Step 3: Trim Background**

In the Background `<ul>`, delete the line:

```html
    <li>Two sole-author papers at IEEE MIT URTC 2024.</li>
```

(Keep the REU and BlueDot items — the REU *experience* stays in Background; the REU *paper* now lives in Papers.)

- [ ] **Step 4: Verify (passing check)**

Run: `grep -c '<h2>Papers</h2>' index.html && grep -c 'Two sole-author papers' index.html || true`
Expected: `1`, then `0`.

Browser check: Papers section renders between Now and Background, links styled like the rest.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add Papers section, trim duplicate from Background"
```

---

### Task 4: Writing section, Contact, footer

**Files:**
- Modify: `index.html` (Writing and Contact sections, end of `<main>`)

**Interfaces:**
- Consumes: `assets/site-data.md` (`LESSWRONG_URL`, `SUBSTACK_URL`, `ADMONYMOUS_URL`)
- Produces: final page content; no section may still contain the word "soon"

- [ ] **Step 1: Verify current state (failing check)**

Run: `grep -c 'Coming soon' index.html || true`
Expected: `1`

- [ ] **Step 2: Replace Writing placeholder**

Replace:

```html
  <h2>Writing</h2>
  <p class="quiet">Coming soon — posts on unlearning robustness and intervention evaluation. Substack and LessWrong links will live here.</p>
```

with (substituting SITE-DATA values; drop the Substack clause if `SUBSTACK_URL: none`):

```html
  <h2>Writing</h2>
  <p>I write about unlearning robustness and intervention evaluation on <a href="{{LESSWRONG_URL}}">LessWrong</a> and <a href="{{SUBSTACK_URL}}">Substack</a>.</p>
```

- [ ] **Step 3: Contact addition (only if `ADMONYMOUS_URL` is not `none`)**

At the end of the Contact `<p>`, before `</p>`, append:

```html
 You can also <a href="{{ADMONYMOUS_URL}}">send me anonymous feedback</a>.
```

- [ ] **Step 4: Add footer**

Directly before `</main>`, insert:

```html
  <p class="quiet">Last updated August 2026.</p>
```

- [ ] **Step 5: Verify (passing check)**

Run: `grep -ci 'coming soon' index.html || true && grep -c 'Last updated' index.html && grep -c '{{' index.html || true`
Expected: `0`, `1`, `0`.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Replace Writing placeholder, add footer"
```

---

### Task 5: Full local verification + README

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: final `index.html` from Tasks 2–4
- Produces: verified page; gates Task 6 (do not deploy on any FAIL)

- [ ] **Step 1: Link check**

Run:

```bash
python3 - <<'EOF'
import re, sys, urllib.request
src = open('index.html').read()
urls = [u for u in re.findall(r'href="([^"]+)"', src) if u.startswith('http')]
fails = []
for u in urls:
    req = urllib.request.Request(u, method='HEAD', headers={'User-Agent': 'Mozilla/5.0'})
    try:
        with urllib.request.urlopen(req, timeout=10) as r:
            print(r.status, u)
    except Exception as e:
        print('FAIL', u, e)
        fails.append(u)
print('---')
print('FAILURES:', len(fails))
EOF
```

Expected: `FAILURES: 0`, except X/LinkedIn which may 403/999 to bots — verify those two in a browser and treat browser-200 as pass.

- [ ] **Step 2: Local file references**

Run: `test -f assets/photo.jpg && test -f assets/cv.pdf && grep -c 'assets/cv.pdf' index.html && echo OK`
Expected: `1` then `OK`

- [ ] **Step 3: Visual check**

Open `index.html` in browser:
- Light mode: readable, photo round, sections ordered Header → Intro → Now → Papers → Background → Writing → Contact → footer
- Dark mode (macOS: System Settings → Appearance → Dark, or DevTools emulation): all text legible, no white flash blocks
- Narrow window (~375px): no horizontal scroll, header wraps acceptably

- [ ] **Step 4: Write README.md**

```markdown
# jollen-site

Personal site of Jollen Dai — AI safety research (unlearning robustness, intervention evaluation).
Single static `index.html`, no build step. Deployed via GitHub Pages.
```

- [ ] **Step 5: Commit**

```bash
git add README.md
git commit -m "Add README"
```

---

### Task 6: GitHub repo + Pages deploy

**Files:**
- None created locally (remote setup)

**Interfaces:**
- Consumes: verified repo state from Task 5
- Produces: live site at `https://jxdai2007.github.io`; remote `origin`

- [ ] **Step 1: Check gh auth**

Run: `gh auth status`
Expected: logged in as `jxdai2007`. If not: user runs `! gh auth login` interactively.

- [ ] **Step 2: Create repo and push**

```bash
gh repo create jxdai2007.github.io --public --source=. --remote=origin --push
```

Expected: repo created, `main` pushed. (Repo named `<user>.github.io` = user site, serves at domain root.)

- [ ] **Step 3: Ensure Pages enabled**

Run: `gh api repos/jxdai2007/jxdai2007.github.io/pages -X POST -f "source[branch]=main" -f "source[path]=/" 2>&1 || true`
Expected: `201`, or `409 Conflict` meaning Pages already auto-enabled for user sites — both pass.

- [ ] **Step 4: Verify live**

Run (allow a few minutes for first build): `curl -sIL -o /dev/null -w "%{http_code}\n" https://jxdai2007.github.io`
Expected: `200`. Open in browser, spot-check sections and dark mode.

---

### Task 7: Custom domain

**Files:**
- Create: `CNAME`

**Interfaces:**
- Consumes: `assets/site-data.md` (`DOMAIN`); live Pages site from Task 6
- Produces: site served at `https://<DOMAIN>` with HTTPS enforced

- [ ] **Step 1: Write CNAME file**

`CNAME` containing exactly one line — the DOMAIN value from site-data (no protocol, no trailing slash):

```
{{DOMAIN}}
```

- [ ] **Step 2: Commit and push**

```bash
git add CNAME
git commit -m "Add custom domain"
git push
```

- [ ] **Step 3: User configures DNS at registrar** (user action — provide these records)

- Apex `{{DOMAIN}}`: four `A` records → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- `www.{{DOMAIN}}`: `CNAME` record → `jxdai2007.github.io`

- [ ] **Step 4: Set domain + HTTPS on GitHub**

```bash
gh api repos/jxdai2007/jxdai2007.github.io/pages -X PUT -f cname="{{DOMAIN}}"
```

After DNS propagates and GitHub issues the cert (can take up to ~1 hour):

```bash
gh api repos/jxdai2007/jxdai2007.github.io/pages -X PUT -F https_enforced=true
```

- [ ] **Step 5: Verify**

Run: `curl -sIL -o /dev/null -w "%{http_code}\n" https://{{DOMAIN}}`
Expected: `200`. Also confirm `http://` redirects to `https://`.
