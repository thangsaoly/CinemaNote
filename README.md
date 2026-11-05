# CinemaNote

A personal movie and TV diary that works **100% offline**.

Find a title in the bundled catalogue, add it to your watchlist, mark movies as watched or tick off TV episodes, save a rating and short review, and look back over your diary. No account, no internet and no social features.

## Features

- **Discover:** search the bundled catalogue and filter by movie or TV
- **Title details:** synopsis, type, year and your current status
- **Watchlist:** keep track of what you want to watch next
- **Movie tracking:** mark a movie as watched
- **TV tracking:** tick off episodes one by one and see your progress
- **Rating and review:** optional 1–5 stars and a review of up to 500 characters
- **Diary:** everything you've watched, newest first

## Team

Mobile Dev Term 1 · Gen 11 · Course Project

| Member            | Role                                                                              |
| ----------------- | --------------------------------------------------------------------------------- |
| Kheng Sokthearith | Catalogue & Watchlist: catalogue data, Discover, Title Details, Watchlist         |
| Thang Saoly       | Viewing & Diary: viewing records, episode progress, Diary, Rating & Review Editor |

## Getting started

Requirements: Flutter SDK and an Android or iOS device or emulator.

```bash
flutter pub get    # install dependencies
flutter run        # run the app
flutter test       # run the unit tests
```

## Project structure

| Folder           | Contents                                                         |
| ---------------- | ---------------------------------------------------------------- |
| `lib/models`   | Data classes: MediaTitle, Episode, WatchlistEntry, ViewingRecord |
| `lib/data`     | Loads the bundled catalogue                                      |
| `lib/services` | App rules: WatchlistService, ViewingService                      |
| `lib/screens`  | One file per screen                                              |
| `lib/widgets`  | Reusable widgets                                                 |
| `assets`       | Catalogue data, artwork and fonts                                |
| `test`         | Unit tests for the services                                      |

State is handled with `StatefulWidget`, `setState`, callbacks and navigation only, with no state-management packages.

## Screenshots

_Coming soon._

## Documents

- [TEAM_PLAN.md](TEAM_PLAN.md): who does what, step by step
