---
tags:
  - frontend/rendering
status: seed
---

# Rendering Strategies

> Every Next.js App Router route is rendered as a mix of two base modes — static (built once, served from cache) and dynamic (rendered per request) — and the familiar acronyms (SSG, SSR, ISR, CSR) are really just names for how you lean on that split. Next.js's own current docs don't use these labels anymore (they say "static rendering" and "dynamic rendering"), but the mapping below is exact, so the old terms still work as a map onto the current model.

## When to reach for it

### SSG — Static Site Generation (Next.js: "static rendering")

Content that's identical for every visitor and doesn't depend on anything request-specific — marketing pages, docs, blog posts, a product catalog that isn't personalized. This is the default you get automatically unless something forces dynamic rendering, so most of a typical app should land here without extra effort. It's also the strongest option for SEO: the HTML is fully built ahead of time, so a crawler sees complete page data and metadata without running any JavaScript.

### SSR — Server-Side Rendering (Next.js: "dynamic rendering")

Anything genuinely request-specific: a dashboard that reads the logged-in user's cookies, a page that behaves differently based on `searchParams`, draft-preview mode. You don't "choose" SSR so much as trigger it — reading a Request-time API (`cookies()`, `headers()`, `searchParams`, `draftMode()`) anywhere in the render path opts that part of the route into it. SEO-wise it's just as safe as SSG — the HTML is still fully built before it reaches the browser, only at request time instead of build time — so it's the right default for dynamic pages that still need to be indexed (a news feed, a personalized-but-public listing).

### ISR — Incremental Static Regeneration

The middle ground: content that's *mostly* static but changes sometimes (a product page, a blog post that gets edited, a catalog with far more possible URLs than you want to prerender at build time). You get build-time-cheap, cache-served pages that still refresh — either on a timer or on demand right after the content actually changes. The SEO case for ISR specifically is scale: it keeps SSG's crawlability and freshness without forcing a full rebuild for every page on a site with millions of them — generate on a per-page basis instead.

### CSR — Client-Side Rendering (Next.js: Client Components)

Whatever genuinely needs the browser: local state, effects, event handlers, `localStorage`, geolocation, a canvas, a third-party widget, anything that only the client knows (viewport size, a live WebSocket update). This isn't CSR in the old SPA sense, though — a Client Component still gets rendered on the server for the first paint, then hydrates. True client-only fetching (fetch-after-mount, nothing from the server) is rare in the App Router and usually a sign the data should've come from a Server Component instead.

**Not recommended for SEO**, full stop, when it means the initial HTML genuinely has no content until JavaScript runs — if page data and metadata aren't available on page load without JS, a crawler may never see them. Reserve plain CSR for content that doesn't need to be indexed at all: a logged-in dashboard, an account settings page, anything behind auth.

### Streaming (no classic acronym — the delivery mechanism under the others)

Any route where part of the data is slow but the rest isn't — don't make a fast page wait on a slow widget. Give the user the shell and the fast parts immediately, stream in the slow part when it resolves. This is what makes ISR and SSR feel fast instead of blocking.

## How

### SSG — static rendering

A component is static unless it reads a Request-time API. The output — HTML plus the RSC Payload — is produced at build time (or in the background during revalidation) and can be cached on a CDN. No special syntax is required; this is what you get by doing nothing.

`generateStaticParams` pre-builds specific dynamic routes at build time — you tell Next.js which param values are worth prerendering, and it builds those pages up front:

```tsx
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map((post) => ({ slug: post.slug }))
}
```

### SSR — dynamic rendering

Triggered, not configured. Reading `cookies()`, `headers()`, `searchParams`, or `draftMode()` anywhere in a component's render opts that part of the tree into request-time rendering:

```tsx
import { cookies } from 'next/headers'

export default async function Dashboard() {
  const session = (await cookies()).get('session')
  // reading cookies() here makes this component dynamic
  return <p>Welcome back, session {session?.value}</p>
}
```

### ISR — time-based and on-demand revalidation

Time-based: set a `revalidate` window either on the `fetch` call or as route segment config, and Next.js serves the cached page while regenerating it in the background after the window passes:

```tsx
fetch('https://api.example.com/posts', { next: { revalidate: 60 } })
// or, for the whole route:
export const revalidate = 60
```

On-demand: call `revalidatePath` or `revalidateTag` from a Server Action or Route Handler right after a mutation, so the next request gets fresh content without waiting for the timer:

```tsx
import { revalidateTag } from 'next/cache'

export async function updatePost(id: string, data: PostData) {
  await db.posts.update(id, data)
  revalidateTag('posts')
}
```

### CSR — Client Components

Every component is a Server Component by default: it renders on the server, can fetch data directly, and costs zero client JavaScript, but can't use state or browser APIs. Add `"use client"` at the top of a file to pull it (and everything it imports) into the client bundle so it can use `useState`, effects, and event handlers:

```tsx
'use client'

export function LikeButton() {
  const [liked, setLiked] = useState(false)
  return <button onClick={() => setLiked(!liked)}>{liked ? 'Liked' : 'Like'}</button>
}
```

It still renders on the server for the initial HTML — `"use client"` means "this needs to run in the browser too," not "this only exists in the browser." That's a bundling boundary, not a rendering location.

### Streaming

A `loading.tsx` file in a route segment automatically wraps that whole segment in a Suspense boundary; a manual `<Suspense fallback={...}>` lets you place the boundary anywhere in the tree, around just the slow piece:

```tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <>
      <Header />
      <Suspense fallback={<p>Loading comments…</p>}>
        <Comments />
      </Suspense>
    </>
  )
}
```

### Cache Components — the newer, opt-in evolution of ISR and Streaming combined

Enable it explicitly in `next.config.ts`:

```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  cacheComponents: true,
  partialPrefetching: true,
}

export default nextConfig
```

With it on, mark cacheable code with `"use cache"`, set how long it stays fresh with `cacheLife()`, tag it with `cacheTag()`, and invalidate on demand with `updateTag()`. The payoff is the **App Shell**: the generic, URL-independent part of a page gets prerendered and served instantly — even for a param combination `generateStaticParams` never listed — while the specific content streams in and gets cached for next time. This is what "ISR" becomes once Cache Components is on; the `revalidate`-based ISR above is still the default when the flag is off.

## SEO — the strategy choice is also an SEO choice

The rule of thumb (straight from Next.js's own SEO guide, not just a convention): **the most important thing for SEO is that page data and metadata are available on page load without JavaScript** — that means SSG or SSR, not CSR. And this is decided per page, not per app: a typical Next.js site mixes all of them — blog posts statically generated (SSG), a news feed server-rendered (SSR), a customer dashboard client-rendered (CSR) because nobody needs Google to index someone's private account page.

## Gotchas

- **"Static rendering" is the SSG you can lose without noticing.** A single `cookies()` call deep inside a shared layout can opt a lot more of the route into SSR/dynamic rendering than you intended — it isn't always visibly contained unless you deliberately wrap the dynamic read in its own `<Suspense>` boundary.
- **The Suspense boundary has to be *around or below* the dynamic read, not above it.** Awaiting `params` (or any Request-time API) above a `<Suspense>` boundary ties everything above that boundary to the specific URL — defeating the point of a shared App Shell. The read needs to happen inside the boundary for Next.js to treat only that inner part as dynamic.
- **CSR ("use client") is a bundling directive, not a "renders only in the browser" switch.** It still gets server-rendered for the first paint and then hydrates — conflating it with classic SPA-style CSR leads to wrong assumptions about SEO and first-paint content.
- **Time-based ISR alone can serve stale content for the entire revalidate window after a real edit.** If staleness right after a content change is unacceptable, pair the timer with an on-demand `revalidatePath`/`revalidateTag` (or `updateTag` under Cache Components) called from wherever the mutation actually happens.
- **Cache Components is opt-in, not the unconditional default yet.** `"use cache"`, `cacheLife()`, `cacheTag()`, and `updateTag()` only apply once `cacheComponents: true` is set in `next.config.ts` — check which model a given project is actually running before assuming either set of APIs is in effect.
- **The acronyms themselves are borrowed, not official Next.js vocabulary right now.** If you read the current docs directly, expect "static rendering" / "dynamic rendering" / "Cache Components", not SSG/SSR/ISR — useful to know so a doc search for "SSR" doesn't come up empty.
- **CSR's SEO cost isn't a Google-specific quirk.** Any crawler or bot that doesn't execute JavaScript — other search engines, link-preview generators, some AI crawlers — sees the same empty shell a Googlebot-without-JS would. Don't reason about this as "Google can render JS now so it's probably fine."

## Sources

- [Next.js Docs — ISR with Cache Components](https://nextjs.org/docs/app/guides/incremental-static-regeneration-cache-components)
- [Next.js Docs — Glossary](https://nextjs.org/docs/app/glossary)
- [Next.js Learn — Rendering Strategies for SEO](https://nextjs.org/learn/seo/rendering-strategies)
