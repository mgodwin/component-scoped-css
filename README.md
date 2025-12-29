# Component-Scoped CSS

A modern, vanilla‑CSS pattern for building isolated, composable UI components using [`@scope`](https://developer.mozilla.org/en-US/docs/Web/CSS/@scope).

@scope prevents style leakage between components.  A couple of small naming patterns help with readability and maintainability.  It's like a modern take on BEM.

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

@scope is now a baseline browser feature in all modern browsers as of late 2025!

## How it works

### 1. Define components with @scope, name them the thing they are, no special prefixes.

Use the `:scope` pseudo class to apply styles to your root component element:

```css
@scope (.card) {
  :scope { border: 1px solid #eee }
}
```

Some example component names:

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

BEM expressed this relationship with `component__part` naming, but this becomes cumbersome when you refactor things.  It also feels unnecessary now that we have native CSS Selector nesting, but your class names can collide without some sort of way to scope your styles to the current component.  For example: `.card ._title` can match a deeply nested `._title` inside a child component.

@scope enables us to prevent those nested class name collisions, so let's just use an `_` to indicate the relationship:

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

---

### 3. Slots control where styles stop

Without some sort of lower bound on our style cascade, class names declared in one component will collide with those in nested components:

To address this, we use  `@scope`'s [**donut scopes**](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@scope#description) feature via `to(...)`, allowing you to declare *holes* or *slots* in a component where its styles should not apply.  This is the **lower bound** of our scope.

This allows us to specify where our styles should stop in the DOM, so we can embed other components without inheriting the styles from the parent component:

```css
@scope (.card) to (._actions) {
  ._title { font-weight: 600; }
  ._text { color: orange; }
  ._body  { color: var(--text-muted); }
}

@scope (.button) {
  ._text { font-weight: bold; }
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

**Note:** Scoping does offer a lot of isolation, especially when reusing class names, e.g. `.card ._title` styles are never merged with `.button ._title` (assuming proper upper and lower bounds on your scopes).  There is a weird gotcha with inherited properties though: they are able to "escape" from the scope.  For example, `color: red` can and will inherit down across scope boundaries.  This is important to be aware of, and can potentially be good or bad, depending on the context.

## How about variants?

You can use whatever naming scheme that makes sense to you to indicate variants of your components, but BEM's modifier syntax works well as second class, e.g. `--variant` :

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
