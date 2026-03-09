# Vite + React + SCSS Demo

This project is a small frontend application built with Vite, React, and SCSS. It demonstrates how authored SCSS styles are transformed into bundled CSS during the build process, with CSS source maps enabled for debugging.

## Setup

### Prerequisites

Node.js 18+

### Install dependencies

```bash
npm install
```

## Running the Project

### Start the development server:

```bash
npm run dev
```

The application will be available at:

http://localhost:5173

### Build the production version:

```bash
npm run build
```

### Preview the production build:

```bash
npm run preview
```

## How CSS Is Transformed

The project uses SCSS as the authoring format for styles.

During development or build:

- SCSS files (.scss) are imported into the React application.
- Vite detects the imports and sends them to the Sass compiler.
- Sass converts SCSS into standard CSS:
  - SCSS variables are compiled into static values
  - Nested selectors are flattened
  - Mixins and functions are expanded
- Vite bundles all generated CSS into a single CSS file for production.

Because of this transformation pipeline, the generated CSS does not correspond 1-to-1 with the original SCSS files.

### Example Transformation

**Source SCSS** (`src/styles/variables.scss` + `src/index.scss`):
```scss
// Variables
$primary-color: #646cff;

// Nested selector with SCSS features
a {
  color: $primary-color;
  
  &:hover {
    color: lighten($primary-color, 10%);
  }
}
```

**Generated CSS** (in browser/build output):
```css
a {
  color: #646cff;
}

a:hover {
  color: #8a8dff;
}
```


## Generated CSS and Source Maps

After running:

```bash
npm run build
```

The output is created in the `dist/` directory:


dist/
  index.html
  assets/
    index-[hash].css
    index-[hash].js
    index-[hash].js.map


- `index-[hash].css` contains the compiled and bundled CSS used by the browser.  
- You might notice there is no separate `.css.map` file. In development mode, CSS source maps are embedded inline in the CSS, so the browser DevTools can still map styles back to your original `.scss` files.  
- Production builds do not generate separate `.css.map` files by default, but DevTools still uses the inline or hidden source maps to show original SCSS filenames and line numbers, check by inspecting styles in browser.  
- `.js.map` files exist for JavaScript because JS source maps are emitted separately, unlike CSS.


When inspecting styles in browser DevTools, these source maps allow developers to see and navigate to the original SCSS source code instead of the generated CSS.



