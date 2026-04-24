# Implementation Plan

- [x] 1. Clean up webapp api-services.ts
  - Remove the `NewsApiService` class (hardcoded `NEWS_API_KEY`, direct `newsapi.org` calls)
  - Remove the `NewsApiData` interface
  - Remove the `transformNewsData` utility function
  - Keep `CryptoApiService`, `CryptoApiData`, and `transformCryptoData` untouched
  - _Requirements: 8.1, 8.2, 8.3_

- [ ]* 1.1 Write property test: no hardcoded API key in webapp source
  - **Property 1: No hardcoded API keys in webapp source**
  - Assert `apps/webapp/lib/api-services.ts` does not contain `NewsApiService`, `NEWS_API_KEY`, or `newsapi.org`
  - **Validates: Requirements 1.3, 1.5, 8.1, 8.2**

- [x] 2. Add BACKEND_API_URL environment variable support to webapp
  - Create `apps/webapp/.env.local.example` documenting `BACKEND_API_URL=http://localhost:3001`
  - _Requirements: 9.1, 9.2, 9.4_

- [x] 3. Create Next.js API route as backend proxy
  - Create `apps/webapp/app/api/news/route.ts`
  - Read `BACKEND_API_URL` from `process.env` (server-side only), defaulting to `http://localhost:3001`
  - Forward query params `limit`, `lang`, `tag`, `category` to the NestJS backend `GET /news`
  - Return the backend JSON response directly, or a 502 on failure
  - Set a 10-second request timeout
  - _Requirements: 1.1, 1.2, 1.4, 4.1, 4.3, 7.4, 9.1, 9.2_

- [x] 4. Create webapp news client service
  - Create `apps/webapp/lib/news-client.ts`
  - Export `fetchCryptoNews(limit?: number): Promise<NewsData[]>` that calls `/api/news`
  - Implement `transformBackendArticle(article, index)` to map `NewsArticleDto` → `NewsData`
    - `excerpt`: first 200 chars of `body`, or `subtitle`, or `"No description available"`
    - `category`: first entry from `categories[]`, or `"Crypto"`
    - `author`: from `authors` field, or `source`
    - `date`: formatted from `publishedAt`
    - `imageUrl`: `imageUrl` or picsum fallback
    - `sentiment`: map `"POSITIVE"/"BULLISH"` → `"Bullish"`, `"NEGATIVE"/"BEARISH"` → `"Bearish"`, else `"Neutral"`
    - `fundingStatus`: random assignment as before
    - `timestamp`: `Date.parse(publishedAt)`
  - _Requirements: 2.2, 4.2, 4.4, 8.4_

- [ ]* 4.1 Write property test: article transformation completeness
  - **Property 2: Article transformation completeness**
  - Use `fast-check` to generate arbitrary `NewsArticleDto` objects
  - Assert `transformBackendArticle` always produces a `NewsData` with non-empty `title`, `url`, `author`, `date`, `imageUrl`, `category`, and non-zero `timestamp`
  - **Validates: Requirements 2.2, 4.4**

- [x] 5. Update NewsSection component to use new client service
  - In `apps/webapp/components/news-section.tsx`:
    - Remove import of `NewsApiService` and `transformNewsData` from `@/lib/api-services`
    - Import `fetchCryptoNews` from `@/lib/news-client`
    - Replace `NewsApiService.getCryptoNews(20)` call with `fetchCryptoNews(20)`
    - Remove the inline `transformNewsData` mapping block (now handled inside `fetchCryptoNews`)
    - Keep all filter, sort, fallback, loading skeleton, and click handler logic unchanged
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5, 8.5_

- [ ]* 5.1 Write property test: fallback on fetch failure
  - **Property 3: Fallback on fetch failure**
  - Mock `fetchCryptoNews` to throw an error
  - Render `NewsSection` and assert `Web3NewsFallback` is shown
  - **Validates: Requirements 2.5, 7.1**

- [ ]* 5.2 Write property test: filter correctness
  - **Property 4: Filter correctness**
  - Use `fast-check` to generate arbitrary `NewsData[]` arrays and filter values
  - Apply the filter logic and assert all results match the selected filter value
  - **Validates: Requirements 3.1, 3.2, 3.3**

- [ ] 6. Checkpoint - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Fix missing imports in web3-news-fallback.tsx
  - The file references `useState` and `useEffect` but does not import them
  - Add `import { useState, useEffect, useMemo } from "react"` to the top of the file
  - _Requirements: 2.5_

- [ ] 8. Final Checkpoint - Ensure all tests pass, ask the user if questions arise.
