# CinemaNote — Milestone 2: Rules & Tests

> [!NOTE]
> **Scope:** Step 2 of the TEAM_PLAN (weeks 6–8). Both teammates work in parallel on their own service and test files. No real screens yet — this milestone is 100% logic and tests.
> **Gate:** Every test must be green before anyone starts Step 3 (Screens).

---

## Overview

```
Milestone 2 = Services + Unit Tests
```

Each service encodes the app's business rules as plain Dart functions. The screens in Milestone 3 will only call these functions and show whatever they return — no business logic inside widgets.

**Tech constraint (project-wide):** No state-management packages. No network calls. Everything is in-memory for now.

---

## Deliverables

| # | File / Folder | Owner | Description |
|---|--------------|-------|-------------|
| 1 | `lib/models/media_title.dart` | Together | Final model with all agreed fields |
| 2 | `lib/models/episode.dart` | Together | Episode model |
| 3 | `lib/models/watchlist_entry.dart` | Together | WatchlistEntry model |
| 4 | `lib/models/viewing_record.dart` | Together | ViewingRecord with `watchedAt` + `updatedAt` |
| 5 | `lib/data/catalogue_loader.dart` | Sokthearith | Loads the JSON catalogue on app start |
| 6 | `assets/catalogue.json` | Sokthearith | ≥12 movies, ≥3 TV shows (with episodes) |
| 7 | `lib/services/watchlist_service.dart` | Sokthearith | Add, remove, list, search |
| 8 | `lib/services/viewing_service.dart` | Saoly | Mark watched, tick episodes, opinion, diary |
| 9 | `test/watchlist_service_test.dart` | Sokthearith | Unit tests for every watchlist rule |
| 10 | `test/viewing_service_test.dart` | Saoly | Unit tests for every viewing rule |

---

## Models (Written Together — Week 6, Day 1)

Agree on these before splitting. Write them once, commit to `main`, then branch.

### `MediaTitle`

```dart
class MediaTitle {
  final String id;        // unique slug, e.g. "tt0068646"
  final String title;
  final String type;      // "movie" | "tv"
  final int year;
  final String synopsis;
  final String? posterAsset;      // null → show placeholder
  final List<Episode> episodes;   // empty for movies
}
```

### `Episode`

```dart
class Episode {
  final String id;         // e.g. "tt0068646-s1e1"
  final String showId;     // parent MediaTitle.id
  final int season;
  final int episodeNumber;
  final String title;
}
```

### `WatchlistEntry`

```dart
class WatchlistEntry {
  final String titleId;
  final DateTime addedAt;
}
```

### `ViewingRecord`

```dart
class ViewingRecord {
  final String titleId;
  final DateTime watchedAt;    // set when first marked / first episode ticked
  DateTime updatedAt;          // updated on every change (opinion, episode tick)
  Set<String> watchedEpisodeIds;
  int? rating;                 // 1–5, null = no rating
  String? review;              // null or trimmed non-empty string ≤500 chars
}
```

> [!IMPORTANT]
> `updatedAt` is what the Diary shows as "Updated [date]". It must be refreshed every time `watchedEpisodeIds`, `rating`, or `review` changes.

---

## Sokthearith's Work — `WatchlistService`

### Rules to implement

| # | Rule | Test cases needed |
|---|------|-------------------|
| S1 | **Catalogue search** — case-insensitive, trims surrounding spaces, blank shows all, filterable by `"movie"` or `"tv"` | normal, blank, whitespace-only, no match, filter movie, filter tv |
| S2 | **Add to watchlist** — refuses unknown title ID | unknown ID → error message |
| S3 | **Add to watchlist** — refuses duplicate (already in list) | add twice → second add ignored/error |
| S4 | **Add to watchlist** — refuses a completed title | call `ViewingService.isCompleted(id)` before adding |
| S5 | **Remove from watchlist** — removing a title not on the list does nothing, returns no error | remove unknown → silent success |
| S6 | **List watchlist** — ordered by `addedAt` descending (newest first) | three entries, check order |
| S7 | **Filter watchlist** — `"movie"` or `"tv"` or `"all"` | each filter, empty result, no filter |
| S8 | **Catalogue load** — loads the JSON file at startup, is not empty | check count ≥ 1, all ids unique |
| S9 | **Artwork linking** — at least one title has `posterAsset = null` (for placeholder test) | check one null poster exists |

### Function signatures (agree with Saoly before coding)

```dart
class WatchlistService {
  // Catalogue
  List<MediaTitle> search(String query, {String filter = 'all'});

  // Watchlist
  String? add(String titleId);     // returns null on success, error message on failure
  void remove(String titleId);
  List<WatchlistEntry> list({String filter = 'all'});
}
```

---

## Saoly's Work — `ViewingService`

### Rules to implement

| # | Rule | Test cases needed |
|---|------|-------------------|
| V1 | **Mark movie watched** — only one record per title; refuses TV or unknown IDs | movie → ok; call twice → error; TV id → error; unknown id → error |
| V2 | **Mark movie watched** — sets `watchedAt` = now, calls `WatchlistService.remove` | check watchedAt set; check entry removed from watchlist |
| V3 | **Tick episode** — refuses episode from wrong show | wrong showId → error |
| V4 | **Tick episode** — refuses unknown episode ID | unknown id → error |
| V5 | **Tick episode (idempotent)** — ticking already-ticked episode does nothing | tick twice → no change, no error |
| V6 | **Untick episode (idempotent)** — unticking an unticked episode does nothing | untick unticked → no change, no error |
| V7 | **Progress** — `watchedCount / totalEpisodes`; zero episodes → 0%, never completed | 0 eps, partial, all ticked |
| V8 | **Completing last episode** — calls `WatchlistService.remove` | last tick → watchlist entry gone |
| V9 | **Unticking last episode after all done** — does NOT re-add to watchlist | untick after complete → watchlist still empty |
| V10 | **Unticking the last watched episode** — deletes the ViewingRecord and opinion | 1 ep watched → untick → record gone |
| V11 | **Save opinion — rating** — 1–5 or null; out of range → no change | 0 → error; 6 → error; null → clears; 3 → saved |
| V12 | **Save opinion — review** — trimmed, max 500 chars; only spaces → treated as no review | "  hi  " → "hi"; 501 chars → error; spaces-only → null |
| V13 | **Save opinion — only for watched titles** — unknown or unwatched ID → no change | unwatched id → error |
| V14 | **Save opinion — bad input changes nothing** | invalid rating + valid review → neither saved |
| V15 | **Clear opinion** — clearing both rating and review removes opinion but keeps progress | clear → rating null, review null, episodes intact |
| V16 | **Diary** — returns all ViewingRecords with newest `updatedAt` first | three records, check order |

### Function signatures

```dart
class ViewingService {
  // Movie
  String? markMovieWatched(String titleId);  // null = ok, string = error

  // TV episodes
  String? tickEpisode(String showId, String episodeId);
  String? untickEpisode(String showId, String episodeId);
  double progress(String showId);            // 0.0–1.0
  bool isCompleted(String titleId);          // used by WatchlistService

  // Opinion
  String? saveOpinion(String titleId, {int? rating, String? review});
  void clearOpinion(String titleId);

  // Diary
  List<ViewingRecord> diary();
}
```

---

## Catalogue File — `assets/catalogue.json`

Minimum content:

| Type | Count | Notes |
|------|-------|-------|
| Movies | ≥ 12 | Mix of genres |
| TV shows | ≥ 3 | Each with ≥ 2 seasons and ≥ 3 episodes per season |
| No-artwork title | 1 | `posterAsset: null` to test the placeholder |

Recommended structure:

```json
{
  "titles": [
    {
      "id": "tt0068646",
      "title": "The Godfather",
      "type": "movie",
      "year": 1972,
      "synopsis": "...",
      "posterAsset": "assets/posters/godfather.png",
      "episodes": []
    },
    {
      "id": "tt0944947",
      "title": "Game of Thrones",
      "type": "tv",
      "year": 2011,
      "synopsis": "...",
      "posterAsset": "assets/posters/got.png",
      "episodes": [
        { "id": "tt0944947-s1e1", "showId": "tt0944947", "season": 1, "episodeNumber": 1, "title": "Winter Is Coming" }
      ]
    }
  ]
}
```

---

## Test Strategy

Write tests **while** you write each function (TDD — red → green → refactor).

```
test/
├── watchlist_service_test.dart   # Sokthearith
└── viewing_service_test.dart     # Saoly
```

Each test file must cover:
1. **Happy path** — the normal expected case
2. **Edge case** — boundary (empty list, 500-char review, 0 episodes)
3. **Invalid input** — wrong type, unknown ID, out-of-range value

Use a **fake catalogue** in every test (don't load the real JSON):

```dart
// In test helper
List<MediaTitle> fakeCatalogue() => [
  MediaTitle(id: 'movie-1', title: 'Test Movie', type: 'movie', year: 2020, ...),
  MediaTitle(id: 'show-1',  title: 'Test Show',  type: 'tv',    year: 2021,
    episodes: [
      Episode(id: 'show-1-s1e1', showId: 'show-1', season: 1, episodeNumber: 1, title: 'Pilot'),
      Episode(id: 'show-1-s1e2', showId: 'show-1', season: 1, episodeNumber: 2, title: 'Episode 2'),
    ]
  ),
  MediaTitle(id: 'show-empty', title: 'Empty Show', type: 'tv', year: 2022, episodes: []),
];
```

---

## Git Workflow for This Milestone

```
main
 ├─ feature/models              ← together, merge first
 ├─ feature/watchlist-service   ← Sokthearith, branches from main after models merge
 └─ feature/viewing-service     ← Saoly, branches from main after models merge
```

1. Work together to finalize and merge `feature/models`
2. Each person makes their own service branch
3. Open a PR, the other person reviews and merges
4. Final check: `flutter test` on `main` — all green

---

## Definition of Done

- [ ] All 4 model files committed and merged to `main`
- [ ] `catalogue.json` has ≥ 12 movies and ≥ 3 TV shows
- [ ] `WatchlistService` implements all 9 rules above
- [ ] `ViewingService` implements all 16 rules above
- [ ] `flutter test` passes with **0 failures**
- [ ] At least **one test per rule** (normal + edge + invalid where applicable)
- [ ] No business logic inside any screen or widget file
- [ ] Both teammates have reviewed each other's PR

---

## Checkpoint

> **Before moving to Milestone 3 (Screens):** run `flutter test` together on a call and confirm every test is green. If anything fails, fix it now — it's much harder to debug once screens are involved.

---

---

# ❓ Decisions & Open Questions for You

These are things I couldn't fill in from the existing docs — please answer them so the milestone is complete.

---

### 🔴 Must Answer Before Coding

**Q1 — Submission deadline for Milestone 2?**
When is this due? I need a date to frame the week-by-week schedule accurately.

**Q2 — What week are you on right now?**
The plan says Step 2 is weeks 6–8. Are you on track, behind, or ahead? This affects what's realistic.

**Q3 — How do the two services share state?**
Right now `WatchlistService.add()` needs to call `ViewingService.isCompleted()`, and `ViewingService.markMovieWatched()` needs to call `WatchlistService.remove()`. How do they talk to each other?

> Options:
> - **A)** Pass each service into the other's constructor (dependency injection)
> - **B)** One `AppState` object that both services read from
> - **C)** The screen wires the calls: `viewingService.markWatched()` then `watchlistService.remove()`
>
> _Option C is simplest and fits the "no packages" rule — but it means the screen does slightly more logic. What does the lecturer expect?_

**Q4 — Error return type: `String?` or a custom `Result` type?**
I wrote `String? add(...)` above (null = success, non-null = error message). Is that the approach agreed with the lecturer, or should it be an enum / exception / bool?

---

### 🟡 Should Confirm Before the PR

**Q5 — Catalogue content: real titles or made-up ones?**
The TEAM_PLAN flags this as a question for the lecturer (_"whether real movie titles from TMDB are OK"_). What was the answer? This determines what goes into `catalogue.json`.

**Q6 — Are posters required for Milestone 2, or can they be placeholders?**
If Figma artwork export isn't done yet, can `posterAsset` be null for all titles in this milestone and filled in later?

**Q7 — Fake catalogue in tests: shared helper or one per test file?**
Should there be a `test/helpers/fake_catalogue.dart` file both Sokthearith and Saoly import, or does each test file define its own minimal data?

---

### 🟢 Nice to Clarify

**Q8 — `isCompleted` for a movie: what counts?**
A movie is completed when it's been marked watched. A TV show is completed when all episodes are ticked. Should `isCompleted("movie-id")` return `true` as soon as `markMovieWatched` is called — even if the user later navigates away and comes back?

**Q9 — Unticking the last episode: "Are you sure?" dialog**
The plan says the screen shows a dialog before deleting the record. But the service just deletes. So who controls the dialog — the service (bad) or the screen (good)?
> I assumed the **screen** shows the dialog, then calls the service only if the user confirms. Is that right?

**Q10 — `ViewingService` owns `watchedEpisodeIds` as a `Set` or `List`?**
I used `Set<String>` because ticking twice must be idempotent. Any objection to that?
