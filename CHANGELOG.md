# Changelog

All notable changes to Image Curator will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Unsplash provider support (free API)
- Pexels provider support (free API)
- Pixabay provider support
- Image dimension filtering
- Color palette filtering
- Parallel downloads
- Resume interrupted downloads
- Docker support

## [0.1.0] - 2025-01-10

### Added
- Initial release of Image Curator
- Flickr API integration with full search capabilities
- Modular plugin architecture for multiple image sources
- CLI interface with comprehensive options
- Smart filtering by license type (Creative Commons, Public Domain)
- Safe search and content type filters
- Automatic retry logic with exponential backoff
- Progress tracking with tqdm progress bars
- Per-category subdirectories with sanitized filenames
- Comprehensive test suite (32+ tests)
- Full type hints and logging
- GitHub Actions CI/CD workflows
- PyPI packaging and distribution
- Complete documentation (README, QUICKSTART, CONTRIBUTING)

### Features
- **Multi-source architecture**: Extensible plugin system
- **Flickr support**: Search and download with license filtering
- **Robust downloads**: Retry logic, error handling, rate limiting
- **CLI tools**: argparse-based command-line interface
- **Testing**: Full unit test coverage with mocked API calls
- **CI/CD**: GitHub Actions for testing, releases, and PyPI publishing

### Configuration
- Config file support (config.py)
- Environment variable support (.env)
- CLI argument overrides
- Flexible API key management

### Documentation
- Comprehensive README with examples
- Quick start guide (QUICKSTART.md)
- Contribution guidelines (CONTRIBUTING.md)
- MIT License

### Technical
- Python 3.8+ support
- Type hints throughout
- PEP 8 compliant
- Modular, maintainable code structure

---

## Release Process

### Creating a New Release

1. Update version in `pyproject.toml`
2. Update this CHANGELOG.md with new version and date
3. Commit changes: `git commit -am "Release vX.Y.Z"`
4. Create git tag: `git tag -a vX.Y.Z -m "Release vX.Y.Z"`
5. Push with tags: `git push origin main --tags`
6. GitHub Actions will automatically:
   - Run tests
   - Create GitHub release
   - Build and publish to PyPI

### Version Numbering

- **Major (X.0.0)**: Breaking changes, major new features
- **Minor (0.X.0)**: New features, backward compatible
- **Patch (0.0.X)**: Bug fixes, minor improvements

### Examples
- `v1.0.0`: First stable release with Unsplash and Pexels support
- `v0.2.0`: Add Unsplash provider
- `v0.1.1`: Bug fixes for Flickr downloads

---

## Migration Guides

### From Pre-release to 0.1.0

If you were using development versions:
- Update import paths if changed
- Check config.py for new options
- Review API key setup (now supports .env)

### Future Migrations

Will be documented here when breaking changes are introduced.

---

## Contributors

Thank you to all contributors who help improve Image Curator!

- @ghostcipher1 - Creator and maintainer

Want to contribute? See [CONTRIBUTING.md](CONTRIBUTING.md)!
