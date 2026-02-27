# reddit-research

A research reference covering Reddit platform features, API capabilities, community dynamics, and use cases for research and data analysis.

---

## Table of Contents

1. [Platform Overview](#platform-overview)
2. [Core Features](#core-features)
3. [Content Architecture](#content-architecture)
4. [Voting & Karma System](#voting--karma-system)
5. [Awards & Coins](#awards--coins)
6. [Reddit API](#reddit-api)
7. [Data Access & Scraping](#data-access--scraping)
8. [Moderation & Community Guidelines](#moderation--community-guidelines)
9. [Advertising & Business Model](#advertising--business-model)
10. [User Demographics & Statistics](#user-demographics--statistics)
11. [Research Use Cases](#research-use-cases)
12. [Key Libraries & Tools](#key-libraries--tools)
13. [References](#references)

---

## Platform Overview

Reddit is a social news aggregation, content rating, and discussion website. Founded in 2005, it is organised into user-created communities called **subreddits**, each dedicated to a topic, theme, or interest. Registered users submit content (links, images, text posts, videos, polls) and engage through nested comments. Popularity is determined by a community-driven **upvote/downvote** voting system.

| Statistic | Value (approx.) |
|---|---|
| Monthly active users | ~1.5 billion (2024) |
| Daily active users | ~70 million |
| Total subreddits | ~3.4 million (active: ~130,000) |
| Posts per day | ~1.5 million |
| Comments per day | ~4.2 million |

---

## Core Features

### Subreddits
- User-created community spaces prefixed with `r/` (e.g., `r/Python`, `r/worldnews`).
- Each subreddit has its own rules, flair system, post types, and moderation team.
- Subreddit types: **public**, **restricted** (post by approval), **private** (view/post by invite), **archived**, **quarantined**, and **banned**.
- Moderators can customise appearance with banners, icons, sidebar widgets, and custom CSS (New Reddit uses "Mod Tools").

### Posts (Submissions)
| Post Type | Description |
|---|---|
| Link | External URL shared with the community |
| Text (self-post) | Body text written in Markdown |
| Image/Gallery | Single image or multi-image carousel |
| Video | Native video or YouTube/Vimeo embed |
| Poll | Multiple-choice voting (24 h – 7 days) |
| Crosspost | Reposts a post from another subreddit |

- Posts can be assigned **flair** (text/emoji tags) configured per subreddit.
- Authors can mark posts **NSFW**, **Spoiler**, or **OC** (Original Content).
- Subreddit moderators can **pin** up to two posts (announcement/sticky) and add a moderator-level sticky comment.

### Comments
- Threaded (nested) comment trees; depth can technically be unlimited but Reddit collapses deep threads.
- Markdown and Reddit-flavoured syntax: `**bold**`, `*italic*`, `[link](url)`, `>quote`, code blocks, tables.
- Comment sorting options: **Best**, **Top**, **New**, **Controversial**, **Old**, **Q&A**.
- Moderators can lock, remove, distinguish (mark as mod), or approve comments.

### User Profiles
- Customisable avatar, banner, bio, and display name.
- Public activity feed shows posts and comments (unless hidden by the user).
- **Trophy case** displays site-wide awards and achievements.
- Reddit Premium subscribers have an animated icon and `[A]` badge.

---

## Content Architecture

```
Reddit
└── Subreddits (r/<name>)
    └── Posts (link / self / image / video / poll)
        ├── Post metadata (author, score, flair, awards, NSFW flag…)
        └── Comment tree
            └── Comment
                ├── Comment metadata (author, score, awards…)
                └── Nested reply thread
```

### IDs & Fullnames
Reddit uses **base-36 IDs** for all objects. A **fullname** combines a type prefix with the ID:

| Prefix | Object type |
|---|---|
| `t1_` | Comment |
| `t2_` | Account |
| `t3_` | Link (post) |
| `t4_` | Message |
| `t5_` | Subreddit |
| `t6_` | Award |

Example: post `t3_abc123` → ID `abc123`.

---

## Voting & Karma System

- Each post and comment can receive **upvotes** (+1) and **downvotes** (−1) from logged-in users.
- The displayed **score** is `upvotes − downvotes`, fuzzed slightly to deter vote manipulation bots.
- **Upvote ratio** (percentage of upvotes among all votes) is exposed in the API.
- **Karma** is an aggregated score shown on a user's profile:
  - **Post karma** – sum of net upvotes on posts.
  - **Comment karma** – sum of net upvotes on comments.
  - Karma does **not** map 1-to-1 to votes; Reddit applies an undisclosed algorithm that caps per-post karma gain.
- Subreddits may impose a **minimum karma threshold** to post or comment (spam prevention).

### Hot Ranking Algorithm
Reddit's hot-ranking formula (open-sourced) computes a post's rank based on score and submission time:

```
score   = upvotes − downvotes
order   = log10(max(abs(score), 1))
sign    = 1 if score > 0, -1 if score < 0, else 0
seconds = epoch_seconds(created_utc) − 1134028003
rank    = sign × order + seconds / 45000
```

Other ranking algorithms: **New** (pure chronological), **Top** (all-time/period net score), **Rising** (new posts with fast velocity), **Controversial** (high vote counts with near-equal up/down split).

---

## Awards & Coins

- **Reddit Coins** are a virtual currency purchased with real money.
- Users spend coins to award posts/comments with various tiers:
  | Award | Coins | Effect |
  |---|---|---|
  | Silver | 100 | No special effect |
  | Gold | 500 | 1 week Reddit Premium for recipient |
  | Platinum | 1800 | 1 month Reddit Premium for recipient |
  | Community awards | Varies | Subreddit-specific, configured by mods |
- Awarded posts/comments display award icons and receive a small karma boost.
- Reddit has periodically restructured its awards system (e.g., introducing "Collectible Avatars" NFTs in 2022, later winding down coin sales in 2023 in favour of a simplified system).

---

## Reddit API

### Authentication
Reddit uses **OAuth 2.0**. Three main grant types are used by third-party apps:

| Grant Type | Use Case |
|---|---|
| `password` (script) | Server-side scripts acting as a single user |
| `client_credentials` | Read-only anonymous access |
| `authorization_code` | Web/mobile apps acting on behalf of users |

Base URL: `https://oauth.reddit.com` (authenticated) or `https://www.reddit.com` (unauthenticated JSON endpoints).

### Rate Limits
- Authenticated apps: **100 requests / 10 minutes** per OAuth token.
- Response headers include `X-Ratelimit-Remaining`, `X-Ratelimit-Reset`, `X-Ratelimit-Used`.
- As of July 2023, Reddit significantly increased API pricing for third-party apps, leading to significant ecosystem changes.

### Key Endpoints

| Endpoint | Description |
|---|---|
| `GET /r/{subreddit}/hot` | Hot posts for a subreddit |
| `GET /r/{subreddit}/new` | Newest posts |
| `GET /r/{subreddit}/top?t={period}` | Top posts (hour/day/week/month/year/all) |
| `GET /r/{subreddit}/comments/{id}` | Post + comment tree |
| `GET /r/{subreddit}/about` | Subreddit metadata |
| `GET /user/{username}/submitted` | A user's posts |
| `GET /user/{username}/comments` | A user's comments |
| `GET /search?q={query}&type={type}` | Site-wide search |
| `POST /api/submit` | Submit a new post |
| `POST /api/comment` | Submit a new comment |
| `POST /api/vote` | Cast a vote |

### Pagination
Reddit uses **cursor-based pagination** via `after` and `before` parameters (fullnames), combined with a `limit` parameter (max 100).

```
GET /r/Python/new?limit=100&after=t3_abc123
```

### Pushshift (Historical Data)
[Pushshift](https://pushshift.io) was a third-party archive of Reddit data that allowed bulk historical queries. In 2023 Reddit restricted API access and Pushshift lost its access. Pushshift is **no longer a reliable source** of recent Reddit data. Researchers now must use the official API or purchase data from Reddit's [Data API](https://www.redditinc.com/policies/data-api-terms).

---

## Data Access & Scraping

### Official Methods
| Method | Notes |
|---|---|
| Reddit API (OAuth) | Rate-limited; best for real-time or moderate-volume data |
| Reddit Data API | Paid bulk data access for research/commercial use |
| `.json` suffix | Append `.json` to any Reddit URL for a raw JSON response (unauthenticated, aggressively rate-limited) |

### PRAW (Python Reddit API Wrapper)
```python
import praw

reddit = praw.Reddit(
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET",
    user_agent="my-research-bot/1.0 by u/YourUsername",
)

for submission in reddit.subreddit("Python").hot(limit=25):
    print(submission.title, submission.score)
```

### asyncpraw
Async version of PRAW for high-throughput collection using `asyncio`.

### RedditWarp
A lower-level Python library providing more granular API control than PRAW.

### PSAW / PMAW (Pushshift wrappers)
These libraries targeted the Pushshift API; they are **largely deprecated** due to Pushshift's restricted access.

---

## Moderation & Community Guidelines

### Content Policy
Reddit's site-wide **Content Policy** prohibits:
- Spam
- Impersonation of users or public figures
- Involuntary pornography
- Posting of personal/private information (doxxing)
- Content sexualising minors
- Inciting violence or harassment against specific individuals/groups
- Vote manipulation or site interference

Full policy: <https://www.redditinc.com/policies/content-policy>

### Subreddit-Level Moderation Tools
- **AutoModerator** – rule-based bot that can automatically remove/approve/flair content based on configurable YAML rules (e.g., keyword filters, account age, karma thresholds).
- **Mod queue** – posts and comments awaiting moderator review (reported or filtered by AutoModerator).
- **Modmail** – private messaging system between users and a subreddit's mod team.
- **Mod log** – audit trail of all moderator actions.
- **Contributor/banned lists** – approved submitters list (for restricted subs) and ban management.

### Reddit Admin Actions
Site-wide administrators can quarantine, ban, or remove entire subreddits. Notable mass bans include the 2020 "Hate Speech" ban wave (2,000+ subreddits including r/The_Donald).

---

## Advertising & Business Model

Reddit generates revenue through:
1. **Reddit Ads** – self-serve CPC/CPM advertising platform targeting by subreddit, interest, location, device type, etc.
2. **Reddit Premium** – subscription ($5.99/month or $49.99/year) granting an ad-free experience, access to r/lounge, and a monthly coin allowance.
3. **Data licensing** – selling access to Reddit's data corpus to AI/ML companies and researchers (significantly expanded post-2023 IPO).
4. **Collectible Avatars** – limited-edition NFT-style avatars sold on the Polygon blockchain (introduced 2022).

Reddit went public on the **New York Stock Exchange (NYSE: RDDT)** on **21 March 2024**, pricing its IPO at $34 per share.

---

## User Demographics & Statistics

| Category | Data |
|---|---|
| Global rank (Similarweb) | ~9th most visited website |
| Primary age group | 18–29 (largest), 30–49 (second) |
| Gender split | ~64% male, ~36% female (US, 2023) |
| Top countries by traffic | United States, United Kingdom, Canada, Australia, Germany |
| Average time on site | ~10–11 minutes per visit |
| Languages | English dominant (~50%); active communities in German, French, Spanish, Portuguese, etc. |

---

## Research Use Cases

Reddit has been extensively used in academic and industry research:

| Domain | Example |
|---|---|
| Natural Language Processing | Sentiment analysis, sarcasm detection, hate speech classification |
| Social network analysis | Community detection, information diffusion, echo chambers |
| Mental health research | Depression/anxiety signals in r/depression, r/SuicideWatch (with IRB approval) |
| Public health | COVID-19 misinformation tracking, vaccine hesitancy |
| Computational social science | Political polarisation, opinion dynamics |
| Recommender systems | Collaborative filtering using upvote behaviour |
| Crisis informatics | Disaster event detection via real-time posts |

### Notable Datasets
| Dataset | Description |
|---|---|
| Reddit Comments Corpus (2005–2023) | ~1 TB+ compressed; hosted via Academic Torrents/Internet Archive |
| RedditBias | Dataset annotated for demographic and other biases |
| TalkDown | Condescension detection corpus sourced from Reddit |
| ChangeMyView (r/changemyview) | Argumentation and persuasion research dataset |
| SocialGrep / Arctic Shift | Post-Pushshift Reddit data access alternatives |

### Ethical Considerations
- Treat public Reddit data as potentially sensitive; users did not necessarily consent to research use.
- Follow the **APA, ACM, and ACL** guidelines on ethical NLP/computational social science research.
- Obtain IRB/Ethics Board approval for studies involving mental health communities or vulnerable populations.
- Anonymise or aggregate data to prevent re-identification of individual users.
- Respect Reddit's [Data API Terms of Service](https://www.redditinc.com/policies/data-api-terms).

---

## Key Libraries & Tools

| Library / Tool | Language | Purpose |
|---|---|---|
| [PRAW](https://praw.readthedocs.io) | Python | Official Reddit API wrapper |
| [asyncpraw](https://asyncpraw.readthedocs.io) | Python | Async Reddit API wrapper |
| [RedditWarp](https://github.com/Pyprohly/redditwarp) | Python | Low-level Reddit API client |
| [snoowrap](https://github.com/not-an-aardvark/snoowrap) | JavaScript | Reddit API wrapper for Node.js |
| [Arctic Shift](https://github.com/ArthurHeitmann/arctic_shift) | Various | Pushshift replacement for bulk Reddit data |
| [SocialGrep](https://socialgrep.com) | Web | Reddit search & data export tool |
| [Reddit Insight](https://www.redditinsight.com) | Web | User and subreddit analytics |
| [Subreddit Stats](https://subredditstats.com) | Web | Subreddit growth and activity statistics |

---

## References

- Reddit Official Blog: <https://www.redditinc.com/blog>
- Reddit API Documentation: <https://www.reddit.com/dev/api/>
- PRAW Documentation: <https://praw.readthedocs.io>
- Reddit Content Policy: <https://www.redditinc.com/policies/content-policy>
- Reddit Data API Terms: <https://www.redditinc.com/policies/data-api-terms>
- Baumgartner, J. et al. (2020). *The Pushshift Reddit Dataset*. ICWSM.
- Reddit IPO S-1 Filing (2024): <https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=RDDT>
