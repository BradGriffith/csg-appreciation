# Columbus School for Girls - Teacher Appreciation

A visual progress tracker for faculty and staff appreciation voucher purchases. Features an interactive pictogram of 100 person icons that fill from bottom to top as vouchers are purchased toward the goal.

![Faculty & Staff Appreciation](https://img.shields.io/badge/Goal-220%20Vouchers-c8102e)
![Built with Vue.js](https://img.shields.io/badge/Vue.js-3.4-4FC08D?logo=vue.js)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-4.0-38B2AC?logo=tailwind-css)

## Features

- **Live Data Integration**: Fetches real-time voucher sales data from Shopify via API
- **Smart Caching**: 1-hour localStorage cache to minimize API calls and improve performance
- **Visual Progress Tracking**: 100-icon pictogram grid representing 220 voucher goal (2.2 vouchers per icon)
- **Smooth Fill Animation**: Icons fill from bottom to top with partial fill support using SVG gradients
- **Real-time Statistics**: Displays percentage of faculty & staff supported
- **Manual Refresh**: Footer with refresh button and last updated timestamp
- **Interactive Testing Controls**: Increment/decrement buttons and quick-select presets for testing
- **Responsive Design**: Optimized for desktop and mobile devices
- **Columbus School for Girls Branding**: Custom colors, fonts, and styling

## Technology Stack

- **Framework**: Vue.js 3 (Composition API)
- **Build Tool**: Vite 5
- **Styling**: Tailwind CSS 4
- **Fonts**: Lora (serif) and Nunito Sans (sans-serif) via Google Fonts
- **Deployment**: Netlify

## Getting Started

### Prerequisites

- Node.js (v18 or higher recommended)
- npm

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd csg-teacher-appreciation
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file from the example:
   ```bash
   cp .env.example .env
   ```

   The `.env` file contains the API endpoint URL for fetching live voucher data.

4. Start the development server:
   ```bash
   npm run dev
   ```

5. Open your browser to `http://localhost:5173` (or the port shown in terminal)

## Development

### Available Scripts

- `npm run dev` - Start Vite development server with hot module replacement
- `npm run build` - Build optimized production bundle to `dist/` directory
- `npm run preview` - Preview the production build locally

### Project Structure

```
src/
├── main.js                    # Application entry point
├── App.vue                    # Root component
├── style.css                  # Global styles and Tailwind imports
└── components/
    ├── TeacherPictogram.vue   # Main pictogram component with logic
    └── TeacherIcon.vue        # Reusable SVG person icon component
```

### Customizing Configuration

**API Endpoint** - Configure in `.env` file:
```bash
VITE_API_URL=http://pa.mycsg.org/.netlify/functions/sales-report?start_date=2025-08-01&end_date=2026-12-31&sources=shopify&product=voucher
```

**Cache Duration** - Edit in `src/components/TeacherPictogram.vue`:
```javascript
const CACHE_DURATION = 60 * 60 * 1000  // 1 hour in milliseconds
```

**Voucher Goals** - Edit constants in `src/components/TeacherPictogram.vue`:
```javascript
const totalIcons = 100        // Number of icons in grid
const goalVouchers = 220      // Total voucher goal
```

**Brand Colors** - Update colors in component styles:
- Primary red: `#c8102e`
- Accent yellow: `#ffc600`
- Teal: `#40c1bb`
- Green: `#73c92d`
- Purple: `#9d7ad2`

## How It Works

**Data Fetching & Caching:**
1. On page load, checks localStorage for cached data (valid for 1 hour)
2. If cache is valid, displays cached voucher count immediately
3. If cache is expired/missing, fetches fresh data from the API
4. Stores fetched data in localStorage with timestamp
5. Manual refresh button forces fresh API call, bypassing cache
6. Error handling falls back to cached data if API is unavailable

**Pictogram Fill Algorithm:**
1. Divides the voucher goal (220) by number of icons (100) = 2.2 vouchers per icon
2. Fills icons from bottom to top as vouchers increase
3. Supports partial fills using SVG linear gradients
4. Each icon generates unique gradient IDs to prevent conflicts when rendering 100 instances

## Deployment

The application is configured for Netlify deployment:

1. Connect your repository to Netlify
2. Build settings are pre-configured in `netlify.toml`:
   - Build command: `npm run build`
   - Publish directory: `dist/`
3. Set environment variable in Netlify dashboard:
   - Go to Site settings → Environment variables
   - Add `VITE_API_URL` with your API endpoint URL
4. Deploy automatically on push to main branch

## Browser Support

- Modern browsers with ES6+ support
- Chrome, Firefox, Safari, Edge (latest versions)
- Mobile browsers (iOS Safari, Chrome Mobile)

## License

© Columbus School for Girls

---

**Tagline**: *She will know her power.*
