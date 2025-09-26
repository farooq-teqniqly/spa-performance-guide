# ⚡ React Performance Troubleshooting Guide (Beginner-Friendly)

Performance problems in React apps can come from **slow API calls, too many re-renders, large UI lists, or heavy bundles**.  
This guide walks you through tools, steps, and fixes with beginner-friendly instructions.

---

## 📑 Table of Contents

- [⚡ React Performance Troubleshooting Guide (Beginner-Friendly)](#-react-performance-troubleshooting-guide-beginner-friendly)
  - [📑 Table of Contents](#-table-of-contents)
  - [1. How to Spot Performance Problems](#1-how-to-spot-performance-problems)
    - [Symptoms](#symptoms)
  - [2. Profiling \& Debugging Tools](#2-profiling--debugging-tools)
    - [A. React DevTools (Chrome / Edge)](#a-react-devtools-chrome--edge)
    - [B. Using the React Profiler](#b-using-the-react-profiler)
    - [C. Chrome Performance Tab](#c-chrome-performance-tab)
    - [D. Chrome Network Tab](#d-chrome-network-tab)
  - [3. Common Problems \& Fixes](#3-common-problems--fixes)
    - [A. Too Many Re-renders](#a-too-many-re-renders)
    - [B. Large Lists](#b-large-lists)
    - [C. Slow API Calls](#c-slow-api-calls)
    - [D. Global State Causing Re-renders](#d-global-state-causing-re-renders)
    - [E. Heavy Images \& Bundles](#e-heavy-images--bundles)
  - [4. Step-by-Step Troubleshooting Process](#4-step-by-step-troubleshooting-process)
  - [5. Example Workflow](#5-example-workflow)
  - [6. Final Checklist](#6-final-checklist)
  - [🚀 LCP (Largest Contentful Paint) Troubleshooting Checklist](#-lcp-largest-contentful-paint-troubleshooting-checklist)
    - [1. Identify the LCP Element](#1-identify-the-lcp-element)
    - [2. Optimize Images (Most Common LCP Problem)](#2-optimize-images-most-common-lcp-problem)
    - [3. Reduce Render-Blocking Resources](#3-reduce-render-blocking-resources)
    - [4. Reduce JavaScript \& Bundle Size](#4-reduce-javascript--bundle-size)
    - [5. Use Caching and CDNs](#5-use-caching-and-cdns)
    - [6. Prioritize Above-the-Fold Content](#6-prioritize-above-the-fold-content)
    - [7. Measure Again](#7-measure-again)
  - [📐 CLS (Cumulative Layout Shift) Troubleshooting Checklist](#-cls-cumulative-layout-shift-troubleshooting-checklist)
    - [1. Identify Shifts](#1-identify-shifts)
    - [2. Reserve Space for Images/Videos](#2-reserve-space-for-imagesvideos)
    - [3. Reserve Space for Ads/Embeds](#3-reserve-space-for-adsembeds)
    - [4. Avoid Injecting Content Above Existing Elements](#4-avoid-injecting-content-above-existing-elements)
    - [5. Use Font Loading Strategies](#5-use-font-loading-strategies)
    - [6. Measure Again](#6-measure-again)
  - [⚡ INP (Interaction to Next Paint) Troubleshooting Checklist](#-inp-interaction-to-next-paint-troubleshooting-checklist)
    - [1. Identify Slow Interactions](#1-identify-slow-interactions)
    - [2. Optimize Event Handlers](#2-optimize-event-handlers)
    - [3. Reduce Re-renders on Input](#3-reduce-re-renders-on-input)
    - [4. Use Debouncing/Throttling](#4-use-debouncingthrottling)
    - [5. Load Less on Initial Interaction](#5-load-less-on-initial-interaction)
    - [6. Measure (Again)](#6-measure-again-1)
  - [🎯 Summary](#-summary)

---

## 1. How to Spot Performance Problems

### Symptoms

- **UI is laggy** → scrolling stutters, typing is slow, buttons feel unresponsive.
- **API feels slow** → data takes too long to load.
- **App takes too long to open** → blank screen before first paint.

👉 Before fixing anything, always **measure first**.

---

## 2. Profiling & Debugging Tools

### A. React DevTools (Chrome / Edge)

1. Install:
   - Open **Chrome Web Store** → search **“React Developer Tools”** → *Add to Chrome*.  
2. Open your app in Chrome → Press **F12** → You should now see two new tabs:  
   - ⚛️ **Components** – shows your component tree.  
   - ⚛️ **Profiler** – lets you record performance.  

---

### B. Using the React Profiler

1. Open the **⚛️ Profiler tab** in DevTools.  
2. Click the **record button (●)**.  
3. Interact with your app (click, scroll, type).  
4. Click the **stop button (■)**.  
5. You’ll see a **flame graph** (bars with colors).  
   - **Wider bars = slower components.**  
   - Hover over a bar → shows component name and render time.  

👉 Example: If `UserList` always shows up in red/orange, it’s taking too long to render.

---

### C. Chrome Performance Tab

1. Open DevTools (**F12**) → **Performance tab**.  
2. Click **record (●)** → reload the page or interact with the app.  
3. Stop recording.  
4. Look at timeline:  
   - **Purple = JavaScript execution**  
   - **Green = rendering**  
   - **Orange = painting**  

👉 If purple dominates, your code is slow. If green/orange dominate, too much DOM rendering.

---

### D. Chrome Network Tab

1. Open DevTools (**F12**) → **Network tab**.  
2. Reload the app.  
3. Look at the list of API calls:  
   - **Name** – file or endpoint name.  
   - **Status** – HTTP code (200 = OK, 500 = error).  
   - **Time** – how long it took.  

👉 Look for requests that take **>500ms**.  
👉 Hover over a request → check **Waterfall view**:

- If “Waiting (TTFB)” is long → backend is slow.  
- If “Content Download” is long → too much data.

---

## 3. Common Problems & Fixes

### A. Too Many Re-renders

- **Problem**: Components re-render even if props/state didn’t change.  
- **Fix**: Use `React.memo`, `useCallback`, and `useMemo`.

```jsx
// Prevent child re-render unless props change
const UserCard = React.memo(function UserCard({ user }) {
  return <div>{user.name}</div>;
});

// Stable function reference
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

---

### B. Large Lists

- **Problem**: Rendering thousands of items at once makes UI freeze.  
- **Fix**: Virtualize lists with `react-window`.

```bash
npm install react-window
```

```jsx
import { FixedSizeList as List } from "react-window";

const MyList = ({ items }) => (
  <List height={500} width={300} itemSize={35} itemCount={items.length}>
    {({ index, style }) => (
      <div style={style}>{items[index]}</div>
    )}
  </List>
);
```

---

### C. Slow API Calls

- **Problem**: Waiting for backend makes app feel slow.  
- **Steps to Debug**:  
  1. Check **Network tab** → look for slow or failed requests.  
- **Fixes**:  
  - Add **loading states** so users see a spinner.  
  - Use **pagination** (don’t load 10,000 items at once).  
  - Cache data with **React Query**:

```bash
npm install @tanstack/react-query
```

```jsx
const { data, isLoading } = useQuery(["users"], fetchUsers);
```

---

### D. Global State Causing Re-renders

- **Problem**: Using Context or Redux incorrectly → entire tree re-renders.  
- **Fix**: Use selectors.

```jsx
// BAD: re-renders whenever *any* state changes
const allState = useSelector(state => state);

// GOOD: only re-renders when this slice changes
const item = useSelector(state => state.items[id]);
```

---

### E. Heavy Images & Bundles

- **Problem**: Large images or oversized JS bundles.  
- **Fixes**:  
  - Compress images (WebP/AVIF).  
  - Lazy load components:

```jsx
const UserPage = React.lazy(() => import("./UserPage"));
```

---

## 4. Step-by-Step Troubleshooting Process

1. **Measure**: Start with Profiler + Performance tab.  
2. **Check network**: Are API calls slow?  
3. **Check rendering**: Look for wasted re-renders.  
4. **Apply fixes**: Memoization, virtualization, caching, code splitting.  
5. **Re-measure**: Compare before vs after.

---

## 5. Example Workflow

You notice a “Users” page is slow:  

1. **Profiler** shows `UserList` renders 1,000 rows.  
2. **Network tab** shows API returns 5,000 users in 3 seconds.  
3. Fixes:  
   - Backend: Add pagination (`/users?page=1&limit=50`).  
   - Frontend: Virtualize list with `react-window`.  
   - Wrap `UserCard` in `React.memo`.  

👉 Result: API loads faster, UI scrolls smoothly, rendering time cut by 90%.

---

## 6. Final Checklist

- [ ] Install React DevTools & practice profiling.  
- [ ] Use Chrome Performance tab to spot bottlenecks.  
- [ ] Use Network tab to check API slowness.  
- [ ] Memoize components (`React.memo`, `useCallback`, `useMemo`).  
- [ ] Virtualize big lists with `react-window`.  
- [ ] Paginate & cache API data.  
- [ ] Lazy-load components.  
- [ ] Compress images & analyze bundles.  
- [ ] Always measure before and after changes.

---

## 🚀 LCP (Largest Contentful Paint) Troubleshooting Checklist

If **LCP > 2.5s**, the page feels slow to load. Follow these steps to improve it:

### 1. Identify the LCP Element

- Open Chrome DevTools → **Performance tab**.  
- Record a session, then hover over the **LCP marker** (diamond icon).  
- DevTools highlights the element (e.g., hero image, `<h1>`, large block of text).  

### 2. Optimize Images (Most Common LCP Problem)

- Use **modern formats**: WebP or AVIF instead of PNG/JPG.  
- Resize to **actual display size**.  
- Enable **lazy loading** for images below the fold.

### 3. Reduce Render-Blocking Resources

- Inline critical CSS.  
- Defer non-critical CSS/JS.  
- Add `defer` to scripts.

### 4. Reduce JavaScript & Bundle Size

- Use code splitting.  
- Analyze with `webpack-bundle-analyzer`.  
- Remove unused libraries.

### 5. Use Caching and CDNs

- Serve assets from CDN.  
- Add caching headers.

### 6. Prioritize Above-the-Fold Content

- Load text/logo/hero first.  
- Defer widgets, ads, or carousels.

### 7. Measure Again

- After fixes, re-run Performance tab and validate.

✅ **Rule of Thumb**: If your LCP is bad, **90% of the time it’s a big image or blocking script**.

---

## 📐 CLS (Cumulative Layout Shift) Troubleshooting Checklist

If **CLS > 0.1**, users see content jump around. Fix by stabilizing layout.

### 1. Identify Shifts

- Open Chrome DevTools → **Performance tab**.  
- Look for **layout shift markers** (blue triangles).  
- Click them to see which elements shifted.

### 2. Reserve Space for Images/Videos

- Always set `width` and `height` attributes.

### 3. Reserve Space for Ads/Embeds

- Give iframes or ads fixed size containers.  
- Use placeholders or skeleton loaders.

### 4. Avoid Injecting Content Above Existing Elements

- Don’t push content down with banners or dynamic headers.  
- Insert new content below the fold.

### 5. Use Font Loading Strategies

- Use `font-display: swap` to prevent invisible text.  
- Preload critical fonts with `<link rel="preload" as="font">`.

### 6. Measure Again

- Check CLS in DevTools after changes.  
- Goal: CLS ≤ 0.1.

✅ **Rule of Thumb**: Unexpected jumps almost always mean **missing width/height attributes**.

---

## ⚡ INP (Interaction to Next Paint) Troubleshooting Checklist

If **INP > 200ms**, your app feels unresponsive after user actions.

### 1. Identify Slow Interactions

- Open Chrome DevTools → **Performance tab**.  
- Record while clicking/typing.  
- Look for **long tasks (>50ms)** blocking input.

### 2. Optimize Event Handlers

- Keep click/keypress handlers lightweight.  
- Move heavy logic into Web Workers or background tasks.

### 3. Reduce Re-renders on Input

- Use `React.memo` to avoid unnecessary re-renders.  
- Split large components into smaller ones.

### 4. Use Debouncing/Throttling

- For inputs like search boxes, debounce API calls.

### 5. Load Less on Initial Interaction

- Prefetch data if possible.  
- Use Suspense/React Query caching.

### 6. Measure (Again)

- After fixes, re-run Performance tab.  
- Goal: INP ≤ 200ms.

✅ **Rule of Thumb**: If your INP is bad, **look for long JavaScript tasks blocking the main thread**.

---

## 🎯 Summary

Now your guide covers **all three Core Web Vitals**:

- **LCP**: Loading speed (fix images, blocking scripts).  
- **CLS**: Visual stability (fix missing dimensions, avoid shifts).  
- **INP**: Responsiveness (fix heavy event handlers, long tasks).  

Always: **Measure → Fix → Measure Again**.
