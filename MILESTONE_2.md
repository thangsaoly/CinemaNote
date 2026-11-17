# CinemaNote: Milestone 2 (Rules & Tests)

---

## Overview

Each service holds the app's rules as plain Dart functions. In Milestone 3 the screens only call these functions and show what they return. Widgets contain no rules.

**Project rules:** No state-management packages. No network calls. Everything stays in memory for now. One file holds one class or one enum.

**Paths:** The Flutter project lives in the `cinemanote/` folder. Every path below starts from there, and `flutter test` runs from there.

### Architecture

```
cinemanote/
├── assets/
│   └── catalogue.json
├── lib/
│   ├── main.dart
│   ├── model/
│   │   ├── media_type.dart
│   │   ├── media_title.dart
│   │   ├── episode.dart
│   │   ├── watchlist_entry.dart
│   │   ├── viewing_record.dart
│   │   └── opinion.dart
│   └── service/
│       ├── catalogue_loader.dart
│       ├── watchlist_service.dart
│       └── viewing_service.dart
└── test/
    ├── helpers/
    │   └── fake_catalogue.dart
    └── service/
        ├── catalogue_loader_test.dart
        ├── watchlist_service_test.dart
        └── viewing_service_test.dart
```

- `model/`: the classes for the business domain.  
- `service/`: the business operations and rules.  
- `test/`: the tests, in the same folders as `lib/`.

---

## Deliverables

| \# | File / Folder | Owner | Description |
| :---- | :---- | :---- | :---- |
| 1 | `lib/model/media_type.dart` | Together | `MediaType` enum (`movie`, `tv`) |
| 2 | `lib/model/media_title.dart` | Together | Final model with all agreed fields |
| 3 | `lib/model/episode.dart` | Together | Episode model |
| 4 | `lib/model/watchlist_entry.dart` | Together | WatchlistEntry model |
| 5 | `lib/model/viewing_record.dart` | Together | ViewingRecord with `watchedAt` and `updatedAt` |
| 6 | `lib/model/opinion.dart` | Together | Opinion model (rating and review) |
| 7 | `lib/service/catalogue_loader.dart` | Sokthearith | Loads the JSON catalogue on app start |
| 8 | `assets/catalogue.json` | Sokthearith | At least 12 movies and 3 TV shows (with episodes) |
| 9 | `lib/service/watchlist_service.dart` | Sokthearith | Add, remove, list, search |
| 10 | `lib/service/viewing_service.dart` | Saoly | Mark watched, tick episodes, opinion, diary |
| 11 | `test/service/catalogue_loader_test.dart` | Sokthearith | Unit tests for the catalogue loader |
| 12 | `test/service/watchlist_service_test.dart` | Sokthearith | Unit tests for every watchlist rule |
| 13 | `test/service/viewing_service_test.dart` | Saoly | Unit tests for every viewing rule |
| 14 | `test/helpers/fake_catalogue.dart` | Together | Shared fake catalogue for the service tests |

> `test/widget_test.dart` is Flutter's default counter test. It fails as soon as `main.dart` changes. Delete it (or replace it) so `flutter test` can reach 0 failures.

---

## Models (Written Together: Week 6, Day 1\)

We agree on these before we split up. We write them on `feature/models`, merge it into `main`, and then each of us branches from there. Each class and the enum gets its own file.

### Relationships

```
MediaTitle
├── MediaType          (enum)
└── List<Episode>      composition

WatchlistEntry
└── MediaTitle         aggregation

ViewingRecord
├── MediaTitle         aggregation
├── Set<Episode>       aggregation
└── Opinion?           composition
```

- **MediaTitle and Episode (composition):** an episode is created with its show and can't exist without it.  
- **ViewingRecord and Opinion (composition):** an opinion belongs to one record. Deleting the record deletes the opinion (V10).  
- **WatchlistEntry and MediaTitle (aggregation):** the entry points to a catalogue title. Removing the entry doesn't delete the title.  
- **ViewingRecord and MediaTitle (aggregation):** same idea. The title lives in the catalogue.  
- **ViewingRecord and Episode (aggregation):** the record points to episodes the show owns. Unticking removes the link, not the episode.

### `MediaType`

```
enum MediaType { movie, tv }
```

The type is an enum, not a `String`. Writing `MediaType.Movie` or `MediaType.tvv` gives a compile error, so a typo can't reach the app. Raw strings only exist in the JSON file, and the loader rejects a bad one (S10).

### `MediaTitle`

```
class MediaTitle {
  final String id;              // unique ID, e.g. "tt0068646"
  final String title;
  final MediaType type;
  final int year;
  final String synopsis;
  final String? posterAsset;    // null means show the placeholder
  final List<Episode> episodes; // composition; empty for movies

  MediaTitle({
    required this.id,
    required this.title,
    required this.type,
    required this.year,
    required this.synopsis,
    this.posterAsset,
    List<Episode> episodes = const [],
  }) : episodes = List.unmodifiable(episodes);

  bool get isMovie => type == MediaType.movie;
  int get episodeCount => episodes.length;

  // The episode with this ID, or null if it is not part of this show.
  Episode? findEpisode(String episodeId) {
    for (final episode in episodes) {
      if (episode.id == episodeId) return episode;
    }
    return null;
  }
}
```

### `Episode`

```
class Episode {
  final String id;         // e.g. "tt0944947-s1e1"
  final int season;
  final int episodeNumber;
  final String title;

  const Episode({
    required this.id,
    required this.season,
    required this.episodeNumber,
    required this.title,
  });

  // Same id means same episode, so Set<Episode> never holds a duplicate.
  @override
  bool operator ==(Object other) => other is Episode && other.id == id;

  @override
  int get hashCode => id.hashCode;
}
```

We removed `showId`. The show owns its episodes, so `show.findEpisode(id)` tells us if an episode belongs to that show.

### `WatchlistEntry`

```
class WatchlistEntry {
  final MediaTitle title;  // aggregation
  final DateTime addedAt;

  WatchlistEntry({required this.title, required this.addedAt});
}
```

### `Opinion`

```
class Opinion {
  final int? rating;       // 1 to 5, null means no rating
  final String? review;    // null, or trimmed non-empty text of max 500 chars

  const Opinion({this.rating, this.review});

  bool get isEmpty => rating == null && review == null;
}
```

### `ViewingRecord`

```
class ViewingRecord {
  final MediaTitle title;              // aggregation
  final DateTime watchedAt;            // set when first marked or first episode ticked
  DateTime updatedAt;                  // updated on every change (opinion, episode tick)
  final Set<Episode> watchedEpisodes;  // aggregation; a Set, so ticking twice adds nothing
  Opinion? opinion;                    // composition; null means no opinion

  ViewingRecord({
    required this.title,
    required this.watchedAt,
    Set<Episode>? watchedEpisodes,
    this.opinion,
  })  : updatedAt = watchedAt,
        watchedEpisodes = watchedEpisodes ?? <Episode>{};

  int get watchedCount => watchedEpisodes.length;
}
```

> The Diary shows `updatedAt` as "Updated \[date\]". Refresh it every time `watchedEpisodes` or `opinion` changes. An action that changes nothing (like ticking an episode twice) does not refresh it.

---

## Sokthearith's Work: `WatchlistService` and Catalogue

### Rules to implement

| \# | Rule | Test cases needed |
| :---- | :---- | :---- |
| S1 | **Catalogue search:** matches the title name only, ignores case, trims surrounding spaces, blank shows all, can filter by `MediaType.movie` or `MediaType.tv` | normal, blank, whitespace-only, no match, filter movie, filter tv |
| S2 | **Add to watchlist:** refuses an unknown title ID | unknown ID returns an error message |
| S3 | **Add to watchlist:** refuses a duplicate (already in the list) | add twice: the second add returns an error, and the list still has one entry |
| S4 | **Add to watchlist:** refuses a completed title | calls `isCompleted(id)` before adding; a completed ID returns an error |
| S5 | **Remove from watchlist:** removing a title that isn't on the list does nothing and gives no error | remove unknown: no error, list unchanged |
| S6 | **List watchlist:** ordered by `addedAt`, newest first | three entries, check order |
| S7 | **Filter watchlist:** `MediaType.movie`, `MediaType.tv`, or `null` for all | each filter, empty result, no filter |
| S8 | **Catalogue load:** loads the JSON file at startup and it is not empty | at least 1 title, all IDs unique |
| S9 | **Artwork linking:** at least one title has `posterAsset = null` (for the placeholder) | one null poster exists |
| S10 | **Catalogue type check:** `"type"` in the JSON must be exactly `"movie"` or `"tv"` | `"movie"` works; `"Movie"`, `"tvv"` and a missing type each return an error |

S8, S9 and S10 test the loader, not `WatchlistService`. S8 and S9 are the only tests that read the real JSON. S10 uses small JSON strings written in the test. The loader is a class with `parse(String json)` (plain Dart, easy to test) and `loadFromAssets()` (reads the asset file).

### Function signatures

```
class CatalogueLoader {
  static List<MediaTitle> parse(String json);        // throws FormatException on bad data
  static Future<List<MediaTitle>> loadFromAssets();
}
```

```
class WatchlistService {
  WatchlistService(List<MediaTitle> catalogue, {DateTime Function() now = DateTime.now});
  bool Function(String titleId) isCompleted = (_) => false;  // set to ViewingService.isCompleted

  // Catalogue
  List<MediaTitle> search(String query, {MediaType? type});  // null = all

  // Watchlist
  String? add(String titleId);     // returns null on success, error message on failure
  void remove(String titleId);
  List<WatchlistEntry> list({MediaType? type});              // null = all
}
```

The filter is `MediaType?` instead of a `String`. `null` means all, so there is no `'all'` string to misspell.

---

## Saoly's Work: `ViewingService`

### Rules to implement

| \# | Rule | Test cases needed |
| :---- | :---- | :---- |
| V1 | **Mark movie watched:** one record per title only; refuses TV or unknown IDs | movie works; twice, a TV ID and an unknown ID each return an error |
| V2 | **Mark movie watched:** sets `watchedAt` to now and calls `WatchlistService.remove` | watchedAt is set; the entry is gone from the watchlist |
| V3 | **Tick episode:** refuses an episode from another show | episode of another show returns an error |
| V4 | **Tick episode:** refuses an unknown episode ID | unknown ID returns an error |
| V5 | **Tick episode (idempotent):** ticking a ticked episode does nothing | tick twice: no change, no error |
| V6 | **Untick episode (idempotent):** unticking an unticked episode does nothing | untick an unticked episode: no change, no error |
| V7 | **Progress:** `watchedCount / episodeCount`; a show with zero episodes has 0% and is never completed | 0 episodes, partial, all ticked |
| V8 | **Completing the last episode:** calls `WatchlistService.remove` | last tick removes the watchlist entry |
| V9 | **Unticking after the show is completed:** does NOT add it back to the watchlist | complete, then untick one: the watchlist stays empty |
| V10 | **Unticking the only watched episode:** deletes the ViewingRecord and its Opinion. The screen shows the "Are you sure?" dialog first; the service just deletes | 1 episode watched, untick it: the record is gone |
| V11 | **Save opinion (rating):** 1 to 5 or null; out of range changes nothing | 0 and 6 return an error; null clears; 3 is saved |
| V12 | **Save opinion (review):** trimmed, max 500 chars; only spaces means no review | "  hi  " is saved as "hi"; 500 chars is saved; 501 chars returns an error; only spaces gives null |
| V13 | **Save opinion only for watched titles:** an unknown or unwatched ID changes nothing | unwatched ID and unknown ID each return an error |
| V14 | **Save opinion (bad input changes nothing)** | invalid rating with a valid review: neither is saved |
| V15 | **Clear opinion:** sets `opinion` to null but keeps progress | clear: opinion is null, episodes stay |
| V16 | **Diary:** returns every ViewingRecord, newest `updatedAt` first | three records, check order |

**Movie completion:** a movie counts as completed once `markMovieWatched` succeeds. A watched movie can't be marked unwatched, so `isCompleted` stays `true` for it.

### Function signatures

```
class ViewingService {
  ViewingService(List<MediaTitle> catalogue, WatchlistService watchlist,
      {DateTime Function() now = DateTime.now});

  // Movie
  String? markMovieWatched(String titleId);  // null = ok, string = error

  // TV episodes
  String? tickEpisode(String showId, String episodeId);
  String? untickEpisode(String showId, String episodeId);
  double progress(String showId);            // 0.0 to 1.0
  bool isCompleted(String titleId);          // used by WatchlistService

  // Record
  ViewingRecord? record(String titleId);     // null = not watched

  // Opinion
  String? saveOpinion(String titleId, {int? rating, String? review});
  void clearOpinion(String titleId);

  // Diary
  List<ViewingRecord> diary();
}
```

> `saveOpinion` sets rating **and** review together. A missing argument counts as null, so it clears that field. When the stars on Title Details change only the rating, the screen must pass the current review too (`record(id)?.opinion?.review`). Otherwise the review gets wiped.

---

## Catalogue File: `assets/catalogue.json`

Minimum content:

| Type | Count | Notes |
| :---- | :---- | :---- |
| Movies | At least 12 | Mix of genres |
| TV shows | At least 3 | Each with at least 2 seasons and 3 episodes per season |
| No-artwork title | 1 | `posterAsset: null` to test the placeholder |

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
        { "id": "tt0944947-s1e1", "season": 1, "episodeNumber": 1, "title": "Winter Is Coming" }
      ]
    }
  ]
}
```

Episodes sit inside their show, so they don't need a `showId`.

---

## Test Strategy

We write the tests **while** we write each function (TDD: red, green, refactor).

```
test/
├── helpers/
│   └── fake_catalogue.dart          # Together
└── service/
    ├── catalogue_loader_test.dart   # Sokthearith
    ├── watchlist_service_test.dart  # Sokthearith
    └── viewing_service_test.dart    # Saoly
```

Each test file covers:

1. **Normal cases:** valid input, and the operation succeeds  
2. **Invalid cases:** wrong type, unknown ID, out-of-range value, or a broken business rule  
3. **Boundary cases:** an edge that can change the result (empty list, 500-char review, 0 episodes)

Service tests use the **fake catalogue**, not the real JSON. Only S8 and S9 read the real file.

```
// test/helpers/fake_catalogue.dart
List<MediaTitle> fakeCatalogue() => [
  MediaTitle(id: 'movie-1', title: 'Test Movie', type: MediaType.movie, year: 2020, synopsis: '...'),
  MediaTitle(id: 'movie-2', title: 'Other Movie', type: MediaType.movie, year: 2019, synopsis: '...'),
  MediaTitle(id: 'show-1', title: 'Test Show', type: MediaType.tv, year: 2021, synopsis: '...',
    episodes: const [
      Episode(id: 'show-1-s1e1', season: 1, episodeNumber: 1, title: 'Pilot'),
      Episode(id: 'show-1-s1e2', season: 1, episodeNumber: 2, title: 'Episode 2'),
    ],
  ),
  MediaTitle(id: 'show-2', title: 'Second Show', type: MediaType.tv, year: 2023, synopsis: '...',
    episodes: const [
      Episode(id: 'show-2-s1e1', season: 1, episodeNumber: 1, title: 'Start'),
    ],
  ),
  MediaTitle(id: 'show-empty', title: 'Empty Show', type: MediaType.tv, year: 2022, synopsis: '...'),
];
```

`movie-2` gives the diary test (V16) three records. `show-2` gives V3 an episode from another show.

**Ordering tests (S6, V16):** three calls in a row can get the same `DateTime.now()`, and then the order test fails at random. So each service takes an optional clock, `DateTime Function() now = DateTime.now`, and the tests pass in fixed times.

---

## Git Workflow for This Milestone

```
main
 ├─ feature/models              (together, merge first)
 ├─ feature/watchlist-service   (Sokthearith, branches from main after models merge)
 └─ feature/viewing-service     (Saoly, branches from main after models merge)
```

1. Finish `feature/models` together and merge it  
2. Each person makes their own service branch  
3. Open a PR; the other person reviews and merges it  
4. Final check: `flutter test` on `main`, all green

---

## Definition of Done

- [ ] All 6 model files merged to `main` (one class or enum per file)  
- [ ] Folders are `lib/model/`, `lib/service/` and `test/service/`  
- [ ] `catalogue.json` has at least 12 movies and 3 TV shows, and is listed in `pubspec.yaml`  
- [ ] `WatchlistService` and the catalogue loader cover rules S1 to S10  
- [ ] `ViewingService` covers rules V1 to V16  
- [ ] Default `widget_test.dart` removed or replaced  
- [ ] `flutter test` passes with **0 failures**  
- [ ] At least **one test per rule** (normal, invalid and boundary cases where they apply)  
- [ ] No business logic in any screen or widget file  
- [ ] Each of us has reviewed the other's PR

---

## Checkpoint

> **Before Milestone 3 (Screens):** we run `flutter test` together on a call and check that every test passes. If anything fails, we fix it then. It's much harder to debug once screens are involved.