# Understanding the Codebase: A React + Vite SPA Explainer

This document walks through every part of the "Build an LLM From Scratch" website
codebase. It is written for someone who knows basic JavaScript and Python but is
new to React, Vite, TypeScript, and the modern frontend toolchain. By the end,
you will understand not just *what* each file does, but *why* it is structured
that way.

---

## Table of Contents

1. [The Big Picture — What Is a SPA?](#1-the-big-picture)
2. [The Toolchain — Vite, React, TypeScript, Tailwind](#2-the-toolchain)
3. [Project Structure](#3-project-structure)
4. [Entry Points — `index.html` and `main.tsx`](#4-entry-points)
5. [The App Component and Routing](#5-the-app-component-and-routing)
6. [Pages — Home, Article, About, NotFound](#6-pages)
7. [Components — Reusable Building Blocks](#7-components)
8. [Data Layer — `articles.ts` and Content Files](#8-data-layer)
9. [Styling — Tailwind CSS and `index.css`](#9-styling)
10. [Diagrams — Inline SVG Components](#10-diagrams)
11. [The Build System — `vite.config.ts` vs `vite.config.github.ts`](#11-the-build-system)
12. [GitHub Pages Deployment — The SPA Routing Problem](#12-github-pages-deployment)
13. [Key Patterns to Remember](#13-key-patterns)

---

## 1. The Big Picture — What Is a SPA?

A **Single-Page Application (SPA)** is a website where the browser loads one
HTML file once, and then JavaScript takes over all navigation. When you click
a link from the home page to an article, no new HTTP request is made to the
server. Instead, JavaScript swaps out the content on the page and updates the
browser's URL bar to match.

This is different from a traditional website where every link causes the browser
to fetch a new HTML file from the server.

The advantages of a SPA are speed (no full page reloads), smooth transitions,
and the ability to share state between pages. The main challenge — which this
codebase solves — is that when someone visits a deep link directly
(e.g. `/article/attention`), the server needs to know to serve the main
`index.html` rather than looking for a file called `attention`.

---

## 2. The Toolchain

### Vite

**Vite** is the build tool. During development, it starts a local server that
serves your files instantly using native ES modules — no bundling step needed.
When you run `vite build`, it bundles everything into optimised static files
ready for deployment.

Think of Vite as the engine that powers both the dev server and the production
build. It replaces older tools like Webpack and Create React App.

### React

**React** is the UI library. It lets you describe your UI as a tree of
components. Each component is a JavaScript function that returns JSX (HTML-like
syntax). When data changes, React figures out the minimum set of DOM updates
needed and applies them.

```jsx
// A simple React component
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

### TypeScript

**TypeScript** is JavaScript with type annotations. Instead of:

```js
function add(a, b) { return a + b; }
```

You write:

```ts
function add(a: number, b: number): number { return a + b; }
```

TypeScript catches mistakes at edit time rather than at runtime. In this
codebase, `.tsx` files are TypeScript files that also contain JSX.

### Tailwind CSS

**Tailwind** is a CSS framework where you style elements by adding utility
classes directly in your HTML/JSX rather than writing separate CSS files.

```jsx
// Instead of writing CSS, you compose utility classes:
<div className="flex flex-col gap-4 p-6 bg-gray-900 text-white rounded-lg">
```

Each class does one thing: `flex` sets `display: flex`, `p-6` sets padding,
`bg-gray-900` sets the background colour, and so on.

---

## 3. Project Structure

```
llm-from-scratch-web/
├── client/                    ← All frontend code lives here
│   ├── index.html             ← The one HTML file the browser loads
│   ├── public/                ← Static assets served as-is (images, etc.)
│   │   ├── 404.html           ← GitHub Pages SPA redirect (Part 1: STORE)
│   │   └── assets/
│   │       └── hero-banner.webp
│   └── src/                   ← All JavaScript/TypeScript source
│       ├── main.tsx           ← Entry point — mounts React into index.html
│       ├── App.tsx            ← Root component — sets up routing
│       ├── index.css          ← Global styles and Tailwind configuration
│       ├── pages/             ← One file per route/page
│       │   ├── Home.tsx
│       │   ├── Article.tsx
│       │   ├── About.tsx
│       │   └── NotFound.tsx
│       ├── components/        ← Reusable UI pieces
│       │   ├── Layout.tsx
│       │   ├── Footer.tsx
│       │   ├── CodeBlock.tsx
│       │   ├── ArticleRenderer.tsx
│       │   ├── GradientOrbs.tsx
│       │   └── diagrams/
│       │       └── index.tsx  ← All 21 SVG diagram components
│       ├── lib/               ← Data and utilities
│       │   ├── articles.ts    ← Article metadata and imports
│       │   ├── part1_content.ts  ← Article 1 full text
│       │   ├── part2_content.ts
│       │   ├── ...
│       │   └── part6_content.ts
│       └── contexts/
│           └── ThemeContext.tsx ← Dark/light theme state
├── vite.config.ts             ← Dev server configuration
├── vite.config.github.ts      ← Production build for GitHub Pages
├── package.json               ← Dependencies and scripts
└── tsconfig.json              ← TypeScript compiler settings
```

The key insight is the separation between `pages/` and `components/`. Pages are
full-screen views that correspond to a URL route. Components are reusable pieces
that can appear on multiple pages. This separation keeps the code organised as
the site grows.

---

## 4. Entry Points — `index.html` and `main.tsx`

### `client/index.html`

This is the only HTML file the browser ever loads. It is almost empty on
purpose — React will fill in all the content dynamically.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Build an LLM From Scratch</title>
    <!-- Google Fonts loaded here so they are available globally -->
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk..." />

    <script>
      // SPA redirect restore script (explained in Section 12)
      (function () {
        var redirect = sessionStorage.getItem('spa_redirect');
        if (redirect) {
          sessionStorage.removeItem('spa_redirect');
          var base = '/BuildLLM/'.replace(/\/$/, '');
          window.history.replaceState(null, '', base + redirect);
        }
      })();
    </script>
  </head>
  <body>
    <div id="root"></div>                    <!-- React mounts here -->
    <script type="module" src="/src/main.tsx"></script>  <!-- Entry point -->
  </body>
</html>
```

The `<div id="root">` is the anchor point. React replaces its contents with
the entire application. The `<script type="module">` tag tells the browser to
load `main.tsx` as a JavaScript module — Vite intercepts this and handles the
TypeScript compilation on the fly during development.

### `client/src/main.tsx`

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

// Find the #root div in index.html and mount the React app inside it
ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

`ReactDOM.createRoot` is the bridge between the HTML world and the React world.
It takes the `#root` DOM element and hands it to React to manage. From this
point on, React controls everything inside that div.

`React.StrictMode` is a development helper that runs each component twice to
catch side effects. It has no effect in production builds.

---

## 5. The App Component and Routing

### `client/src/App.tsx`

```tsx
import { Route, Switch, Router as WouterRouter } from "wouter";

// BASE_URL is set by Vite. In dev it is "/". In the GitHub Pages
// build it is "/BuildLLM/". We strip the trailing slash.
const BASE = import.meta.env.BASE_URL.replace(/\/$/, "");

function Router() {
  return (
    <WouterRouter base={BASE}>
      <Switch>
        <Route path="/" component={Home} />
        <Route path="/article/:slug" component={Article} />
        <Route path="/about" component={About} />
        <Route component={NotFound} />
      </Switch>
    </WouterRouter>
  );
}
```

**Wouter** is the routing library. It watches the browser's URL and renders
the matching component. Think of it as a switch statement for URLs:

- If the URL is `/`, render `<Home />`
- If the URL is `/article/attention`, render `<Article />` with `slug="attention"`
- If nothing matches, render `<NotFound />`

The `:slug` in `/article/:slug` is a **route parameter** — a variable part of
the URL. The Article component reads it with `useParams()` to know which
article to display.

The `base={BASE}` prop is critical for GitHub Pages. Without it, wouter sees
the full path `/BuildLLM/article/attention` and finds no match. With it, wouter
strips `/BuildLLM` first and correctly matches `/article/attention`.

**`import.meta.env.BASE_URL`** is a Vite-specific environment variable. Vite
replaces it at build time with the value of the `base` option in your
`vite.config.ts`. In dev mode it is `"/"`. In the GitHub Pages build it is
`"/BuildLLM/"`. This means the same source code works correctly in both
environments without any manual changes.

### Context Providers

The `App` component also wraps everything in several **context providers**:

```tsx
function App() {
  return (
    <ErrorBoundary>
      <ThemeProvider defaultTheme="dark">
        <TooltipProvider>
          <Toaster />
          <Router />
        </TooltipProvider>
      </ThemeProvider>
    </ErrorBoundary>
  );
}
```

A **context** in React is a way to share data across the component tree without
passing it as props through every level. `ThemeProvider` makes the current
theme (dark/light) available to any component anywhere in the tree. Any
component can call `useTheme()` to read or change the theme.

`ErrorBoundary` is a safety net — if any component throws an error, it catches
it and shows a fallback UI instead of a blank page.

---

## 6. Pages

### `client/src/pages/Home.tsx`

The home page is a standard React functional component. It imports data from
`articles.ts` and renders the hero section, stats bar, and article cards.

```tsx
import { articles } from "@/lib/articles";

export default function Home() {
  return (
    <Layout>
      <HeroSection />
      <StatsBar />
      <ArticleCards articles={articles} />
    </Layout>
  );
}
```

The `@/` prefix is a path alias configured in `vite.config.ts`. It maps to
`client/src/`, so `@/lib/articles` means `client/src/lib/articles.ts`. This
avoids messy relative paths like `../../lib/articles`.

### `client/src/pages/Article.tsx`

```tsx
import { useParams } from "wouter";
import { articles } from "@/lib/articles";

export default function Article() {
  const { slug } = useParams<{ slug: string }>();
  const article = articles.find(a => a.id === slug);

  if (!article) return <NotFound />;

  return (
    <Layout>
      <ArticleHeader article={article} />
      <ArticleRenderer content={article.content} slug={slug} />
      <PrevNextNav articles={articles} currentSlug={slug} />
    </Layout>
  );
}
```

`useParams()` is a wouter hook that reads the URL parameters. For the URL
`/article/attention`, it returns `{ slug: "attention" }`. The component then
looks up the matching article in the data array and renders it.

---

## 7. Components

### `client/src/components/Layout.tsx`

The Layout component wraps every page with the shared sidebar, reading progress
bar, and footer. Every page imports and uses it, so changes to the layout
automatically apply everywhere.

```tsx
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex min-h-screen">
      <Sidebar />
      <main className="flex-1">
        <ProgressBar />
        {children}        {/* ← The page content goes here */}
        <Footer />
      </main>
    </div>
  );
}
```

The `children` prop is a special React prop that represents whatever JSX is
placed between the opening and closing tags of a component. This is how Layout
wraps page content without needing to know what that content is.

### `client/src/components/ArticleRenderer.tsx`

This is the most complex component. It takes a markdown string and renders it
as styled HTML, with special handling for `[[DIAGRAM:name]]` markers.

The rendering pipeline works in three steps:

1. **Split on diagram markers.** The content string is split on the pattern
   `[[DIAGRAM:name]]`. This produces an array of alternating text segments and
   diagram names.

2. **Render text segments with `react-markdown`.** Each text segment is passed
   to the `<Markdown>` component, which parses markdown syntax (headings, bold,
   code blocks, lists) and renders it as React elements. Custom renderers are
   provided for code blocks so they use the `<CodeBlock>` component with syntax
   highlighting instead of plain `<pre>` tags.

3. **Render diagram markers.** Each diagram name is looked up in a registry
   object that maps names to SVG components. If found, the component is
   rendered inline. If not found, a warning box is shown.

```tsx
const DIAGRAMS: Record<string, React.ComponentType> = {
  tokenization:        TokenizationDiagram,
  qkv_diagram:         QKVDiagram,
  transformer_block:   TransformerBlockDiagram,
  // ... 18 more
};

function renderSegment(segment: string, slug: string) {
  const diagramMatch = segment.match(/^\[\[DIAGRAM:(.+)\]\]$/);
  if (diagramMatch) {
    const DiagramComponent = DIAGRAMS[diagramMatch[1]];
    return DiagramComponent ? <DiagramComponent /> : <DiagramNotFound />;
  }
  return <Markdown components={customRenderers}>{segment}</Markdown>;
}
```

### `client/src/components/CodeBlock.tsx`

```tsx
export default function CodeBlock({ language, code }: Props) {
  const [copied, setCopied] = useState(false);

  function handleCopy() {
    navigator.clipboard.writeText(code);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  }

  return (
    <div className="relative">
      <div className="language-label">{language.toUpperCase()}</div>
      <button onClick={handleCopy}>{copied ? "Copied!" : "Copy"}</button>
      <pre>
        <code dangerouslySetInnerHTML={{ __html: highlighted }} />
      </pre>
    </div>
  );
}
```

`useState` is a React hook that gives a component its own memory. `copied`
starts as `false`. When the button is clicked, `setCopied(true)` updates it,
which causes React to re-render the component with the new value. The
`setTimeout` resets it to `false` after 2 seconds.

`dangerouslySetInnerHTML` is React's way of injecting raw HTML. It is named
"dangerously" to remind you that injecting untrusted HTML can cause XSS
security vulnerabilities. Here it is safe because the HTML comes from the
syntax highlighter, not from user input.

---

## 8. Data Layer — `articles.ts` and Content Files

### `client/src/lib/articles.ts`

```ts
import { part1Content } from "./part1_content";
import { part2Content } from "./part2_content";
// ...

export interface Article {
  id: string;           // URL slug, e.g. "attention"
  number: number;       // Display number (1-6)
  title: string;
  subtitle: string;
  readTime: string;
  chapters: string[];   // Section headings for the table of contents
  content: string;      // Full markdown text
}

export const articles: Article[] = [
  {
    id: "foundations",
    number: 1,
    title: "Text, Tokens, and the Vocabulary",
    subtitle: "...",
    readTime: "14 min read",
    chapters: ["What Is a Token?", "Building a Vocabulary", ...],
    content: part1Content,
  },
  // ...
];
```

The `interface` keyword defines a TypeScript type. It says: any object that
claims to be an `Article` must have these exact fields with these exact types.
If you try to create an article without a `title`, TypeScript will show an
error immediately in your editor.

The content is kept in separate files (`part1_content.ts` through
`part6_content.ts`) rather than inline in `articles.ts` because each article
is thousands of lines of markdown. Splitting them keeps each file manageable.

### `client/src/lib/part1_content.ts`

```ts
export const part1Content = `
# Text, Tokens, and the Vocabulary

Before a language model can do anything useful, it needs to turn text into
numbers. Computers do not understand words — they understand numbers.

## What Is a Token?

A token is the basic unit of text that the model works with...

\`\`\`python
text = "The cat sat on the mat"
tokens = text.lower().split()
print(tokens)
# ['the', 'cat', 'sat', 'on', 'the', 'mat']
\`\`\`

[[DIAGRAM:tokenization]]

...
`;
```

This is a **template literal** — a JavaScript string delimited by backticks
instead of quotes. Template literals can span multiple lines and contain
embedded expressions. The `\`\`\`` sequences inside are escaped backticks that
become the markdown code fence syntax.

The `[[DIAGRAM:tokenization]]` marker is a custom convention. It is not
standard markdown — the `ArticleRenderer` component intercepts it and replaces
it with the actual SVG diagram component.

---

## 9. Styling — Tailwind CSS and `index.css`

### How Tailwind Works

Tailwind scans all your source files at build time, finds every class name you
used, and generates a CSS file containing only those classes. This means the
final CSS file is small — only the styles you actually used are included.

```jsx
// These Tailwind classes produce the dark article card styling:
<div className="
  bg-slate-900          // dark background
  border border-slate-700  // subtle border
  rounded-lg            // rounded corners
  p-6                   // padding on all sides
  hover:border-indigo-500  // border glows on hover
  transition-colors     // smooth colour transition
  duration-200          // 200ms transition speed
">
```

### `client/src/index.css`

This file does three things:

**1. Imports Tailwind:**
```css
@import "tailwindcss";
@import "tw-animate-css";
```

**2. Defines CSS custom properties (variables) for the colour palette:**
```css
:root {
  --background: oklch(0.12 0.018 265);   /* very dark blue-grey */
  --foreground: oklch(0.92 0.008 250);   /* near-white */
  --primary:    oklch(0.65 0.22 275);    /* electric indigo */
}
```

OKLCH is a modern colour format that is more perceptually uniform than HSL or
hex. The three values are: Lightness (0-1), Chroma (colour intensity), and Hue
(angle on the colour wheel).

**3. Defines base styles:**
```css
@layer base {
  body {
    @apply bg-background text-foreground;
  }
}
```

`@apply` is a Tailwind directive that lets you use utility classes inside CSS
rules. `bg-background` maps to the `--background` CSS variable defined above.

---

## 10. Diagrams — Inline SVG Components

All 21 diagrams live in `client/src/components/diagrams/index.tsx`. Each is a
React component that returns an SVG element.

SVG (Scalable Vector Graphics) is an XML-based format for vector graphics. It
is written directly in HTML/JSX and scales perfectly at any size.

```tsx
export const TokenizationDiagram = () => (
  <svg viewBox="0 0 700 200" xmlns="http://www.w3.org/2000/svg">
    {/* Background */}
    <rect width="700" height="200" fill="oklch(0.16 0.018 265)" rx="8" />

    {/* Title */}
    <text x="350" y="30" textAnchor="middle" fill="oklch(0.85 0.008 250)"
          fontSize="14" fontFamily="Space Grotesk, sans-serif">
      Tokenization: "The cat sat"
    </text>

    {/* Token boxes */}
    {["The", "cat", "sat"].map((word, i) => (
      <g key={word} transform={`translate(${100 + i * 170}, 60)`}>
        <rect width="120" height="40" fill="oklch(0.25 0.05 275)" rx="4" />
        <text x="60" y="26" textAnchor="middle" fill="white" fontSize="14">
          {word}
        </text>
      </g>
    ))}
  </svg>
);
```

SVG coordinates start at the top-left corner (0, 0). The `viewBox` attribute
defines the coordinate system — `0 0 700 200` means the SVG is 700 units wide
and 200 units tall internally, regardless of how large it is displayed on screen.

Using React components for diagrams (rather than image files) means the
diagrams are part of the JavaScript bundle, load instantly, are perfectly
sharp at any screen resolution, and can be styled with the same CSS variables
as the rest of the site.

---

## 11. The Build System

### `vite.config.ts` (development)

```ts
export default defineConfig({
  base: "/",                    // Root path in development
  plugins: [react(), tailwindcss()],
  root: path.resolve(__dirname, "client"),
  build: {
    outDir: path.resolve(__dirname, "dist/public"),
  },
});
```

### `vite.config.github.ts` (GitHub Pages production)

```ts
export default defineConfig({
  base: "/BuildLLM/",           // Sub-path on GitHub Pages
  plugins: [react(), tailwindcss()],
  root: path.resolve(__dirname, "client"),
  build: {
    outDir: path.resolve(__dirname, "client/dist-github"),
    emptyOutDir: true,
  },
});
```

The only meaningful difference is `base`. This value is injected into the
built HTML and JavaScript as `import.meta.env.BASE_URL`. Every asset URL
(CSS, JS, images) is automatically prefixed with it. The router reads it to
strip the prefix before matching routes.

Having two config files means you never need to remember to change a setting
before building. Run `vite build` for dev/Manus deployment, run
`vite build --config vite.config.github.ts` for GitHub Pages.

---

## 12. GitHub Pages Deployment — The SPA Routing Problem

This is the most important concept to understand when deploying any SPA to a
static host.

### The Problem

GitHub Pages is a static file server. It serves files from a directory. When
a browser requests `/BuildLLM/article/attention`, the server looks for a file
at that path. There is no such file — there is only `index.html` at the root.
So the server returns a 404 error.

This works fine when you navigate *within* the site (clicking links), because
the browser never actually requests `/article/attention` from the server — the
JavaScript router handles it internally. But it breaks when you:

- Visit a URL directly (bookmark, shared link)
- Refresh the page on any route other than the root

### The Solution: Two-File Redirect

**File 1: `client/public/404.html` (the STORE step)**

GitHub Pages serves this file whenever it cannot find a requested path. We put
a script in it that:
1. Reads the URL the user was trying to reach
2. Saves the path portion to `sessionStorage`
3. Redirects to the root (`/BuildLLM/?spa=1`)

```html
<script>
  (function () {
    var base = '/BuildLLM';
    var path = location.pathname.slice(base.length) || '/';
    sessionStorage.setItem('spa_redirect', path + location.search + location.hash);
    location.replace(base + '/?spa=1');
  })();
</script>
```

**File 2: `client/index.html` (the RESTORE step)**

Before React mounts, a script checks `sessionStorage`. If a saved path is
found, it calls `history.replaceState` to silently rewrite the URL back to the
original destination. React then mounts and the router sees the correct URL.

```html
<script>
  (function () {
    var redirect = sessionStorage.getItem('spa_redirect');
    if (redirect) {
      sessionStorage.removeItem('spa_redirect');
      var base = '/BuildLLM/'.replace(/\/$/, '');
      window.history.replaceState(null, '', base + redirect);
    }
  })();
</script>
```

**The router: `base` prop in `App.tsx`**

Even with the URL correctly restored, the router still needs to know about the
base path. Wouter's `<Router base="/BuildLLM">` strips the prefix before
matching, so `/BuildLLM/article/attention` correctly matches the
`/article/:slug` route.

### Why `client/public/` for `404.html`?

Files placed in `client/public/` are copied verbatim to the build output
without being processed by Vite. This is exactly what we want — `404.html`
must be a plain HTML file with inline JavaScript, not a React component.

### The `.nojekyll` File

GitHub Pages runs Jekyll (a static site generator) on your files by default.
Jekyll ignores files and folders that start with an underscore. Vite's build
output includes an `assets/` folder with files like `_app.js`. The `.nojekyll`
file (an empty file at the root of the `gh-pages` branch) tells GitHub Pages
to skip Jekyll processing entirely.

---

## 13. Key Patterns to Remember

| Pattern | What it does | Where it appears |
|---|---|---|
| `useState(initialValue)` | Gives a component local memory that triggers re-renders when changed | `CodeBlock.tsx` (copy button state) |
| `useEffect(() => {}, [deps])` | Runs side effects after render; re-runs when `deps` change | Reading progress bar in `Layout.tsx` |
| `useParams()` | Reads URL parameters from the current route | `Article.tsx` (reads `:slug`) |
| `children` prop | Passes JSX content into a wrapper component | `Layout.tsx` wraps every page |
| Context + Provider | Shares state across the component tree without prop drilling | `ThemeContext.tsx` |
| `import.meta.env.BASE_URL` | Vite-injected base path, changes between dev and production builds | `App.tsx` router base |
| `@/` path alias | Maps to `client/src/`, avoids `../../` relative paths | Every import in `src/` |
| `interface` / TypeScript types | Defines the shape of data objects, catches errors at edit time | `articles.ts` Article interface |
| Template literals (backtick strings) | Multi-line strings, used for article markdown content | All `part*_content.ts` files |
| SVG in JSX | Inline vector graphics, sharp at any size, styled with CSS variables | All diagram components |
| `dangerouslySetInnerHTML` | Injects raw HTML into a component (use only with trusted content) | `CodeBlock.tsx` syntax highlighting |
| `emptyOutDir: true` | Clears the output folder before each build to avoid stale files | `vite.config.github.ts` |
| `.nojekyll` | Disables Jekyll on GitHub Pages, required for Vite builds | Root of `gh-pages` branch |
| `history.replaceState` | Changes the browser URL without triggering a page load | SPA redirect restore script |
| `sessionStorage` | Browser key-value store that persists for the current tab session | SPA redirect store/restore |

---

## Where to Go Next

If you want to extend this site or build your own SPA, here are the most
useful things to learn next:

**React:** The official React docs at [react.dev](https://react.dev) are
excellent. Focus on hooks (`useState`, `useEffect`, `useContext`, `useMemo`)
and the component lifecycle.

**Vite:** The [Vite guide](https://vite.dev/guide/) covers environment
variables, static assets, and the build pipeline in depth.

**TypeScript:** The [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
is the definitive reference. Start with "Everyday Types" and "Narrowing".

**Tailwind CSS:** The [Tailwind docs](https://tailwindcss.com/docs) have a
searchable reference for every utility class. The "Core Concepts" section
explains the philosophy.

**Wouter:** Wouter's [README on GitHub](https://github.com/molefrog/wouter)
is short and covers everything — routes, params, programmatic navigation,
and the `base` option used in this project.
