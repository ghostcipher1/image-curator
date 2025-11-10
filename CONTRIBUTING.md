# Contributing to Image Curator

Thank you for your interest in contributing to Image Curator! We welcome contributions from the community.

## 🤝 How to Contribute

### Reporting Bugs

If you find a bug, please open an issue with:
- A clear, descriptive title
- Steps to reproduce the problem
- Expected vs actual behavior
- Your environment (OS, Python version)
- Relevant logs or error messages

### Suggesting Features

Feature requests are welcome! Please:
- Check if the feature has already been requested
- Clearly describe the feature and its use case
- Explain why it would be useful to most users

### Code Contributions

1. **Fork the repository** and clone your fork
2. **Create a new branch** for your feature/fix:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes** following our coding standards
4. **Write tests** for new functionality
5. **Run the test suite** to ensure nothing breaks:
   ```bash
   pytest
   ```
6. **Commit your changes** with clear, descriptive messages
7. **Push to your fork** and submit a pull request to the `main` branch

## 📝 Coding Standards

### Python Style
- Follow [PEP 8](https://pep8.org/) style guidelines
- Use type hints for all function signatures
- Write docstrings for all public functions and classes (Google style)
- Keep functions focused and under 50 lines when possible

### Code Organization
- One class per file (exceptions allowed for small helpers)
- Group imports: standard library, third-party, local
- Use absolute imports

### Testing
- Write unit tests for all new features
- Mock external API calls (no real network requests in tests)
- Aim for >80% code coverage
- Test both success and failure cases

### Documentation
- Update README.md if you add features
- Add docstrings to new functions/classes
- Comment complex logic (but prefer self-documenting code)

## 🔌 Adding New Image Sources

To add support for a new image source (e.g., Unsplash, Pexels):

1. **Create a new module** in `image_sources/`:
   ```python
   # image_sources/unsplash.py
   class UnsplashDownloader:
       def __init__(self, api_key: str, config: dict = None):
           self.api_key = api_key
           self.config = config or {}

       def download(self, query: str, num_images: int, output_dir: str) -> None:
           # Implementation
           pass
   ```

2. **Register the provider** in `image_sources/__init__.py`:
   ```python
   PROVIDERS = {
       "flickr": "image_sources.flickr.FlickrDownloader",
       "unsplash": "image_sources.unsplash.UnsplashDownloader",
   }
   ```

3. **Add configuration** to `config.py`:
   ```python
   API_KEYS = {
       "flickr": "your_flickr_key",
       "unsplash": "your_unsplash_key",
   }

   UNSPLASH_CONFIG = {
       "orientation": "landscape",
       # provider-specific options
   }
   ```

4. **Write tests** in `tests/test_unsplash_downloader.py`

5. **Update documentation** in README.md

## 🧪 Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=. --cov-report=html

# Run specific test file
pytest tests/test_flickr_downloader.py -v

# Run with verbose output
pytest -vv
```

## 📋 Pull Request Checklist

Before submitting a PR, ensure:
- [ ] Code follows PEP 8 style guidelines
- [ ] All tests pass locally
- [ ] New features have tests
- [ ] Documentation is updated
- [ ] No sensitive data (API keys, tokens) in commits
- [ ] Commit messages are clear and descriptive
- [ ] PR description explains what and why

## 🚫 What NOT to Contribute

Please avoid:
- Breaking changes without discussion
- Adding dependencies without justification
- Committing API keys or credentials
- Large refactors without prior approval
- Features that violate image source Terms of Service

## 📜 Code of Conduct

### Our Standards

- Be respectful and inclusive
- Welcome constructive feedback
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment, discrimination, or trolling
- Publishing others' private information
- Spamming or off-topic discussions
- Any conduct harmful to the community

## 📞 Questions?

Feel free to:
- Open an issue for questions
- Start a discussion on GitHub Discussions
- Check existing issues and PRs for answers

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for helping make Image Curator better! 🎉
