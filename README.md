
# Vue Matching Cards (Unsplash Edition)

A Vue 3 + Vite memory game that fetches card images from the Unsplash API.

## Features
- Vue 3 Composition API with `<script setup>`
- Responsive grid, flip animation
- Move counter, match counter, timer
- Category and pair count selector
- Reset/shuffle and refetch images
- Uses Unsplash API (falls back to picsum placeholders if no API key)

## Setup
1. Install Node 18+ and npm.
2. Clone or unzip the project.
3. Install deps:
   ```bash
   npm install
   ```
4. Configure Unsplash:
   - Copy `.env.example` to `.env`
   - Set `VITE_UNSPLASH_ACCESS_KEY` to your Unsplash access key

## Run
```bash
npm run dev
```

Open the URL printed by Vite (default http://localhost:5173).

## Build for Production
```bash
npm run build
npm run preview
```

## Notes
- The app uses the `Authorization: Client-ID <key>` header to call Unsplash from the browser.
- If you hit rate limits or do not set a key, the app will automatically use placeholder images from picsum.photos.
