# AI Product Portfolio Gallery Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the existing personal homepage into a truthful, art-directed AI product manager portfolio with an unchanged fluid cover, a freely explorable horizontal night-gallery corridor, and accessible project viewing rooms.

**Architecture:** Keep the current Pug, Less, vanilla JavaScript, and Gulp static build. Render all portfolio content from `src/data/portfolio.json` into semantic HTML at build time, then add progressive JavaScript enhancements for horizontal navigation, project-room history, and focus restoration. Preserve readable vertical content when JavaScript, WebGL, motion, or desktop layout support is unavailable.

**Tech Stack:** Node.js 20+, npm, Gulp 4, Pug, Less, vanilla JavaScript, Node test runner, jsdom 26, GitHub Pages Actions.

## Global Constraints

- Preserve the existing WebGL fluid cover, its title/subtitle animation, and the `ENTER` transition.
- Target role copy must say `通用 AI 产品经理`.
- Do not invent internships, users, revenue, growth metrics, or commercial outcomes.
- Desktop is a horizontal night-gallery corridor; mobile at approximately 390px is a vertical gallery.
- `INDEX / ROOM MAP`, resume, and contact actions remain reachable without completing the corridor.
- Project rooms must close with a visible button, `Escape`, and browser Back, restoring scroll position and focus.
- Core content must remain readable with JavaScript disabled.
- Respect `prefers-reduced-motion`.
- Do not add React, Vue, a CMS, authentication, comments, analytics dashboards, or a 3D game engine.
- Repository metadata targets `aaaloooooof/personalwebsite` and GitHub Pages at `https://aaaloooooof.github.io/personalwebsite/`.
- When no PDF resume exists, omit the download action rather than rendering a dead link.

---

## File Structure

### New files

- `src/data/portfolio.json` — single source of truth for profile, abilities, works, notes, and contact links.
- `src/components/gallery-mixins.pug` — reusable artwork, note, archive, and project-room Pug mixins.
- `src/components/gallery-shell.pug` — fixed museum toolbar, horizontal corridor, static fallback order, and project rooms.
- `src/css/gallery/gallery.less` — night-gallery visual system, corridor, frames, viewing rooms, responsive behavior, and reduced-motion rules.
- `src/js/gallery-state.js` — pure hash, input-delta, and endless-position helpers usable in Node tests and the browser.
- `src/js/gallery.js` — DOM controller for corridor navigation, room open/close, history, and focus restoration.
- `scripts/validate-portfolio.js` — deterministic content validation used by tests and builds.
- `tests/portfolio-data.test.js` — portfolio schema and truthfulness guard tests.
- `tests/build-output.test.js` — generated HTML/CSS/static-fallback assertions.
- `tests/gallery-state.test.js` — pure navigation-state unit tests.
- `tests/gallery-dom.test.js` — jsdom interaction and accessibility tests.
- `docs/testing/gallery-acceptance.md` — desktop/mobile/keyboard/manual acceptance record.

### Modified files

- `.gitignore` — ignore local brainstorming state, macOS metadata, dependency folders, and archived dependency backup.
- `package.json` and `package-lock.json` — real repository metadata, validation/test scripts, and jsdom dependency.
- `gulpfile.js` — validate/load portfolio data, compile Pug with both config objects, watch Less correctly, and test before deploy.
- `src/index.pug` — use `lang="zh-CN"`, add `no-js`, and retain the cover plus generated gallery.
- `src/components/main.pug` — replace the current four-link card with the gallery shell.
- `src/components/head.pug` — switch `no-js` to `js` before rendering and retain meaningful metadata.
- `src/components/scripts.pug` — load gallery state/controller after the current scripts.
- `src/css/style.less` — import the gallery stylesheet.
- `.github/workflows/deploy-pages.yml` — use `npm ci`, validate, test, and build before upload.
- `DEPLOY_GITHUB_PAGES.md` — replace repository-name examples with the selected repository and deployment checks.

---

### Task 1: Establish a clean, reproducible repository baseline

**Files:**
- Modify: `.gitignore`
- Modify: `package.json:6-13`
- Track: existing source, workflow, lockfile, and current `dist/` snapshot

**Interfaces:**
- Consumes: current root commit containing the approved design spec.
- Produces: a clean tracked baseline on `main`, real repository metadata, and ignored local-only artifacts.

- [ ] **Step 1: Verify the current build before changing repository metadata**

Run:

```bash
npm install
npm run build
```

Expected: both commands exit `0`; `dist/index.html`, `dist/css/style.css`, and `dist/js/main.js` exist.

- [ ] **Step 2: Update `.gitignore` with explicit local-only paths**

Remove the existing `package-lock.json` ignore rule and add:

```gitignore
.DS_Store
.superpowers/
node_modules_incomplete_backup_*/
```

Keep `dist/` tracked because it currently contains the deployable snapshot and legacy content needed during migration.

- [ ] **Step 3: Replace repository metadata in `package.json`**

Set these exact fields:

```json
{
  "homepage": "https://aaaloooooof.github.io/personalwebsite/",
  "repository": "https://github.com/aaaloooooof/personalwebsite",
  "bugs": "https://github.com/aaaloooooof/personalwebsite/issues"
}
```

- [ ] **Step 4: Verify ignored and tracked scope**

Run:

```bash
git status --short
git check-ignore .superpowers/ node_modules_incomplete_backup_20260423_2023/ .DS_Store
```

Expected: the three local paths are ignored; `package-lock.json`, `src/`, `.github/`, and the current `dist/` files remain available to add.

- [ ] **Step 5: Commit the baseline**

```bash
git add .editorconfig .gitignore package.json package-lock.json gulpfile.js config.json src dist .github README.md README.zh_CN.md DEPLOY_GITHUB_PAGES.md LICENSE WEEKLY_TEMPLATE.js WEEKLY_UPDATE_GUIDE.md personal-website-qr.svg personal-website-local-qr.svg personal-website-github-pages-qr.svg pnpm-lock.yaml 替换说明.md
git commit -m "chore: establish personal website baseline"
```

Expected: commit succeeds and `.superpowers/` plus the dependency backup remain untracked and ignored.

---

### Task 2: Add validated portfolio content as the single source of truth

**Files:**
- Create: `src/data/portfolio.json`
- Create: `scripts/validate-portfolio.js`
- Create: `tests/portfolio-data.test.js`
- Modify: `package.json:10-34`
- Modify: `package-lock.json`

**Interfaces:**
- Consumes: Node.js `fs` and the JSON structure below.
- Produces: `validatePortfolio(portfolio): string[]`, a zero-error `src/data/portfolio.json`, and `npm test` / `npm run validate:data` commands.

- [ ] **Step 1: Add test scripts and install the DOM test dependency**

Run:

```bash
npm install --save-dev jsdom@26.1.0
```

Set scripts to:

```json
{
  "predev": "gulp build",
  "dev": "gulp watch",
  "validate:data": "node scripts/validate-portfolio.js",
  "test": "node --test tests/*.test.js",
  "prebuild": "npm run validate:data",
  "build": "gulp build"
}
```

- [ ] **Step 2: Write the failing portfolio validation test**

Create `tests/portfolio-data.test.js`:

```js
const test = require('node:test')
const assert = require('node:assert/strict')
const portfolio = require('../src/data/portfolio.json')
const { validatePortfolio } = require('../scripts/validate-portfolio')

test('portfolio contains a complete truthful launch set', () => {
  assert.deepEqual(validatePortfolio(portfolio), [])
  assert.equal(portfolio.profile.role, '通用 AI 产品经理')
  assert.deepEqual(portfolio.projects.map(project => project.id), [
    'mavis-demo-generator',
    'learnloop',
    'mirror-campus'
  ])
  assert.equal(portfolio.resume.url, null)
})

test('validator rejects missing project evidence', () => {
  const invalid = structuredClone(portfolio)
  invalid.projects[0].decisions = []
  assert.match(validatePortfolio(invalid).join('\n'), /decisions/)
})
```

- [ ] **Step 3: Run the test and verify it fails**

Run:

```bash
npm test
```

Expected: FAIL because `src/data/portfolio.json` and `scripts/validate-portfolio.js` do not exist.

- [ ] **Step 4: Create the launch portfolio data**

Create `src/data/portfolio.json` with this exact shape and truthful initial copy:

```json
{
  "profile": {
    "displayName": "aaaloooooof",
    "role": "通用 AI 产品经理",
    "statement": "从真实用户问题出发，把 AI 能力转化为可运行、可验证的产品原型。",
    "status": "正在寻找通用 AI 产品经理机会",
    "avatar": "assets/avatar.jpg"
  },
  "capabilities": [
    { "id": "research", "label": "用户研究与需求分析" },
    { "id": "product", "label": "产品方案与 PRD" },
    { "id": "workflow", "label": "AI Workflow / LLM 应用设计" },
    { "id": "prototype", "label": "原型构建与快速验证" }
  ],
  "projects": [
    {
      "id": "mavis-demo-generator",
      "title": "Mavis Demo Generator",
      "category": "AI Workflow",
      "year": "2026",
      "summary": "把产品与需求材料转化为可展示 Demo 内容的 AI 生成与导出平台。",
      "problem": "产品材料来源分散，转化成结构完整、可展示的 Demo 内容需要重复整理。",
      "audience": "需要快速制作产品演示内容的产品实践者。",
      "role": "个人项目实践：问题定义、工作流设计与原型构建。",
      "insights": ["生成结果必须服务于演示任务，而不是停留在单次回答。"],
      "decisions": ["把流程拆为材料输入、结构整理、内容生成和导出四个阶段。"],
      "outputs": ["可运行的平台原型", "生成与导出流程"],
      "reflection": "下一步应使用真实演示任务验证生成结构和导出质量。",
      "repository": "https://github.com/aaaloooooof/mavis-demo-generator",
      "palette": ["#202535", "#745569", "#c28c5c"]
    },
    {
      "id": "learnloop",
      "title": "LearnLoop",
      "category": "AI Learning Experience",
      "year": "2026",
      "summary": "把碎片化知识组织成个性化学习路径的 AI 辅助学习工具。",
      "problem": "学习者拥有大量碎片内容，却缺少持续可执行的学习路径。",
      "audience": "需要整理知识并持续推进学习任务的学习者。",
      "role": "个人项目实践：学习流程梳理、产品结构设计与原型构建。",
      "insights": ["个性化路径需要被拆成可以执行和追踪的任务。"],
      "decisions": ["使用卡片式任务和进度追踪承载学习闭环。"],
      "outputs": ["学习路径产品原型", "任务与进度机制"],
      "reflection": "下一步应验证任务粒度和反馈频率是否能够支持长期使用。",
      "repository": "https://github.com/aaaloooooof/learnloop",
      "palette": ["#172331", "#53717b", "#b7a36f"]
    },
    {
      "id": "mirror-campus",
      "title": "MirrorCampus",
      "category": "AI Governance Platform",
      "year": "2026",
      "summary": "面向政策、风险、学习分析与治理看板场景的校园 AI 治理平台原型。",
      "problem": "校园 AI 治理信息分散在政策、风险和学习分析等不同视角中。",
      "audience": "需要理解和处理校园 AI 治理信息的管理与研究人员。",
      "role": "个人项目实践：治理场景梳理、信息架构与看板原型构建。",
      "insights": ["复杂治理议题需要同时支持总览和分主题深入查看。"],
      "decisions": ["按政策、风险、学习分析和治理看板拆分核心场景。"],
      "outputs": ["治理平台原型", "分场景信息架构"],
      "reflection": "下一步应与真实治理角色核对指标、权限和决策流程。",
      "repository": "https://github.com/aaaloooooof/mirror-campus",
      "palette": ["#1c202b", "#586882", "#9d6573"]
    }
  ],
  "notes": [
    { "title": "训练后时代：大模型能力边界的下一站", "category": "模型", "summary": "重新看后训练如何进入真实产品任务。" },
    { "title": "AI 产品的冷启动：从种子用户到正反馈飞轮", "category": "产品", "summary": "从最小场景和种子用户开始建立反馈闭环。" },
    { "title": "教育 AI 的下一段路：个性化与数据隐私的平衡", "category": "教育 AI", "summary": "在产品效果之外设计边界、信任和解释。" }
  ],
  "about": {
    "summary": "关注 AI 产品、教育场景、用户研究和可运行原型，持续把研究与观察转化为产品实践。",
    "methods": ["问题定义", "用户研究", "工作流设计", "快速原型", "复盘迭代"]
  },
  "resume": { "url": null, "label": "下载简历" },
  "contact": {
    "email": "51284102015@stu.ecnu.edu.cn",
    "github": "https://github.com/aaaloooooof",
    "rednote": "https://xhslink.com/m/8VrAxdmEfE9"
  }
}
```

- [ ] **Step 5: Implement strict validation**

Create `scripts/validate-portfolio.js`:

```js
const portfolio = require('../src/data/portfolio.json')

function validatePortfolio(data) {
  const errors = []
  const requiredProfile = ['displayName', 'role', 'statement', 'status', 'avatar']
  requiredProfile.forEach(key => {
    if (!data.profile || !data.profile[key]) errors.push(`profile.${key} is required`)
  })
  if (!Array.isArray(data.capabilities) || data.capabilities.length !== 4) {
    errors.push('capabilities must contain exactly 4 items')
  }
  if (!Array.isArray(data.projects) || data.projects.length !== 3) {
    errors.push('projects must contain exactly 3 launch projects')
  }
  const projectFields = ['id', 'title', 'category', 'year', 'summary', 'problem', 'audience', 'role', 'reflection', 'repository']
  for (const project of data.projects || []) {
    projectFields.forEach(key => {
      if (!project[key]) errors.push(`projects.${project.id || 'unknown'}.${key} is required`)
    })
    ;['insights', 'decisions', 'outputs'].forEach(key => {
      if (!Array.isArray(project[key]) || project[key].length === 0) {
        errors.push(`projects.${project.id || 'unknown'}.${key} must not be empty`)
      }
    })
    if (!Array.isArray(project.palette) || project.palette.length !== 3) {
      errors.push(`projects.${project.id || 'unknown'}.palette must contain 3 colors`)
    }
  }
  if (!data.contact || !data.contact.email || !data.contact.github) {
    errors.push('contact.email and contact.github are required')
  }
  return errors
}

if (require.main === module) {
  const errors = validatePortfolio(portfolio)
  if (errors.length) {
    console.error(errors.join('\n'))
    process.exitCode = 1
  }
}

module.exports = { validatePortfolio }
```

- [ ] **Step 6: Run validation and tests**

Run:

```bash
npm run validate:data
npm test
```

Expected: validation exits `0`; both tests PASS.

- [ ] **Step 7: Commit**

```bash
git add package.json package-lock.json src/data/portfolio.json scripts/validate-portfolio.js tests/portfolio-data.test.js
git commit -m "feat: add validated portfolio content model"
```

---

### Task 3: Render the semantic gallery and static fallback

**Files:**
- Create: `src/components/gallery-mixins.pug`
- Create: `src/components/gallery-shell.pug`
- Create: `tests/build-output.test.js`
- Modify: `src/components/main.pug:1-50`
- Modify: `src/index.pug:1-9`
- Modify: `gulpfile.js:14-54`

**Interfaces:**
- Consumes: `portfolio.profile`, `portfolio.capabilities`, `portfolio.projects`, `portfolio.notes`, `portfolio.about`, `portfolio.resume`, and `portfolio.contact`.
- Produces: semantic `#portfolio-gallery`, `.gallery-scroll`, `[data-project-id]`, `[data-project-room]`, and anchor targets for every content section.

- [ ] **Step 1: Write the failing build-output test**

Create `tests/build-output.test.js`:

```js
const test = require('node:test')
const assert = require('node:assert/strict')
const fs = require('node:fs')
const { execFileSync } = require('node:child_process')

test('build renders the gallery and all launch projects', () => {
  execFileSync('npm', ['run', 'build'], { stdio: 'pipe' })
  const html = fs.readFileSync('dist/index.html', 'utf8')
  assert.match(html, /id="portfolio-gallery"/)
  assert.match(html, /class="[^"]*gallery-scroll/)
  for (const id of ['mavis-demo-generator', 'learnloop', 'mirror-campus']) {
    assert.match(html, new RegExp(`data-project-id="${id}"`))
    assert.match(html, new RegExp(`data-project-room="${id}"`))
  }
  assert.match(html, /通用 AI 产品经理/)
  assert.doesNotMatch(html, /href=""/)
})
```

- [ ] **Step 2: Run the focused test and verify it fails**

Run:

```bash
node --test tests/build-output.test.js
```

Expected: FAIL because `#portfolio-gallery` is absent.

- [ ] **Step 3: Load portfolio data in Gulp**

Add beside `config`:

```js
const portfolio = require('./src/data/portfolio.json')
const { validatePortfolio } = require('./scripts/validate-portfolio')
```

Update the `pug` task body to validate and pass both objects:

```js
gulp.task('pug', function () {
  const errors = validatePortfolio(portfolio)
  if (errors.length) throw new Error(errors.join('\n'))
  return gulp
    .src('./src/index.pug')
    .pipe(pug({ data: { ...config, portfolio } }))
    .pipe(gulp.dest('./dist'))
})
```

- [ ] **Step 4: Create reusable gallery mixins**

Create `src/components/gallery-mixins.pug`:

```pug
mixin artwork(project)
  button.artwork-card(type="button" data-project-id=project.id aria-haspopup="dialog" aria-controls=`room-${project.id}` style=`--art-a:${project.palette[0]};--art-b:${project.palette[1]};--art-c:${project.palette[2]}`)
    span.artwork-image(aria-hidden="true")
    span.artwork-label
      span.artwork-number= project.year
      strong= project.title
      span= project.category

mixin projectRoom(project)
  article.project-room(id=`room-${project.id}` data-project-room=project.id aria-labelledby=`title-${project.id}`)
    button.project-room-close(type="button" data-close-room aria-label=`关闭 ${project.title}`) ×
    header.project-room-hero(style=`--art-a:${project.palette[0]};--art-b:${project.palette[1]};--art-c:${project.palette[2]}`)
      p.project-kicker #{project.category} · #{project.year}
      h2(id=`title-${project.id}`)= project.title
      p= project.summary
    .project-room-content
      section
        h3 问题与用户
        p= project.problem
        p= project.audience
      section
        h3 我的角色
        p= project.role
      section
        h3 调研与洞察
        ul
          each insight in project.insights
            li= insight
      section
        h3 产品决策
        ul
          each decision in project.decisions
            li= decision
      section
        h3 原型与产出
        ul
          each output in project.outputs
            li= output
        a(href=project.repository target="_blank" rel="noreferrer") 查看 GitHub 项目
      section
        h3 复盘与下一步
        p= project.reflection
```

- [ ] **Step 5: Create the gallery shell**

Create `src/components/gallery-shell.pug` with semantic, build-time content:

```pug
include gallery-mixins.pug

section#portfolio-gallery.content.content-main.gallery-shell(aria-label="AI 产品作品展")
  header.gallery-toolbar
    a.gallery-brand(href="#profile") #{portfolio.profile.displayName} / #{portfolio.profile.role}
    nav.gallery-actions(aria-label="展厅快捷导航")
      a(href="#index") INDEX / ROOM MAP
      if portfolio.resume.url
        a(href=portfolio.resume.url download)= portfolio.resume.label
      a(href=`mailto:${portfolio.contact.email}`) 联系我

  .gallery-scroll(tabindex="0" aria-label="横向项目长廊")
    .gallery-sequence
      section.gallery-panel.profile-panel#profile
        p.gallery-kicker AI PRODUCT EXHIBITION
        h1= portfolio.profile.role
        p= portfolio.profile.statement
        p= portfolio.profile.status

      section.gallery-panel.capability-wall#capabilities
        h2 能力墙
        each capability, index in portfolio.capabilities
          article.capability-plaque
            span= String(index + 1).padStart(2, '0')
            h3= capability.label

      section.gallery-panel.works-wall#works
        h2 Selected Works
        each project in portfolio.projects
          +artwork(project)

      section.gallery-panel.notes-wall#notes
        h2 产品思考与 AI 实践
        each note in portfolio.notes
          article.note-plaque
            span= note.category
            h3= note.title
            p= note.summary

      section.gallery-panel.archive-wall#about
        h2 实践档案
        p= portfolio.about.summary
        ul
          each method in portfolio.about.methods
            li= method

      section.gallery-panel.contact-desk#contact
        h2 Exit Desk
        a(href=`mailto:${portfolio.contact.email}`)= portfolio.contact.email
        a(href=portfolio.contact.github target="_blank" rel="noreferrer") GitHub
        a(href=portfolio.contact.rednote target="_blank" rel="noreferrer") Rednote

  nav.gallery-index#index(aria-label="展厅目录")
    a(href="#profile") 个人定位
    a(href="#capabilities") 能力墙
    a(href="#works") 项目
    a(href="#notes") 产品思考
    a(href="#about") 实践档案
    a(href="#contact") 联系方式

  each project in portfolio.projects
    +projectRoom(project)
```

- [ ] **Step 6: Replace the old main card and set document language**

Replace `src/components/main.pug` with:

```pug
include gallery-shell.pug
```

Change the first HTML line in `src/index.pug` to:

```pug
html.no-js(lang="zh-CN")
```

Retain both `components/intro.pug` and `components/main.pug` under the existing `main` element.

- [ ] **Step 7: Run tests**

Run:

```bash
npm test
```

Expected: all tests PASS and `dist/index.html` contains all three projects and project rooms.

- [ ] **Step 8: Commit**

```bash
git add gulpfile.js src/index.pug src/components/main.pug src/components/gallery-mixins.pug src/components/gallery-shell.pug tests/build-output.test.js dist/index.html
git commit -m "feat: render semantic portfolio gallery"
```

---

### Task 4: Build the night-gallery visual system and responsive layout

**Files:**
- Create: `src/css/gallery/gallery.less`
- Modify: `src/css/style.less:1-8`
- Modify: `tests/build-output.test.js`

**Interfaces:**
- Consumes: gallery classes and `--art-a`, `--art-b`, `--art-c` custom properties from Task 3.
- Produces: desktop horizontal panels, framed artwork, fixed toolbar, vertical mobile layout, project-room presentation, and visible focus styles.

- [ ] **Step 1: Extend the failing build-output test for required CSS contracts**

Append inside the existing build test:

```js
const css = fs.readFileSync('dist/css/style.css', 'utf8')
for (const selector of ['.gallery-shell', '.gallery-scroll', '.artwork-card', '.project-room']) {
  assert.match(css, new RegExp(selector.replace('.', '\\.')))
}
assert.match(css, /prefers-reduced-motion/)
```

- [ ] **Step 2: Run the focused test and verify it fails**

Run:

```bash
node --test tests/build-output.test.js
```

Expected: FAIL because gallery selectors do not exist in compiled CSS.

- [ ] **Step 3: Create the gallery stylesheet**

Create `src/css/gallery/gallery.less` with these required layout contracts and expand only within the same selectors:

```less
.gallery-shell {
  position: absolute;
  inset: 0;
  overflow: hidden;
  color: #eeeae2;
  background: #08080a;
}

.gallery-toolbar {
  position: fixed;
  inset: 0 0 auto;
  z-index: 30;
  display: flex;
  justify-content: space-between;
  padding: 18px 24px;
  background: linear-gradient(#08080a 25%, transparent);
}

.gallery-actions { display: flex; gap: 18px; }
.gallery-toolbar a { color: inherit; text-decoration: none; }

.gallery-scroll {
  display: flex;
  align-items: stretch;
  width: 100%;
  height: 100vh;
  overflow-x: auto;
  overflow-y: hidden;
  scroll-snap-type: x proximity;
  background: linear-gradient(180deg, #151519 0 74%, #09090b 74%);
  cursor: grab;
  touch-action: pan-y;
}

.gallery-scroll.is-dragging { cursor: grabbing; scroll-snap-type: none; user-select: none; }
.gallery-sequence { display: flex; flex: 0 0 auto; min-width: max-content; }
.gallery-panel {
  flex: 0 0 auto;
  min-width: min(92vw, 1180px);
  padding: 110px 8vw 72px;
  scroll-snap-align: start;
  border-right: 1px solid rgba(255,255,255,.08);
}

.artwork-card {
  display: inline-grid;
  width: min(32vw, 390px);
  margin: 0 34px 34px 0;
  padding: 0;
  color: inherit;
  text-align: left;
  border: 0;
  background: transparent;
  cursor: pointer;
}

.artwork-image {
  display: block;
  aspect-ratio: 4 / 3;
  border: 12px solid #39332f;
  background:
    radial-gradient(circle at 68% 28%, var(--art-c) 0 11%, transparent 12%),
    linear-gradient(135deg, var(--art-a), var(--art-b) 54%, var(--art-c));
  box-shadow: 0 24px 48px rgba(0,0,0,.45);
}

.artwork-label { display: grid; gap: 4px; margin-top: 14px; }
.artwork-label span { color: #aaa5ae; }

.project-room {
  position: fixed;
  inset: 5vh 5vw;
  z-index: 50;
  overflow-y: auto;
  color: #1c1c20;
  background: #eeeae2;
  box-shadow: 0 30px 90px rgba(0,0,0,.7);
}

.js .project-room { display: none; }
.js .project-room[aria-hidden="false"] { display: block; }
.project-room-open { overflow: hidden; }
.project-room-close { position: sticky; top: 18px; float: right; z-index: 3; }
.project-room-hero { min-height: 52vh; padding: 12vh 8vw; background: linear-gradient(135deg, var(--art-a), var(--art-b), var(--art-c)); color: white; }
.project-room-content { max-width: 760px; margin: auto; padding: 64px 24px 96px; }
.project-room-content section { margin-bottom: 48px; }

.artwork-card:focus-visible,
.gallery-toolbar a:focus-visible,
.project-room a:focus-visible,
.project-room button:focus-visible { outline: 3px solid #b8aaff; outline-offset: 5px; }

@media (max-width: 720px) {
  .gallery-shell { position: relative; min-height: 100vh; overflow: visible; }
  .gallery-toolbar { position: sticky; flex-wrap: wrap; background: #08080a; }
  .gallery-scroll { display: block; height: auto; overflow: visible; scroll-snap-type: none; }
  .gallery-sequence { display: block; min-width: 0; }
  .gallery-sequence[data-gallery-clone] { display: none; }
  .gallery-panel { min-width: 0; padding: 80px 20px 48px; }
  .artwork-card { display: grid; width: 100%; margin-right: 0; }
  .project-room { inset: 0; }
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { scroll-behavior: auto !important; animation-duration: .01ms !important; animation-iteration-count: 1 !important; transition-duration: .01ms !important; }
}
```

- [ ] **Step 4: Import the stylesheet**

Append to `src/css/style.less`:

```less
@import 'gallery/gallery';
```

- [ ] **Step 5: Build and run tests**

Run:

```bash
npm run build
npm test
```

Expected: build exits `0`; all tests PASS; compiled CSS contains all required gallery selectors.

- [ ] **Step 6: Commit**

```bash
git add src/css/style.less src/css/gallery/gallery.less tests/build-output.test.js dist/css/style.css
git commit -m "feat: style the responsive night gallery"
```

---

### Task 5: Implement and test pure gallery navigation state

**Files:**
- Create: `src/js/gallery-state.js`
- Create: `tests/gallery-state.test.js`

**Interfaces:**
- Produces: `GalleryState.projectHash(id)`, `GalleryState.parseHash(hash)`, `GalleryState.horizontalDelta(deltaX, deltaY)`, and `GalleryState.wrapPosition(value, cycleWidth)`.
- Consumed by: `src/js/gallery.js` in Task 6.

- [ ] **Step 1: Write failing state tests**

Create `tests/gallery-state.test.js`:

```js
const test = require('node:test')
const assert = require('node:assert/strict')
const GalleryState = require('../src/js/gallery-state')

test('project hashes round-trip safely', () => {
  const hash = GalleryState.projectHash('mavis-demo-generator')
  assert.equal(hash, '#project/mavis-demo-generator')
  assert.deepEqual(GalleryState.parseHash(hash), { type: 'project', id: 'mavis-demo-generator' })
})

test('section hashes parse and invalid hashes are ignored', () => {
  assert.deepEqual(GalleryState.parseHash('#works'), { type: 'section', id: 'works' })
  assert.equal(GalleryState.parseHash('#project/'), null)
  assert.equal(GalleryState.parseHash(''), null)
})

test('wheel input chooses the dominant movement', () => {
  assert.equal(GalleryState.horizontalDelta(3, 40), 40)
  assert.equal(GalleryState.horizontalDelta(50, 4), 50)
})

test('wrapPosition keeps endless-corridor offsets inside one cycle', () => {
  assert.equal(GalleryState.wrapPosition(-20, 100), 80)
  assert.equal(GalleryState.wrapPosition(140, 100), 40)
  assert.equal(GalleryState.wrapPosition(40, 0), 40)
})
```

- [ ] **Step 2: Run tests and verify failure**

Run:

```bash
node --test tests/gallery-state.test.js
```

Expected: FAIL because `gallery-state.js` does not exist.

- [ ] **Step 3: Implement the pure state module**

Create `src/js/gallery-state.js`:

```js
(function (root, factory) {
  const api = factory()
  if (typeof module === 'object' && module.exports) module.exports = api
  if (root) root.GalleryState = api
})(typeof window !== 'undefined' ? window : globalThis, function () {
  function projectHash(id) {
    return `#project/${encodeURIComponent(id)}`
  }

  function parseHash(hash) {
    if (!hash) return null
    if (hash.startsWith('#project/')) {
      const encoded = hash.slice('#project/'.length)
      if (!encoded) return null
      return { type: 'project', id: decodeURIComponent(encoded) }
    }
    if (/^#[a-z0-9-]+$/i.test(hash)) return { type: 'section', id: hash.slice(1) }
    return null
  }

  function horizontalDelta(deltaX, deltaY) {
    return Math.abs(deltaX) > Math.abs(deltaY) ? deltaX : deltaY
  }

  function wrapPosition(value, cycleWidth) {
    if (cycleWidth <= 0) return value
    return ((value % cycleWidth) + cycleWidth) % cycleWidth
  }

  return { projectHash, parseHash, horizontalDelta, wrapPosition }
})
```

- [ ] **Step 4: Run tests**

Run:

```bash
node --test tests/gallery-state.test.js
```

Expected: 4 tests PASS.

- [ ] **Step 5: Commit**

```bash
git add src/js/gallery-state.js tests/gallery-state.test.js
git commit -m "feat: add gallery navigation state helpers"
```

---

### Task 6: Add corridor and project-room interactions

**Files:**
- Create: `src/js/gallery.js`
- Create: `tests/gallery-dom.test.js`
- Modify: `src/components/scripts.pug:1-42`

**Interfaces:**
- Consumes: `window.GalleryState`, `.gallery-scroll`, `.gallery-sequence`, `[data-project-id]`, `[data-project-room]`, and `[data-close-room]`.
- Produces: `GalleryController.create(document, window)` returning `{ openProject(id, options), closeProject(options), destroy() }`.

- [ ] **Step 1: Write failing DOM tests**

Create `tests/gallery-dom.test.js`:

```js
const test = require('node:test')
const assert = require('node:assert/strict')
const { JSDOM } = require('jsdom')
global.GalleryState = require('../src/js/gallery-state')
const GalleryController = require('../src/js/gallery')

function fixture(url = 'https://example.test/') {
  const dom = new JSDOM(`<!doctype html><html><body>
    <div class="gallery-scroll" tabindex="0"><div class="gallery-sequence"><section>Gallery</section></div></div>
    <button data-project-id="mavis-demo-generator" aria-controls="room-mavis-demo-generator">Open</button>
    <article id="room-mavis-demo-generator" data-project-room="mavis-demo-generator">
      <button data-close-room>Close</button>
      <a href="https://example.com">Project link</a>
    </article>
  </body></html>`, { url })
  dom.window.matchMedia = () => ({ matches: false })
  return dom
}

test('opening and closing a room restores focus and visibility', () => {
  const dom = fixture()
  const controller = GalleryController.create(dom.window.document, dom.window)
  const trigger = dom.window.document.querySelector('[data-project-id]')
  const room = dom.window.document.querySelector('[data-project-room]')
  trigger.focus()
  controller.openProject('mavis-demo-generator', { updateHistory: false })
  assert.equal(room.hidden, false)
  assert.equal(dom.window.document.body.classList.contains('project-room-open'), true)
  controller.closeProject({ updateHistory: false })
  assert.equal(room.hidden, true)
  assert.equal(dom.window.document.activeElement, trigger)
})

test('Escape closes an open project room', () => {
  const dom = fixture()
  GalleryController.create(dom.window.document, dom.window)
  dom.window.document.querySelector('[data-project-id]').click()
  dom.window.document.dispatchEvent(new dom.window.KeyboardEvent('keydown', { key: 'Escape', bubbles: true }))
  assert.equal(dom.window.document.querySelector('[data-project-room]').hidden, true)
})

test('pointer dragging moves the desktop corridor', () => {
  const dom = fixture()
  GalleryController.create(dom.window.document, dom.window)
  const corridor = dom.window.document.querySelector('.gallery-scroll')
  corridor.scrollLeft = 0
  corridor.dispatchEvent(new dom.window.MouseEvent('pointerdown', { clientX: 300, button: 0, bubbles: true }))
  corridor.dispatchEvent(new dom.window.MouseEvent('pointermove', { clientX: 180, bubbles: true }))
  corridor.dispatchEvent(new dom.window.MouseEvent('pointerup', { bubbles: true }))
  assert.equal(corridor.scrollLeft, 120)
})

test('a direct project URL opens safely and closes to the works section', () => {
  const dom = fixture('https://example.test/#project/mavis-demo-generator')
  const controller = GalleryController.create(dom.window.document, dom.window)
  const room = dom.window.document.querySelector('[data-project-room]')
  assert.equal(room.hidden, false)
  controller.closeProject()
  assert.equal(dom.window.location.hash, '#works')
  assert.equal(room.hidden, true)
})
```

- [ ] **Step 2: Run the focused tests and verify failure**

Run:

```bash
node --test tests/gallery-dom.test.js
```

Expected: FAIL because `src/js/gallery.js` does not exist.

- [ ] **Step 3: Implement the controller**

Create `src/js/gallery.js` with this public contract:

```js
(function (root, factory) {
  const api = factory(root && root.GalleryState, root)
  if (typeof module === 'object' && module.exports) module.exports = api
  if (root) root.GalleryController = api
})(typeof window !== 'undefined' ? window : globalThis, function (state, root) {
  function create(doc, win) {
    const corridor = doc.querySelector('.gallery-scroll')
    const sequence = corridor && corridor.querySelector('.gallery-sequence')
    const triggers = Array.from(doc.querySelectorAll('[data-project-id]'))
    const rooms = Array.from(doc.querySelectorAll('[data-project-room]'))
    let activeTrigger = null
    let activeRoom = null
    let savedScrollLeft = 0
    let cycleWidth = 0
    let dragStartX = 0
    let dragStartScroll = 0
    let dragging = false
    let ownsProjectHistoryEntry = false
    const clones = []

    function cloneSequence() {
      const clone = sequence.cloneNode(true)
      clone.dataset.galleryClone = ''
      clone.setAttribute('aria-hidden', 'true')
      clone.inert = true
      clone.querySelectorAll('[id]').forEach(node => node.removeAttribute('id'))
      clone.querySelectorAll('a, button, [tabindex]').forEach(node => node.setAttribute('tabindex', '-1'))
      return clone
    }

    function prepareEndlessCorridor() {
      if (!corridor || !sequence || win.matchMedia('(max-width: 720px)').matches) return
      const before = cloneSequence()
      const after = cloneSequence()
      clones.push(before, after)
      corridor.prepend(before)
      corridor.append(after)
      cycleWidth = sequence.getBoundingClientRect().width || sequence.scrollWidth
      corridor.scrollLeft = cycleWidth
    }

    function normalizeEndlessPosition() {
      if (!cycleWidth || dragging) return
      if (corridor.scrollLeft < cycleWidth || corridor.scrollLeft >= cycleWidth * 2) {
        corridor.scrollLeft = cycleWidth + state.wrapPosition(corridor.scrollLeft - cycleWidth, cycleWidth)
      }
    }

    function openProject(id, options = { updateHistory: true }) {
      const room = rooms.find(item => item.dataset.projectRoom === id)
      const trigger = triggers.find(item => item.dataset.projectId === id)
      if (!room || !trigger) return false
      activeTrigger = trigger
      activeRoom = room
      savedScrollLeft = corridor ? corridor.scrollLeft : 0
      room.hidden = false
      room.setAttribute('aria-hidden', 'false')
      doc.body.classList.add('project-room-open')
      const focusTarget = room.querySelector('[data-close-room]') || room
      focusTarget.focus()
      if (options.updateHistory && state) {
        win.history.pushState({ projectId: id }, '', state.projectHash(id))
        ownsProjectHistoryEntry = true
      } else if (Object.hasOwn(options, 'ownsHistoryEntry')) {
        ownsProjectHistoryEntry = options.ownsHistoryEntry
      }
      return true
    }

    function closeProject(options = { updateHistory: true }) {
      if (!activeRoom) return false
      activeRoom.hidden = true
      activeRoom.setAttribute('aria-hidden', 'true')
      doc.body.classList.remove('project-room-open')
      if (corridor) corridor.scrollLeft = savedScrollLeft
      if (activeTrigger) activeTrigger.focus()
      activeRoom = null
      activeTrigger = null
      const shouldGoBack = ownsProjectHistoryEntry
      ownsProjectHistoryEntry = false
      if (options.updateHistory) {
        if (shouldGoBack) win.history.back()
        else win.history.replaceState({}, '', '#works')
      }
      return true
    }

    function onWheel(event) {
      if (!corridor || win.matchMedia('(max-width: 720px)').matches) return
      corridor.scrollLeft += state.horizontalDelta(event.deltaX, event.deltaY)
      event.preventDefault()
    }

    function onPointerDown(event) {
      if (!corridor || event.button !== 0 || win.matchMedia('(max-width: 720px)').matches) return
      dragging = true
      dragStartX = event.clientX
      dragStartScroll = corridor.scrollLeft
      corridor.classList.add('is-dragging')
      if (corridor.setPointerCapture && event.pointerId != null) corridor.setPointerCapture(event.pointerId)
    }

    function onPointerMove(event) {
      if (!dragging) return
      corridor.scrollLeft = dragStartScroll + dragStartX - event.clientX
      event.preventDefault()
    }

    function onPointerUp() {
      if (!dragging) return
      dragging = false
      corridor.classList.remove('is-dragging')
      normalizeEndlessPosition()
    }

    function onKeydown(event) {
      if (event.key === 'Escape') closeProject()
      if (!activeRoom && corridor && event.key === 'ArrowRight') corridor.scrollLeft += 120
      if (!activeRoom && corridor && event.key === 'ArrowLeft') corridor.scrollLeft -= 120
    }

    function onPopstate() {
      const parsed = state.parseHash(win.location.hash)
      if (parsed && parsed.type === 'project') {
        openProject(parsed.id, {
          updateHistory: false,
          ownsHistoryEntry: Boolean(win.history.state && win.history.state.projectId)
        })
      }
      else closeProject({ updateHistory: false })
    }

    rooms.forEach(room => {
      room.hidden = true
      room.setAttribute('role', 'dialog')
      room.setAttribute('aria-modal', 'true')
      room.setAttribute('aria-hidden', 'true')
    })
    triggers.forEach(trigger => trigger.addEventListener('click', () => openProject(trigger.dataset.projectId)))
    rooms.forEach(room => room.querySelector('[data-close-room]').addEventListener('click', () => closeProject()))
    if (corridor) {
      prepareEndlessCorridor()
      corridor.addEventListener('wheel', onWheel, { passive: false })
      corridor.addEventListener('scroll', normalizeEndlessPosition)
      corridor.addEventListener('pointerdown', onPointerDown)
      corridor.addEventListener('pointermove', onPointerMove)
      corridor.addEventListener('pointerup', onPointerUp)
      corridor.addEventListener('pointercancel', onPointerUp)
    }
    doc.addEventListener('keydown', onKeydown)
    win.addEventListener('popstate', onPopstate)

    const parsed = state && state.parseHash(win.location.hash)
    if (parsed && parsed.type === 'project') {
      openProject(parsed.id, {
        updateHistory: false,
        ownsHistoryEntry: Boolean(win.history.state && win.history.state.projectId)
      })
    }

    return {
      openProject,
      closeProject,
      destroy() {
        if (corridor) {
          corridor.removeEventListener('wheel', onWheel)
          corridor.removeEventListener('scroll', normalizeEndlessPosition)
          corridor.removeEventListener('pointerdown', onPointerDown)
          corridor.removeEventListener('pointermove', onPointerMove)
          corridor.removeEventListener('pointerup', onPointerUp)
          corridor.removeEventListener('pointercancel', onPointerUp)
        }
        clones.forEach(clone => clone.remove())
        doc.removeEventListener('keydown', onKeydown)
        win.removeEventListener('popstate', onPopstate)
      }
    }
  }

  if (root && root.document) root.addEventListener('DOMContentLoaded', () => create(root.document, root))
  return { create }
})
```

- [ ] **Step 4: Load scripts in order**

Append to `src/components/scripts.pug` after `main.js`:

```pug
script(src='js/gallery-state.js')
script(src='js/gallery.js')
```

- [ ] **Step 5: Run tests and build**

Run:

```bash
npm test
npm run build
```

Expected: all tests PASS; `dist/js/gallery-state.js` and `dist/js/gallery.js` exist.

- [ ] **Step 6: Commit**

```bash
git add src/js/gallery.js src/components/scripts.pug tests/gallery-dom.test.js dist/js/gallery.js dist/js/gallery-state.js dist/index.html
git commit -m "feat: add accessible gallery interactions"
```

---

### Task 7: Add no-JavaScript, WebGL, link, and motion fallbacks

**Files:**
- Modify: `src/components/head.pug`
- Modify: `src/components/scripts.pug`
- Modify: `src/css/gallery/gallery.less`
- Modify: `src/js/gallery.js`
- Modify: `tests/build-output.test.js`
- Modify: `tests/gallery-dom.test.js`

**Interfaces:**
- Consumes: root `.no-js` / `.js` class, optional `portfolio.resume.url`, WebGL canvas, and existing gallery DOM.
- Produces: readable vertical `.no-js` output, static cover fallback, no dead resume link, and safe missing-image behavior.

- [ ] **Step 1: Add failing fallback assertions**

Append to `tests/build-output.test.js`:

```js
assert.match(html, /<html[^>]*class="no-js"/)
assert.match(html, /documentElement\.classList\.replace\(['"]no-js['"],['"]js['"]\)/)
assert.doesNotMatch(html, />下载简历</)
assert.match(html, /mailto:51284102015@stu\.ecnu\.edu\.cn/)
```

Append to `tests/gallery-dom.test.js`:

```js
test('unknown project ids fail closed without changing the page', () => {
  const dom = fixture()
  const controller = GalleryController.create(dom.window.document, dom.window)
  assert.equal(controller.openProject('missing-project', { updateHistory: false }), false)
  assert.equal(dom.window.document.body.classList.contains('project-room-open'), false)
})
```

- [ ] **Step 2: Run tests and verify at least one failure**

Run:

```bash
npm test
```

Expected: FAIL because the `no-js` switch is absent.

- [ ] **Step 3: Switch `no-js` early and add meaningful metadata**

Inside `src/components/head.pug`, immediately after the existing metadata declarations, add:

```pug
script.
  document.documentElement.classList.replace('no-js','js')
```

Keep the title and description from `config.json`; do not add remote analytics.

- [ ] **Step 4: Add fallback styles**

Append to `src/css/gallery/gallery.less`:

```less
.no-js .content-intro { display: none; }
.no-js .gallery-shell { position: relative; min-height: 100vh; overflow: visible; }
.no-js .gallery-scroll { display: block; height: auto; overflow: visible; }
.no-js .gallery-sequence { display: block; min-width: 0; }
.no-js .gallery-panel { min-width: 0; }
.no-js .project-room { position: relative; inset: auto; display: block; margin: 24px; box-shadow: none; }
.no-js .project-room-close { display: none; }
.webgl-unavailable .content-intro { background: url('../assets/background.png') center / cover no-repeat #1e1f21; }
.artwork-image.is-missing { background: linear-gradient(135deg, var(--art-a), var(--art-b), var(--art-c)); }
```

- [ ] **Step 5: Mark WebGL and image failures without blocking navigation**

In `src/components/scripts.pug`, before loading `background.js`, add an inline WebGL capability check that only sets a class:

```pug
script.
  try {
    const probe = document.createElement('canvas')
    if (!probe.getContext('webgl') && !probe.getContext('experimental-webgl')) document.documentElement.classList.add('webgl-unavailable')
  } catch (error) {
    document.documentElement.classList.add('webgl-unavailable')
  }
```

In `src/js/gallery.js`, register image errors only if real `<img>` artwork is added later:

```js
doc.querySelectorAll('.artwork-image img').forEach(image => {
  image.addEventListener('error', () => image.parentElement.classList.add('is-missing'))
})
```

- [ ] **Step 6: Run tests and build**

Run:

```bash
npm test
npm run build
```

Expected: all tests PASS; no resume link appears while `resume.url` is `null`; email and GitHub remain available.

- [ ] **Step 7: Commit**

```bash
git add src/components/head.pug src/components/scripts.pug src/css/gallery/gallery.less src/js/gallery.js tests/build-output.test.js tests/gallery-dom.test.js dist/index.html dist/css/style.css dist/js/gallery.js
git commit -m "feat: add resilient portfolio fallbacks"
```

---

### Task 8: Make build, watch, and GitHub Pages deployment reproducible

**Files:**
- Modify: `gulpfile.js:16-74`
- Modify: `.github/workflows/deploy-pages.yml`
- Modify: `DEPLOY_GITHUB_PAGES.md`
- Modify: `tests/build-output.test.js`

**Interfaces:**
- Consumes: locked npm dependencies, `npm test`, `npm run validate:data`, and generated `dist/`.
- Produces: deterministic local and CI builds using Node 20 and `npm ci`.

- [ ] **Step 1: Add a failing watch/build-contract assertion**

Append to `tests/build-output.test.js`:

```js
const gulpfile = fs.readFileSync('gulpfile.js', 'utf8')
assert.match(gulpfile, /src\/css\/\*\*\/\*\.less/)
assert.match(gulpfile, /src\/data\/portfolio\.json/)
```

- [ ] **Step 2: Run the focused test and verify it fails**

Run:

```bash
node --test tests/build-output.test.js
```

Expected: FAIL because the current watcher incorrectly watches `*.scss` and does not watch portfolio data.

- [ ] **Step 3: Fix clean and watch tasks**

Update Gulp tasks to use these paths:

```js
gulp.task('clean', function () {
  return del(['./dist/css/', './dist/js/', './dist/index.html'])
})

gulp.task('watch', function () {
  gulp.watch('./src/components/*.pug', gulp.parallel('pug'))
  gulp.watch('./src/index.pug', gulp.parallel('pug'))
  gulp.watch('./src/data/portfolio.json', gulp.parallel('pug'))
  gulp.watch('./src/css/**/*.less', gulp.parallel('css'))
  gulp.watch('./src/js/*.js', gulp.parallel('js'))
  connect.server({ root: 'dist', livereload: true, port: 8080 })
})
```

- [ ] **Step 4: Gate deployment on validation, tests, and build**

In `.github/workflows/deploy-pages.yml`, replace install/build steps with:

```yaml
      - name: Install dependencies
        run: npm ci

      - name: Validate portfolio content
        run: npm run validate:data

      - name: Run tests
        run: npm test

      - name: Build site
        run: npm run build
```

- [ ] **Step 5: Replace deployment examples with the selected repository**

In `DEPLOY_GITHUB_PAGES.md`, state:

```markdown
Repository: `aaaloooooof/personalwebsite`
Expected Pages URL: `https://aaaloooooof.github.io/personalwebsite/`

Before publishing, run:

```bash
npm ci
npm test
npm run build
```
```

- [ ] **Step 6: Run the CI-equivalent sequence**

Run:

```bash
npm ci
npm run validate:data
npm test
npm run build
```

Expected: every command exits `0` from a clean dependency install.

- [ ] **Step 7: Commit**

```bash
git add gulpfile.js .github/workflows/deploy-pages.yml DEPLOY_GITHUB_PAGES.md tests/build-output.test.js dist
git commit -m "ci: verify portfolio before Pages deploy"
```

---

### Task 9: Complete visual, keyboard, mobile, and performance acceptance

**Files:**
- Create: `docs/testing/gallery-acceptance.md`
- Modify as findings require: `src/css/gallery/gallery.less`, `src/js/gallery.js`, related tests

**Interfaces:**
- Consumes: complete built site served from `dist/` at `http://localhost:8080`.
- Produces: a checked acceptance record and a release-ready build with no known blocking defects.

- [ ] **Step 1: Create the acceptance record before testing**

Create `docs/testing/gallery-acceptance.md`:

```markdown
# Gallery Acceptance Record

Date: 2026-08-01

## Desktop 1440px and 1280px

- [ ] Fluid cover is visually unchanged and ENTER works.
- [ ] Mouse wheel, trackpad, and pointer dragging move the corridor horizontally.
- [ ] Crossing either corridor boundary wraps without exposing an empty end.
- [ ] INDEX, contact, and resume state remain reachable.
- [ ] Each of the three project artworks opens the correct room.
- [ ] Closing a room restores corridor position and focus.

## Mobile 390px

- [ ] Gallery becomes vertical with no forced horizontal dragging.
- [ ] Toolbar remains usable without covering content.
- [ ] Project rooms fit the viewport and scroll vertically.

## Keyboard and motion

- [ ] Tab order and focus outlines are visible.
- [ ] Left and Right arrows move the corridor.
- [ ] Escape closes a project room.
- [ ] Browser Back closes a project room or returns to the previous section.
- [ ] Reduced-motion mode removes nonessential transitions.

## Reliability

- [ ] JavaScript-disabled output exposes all core content.
- [ ] WebGL-disabled output exposes a static cover and working ENTER path.
- [ ] Email and GitHub links work.
- [ ] Resume action is hidden while no PDF exists.
- [ ] No blocking console errors occur.
- [ ] `npm ci && npm test && npm run build` passes.
```

- [ ] **Step 2: Start the local site**

Run:

```bash
npm run dev
```

Expected: server reports `http://localhost:8080` and remains running for browser checks.

- [ ] **Step 3: Test desktop and mobile behavior**

Use browser tools at 1440px, 1280px, and 390px. Check each item in `docs/testing/gallery-acceptance.md`; capture a screenshot of the cover, corridor, one open project room, and the mobile vertical gallery for comparison.

- [ ] **Step 4: Test keyboard, history, and reduced motion**

Navigate without a mouse, open and close every room, use Back/Forward, and emulate `prefers-reduced-motion: reduce`. If a check fails, first add a failing automated test when the behavior is deterministic, then apply the smallest source fix and rerun `npm test`.

- [ ] **Step 5: Test fallbacks and links**

Disable JavaScript for one reload, simulate unavailable WebGL, verify no resume link is rendered, and open the email/GitHub links. Mark only observed passing items as complete.

- [ ] **Step 6: Run final verification**

Run:

```bash
npm ci
npm test
npm run build
git diff --check
git status --short
```

Expected: dependency install, tests, build, and diff check all exit `0`; status contains only the intended acceptance/fix files.

- [ ] **Step 7: Commit**

```bash
git add docs/testing/gallery-acceptance.md src/css/gallery/gallery.less src/js/gallery.js tests dist
git commit -m "test: verify portfolio gallery experience"
```

---

## Completion Criteria

- All nine tasks have independently passing tests and commits.
- `npm ci`, `npm run validate:data`, `npm test`, and `npm run build` pass on Node 20.
- The original fluid cover remains visually and behaviorally intact.
- Desktop uses the approved horizontal endless corridor.
- Project artworks open independent vertical viewing rooms and restore position/focus when closed.
- Mobile uses a readable vertical gallery.
- JavaScript, WebGL, image, motion, and missing-resume fallbacks behave as specified.
- GitHub Pages workflow gates deployment on validation, tests, and build.
- No content implies nonexistent internships or unsupported business results.
