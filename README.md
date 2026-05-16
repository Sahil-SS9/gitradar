# GitRadar Community Edition

Automated GitHub repository discovery with self-tuning thresholds and quality metrics.

## Overview

GitRadar is an automated pipeline that discovers trending GitHub repositories, filters out noise, and surfaces actionable finds. Originally built for the KENSEI AI agent ecosystem, this community edition makes it available to developers who want to monitor GitHub for:

- New tools and libraries in their stack
- Emerging trends in specific topics (MCP, agent frameworks, etc.)
- High-quality repos worth extracting concepts from
- Potential product ideas or internal tool inspiration

## Features

- **Smart Discovery**: Queries GitHub Search API + scrapes trending page
- **Self-Tuning Thresholds**: Automatically adjusts star requirements based on signal quality
- **Noise Filtering**: Removes tutorial repos, awesome lists, dead repos, and non-code
- **Deduplication**: Avoids processing the same repo multiple times
- **Quality Metrics**: Tracks signal-to-noise ratio over time
- **Hermes Agent Ready**: Outputs JSON compatible with existing cron jobs

## Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/gitradar-community.git
cd gitradar-community

# Install dependencies (none required beyond Python 3.8+ and gh CLI)
pip install -r requirements.txt  # Actually no deps needed

# Ensure you're authenticated to GitHub
gh auth login

# Run discovery once
python3 scripts/github-radar-discover.py

# View results
cat data/discoveries.json
```

## How It Works

GitRadar runs in three stages:

1. **Collection**: Queries GitHub API for repos created in the last 7 days, supplements with trending scrape
2. **Filtering**: Applies rule-based noise filters (awesome lists, tutorials, dead repos, non-code)
3. **Self-Tuning**: After each run, analyzes recent signal quality and adjusts thresholds:
   - If noise is consistently high → increases minimum star requirement
   - If signal is consistently good → decreases minimum star requirement to catch more
   - Output includes tuning decisions for transparency

## Configuration

Thresholds are automatically tuned and saved to `data/thresholds.json`. You can manually adjust:

```json
{
  "star_threshold": 100,
  "min_star_threshold": 25,
  "max_star_threshold": 500,
  "noise_keywords": ["awesome", "curated list", "learn", "tutorial", "list", "resource", "cheatsheet"],
  "language_filters": ["HTML", "CSS", "Markdown"],
  "dead_repo_forks_ratio": 3.0,
  "dead_repo_min_stars": 10,
  // Tuning parameters
  "consecutive_noise_high_days": 3,
  "consecutive_signal_good_days": 3,
  "consecutive_signal_low_days": 5,
  "noise_high_threshold_pct": 40.0,
  "noise_low_threshold_pct": 20.0,
  "signal_high_threshold_pct": 60.0,
  "signal_low_threshold_pct": 10.0,
  "star_adjust_step": 25
}
```

## Output Format

The script outputs JSON to stdout and saves a full copy to `data/discoveries.json`:

```json
{
  "collected_at": "2026-05-16T19:30:12.317159Z",
  "stats": {
    "total_collected": 46,
    "after_filter": 42,
    "after_dedup": 42,
    "collection_queries": 9,
    "noise_rate_pct": 8.7,
    "signal_rate_pct": 91.3,
    "active_threshold": 100
  },
  "filter_reasons": {
    "non_code": 2,
    "dead_repo": 2
  },
  "tuning": {
    "actions": ["HOLD: noise 8.7%, signal 91.3% — thresholds unchanged"],
    "thresholds": {
      "star_threshold": 100,
      "noise_keywords_count": 8,
      "language_filters": ["HTML", "CSS", "Markdown"]
    }
  },
  "repos": [
    {
      "full_name": "owner/repo-name",
      "description": "Repo description",
      "stars": 420,
      "forks": 0,
      "language": "Python",
      "topics": ["topic1", "topic2"],
      "created_at": "2026-05-14T21:29:20Z",
      "pushed_at": "2026-05-14T21:29:26Z",
      "open_issues": 0,
      "license": "MIT",
      "html_url": "https://github.com/owner/repo-name",
      "source": "api"
    }
  ]
}
```

## Integration with Hermes Agent

If you're using Hermes Agent, GitRadar integrates directly with the existing `code-discovery-pipeline` skill:

1. Copy `scripts/github-radar-discover.py` to `~/.hermes/scripts/`
2. Ensure the `code-discovery-pipeline` skill is installed (it's bundled with Hermes)
3. The merged cron job at 08:30 will automatically pick it up
4. Results feed into the research board for classification and kensei-review

## Self-Tuning Explained

GitRadar learns from its own performance:

- **Noise > 40% for 3 consecutive runs** → Star threshold increases by 25
- **Signal > 60% AND Noise < 20% for 3 consecutive runs** → Star threshold decreases by 25
- **Signal < 10% for 5 consecutive runs** → Aggressive increase (threshold +50)

All tuning decisions are logged in the output and saved to `thresholds.json` for inspection.

## Requirements

- Python 3.8+
- GitHub CLI (`gh`) authenticated with `repo` scope
- Internet access to GitHub API

No Python packages required — uses only standard library.

## License

MIT License — feel free to fork, modify, and deploy.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a pull request

Please ensure any changes maintain the deterministic nature of the discovery script.

## Related Projects

- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — The AI agent framework that originally used GitRadar