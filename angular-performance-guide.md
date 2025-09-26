# ⚡ Angular Performance Troubleshooting Guide (Beginner-Friendly)

Performance problems in Angular apps can come from **slow API calls, excessive change detection, large lists, or heavy bundles**.
This guide walks you through tools, steps, and fixes with beginner-friendly instructions.

---

## 📑 Table of Contents

- [⚡ Angular Performance Troubleshooting Guide (Beginner-Friendly)](#-angular-performance-troubleshooting-guide-beginner-friendly)
  - [📑 Table of Contents](#-table-of-contents)
  - [1. How to Spot Performance Problems](#1-how-to-spot-performance-problems)
    - [Symptoms](#symptoms)
  - [2. Profiling \& Debugging Tools](#2-profiling--debugging-tools)
    - [A. Angular DevTools (Chrome / Edge)](#a-angular-devtools-chrome--edge)
    - [B. Using the Angular Profiler](#b-using-the-angular-profiler)
    - [C. Chrome Performance Tab](#c-chrome-performance-tab)
    - [D. Chrome Network Tab](#d-chrome-network-tab)
  - [3. Common Problems \& Fixes](#3-common-problems--fixes)
    - [A. Excessive Change Detection](#a-excessive-change-detection)
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
    - [3. Reduce Change Detection on Input](#3-reduce-change-detection-on-input)
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

### A. Angular DevTools (Chrome / Edge)

1. Install:
   - Open **Chrome Web Store** → search **"Angular DevTools"** → *Add to Chrome*.
2. Open your app in Chrome → Press **F12** → You should now see a new tab:
   - 🅰️ **Angular** – shows your component tree and profiler.

---

### B. Using the Angular Profiler

1. Open the **🅰️ Angular tab** in DevTools.
2. Go to the **Profiler** sub-tab.
3. Click the **record button (●)**.
4. Interact with your app (click, scroll, type).
5. Click the **stop button (■)**.
6. You'll see performance data including:
   - **Change detection cycles**
   - **Component render times**
   - **Source maps for debugging**

👉 Example: If `UserListComponent` shows high change detection time, it's triggering too many cycles.

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

- If "Waiting (TTFB)" is long → backend is slow.
- If "Content Download" is long → too much data.

---

## 3. Common Problems & Fixes

### A. Excessive Change Detection

- **Problem**: Angular runs change detection too frequently, causing slow re-renders.
- **Fix**: Use `OnPush` change detection, trackBy functions, and avoid mutable objects.

```typescript
// Use OnPush to reduce unnecessary checks
@Component({
  selector: 'app-user-card',
  template: `<div>{{user.name}}</div>`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user: User;
}

// Use trackBy in *ngFor to optimize list updates
trackByUserId(index: number, user: User): string {
  return user.id;
}
```

---

### B. Large Lists

- **Problem**: Rendering thousands of items at once makes UI freeze.
- **Fix**: Use virtual scrolling with Angular CDK.

```bash
ng add @angular/cdk
```

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackByFn" class="item">
        {{item}}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
export class VirtualListComponent {
  items = Array.from({length: 100000}).map((_, i) => `Item #${i}`);
  trackByFn(index: number, item: string): number {
    return index;
  }
}
```

---

### C. Slow API Calls

- **Problem**: Waiting for backend makes app feel slow.
- **Steps to Debug**:
  1. Check **Network tab** → look for slow or failed requests.
- **Fixes**:
  - Add **loading states** so users see a spinner.
  - Use **pagination** (don't load 10,000 items at once).
  - Cache data with **Angular's HttpClient interceptors** or **NgRx Data**:

```typescript
// Simple caching with HttpClient
@Injectable()
export class CacheInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Add caching logic here
    return next.handle(req);
  }
}
```

---

### D. Global State Causing Re-renders

- **Problem**: Using NgRx or services incorrectly → entire component tree re-renders.
- **Fix**: Use selectors and avoid subscribing to entire state.

```typescript
// BAD: subscribes to entire state
this.store.select(state => state).subscribe(/* ... */);

// GOOD: only subscribes to specific slice
this.store.select(selectItems).subscribe(/* ... */);
```

---

### E. Heavy Images & Bundles

- **Problem**: Large images or oversized JS bundles.
- **Fixes**:
  - Compress images (WebP/AVIF).
  - Lazy load modules:

```typescript
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module').then(m => m.UsersModule)
  }
];
```

---

## 4. Step-by-Step Troubleshooting Process

1. **Measure**: Start with Angular DevTools Profiler + Chrome Performance tab.
2. **Check network**: Are API calls slow?
3. **Check change detection**: Look for excessive cycles.
4. **Apply fixes**: OnPush strategy, virtualization, caching, lazy loading.
5. **Re-measure**: Compare before vs after.

---

## 5. Example Workflow

You notice a "Users" page is slow:

1. **Angular Profiler** shows `UserListComponent` triggers many change detection cycles.
2. **Network tab** shows API returns 5,000 users in 3 seconds.
3. Fixes:
   - Backend: Add pagination (`/users?page=1&limit=50`).
   - Frontend: Use virtual scrolling with CDK.
   - Set components to `OnPush` change detection.

👉 Result: API loads faster, UI scrolls smoothly, change detection time cut by 90%.

---

## 6. Final Checklist

- [ ] Install Angular DevTools & practice profiling.
- [ ] Use Chrome Performance tab to spot bottlenecks.
- [ ] Use Network tab to check API slowness.
- [ ] Use OnPush change detection strategy.
- [ ] Implement trackBy functions in *ngFor.
- [ ] Virtualize big lists with Angular CDK.
- [ ] Paginate & cache API data.
- [ ] Lazy-load modules.
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

- Use lazy loading for modules.
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

✅ **Rule of Thumb**: If your LCP is bad, **90% of the time it's a big image or blocking script**.

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

- Don't push content down with banners or dynamic headers.
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

### 3. Reduce Change Detection on Input

- Use `OnPush` change detection to avoid unnecessary checks.
- Split large components into smaller ones.

### 4. Use Debouncing/Throttling

- For inputs like search boxes, debounce API calls.

### 5. Load Less on Initial Interaction

- Prefetch data if possible.
- Use caching with HttpClient interceptors.

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
