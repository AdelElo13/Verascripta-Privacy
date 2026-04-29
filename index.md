# Privacy Policy — Verascripta

**Last updated:** 2026-04-29

Verascripta is a sacred-text search application for iOS. This policy
describes exactly what data Verascripta handles, why, and how we keep it
minimal. Every claim below corresponds to a constant or behaviour that
can be inspected in the project source code (`supabase/functions/*` and
the iOS sources). The internal code-name of the project is "Logos".

## Summary in one sentence

Verascripta works without an account, does not collect personal information,
does not show ads, does not sell data, and stores everything locally on
your device except the queries you actively send to our retrieval and
synthesis backend.

## What Verascripta does NOT collect

- We do not collect your name, email address, phone number, or any other
  personally identifying information.
- We do not require an account to use the app.
- We do not link queries to your Apple ID, device serial number, IDFA,
  IDFV, or any other persistent device identifier.
- We do not use third-party advertising SDKs, attribution networks, or
  third-party analytics SDKs.
- We do not sell, share, or monetise data with third parties.

## Anonymous query identifier

Verascripta generates a **random UUID** the first time you open the app and stores
it in your iOS Keychain. This identifier is sent with every query so we can:

- Enforce the monthly free quota for AI synthesis (**10 queries / month**,
  matching `MONTHLY_QUOTA = 10` in every Edge Function and
  `QuotaState.monthlyTotal = 10` in the iOS client).
- Apply per-endpoint rate limits to prevent abuse:
  - Search: 60 requests / minute
  - Ask, Reply, Chat: 8 requests / minute
  - Synthesis: 5 requests / minute
- Distinguish "you" from "another anonymous user" *for quota accounting only*.

You can reset this UUID at any time in **Filters → Reset anonymous ID**.
Doing so cannot be linked back to your previous activity — the new UUID is
unconnected to the old one. Server-side log rows tied to your old UUID
remain until automatic monthly expiry (see "Where data lives") but cannot
be re-associated with you.

## What is sent to our backend

When you use Search, Ask, Reply, or Chat, the following is transmitted to
our Supabase-hosted backend:

- The text of your query or argument (used for embedding + retrieval, see
  below for retention).
- (Optional) A screenshot you explicitly attach in Reply mode.
- Your filter selections (religion, work, language).
- Your anonymous UUID (header `x-anonymous-id`).
- Your selected output language.

### What we store about queries

We do **not** persist the raw text of your queries. The `query_logs` table
stores only:

- A SHA-256 **hash** of `query + filters` (used for cache lookup and dedup;
  not reversible to the original query).
- Your anonymous UUID (for quota / rate-limit attribution).
- Filter selections (jsonb).
- Latency metrics (embed, search, total).
- The top-result passage ID and similarity score (used to tune retrieval).
- Error codes if a request failed.

The query text itself is held in memory only for the duration of the
request and is never written to disk.

The `query_cache` table stores the **result** of a query keyed by the same
hash for one hour; this is a public corpus retrieval cache and contains no
user-identifying data.

### Screenshots in Reply

If you attach a screenshot, the image is base64-encoded and sent inline to
Anthropic's Claude vision endpoint as part of the same request. We do not
write screenshots to our database, object storage, or any persistent
location. The image lives in memory for the lifetime of the request and is
discarded.

## Where data lives

| Data | Location | Retention |
|---|---|---|
| Anonymous UUID | iOS Keychain on your device | Until you tap "Reset anonymous ID" or uninstall |
| Bookmarks | iOS UserDefaults on your device | Until you delete or uninstall |
| Onboarding preferences | iOS UserDefaults on your device | Until reset or uninstall |
| Recent Ask questions | iOS UserDefaults on your device | Last 8 entries, until you tap "Clear history" |
| Quota counters | Supabase `synth_quotas` table, keyed by UUID + month | Resets each calendar month |
| Rate-limit counters | Supabase `rate_limits` table | Garbage-collected after 5 minutes |
| Query log rows | Supabase `query_logs` table | Retained for retrieval-quality analysis; no raw query text |
| Query result cache | Supabase `query_cache` table | 1 hour TTL (`CACHE_TTL_SECONDS = 3600`) |
| Embedding requests | Modal-hosted BGE-M3 endpoint | Not persisted by Modal beyond request lifetime |
| AI synthesis requests | Anthropic Claude API | Not persisted per Anthropic API policy |

## Search architecture (transparency)

Search uses **dense semantic retrieval** with the BGE-M3 multilingual
embedding model and an HNSW index over a pgvector column. The RPC name
`search_hybrid` is historical: in MVP, sparse and RRF fusion are accepted
as parameters but not yet active (`sparse_score` is always NULL). Real
hybrid retrieval is planned for v1.5 once pgvector ≥ 0.8 sparsevec is
enabled. We do not advertise hybrid search anywhere in the app or App
Store listing until that change ships.

## Third-party processors

- **Supabase** (hosting + database, Frankfurt EU) — backend infrastructure
- **Anthropic Claude API** — AI synthesis. Anthropic does not train on
  inputs sent via the API ([API privacy](https://www.anthropic.com/legal/privacy)).
- **Modal Labs** — hosts our multilingual embedding model (BGE-M3)
- **Stripe** (only when you tap "Support Verascripta") — handles donations
  outside the app via Safari. We never see your payment details.

## Children

Verascripta does not target users under 13. The app is rated 12+ in the App Store
because it surfaces canonical religious texts which contain mature themes
(violence, sexuality, theological dispute) presented in their original
form.

## Your controls

- **Reset anonymous ID** — Filters → Reset anonymous ID
- **Clear bookmarks** — Open Bookmarks → trash any entry
- **Re-run onboarding** — Filters → Re-run onboarding
- **Reset filters** — Filters → Reset filters

Uninstalling the app removes all locally stored data immediately and
permanently. Server-side quota records expire automatically when the
calendar month rolls over; rate-limit records are garbage-collected after
five minutes; cached search results expire after one hour.

## Contact

Questions or data requests:
- GitHub issue: <https://github.com/AdelElo13/Logos/issues> (if public)
- Email: beatboymfkr@gmail.com

If you are a citizen of the EU/EEA, UK, or California, you have rights
under GDPR / UK-GDPR / CCPA respectively. Because Verascripta collects no
personally identifying information, most rights are satisfied by uninstalling
the app and resetting your anonymous UUID. For any other request, email us
above and we will respond within 30 days.

## Changes

We will update the "Last updated" date when this policy changes. Material
changes will surface in-app on next launch.
