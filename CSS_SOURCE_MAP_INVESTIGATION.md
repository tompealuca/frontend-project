# CSS Source Map Investigation Report

## Element Selected for Analysis

**Element**: The main button with text "count is X"  
**Selector**: `button` (the counter button in the card)  
**Why chosen**: This element has complex styling from multiple sources:
- Base button styles from `index.scss`
- Theme-aware styles (light/dark mode)
- Hover states and transitions
- Multiple CSS properties with cascading rules


> **Note:** In this project, Vite bundles CSS into JavaScript and generates a JS source map (`index-[hash].js.map`). No separate `.css.map` file exists, but DevTools can still map styles back to original SCSS lines.


## CSS Properties Analysis

I analyzed five CSS properties of the button element, tracing each from the computed value back to the original SCSS source.


### Property 1: `background-color`

**Computed Value**: `rgb(26, 26, 26)` (dark mode) / `rgb(249, 249, 249)` (light mode)

**DevTools Styles Panel**:
```
button {
  background-color: #1a1a1a;
}
@media (prefers-color-scheme: light) {
  button {
    background-color: #f9f9f9;
  }
}
```

**Generated CSS Location**: `index-[hash].css:1:1234` (bundled in JS, mapped via `index-[hash].js.map`)

**Source Map Trace**:
- **Direct mapping**: Points to `src/index.scss:65`
- **Original source**: `$bg-dark-secondary: #1a1a1a;` in `src/styles/variables.scss`
- **Transformation visible**: SCSS variable `$bg-dark-secondary` compiled to hex value `#1a1a1a`

### Property 2: `border-radius`

**Computed Value**: `8px`

**DevTools Styles Panel**:
```
button {
  border-radius: 8px;
}
```

**Generated CSS Location**: `index-[hash].css:1:890`

**Source Map Trace**:
- **Direct mapping**: Points to `src/index.scss:58`
- **Original source**: `border-radius: 8px;` (literal value, no variables)
- **No transformation**: Value passes through unchanged

### Property 3: `transition`

**Computed Value**: `border-color 0.25s ease 0s`

**DevTools Styles Panel**:
```
button {
  transition: border-color .25s;
}
```

**Generated CSS Location**: `index-[hash].css:1:1056`

**Source Map Trace**:
- **Direct mapping**: Points to `src/index.scss:62`
- **Original source**: `transition: border-color $transition-fast;` in `src/index.scss`
- **Variable expansion**: `$transition-fast: 0.25s;` from `src/styles/variables.scss`
- **Unit conversion**: SCSS processes the variable correctly

### Property 4: `color` (inherited from root)

**Computed Value**: `rgb(255, 255, 255)` (dark mode) / `rgb(33, 53, 71)` (light mode)

**DevTools Styles Panel**:
```
:root {
  color: rgba(255, 255, 255, 0.87);
}
@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
  }
}
```

**Generated CSS Location**: `index-[hash].css:1:234`

**Source Map Trace**:
- **Direct mapping**: Points to `src/index.scss:6`
- **Original source**: `color: vars.$text-color;` in `src/index.scss`
- **Variable resolution**: `$text-color: rgba(255, 255, 255, 0.87);` from `src/styles/variables.scss`
- **Inheritance**: Button inherits color from `:root` element

### Property 5: `font-family`

**Computed Value**: `system-ui, Avenir, Helvetica, Arial, sans-serif`

**DevTools Styles Panel**:
```
button {
  font-family: inherit;
}
Inherited from body
body {
  font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;
}
```

**Generated CSS Location**: `index-[hash].css:1:756` (button), `index-[hash].css:1:145` (body)

**Source Map Trace**:
- **Direct mapping**: Button points to `src/index.scss:61`
- **Inheritance chain**: `font-family: inherit;` → inherits from `body`
- **Body styles**: Points to `src/index.scss:4`
- **Original source**: `font-family: system-ui, Avenir, Helvetica, Arial, sans-serif;`

## Cases Where Source Map Mapping Breaks Down

### Case 1: CSS Cascade and Specificity Conflicts

**Issue**: When multiple rules target the same element, DevTools shows only the winning rule, but source maps don't reveal the cascade hierarchy.

**Example**: The button's `background-color` has both a base rule and a media query override. The source map points to the winning rule, but doesn't show:
- Which rules were considered but overruled
- Specificity calculations
- Cascade order resolution

**Impact**: Developer must manually check all related selectors to understand why a particular value "wins" the cascade.

### Case 2: PostCSS and Browser-Specific Transformations

**Issue**: Modern CSS features get transformed by PostCSS/browser engines, breaking the direct source map link.

**Example**: CSS custom properties, `calc()` expressions, or modern color functions may be transformed:
```scss
// Source: SCSS variable with alpha
$primary-hover: #535bf2;

// Generated: Browser processes rgba()
filter: drop-shadow(0 0 2em rgba(83, 91, 242, 0.67));
```

**Impact**: The computed value in DevTools doesn't match the authored SCSS value, requiring manual translation of the transformation.

### Case 3: Inherited and Computed Values

**Issue**: Many CSS properties are inherited or computed, making it unclear which element actually defines the value.

**Example**: The button's `color` property shows as inherited from `:root`, but the source map points to the button's rule (`font-family: inherit;`) rather than the actual source of the inherited value.

**Impact**: For inherited properties, developers must manually trace up the DOM tree to find the originating rule, as source maps only point to the inheritance declaration, not the inherited value's source.