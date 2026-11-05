# CinemaNote — Team Plan

A step-by-step plan for building CinemaNote as a team of two. Tick each box when it's done.

- **Sokthearith**: Catalogue & Watchlist
- **Saoly**: Viewing & Diary
- **Rules for the whole project:** Flutter only, 100% offline, no state-management packages (only StatefulWidget, setState, callbacks and navigation). All rules live in services, not in screens.

---

## Who owns what

|          | Sokthearith                                                                                     | Saoly                                                                            |
| -------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Data     | Catalogue file (about 12 movies, 3 TV shows with episodes), posters                             | –                                                                               |
| Models   | MediaTitle, Episode, WatchlistEntry (written together)                                          | ViewingRecord (written together)                                                 |
| Service  | WatchlistService + catalogue search                                                             | ViewingService                                                                   |
| Screens  | Discover, Title Details (movie and TV versions), Watchlist                                      | Diary, Rating & Review Editor                                                    |
| Widgets  | Poster image with placeholder, empty-state message, catalogue tile, watchlist row, filter chips | Episode checklist, star rating input, diary card, progress bar with status badge |
| Tests    | Search and watchlist tests                                                                      | Movie, TV progress, opinion and diary tests                                      |
| Together | Project setup, models, app navigation, joining features, final testing, presentation            |                                                                                  |

**Where your work meets:**

- When a movie is marked watched, or a show's last episode is ticked, it must leave the watchlist. Saoly's service uses Sokthearith's "remove from watchlist".
- The watchlist must refuse a title that's already completed. Sokthearith's service asks Saoly's whether a title is completed.
- Saoly's episode checklist sits inside Sokthearith's Title Details screen.

Agree on these three things in the first session so neither of you is guessing.

---

## Design (Figma: Afterglow)

The Figma file is the look we follow. The screens map to the plan like this:

| Figma screen                     | Our screen                                                                                          | Owner                                                        |
| -------------------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| 01 Discover                      | Discover                                                                                            | Sokthearith                                                  |
| 02 Movie details + 03 TV details | Title Details (one screen, changes for movie or TV)                                                 | Sokthearith, with Saoly's episode checklist and stars inside |
| 04 Watchlist                     | Watchlist                                                                                           | Sokthearith                                                  |
| 05 Diary                         | Diary                                                                                               | Saoly                                                        |
| 06 Review editor                 | Rating & Review Editor                                                                              | Saoly                                                        |
| Additional states                | Empty lists, no search results, missing artwork, review too long, completed show, clearing a rating | Owner of that screen                                         |

What the design adds to the plan:

- One dark theme: colours, the Inter font and spacing taken from the Foundations page
- Watchlist has All / Movies / TV filter chips (same as Discover)
- Diary cards show "Watched [date]" or "Updated [date]", so a record keeps a **watched date** as well as an updated date
- Stars can be tapped right on the details screen, not only in the editor (same save rule)
- After "Mark as watched", the buttons change to "Watched on [date]" + "Rate or review"
- Review editor: the counter turns red and Save is disabled above 500 characters
- TV progress shows a badge: Watching or Completed, plus "3 of 6 episodes"
- Ignore the "9:41" status bar in the screens; the phone draws its own

⚠️ The Figma Foundations page says it's an **AI-generated concept**. PLAN.md check #31 says designs must be drawn by hand by the team. Ask the lecturer before presenting it, or redraw the screens by hand and use the Figma only as a reference.

---

## Project folders

| Folder   | What goes in it               |
| -------- | ----------------------------- |
| models   | Plain data classes only       |
| data     | Loading the bundled catalogue |
| services | All rules and checks (no UI)  |
| screens  | One file per screen, UI only  |
| widgets  | Small reusable pieces         |
| assets   | Catalogue file, posters, font |
| test     | Service tests                 |

---

## Step 1: Foundation (week 5–6, together)

- [X] Create the Flutter project and the GitHub repo; add each other as collaborators
- [X] Create the folders above
- [ ] Add the Inter font files to the app (don't use a package that downloads fonts)
- [ ] Set up the app theme from the Figma Foundations page: colours, text styles, spacing, corner radius
- [ ] Export the 6 artwork images from Figma as PNGs into the assets folder
- [ ] Agree on the fields for the 4 models and write them (ViewingRecord needs both a watched date and an updated date)
- [ ] Agree on the name of every service function and what it should do, then make empty versions
- [ ] Agree on how errors work: a service refuses with a message, and the screen shows that message
- [ ] Make a small fake catalogue for tests (include one TV show with zero episodes)
- [ ] Ask the lecturer whether the AI-generated Figma is allowed; if not, hand-draw the 5 screens
- [ ] The app runs and shows an empty screen

**Checkpoint:** you both know every function name. From here you can work without waiting on each other.

---

## Step 2: Rules + tests (week 6–8, separately)

Write the tests from the proposal's acceptance criteria while you build each function.

### Sokthearith

- [ ] Write the catalogue file: titles, years, synopses, types, episodes
- [ ] Load the catalogue when the app starts
- [ ] Search: ignore letter case and surrounding spaces; a blank search shows everything; filter by movie or TV
- [ ] Watchlist add: refuse unknown, duplicate or completed titles
- [ ] Watchlist remove: removing a title that isn't there does nothing and gives no error
- [ ] Watchlist list: ordered by date added, can be filtered by movie or TV
- [ ] Link the exported artwork to the catalogue titles; leave one title with no artwork so the placeholder shows
- [ ] Tests for all of the above

### Saoly

- [ ] Mark movie watched: one record only, saves the watched date, removes it from the watchlist, refuses TV or unknown IDs
- [ ] Tick or untick an episode: refuses episodes from another show or unknown episodes; repeating the same action does nothing
- [ ] Progress: watched out of total; zero episodes means 0% and never completed
- [ ] Completing the last episode removes the show from the watchlist; unticking later does NOT add it back
- [ ] Unticking the last watched episode deletes the record and opinion (the screen asks first)
- [ ] Save opinion: rating 1–5 or empty; review trimmed, max 500; only for watched titles; a bad input changes nothing
- [ ] Clearing both rating and review removes the opinion but keeps the progress
- [ ] Diary: newest update first
- [ ] Tests for all of the above (normal, edge and invalid cases)

**Checkpoint:** all tests pass before anyone builds a real screen.

---

## Step 3: Screens (week 8–10, separately)

### Sokthearith

- [ ] App navigation first, with a bottom bar for Discover, Watchlist and Diary (Saoly needs it)
- [ ] Discover (Figma 01): search box with a clear button, All/Movies/TV chips, poster grid, "No matches for …" message
- [ ] Title Details (Figma 02 + 03): artwork, type badge, info, synopsis, watchlist button; movie: Mark as watched → "Watched on [date]" + Rate or review; TV: room for the progress, stars and episode checklist
- [ ] Watchlist (Figma 04): All/Movies/TV chips, rows with a bookmark to remove, "Nothing saved yet" + Browse Discover button
- [ ] Reusable widgets: artwork with "No artwork" placeholder, empty-state message, catalogue tile, watchlist row

### Saoly

- [ ] Star rating widget with a Clear button (tap a star to set, Clear to remove)
- [ ] Progress bar with a Watching / Completed badge and "3 of 6 episodes"
- [ ] Episode checklist widget grouped by season, with an "Are you sure?" dialog for the last untick
- [ ] Rating & Review Editor (Figma 06): title card, stars, text box with a 500 counter; above 500 the counter turns red and Save is disabled; Save / Cancel
- [ ] Diary (Figma 05): cards newest first with "Watched / Updated [date]", stars, review snippet or progress; tap opens Title Details; "Your diary is empty" + Browse Discover button

**Rule for every screen:** call the service, show the result or the error message, and refresh. No checks inside the screen.

**Checkpoint:** each screen works alone with the real services.

---

## Step 4: Join it up (week 10–11, together)

- [ ] Put the progress bar, stars, "Rate or review" button and episode checklist into the TV version of Title Details
- [ ] Compare every screen and every "Additional states" example with the Figma
- [ ] Walk the full journey: find → add to watchlist → watch → rate → see it in the diary
- [ ] Move between all screens and check they always show the same state
- [ ] Turn on airplane mode, reinstall, and test everything again (including posters and font)
- [ ] Fix the bugs you find

---

## Step 5: Saving between sessions (week 12, optional)

Only start this if Steps 1–4 are solid.

- [ ] Ask the lecturer which local storage is allowed
- [ ] Save the watchlist and viewing records whenever they change
- [ ] Restore them on app start
- [ ] First launch starts empty; broken or unknown saved data doesn't crash the app
- [ ] If you skip this step, call the app "session-only" in the presentation

---

## Step 6: Polish and present (week 13–14, together)

- [ ] Same spacing, colours and font on every screen
- [ ] Go through all 32 checks in PLAN.md
- [ ] Update the README and documentation
- [ ] Each person practises explaining the **other person's** service and screens
- [ ] Rehearse the demo in airplane mode

---

## How you work together

- Never push straight to main. Make one branch per task (for example "watchlist-service").
- Each branch becomes a pull request, and **the other person** reviews and merges it.
- Commit small and often, so the Git history shows work from both of you.
- Have a short check-in twice a week: what's done, what's next, what's blocked.

---

## Decisions to confirm with the lecturer

- Search looks at the title name only
- A review that's only spaces counts as no review
- A watched movie can't be marked unwatched
- The rating uses stars from 1 to 5, with a Clear button
- The Figma design is AI-generated: is it allowed, or must designs be hand-drawn?
- The watchlist filter chips are an extra beyond the proposal: OK to add?
- Which local storage is allowed (for Step 5)
- Whether real movie titles from a source like TMDB are OK in the bundled catalogue
