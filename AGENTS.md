# Agent Onboarding Guide & Rules (AGENTS.md)

Welcome to the **Netrunner Gateway** codebase! This guide is designed to help LLM agents and developers understand the repository's architecture, build constraints, conventions, and style guidelines to work safely and efficiently.

---

## 🚀 Crucial Project Gotchas (Read This First!)

1. **Custom Build Directory (`docs/` instead of `dist/`) & CNAME**:
   - Astro is configured to compile output directly into the `/docs` folder instead of the default `/dist` folder. This is defined by `outDir: "docs"` in `astro.config.mjs`.
   - **Why?** This allows GitHub Pages to serve the production site directly from the `/docs` directory on the `main` branch.
   - **CNAME Preservation**: Because Astro cleans the output directory on build, a `CNAME` file must be stored in `/public/CNAME` so it is automatically copied to `/docs/CNAME` on compile. Do not manually edit `/docs/CNAME`.
   - **Rule**: Whenever you modify source files, you **must** compile the site using `npm run build` so that the updated static HTML in `/docs/` is generated and can be committed.

2. **CSS Duplication**:
   - The CSS for the site is located in `public/styles.css`.
   - There are identical copies at `styles.css` (root) and `docs/styles.css` (the built output).
   - **Rule**: Always edit `public/styles.css` for styling changes. Astro will copy files in `public/` directly into the `docs/` output folder when you run `npm run build`.

3. **No Test Frameworks**:
   - There are no unit or integration tests (Vitest, Jest, Cypress, etc.) configured in this repository.
   - **Rule**: To verify changes, run the local dev server (`npm run dev`) and visually inspect the changes.

---

## 🛠 Essential Commands

All commands must be run from the root of the project:

| Command | Action |
| :--- | :--- |
| `npm install` | Installs dependencies |
| `npm run dev` | Starts local dev server at `localhost:4321` |
| `npm run build` | Compiles your production site directly to the `./docs/` folder |
| `npm run preview` | Previews the compiled site in `./docs/` locally |
| `npm run astro -- --help` | Get help using the Astro CLI |

---

## 🏗 Directory & Architecture Overview

The repository has the following layout:

```text
/
├── public/
│   ├── favicon.svg
│   └── styles.css          <-- Primary stylesheet to edit!
├── src/
│   ├── assets/             <-- Static vectors/graphics
│   ├── components/
│   │   └── Tile.astro      <-- Standard content container
│   ├── layouts/
│   │   └── Layout.astro    <-- Main site shell, header, footer, & common resources
│   └── pages/
│       ├── index.astro     <-- Home page listing regional meetup tiles
│       └── [region].astro  <-- Specific region files (e.g., ph.astro, de.astro)
├── docs/                   <-- Static build output (served by GitHub Pages)
├── astro.config.mjs        <-- Astro configuration (sets outDir to "docs")
├── package.json            <-- Project metadata & dependencies
└── tsconfig.json           <-- TypeScript configuration
```

### Component Breakdown & Data Flow

1. **`src/layouts/Layout.astro`**:
   - Provides the standard HTML boilerplate, Google Fonts (`Play`), and Google Analytics scripts (`G-3EWPLE39P4`).
   - Standard, global Netrunner resource cards (Null Signal Games, Learn to Play, Play Online, Discord, Cards, Tokens) are hardcoded directly into this layout.
   - The main `<slot />` is positioned between the *Null Signal Games* card and the *Learn to Play* card. Child content from individual pages injects here.

2. **`src/components/Tile.astro`**:
   - This is the standard component used for displaying groups, regions, or venues.
   - It accepts the following props:
     - `containerClass` (string): Suffix for class names, e.g., passing `"manila"` generates `<div class="container manila-container">`.
     - `title` (string): Header displayed in the tile.
     - `cta` (array of objects): A list of Call-To-Action buttons to display, with elements structured as `{ action: "URL", label: "Button Text" }`.
   - Content is slotted inside `<slot />` as standard HTML.

3. **`src/pages/`**:
   - Routing in Astro is file-system based.
   - `index.astro` is the homepage. It renders a list of `Tile` elements representing regions (e.g., Pacific Northwest, Germany, Philippines, etc.), with their `cta` linking to subpages `/pnw`, `/de`, `/ph`, etc.
   - Each regional file (such as `ph.astro` or `de.astro`) uses the common layout `<Layout title="...">` and defines regional/city-specific groups inside custom `Tile`s with direct links to local Facebook pages, Discord servers, and Google Maps venues.

---

## 🎨 Naming Conventions & Styling Patterns

- **Clean URLs**: Always use absolute path-relative URLs (e.g., `/pnw`, `/ph`, `/de`) when linking between pages in the site. Astro translates these clean paths during build by placing `index.html` inside subfolders (e.g., `/docs/pnw/index.html`).
- **Styling**: Standard plain CSS is used. Do not attempt to use Tailwind or SASS unless you install and configure them first (though staying vanilla is strongly preferred).
- **Theme**: Cyberpunk-inspired (dark background `#000000`, vibrant neon pink `#ff2e95`, violet `#5e5bd8`, and linear gradients).
- **Buttons**: Styling for links marked with the `.button` class uses a pseudo-element (`::before` and `::after`) trick to give a cybernetic, corner-cut border look. Be careful when modifying button styles, as these selectors rely on absolute positions and hardcoded border colors matching the surrounding containers.

---

## 📝 Guide on Adding a New Region

If a user asks you to add a new region meetup (e.g., "Chicago"):

1. **Create the Page**: Create `src/pages/chicago.astro`.
2. **Page Content**:
   - Use the `<Layout>` and `<Tile>` components.
   - Structure details logically: Use `<h3>` for venue/meetup names, `<p>` for meeting times, and include Google Maps links.
   - Provide CTAs to their local community platforms (Discord/Facebook/etc.).
3. **Add to Homepage**: Open `src/pages/index.astro` and add a new `<Tile>` linking to `/chicago`.
4. **Compile & Deploy**:
   - Run `npm run build` to generate the static files in `/docs/chicago/index.html`.
   - Verify the homepage links to `/chicago` correctly and the page renders nicely.

---

## 🤖 Autodeploy Integration (CI/CD)

The project has an automated GitHub Actions workflow configured in `.github/workflows/deploy.yml` that:
- Triggers on push to `main` branch (whenever Astro source code/assets are changed, ignoring automatic build updates to `/docs`).
- Automatically installs dependencies, runs `npm run build` (safely copying the `public/CNAME` domain config to `docs/CNAME`), and commits the compiled build folder back to `main`.
- This ensures any push to Astro source files automatically compiles and deploys live to GitHub Pages.

