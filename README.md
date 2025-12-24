# Component-Scoped CSS

A modern, vanilla‑CSS pattern for building isolated, composable UI components using [`@scope`](https://developer.mozilla.org/en-US/docs/Web/CSS/@scope).

```css
@scope (.card) to (._actions) {
  :scope {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 8px;
  }

  ._title { font-weight: 600; }
  ._body  { color: #555; }
}

@scope (.button) {
  :scope { padding: 0.5rem 1rem; background: black; color: white; }
  ._text { font-weight: 500; }
}
```

```html
<div class="card">
  <h3 class="_title">Pro Plan</h3>
  <p class="_body">Everything you need.</p>

  <div class="_actions">
    <button class="button">Buy</button>
  </div>
</div>
```
## Rules

### 1. Components are defined with @scope

@scope is [now widely supported across all browsers](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@scope#browser_compatibility). It allows you to define an upper bound and lower bound where your styles apply within the DOM.  This allows us to write styles that are scoped only to our component!

When you define your component with @scope, we're declaring the **upper bound** of our scope where our styles apply to:

```css
@scope (.card) {
  /* apply styles to the .card class within the :scope pseudo selector: */
  :scope { border: 1px solid #eee }
}
```

Components should be named without any special prefixes:

* `card`
* `dropdown`
* `data-table`

Anytime you come across a class in your markup without a prefix, you know it's the root component definition.

---

### 2. Component parts use _ prefixes

Without some sort of convention, it's hard to know how one class relates to one another when scanning the DOM:

```html
<div class="card">
  <h3 class="title">Title</h3> <!-- Is title a component, or the title of the card? -->
  <p class="body">Body</p> <!-- Is body a component, or the body of the card? -->
</div>
```

Use a `_` prefix to denote that this class is a part of the component higher up in the DOM hierarchy:

```html
<div class="card">
  <h3 class="_title">Title</h3>
  <p class="_body">Body</p>
</div>
```

This is easy to represent within our @scope definition:

```css
@scope (.card) {
  ._title { font-weight: 600; }
  ._body  { color: var(--text-muted); }
}
```

Don't worry about name collisions with other component parts, that's what @scope is here to help with!

---

### 3. Slots control where styles stop

`@scope` supports [**donut scopes**](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@scope#description) via `to(...)`, allowing you to declare *holes* or *slots* in a component where its styles should not apply.  This is the **lower bound** of our scope.

This allows us to safely compose components without style interference:

```css
@scope (.card) to (._actions) {
  ._title { font-weight: 600; }
  ._body  { color: var(--text-muted); }
}
```

```html
<div class="card">
  <div class="_body">
    <span class="_text">Description</span>
  </div>

  <div class="_actions">
    <!-- nested components live here -->
    <button class="button">
      <span class="_text">Buy</span>
    </button>
  </div>
</div>
```

Here, `.card` styles apply to everything **except** what appears inside `._actions`. That prevents the card’s typography and spacing rules from bleeding into nested components like `.button`.

You can write as many classes as you want to indicate as lower bounds for your scope:

```css
@scope(.card) to (._actions, .__slot__, .header) {...}
```

**Note:** Scoping does not guarantee that inherited properties don't "escape" from the scope.  It does guarantee that you don't have to worry about styles from a parent scope bleeding into your child scope, e.g. `card ._title` is independent from `figure ._title`.  Inherited properties for things like color, font-weight, font-size, etc. need to be considered and accounted for.

## How about variants?

You can use whatever naming scheme that makes sense to you to indicate variants of your components, but BEM has a great recommendation with their modifier syntax, `--variant` :

```css
@scope(.card) {
  :scope { ... }
  &.--red {
    border: 1px solid red;
  }
}
```

For example, `card --red` would correctly apply the styles you're looking for.

Or you could do something like `button --primary` for example.

## Why components?

Structuring your CSS around component boundaries feels fairly natural, components are the part of the web equation that React got right.  You're already calling it a `Card` in your server side code, so why do you have to call it something else in your CSS?  Because scope affords us so much isolation, let's call things what they are as much as possible.

## Why not BEM?

Most CSS pain comes from global naming: figuring out who owns an element, preventing collisions, and being afraid to refactor.

BEM, OOCSS, and CSS‑in‑JS all try to solve this with conventions, tooling, or indirection — but the underlying problem is that CSS never had real component boundaries.

`@scope` finally gives us those boundaries. Component‑Scoped CSS uses it to make ownership, nesting, and composition visible directly in the markup, without prefixes, hashes, or build steps.
