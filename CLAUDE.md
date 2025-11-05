# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Teacher/Faculty Appreciation pictogram visualization app for Columbus School for Girls. Built as a single-page Vue.js 3 application with Vite and Tailwind CSS 4. The app displays a 10x10 grid of 100 person icons that fill from bottom to top as vouchers are purchased, with support for partial fills. Fetches live voucher sales data from Shopify API with 1-hour localStorage caching.

## Development Commands

- `npm run dev` - Start Vite development server
- `npm run build` - Build for production (outputs to `dist/`)
- `npm run preview` - Preview production build locally

## Architecture

### Component Structure

- **App.vue** - Root component that renders TeacherPictogram
- **TeacherPictogram.vue** - Main component containing:
  - API integration with localStorage caching (1-hour duration)
  - Statistics display showing percentage of faculty/staff supported
  - Grid of 100 TeacherIcon components
  - Footer with last refreshed timestamp and manual refresh button
  - Testing controls (counter, quick-select buttons)
  - All business logic for calculating icon fill percentages
- **TeacherIcon.vue** - Reusable SVG person icon with gradient fill support

### Key Technical Details

**API Integration & Caching:**
- API endpoint configured via `VITE_API_URL` environment variable in `.env`
- Fetches voucher sales data from Shopify via Netlify function
- 1-hour localStorage cache (`CACHE_DURATION = 60 * 60 * 1000`)
- Cache key: `teacher_appreciation_data`
- On load: checks cache validity before making API call
- Manual refresh button bypasses cache and forces fresh API fetch
- Error handling falls back to cached data if API unavailable
- Data extraction: `item_count` from `summary.by_product[0]` in API response

**Pictogram Fill Logic:**
- 100 icons total representing 220 vouchers (2.2 vouchers per icon)
- Icons fill from bottom to top as vouchers increase
- Partial fill supported via SVG linear gradients (bottom to top)
- Each icon generates unique gradient and clip path IDs to prevent conflicts

**Styling:**
- Tailwind CSS 4 integrated via `@tailwindcss/vite` plugin
- Custom fonts via Google Fonts:
  - Lora (serif) for titles
  - Nunito Sans (sans-serif) for body text
- Columbus School for Girls brand colors:
  - Primary red: `#c8102e`
  - Accent yellow: `#ffc600`
  - Gradient background: red to purple (`#9d7ad2`)
  - Additional accent colors: teal (`#40c1bb`), green (`#73c92d`)
- Responsive design with mobile breakpoint at 768px

**State Management:**
- Uses Vue 3 Composition API with `<script setup>`
- Reactive state via `ref()` for vouchers purchased
- Computed properties for percentage calculations and icon data generation

### Deployment

Configured for Netlify deployment:
- Build command: `npm run build`
- Publish directory: `dist/`
- Configuration in `netlify.toml` (root level)
- Environment variables must be set in Netlify dashboard:
  - `VITE_API_URL` - API endpoint for fetching voucher data

### File Organization

```
src/
├── main.js              # Vue app initialization
├── App.vue              # Root component
├── style.css            # Global styles (Tailwind import)
└── components/
    ├── TeacherPictogram.vue  # Main pictogram component
    └── TeacherIcon.vue       # Reusable person SVG icon
```

## Important Constants

In [TeacherPictogram.vue](src/components/TeacherPictogram.vue):
- `totalIcons = 100` - Number of icons in the grid
- `goalVouchers = 220` - Total voucher goal
- `vouchersPerIcon = 2.2` - Calculated ratio
- `iconSize = 40` - Default icon size in pixels
