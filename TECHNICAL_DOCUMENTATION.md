# Technical Documentation

## Image Downloader Project

**Version:** 1.0.0  
**Last Updated:** 2025  
**License:** MIT

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [System Requirements](#system-requirements)
4. [Dependencies](#dependencies)
5. [Module Documentation](#module-documentation)
6. [Data Flow](#data-flow)
7. [Error Handling](#error-handling)
8. [Configuration Management](#configuration-management)
9. [API Integration](#api-integration)
10. [Testing Strategy](#testing-strategy)
11. [Extension Points](#extension-points)
12. [Performance Considerations](#performance-considerations)
13. [Security Considerations](#security-considerations)
14. [Troubleshooting Guide](#troubleshooting-guide)

---

## Overview

The Image Downloader is a production-ready Python application designed to download images from external sources (currently Flickr) based on configurable category searches. The system is built with extensibility in mind, featuring a modular architecture that allows for easy integration of additional image source providers.

### Key Features

- **Modular Provider System**: Plugin-style architecture supporting multiple image sources
- **Robust Error Handling**: Automatic retries with exponential backoff
- **Rate Limiting**: Built-in respect for API rate limits
- **Progress Tracking**: Real-time progress indicators using tqdm
- **Flexible Configuration**: File-based and CLI-based configuration options
- **Type Safety**: Full type hints throughout the codebase
- **Comprehensive Logging**: Structured logging for debugging and monitoring
- **Test Coverage**: Unit tests with mocked API responses

### Use Cases

- Batch downloading images for machine learning datasets
- Creating image collections for research or personal use
- Automated image gathering for content creation
- Educational purposes demonstrating API integration patterns

---

## Architecture

### High-Level Architecture

The application follows a **layered architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────┐
│                    CLI Layer (main.py)                  │
│  - Argument parsing                                      │
│  - Logging configuration                                 │
│  - User interaction                                      │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│              Dispatcher Layer (downloader.py)           │
│  - Provider selection                                   │
│  - Configuration management                             │
│  - Session orchestration                                │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│          Provider Layer (image_sources/)                │
│  - Provider registry                                    │
│  - Provider implementations (flickr.py)                 │
│  - Provider interface                                   │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│            Utility Layer (utils.py)                     │
│  - HTTP requests with retry logic                       │
│  - File operations                                      │
│  - API key management                                   │
│  - Filename sanitization                                │
└─────────────────────────────────────────────────────────┘
```

### Component Interaction

1. **CLI Entry Point** (`main.py`): Parses arguments, configures logging, and invokes the dispatcher
2. **Dispatcher** (`downloader.py`): Selects appropriate provider, manages API keys, and coordinates downloads
3. **Providers** (`image_sources/`): Implement source-specific download logic
4. **Utilities** (`utils.py`): Provide shared functionality across all layers

### Design Patterns

- **Factory Pattern**: Provider registry uses factory pattern to instantiate providers
- **Strategy Pattern**: Different providers implement the same interface, allowing runtime selection
- **Retry Pattern**: Exponential backoff for network requests
- **Template Method**: Common download workflow with provider-specific implementations

---

## System Requirements

### Runtime Requirements

- **Python**: 3.8 or higher
- **Operating System**: Linux, macOS, Windows
- **Network**: Internet connection for API access
- **Storage**: Sufficient disk space for downloaded images

### Development Requirements

- **Python**: 3.8+
- **pytest**: For running tests
- **Git**: For version control (if contributing)

---

## Dependencies

### Core Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `requests` | >=2.32.0 | HTTP client for API calls |
| `python-dotenv` | >=1.0.0 | Environment variable management |
| `tenacity` | >=8.2.0 | Retry logic with exponential backoff |
| `tqdm` | >=4.66.0 | Progress bar visualization |

### Development Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `pytest` | >=8.0.0 | Testing framework |

### Dependency Rationale

- **requests**: Industry-standard HTTP library with excellent error handling
- **python-dotenv**: Secure API key management without hardcoding
- **tenacity**: Robust retry mechanism with configurable strategies
- **tqdm**: User-friendly progress indication for long-running operations
- **pytest**: Modern testing framework with excellent mocking capabilities

---

## Module Documentation

### `main.py` - CLI Entry Point

**Purpose**: Command-line interface for the application.

**Key Functions**:

- `setup_logging(level)`: Configures logging format and level
- `parse_arguments()`: Parses CLI arguments using argparse
- `parse_categories(categories_str)`: Converts comma-separated string to list
- `main()`: Application entry point, orchestrates configuration and execution

**CLI Arguments**:

| Argument | Type | Default | Description |
|----------|------|---------|-------------|
| `--source` | str | `config.SOURCE` | Image source provider |
| `--limit` | int | `config.NUM_IMAGES` | Images per category |
| `--out` | str | `config.OUTPUT_DIR` | Output directory |
| `--categories` | str | `config.CATEGORIES` | Comma-separated categories |
| `--safe_search` | int | `config.FLICKR_CONFIG['safe_search']` | Safe search level (1-3) |
| `--license` | str | `config.FLICKR_CONFIG['license']` | License filter |
| `--content_type` | int | `config.FLICKR_CONFIG['content_type']` | Content type (1-3) |
| `--verbose` / `-v` | flag | False | Enable DEBUG logging |

**Error Handling**: Catches `ValueError`, `KeyboardInterrupt`, and generic exceptions, exiting with appropriate codes.

---

### `config.py` - Configuration Management

**Purpose**: Centralized configuration storage.

**Configuration Variables**:

```python
CATEGORIES: List[str]          # Search categories
NUM_IMAGES: int                # Images per category
OUTPUT_DIR: str                # Base output directory
SOURCE: str                    # Default provider
API_KEYS: Dict[str, str]       # Provider API keys
FLICKR_CONFIG: Dict            # Flickr-specific settings
```

**FLICKR_CONFIG Structure**:

- `license`: License filter ("any", "0", "4", "5", "6", "7", "8", "9", "10")
- `safe_search`: Content safety level (1=safe, 2=moderate, 3=restricted)
- `content_type`: Content type filter (1=photos, 2=screenshots, 3=other)

**Security Note**: API keys can be set via environment variables for production use.

---

### `downloader.py` - Dispatcher Module

**Purpose**: Central orchestration for image downloads.

**Key Function**:

```python
download_images(
    categories: List[str],
    num_images: int,
    output_dir: str,
    source: str,
    api_keys: Dict[str, str],
    extra_config: Optional[Dict] = None
) -> None
```

**Responsibilities**:

1. Validates source provider availability
2. Retrieves and validates API keys
3. Instantiates appropriate provider class
4. Executes downloads for each category
5. Handles errors gracefully (continues with next category on failure)

**Error Handling**: 
- Raises `ValueError` for configuration errors
- Logs errors but continues processing remaining categories
- Re-raises unexpected exceptions

---

### `image_sources/__init__.py` - Provider Registry

**Purpose**: Provider factory and registry.

**Key Components**:

- `PROVIDERS`: Dictionary mapping provider names to class paths
- `get_provider_class(source)`: Factory function that dynamically imports and returns provider class

**Current Providers**:

- `flickr`: `image_sources.flickr.FlickrDownloader`

**Extension**: Add new providers by updating the `PROVIDERS` dictionary.

---

### `image_sources/flickr.py` - Flickr Provider

**Purpose**: Flickr-specific image download implementation.

**Class**: `FlickrDownloader`

**Initialization**:

```python
FlickrDownloader(api_key: str, config: Optional[Dict] = None)
```

**Key Methods**:

1. **`_build_search_params(query, per_page)`**: Constructs Flickr API request parameters
2. **`_search_photos(query, num_images)`**: Searches Flickr API for photos
3. **`_get_photo_url(photo)`**: Extracts best available photo URL (prioritizes larger sizes)
4. **`download(query, num_images, output_dir)`**: Main download method

**Flickr API Integration**:

- **Endpoint**: `https://www.flickr.com/services/rest/`
- **Method**: `flickr.photos.search`
- **Response Format**: JSON
- **Rate Limit**: 3600 requests per hour per API key

**URL Priority** (largest to smallest):
1. `url_o` - Original size
2. `url_l` - Large (1024px)
3. `url_c` - Medium (800px)
4. `url_m` - Small (500px)
5. Constructed URL (fallback)

**File Naming Convention**: `{category}_{index:03d}__{photo_id}.{ext}`

**Features**:
- Skips existing files (idempotent downloads)
- Progress bar with tqdm
- Detailed logging of success/failure counts
- Handles missing URLs gracefully

---

### `utils.py` - Utility Functions

**Purpose**: Shared utility functions.

**Key Functions**:

#### `get_api_key(source, config_keys) -> str`

Retrieves API key from config or environment variables.

**Priority**:
1. `config.API_KEYS[source]` (if not placeholder)
2. Environment variable `{SOURCE}_API_KEY`
3. Raises `ValueError` if not found

#### `sanitize_filename(filename) -> str`

Sanitizes strings for filesystem-safe filenames.

**Operations**:
- Replaces spaces with underscores
- Removes unsafe characters: `<>:"/\|?*`
- Strips leading/trailing dots and spaces
- Limits length to 200 characters
- Returns "image" for empty inputs

#### `ensure_directory(path) -> Path`

Creates directory structure if it doesn't exist.

**Behavior**: Uses `Path.mkdir(parents=True, exist_ok=True)`

#### `validate_license(license_value) -> Optional[str]`

Validates license filter values.

**Behavior**: Returns `None` for "any", otherwise returns original value.

#### `http_get_with_retry(url, params, timeout) -> requests.Response`

HTTP GET with automatic retry logic.

**Retry Strategy**:
- **Max Attempts**: 3
- **Backoff**: Exponential (2s, 4s, 8s, max 10s)
- **Retries On**: `requests.exceptions.RequestException`
- **Timeout**: 30 seconds (default)

**Decorator**: Uses `@retry` from `tenacity` library.

#### `download_file(url, output_path) -> bool`

Downloads file from URL to local path.

**Behavior**:
- Uses `http_get_with_retry` for fetching
- Writes binary content to file
- Returns `True` on success, `False` on failure
- Logs errors without raising exceptions

---

## Data Flow

### Download Workflow

```
1. User invokes CLI with arguments
   ↓
2. main.py parses arguments and loads config
   ↓
3. downloader.py receives download request
   ↓
4. Provider class instantiated with API key and config
   ↓
5. For each category:
   a. Provider searches API (flickr.photos.search)
   b. API returns photo metadata
   c. Provider extracts photo URLs
   d. For each photo:
      - Sanitize category name for directory
      - Create category subdirectory
      - Generate filename
      - Check if file exists (skip if yes)
      - Download image via http_get_with_retry
      - Write to disk
      - Update progress bar
   ↓
6. Log completion statistics
```

### Configuration Resolution

```
CLI Arguments
    ↓
Override config.py defaults
    ↓
Merge with provider-specific config
    ↓
Validate and use
```

### API Key Resolution

```
config.API_KEYS[source]
    ↓ (if placeholder or missing)
Environment Variable: {SOURCE}_API_KEY
    ↓ (if missing)
Raise ValueError
```

---

## Error Handling

### Error Handling Strategy

The application implements a **multi-layered error handling** approach:

1. **Network Errors**: Retry with exponential backoff (3 attempts)
2. **API Errors**: Log and raise `ValueError` with descriptive message
3. **File System Errors**: Log and return `False` (non-fatal)
4. **Configuration Errors**: Raise `ValueError` immediately
5. **Category-Level Errors**: Log error, continue with next category
6. **Session-Level Errors**: Log and exit with error code

### Error Types

| Error Type | Handling Strategy | User Impact |
|------------|-------------------|-------------|
| Network timeout | Retry 3x with backoff | Transparent retry |
| Invalid API key | Raise ValueError | Immediate failure with message |
| No photos found | Log warning, continue | Category skipped |
| File write failure | Log error, continue | Individual file skipped |
| Missing API key | Raise ValueError | Immediate failure with instructions |
| Keyboard interrupt | Catch, log, exit 130 | Graceful shutdown |

### Logging Levels

- **DEBUG**: Detailed execution flow (verbose mode)
- **INFO**: Normal operations, progress updates
- **WARNING**: Non-fatal issues (missing photos, skipped files)
- **ERROR**: Failures that prevent operation

---

## Configuration Management

### Configuration Sources (Priority Order)

1. **CLI Arguments**: Highest priority, override all
2. **config.py**: Default values
3. **Environment Variables**: For API keys and sensitive data

### Configuration Validation

The application validates:

- Category list is non-empty
- `NUM_IMAGES` is positive
- `safe_search` is 1, 2, or 3
- `content_type` is 1, 2, or 3
- API key is present and not placeholder

### Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `FLICKR_API_KEY` | Flickr API key | `abc123def456...` |

**Loading**: Automatically loaded via `python-dotenv` from `.env` file or system environment.

---

## API Integration

### Flickr REST API

**Base URL**: `https://www.flickr.com/services/rest/`

**Authentication**: API key via query parameter

**Rate Limits**:
- 3600 requests per hour per API key
- No explicit rate limiting in code (relies on retry logic)

**Request Format**:
```
GET /services/rest/?method=flickr.photos.search
    &api_key={key}
    &text={query}
    &per_page={count}
    &format=json
    &nojsoncallback=1
    &safe_search={level}
    &content_type={type}
    &license={license}
    &media=photos
    &sort=relevance
    &extras=url_c,url_o,url_l,url_m
```

**Response Format**:
```json
{
  "stat": "ok",
  "photos": {
    "page": 1,
    "pages": 100,
    "perpage": 10,
    "total": 1000,
    "photo": [
      {
        "id": "123456",
        "server": "1234",
        "secret": "abcdef",
        "url_c": "https://...",
        "url_o": "https://..."
      }
    ]
  }
}
```

**Error Response**:
```json
{
  "stat": "fail",
  "code": 100,
  "message": "Invalid API Key"
}
```

---

## Testing Strategy

### Test Structure

Tests are located in `tests/` directory:

- `test_config.py`: Configuration validation tests
- `test_flickr_downloader.py`: Provider implementation tests

### Testing Approach

**Unit Tests**: Test individual functions and methods in isolation

**Mocking Strategy**:
- API responses are mocked using `unittest.mock`
- Network calls are intercepted to prevent real API usage
- File system operations use `tmp_path` fixture

**Test Coverage Areas**:

1. **Configuration**:
   - Required constants exist
   - Type validation
   - Value validation

2. **Utilities**:
   - Filename sanitization edge cases
   - API key resolution (config vs env)
   - License validation
   - Directory creation

3. **Flickr Provider**:
   - Initialization with various configs
   - Parameter building
   - URL extraction (various sizes)
   - Search API integration
   - Download workflow
   - Error handling

4. **Provider Registry**:
   - Valid provider lookup
   - Invalid provider error handling

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=. --cov-report=html

# Run specific test file
pytest tests/test_flickr_downloader.py -v

# Run with verbose output
pytest -v
```

### Test Fixtures

- `downloader`: Creates `FlickrDownloader` instance with test config
- `tmp_path`: Temporary directory for file operations (pytest built-in)

---

## Extension Points

### Adding a New Image Source Provider

To add a new provider (e.g., Unsplash, Pexels):

#### Step 1: Create Provider Module

Create `image_sources/new_provider.py`:

```python
from typing import Dict, Optional
import logging

logger = logging.getLogger(__name__)

class NewProviderDownloader:
    """Downloader for NewProvider image source."""
    
    def __init__(self, api_key: str, config: Optional[Dict] = None):
        """
        Initialize the downloader.
        
        Args:
            api_key: API key for the provider
            config: Optional provider-specific configuration
        """
        self.api_key = api_key
        self.config = config or {}
    
    def download(self, query: str, num_images: int, output_dir: str) -> None:
        """
        Download images for a given query.
        
        Args:
            query: Search query string
            num_images: Number of images to download
            output_dir: Base output directory
        """
        # Implement search and download logic
        # Use utils functions: ensure_directory, sanitize_filename, 
        # download_file, http_get_with_retry
        pass
```

#### Step 2: Register Provider

Update `image_sources/__init__.py`:

```python
PROVIDERS: Dict[str, str] = {
    "flickr": "image_sources.flickr.FlickrDownloader",
    "new_provider": "image_sources.new_provider.NewProviderDownloader",
}
```

#### Step 3: Add Configuration

Update `config.py`:

```python
API_KEYS = {
    "flickr": "your_flickr_api_key_here",
    "new_provider": "your_new_provider_api_key_here"
}

NEW_PROVIDER_CONFIG = {
    "option1": "value1",
    "option2": "value2"
}
```

#### Step 4: Update Dispatcher (if needed)

If provider needs special handling, update `downloader.py`:

```python
provider_config = (
    flickr_config if source == "flickr" 
    else new_provider_config if source == "new_provider"
    else {}
)
```

### Interface Contract

All providers must implement:

- `__init__(api_key: str, config: Optional[Dict] = None)`
- `download(query: str, num_images: int, output_dir: str) -> None`

**Best Practices**:
- Use utility functions from `utils.py`
- Implement proper error handling
- Log operations at appropriate levels
- Use progress bars for long operations
- Skip existing files (idempotent)

---

## Performance Considerations

### Performance Characteristics

**Bottlenecks**:
1. **Network I/O**: Primary bottleneck, mitigated by retry logic
2. **API Rate Limits**: Flickr allows 3600 requests/hour
3. **Disk I/O**: Sequential writes, generally fast

**Optimization Opportunities**:

1. **Parallel Downloads**: Could implement concurrent downloads using `asyncio` or `threading`
2. **Caching**: Cache API responses to avoid redundant searches
3. **Batch Operations**: Group API calls where possible

**Current Limitations**:

- Sequential downloads (one image at a time)
- No connection pooling (new connection per request)
- No response caching

### Scalability

**Current Capacity**:
- Limited by API rate limits (3600 requests/hour for Flickr)
- Disk space for storage
- Network bandwidth

**Scaling Strategies**:
- Multiple API keys (if allowed by provider)
- Distributed execution across machines
- Caching layer for API responses

---

## Security Considerations

### API Key Management

**Best Practices Implemented**:
- Environment variable support for production
- `.env` file support (should be in `.gitignore`)
- No hardcoded keys in version control

**Recommendations**:
- Never commit API keys to version control
- Use environment variables in production
- Rotate keys periodically
- Use least-privilege API keys when available

### Input Validation

**Validated Inputs**:
- Category names (sanitized for filenames)
- Configuration values (type and range checks)
- API responses (structure validation)

**Potential Risks**:
- Malicious API responses (mitigated by response validation)
- Path traversal in category names (mitigated by filename sanitization)
- Large file downloads (no size limits currently)

### Network Security

- Uses HTTPS for all API calls
- Validates SSL certificates (default requests behavior)
- Timeout protection (30s default, 60s for downloads)

---

## Troubleshooting Guide

### Common Issues and Solutions

#### Issue: "No API key found for flickr"

**Symptoms**: `ValueError` on startup

**Solutions**:
1. Check `config.py` has valid API key (not placeholder)
2. Set `FLICKR_API_KEY` environment variable
3. Verify `.env` file exists and contains key

#### Issue: "Flickr API error: Invalid API Key"

**Symptoms**: API returns error code 100

**Solutions**:
1. Verify API key is correct
2. Check API key is active on [Flickr App Garden](https://www.flickr.com/services/api/keys/)
3. Ensure no extra whitespace in key

#### Issue: "No photos found for query"

**Symptoms**: Warning logged, category skipped

**Solutions**:
1. Try different search terms
2. Adjust license filter (some licenses have fewer results)
3. Relax safe_search setting
4. Check query on Flickr.com directly

#### Issue: Rate limit errors

**Symptoms**: HTTP 429 or API error messages

**Solutions**:
1. Reduce `NUM_IMAGES` per category
2. Wait before retrying (rate limit resets hourly)
3. Use multiple API keys if allowed

#### Issue: Connection timeout errors

**Symptoms**: `requests.exceptions.Timeout`

**Solutions**:
1. Check internet connection
2. Increase timeout in `http_get_with_retry` (default 30s)
3. Retry (automatic retry will attempt 3 times)

#### Issue: Permission denied errors

**Symptoms**: `OSError` when creating directories

**Solutions**:
1. Check write permissions for output directory
2. Ensure sufficient disk space
3. Verify path is valid

### Debug Mode

Enable verbose logging:

```bash
python main.py --verbose
```

This provides:
- Detailed API request/response logging
- File operation details
- Step-by-step execution flow

### Log Analysis

Logs include:
- Timestamps
- Module names
- Log levels
- Detailed messages

Example log entry:
```
2025-01-15 10:30:45 - image_sources.flickr - INFO - Found 10 photos for query 'sunsets' (total available: 1250)
```

---

## Appendix

### License Reference

Flickr license codes:
- `0`: All Rights Reserved
- `4`: Attribution License (CC BY)
- `5`: Attribution-ShareAlike License (CC BY-SA)
- `6`: Attribution-NoDerivs License (CC BY-ND)
- `7`: No known copyright restrictions
- `8`: United States Government Work
- `9`: Public Domain Dedication (CC0)
- `10`: Public Domain Mark

### File Structure Reference

```
image-downloader/
├── config.py                 # Configuration
├── downloader.py             # Dispatcher
├── main.py                   # CLI entry point
├── utils.py                  # Utilities
├── image_sources/
│   ├── __init__.py          # Provider registry
│   └── flickr.py            # Flickr provider
├── tests/
│   ├── test_config.py       # Config tests
│   └── test_flickr_downloader.py  # Provider tests
├── requirements.txt         # Dependencies
└── README.md                # User documentation
```

### API Response Example

Flickr search response structure:
```json
{
  "stat": "ok",
  "photos": {
    "page": 1,
    "pages": 10,
    "perpage": 10,
    "total": "100",
    "photo": [
      {
        "id": "123456789",
        "owner": "12345678@N00",
        "secret": "abcdef123",
        "server": "1234",
        "farm": 1,
        "title": "Sunset Photo",
        "ispublic": 1,
        "isfriend": 0,
        "isfamily": 0,
        "url_c": "https://live.staticflickr.com/1234/123456789_abcdef123_c.jpg",
        "height_c": 800,
        "width_c": 600
      }
    ]
  }
}
```

---

## Document Revision History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0.0 | 2025-01-15 | Initial technical documentation | Technical Writer |

---

**End of Technical Documentation**

