# Movie Info System Documentation

## Overview

The `Auto_Filter_Bot` collects movie information to display posters, ratings, plots, and other details when a user searches for a movie or TV show.

Currently, the bot supports the following methods for retrieving movie information:

1.  **Hybrid Approach (DuckDuckGo + Cinemagoer):** This is the primary method.
2.  **Cinemagoer (IMDbPY):** The legacy library for interacting with IMDb.
3.  **TMDB (The Movie Database):** An external API method (requires API key).
4.  **Google Scraping:** A fallback method for scraping Google search results (often unreliable due to blocking).

## Recent Changes (Fix for Broken Search)

As of late 2024/2025, the `Cinemagoer` library's `search_movie` function has been reported as broken (returning empty lists) due to changes in IMDb's website structure. Additionally, scraping Google directly is often blocked.

To resolve this, a **Hybrid Approach** has been implemented in `utils.py`.

### The Hybrid Approach

Instead of relying on `Cinemagoer` to *search* for the movie (which fails), we use **DuckDuckGo** to find the IMDb page for the movie, and then use `Cinemagoer` to *fetch* the details using the ID found.

**Steps:**
1.  **Search:** The bot uses the `ddgs` (DuckDuckGo Search) library to search for `[Movie Name] site:imdb.com/title`.
2.  **Extract ID:** It parses the search results to find a URL containing an IMDb ID (e.g., `tt1234567`).
3.  **Fetch Details:** It passes this ID to `Cinemagoer.get_movie(id)`, which still works correctly to retrieve metadata.

### Configuration

#### Requirements
The following Python package has been added to `requirements.txt`:
```
ddgs
```

#### Environment Variables
No new environment variables are strictly required for the hybrid approach to work, as DuckDuckGo does not require an API key.

However, for the **TMDB** method to work (if enabled via `TMDB_ON_SEARCH`), you must provide:
-   `TMDB_API_KEY`: Your API Key from The Movie Database.

### Usage in Code

The core logic is in `Auto_Filter_Bot/utils.py`.

**Function:** `get_poster(query, ...)`

```python
# New logic
movieid = search_imdb_via_ddg(title)

if not movieid:
    # Fallback to legacy Cinemagoer search
    movieid = imdb.search_movie(title)
```

**Function:** `search_imdb_via_ddg(query)`
This helper function performs the search and returns a list of minimal `Movie` objects containing the ID and Title.

## Troubleshooting

-   **DuckDuckGo Rate Limits:** If the bot makes too many requests rapidly, DuckDuckGo might temporarily block the IP. The `ddgs` library usually handles this gracefully or throws an exception, which is caught, and the bot falls back to other methods.
-   **Wrong Movie Found:** The search relies on the accuracy of DuckDuckGo's top results. Generally, searching with `site:imdb.com/title` provides accurate results.
-   **TMDB Errors:** If using TMDB, ensure your API key is valid.

## Future Improvements

-   If `Cinemagoer` releases an update fixing the `search_movie` function, the hybrid approach can be reverted or kept as a robust fallback.
-   Implementing a proper caching mechanism for search results (beyond what might be in place) can reduce outgoing requests.
