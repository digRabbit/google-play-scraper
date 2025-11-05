# Google Play Scraper - Project Overview

## Executive Summary

**Google-Play-Scraper** is a Python library that provides APIs to crawl and extract data from the Google Play Store **without any external dependencies**. The library uses only Python standard library modules (`urllib`, `json`, `html`, etc.) to scrape data from Google Play Store web pages.

- **Version**: 1.2.7
- **License**: MIT
- **Python Support**: 3.7 - 3.11
- **Status**: Production/Stable
- **Repository**: https://github.com/JoMingyu/google-play-scraper

## Core Functionality

The library provides four main features:

1. **App Details** - Extract comprehensive information about a specific app
2. **App Reviews** - Retrieve user reviews with pagination support
3. **App Permissions** - Get permission information for apps
4. **App Search** - Search for apps on Google Play Store

## Architecture Overview

### Project Structure

```
google_play_scraper/
├── __init__.py              # Public API exports
├── exceptions.py            # Custom exception classes
├── constants/
│   ├── element.py          # Data extraction specifications
│   ├── google_play.py      # Enums (Sort, Device)
│   ├── regex.py            # Regex patterns for parsing
│   └── request.py          # URL and payload formatters
├── features/
│   ├── app.py             # App detail extraction
│   ├── reviews.py         # Review scraping with pagination
│   ├── permissions.py    # Permission extraction
│   └── search.py          # Search functionality
└── utils/
    ├── request.py         # HTTP request handlers
    └── data_processors.py # Data cleaning utilities
```

### Key Design Patterns

#### 1. **Element Specification Pattern**
The library uses an `ElementSpec` class to define how to extract data from nested JSON structures. Each spec contains:
- `ds_num`: Dataset number identifier (e.g., "ds:5")
- `data_map`: List of indices to navigate nested data structures
- `post_processor`: Optional function to transform extracted data
- `fallback_value`: Default value if extraction fails

Example:
```python
"title": ElementSpec(5, [1, 2, 0, 0])
# Extracts title from dataset["ds:5"][1][2][0][0]
```

#### 2. **Request Format Abstraction**
The `Formats` class provides a clean interface for building URLs and POST payloads:
- Each format class handles URL construction with language/country parameters
- POST payloads are URL-encoded JSON arrays
- Fallback URL formats handle edge cases

#### 3. **Pagination with Continuation Tokens**
Reviews use a continuation token system:
- Returns a token with each batch of reviews
- Token encodes pagination state and query parameters
- Enables resuming from any point in the review list

## Technical Implementation Details

### Data Extraction Flow

1. **HTTP Request**: Uses `urllib.request` to fetch HTML pages
2. **Script Tag Parsing**: Extracts JSON data from `<script>` tags using regex
3. **Data Structure Navigation**: Uses nested index lookups to find specific fields
4. **Data Transformation**: Applies post-processors (unescaping, type conversion, etc.)
5. **Error Handling**: Falls back to default values or alternative extraction paths

### HTTP Request Handling

**Key Features**:
- SSL context configured to handle certificate verification
- Retry mechanism (3 attempts) for rate limiting
- Automatic rate limit detection and exponential backoff
- Error handling for 404 and other HTTP errors

**Rate Limiting**:
- Detects `PlayGatewayError` responses
- Implements exponential backoff (5s, 10s, 15s)
- Maximum 3 retries before failure

### Data Structures

#### App Detail Structure
Extracts ~40 fields including:
- Basic info: title, description, summary, icon, screenshots
- Metrics: installs, ratings, reviews, score, histogram
- Pricing: price, currency, free status, IAP information
- Developer: name, email, website, address
- Metadata: genre, categories, content rating, version, dates

#### Review Structure
- Review ID, user info (name, image)
- Content, score, thumbs up count
- Timestamps (created, replied)
- App version information

#### Permission Structure
- Grouped by permission category (Camera, Storage, etc.)
- Each category contains a sorted list of permissions

### Regex Patterns

The library uses regex to extract:
- Script tags containing JSON data
- Key-value pairs from script content
- Review data from POST responses
- Permission data structures

### Constants and Configuration

- **Sort Options**: `MOST_RELEVANT`, `NEWEST`, `RATING`
- **Device Types**: `MOBILE`, `TABLET`, `CHROMEBOOK`, `TV`
- **Max Fetch Limits**: 4500 reviews per batch (Google Play limit: 200 per page)
- **Rate Limiting**: 3 retries with 5-second base delay

## API Surface

### Public Functions

```python
from google_play_scraper import app, reviews, reviews_all, permissions, search, Sort

# App details
app(app_id, lang='en', country='us') -> dict

# Reviews with pagination
reviews(app_id, lang='en', country='us', sort=Sort.NEWEST, count=100, 
        filter_score_with=None, continuation_token=None) -> (list, token)

# All reviews (infinite)
reviews_all(app_id, sleep_milliseconds=0, **kwargs) -> list

# Permissions
permissions(app_id, lang='en', country='us') -> dict

# Search
search(query, n_hits=30, lang='en', country='us') -> list
```

### Parameters

- **lang**: ISO 639-1 language code (e.g., 'en', 'ko', 'ja')
- **country**: ISO 3166 country code (e.g., 'us', 'kr', 'jp')
- **sort**: Sort enum for reviews (NEWEST, MOST_RELEVANT, RATING)
- **count**: Number of items to fetch (reviews: up to 4500 per batch)
- **filter_score_with**: Filter reviews by rating (1-5)
- **continuation_token**: Resume pagination from a specific point

## Error Handling

### Custom Exceptions

- `GooglePlayScraperException`: Base exception class
- `NotFoundError`: Raised when app/search returns 404
- `ExtraHTTPError`: Raised for other HTTP errors

### Fallback Mechanisms

1. **URL Fallbacks**: If country-specific URL fails, tries language-only URL
2. **Data Extraction Fallbacks**: Uses fallback values or alternative ElementSpecs
3. **Retry Logic**: Automatically retries on rate limit errors

## Testing

End-to-end tests cover:
- App detail extraction
- Review pagination
- Review filtering
- Permission extraction
- Search functionality

Tests are located in `tests/e2e_tests/` and validate actual Google Play Store responses.

## Dependencies

**Zero external dependencies** - Uses only Python standard library:
- `urllib.request` - HTTP requests
- `urllib.parse` - URL encoding
- `json` - JSON parsing
- `html` - HTML unescaping
- `datetime` - Timestamp conversion
- `enum` - Enumerations
- `ssl` - SSL context configuration
- `time` - Sleep for rate limiting

## Known Limitations

1. **Google Play Rate Limits**: Library includes retry logic, but excessive requests may still fail
2. **Google Play Structure Changes**: Data extraction relies on specific JSON structure paths that may change
3. **Review Pagination**: Maximum 200 reviews per page (Google Play limitation)
4. **Large Review Sets**: `reviews_all()` can generate thousands of HTTP requests for popular apps

## Performance Considerations

- **Batch Size**: Reviews fetched in batches of up to 4500
- **Sleep Delays**: Optional sleep between batches in `reviews_all()`
- **Rate Limiting**: Built-in exponential backoff for rate limit handling
- **Caching**: No built-in caching (each call makes fresh requests)

## Maintenance Notes

- Data structure paths (indices in ElementSpecs) may need updates if Google Play changes their JSON structure
- Regex patterns may need updates if Google Play changes their HTML/script structure
- Payload formats for POST requests are hardcoded and may need updates

## Future Extensibility

The architecture supports easy addition of:
- New data fields (add ElementSpecs)
- New endpoints (add Format classes)
- New post-processors (add transformation functions)
- New filtering options (extend function parameters)

