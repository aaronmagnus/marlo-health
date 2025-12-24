# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Shopify theme for Marlo Health, based on the "Wonder" theme (v2.1.2) by Nethype. It's a pure Shopify theme with no Node.js build process - assets are served directly via Shopify's CDN.

## Development Commands

```bash
# Start local development server (requires Shopify CLI)
shopify theme dev

# Push theme to store
shopify theme push

# Pull theme from store
shopify theme pull

# Check theme for errors
shopify theme check
```

## Architecture

### JavaScript: Web Components + Pub/Sub

The theme uses native Web Components (`customElements.define`) for all interactive elements. Components communicate via a pub/sub event system.

**Core files loaded on every page (in order):**
1. `constants.js` - Defines `PUB_SUB_EVENTS` (cartUpdate, cartDrawerOpen, variantChange, cartError, etc.)
2. `pubsub.js` - `subscribe(eventName, callback)` and `publish(eventName, data)` functions
3. `global.js` - Shopify utilities, debounce, helper functions
4. `base.js` - Base component definitions

**Key Web Components:**
- `<product-form>` - Product variant selection and cart addition
- `<cart-drawer>` - Slide-out shopping cart
- `<variant-options>` - Product variant picker
- `<color-swatch>` - Color/option swatches
- `<facet-filters-form>` - Collection filtering

**Event communication pattern:**
```javascript
// Subscribe to cart updates
subscribe(PUB_SUB_EVENTS.cartUpdate, (data) => { /* handle update */ });

// Publish cart update
publish(PUB_SUB_EVENTS.cartUpdate, { source: 'cart-drawer', cartData: response });
```

### CSS Structure

- `critical.css` - Above-the-fold styles, loaded blocking
- `main.css` - Main styles, lazy-loaded with preload hint
- `custom.css` - Empty file for store-specific overrides
- `settings.css.liquid` - Dynamic CSS variables from theme settings

CSS uses custom properties extensively (`--color-*`, `--font-*`, `--gap`, etc.) defined via theme settings.

### Liquid Templates

**Template hierarchy:**
- `layout/theme.liquid` - Main wrapper, loads global scripts/styles
- `templates/*.json` - Page templates defining section order
- `sections/*.liquid` - Self-contained page sections with schema
- `snippets/*.liquid` - Reusable partials (use `{% render 'name' %}`)

**Section groups:** Header and footer managed via `sections/header-group.json` and `sections/footer-group.json`.

### Third-Party Libraries

- **Swiper** - Carousels and sliders (`swiper-bundle.min.css`, `swiper-bundle.mjs`)
- **PhotoSwipe** - Image lightbox gallery
- **noUiSlider** - Price range slider in filters
- **img-comparison-slider** - Before/after image comparison

## Theme Settings

Global customization via `config/settings_schema.json`:
- Layout width (1400-1900px)
- Border radius values
- Color palette (background, text, buttons, badges)
- Typography (separate fonts for body, headings, navigation, buttons, prices, breadcrumbs)
- Custom font imports via `fontfaces` setting

Settings applied as CSS variables in `settings.css.liquid`.

## Key Conventions

- Scripts use `defer` attribute for non-blocking loading
- Product pages conditionally load `gallery.js` as ES module
- Mobile breakpoint: 1200px
- RTL language support via `dir` attribute detection
- Translations in `locales/*.json` using `t:` filter keys
