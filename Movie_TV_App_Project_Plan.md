---
title: Offline Movie and TV Diary - Project Plan
aliases:
  - Movie TV App Plan
tags:
  - project/flutter
  - coursework
status: finalized-baseline
version: "1.0"
updated: 2026-10-20
team_size: 2
current_course_week: 4
final_jury_week: 14
---

# Offline Movie & TV Diary — Flutter Project Plan

**Version 1.0 — finalized planning baseline, 20 October 2026.** Choose the app name and fill in member assignments together.

Confirmed: team of two; currently course Week 4. Final jury: course Week 14. Exact calendar deadlines and weekly availability still need confirmation.

> [!summary] Team brief
> Two students; Week 4 to Week 14. Build an offline movie-and-TV diary with five screen types, four entities and two core services. No runtime TMDB API or social network features. Local persistence is the first optional enhancement.

> [!important] Immediate deadlines
> Week 5: proposal. Week 6: models, services and tests. Begin domain work now; UI design and widgets are assessed in Week 9.

## Navigation

- [[#1. Recommendation]]
- [[#2. What the assignment actually requires]]
- [[#3. Scope boundaries]]
- [[#4. Proposed domain model — for you to refine and implement]]
- [[#5. Two services with real business rules]]
- [[#6. Screen scope and UX work]]
- [[#7. Architecture and application state]]
- [[#8. Proposal preparation prompts and acceptance checks]]
- [[#9. Milestone-aligned schedule]]
- [[#10. Time budget and scope cuts]]
- [[#11. Definition of done and demonstration]]
- [[#12. Two-person ownership plan]]
- [[#13. Week 4 checklist — start here]]
- [[#14. Remaining decisions]]
- [[#15. Decision register]]
- [[#16. Team tracking]]
- [[#17. Using and sharing this note]]
- [[#Source and evidence boundary]]

## 1. Recommendation

Build a polished **offline personal movie and TV diary**, inspired by the personal tracking aspects of Letterboxd and Serializd. Do not attempt to reproduce their full products.

The core journey is: **find a bundled title → add it to your watchlist → track watching → save a rating and short review → revisit your diary.** Movies and TV share one catalogue, but TV has episode progress.

This is a realistic candidate for roughly ten weeks if you keep the dataset small, avoid accounts and social features, and complete the business logic before spending heavily on visual polish. Feasibility depends on your weekly availability and current Flutter skills; the estimates below are planning assumptions, not guarantees.

## 2. What the assignment actually requires

Source: `PROJECT - Description.pdf`, all five pages, inspected for this plan. Page references below are PDF page numbers. The brief is the authority; recommendations in later sections are proposed choices.

| Requirement | Source | What it means for this app |
|---|---|---|
| Progress, understanding and engineering matter more than feature count | pp. 1–2 | Prioritise a small app you can explain |
| Team of 1–2; each member contributes and explains their code | pp. 1, 3 | Use genuine, ongoing Git commits |
| You must write implementation code yourself; AI must not generate or complete it | p. 3 | Use this document as learning/planning support, not generated implementation |
| Figma wireframes and UI designs must be created manually | p. 3 | Draw your own screens; no AI-generated mockups or interfaces |
| App works 100% offline; no backends, online APIs, requests or fetching | p. 3 | Bundle catalogue and artwork; no TMDB/other catalogue integration, login server or cloud sync |
| Local persistence is optional | p. 4 | Saving after restart is desirable, but not a mandatory technical requirement |
| At least 4 meaningful entities, plain OO classes, relationships | p. 4 | Model titles, episodes, watchlist entries and viewing records |
| At least 2 services; business rules beyond CRUD; one operation involving multiple entities | p. 4 | Separate watchlist rules from viewing/progress rules |
| Test every service operation, with normal, edge and invalid cases | p. 4 | Every new public service operation expands your test workload |
| At least 4 distinct screens; reusable StatelessWidget and StatefulWidget | p. 4 | Plan five screen types and reusable UI pieces |
| No state-management framework | p. 4 | Use StatefulWidget, setState, callbacks and navigation |
| Follow the course architecture; separate UI, models, services and data | p. 4 | Adapt this conceptual structure to the exact architecture taught in class |
| Proposal includes goal/users, UML use cases, user stories and acceptance criteria | p. 5 | Prepare these manually in your own words for Week 5 |

**Important timing:** the brief schedules selection in Week 4 and the final jury in Week 14. That is approximately ten elapsed weeks, not ten unrestricted feature-building weeks. It does not provide calendar dates. You confirmed that you are currently in Week 4, so this schedule starts now.

## 3. Scope boundaries

### Committed core

| Feature | Smallest useful version | Limit |
|---|---|---|
| Offline catalogue | Browse and search bundled movies and TV shows; filter by type | Approximately 12 movies and 3 short TV shows; no catalogue editor |
| Title details | Title, year, short synopsis, type, local artwork/placeholder and actions | No cast biographies, trailers or streaming availability |
| Watchlist | Add, remove and view titles intended for later | One built-in watchlist, no custom lists |
| Movie tracking | Mark a movie watched; optionally rate and review it | One current viewing record per title; no rewatch history |
| TV tracking | Mark/unmark individual episodes; show progress | Only bundled episodes; no season-level bulk actions |
| Diary | View and edit saved records and reviews | Simple ordering by most recently updated |
| UI quality | Consistent spacing, typography, navigation and clear feedback | One theme, restrained animation |

Use short, complete example shows or original sample data. Progress means completion of the **bundled episode list**, not a claim that every real-world episode of an ongoing show has been watched. Label sample/incomplete catalogue coverage clearly.

### Offline data and TMDB decision

**No TMDB API calls in the app.** Downloading once and caching still involves a prohibited network request. Bundle catalogue data, images and fonts locally.

Using TMDB beforehand to prepare bundled data is **not approved by this plan**. The brief permits predefined datasets but does not clarify that sourcing method. Ask the lecturer and separately verify applicable provider usage terms before adopting it. Original sample data and permitted local assets let the project proceed without this dependency.

- [ ] Select the small sample catalogue and document episode coverage.
- [ ] Assign stable IDs and check duplicate IDs and episode relationships.
- [ ] Prepare short descriptions and local artwork or placeholders.
- [ ] Record asset sources and permissions in project documentation.
- [ ] Verify first launch in airplane mode, including artwork and fonts.

### Valuable after the core works

1. Local saving and restoration after app restart. This is the first optional feature to attempt because it makes a diary useful beyond one session.
2. Small summary counts: watched movies, completed shows, watchlist size.
3. One extra sort option if it adds little complexity.

### Excluded from this submission

- Accounts, online profiles, friends, followers, public reviews, comments and activity feeds.
- Online catalogue APIs, remote images, cloud databases or sync: prohibited by the offline rule.
- Recommendations powered by AI, notifications, release calendars and streaming integrations.
- Custom lists, extensive statistics, imported Letterboxd histories and multiple local profiles.
- Season reviews, episode reviews, half-star ratings and repeat-watch diaries.
- Multiple themes, complex transitions, camera/GPS features and store publication work.

Some excluded features could operate offline, but their extra implementation and testing are not worth the schedule risk.

## 4. Proposed domain model — for you to refine and implement

Do not create empty classes just to reach four entities. Each proposed entity has a distinct purpose.

| Entity | Responsibility | Suggested information | Relationships |
|---|---|---|---|
| MediaTitle | Catalogue item representing a movie or TV show | Stable ID, name, type, year, synopsis, local artwork reference | Referenced by watchlist entries and viewing records; TV title owns episode definitions |
| Episode | A watchable unit within a show | Stable ID, show ID, season number, episode number, name | Belongs to exactly one TV title |
| WatchlistEntry | A saved intention to watch a title | Title ID, date added | References one MediaTitle |
| ViewingRecord | Personal viewing progress and optional opinion | Title ID, watched movie flag or watched episode IDs, optional integer rating, review, updated time | References one MediaTitle and, for TV progress, its Episode IDs |

Keep ratings and short reviews inside ViewingRecord initially. A separate Review entity is unnecessary unless you later introduce multiple reviews per title, which is outside this plan.

Key modelling decisions to explain:

- A movie must not have episode progress; an episode must belong to a TV show.
- TV completion is calculated from known episodes, not maintained as an unrelated flag that can become inconsistent.
- A set of watched episode IDs prevents double-counting.
- Watchlist and ViewingRecord are different concepts: planning to watch versus actual progress/opinion.
- Catalogue data is predefined. Personal records are mutable application data.
- Reject missing title IDs and episode IDs rather than silently creating broken relationships.

Have your teacher review whether this model and the selected course architecture meet expectations at Milestone 2.

## 5. Two services with real business rules

These are behavioural specifications, not implementation code. Keep the public operation list small enough to test completely.

### WatchlistService

| Operation | Rule | Essential tests |
|---|---|---|
| Add title | Title must exist; duplicates and fully completed titles are rejected | Valid title; duplicate; unknown title; completed title |
| Remove title | Removes existing entry; removing an absent valid title is a harmless no-op | Existing entry; absent entry; unknown title |
| Retrieve watchlist | Resolve saved IDs to titles; order consistently by date added | Several entries; empty list; equal dates with stable tie-break |

### ViewingService

| Operation | Rule | Essential tests |
|---|---|---|
| Mark movie watched | Only movies; create/update one record; automatically remove from watchlist | Listed/unlisted movie; repeated action; TV or unknown ID rejected |
| Set episode watched/unwatched | Episode must belong to the selected show; repeated state is harmless | First/last episode; repeated action; wrong show; unknown episode; movie rejected |
| Calculate TV progress | Watched known episodes / total known episodes; zero episodes means 0%, not completed | 0, some, all; no episodes; unknown/non-TV title |
| Save opinion | Movie must be watched; TV needs at least one watched episode; rating absent or integer 1–5; review trimmed and at most 500 characters | Valid/boundary ratings; absent rating; out-of-range; whitespace; 500/501 characters; no viewing progress |
| Retrieve diary | Return viewing records with their title details, newest update first | Empty; several records; ties; updates reorder correctly |

Cross-entity operation: completing a movie or the final known TV episode updates ViewingRecord and removes the associated WatchlistEntry, using MediaTitle/Episode to validate the action. This directly demonstrates the multiple-entity requirement.

Consistency policy:

- Validation happens before changes; failed actions leave all records unchanged.
- Unmarking an episode can return a show from completed to in progress; it does not automatically add it back to the watchlist.
- If unmarking the final watched episode would leave zero progress, clear the opinion and remove the empty viewing record. Warn the user before this destructive change.
- Removing a watchlist entry does not delete viewing history or reviews.
- Clearing both rating and review removes the opinion but retains valid viewing progress.
- Editing an opinion replaces the same record, not a second copy.

If you change these policies, change the acceptance criteria and tests together. Every additional service operation must have normal, edge and invalid-case coverage where applicable, as required by the brief.

## 6. Screen scope and UX work

Create your mockups yourself. The descriptions below specify user tasks, not a generated screen layout.

| Screen | User task | Important states |
|---|---|---|
| Discover | Browse/search the bundled catalogue | Default list, filtered results, no matches, missing artwork |
| Title details | Understand a title and manage viewing/watchlist | Movie versus TV, not started, in progress, completed |
| Watchlist | Choose what to watch next | Empty and populated; removal feedback |
| Diary | Revisit personal viewing and opinions | Empty, records without ratings, edited records |
| Rating/review editor | Save or edit an opinion | Optional fields, validation feedback, save/cancel, unsaved changes |

Keep the episode checklist inside TV details for the first version. Five screen types provide margin over the four-screen minimum without creating a large navigation system.

Manual design checklist:

- [ ] Sketch the full core journey before high-fidelity styling.
- [ ] Choose one theme, one accent colour and a small typography scale.
- [ ] Define reusable spacing, buttons, title cards and rating controls.
- [ ] Keep movie and TV terminology clear: watched movie versus episode progress.
- [ ] Show personal rating separately from any catalogue information; do not imply a real community score.
- [ ] Provide readable text, comfortable touch areas and status labels that do not depend on colour alone.
- [ ] Check long titles, long reviews, keyboard overlap, small screens and back navigation.
- [ ] Bundle all images and fonts used; ensure no runtime remote loading.
- [ ] Use your own or appropriately permitted assets and document their sources.

Do not spend weeks making a complex home dashboard. Good UX here means the core journey is predictable and pleasant.

## 7. Architecture and application state

Follow your lecturer's exact course architecture when available. Conceptually separate:

1. **Models:** plain Dart objects and relationships, without Flutter widgets or storage dependencies.
2. **Services:** validation, viewing rules, watchlist rules and progress calculations, testable without UI.
3. **Data handling:** loading bundled catalogue data and, optionally, reading/writing personal records locally.
4. **UI:** screens and reusable widgets; collects input, calls services and displays results.

Use StatefulWidget, setState, callbacks and navigation only for application state. Do not introduce Provider, Riverpod, Bloc or another state-management framework.

Choose one shared owner for the in-memory personal collection at an appropriate parent level. Pass the needed data and actions to screens. After an action, refresh dependent screens from the same source of truth. Do not maintain independent watchlists on several screens.

If adding persistence, keep it behind data handling and use an approach permitted/taught in class. Validate restored IDs, ratings and episode references. A corrupt file must not crash startup or silently overwrite recoverable data. If persistence is omitted, explicitly describe the app as a session-only prototype; do not claim saved data survives restart.

## 8. Proposal preparation prompts and acceptance checks

Write the final proposal yourself using these planning prompts. The brief allows AI for idea exploration and conceptual help, but prohibits generated implementation and mockups.

Problem to investigate: movie and TV viewers want one private place to remember what they plan to watch, track progress and record opinions, without needing connectivity.

Target scope: individual movie/TV enthusiasts using a small offline catalogue. This is a course prototype, not a complete real-world entertainment database.

UML use-case work: draw one primary actor, the local viewer. Consider browse/search, view details, manage watchlist, record a watched movie, update episode progress, save/edit an opinion and view diary. Do not add a remote API actor or administrator unless you actually implement such a role; neither belongs in this scope.

| Story topic to write in your own words | Acceptance checks |
|---|---|
| Find a title | Search ignores letter case and trims surrounding spaces; blank query shows all; no match shows a helpful empty state; movie/TV filter works offline |
| Plan viewing | Adding a valid title displays it exactly once; duplicate/completed additions are blocked; removing it does not erase progress |
| Log a movie | Marking watched creates one record and removes a watchlist entry if present; repeating the action creates no duplicate |
| Track TV | Checking/unchecking valid episodes updates progress; final known episode completes the show; unrelated episode IDs are rejected |
| Record an opinion | Valid rating/review saves; invalid values leave existing data unchanged; cancel preserves old values; user can clear an opinion |
| Revisit diary | Latest changes appear consistently after navigation; titles without ratings still appear; empty diary has a useful explanation |
| Optional: retain data | A cold restart restores records; missing data starts cleanly; malformed data has a deliberate recovery path |

## 9. Milestone-aligned schedule

Use course weeks until the lecturer confirms dates. You are currently in Week 4, so follow this schedule from the first row.

| Course week | Main work | Evidence / exit condition |
|---|---|---|
| 4 | Confirm team, scope and rules; create repository; choose tiny example dataset | One-page scope; exclusions; each teammate knows responsibilities |
| 5 — Milestone 1 | Write proposal, manually draw UML use cases, write user stories and acceptance criteria | Explain problem/users/solution and demonstrate realistic boundaries |
| 6 — Milestone 2 | Implement models and two services yourself; test every service operation | Explain relationships; normal/edge/invalid tests; demonstrate cross-entity completion rule without UI |
| 7 | Apply model feedback; manually sketch the five screens and navigation; choose visual conventions | Core journey is understandable in your sketches |
| 8 | Build reusable widgets and screen structure using bundled data | Five screen types are reachable; reusable stateless and stateful widgets are identifiable |
| 9 — Milestone 3 | Refine your manual mockups and Flutter UI; document widget responsibilities | Present designed mockups and widget architecture; include empty/validation states |
| 10 | Connect watchlist, movie tracking and review actions to services using callbacks/setState | Complete movie journey works across screens |
| 11 | Complete TV checklist/progress; resolve shared-state issues; attempt persistence only if core is stable | Complete TV journey; no stale watchlist or diary state |
| 12 — Milestone 4 | Demonstrate interactions, state and data handling; freeze features | Full offline walkthrough; optional persistence verified or explicitly omitted |
| 13 | Fix bugs, test on target device, polish accessibility/spacing and rehearse | Stable build; no new features; each person explains their work |
| 14 — Final jury | Demonstrate final product and engineering decisions | Working offline build, test evidence, manual designs, repository history and concise demo |

Milestone 2 is the early bottleneck. Begin models, service rules and tests while preparing the proposal; do not leave them until after UI design.

## 10. Time budget and scope cuts

The table below retains a rough **80–120 person-hour baseline** for the proposed work; it is not 80–120 hours per teammate. For your two-person team, provisionally reserve **6–8 focused hours per person per week** (120–160 person-hours across ten elapsed weeks). The difference provides room for learning, reviews, integration and rework. Availability is not yet confirmed, and two people will not automatically halve the duration. Re-estimate after Week 6 using actual progress.

| Work area | Approximate hours |
|---|---:|
| Scope, proposal and manual use-case work | 6–8 |
| Models, services and meaningful tests | 18–24 |
| Manual UX/UI design | 8–12 |
| Flutter screens and reusable widgets | 16–22 |
| State, interaction and integration | 12–18 |
| Offline QA, fixes, explanation practice and final demo | 12–18 |
| Optional persistence or additional buffer | 8–18 |
| Total planning range | 80–120 |

Cut in this order if behind:

1. Remove statistics, extra sorts and decorative animation.
2. Use fewer bundled titles and simpler local artwork/placeholders.
3. Omit persistence if it threatens required milestones; it is optional in the brief.
4. Simplify filters and review styling while retaining tested core behaviour.
5. If episode tracking remains too difficult, agree a revised title-level TV diary scope with the teacher. Revisit the entity model so it still contains four meaningful entities; do not leave Episode as an unused class.

Never cut required service tests, entity relationships, the fourth screen, or the ability to explain your own code.

For two students, keep the same feature scope and use the extra capacity for quality. Both should contribute models/services, UI and tests, rather than one person owning only design. Review interfaces together and ensure Git reflects real work.

## 11. Definition of done and demonstration

- [ ] Required entities and relationships are meaningful and explainable.
- [ ] At least two services implement rules beyond CRUD.
- [ ] Completing viewing updates multiple related entities coherently.
- [ ] Every service operation has appropriate normal, edge and invalid-case tests.
- [ ] Five planned screen types work; at least four are genuinely distinct screens.
- [ ] Reusable StatelessWidget and StatefulWidget are present.
- [ ] All state changes use the course-permitted mechanisms.
- [ ] Business logic is outside screen widgets.
- [ ] First launch and the full journey work in airplane mode, including artwork/fonts.
- [ ] Watchlist, details and diary stay consistent when navigating back and forth.
- [ ] Optional persistence survives restart if claimed.
- [ ] Manual designs and implementation are your own work, with feedback incorporated.
- [ ] Each teammate can explain their code and Git contributions.

Suggested short jury demo:

1. Launch offline and explain the intended user and deliberately limited catalogue.
2. Find a movie, add it to the watchlist, mark it watched and save an opinion.
3. Show the watchlist removal and diary entry as one coherent business operation.
4. Track a short TV show's episodes and demonstrate completion and undo behaviour.
5. Show a meaningful validation case and the corresponding service tests.
6. Explain state ownership, model relationships and storage choice. Restart only if demonstrating persistence.

## 12. Two-person ownership plan

These are suggested roles, not an assignment of Den to a particular role. Choose together based on interests and experience. Both members implement their own code, create manual designs and test their work.

| Workstream | Member A leads | Member B leads | Joint review / integration |
|---|---|---|---|
| Proposal and model | Catalogue/watchlist use cases; MediaTitle and WatchlistEntry | Viewing/review use cases; Episode and ViewingRecord | Agree relationships, IDs and acceptance criteria |
| Services and tests | WatchlistService and its tests | ViewingService and its tests | Completion removes watchlist entry; invalid actions preserve state |
| Manual UI and widgets | Discover and Watchlist | Diary and review editor | Title details, shared visual conventions and navigation |
| State and data | Bundled catalogue loading and shared state integration | Viewing action integration; optional persistence investigation | Shared state ownership, callbacks and offline behaviour |
| Final quality | Search, watchlist and layout checks | TV progress, opinions and restoration checks | Cross-review tests, fix integration issues, rehearse together |

ViewingService is the heavier service. Member A therefore takes more shared integration/data work. Rebalance after Week 6 according to actual effort; aim for meaningful contributions, not identical commit counts.

Working rhythm:

- Hold two short check-ins each week: agree the next small deliverables, then integrate and review them.
- Agree model fields and service inputs/outputs before implementing dependent UI.
- Keep each change small; review each other's work before merging into the main branch.
- Integrate at least weekly. Do not leave two disconnected halves until Week 12.
- Before each milestone, each member explains their own implementation and the shared data flow.
- Keep one task board with task, owner, course-week deadline and completion evidence.

## 13. Week 4 checklist — start here

- [ ] **Together:** agree the one-sentence goal and freeze the core features in Section 3.
- [ ] **Together:** choose an app name and create the shared repository; verify both members can contribute.
- [ ] **Member A:** outline catalogue/watchlist requirements and select a tiny offline sample catalogue.
- [ ] **Member B:** outline viewing/review requirements and choose explicit episode-progress and rating rules.
- [ ] **Together:** refine the four entities and two services; check the lecturer's architecture guidance.
- [ ] **Each member:** draft their use cases and acceptance criteria; combine and manually draw the proposal's UML use-case diagram.
- [ ] **Together:** review the Week 5 proposal and identify the first model/service tasks for Week 6.

End-of-week evidence: agreed scope, explicit exclusions, repository contributions from both members, draft proposal, initial model relationships and a task list with owners. Start with paper sketches if helpful; high-fidelity UI is not this week's priority.

## 14. Remaining decisions

Confirmed: **two members, currently Week 4**. No need to reconfirm these.

- Actual milestone dates from the lecturer.
- Realistic weekly hours for each member.
- Course-prescribed architecture and permitted local storage approach.
- Which member takes each lead role.

Keep the same core scope. Local persistence is the first optional enhancement; decide after the core is stable whether the team has capacity. Do not add social features just because there are two developers.

## 15. Decision register

| ID | Decision | Rationale |
|---|---|---|
| D01 | Personal movie-and-TV diary | Manageable scope |
| D02 | Bundled catalogue, no runtime API or remote images | Offline rule, brief p. 3 |
| D03 | Five screen types, four entities, two services | Meets required minimums |
| D04 | setState, callbacks and navigation | Brief p. 4 |
| D05 | Persistence is the first optional enhancement | Useful but not mandatory |
| D06 | One record per title; no rewatch history | Limits complexity |
| D07 | Both members implement, test and explain | Participation requirement |
| D08 | Feature freeze at Week 12 | Protect fixes and jury preparation |

Before adding a feature, record the benefit, owner, test impact and what will be deferred. Update affected acceptance criteria, tests and schedule together.

## 16. Team tracking

### Team details

| Item | Team entry |
|---|---|
| App name | To choose |
| Member A | To assign |
| Member B | To assign |
| Repository | Add team repository link |
| Milestone calendar dates | Confirm with lecturer |
| Weekly availability | Confirm for each member |
| Course architecture reference | Add reference |

### Milestone checklist

- [ ] **Week 4:** scope, repository, roles and proposal draft.
- [ ] **Week 5 — M1:** proposal, manual UML and acceptance criteria.
- [ ] **Week 6 — M2:** models, services, tests and cross-entity operation.
- [ ] **Week 9 — M3:** manual designs and widget architecture.
- [ ] **Week 12 — M4:** state, interactions and data handling; feature freeze.
- [ ] **Week 13:** offline/device checks, fixes and rehearsal.
- [ ] **Week 14:** final jury demonstration.

### Active tasks

| Task | Owner | Due | Status | Evidence |
|---|---|---|---|---|
| Confirm scope and roles | Both | Week 4 | To do | Updated team details |
| Create repository and verify access | Both | Week 4 | To do | Genuine contribution from each |
| Catalogue/watchlist requirements | Member A | Week 4 | To do | Reviewed requirements |
| Viewing/review requirements | Member B | Week 4 | To do | Reviewed requirements |
| Agree models and service contracts | Both | Week 4 | To do | Relationships and rules agreed |
| Complete proposal and manual UML | Both | Week 5 | To do | Milestone 1 materials |
| Implement/test watchlist domain work | Member A | Week 6 | To do | Explained code and passing tests |
| Implement/test viewing domain work | Member B | Week 6 | To do | Explained code and passing tests |
| Review completion/watchlist integration | Both | Week 6 | To do | Cross-entity behaviour and tests |

### Meeting log template

Copy this block for each check-in:

- **Date / course week:**
- **Attendees:**
- **Completed work:**
- **Blockers or lecturer feedback:**
- **Decisions and affected sections:**
- **Next actions, owners and due weeks:**

### Change log

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-10-20 | Finalized two-person Week 4 baseline; explicit TMDB decision, Obsidian navigation and team tracking |

## 17. Using and sharing this note

1. Save this Markdown file inside your Obsidian vault, in a project folder of your choice.
2. Keep the filename stable; use the internal navigation or headings to find sections.
3. Fill in names and dates; update checkboxes and the task table during check-ins.
4. Share this same file with your teammate. Keep one authoritative copy in your shared repository, for example under `docs/`, and review edits together.
5. Include the original `PROJECT - Description.pdf` alongside shared project materials. It is referenced here, not embedded.
6. Downloaded copies do not synchronize future edits; agree which repository copy is authoritative.

This is a self-contained note with no plugins or external embeds required. Internal navigation uses Obsidian heading links; all substantive content remains plain Markdown. No message or access invitation has been sent to your teammate.

## Source and evidence boundary

- `PROJECT - Description.pdf`, pp. 1–5: assignment rules, technical constraints, milestones and proposal deliverables.
- All feature choices, entity/service suggestions, UI task descriptions, estimates and scope-cut rules in this plan are advisory recommendations, not requirements stated by the lecturer.
- No external product feature comparison or package-version recommendation was needed for this plan. Letterboxd and Serializd are inspiration supplied by the student; this document does not claim feature parity.
