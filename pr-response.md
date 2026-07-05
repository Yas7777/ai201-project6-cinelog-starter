# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` so the function follows the project’s existing `verb_to_noun` naming pattern, like `add_to_collection()`. I also updated the call site in `routes/watchlist/watchlist.py` so the route now imports and calls `add_to_watchlist()`.

**How I verified:**
Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` so the function follows the project’s existing `verb_to_noun` naming pattern, like `add_to_collection()`. I also updated the call site in `routes/watchlist/watchlist.py` so the route now imports and calls `add_to_watchlist()`.


## Comment 2 — Deduplication
**What I did:**  
Added a duplicate check in `add_to_watchlist()` before creating a new `WatchlistEntry`. The function now queries for an existing row with the same `user_id` and `film_id`, and raises an error instead of inserting a duplicate.

**How I verified:**  
I compared the logic against the existing `add_to_collection()` pattern in `services/collection_service.py`. I also checked the code path manually to confirm the duplicate lookup happens before the new `WatchlistEntry` is created and committed.


## Comment 3 — Missing test
**What I did:**  
Added a test for the case where `add_to_watchlist()` is called with a `film_id` that does not exist in the database. The test uses a missing UUID-style film ID and confirms that the service raises `FilmNotFoundError`.

**How I verified:**  
I ran `pytest tests/test_watchlist.py -v` to confirm the watchlist tests pass.


## Comment 4 — Default visibility
**My position:**  
I am keeping the watchlist default as `public=True`.

**Reasoning:**  
My reasoning is that CineLog watchlists are meant to support discovery and sharing. The user behavior I am optimizing for is social discovery: users can see what films other people want to watch, compare lists, and use watchlists as a public signal of film interests. Since the app already has film collection behavior, keeping watchlists public by default makes the most sense to the app's rationale. 

**Tradeoff acknowledged:**  
The tradeoff is that `public=False` would be more privacy-protective. Some users may not want others to know what they plan to watch, especially if their watchlist is personal/sensitive in nature. In a production version, I would make visibility very clear in the UI and give users an easy way to switch a watchlist to private. If privacy became the main priority, I would change the default to `public=False`.


## Comment 5 — Sort order
**My position:**  
I am implementing the maintainer’s preference and changing the watchlist sort order to `date_added` descending.

**Reasoning:**  
I agree that watchlists should prioritize time over alphabetical order. A watchlist is closer to a “things I recently decided I want to watch” list than a static library. Showing the most recently added films first helps users quickly find the movies they just saved, which is likely the most common behavior after adding something to the list.

**Engagement with reviewer's point:**  
The reviewer pointed out that most users want to see what they added recently, and I think that is a stronger default than alphabetical order for this feature. Alphabetical sorting can still be useful for browsing a long list, but it is less aligned with the immediate user flow of saving a film and then returning to it later. For this version, I changed the default ordering to newest-added first. In a fuller product, I would consider adding a user-selectable sort option for alphabetical, oldest-added, or newest-added.


## Comment 6 — Rebase
**What conflicted:**  
There was no file-level Git conflict during the rebase. The rebase completed without Git stopping for conflict resolution. The issue was a logical conflict from the main-branch refactor: the watchlist code still described `film_id` as an integer even though films now use UUID IDs.

**How I resolved it:**  
I updated the remaining watchlist references from integer IDs to UUIDs. In `routes/watchlist/watchlist.py`, I changed the request body documentation from `<int>` to `<uuid string>`. In `services/watchlist_service.py`, I changed the `film_id` docstring from an integer/pre-refactor ID to a UUID string.

**How I verified no conflict remains:**  
I searched the watchlist route and service files for old integer references like `film_id (int)`, `<int>`, and `pre-refactor`, and confirmed they no longer appear. I also checked the branch history with `git log --oneline --merges origin/main..HEAD` to confirm there are no merge commits remaining after the rebase.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->