# CinemaNote: Milestone 2 (Rules & Tests)

> [!NOTE]
> **Scope:** Step 2 of the TEAM_PLAN (weeks 6–8). We each work on our own service and test file at the same time. No real screens yet. This milestone is only logic and tests.
> **Due:** End of week 8.
> **Gate:** Every test must pass before anyone starts Step 3 (Screens).

---

## Overview

```
Milestone 2 = Services + Unit Tests
```

Each service holds the app's rules as plain Dart functions. In Milestone 3 the screens only call these functions and show what they return. Widgets contain no rules.

**Project rules:** No state-management packages. No network calls. Everything stays in memory for now.

**Paths:** The Flutter project lives in the `cinemanote/` folder. Every path below starts from there, and `flutter test` runs from there.

---

## Deliverables

| #  | File / Folder                           | Owner       | Description                                      |
| -- | --------------------------------------- | ----------- | ------------------------------------------------ |
| 1  | `lib/models/media_title.dart`         | Together    | Final model with all agreed fields               |
| 2  | `lib/models/episode.dart`             | Together    | Episode model                                    |
| 3  | `lib/models/watchlist_entry.dart`     | Together    | WatchlistEntry model                             |
| 4  | `lib/models/viewing_record.dart`      | Together    | ViewingRecord with `watchedAt` + `updatedAt` |
| 5  | `lib/data/catalogue_loader.dart`      | Sokthearith | Loads the JSON catalogue on app start            |
| 6  | `assets/catalogue.json`               | Sokthearith | ≥12 movies, ≥3 TV shows (with episodes)        |
| 7  | `lib/services/watchlist_service.dart` | Sokthearith | Add, remove, list, search                        |
| 8  | `lib/services/viewing_service.dart`   | Saoly       | Mark watched, tick episodes, opinion, diary      |
| 9  | `test/watchlist_service_test.dart`    | Sokthearith | Unit tests for every watchlist rule              |
| 10 | `test/viewing_service_test.dart`      | Saoly       | Unit tests for every viewing rule                |
| 11 | `test/helpers/fake_catalogue.dart`    | Together    | Shared fake catalogue for both test files        |

> [!WARNING]
> `test/widget_test.dart` is Flutter's default counter test. It fails as soon as `main.dart` changes. Delete it (or replace it) so `flutter test` can reach 0 failures.

---

## Models (Written Together: Week 6, Day 1)

Agree on these before splitting up. Write them on `feature/models`, merge it into `main`, then each of us branches from there.

### `MediaTitle`

```dart
class MediaTitle {
  final String id;        // unique ID, e.g. "tt0068646"
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
  final String id;         // e.g. "tt0944947-s1e1"
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
  Set<String> watchedEpisodeIds;   // Set, so ticking twice adds nothing
  int? rating;                 // 1–5, null = no rating
  String? review;              // null or trimmed non-empty string ≤500 chars
}
```

> [!IMPORTANT]
> The Diary shows `updatedAt` as "Updated [date]". Refresh it every time `watchedEpisodeIds`, `rating`, or `review` changes. An action that changes nothing (like ticking an episode twice) does not refresh it.

---

## Sokthearith's Work: `WatchlistService` + Catalogue

### Rules to implement

| #  | Rule                                                                                                                                                    | Test cases needed                                                            |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| S1 | **Catalogue search:** matches the title name only, ignores case, trims surrounding spaces, blank shows all, can filter by `"movie"` or `"tv"` | normal, blank, whitespace-only, no match, filter movie, filter tv            |
| S2 | **Add to watchlist:** refuses an unknown title ID                                                                                                 | unknown ID → error message                                                  |
| S3 | **Add to watchlist:** refuses a duplicate (already in the list)                                                                                   | add twice → second add returns an error, list still has one entry           |
| S4 | **Add to watchlist:** refuses a completed title                                                                                                   | asks `ViewingService.isCompleted(id)` before adding; completed id → error |
| S5 | **Remove from watchlist:** removing a title that isn't on the list does nothing and gives no error                                                | remove unknown → silent success                                             |
| S6 | **List watchlist:** ordered by `addedAt`, newest first                                                                                          | three entries, check order                                                   |
| S7 | **Filter watchlist:** `"movie"`, `"tv"` or `"all"`                                                                                          | each filter, empty result, no filter                                         |
| S8 | **Catalogue load:** loads the JSON file at startup and it is not empty                                                                            | count ≥ 1, all IDs unique                                                   |
| S9 | **Artwork linking:** at least one title has `posterAsset = null` (for the placeholder)                                                          | one null poster exists                                                       |

S8 and S9 test the loader and the real catalogue, not `WatchlistService`. They are the only tests that read the real JSON. Split the loader into `parseCatalogue(String json)` (plain Dart, easy to test) and the part that reads the asset file.

### Function signatures (agree with Saoly before coding)

```dart
class WatchlistService {
  WatchlistService(List<MediaTitle> catalogue);
  bool Function(String titleId) isCompleted = (_) => false;  // set to ViewingService.isCompleted

  // Catalogue
  List<MediaTitle> search(String query, {String filter = 'all'});

  // Watchlist
  String? add(String titleId);     // returns null on success, error message on failure
  void remove(String titleId);
  List<WatchlistEntry> list({String filter = 'all'});
}
```

---

## Saoly's Work: `ViewingService`

### Rules to implement

| #   | Rule                                                                                                                                                                 | Test cases needed                                                             |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| V1  | **Mark movie watched:** one record per title only; refuses TV or unknown IDs                                                                                   | movie → ok; twice → error; TV id → error; unknown id → error              |
| V2  | **Mark movie watched:** sets `watchedAt` to now and calls `WatchlistService.remove`                                                                        | watchedAt is set; entry is gone from the watchlist                            |
| V3  | **Tick episode:** refuses an episode from another show                                                                                                         | wrong showId → error                                                         |
| V4  | **Tick episode:** refuses an unknown episode ID                                                                                                                | unknown id → error                                                           |
| V5  | **Tick episode (idempotent):** ticking a ticked episode does nothing                                                                                           | tick twice → no change, no error                                             |
| V6  | **Untick episode (idempotent):** unticking an unticked episode does nothing                                                                                    | untick unticked → no change, no error                                        |
| V7  | **Progress:** `watchedCount / totalEpisodes`; zero episodes → 0%, never completed                                                                           | 0 eps, partial, all ticked                                                    |
| V8  | **Completing the last episode:** calls `WatchlistService.remove`                                                                                             | last tick → watchlist entry gone                                             |
| V9  | **Unticking after the show is completed:** does NOT add it back to the watchlist                                                                               | complete → untick one → watchlist still empty                               |
| V10 | **Unticking the only watched episode:** deletes the ViewingRecord and its opinion. The screen shows the "Are you sure?" dialog first; the service just deletes | 1 ep watched → untick → record gone                                         |
| V11 | **Save opinion (rating):** 1–5 or null; out of range → no change                                                                                             | 0 → error; 6 → error; null → clears; 3 → saved                            |
| V12 | **Save opinion (review):** trimmed, max 500 chars; only spaces → no review                                                                                    | "  hi  " → "hi"; 500 chars → saved; 501 chars → error; spaces only → null |
| V13 | **Save opinion only for watched titles:** unknown or unwatched ID → no change                                                                                 | unwatched id → error; unknown id → error                                    |
| V14 | **Save opinion (bad input changes nothing)**                                                                                                                   | invalid rating + valid review → neither saved                                |
| V15 | **Clear opinion:** clearing both rating and review removes the opinion but keeps progress                                                                      | clear → rating null, review null, episodes intact                            |
| V16 | **Diary:** returns every ViewingRecord, newest `updatedAt` first                                                                                             | three records, check order                                                    |

**Movie completion:** a movie counts as completed once `markMovieWatched` succeeds. A watched movie can't be marked unwatched, so `isCompleted` stays `true` for it.

### Function signatures

```dart
class ViewingService {
  ViewingService(List<MediaTitle> catalogue, WatchlistService watchlist);

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

> [!IMPORTANT]
> `saveOpinion` sets rating **and** review together. A missing argument counts as null, so it clears that field. When the stars on Title Details change only the rating, the screen must pass the current review too. Otherwise the review gets wiped.

---

## Catalogue File: `assets/catalogue.json`

Minimum content:

| Type             | Count | Notes                                               |
| ---------------- | ----- | --------------------------------------------------- |
| Movies           | ≥ 12 | Mix of genres                                       |
| TV shows         | ≥ 3  | Each with ≥ 2 seasons and ≥ 3 episodes per season |
| No-artwork title | 1     | `posterAsset: null` to test the placeholder       |

Register the file in `pubspec.yaml` under `flutter: assets:`, or the app can't load it.

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

Write the tests **while** you write each function (TDD: red → green → refactor).

```
test/
├── helpers/
│   └── fake_catalogue.dart       # Together
├── watchlist_service_test.dart   # Sokthearith
└── viewing_service_test.dart     # Saoly
```

Each test file covers:

1. **Happy path:** the normal case
2. **Edge case:** a boundary (empty list, 500-char review, 0 episodes)
3. **Invalid input:** wrong type, unknown ID, out-of-range value

Service tests use the **fake catalogue**, not the real JSON (only S8 and S9 read the real file):

```dart
// test/helpers/fake_catalogue.dart
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

**Ordering tests (S6, V16):** three calls in a row can get the same `DateTime.now()`, so the order test may fail at random. Give each service an optional clock, e.g. `DateTime Function() now = DateTime.now`, and pass fixed times in the test.

---

## Git Workflow for This Milestone

```
main
 ├─ feature/models              ← together, merge first
 ├─ feature/watchlist-service   ← Sokthearith, branches from main after models merge
 └─ feature/viewing-service     ← Saoly, branches from main after models merge
```

1. Finish `feature/models` together and merge it
2. Each person makes their own service branch
3. Open a PR; the other person reviews and merges it
4. Final check: `flutter test` on `main`, all green

---

## Definition of Done

- [ ] All 4 model files merged to `main`
- [ ] `catalogue.json` has ≥ 12 movies and ≥ 3 TV shows, and is listed in `pubspec.yaml`
- [ ] `WatchlistService` and the catalogue loader cover rules S1–S9
- [ ] `ViewingService` covers rules V1–V16
- [ ] Default `widget_test.dart` removed or replaced
- [ ] `flutter test` passes with **0 failures**
- [ ] At least **one test per rule** (normal + edge + invalid where they apply)
- [ ] No business logic in any screen or widget file
- [ ] Each of us has reviewed the other's PR

---

## Checkpoint

> **Before Milestone 3 (Screens):** run `flutter test` together on a call and check that every test passes. If anything fails, fix it now. It's much harder to debug once screens are involved.

---

## Decisions

**How the two services talk to each other.** `WatchlistService.add()` needs `ViewingService.isCompleted()`, and `ViewingService` needs `WatchlistService.remove()` (S4, V2, V8). We pass one service into the other. They depend on each other, so `WatchlistService` gets a function instead of the whole service:

```dart
final watchlist = WatchlistService(catalogue);
final viewing = ViewingService(catalogue, watchlist);
watchlist.isCompleted = viewing.isCompleted;
```

This needs no packages, keeps the rules out of the screens, and lets each test pass in a fake. In `watchlist_service_test.dart`, set `isCompleted` to a fake like `(id) => id == 'movie-1'` to test S4 without `ViewingService`.

---

## Open Questions (ask the lecturer before the PR)

**Q1: Real titles or made-up ones in `catalogue.json`?** TEAM_PLAN lists this as a question for the lecturer (real titles from a source like TMDB). The answer decides the catalogue content and the ID format.

**Q2: Are posters needed for this milestone?** If the Figma artwork isn't exported yet, can every title except the S9 one use a placeholder for now and get its poster later?
