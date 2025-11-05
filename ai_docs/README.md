# Google Play Scraper - AI Documentation

This directory contains AI-focused documentation for the **google-play-scraper** project.

## Quick Reference

### What is this project?

A Python library that scrapes data from Google Play Store **without external dependencies**. It extracts app details, reviews, permissions, and search results using only Python standard library.

### Key Facts

- **Language**: Python 3.7+
- **Dependencies**: None (standard library only)
- **License**: MIT
- **Status**: Production/Stable (v1.2.7)

## Core Capabilities

### 1. App Details (`app`)
Extract comprehensive information about any app:
- Metadata (title, description, screenshots, videos)
- Metrics (installs, ratings, reviews, score)
- Pricing (price, currency, IAP info)
- Developer information
- Categories and genres
- Content ratings

### 2. App Reviews (`reviews`, `reviews_all`)
Retrieve user reviews with:
- Pagination support (continuation tokens)
- Filtering by rating (1-5 stars)
- Sorting (newest, most relevant, rating)
- Batch fetching (up to 4500 per batch)

### 3. App Permissions (`permissions`)
Get permission information:
- Grouped by category (Camera, Storage, etc.)
- Sorted permission lists

### 4. App Search (`search`)
Search Google Play Store:
- Query-based search
- Configurable result count (up to 30)
- Returns app previews with key details

## Architecture Highlights

### Data Extraction Pattern
- Uses `ElementSpec` classes to navigate nested JSON structures
- Extracts data from `<script>` tags using regex
- Post-processes data (unescaping, type conversion)
- Handles missing data with fallbacks

### Request Handling
- HTTP requests via `urllib.request`
- Automatic retry with exponential backoff
- Rate limit detection and handling
- SSL context configuration

### Key Components

```
constants/
  element.py      # Data extraction specifications
  request.py      # URL/payload builders
  regex.py        # Parsing patterns
  
features/
  app.py          # App detail extraction
  reviews.py      # Review scraping with pagination
  permissions.py  # Permission extraction
  search.py       # Search functionality
  
utils/
  request.py      # HTTP client
  data_processors.py  # Data cleaning
```

## Usage Examples

```python
from google_play_scraper import app, reviews, reviews_all, permissions, search, Sort

# Get app details
result = app('com.nianticlabs.pokemongo', lang='en', country='us')

# Get reviews with pagination
reviews_data, token = reviews(
    'com.fantome.penguinisle',
    lang='en',
    country='us',
    sort=Sort.NEWEST,
    count=100
)

# Get all reviews (may take a while for popular apps)
all_reviews = reviews_all('com.spotify.music', sleep_milliseconds=100)

# Get permissions
perms = permissions('com.spotify.music', lang='en', country='us')

# Search apps
results = search('best Pikachu game', n_hits=3, lang='en', country='us')
```

## Important Notes for AI

### Data Structure Access
- Data is extracted from nested JSON arrays accessed via index paths
- Paths are stored in `ElementSpec` objects (e.g., `[1, 2, 0, 0]`)
- These paths may change if Google Play updates their structure

### Error Handling
- Custom exceptions: `NotFoundError`, `ExtraHTTPError`
- Automatic fallbacks for URLs and data extraction
- Retry logic for rate limiting

### Limitations
- Google Play rate limits may affect large-scale scraping
- Review pagination limited to 200 reviews per page
- No built-in caching (fresh requests each time)

### Parameter Formats
- `lang`: ISO 639-1 code (e.g., 'en', 'ko', 'ja')
- `country`: ISO 3166 code (e.g., 'us', 'kr', 'jp')
- `sort`: `Sort.NEWEST`, `Sort.MOST_RELEVANT`, or `Sort.RATING`

## Related Files

- `project_overview.md` - Detailed technical documentation
- Main `README.md` - User-facing documentation with examples
- `pyproject.toml` - Project metadata and dependencies

## Testing

End-to-end tests validate actual Google Play Store responses:
- `test_app.py` - App detail extraction
- `test_reviews.py` - Review pagination
- `test_reviews_all.py` - Bulk review fetching
- `test_permissions.py` - Permission extraction
- `test_search.py` - Search functionality

## Maintenance Considerations

When Google Play changes their structure:
1. Update `ElementSpec` data_map paths in `constants/element.py`
2. Update regex patterns in `constants/regex.py` if script structure changes
3. Update payload formats in `constants/request.py` if API changes
4. Run end-to-end tests to validate changes

