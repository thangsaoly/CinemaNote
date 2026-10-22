# Offline Movie & TV Diary – Milestone 1 Proposal

Mobile Dev Term 1 – Gen 11 · Course Project · Milestone 1 (Week 5)

Team: Mr. Kheng Sokthearith and Thang Saoly · App name: \[to be chosen\]

## 1. Application Goal and Target Users

### The problem

People who watch movies and TV shows lose track of things. They forget which titles they meant to watch, which episode they stopped at, and what they thought of something once it is over. Existing tools such as Letterboxd or Serializd solve this, but they depend on an internet connection, an account and a social layer that many people do not want for a private watching log.

### Who experiences it

Individual movie and TV enthusiasts who want a private place for their own viewing history. They may be on a weak or expensive mobile connection, or simply prefer an app that keeps their opinions to themselves. In this course, the target user is a single local viewer using a small bundled catalogue.

### How the application helps

The app is a personal, fully offline movie and TV diary. A viewer can find a title in the bundled catalogue, add it to a watchlist, mark a movie as watched or tick off TV episodes one by one, save a short rating and review, and look back over their diary. Movies and TV shows share one catalogue, but TV shows additionally track episode progress.

### Goal statement

> Give a viewer one private, offline place to plan what to watch, track what they have watched and remember what they thought.

### Boundaries of this proposal

This is a course prototype, not a complete entertainment database. The catalogue is deliberately small (about 12 movies and 3 short TV shows) and ships inside the app. There are no accounts, no social features and no network requests. TV progress means completion of the bundled episode list, not every real-world episode of an ongoing show.

## 2. Functional Scope – UML Use Case Diagram

### Actor and system boundary

There is one actor, the **Viewer**, who uses the app locally. There is no administrator and no external system actor, because the app has no backend and calls no online API. The system boundary is the app itself.

### Use cases

| ID | Use case | Notes |
| --- | --- | --- |
| UC-1 | Browse catalogue | Entry point: list of bundled movies and TV shows |
| UC-2 | Search and filter titles | Extends UC-1; search text and movie/TV filter |
| UC-3 | View title details | Shows synopsis, type and current status |
| UC-4 | Add title to watchlist | Extends UC-3 |
| UC-5 | Remove title from watchlist | Extends UC-3 and is also available from the watchlist |
| UC-6 | Mark movie as watched | Extends UC-3; includes UC-10 |
| UC-7 | Update TV episode progress | Extends UC-3; tick or untick episodes; includes UC-10 when the final episode is checked |
| UC-8 | Save, edit or clear rating and review | Extends UC-3; includes UC-11 |
| UC-9 | View diary | Lists viewing records, newest update first |
| UC-10 | Remove completed title from watchlist | Automatic step, not started by the Viewer directly |
| UC-11 | Validate rating and review | Rating 1–5 or empty; review at most 500 characters |

### Relationships to draw

- The Viewer is associated with UC-1, UC-3, UC-4, UC-5, UC-6, UC-7, UC-8 and UC-9.
- UC-2 extends UC-1. UC-4, UC-5, UC-6, UC-7 and UC-8 extend UC-3.
- UC-6 and UC-7 include UC-10. UC-8 includes UC-11.

The diagram itself should be drawn by hand or in a diagramming tool by the team, using the table above as the source of truth.

## 3. User Stories and Acceptance Criteria

### US-1 – Find a title

As a viewer, I want to search and filter the catalogue, so that I can quickly find a movie or show I have in mind.

- Typing a title shows matching results; matching ignores letter case and surrounding spaces.
- A blank search shows the full catalogue.
- The movie/TV filter shows only titles of the chosen type.
- If nothing matches, the app shows a clear empty-state message instead of a blank screen.
- Everything works with no internet connection, including artwork.

### US-2 – Plan what to watch

As a viewer, I want to keep a watchlist, so that I remember what I intend to watch next.

- Adding a valid title makes it appear in the watchlist exactly once.
- Adding a title that is already on the watchlist is rejected with a clear message and creates no duplicate.
- Adding a title that is already fully watched is rejected.
- Removing a title takes it off the watchlist and does not delete any viewing progress or review.
- Removing a title that is not on the watchlist does nothing and does not cause an error.
- Watchlist order is consistent, by date added.

### US-3 – Log a movie as watched

As a viewer, I want to mark a movie as watched, so that my diary reflects what I have seen.

- Marking a movie watched creates one viewing record for it.
- If the movie was on the watchlist, it is removed automatically as part of the same action.
- Repeating the action does not create a second record.
- A TV show or an unknown title ID cannot be marked as a watched movie; the action is rejected and nothing changes.

### US-4 – Track TV progress

As a viewer, I want to tick off the episodes I have watched, so that I can see how far through a show I am.

- Checking a valid episode updates the progress shown (watched episodes out of total known episodes).
- Unchecking an episode lowers progress; repeating the same check or uncheck is harmless.
- Checking the final known episode marks the show completed and removes it from the watchlist if present.
- Unchecking an episode of a completed show returns it to in progress; it is not added back to the watchlist automatically.
- An episode that does not belong to the selected show, an unknown episode, or a movie ID is rejected and leaves all data unchanged.
- A show with zero known episodes shows 0% progress and is never treated as completed.
- Unchecking the last remaining watched episode asks the user to confirm first, because it also clears the rating and review.

### US-5 – Record my opinion

As a viewer, I want to save a rating and short review, so that I remember what I thought of a title.

- A rating is optional; if given, it must be a whole number from 1 to 5.
- A review is optional, is trimmed of surrounding spaces and may be at most 500 characters (500 is accepted, 501 is rejected).
- An opinion can only be saved for a watched movie or a TV show with at least one watched episode.
- Invalid input shows a validation message and leaves any existing opinion unchanged.
- Cancelling the editor keeps the previously saved values.
- Editing replaces the existing opinion rather than creating a second one; clearing both rating and review removes the opinion but keeps the viewing progress.

### US-6 – Revisit my diary

As a viewer, I want to look back over everything I have watched, so that I can remember and revise my opinions.

- The diary lists every viewing record with its title details, newest update first.
- Titles with no rating or review still appear.
- After editing a record, the diary, the title details and the watchlist all show the latest state when I navigate back and forth.
- An empty diary shows a short explanation of how to add entries.

### US-7 (optional enhancement) – Keep my diary after closing the app

As a viewer, I want my watchlist and diary to survive closing the app, so that I do not lose my history between sessions.

- After a cold restart, the previous watchlist and viewing records are restored.
- On first launch with no saved data, the app starts cleanly with an empty watchlist and diary.
- If saved data is missing, corrupt or refers to unknown titles or episodes, the app does not crash and falls back to a safe state.
- This story is attempted only after the core stories are complete and stable. If it is not delivered, the app is described honestly as session-only.

## 4. Scope and Constraint Check

| Course constraint | How this proposal meets it |
| --- | --- |
| Works 100% offline | Bundled catalogue, artwork and fonts; no API calls, backend or remote images |
| At least 4 meaningful entities | MediaTitle, Episode, WatchlistEntry, ViewingRecord, linked by title and episode IDs |
| At least 2 services with real rules | WatchlistService and ViewingService, with validation and business rules beyond CRUD |
| One operation across multiple entities | Completing a movie or the final episode updates the ViewingRecord and removes the WatchlistEntry |
| Unit tests for every service operation | Normal, edge and invalid cases for each operation, as written in the acceptance criteria above |
| At least 4 distinct screens | Five: Discover, Title details, Watchlist, Diary, Rating/review editor |
| No state-management framework | StatefulWidget, setState, callbacks and navigation only |
| Separation of concerns | Models, services, data handling and UI kept separate; no business logic in screen widgets |

### Deliberately out of scope

Accounts, friends, followers, public reviews, online catalogue APIs, custom lists, extensive statistics, rewatch history, season or episode reviews, notifications and AI recommendations. These are excluded to keep the project realistic for one semester and fully explainable by both team members.

## 5. Open Items to Confirm

- App name and which member leads which area.
- Exact milestone dates and the course-prescribed architecture, to be confirmed with the lecturer.
- Whether the lecturer accepts local persistence as a stretch goal.
