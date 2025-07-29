---
layout: post
title: "Why My Spring Boot + Thymeleaf App Feels So Fast (and Why \"Boring\" Tech Wins More Than You Think)"
date: 2025-01-29
categories: [tech, architecture, spring-boot]
---

I recently built a CRUD‑heavy application using Spring Boot + Thymeleaf, and I was genuinely stunned by how *fast* it felt. After years of building React SPAs, I'd forgotten what instant page loads actually looked like. No loading spinners, no skeleton screens, just immediate content. The simplicity was equally surprising—no state management libraries, no API layer complexity, just straightforward server‑side rendering that worked. Here's what I discovered about why this "boring" stack performs so well, and when you might still want that React SPA.

---

## 1) Server‑Side Rendering: Work Done Before the First Paint

When a user requests `/tasks`, Spring Boot executes a controller, runs database queries, renders the Thymeleaf template, and returns **complete HTML**. The browser can paint immediately—no waiting for JS bundles to download, parse, and execute before data even appears.

Contrast that with many SPAs: the browser loads a JS shell, initializes the app, calls APIs for data, then renders. Those extra steps are visible to users, especially on slower networks or devices.

---

## 2) Minimal JavaScript Bundle Overhead

A production React build typically includes the React libraries, your app code, a router, state utilities, UI frameworks, polyfills, and transpiled output. Even when gzipped, those bundles add up.

In a Spring MVC app, you can load only what you need (e.g., Bootstrap's small JS for dropdowns). Less JavaScript means less to download, parse, and execute—often translating to lower time‑to‑interactive.

---

## 3) Fewer Network Round Trips

**Typical SPA flow**

1. `GET /tasks` → HTML shell
2. Download JS bundles
3. Parse/execute JavaScript
4. `GET /api/tasks` → task data
5. `GET /api/projects` → metadata for UI
6. Render UI

**Spring MVC flow**

1. `GET /tasks` → Fully rendered HTML with data
2. Done

Each additional request adds latency and increases the chance of user‑perceived delay.

---

## 4) Database Proximity Pays Off

A straightforward architecture like:

```
Browser → Spring Boot → MySQL (same server or low‑latency network)
```

keeps data access close to the renderer. Many SPA backends add hops—CDN → API Gateway → Load Balancer → API servers → Database—each with overhead. The simpler path yields consistently lower tail latency.

---

## 5) Sensible HTTP Caching

Spring can participate in HTTP caching with headers like `ETag`, `Last-Modified`, and `Cache-Control` (via filters/config). When the browser sends `If-None-Match` and nothing has changed, the server responds with `304 Not Modified`—no body, minimal bandwidth, faster loads. With server rendering, you can apply caching at the template, fragment, and resource levels for substantial wins.

---

## 6) State Management That Stays Simple

Client‑side state is powerful—and complicated. SPAs juggle optimistic updates, conflict resolution, cache invalidation, and error recovery.

In classic MVC: **submit form → server updates DB → redirect to a fresh page**. The database remains the single source of truth. Less duplication of state means fewer edge‑case bugs.

---

## 7) Query Efficiency by Design

JPA/Hibernate can eliminate the N+1 problem with patterns like `JOIN FETCH`:

```java
@Query("SELECT t FROM Task t LEFT JOIN FETCH t.project WHERE t.id = :id AND t.deletedAt IS NULL")
Optional<Task> findTaskWithProject(@Param("id") Long id);
```

Fetching related data in one query reduces chatter and improves predictability under load.

---

## 8) Smaller Over‑the‑Wire Payloads

For many pages, the HTML you send is exactly what the user needs to see—no extra structure, no unused fields. HTML also compresses very well. In practice, the payload for a server‑rendered table row can be smaller than the equivalent JSON response plus client rendering logic.

---

## 9) Progressive Enhancement by Default

Server‑rendered pages function without JavaScript:

* Links navigate
* Forms submit
* Content remains accessible

JavaScript then **enhances** the UI (dropdowns, date pickers, live search) rather than being a hard requirement for basic functionality. This baseline resilience pays dividends on constrained devices, corporate environments, and flaky networks.

---

## 10) Architectural Simplicity Becomes Performance

Typical SPA complexity involves bundlers, transpilers, code‑splitting, hydration, client caches, API clients, error boundaries, suspense, and more. The Spring MVC alternative is direct:

* Controllers return views
* Templates render HTML
* Forms POST data
* The database stores state

Fewer layers mean fewer failure modes and faster paths from click to response.

---

## The "Boring Technology" Advantage

Dan McKinley's "Choose Boring Technology" captures a truth many teams learn the hard way: stable, well‑understood tools help you ship. Server‑side rendering dates to the 1990s, MVC is deeply internalized by the industry, and relational databases have benefited from decades of optimization. Your innovation budget can go to user value, not plumbing.

### Real‑World Endorsements

Large, durable platforms (e.g., GitHub, Stack Overflow, Wikipedia) have long leaned on server‑rendered architectures. The broader ecosystem reflects this pendulum: Next.js/Nuxt brought SSR back to JS frameworks; Remix, HTMX, Phoenix LiveView, and Rails' Hotwire embrace server‑first patterns with minimal client JS.

### Maintenance Over the Long Haul

A server‑rendered monolith (à la DHH's "Majestic Monolith") centralizes logic, keeps deployments simple, and preserves transactional integrity. Five years on, a straightforward Spring Boot app tends to age gracefully; many 2019 SPA stacks already feel dated due to churn in libraries, build tooling, and patterns.

---

## When a SPA *Is* the Right Choice

Balance matters. Prefer a SPA when your product genuinely needs:

* **Rich, long‑lived client state** (complex editors, design tools)
* **Offline‑first workflows** with background sync
* **Highly interactive graphics** (e.g., canvases, realtime dashboards that must update without full navigations)
* **App‑like navigation** where full page loads break the UX expectation
* **Third‑party embed** constraints that require a self‑contained client

If these needs are central, a SPA can be a better fit—potentially paired with SSR/streaming to regain initial load performance.

---

## A Practical Checklist for Spring MVC Speed

* **Render server‑side first.** Treat JS as enhancement.
* **Use PRG (Post/Redirect/Get).** Keep state authoritative in the DB.
* **Eliminate N+1 queries.** Audit with logs and add `JOIN FETCH` where appropriate.
* **Profile your controllers.** Track TTFB and database timings.
* **Enable HTTP caching.** Configure `ETag`/`Last‑Modified` where applicable; cache static assets aggressively.
* **Trim JS.** Only include scripts that measurably improve UX.
* **Keep the path short.** Minimize network hops between app and database.

---

## Closing Thought

For data‑centric web applications, server‑rendered Spring Boot with Thymeleaf delivers a fast, resilient user experience with less operational complexity. "Boring" isn't a compromise—it's often the most effective way to ship quickly, maintain velocity, and keep users happy over the long term.