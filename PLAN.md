---
title: Offline Movie and TV Diary - Project Plan
aliases:
  - Movie TV App Plan
tags:
  - project/flutter
  - coursework
status: draft-baseline
version: "0.1"
updated: 2026-10-22
team_size: 2
current_course_week: 4
final_jury_week: 14
---

# Offline Movie & TV Diary — Project Plan

**Version 0.1 — draft baseline, 22 October 2026.** This plan contains only what is decided in *Movie_TV_App_Project_Plan.md*. Anything unclear is listed at the end under *Questions for me*, not guessed.

> [!summary] In one line
> A personal movie-and-TV diary that works 100% offline: find a bundled title → add it to your watchlist → track watching → save a rating and short review → revisit your diary.

## 1. What it is

An **offline personal movie and TV diary** built in Flutter, as a team of two. Movies and TV shows share one bundled catalogue; TV shows also have episode progress.

The core journey is: **find a bundled title → add it to your watchlist → track watching → save a rating and short review → revisit your diary.**

It works with no backend, no online API, no requests and no fetching. Catalogue data, artwork and fonts are all bundled with the app.

## 2. Who uses it

| Who | Signs in? | What they do |
|---|---|---|
| The viewer (one local person, an individual movie or TV enthusiast) | No | Browses the bundled catalogue, plans, tracks and reviews what they watch |

- There is one kind of user. No accounts, no profiles, no administrator and no remote API.
- The catalogue is small and fixed: this is a course prototype, not a complete entertainment database.

## 3. What is in and what is out

### In

| Feature | Smallest useful version | Limit |
|---|---|---|
| Offline catalogue | Browse and search bundled movies and TV shows; filter by type | Approximately 12 movies and 3 short TV shows; no catalogue editor |
| Title details | Title, year, short synopsis, type, local artwork or placeholder, and actions | No cast biographies, trailers or streaming availability |
| Watchlist | Add, remove and view titles intended for later | One built-in watchlist, no custom lists |
| Movie tracking | Mark a movie watched; optionally rate and review it | One viewing record per title; no rewatch history |
| TV tracking | Mark and unmark individual episodes; show progress | Only bundled episodes; no season-level bulk actions |
| Diary | View and edit saved records and reviews | Ordered by most recently updated |
| UI | Consistent spacing, typography, navigation and clear feedback | One theme, restrained animation |

Five screen types: **Discover, Title details, Watchlist, Diary, Rating/review editor.** The episode checklist lives inside Title details.

### Optional, only after the core works

1. Local saving and restoration after an app restart (the first optional enhancement).
2. Small summary counts: watched movies, completed shows, watchlist size.
3. One extra sort option, if it adds little complexity.

### Out

- Accounts, online profiles, friends, followers, public reviews, comments and activity feeds.
- Online catalogue APIs (including TMDB), remote images, cloud databases and sync.
- AI recommendations, notifications, release calendars and streaming integrations.
- Custom lists, extensive statistics, imported Letterboxd histories and multiple local profiles.
- Season reviews, episode reviews, half-star ratings and repeat-watch diaries.
- Multiple themes, complex transitions, camera or GPS features, and store publication.
- State-management frameworks (Provider, Riverpod, Bloc and the like): only StatefulWidget, setState, callbacks and navigation.

## 4. What it keeps

The catalogue is predefined and fixed. Personal records can change.

| Entity | Kept for each | Relationships |
|---|---|---|
| MediaTitle | Stable ID; name; type (movie or TV show); year; synopsis; local artwork reference | Referenced by watchlist entries and viewing records; a TV title owns its episodes |
| Episode | Stable ID; show ID; season number; episode number; name | Belongs to exactly one TV title |
| WatchlistEntry | Title ID; date added | References one MediaTitle |
| ViewingRecord | Title ID; watched flag (movie) or set of watched episode IDs (TV); rating (optional); review (optional); time last updated | References one MediaTitle and, for TV, its Episode IDs |

Rules for what is kept:

- **Rating:** absent, or a whole number from 1 to 5.
- **Review:** optional; trimmed; at most 500 characters.
- A movie has no episode progress; an episode belongs to a TV show.
- TV completion is calculated from the known episodes (watched ÷ total), not stored as a separate flag. Zero episodes means 0%, not completed.
- Watchlist (a plan to watch) and ViewingRecord (actual progress and opinion) are separate.
- One viewing record per title; editing an opinion replaces the same record, never a second copy.
- Missing title IDs and episode IDs are rejected, so no broken relationships are created.
- Without persistence, the app is described as a session-only prototype; saved data is claimed to survive a restart only if persistence is built.

## 5. Who may do what

Only the viewer acts. The rules below decide which actions are allowed.

| Action | Allowed when |
|---|---|
| Browse, search, filter by movie or TV | Always. Search ignores letter case and trims surrounding spaces; a blank search shows everything |
| Add a title to the watchlist | The title exists, is not already on the watchlist, and is not fully completed |
| Remove a title from the watchlist | Always; removing a title that is not on the list changes nothing. Viewing history and reviews are kept |
| Mark a movie watched | The title is a movie. Creates or updates one record; removes the title from the watchlist |
| Mark or unmark an episode watched | The episode belongs to the selected TV show. Repeating the same state changes nothing |
| Save a rating and review | A movie must be watched; a TV show needs at least one watched episode |
| Edit or clear an opinion | A record exists. Clearing both rating and review removes the opinion but keeps the viewing progress |
| View the diary | Always; newest update first |
| Accounts, network requests, remote data | Nobody, never |

Consistency rules:

- Every action is checked before anything changes; a failed action leaves all records unchanged.
- Completing a movie or the final known episode of a show updates the viewing record and removes the watchlist entry.
- Unmarking an episode can return a completed show to in progress; it does not add the show back to the watchlist.
- Unmarking the final watched episode of a show warns the viewer first; if confirmed, it clears the opinion and removes the empty viewing record.

## 6. Done when

One check a tester can do for each decision.

| # | Decision | Check |
|---|---|---|
| 1 | Works 100% offline | In airplane mode, the app launches for the first time and the whole journey works, including artwork and fonts. |
| 2 | No remote data | Every image and font comes from the app bundle, and no request is made to any online service. |
| 3 | Bundled catalogue | Discover lists the bundled movies and TV shows; the movie filter shows only movies and the TV filter shows only TV shows. |
| 4 | Search | A search in different letter case with extra spaces around it finds the same title; a blank search shows all; a search with no match shows an empty-state message. |
| 5 | Title details | Details show title, year, synopsis and type, and a title with missing artwork shows a placeholder. |
| 6 | Add to watchlist | Adding a valid title shows it on the watchlist exactly once. |
| 7 | No duplicates or completed titles | Adding a title already on the watchlist, or a fully completed title, is rejected and the list is unchanged. |
| 8 | Unknown titles rejected | Adding or marking a title ID that does not exist is rejected. |
| 9 | Remove from watchlist | Removing a listed title removes it; removing a title not on the list changes nothing and does not fail. |
| 10 | History survives removal | After removing a title from the watchlist, its progress and review are still in the diary. |
| 11 | Mark movie watched | Marking a movie creates one record; repeating the action creates no second record. |
| 12 | Completion removes from watchlist | Marking a listed movie watched removes it from the watchlist; completing the final known episode of a listed show does the same. |
| 13 | Movies only | A TV show cannot be marked watched as a movie, and a movie cannot get episode progress. |
| 14 | Episode rules | Marking an episode of another show, or an unknown episode, is rejected; marking the same state twice changes nothing. |
| 15 | TV progress | Progress shows 0%, a partial value and 100% as episodes are marked; a show with zero episodes shows 0% and is not completed. |
| 16 | Unmarking an episode | Unmarking an episode of a completed show returns it to in progress and does not add it back to the watchlist. |
| 17 | Unmarking the last watched episode | A warning appears first; cancelling changes nothing; confirming clears the opinion and removes the viewing record. |
| 18 | Opinion needs viewing | Saving an opinion for an unwatched movie, or a TV show with no watched episodes, is rejected. |
| 19 | Rating 1 to 5 | Ratings 1 and 5 are saved; 0 and 6 are rejected; a rating left empty is saved as no rating. |
| 20 | Review up to 500 | A review of exactly 500 characters is saved; one of 501 is rejected; spaces around the review are trimmed. |
| 21 | Failed or cancelled save | An invalid save or a cancel leaves the existing opinion unchanged. |
| 22 | Edit and clear | Editing an opinion updates the same record; clearing both rating and review removes the opinion and keeps the progress. |
| 23 | Diary | Records appear newest update first, records without ratings still appear, editing a record moves it to the top, and an empty diary explains itself. |
| 24 | One source of truth | After going back and forth between screens, Discover, Title details, Watchlist and Diary show the same state. |
| 25 | Five screen types | Discover, Title details, Watchlist, Diary and Rating/review editor can each be reached, and at least four are distinct screens. |
| 26 | Reusable widgets | The code contains at least one reusable StatelessWidget and one reusable StatefulWidget. |
| 27 | No state-management framework | The dependency list has no Provider, Riverpod, Bloc or similar, and state changes use setState, callbacks and navigation. |
| 28 | Logic outside screens | The service tests run with no screen or UI involved. |
| 29 | Two services with tested operations | WatchlistService and ViewingService exist, and every public operation has passing normal, edge and invalid-case tests. |
| 30 | Persistence, only if built | After a restart, records are restored; missing data starts clean; malformed data does not crash startup. If not built, the app is described as session-only. |
| 31 | Manual design | Wireframes and UI designs for the five screens exist and were drawn by hand by the team. |
| 32 | Both members contribute | Each member can explain their own code, and the Git history shows real work from both. |

- [ ] All 32 checks pass.

## Questions for me

Not clear from the Movie & TV plan, so not decided here:

1. **App name:** the plan says "to choose". What is it called?
2. **Sections left out:** your six headings do not include schedule, roles, time budget or milestones, which the Movie & TV plan has. Should I add them?
3. **Persistence:** it is optional and "decided after the core is stable". Should "Done when" treat it as in or out for now?
4. **Sample catalogue:** which exact titles go in? Original sample data, or titles prepared from a source such as TMDB? The plan says that source is not approved until the lecturer is asked.
5. **Search:** does search look at the title only, or also the synopsis?
6. **Undoing a watched movie:** episodes can be unmarked, but nothing says a movie can be marked unwatched again. May it?
7. **Whitespace review:** if a review is only spaces, is it treated as no review, or rejected?
8. **Rating screen details:** is the rating chosen as stars or as numbers 1 to 5?
9. **Architecture and storage:** the lecturer's exact course architecture and permitted local storage approach are not yet known. Should the plan wait for them?