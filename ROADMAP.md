# Popcorn Time — Build Roadmap

Goal: turn a tutorial app into something I fully understand and built myself.
Learning project, not a product.

## Rules (so this isn't vibe coding)

1. **I write the code.** Claude explains concepts, gives hints, reviews. Full solutions only on request after I'm stuck 20–30 min.
2. **Approach first.** Before each ticket, write 2–3 sentences in the PR description: what I'll change and why.
3. **One ticket = one branch = one PR.** Branch name: `p<phase>-<ticket>-<slug>` (e.g. `p0-1-search-race`).
4. **Definition of done:** works in the browser · `npm run lint` passes · `npm run build` passes · I can explain every line · checkbox ticked here.
5. **Cadence:** fixed sessions per week, small tickets. Shipping one small ticket beats planning five.

---

## Phase 0 — Fix & own it

Concepts: async/await, AbortController, error handling, ES modules, build-time env vars. (All JSA material.)

- [ ] **0.1 Search race condition**
  Typing "bat" → "batman" can show results for "bat" if that request finishes last.
  - Done when: rapid typing always shows results for the latest term; no error message flashes for cancelled requests.
  - Hints: `fetch(url, { signal })` · `useEffect` cleanup function · aborted fetch throws an error with `name === 'AbortError'`.

- [ ] **0.2 Trending crash on failure**
  `getTrendingMovies` (`src/appwrite.js`) returns `undefined` on error → `trendingMovies.length` throws.
  - Done when: Appwrite down/misconfigured → app still renders, trending section just hides.
  - Hint: what should a function that "gets a list" return when it fails? Decide who handles the error — `appwrite.js` or `App.jsx` — not both.

- [ ] **0.3 Extract the API layer**
  Move all TMDB logic out of `App.jsx` into `src/api/tmdb.js` exporting `searchMovies(query, signal)` and `discoverMovies(signal)`.
  - Done when: `App.jsx` has no URLs or API keys in it; components only call functions.
  - Hint: the API module throws on failure; the component decides what message to show.

- [ ] **0.4 Fix the deployed build (env vars)**
  `.env` isn't in the repo, and neither the Dockerfile nor the workflow passes build args → the live build likely has no API key.
  - Done when: deployed app loads movies.
  - Hints: Vite reads env at **build** time, not runtime · Dockerfile `ARG` + `ENV` · workflow `buildArguments` from GitHub secrets.
  - Also: rename the workflow file (it says `popcron`).

- [ ] **0.5 Small cleanups**
  Double slash in poster URLs (`w500/${poster_path}` — `poster_path` already starts with `/`) · `star.svg`/`search.svg` use relative paths (will break on nested routes in Phase 1) · remove committed `.DS_Store`.

---

## Phase 1 — Movie detail page

Concepts: client-side routing, URL params, SPA hosting.

- [ ] **1.1 Add React Router**, `/` = current home, `/movie/:id` = detail page. Cards link to it.
- [ ] **1.2 Detail page:** backdrop, overview, runtime, genres, rating. (TMDB `/movie/{id}`)
- [ ] **1.3 Trailer + cast** using `append_to_response=videos,credits` — one request, not three.
- [ ] **1.4 Loading / error / not-found states** for a bad `:id`.
- [ ] **1.5 Fix refresh 404 in production.** Refresh `/movie/123` on the deployed site → nginx 404. Write an `nginx.conf` with an SPA fallback and copy it in the Dockerfile.
  - Understand *why* it breaks before fixing it.

---

## Phase 2 — Real browsing

Concepts: URL as state, `URLSearchParams`, `IntersectionObserver`, custom hooks.

- [ ] **2.1 `useMovies` custom hook** — owns fetching, loading, error, abort. `App.jsx` just renders.
- [ ] **2.2 Search term in the URL** (`/?q=batman`) — shareable, survives refresh, back button works.
- [ ] **2.3 Filters:** genre + year + sort, also in the URL. (TMDB `/genre/movie/list`, `/discover/movie` params)
- [ ] **2.4 Infinite scroll** with `IntersectionObserver` (no library). Handle "no more pages".
- [ ] **2.5 Debounce without `react-use`** — write your own `useDebounce`, remove the dependency.

---

## Phase 3 — Own backend (the big step)

Concepts: Node, REST design, secrets, caching, CORS, rate limiting. This is where frontend → full-stack.

Structure: move to `client/` and `server/` folders.

- [ ] **3.1 Express server** with `GET /api/health`.
- [ ] **3.2 TMDB proxy:** `GET /api/movies/search?q=`, `/api/movies/discover`, `/api/movies/:id`. TMDB key lives **only** on the server. Frontend calls your API.
  - Done when: search the built JS bundle for your TMDB key → not found.
- [ ] **3.3 Input validation** — bad/missing params → `400` with a clear JSON error.
- [ ] **3.4 In-memory cache** (Map + TTL) for TMDB responses. Log hits/misses. Then ask: what breaks with 2 server instances?
- [ ] **3.5 Trending moves to the server:** `POST /api/searches` + `GET /api/trending`, stored in Postgres (Supabase). Remove Appwrite.
- [ ] **3.6 Rate limiting** on `POST /api/searches` so nobody can spam trending.
- [ ] **3.7 Dev setup:** Vite proxy `/api` → server, one command runs both.

---

## Phase 4 — Users

Concepts: auth, sessions vs JWT, password hashing, relational schema.

- [ ] **4.1 Schema design on paper first:** `users`, `watchlist`, `ratings`. Keys, relations, unique constraints. Get it reviewed before writing SQL.
- [ ] **4.2 Sign up / log in / log out** (httpOnly cookie session). Hash with bcrypt/argon2.
- [ ] **4.3 Protected routes** server-side (middleware) and client-side (redirect).
- [ ] **4.4 Watchlist:** add/remove from card + detail page, `/watchlist` page.
- [ ] **4.5 Personal ratings** (1–10) shown next to TMDB rating.

---

## Phase 5 — Make it professional

- [ ] **5.1 Vitest** — unit tests for pure functions (API URL building, cache, debounce).
- [ ] **5.2 API tests** with Supertest for each endpoint.
- [ ] **5.3 CI:** lint + test + build must pass on every PR; deploy only from `main` after that.
- [ ] **5.4 Containerize the server**, `docker compose up` runs client + server + db locally.
- [ ] **5.5 README rewrite** — architecture diagram, how to run, decisions I made and why.

---

## Phase 6 — My twist

Something the tutorial never had. Pick one when I get here:
- "What should I watch tonight?" — recommendations from my watchlist + ratings
- Watch party: share a link, friends vote, winner picked
- Stats page: my genres, decades, average rating vs TMDB

---

## Log

| Date | Ticket | Notes / what I learned |
|---|---|---|
| | | |
