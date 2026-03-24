# Flipboard

Flipboard is a content curation and social magazine platform that aggregates articles, videos, and social media posts into personalized, magazine-style feeds. Founded in 2010, Flipboard allows users to curate content into magazines organized around topics of interest and follow curators and publishers. The platform serves both consumers seeking personalized news and publishers looking to distribute content to engaged readers.

Flipboard has embraced open web standards as a core part of its strategy. The company implemented ActivityPub to federate with the Fediverse, allowing Flipboard magazines and curators to be followed from Mastodon, Threads, Pixelfed, and other decentralized social platforms. In late 2024, Flipboard launched Surf, a companion app designed as a browser for the open social web that aggregates content across ActivityPub, AT Protocol (Bluesky), and RSS feeds.

**URL:** [https://flipboard.com/](https://flipboard.com/)

## APIs

### RSS Feeds for Publishers

Flipboard's primary integration mechanism for publishers is RSS. Publishers submit RSS feeds to be featured as Flipboard Magazines, distributing content to Flipboard's audience. Feeds must conform to Flipboard's RSS guidelines: full article body content (not just summaries), at least one image per article at 400px minimum width, at least 30 items in the feed, and updates pushed via PubSubHubbub (Superfeedr is recommended). Flipboard provides a feed validator tool at feedvalidator.flipboard.com to help publishers verify compliance before submission.

- **RSS Guidelines:** [https://about.flipboard.com/rss-guidelines/](https://about.flipboard.com/rss-guidelines/)
- **For Publishers:** [https://about.flipboard.com/forpublishers/](https://about.flipboard.com/forpublishers/)
- **Feed Validator:** [https://feedvalidator.flipboard.com/](https://feedvalidator.flipboard.com/)
- **Submit Publisher Application:** [https://flipboard.helpshift.com/hc/en/1-flipboard/faq/1024-submit-a-publisher-application/](https://flipboard.helpshift.com/hc/en/1-flipboard/faq/1024-submit-a-publisher-application/)

### ActivityPub / Fediverse Federation

Flipboard implemented ActivityPub in late 2023 and has built its entire architecture around the protocol, making it one of the most deeply federated mainstream social platforms. Flipboard magazines and curators are available to any ActivityPub-compatible Fediverse platform. By mid-2024, Flipboard had federated over 700 curators and publishers with more than 15,000 magazines accessible across the Fediverse. Flipboard's ActivityPub implementation is open source and available on GitHub.

- **ActivityPub Implementation (GitHub):** [https://github.com/Flipboard/activitypub](https://github.com/Flipboard/activitypub)
- **Federation Announcement:** [https://about.flipboard.com/inside-flipboard/flipboard-begins-to-federate/](https://about.flipboard.com/inside-flipboard/flipboard-begins-to-federate/)
- **Publisher Federation Guide:** [https://about.flipboard.com/business/publisher-federation-flipboard/](https://about.flipboard.com/business/publisher-federation-flipboard/)
