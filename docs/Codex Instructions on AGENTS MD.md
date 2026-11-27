# Codex Instructions on AGENTS.MD

# Frontend Engineering Specification (`AGENTS.md`)

## 1. Tech Stack & Architecture

- **Stack:** React 19 + TypeScript (strict mode) + Vite + Tailwind CSS.
- **Rendering:** Pure DOM / CSS / SVG + lightweight Spine/Rive canvas player for mascot animations. Avoid heavy WebGL game engines.
- **Data & State:** Read all schemas strictly from `src/types/contracts.ts`. Route all persistence (`getLesson`, `saveSessionResult`, `getDashboard`) exclusively through async methods in `src/services/storageAdapter.ts` (`localStorage` in prototype -> REST API in production). Never hardcode syllables, letters, or asset URLs inside UI components.

## 2. Dual Layout Modes (`AppViewMode`)

Every game screen and map view must dynamically adapt to `viewMode` from `AppSessionContext`:

- **`HOME_PERSONAL` (Phones, Tablets, Desktop):**
    - Responsive viewport (`100dvh`, `overflow: hidden` during gameplay).
    - Minimum touch target size: **`64x64px`**, minimum gap between interactive targets: **`16px`**.
    - Enables personalized family names, reward chest selection, and Parental Gate (`birthYear` check) before parent dashboard or subscription views.
- **`SMARTBOARD_16_9` (65"+ Kindergarten Panels & Projectors — SanPiN):**
    - **55% Bottom Zone Rule:** Render all child-interactive buttons, cards, and drop zones strictly within the **bottom 55% of the viewport** (reachable for children 110–120 cm tall). Reserve the **top 45%** exclusively for the mascot, visual scene, and educator controls.
    - Minimum touch target size: **`96x96px`**, minimum gap: **`24px`**.
    - **Zero Commercial UI:** When `access.adsAndPaywallHidden === true`, hide all paywalls, lock prices, trial banners, and external links.
    - **SanPiN 5-Minute Cap:** Enforce a `300s` lesson timer inside `LessonPlayer` that triggers a 30-second animated eye-gymnastics screen (`requireEyeGymnastics: true`).

## 3. Child Touch UX & Dual Input

- **Anti-Gesture Shield:** Apply `touch-action: none`, `user-select: none`, `webkit-user-select: none`, and `webkit-touch-callout: none` to all game containers to block pull-to-refresh, accidental pinch-zoom, and iOS long-press menus.
- **Pointer Events + Click Fallback:** Build interactions on unified Pointer Events (`onPointerDown`, `onPointerMove`, `onPointerUp`). Every drag-and-drop mechanic (including `SYLLABLE_MERGE`) must also support **Tap-to-Select -> Tap-to-Place** (or single-click trigger) so educators can operate lessons with a laptop mouse on non-touch projectors.
- **Zero-Reading & Anti-Frustration:**
    - Zero text instructions for children; rely on voice prompts + visual cues.
    - Show an animated guiding hand cursor after `idleHintDelayMs` (`4500–5000ms`) of inactivity.
    - Never render "Game Over" screens or harsh red error crosses. On wrong answers, gently dim/disable the incorrect option (`opacity-40 pointer-events-none`) and replay the target phoneme audio.

## 4. iOS Safari Audio Unlock & Phonetics

- **Singleton `audioManager.ts`:** Use Web Audio API (`AudioContext`) with HTML5 Audio fallback, never raw `<audio autoPlay>` tags inside components.
- **First-Gesture Unlock:** Mobile Safari blocks audio until an explicit user tap. Require an initial big play button ("В полёт!") whose `onPointerDown` synchronously calls `audioContext.resume()` and plays a silent 1-frame buffer before starting lesson instructions.
- **Preload & Lock:** Preload all audio buffers for the active `LessonUnitConfig` before hiding the transition loader. Ignore or queue rapid child taps while critical phonetic instruction audio is playing.
- **Russian Phonetics Rule:** Never use browser `speechSynthesis` (TTS) for Russian phonemes or syllables—only play verified pre-generated `.mp3` / `.ogg` files (pure sounds like `[М]`, never letter names like `[ЭМ]`).

## 5. Color & Typography Standards

- **Typography:** Rounded geometric sans-serif with standard Cyrillic letterforms (`Nunito` or `Rubik` — clear distinction between `а`, `б`, `д`, `и`, `й`). High contrast, soft warm backgrounds (avoid pure `#FFFFFF` glare on large smartboards).
- **Russian FOP DO Color Coding (strictly reserved for sound analysis mechanics):**
    - Vowel sounds: Red (`#EF4444`)
    - Hard consonants: Blue (`#3B82F6`)
    - Soft consonants: Green (`#22C55E`)