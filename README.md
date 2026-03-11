# Performance Analysis & Vue Architecture Proposal - Travel Booking Platform (Case Study)

## Frontend Performance Analysis & Vue Architecture Proposal

This repository contains a technical case study analyzing the performance of a large travel booking website.

The goal of the analysis was to:

- Identify performance bottlenecks using Lighthouse and Chrome DevTools
- Propose concrete performance improvements
- Design a scalable Vue 3 component architecture for the landing page
- Break down the implementation into actionable engineering tasks

The case study focuses on real-world frontend performance challenges such as large JavaScript bundles, render-blocking CSS, heavy API payloads, and image optimization.

## Table of Contents

1. Performance Analysis
2. Suggested Improvements
3. Vue Architecture Proposal
4. Work Breakdown
5. Conclusion

## **1. Performance Analysis: filiptravel.rs**

### 1.1 Website Overview

The selected website for analysis is a travel booking platform in Serbia. The site features interactive elements such as dynamic image carousels, API-driven content, chat widgets, and third-party scripts.

The website uses **jQuery-based frontend architecture** with dynamic content loaded via API requests and JavaScript bundles.

This makes it a suitable candidate for performance analysis because multiple interactive components can impact rendering performance.

### 1.2 **Analysis Context**

The website is protected by **`Cloudflare` bot verification**, which prevents Lighthouse from correctly loading the **mobile version** of the site.

Because of this limitation, the performance analysis is based on:

- **Desktop Lighthouse results**
- **Chrome DevTools Network analysis**

These tools were used to identify major performance bottlenecks such as **large CSS files, heavy JavaScript bundles, large API payloads, and unoptimized images**.

### **1.3 Lighthouse Performance Metrics**

| Metric                         | Value  | Status               | Recommended / Reference |
| ------------------------------ | ------ | -------------------- | ----------------------- |
| Performance Score              | 20     | 🔴 Poor              | > 90                    |
| First Contentful Paint (FCP)   | 3.5 s  | 🟡 Needs Improvement | < 1.8 s                 |
| Largest Contentful Paint (LCP) | 6.2 s  | 🔴 Poor              | < 2.5 s                 |
| Total Blocking Time (TBT)      | 510 ms | 🟡 Needs Improvement | < 200 ms                |
| Cumulative Layout Shift (CLS)  | 0.31   | 🔴 Poor              | < 0.1                   |
| Speed Index                    | 3.8 s  | 🟡 Needs Improvement | < 3 s                   |

### **1.4 Explanation of Each Metric**

**Overall Performance Score (20): An** aggregate score from Lighthouse. A low score indicates **major performance bottlenecks**, meaning the website takes long time to render and respond to users.

**First Contentful Paint (FCP) – 3.5s: The** Time until the **first text or image** appears on the screen. A high FCP makes the page appear slow to users, even if some content eventually loads quickly.

**Largest Contentful Paint (LCP) – 6.2s: Measures the t**me until the **largest visible element** (often hero image or banner) is rendered. A high LCP directly affects **perceived loading speed** — users feel the page is slow.

**Total Blocking Time (TBT) – 510 ms:** Time during which the **main thread is blocked by JavaScript execution**, preventing the user from interacting with the page. High TBT means the page may appear unresponsive.

**Cumulative Layout Shift (CLS) – 0.31:** Measures **visual stability**. High CLS indicates elements shift position while loading (images, banners, sliders), which negatively affects user experience.

**Speed Index – 3.8s:** How quickly the page **visually populates**. A high speed index means content appears slowly, even if the page technically loaded.

---

## **2. Suggested Improvements – filiptravel.rs**

### **2.1 First Contentful Paint (FCP) – 3.5 s**

- **Identified issues:**
  - Render-blocking CSS: `style.min.css` (358 KB) loaded in `<head>`
  - `index.js` is loaded after CSS → parses and executes after CSS is ready
  - Hero carousel dynamically loaded via Carousel API endpoint
- **Suggested improvements:**
  1. **Critical CSS** → extract only the CSS for header, navigation and hero placeholder → inline in `<head>`
  2. **Split CSS** → Split CSS into smaller files or load the remaining `style.min.css` asynchronously.
  3. **Lazy-load hero carousel** → show placeholder or low-res version first

### **2.2 Largest Contentful Paint (LCP) – 6.2 s**

- **Identified issues:**
  - Hero image (Hero carousel image, 800 KB) loaded via Carousel API endpoint (~4.4 MB payload)
  - Large payload increases parsing and execution time before the carousel images can be inserted into the DOM.
  - Images not optimized (high-res PNG/JPEG)
- **Suggested improvements:**
  1. **Optimize hero image** → resize + WebP/AVIF (~400–500 KB)
  2. **Optimize API** → return only image URLs and minimal metadata
  3. **Lazy-load / placeholder** → low-res version while full image loads
  4. **Split carousel JS** → load after critical content

### **2.3 Total Blocking Time (TBT) – 510 ms**

- **Identified issues:**
  - `index.js` → critical site JS
  - `script-home.min.js` → 966 KB bundle, executes after `index.js`
  - Large API payload (Carousel API endpoint, 4.4 MB)
  - Third-party JS: recaptcha (`recaptcha__en.js` 367 KB), Google Analytics (`gtag.js`), chat widget scripts
- **Suggested improvements:**
  1. **Split JS bundles** → feature-based chunks (carousel, chat, analytics)
  2. **Defer / async non-critical scripts** → `recaptcha__en.js`, `gtag.js,` `gtm.js`
  3. **Lazy-load third-party scripts** → chat scripts only when user clicks chat widget

### **2.4 Cumulative Layout Shift (CLS) – 0.31**

- **Identified issues:**
  - Hero carousel dynamically injected via JS/API
  - Chat widget added to DOM when user clicks chat
  - CSS causing layout/font shifts
- **Suggested improvements:**
  1. **Set width/height for images and containers** →  (Hero carousel image , `chat-logo.png`)
  2. **Lazy-load widget images** → chat logo loads only on user interaction
  3. **Critical CSS + placeholders** → for hero carousel and main elements

### **2.5 Speed Index – 3.8 s**

- **Identified issues:**
  - Hero image and carousel load late due to large API payload
  - Large JS and CSS bundles
  - Third-party scripts executing on main thread
  - **API loads extra content not immediately visible**
- **Suggested improvements:**
  1. **Optimize images** → WebP, resize, lazy-load
  2. **Split / defer JS** → reduce render blocking
  3. **Critical CSS first** → load remaining CSS asynchronously
  4. **Lazy-load third-party scripts**
  5. **Split API requests and fetch data on demand** → load only critical content first, secondary slides/images lazily

### **2.6 JavaScript Files Summary**

| **File** | **Size** | **Issue** | **Recommended Action** |
| -------- | -------- | --------- | ---------------------- |

| `index.js`
| Main site script
| Executes core site logic and contributes to Total Blocking Time
| Keep only critical initialization logic and move feature logic (carousel, chat) into separate scripts
|
| `script-home.min.js` | ~966 KB | Large bundle that dynamically injects hero carousel images and increases TBT | Split into feature-based bundles (carousel, chat). Load carousel logic only after initial page content renders |
| `jquery.min.js` | ~26 KB | Small dependency required by other scripts | No action needed, performance impact is minimal |
| `recaptcha__en.js` | ~367 KB | Third-party script that adds additional JS parsing and execution time | Load only on pages where CAPTCHA is required or trigger loading when the form becomes visible |
| `gtag.js`/`gtm.js` | Third-party | Additional scripts that increase JS execution and network requests | Load with async or defer so they do not block page rendering |

### **2.7 CSS Files Summary**

| **File**        | **Size** | **Issue**                                                          | **Recommended Action**                                                                                             |
| --------------- | -------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `style.min.css` | ~358 KB  | Main render-blocking CSS file loaded in the <head> before index.js | Extract critical CSS for above-the-fold content and inline it in the <head>. Load the remaining CSS asynchronously |
| `custom.css`    | ~0.5 KB  | Very small file with negligible performance impact                 | No significant optimization required                                                                               |

### **2.8 Images Summary**

| **Image / Source**                              | **Size**     | **Issue**                                                             | **Recommended Action**                                                                               |
| ----------------------------------------------- | ------------ | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Hero carousel images (e.g. Hero carousel image) | ~800 KB each | Largest visible elements on the page and main contributor to slow LCP | Compress images and convert to WebP or AVIF. Resize to display size and lazy-load non-visible slides |
| `chat-logo.png`                                 | ~1.4 MB      | Large image used by the chat widget                                   | Compress the image and ensure it loads only when the chat widget is opened                           |
| Other page images                               | <300 KB      | Minor performance impact                                              | Optimize with compression and modern formats when possible                                           |

### **2.9 API Requests Summary**

| **Endpoint**           | **Payload Size** | **Issue**                                                                         | **Recommended Action**                                                                       |
| ---------------------- | ---------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Carousel API endpoint  | ~4.4 MB          | Large API response used to populate the hero carousel                             | Reduce payload size by returning only necessary data such as image URLs and minimal metadata |
| Carousel image loading | Dynamic          | Images are injected into the DOM by `script-home.min.js`, delaying hero rendering | Load only the first carousel image initially and lazy-load the rest                          |

### **Additional Notes**

> _All recommendations above are **framework-agnostic** and can be implemented within the current architecture._
>
> _While the website is built using **jQuery**, most performance issues can be resolved through optimization of **JavaScript execution, CSS delivery, image assets, and API responses**._
>
> _A migration to a modern frontend framework could improve **maintainability and long-term scalability**, but it is **not required to address the current performance bottlenecks**._

---

## **3. Identify Components – Landing Page (filiptravel.rs)**

**Note on Assumptions:** This component structure assumes a **design system following Atomic Design principles:**

- Components in shared/components are **reusable across multiple features or pages** (atoms, molecules, organisms).
- Components inside features/home/components are **specific to the Landing Page feature** and not expected to be reused elsewhere.
- **Composables and API modules** follow a similar logic:
  - Shared composables (shared/composables) and shared API modules (shared/api) handle **logic or data fetching used across multiple features**.
  - Feature-specific composables (features/home/composables) and API modules (features/home/api) handle **logic or API calls only relevant to this feature**.
- For simplicity, **shared state and cross-page stores** are not included in this task, as the focus is only on the Landing Page
- Header and Footer could be further decomposed into smaller components (e.g., Navigation, FooterLinks) for clarity and reusability, but for this task, they are considered as whole organisms.

These assumptions allow us to clearly separate presentation, logic, and data fetching, resulting in a scalable and maintainable project structure.

### **📁 Folder Structure Overview**

```
src/
├── App.vue
│   ├── Header.vue           # Always visible, site-wide navigation
│   ├── PageLayout.vue       # Wraps page content for consistent layout
│   │   └── <router-view />  # Dynamic page content injected here
│   ├── Footer.vue           # Always visible, site-wide footer
│   └── Chatbot.vue          # Always visible, site-wide Chatbot
│
├── layouts/
│   └── PageLayout.vue       # Handles consistent padding, max-width, background
│
├── features/
│   └── home/                # Feature-specific logic & components
│       ├── pages/
│       │   └── LandingPage.vue
│       ├── components/
│       │       ├── SearchSection.vue
│       │       ├── FeaturedDestinationCard.vue
│       │       ├── FeaturedDestinations.vue
│       │       ├── WellnessCarousel.vue
│       │       ├── SeasonSection.vue
│       │       └── SeasonCard.vue
│       │
│       ├── composables/     # Feature-specific composables
│       │   ├── useSearch.ts
│       │   ├── useFeaturedDestinations.ts
│       │   ├── useWellness.ts
│       │   └── useSeasons.ts
│       │
│       └── api/             # Feature-specific API calls
│           ├── searchApi.ts
│           ├── featuredDestinationsApi.ts
│           ├── wellnessApi.ts
│           └── seasonsApi.ts
│
├── shared/
│   ├── components/
│   │   ├── atoms/
│   │   │   ├── Button.vue
│   │   │   ├── Input.vue
│   │   │   ├── Select.vue
│   │   │   └── Icon.vue
│   │   │
│   │   ├── molecules/
│   │   │   ├── DestinationCard.vue
│   │   │   ├── PromotionCard.vue
│   │   │   └── AdditionalServiceCard.vue
│   │   │
│   │   └── organisms/
│   │       ├── Destinations.vue
│   │       ├── Carousel.vue
│   │       └── ActivityIndicator.vue
│   │
│   ├── composables/
│   │   ├── useCarouselConfig.ts
│   │   ├── useDestinations.ts
│   │   └── usePromotions.ts
│   │
│   └── api/                 # Shared API modules for fetching data
│       ├── carouselApi.ts
│       ├── destinationsApi.ts
│       └── promotionsApi.ts
│
├── assets/
│   ├── images/
│   └── styles/
│
└── router/
    └── index.js             # Page routes (LandingPage, Destinations, etc.)
```

### **3.1 Page Layout & Global Shell**

- `PageLayout.vue` wraps the page content consistently across the app.
- `Header.vue` and `Footer.vue` are **site-wide**, so they live in `App.vue` for global visibility.
- `Chatbot.vue` is also site-wide and rendered as an overlay on all pages.

### **3.2 Landing Page Components**

The landing page is composed of several **feature-specific components** and **shared reusable components**. Feature components encapsulate logic specific to this page, while shared components can be reused across multiple pages.

### **Feature-Specific Components**

These components are tightly coupled to the landing page and contain logic or layouts specific to this feature.

- **`SearchSection.vue**` – Handles user input for searching destinations, including destination selection, travel dates, and number of guests.
  - Encapsulates form elements such as **Input**, **Select**, and **Button** atoms.
  - Contains the logic responsible for collecting and validating user input
- **`FeaturedDestinations.vue`** – Displays a curated list of highlighted destinations.
  - Uses `FeaturedDestinationCard.vue` as child component.
  - Responsible for rendering a list of featured destinations returned from the API.
- **`WellnessCarousel.vue`** – Shows wellness-related offers using a carousel layout.
  - Encapsulates carousel behavior such as **slide navigation and internal state**.
  - Dedicated to displaying wellness promotions relevant to the landing page.
- **`SeasonSection.vue`** – Displays seasonal offers or content.
  - Uses `SeasonCard.vue`.
  - Specific to this page and not reused elsewhere

### **Shared Components Used on the Page**

In addition to feature-specific components, the landing page relies on several **shared components** that are reusable across the application.

- `Carousel.vue` - A reusable component responsible for rendering image or content sliders.
  - Provides navigation controls and slide transitions.
- `Destinations.vue`- Displays a list of destinations using **`DestinationCard.vue`** components.
  - Includes navigation or filtering controls that allow users to switch between destination categories.
- `PromotionCard.vue`- A reusable card component used to display promotional content.
  - Typically contains a **large promotional image and title**.
  - Clicking the card routes the user to a detailed promotion page.

### **3.3 Shared Components and Atomic Design**

- **Atoms:** Smallest elements (`Button.vue`, `Input.vue`, `Select.vue`, `Icon.vue`).
- **Molecules:** Combinations of atoms forming UI units (`DestinationCard.vue`, `PromotionCard.vue`, `AdditionalServiceCard.vue`).
- **Organisms:** Larger, self-contained sections of UI (`Destinations.vue`, `Carousel.vue`, `ActivityIndicator.vue`).
- **Reasoning:** Shared components live in shared/components to **promote reusability** across multiple features or pages.

### **3.4 Composables and API**

- **Shared composables:** (`useDestinations.ts`, `useFeaturedDestinations.ts`) handle **fetching data** or logic reused across features.
- **Feature-specific composables:** (`useSearch.ts`) handle **logic only relevant to LandingPage**.
- **API modules:** Organized similarly: shared API calls in shared/api, feature-specific calls in features/home/api.

### **3.5 Data Flow & Prop Passing**

- **Composables** are called in the **`LandingPage.vue`** and their data is passed **via props** to child sections.
- This keeps **child sections presentation-focused** and avoids coupling presentation components directly to API logic.

---

## 4. Finalize the Solution: Work Breakdown

**Note:** Ticket IDs are illustrative and used only to demonstrate how the work could be organized in a task management system.

### ⚡Immediate Performance Fixes (jQuery Site)

_Goal: Quick wins to improve Lighthouse scores on the existing `filiptravel.rs` site._

### **🎫 [Optimization Task 1]: Critical CSS & Asset Optimization**

- **Scope:** Extract CSS required for the header and hero section and inline it in the `<head>`.
- **Performance Acceptance Criteria:** Reduce **FCP** by moving `style.min.css` to load asynchronously.

### **🎫 [Optimization Task 2]: Image Compression & Format Conversion**

- **Scope:** Improve **LCP** and **CLS** by optimizing assets and preventing layout shifts.
  - **Asset Optimization:** Batch-convert hero carousel images and the `chat-logo.png` to WebP/AVIF.
  - **Layout Stability:** Implement explicit `aspect-ratio` or fixed height/width in CSS for all hero image containers to reserve space before the API response triggers the image load.
- **Performance Acceptance Criteria:** Target a reduction in **LCP** by at least 2 seconds and eliminate content-shifting-induced **CLS**in the hero section.

### 🎫 [Optimization Task 3]: Script Bundle Splitting & Deferral

- **Scope:** Refactor `script-home.min.js` into feature-based chunks (e.g., `carousel.js`, `chat-widget.js`, `analytics.js`).
  - Implement `defer` for all non-critical scripts to prioritize main-thread availability.
- **Performance Acceptance Criteria:** Reduce **TBT** by at least 300ms by offloading non-critical JS execution from the critical rendering path.

### 🎫 [Optimization Task 4]: Carousel API endpoint Integration

- **Scope:**  Implement consumption of the updated Carousel API endpoint.
  - Refactor frontend logic to handle paginated/sharded API responses, ensuring only the first two images are fetched on initial load.
- **Performance Acceptance Criteria:** Achieve a reduction in **LCP** by minimizing the initial network payload from 4.4 MB to < 500 KB.
  - Improve **TBT** by reducing the amount of data the main thread needs to parse on startup.
- **Note:** This ticket requires backend support to modify the endpoint response structure. A separate backend ticket has been created to track the API optimization. Frontend development can proceed using the Mock API provided in the design specs.

### 🎫[Optimization Task 5]: 3rd-Party Widget & Script Orchestration

**Scope:** Improve **TBT** and **Speed Index** by offloading non-critical execution.

- **Widget Lazy-Loading:** Refactor the chat widget and heavy 3rd-party scripts (e.g., `recaptcha__en.js`) to load only upon user interaction (e.g., button click or scroll trigger).
- **Performance Acceptance Criteria:** Reduce **CLS** to < 0.1 and further reduce **TBT** by offloading 3rd-party execution from the initial page load.

### **🧩 Vue 3 Landing Page Implementation**

_Goal: Rebuild the FilipTravel landing page using a modern Vue 3 architecture that improves maintainability, scalability, and performance._

### 🎫 [LANDING-PAGE-EPIC] Vue 3 Landing Page

### Description

This Epic tracks the development of the new, high-performance Landing Page using the Vue 3 framework. The primary goal is to build a scalable, component-driven architecture that prioritizes modularity, performance (LCP, CLS), and reusability.

### Project Roadmap

- **Track A:** Shared UI Component Library (Atoms)
- **Track B:** Landing Page Feature Slicing (API + UI Integration)

### ⚙️ Strategy & Workflow

- **Foundation-First Dependency:** **Track A** must be completed first to establish the design system (Atoms). Once the foundations are ready, **Track B** tasks can be picked up and developed asynchronously.
- Each ticket in **Track B** is a self-contained feature unit. It bundles the UI components (Molecules/Organisms), business logic, and API integration, ensuring developers can own their feature from end-to-end without cross-team bottlenecks.

### 🎯 Definition of Done (DoD)

- [ ] **Performance:** Lighthouse score > 90 on mobile; critical CSS and lazy-loading implemented.
- [ ] **Component Standards:** All UI components documented and verified in **`Storybook`**.
- [ ] **Modularity:** High cohesion achieved by bundling UI components with their own API/Composable logic.
- [ ] **Testing:** Full regression testing passed on the final `LandingPage.vue` assembly.

### 🧱 Track A: Shared UI Library (Foundation)

_Goal: Establish the base library to unblock feature development._

- **[FE-01] Shared UI Atoms** | Design: [Figma Atoms]

### 🚀 Track B: Landing Page Feature Slices (Vertical Feature Units)

_Each ticket contains the UI component (Organism/Molecules) and its API logic._

| Ticket      | Scope                        | Mock API                            | Design Link          |
| ----------- | ---------------------------- | ----------------------------------- | -------------------- |
| **[FE-02]** | **Carousel Organism**        | `GET /api/shared/carousel-config`   | [Figma Carousel]     |
| **[FE-03]** | **SearchSection Feature**    | `POST /api/v1/search`               | [Figma Search]       |
| **[FE-04]** | **Featured Destinations**    | `GET /api/v1/destinations/featured` | [Figma Featured]     |
| **[FE-05]** | **Destinations Organism**    | `GET /api/v1/destinations/all`      | [Figma Destinations] |
| **[FE-06]** | **WellnessCarousel Feature** | `GET /api/v1/wellness`              | [Figma Wellness]     |
| **[FE-07]** | **Seasonal Offers Feature**  | `GET /api/v1/seasons`               | [Figma Seasons]      |
| **[FE-08]** | **Promotions & Services**    | `GET /api/v1/promotions`            | [Figma Promotions]   |

### 🛡️ Integration & QA Hand-off Protocol

1. **Component QA:** As each `[FE-XX]` ticket is marked "Done," it is verified by QA in the **Storybook** sandbox. This ensures atoms and molecules are bug-free before they are integrated into the full page.
2. **Parent Ticket Logic:** The `[LANDING-PAGE-EPIC]` serves as the master checklist. Once all feature slices (Track B) are merged, the Epic is updated to include the staging URL for final integration testing.
3. **Performance Verification:** A mandatory audit is performed on the staging environment to ensure all lazy-loading, async-importing, and caching strategies are functioning as expected.

### **TRACK A: Foundation (Shared UI - No API)**

_These tickets are the "Source of Truth" for visual consistency._

- **🎫 [FE-01] Shared Atoms**
  - **Requirement:** Build `Button.vue`, `Input.vue`, `Select.vue`, `Icon.vue`.
  - **Design:** [Link to Figma Atoms]
  - Acceptance Criteria:
    - [ ] **`Button.vue`:** Implemented strictly per Figma design system; supports all primary/secondary/ghost variants and states (hover, active, disabled).
    - [ ] **`Input.vue`:** Implemented strictly per Figma design system; supports label, placeholder, and error states.
    - [ ] **`Select.vue`:** Implemented strictly per Figma design system; handles open/closed states and option selection gracefully.
    - [ ] **`Icon.vue`:** Implemented strictly per Figma design system; supports configurable sizes and brand-compliant colors.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.

### **TRACK B: Feature-Driven Development (API + UI)**

_Each ticket contains the UI component (Molecule/Organism) and its corresponding Data/API logic._

- **🎫 [FE-02] Carousel Feature (Shared Organism)**
  - **Requirement:** Build `Carousel.vue`  + `useCarouselConfig.ts` (Shared). It must accept a generic `slides` prop.
  - **Mock API:** `GET /api/shared/carousel-config` (BE-Ticket: [API-101])
  - **Design:** [Link to Figma Carousel]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/shared/carousel-config`; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `Carousel.vue` implemented strictly per the provided Figma design system.
    - [ ] **Performance:** Implements lazy-loading for carousel images; fixed container aspect ratio ensures no layout shift (CLS) during load.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-03] SearchSection Feature (Feature Component)**
  - **Requirement:** Implement `SearchSection.vue` + `useSearch.ts`.
  - **Mock API:** `POST /api/v1/search`(BE-Ticket: [API-102])
  - **Design:** [Link to Figma Search Section]
  -
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully triggers `POST /api/v1/search` with the correct payload schema.
    - [ ] **Design System:** `SearchSection.vue` implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-04] Featured Destinations Feature (Feature Component)**
  - **Requirement:** Implement `FeaturedDestinations.vue` + `FeaturedDestinationCard.vue` (Molecule) + `useFeaturedDestinations.ts`.
  - **Mock API:** `GET /api/v1/destinations/featured`(BE-Ticket: [API-103])
  - **Design:** [Figma Featured]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/v1/destinations/featured` ; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `FeaturedDestinations.vue` + `FeaturedDestinationCard.vue` implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-05] Destinations Feature (Shared Organism)**
  - **Requirement:** Build `Destinations.vue`  + `useDestinations.ts`. Note: API returns grouped, flexi, and far categories.
  - **Mock API:** `GET /api/v1/destinations/all`(BE-Ticket: [API-104])
  - **Design:** [Figma Destinations]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/v1/destinations/all`; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `Destinations.vue`  implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-06] WellnessCarousel Feature (Feature-Local)**
  - **Requirement:** Build `WellnessCarousel.vue` + `useWellness.ts`.
  - **Mock API:** `GET /api/v1/wellness`(BE-Ticket: [API-105])
  - **Design:** [Figma Wellness]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/v1/wellness`; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `WellnessCarousel.vue`  implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-07] Seasonal Offers Feature**
  - **Requirement:** Build `SeasonSection.vue` + `SeasonCard.vue` + `useSeasons.ts`.
  - **Mock API:** `GET /api/v1/seasons`(BE-Ticket: [API-106])
  - **Design:** [Figma Seasons]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/v1/seasons`; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `SeasonSection.vue` + `SeasonCard.vue` implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.
- **🎫 [FE-08] Promotions & Services Feature**
  - **Requirement:** Build `PromotionCard.vue` + `AdditionalServiceCard.vue` + `usePromotions.ts`.
  - **Mock API:** `GET /api/v1/promotions`(BE-Ticket: [API-107])
  - **DoD:** Cards are reusable across the app.
  - **Design:** [Figma Promotions and Services]
  - Acceptance Criteria:
    - [ ] **API Integration:** Successfully fetches data from `GET /api/v1/promotions`; includes error handling for empty states or failed network requests.
    - [ ] **Design System:** `PromotionCard.vue` + `AdditionalServiceCard.vue` implemented strictly per the provided Figma design system.
    - [ ] **Documentation:** Fully documented in Storybook with all props, slots, and events verified.

---

## **Conclusion**

This document analyzed the performance characteristics of **filiptravel.rs**, identifying several bottlenecks affecting rendering speed, including large JavaScript bundles, render-blocking CSS, oversized images, and heavy API responses.

Several **immediate optimizations** were proposed that could significantly improve the current platform’s Lighthouse metrics without requiring a full architectural rewrite.

Additionally, a **Vue 3 component architecture** for the Landing Page was proposed, demonstrating how the page could be rebuilt using a modular, scalable structure based on **Atomic Design principles and feature-driven architecture**. The work breakdown organizes the development effort into clear, independent tasks that allow multiple developers to work in parallel.

The goal of this proposal is to improve both **performance and long-term maintainability**, ensuring the platform remains scalable for a large user base while enabling faster feature development in the future.
