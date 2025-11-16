# Reddit Integration Guide

## Overview

TrendRadar now supports Reddit as a news source! The Reddit crawler fetches hot posts from r/all to provide international trending content alongside Chinese news platforms.

## Features

- **Source**: r/all hot posts (top 50)
- **Format**: Compatible with TrendRadar's existing data structure
- **Authentication**: No API key required (uses public JSON endpoint)
- **Integration**: Seamlessly works with keyword filtering and notifications

## Configuration

### Enable Reddit in config.yaml

Reddit is included by default in the platforms list:

```yaml
platforms:
  - id: "toutiao"
    name: "今日头条"
  # ... other platforms ...
  - id: "reddit"
    name: "Reddit热门"
```

### Optional: Disable Reddit

To disable Reddit, simply remove or comment out the Reddit entry:

```yaml
platforms:
  # - id: "reddit"
  #   name: "Reddit热门"
```

## How It Works

### Data Source

The Reddit crawler fetches data from:
- **Endpoint**: `https://reddit.com/r/all/hot.json?limit=50`
- **Subreddit**: r/all (all of Reddit's popular content)
- **Post Count**: Top 50 hot posts (excluding stickied posts)

### Data Processing

1. **Fetch**: Retrieves JSON data from Reddit's public API
2. **Parse**: Extracts title and permalink from each post
3. **Filter**: Removes stickied/pinned posts
4. **Format**: Converts to NewsNow-compatible structure
5. **Integrate**: Merges with other platform data

### User-Agent

Reddit requires a descriptive User-Agent string:
- **Format**: `python:TrendRadar:<version> (by /u/TrendRadarBot)`
- **Compliance**: Follows Reddit API guidelines

## Troubleshooting

### 403 Forbidden Error

If you encounter HTTP 403 errors when crawling Reddit:

**Cause**: Reddit may block requests from certain server/datacenter IP addresses.

**Solutions**:

1. **Enable Proxy** (Recommended):
   ```yaml
   crawler:
     use_proxy: true
     default_proxy: "http://127.0.0.1:10086"
   ```

2. **Run Locally**: Deploy on your personal computer instead of cloud servers

3. **Use Different Network**: GitHub Actions may work if local IP is blocked

4. **Check Logs**: Look for specific error messages in crawler output

### Rate Limiting

Reddit has rate limits for unauthenticated requests:
- **Limit**: ~60 requests per minute
- **TrendRadar Impact**: Minimal (only 1 request per crawl cycle)
- **Solution**: Default request interval (1000ms) is sufficient

### Empty Results

If Reddit returns 0 posts:
- Check if r/all is accessible in your region
- Verify network connectivity
- Review proxy settings if enabled
- Check Reddit's status page: https://www.redditstatus.com/

## Technical Details

### Implementation

**Location**: `main.py:440-510`

**Class**: `DataFetcher`

**Method**: `fetch_reddit_data()`

**Key Code**:
```python
def fetch_reddit_data(self, id_value: str, alias: str) -> Tuple[Optional[str], str, str]:
    """获取Reddit数据，使用Reddit公开JSON API"""
    url = "https://reddit.com/r/all/hot.json?limit=50"

    headers = {
        "User-Agent": f"python:TrendRadar:{VERSION} (by /u/TrendRadarBot)",
        "Accept": "application/json",
    }

    # Fetch and parse Reddit data
    # Convert to NewsNow format
    # Return standardized structure
```

### Data Structure

**Reddit API Response**:
```json
{
  "data": {
    "children": [
      {
        "data": {
          "title": "Post title",
          "permalink": "/r/subreddit/comments/...",
          "stickied": false
        }
      }
    ]
  }
}
```

**Converted to NewsNow Format**:
```json
{
  "status": "success",
  "items": [
    {
      "title": "Post title",
      "url": "https://www.reddit.com/r/subreddit/comments/...",
      "mobileUrl": "https://www.reddit.com/r/subreddit/comments/..."
    }
  ]
}
```

## Limitations

1. **Language**: Most Reddit content is in English
   - May not match Chinese frequency keywords well
   - Consider adding English keywords to `frequency_words.txt`

2. **Subreddit**: Currently fixed to r/all
   - Future enhancement: Make subreddit configurable
   - Potential: Support multiple subreddits

3. **Authentication**: Uses public endpoints
   - No OAuth implementation
   - Subject to public API rate limits

4. **IP Restrictions**: Some environments may be blocked
   - Datacenter IPs often restricted
   - Proxy recommended for server deployments

## Future Enhancements

Potential improvements for Reddit integration:

- [ ] Configurable subreddit selection (r/worldnews, r/technology, etc.)
- [ ] Multiple subreddit support
- [ ] OAuth authentication for higher rate limits
- [ ] Score/upvote filtering (e.g., only posts with >1000 upvotes)
- [ ] Subreddit-specific configuration in config.yaml
- [ ] Time range filtering (hot, new, top-hour, top-day)
- [ ] NSFW filtering options

## FAQ

**Q: Do I need a Reddit account?**
A: No, the current implementation uses public endpoints without authentication.

**Q: Can I customize which subreddit to crawl?**
A: Currently no. The implementation is hardcoded to r/all. This is planned for future updates.

**Q: Why am I getting 403 errors?**
A: Reddit blocks many datacenter IPs. Try enabling a proxy in config.yaml or run locally.

**Q: How often does it crawl Reddit?**
A: Same as other platforms - every hour by default in GitHub Actions, or per your cron schedule.

**Q: Does this affect my Reddit account?**
A: No, this doesn't interact with your Reddit account at all. It's read-only public data.

## Contributing

Have ideas for improving Reddit integration?

1. Check existing issues: https://github.com/sansan0/TrendRadar/issues
2. Open a feature request
3. Submit a pull request

## References

- **Reddit JSON API**: https://www.reddit.com/dev/api
- **Reddit Status**: https://www.redditstatus.com/
- **TrendRadar Main Docs**: ../readme.md
- **Configuration Guide**: ../CLAUDE.md

---

**Last Updated**: 2025-11-15
**Version**: 3.0.5+reddit
**Maintainer**: TrendRadar Community
