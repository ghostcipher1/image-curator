# Quick Start Guide

Get up and running with Image Curator in 5 minutes!

## ⚠️ Important: Flickr API Subscription Required

**As of 2024, Flickr API access requires a paid Flickr Pro subscription ($8.25/month or $71.99/year).**

If you don't have a Flickr Pro account, consider:
- **Contributing** a new free provider (Unsplash, Pexels) - see [CONTRIBUTING.md](CONTRIBUTING.md)
- **Waiting** for free provider support (planned for future releases)
- **Subscribing** to Flickr Pro if you need immediate access to Flickr's extensive image library

## 🎯 Prerequisites

- Python 3.8 or higher
- Git
- **Flickr Pro subscription** (required for API access)

## 🚀 Installation (3 steps)

### 1. Clone and Setup

```bash
# Clone the repository
git clone https://github.com/ghostcipher1/image-curator.git
cd image-curator

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Get Your Flickr API Key

**Step 1: Subscribe to Flickr Pro**
1. Visit [Flickr Pro](https://www.flickr.com/account/upgrade/pro)
2. Choose a subscription plan:
   - Monthly: $8.25/month
   - Annual: $71.99/year (best value)
3. Complete the subscription process

**Step 2: Request API Key**
1. Go to https://www.flickr.com/services/apps/create/
2. Click "Request an API Key"
3. Choose:
   - **Non-Commercial Key** for personal use
   - **Commercial Key** for business use
4. Fill out the form:
   - App name: "Image Curator"
   - App description: "Personal image collection and curation tool"
5. Copy your **API Key** (the long string of letters and numbers)

### 3. Configure Your API Key

**Option A:** Edit `config.py`
```python
API_KEYS = {
    "flickr": "paste_your_api_key_here"  # Replace with actual key
}
```

**Option B:** Create a `.env` file
```bash
echo "FLICKR_API_KEY=your_actual_api_key_here" > .env
```

## ✨ First Download

### Try it out with default settings:

```bash
python main.py
```

This will download 10 images each for:
- Sunsets
- Mountains
- Wildlife

Images will be saved in `downloads/` folder organized by category.

### Customize your download:

```bash
# Download 20 cat photos
python main.py --categories "cats" --limit 20

# Download multiple categories
python main.py --categories "dogs,birds,flowers" --limit 15

# Only Creative Commons images
python main.py --license 4 --limit 10

# Save to custom directory
python main.py --out my_images
```

## 🎨 Customize Categories

Edit `config.py` to set your favorite categories:

```python
CATEGORIES = ["landscapes", "architecture", "street photography"]
NUM_IMAGES = 25
OUTPUT_DIR = "my_collection"
```

Then just run:
```bash
python main.py
```

## 🔍 Available CLI Options

```bash
python main.py --help
```

Key options:
- `--categories "cat1,cat2"` - Comma-separated categories
- `--limit N` - Number of images per category
- `--out DIR` - Output directory
- `--license {any,4,5,6,9,10}` - License filter (Creative Commons, Public Domain)
- `--safe_search {1,2,3}` - Safe search level (1=safe, 2=moderate, 3=restricted)
- `--verbose` - Show detailed logging

## 📊 License Filters

Use `--license` to filter by image license:

| Value | License Type | Usage Rights |
|-------|-------------|--------------|
| `any` | All licenses (default) | Mixed - check individual licenses |
| `4` | CC BY | Free with attribution |
| `5` | CC BY-SA | Free with attribution, share-alike |
| `6` | CC BY-ND | Free with attribution, no derivatives |
| `9` | CC0 | Public domain dedication |
| `10` | Public domain | No restrictions |

**Example - Download only public domain images:**
```bash
python main.py --categories "nature,history" --license 9 --limit 50
```

## 🧪 Verify Installation

Run the test suite:

```bash
pytest
```

You should see: `32 passed in X.XXs ✅`

## 📁 Output Structure

After running, your folder structure will look like:

```
image-curator/
├── downloads/              # Default output directory
│   ├── sunsets/
│   │   ├── sunsets_001__12345678.jpg
│   │   ├── sunsets_002__23456789.jpg
│   │   └── ...
│   ├── mountains/
│   │   └── ...
│   └── wildlife/
│       └── ...
├── config.py
├── main.py
└── ...
```

Filename format: `{category}_{index}__{flickr_photo_id}.{ext}`

## ⚠️ Important Notes

### API Rate Limits
- Flickr allows **3600 API calls per hour** per API key
- Each category search = 1 API call
- Each image download = 1 HTTP request (not counted against API limit)
- For large downloads (1000+ images), pace your requests

### Cost Considerations
- Flickr Pro: $8.25/month or $71.99/year
- No additional API usage fees
- Unlimited downloads within rate limits
- Consider the subscription cost vs. value for your use case

### Legal & Ethical Use
- **Always check image licenses** before using downloaded images
- **Give proper attribution** when required by Creative Commons licenses
- **Respect photographers' rights** - don't claim others' work as your own
- **Follow Flickr's Terms of Service**: https://www.flickr.com/tos
- Don't use for commercial purposes without proper licenses

### Best Practices
- Start with small batches (10-20 images) to test your setup
- Use specific, descriptive category names for better search results
- Use `--verbose` flag to see detailed progress and debug issues
- Check the `downloads/` folder after each run
- Regularly review your Flickr Pro subscription value

## 🐛 Troubleshooting

### "No API key found for flickr"
**Solution:** Make sure you've set the API key in `config.py` or `.env` file
```bash
# Check if .env exists and contains your key
cat .env
# Should show: FLICKR_API_KEY=your_key_here
```

### "API error: Invalid API Key"
**Possible causes:**
- API key is incorrect (copy-paste error)
- API key has been revoked or expired
- Flickr Pro subscription has expired

**Solution:**
1. Verify your Flickr Pro subscription is active
2. Check your API key at https://www.flickr.com/services/api/keys/
3. Generate a new API key if needed

### "No photos found for query"
**Possible causes:**
- Search term is too specific or obscure
- License filter is too restrictive
- Combination of filters returns no results

**Solution:**
- Try broader search terms
- Use `--license any` to see if any results exist
- Adjust `--safe_search` level
- Check Flickr.com directly to verify photos exist

### "Request failed" or timeout errors
**Possible causes:**
- Internet connection issues
- Flickr API is temporarily down
- Rate limit exceeded

**Solution:**
- Check your internet connection
- Wait a few minutes and retry
- Check Flickr API status: https://www.flickr.com/help/api

### Connection/HTTP errors
**Solution:** The tool automatically retries with exponential backoff. If issues persist:
- Check your firewall settings
- Try a different network
- Reduce `--limit` to download fewer images at once

## 💡 Example Use Cases

### Build a machine learning dataset:
```bash
python main.py \
  --categories "cat,dog,bird,fish,horse" \
  --limit 1000 \
  --license 9 \
  --out ml_training_data \
  --safe_search 1
```

### Collect wallpapers for personal use:
```bash
python main.py \
  --categories "landscapes,space,abstract art,nature" \
  --limit 50 \
  --safe_search 1 \
  --out wallpapers
```

### Create a Creative Commons photo library:
```bash
python main.py \
  --categories "technology,business,people,cities" \
  --limit 200 \
  --license 4 \
  --out cc_photo_library
```

### Curate reference images for design work:
```bash
python main.py \
  --categories "typography,color palettes,minimalism,architecture" \
  --limit 100 \
  --license any \
  --out design_references
```

## 🚀 Next Steps

1. **Read the full [README.md](README.md)** for comprehensive documentation
2. **Check [CONTRIBUTING.md](CONTRIBUTING.md)** to add support for free image sources
3. **Explore the code** in `image_sources/flickr.py` to understand the implementation
4. **Star the repo** on GitHub if you find it useful!

## 🆓 Want Free Alternatives?

We're planning to add support for free image sources. You can help by:
1. **Contributing** a new provider implementation (Unsplash, Pexels, Pixabay)
2. **Opening an issue** to request specific sources
3. **Starring the repo** to stay updated on new features

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on adding new sources.

## 🎉 You're Ready!

Start curating your image collection:

```bash
python main.py --categories "your,favorite,topics" --limit 20 --verbose
```

Happy curating! 📸

---

**Questions?** Open an issue on GitHub: https://github.com/ghostcipher1/image-curator/issues
