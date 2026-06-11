<img src="./svg-insert-logo.jpg" alt="svgInsert logo" style="max-width: 360px">

# svg-insert

Drop an SVG into your HTML like an image. Get `currentColor`, CSS variables, and your full cascade for free.

---

## Why svg-insert ?

`<img src="logo.svg">` renders an SVG as a replaced element. It is isolated from the page's CSS: `currentColor`, CSS custom properties, and inherited styles do not reach inside it. The only way to make an SVG respond to the page's theme or accent color is to inline it.

Inlining SVGs by hand is tedious and bloats your HTML. `svg-insert` does it at runtime:

- **Inherits CSS.** The inlined `<svg>` sits in the normal DOM, so `currentColor`, `fill: inherit`, and CSS variables all work.
- **Clean markup.** Write `<svg-insert src="/icons/logo.svg">` instead of pasting hundreds of SVG nodes by hand.
- **Attribute forwarding.** Pass `id`, `class`, `aria-label`, `viewBox`, or any attribute to the inserted `<svg>` via `svg-*` prefixed attributes on the wrapper. No JavaScript needed.
- **Optional wrapper.** By default the `<svg-insert>` element is replaced entirely. Add `keep-parent` to keep it as a container.
- **Dynamic.** Change `src` at runtime and the element re-fetches and re-injects automatically.
- **No build step.** Drop one `<script type="module">` in your HTML and it works.

---

## Installation

### Package managers

```bash
npm install @nativelayer.dev/svg-insert
```

```bash
pnpm add @nativelayer.dev/svg-insert
```

```bash
yarn add @nativelayer.dev/svg-insert
```

Then import and register:

```js
import svgInsert from '@nativelayer.dev/svg-insert'
customElements.define('svg-insert', svgInsert)
```

### CDN

No install. Paste directly into HTML:

```html
<script type="module">
  import svgInsert from 'https://cdn.jsdelivr.net/npm/@nativelayer.dev/svg-insert/index.js'
  customElements.define('svg-insert', svgInsert)
</script>
```

---

## Usage

### Default: replace the wrapper

By default, `<svg-insert>` is completely replaced by the fetched `<svg>` element. The wrapper disappears from the DOM.

```html
<svg-insert src="/icons/logo.svg"></svg-insert>
```

After (in live DOM):

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <!-- SVG content -->
</svg>
```

### Keep the wrapper with `keep-parent`

Add `keep-parent` if you need the `<svg-insert>` element to remain in the DOM as a container.

```html
<svg-insert src="/icons/logo.svg" keep-parent></svg-insert>
```

After (in live DOM):

```html
<svg-insert keep-parent>
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <!-- SVG content -->
  </svg>
</svg-insert>
```

### Forwarding attributes to the `<svg>` with `svg-*`

Any attribute on `<svg-insert>` that starts with `svg-` is automatically stripped of the prefix and applied to the inserted `<svg>` element.

```html
<svg-insert
  src="/icons/arrow.svg"
  svg-id="nav-arrow"
  svg-class="icon icon--small"
  svg-aria-label="Navigate forward"
  svg-role="img"
></svg-insert>
```

After (in live DOM):

```html
<svg
  xmlns="http://www.w3.org/2000/svg"
  id="nav-arrow"
  class="icon icon--small"
  aria-label="Navigate forward"
  role="img"
>
  <!-- SVG content -->
</svg>
```

---

## Core API

### Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `src` | string | - | URL of the SVG file to fetch and inline. Changing this at runtime triggers a new fetch. |
| `keep-parent` | boolean | false | When present, the `<svg-insert>` wrapper is kept in the DOM and the `<svg>` is appended inside it. When absent, the wrapper is replaced entirely by the `<svg>`. |
| `svg-*` | string | - | Any attribute prefixed with `svg-` is forwarded to the inserted `<svg>` element with the prefix removed. |

### Observed attributes

`svg-insert` re-fetches and re-injects when `src` changes. All `svg-*` attributes are read and applied at fetch time, not observed individually after injection.

### DOM transform

| Mode | `keep-parent` | Result in DOM |
|------|--------------|---------------|
| Replace (default) | absent | `<svg-insert>` is gone; `<svg>` is in its place |
| Container | present | `<svg-insert>` remains; `<svg>` is its only child |

---

## Examples

### Icon inheriting `currentColor`

```html
<style>
  .icon { fill: currentColor; width: 1em; height: 1em; }
</style>

<button style="color: hsl(220 80% 55%);">
  <svg-insert src="/icons/star.svg" svg-class="icon"></svg-insert>
  Favourite
</button>
```

### Themed SVG with CSS custom properties

```html
<style>
  :root { --brand: hsl(260 70% 50%); }
  .brand-icon path { fill: var(--brand); }
</style>

<svg-insert src="/img/logo.svg" svg-class="brand-icon" keep-parent></svg-insert>
```

Because the SVG is inlined, `var(--brand)` resolves correctly inside it. This is impossible with `<img>`.

### Accessible SVG icon

```html
<svg-insert
  src="/icons/close.svg"
  svg-aria-label="Close dialog"
  svg-role="img"
  svg-focusable="false"
></svg-insert>
```

### Runtime `src` swap

```html
<!-- use keep-parent so the element stays in the DOM after first inject -->
<svg-insert id="icon" src="/icons/moon.svg" keep-parent></svg-insert>

<button id="toggle">Toggle theme</button>

<script type="module">
  const icon = document.getElementById('icon')
  const btn  = document.getElementById('toggle')
  let dark = true

  btn.addEventListener('click', () => {
    dark = !dark
    icon.setAttribute('src', dark ? '/icons/moon.svg' : '/icons/sun.svg')
  })
</script>
```

### Using `keep-parent` for layout control

```html
<style>
  .icon-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background: hsl(220 100% 95%);
  }
  .icon-wrapper svg {
    width: 24px;
    height: 24px;
    fill: hsl(220 80% 45%);
  }
</style>

<svg-insert
  src="/icons/bell.svg"
  keep-parent
  class="icon-wrapper"
></svg-insert>
```

---

## Edge cases

### `src` swap without `keep-parent`

Without `keep-parent`, `<svg-insert>` is removed from the DOM after the first injection. Changing `src` on the detached element causes `this.parentNode.replaceChild(...)` to throw.

```html
<!-- ❌ element is detached after first inject — src swap throws -->
<svg-insert id="icon" src="/icons/moon.svg"></svg-insert>

<!-- ✅ keep-parent keeps the element in the DOM -->
<svg-insert id="icon" src="/icons/moon.svg" keep-parent></svg-insert>
```

### `src` not set

`_fetchAndInject` returns immediately. No error, no placeholder — the element stays empty until `src` is set.

### SVG file returns 404 or non-SVG response

Non-ok responses and responses with no `<svg>` element are caught and logged as `[svg-insert] …` console errors. The page continues to function; the icon is simply absent.

### Cross-origin SVGs

Requires the server to send `Access-Control-Allow-Origin`. Without it, `fetch` throws a network error.

```html
<!-- ✅ use a CDN that serves CORS headers -->
<svg-insert src="https://cdn.jsdelivr.net/npm/your-icon-set/logo.svg"></svg-insert>
```

### Multiple elements sharing the same `src`

Each element fetches independently. The browser cache covers subsequent requests, but consider `<use>` with a hidden `<symbol>` for many copies of the same icon.

---

## Anti-patterns

### Swapping `src` on a replace-mode element

```html
<!-- ❌ element is gone after first inject -->
<svg-insert id="icon" src="/icons/a.svg"></svg-insert>

<!-- ✅ -->
<svg-insert id="icon" src="/icons/a.svg" keep-parent></svg-insert>
```

### Using `<img src="*.svg">` when CSS theming is needed

```html
<!-- ❌ CSS variables and currentColor cannot reach inside an <img> SVG -->
<img src="/icons/logo.svg" style="color: var(--brand);">

<!-- ✅ -->
<svg-insert src="/icons/logo.svg" svg-class="brand-icon"></svg-insert>
```

### Forcing dimensions with `svg-width`/`svg-height` against a `viewBox`

```html
<!-- ❌ may distort or clip if aspect ratios don't match -->
<svg-insert src="/icons/logo.svg" svg-width="100" svg-height="32"></svg-insert>

<!-- ✅ size via CSS to respect the natural aspect ratio -->
<svg-insert src="/icons/logo.svg" svg-class="icon" keep-parent></svg-insert>
```

```css
.icon { width: 100px; height: auto; }
```

### No accessible label on a meaningful icon

```html
<!-- ❌ -->
<svg-insert src="/icons/delete.svg"></svg-insert>

<!-- ✅ meaningful icon -->
<svg-insert src="/icons/delete.svg" svg-aria-label="Delete item" svg-role="img"></svg-insert>

<!-- ✅ decorative icon -->
<svg-insert src="/icons/divider.svg" svg-aria-hidden="true"></svg-insert>
```

### Pointing `src` at user-supplied input without validation

```js
// ❌
const icon = new URLSearchParams(location.search).get('icon')
el.setAttribute('src', icon) // attacker-controlled path

// ✅
const ALLOWED = ['/icons/moon.svg', '/icons/sun.svg']
const safe = ALLOWED.includes(icon) ? icon : '/icons/default.svg'
el.setAttribute('src', safe)
```

---

## Security

`svg-insert` injects raw SVG markup into the live DOM. SVG files can contain `<script>` tags and event handlers. Only inline SVGs you own. See the [Security](#security) section of the full docs for mitigations (CSP, server-side sanitization, allowlisting).

---

## License

MIT License. Copyright (c) 2026 ynck-chrl.

See [LICENSE](../LICENSE) for the full text.
