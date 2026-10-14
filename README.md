# Vicrix
Vicrix is the leading digital directory for viral social media creators. Explore detailed influencer bios, engagement stats, and viral highlights.
# Vicrix — Technical Architecture & Complete Implementation Blueprint

> **Vicrix is an API-first creator discovery platform that unifies creator profiles, social media statistics, and public content into one lightweight, searchable directory.**

Vicrix is designed to minimize database storage while delivering fast, reliable, and SEO-friendly creator profiles. It stores essential creator identifiers and manually maintained profile information, retrieves supported public data through external APIs, and uses caching to reduce redundant requests.

**Core vision:** One directory. Multiple platforms. A unified creator experience.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Core Architecture](#2-core-architecture)
3. [Technology Stack](#3-technology-stack)
4. [Database Design](#4-database-design)
5. [Environment Configuration](#5-environment-configuration)
6. [Social API Integrations](#6-social-api-integrations)
7. [Unified Creator API](#7-unified-creator-api)
8. [Dynamic Creator Profile Pages](#8-dynamic-creator-profile-pages)
9. [Caching and Performance](#9-caching-and-performance)
10. [SEO and Structured Data](#10-seo-and-structured-data)
11. [Security and Reliability](#11-security-and-reliability)
12. [Complete Project Structure](#12-complete-project-structure)
13. [Complete Implementation Files](#13-complete-implementation-files)
14. [Development and Testing](#14-development-and-testing)
15. [Deployment Guide](#15-deployment-guide)
16. [Implementation Roadmap](#16-implementation-roadmap)
17. [Future Enhancements](#17-future-enhancements)
18. [Production Checklist](#18-production-checklist)

---

# 1. Project Overview

## 1.1 What Is Vicrix?

Vicrix is a lightweight, API-first creator directory that enables users to discover creators and explore their public presence across supported social media platforms.

Each creator receives a dedicated profile page containing available profile information, social statistics, official links, and selected public content.

Rather than storing complete social media feeds or media files locally, Vicrix retrieves supported data from external sources and caches responses where permitted.

## 1.2 Project Objectives

- **Minimal database storage:** Maintain essential creator identifiers and profile configuration.
- **Dynamic data aggregation:** Retrieve available information from supported social APIs.
- **Fast page delivery:** Use server rendering and caching.
- **Search engine optimization:** Generate crawlable pages, unique metadata, and structured data.
- **Modular architecture:** Integrate additional social platforms independently.
- **Reliable rendering:** Continue displaying available information when an external service fails.
- **Scalable infrastructure:** Support directory growth without storing unnecessary content.

## 1.3 Core Features

| Feature | Description |
|---|---|
| Creator Directory | Browse registered creators |
| Creator Profiles | Individual public profile pages |
| Social Statistics | Display available subscribers, followers, and views |
| Video Highlights | Embed supported public videos |
| Creator Search | Search by name, slug, or platform |
| Featured Creators | Highlight selected profiles |
| Unified API | Normalize platform responses |
| SEO Metadata | Generate page titles and descriptions |
| Structured Data | Provide appropriate Schema.org markup |
| Smart Caching | Reduce redundant API requests |
| Responsive Interface | Support desktop and mobile devices |

## 1.4 Design Philosophy

Vicrix should remain small, modular, and maintainable.

The database is the source of truth for creator identity and manually maintained information. External APIs are the source of truth for the statistics they expose. Cached responses improve performance but must not be presented as guaranteed real-time data.

---

# 2. Core Architecture

## 2.1 High-Level Architecture

```text
                  USER / SEARCH ENGINE
                           |
                           v
                   NEXT.JS APP ROUTER
                           |
              +------------+------------+
              |                         |
              v                         v
        HOME / DIRECTORY          CREATOR PROFILE
              |                         |
              +------------+------------+
                           |
                           v
                   SERVER DATA LAYER
                           |
              +------------+------------+
              |                         |
              v                         v
        SUPABASE DATABASE         CACHE LAYER
              |                         |
              +------------+------------+
                           |
                           v
                  SOCIAL API SERVICES
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          YOUTUBE       INSTAGRAM      FACEBOOK
          DATA API      GRAPH API      GRAPH API
             |             |             |
             +-------------+-------------+
                           |
                           v
                   DATA NORMALIZATION
                           |
                           v
                  UNIFIED CREATOR DATA
                           |
                           v
                  SERVER-RENDERED HTML
                           |
                           v
                    USER / GOOGLE BOT
```

## 2.2 Request Lifecycle

When a user opens `/creator/example-creator`:

1. Next.js receives the request.
2. The server retrieves the creator record using its slug.
3. If the creator does not exist, the application returns a not-found page.
4. The integration layer retrieves available social data.
5. Cached responses are reused when valid.
6. Responses are normalized into a consistent data structure.
7. Next.js renders the creator page and its metadata.
8. The browser receives the page with the available profile information.

## 2.3 Core Architectural Rules

**Database:** Store creator identifiers, profile details, and editorial configuration.

**API layer:** Keep platform-specific requests inside independent service modules.

**Caching:** Cache reusable public data instead of repeating external requests.

**Rendering:** Render essential creator information on the server.

**Failure handling:** One unavailable platform must not break the entire profile.

**Security:** Keep private API credentials and privileged database access on the server.

**Compliance:** Use only supported, authorized access methods for external platform data.

---

# 3. Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Framework | Next.js App Router | Routing and server rendering |
| Frontend | React | Reusable components |
| Styling | Tailwind CSS | Responsive UI |
| Icons | Lucide React | Interface icons |
| Backend | Next.js Route Handlers | Public API endpoints |
| Database | Supabase PostgreSQL | Creator records |
| Social API | YouTube Data API v3 | Channel and video data |
| Social API | Instagram Graph API | Eligible Instagram data |
| Social API | Facebook Graph API | Eligible Facebook data |
| Cache | Next.js data cache | Reuse external responses |
| Optional cache | Redis | Shared caching and coordination |
| Hosting | Vercel | Application deployment |
| Validation | Server-side validation | Validate identifiers and responses |

Supabase PostgreSQL is recommended for persistent creator records. Redis is optional and should be introduced only when shared caching or higher traffic justifies it.

---

# 4. Database Design

## 4.1 Creators Table

The `creators` table stores the minimum information required to identify and render a creator profile.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Unique record identifier |
| `slug` | TEXT, UNIQUE | Public profile slug |
| `name` | TEXT | Creator display name |
| `custom_bio` | TEXT, nullable | Manually maintained biography |
| `yt_channel_id` | TEXT, nullable | YouTube channel ID |
| `ig_handle` | TEXT, nullable | Instagram username |
| `fb_page_id` | TEXT, nullable | Facebook Page ID |
| `created_at` | TIMESTAMPTZ | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | Last modification timestamp |

## 4.2 SQL Schema

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE IF NOT EXISTS creators (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    slug TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    custom_bio TEXT,

    yt_channel_id TEXT,
    ig_handle TEXT,
    fb_page_id TEXT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_creators_name
    ON creators (name);

CREATE INDEX IF NOT EXISTS idx_creators_youtube
    ON creators (yt_channel_id);

CREATE INDEX IF NOT EXISTS idx_creators_instagram
    ON creators (ig_handle);
```

## 4.3 Insert a Test Creator

```sql
INSERT INTO creators (
    slug,
    name,
    custom_bio,
    yt_channel_id,
    ig_handle,
    fb_page_id
)
VALUES (
    'example-creator',
    'Example Creator',
    'Discover this creator on Vicrix.',
    'REPLACE_WITH_REAL_YOUTUBE_CHANNEL_ID',
    NULL,
    NULL
);
```

Replace the sample YouTube channel ID with a valid one before testing.

## 4.4 Database Rules

- Use stable platform identifiers whenever available.
- Allow creators to connect only one social platform initially.
- Do not store full videos, reels, or profile image files.
- Keep manually maintained profile details separate from API responses.
- Add fields only when a real feature requires them.
- Use database constraints and appropriate access controls.
- Avoid returning internal database columns through public APIs.

---

# 5. Environment Configuration

## 5.1 Create the Next.js Project

```bash
npx create-next-app@latest vicrix --js --tailwind --app
cd vicrix
npm install lucide-react @supabase/supabase-js server-only
```

Select JavaScript, Tailwind CSS, and the App Router during setup if prompted.

## 5.2 Configure `.env.local`

```ini
# YouTube Data API v3
YOUTUBE_API_KEY=your_youtube_api_key

# Supabase server-side credentials
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your_server_only_service_role_key

# Production website URL
NEXT_PUBLIC_SITE_URL=https://your-vicrix-domain.com
```

Create a `.env.example` file with placeholder values for repository documentation.

### Security Requirements

- Never commit `.env.local`.
- Never expose private API keys to browser-side JavaScript.
- Never use `NEXT_PUBLIC_` for private credentials.
- Keep the Supabase service-role key in server-only modules.
- Configure production secrets in the hosting dashboard.
- Rotate credentials if they are exposed accidentally.

If you use a Supabase publishable key instead, apply appropriate Row Level Security policies and grant only the permissions needed by your application.

---

# 6. Social API Integrations

## 6.1 Phase One — YouTube Data API v3

YouTube is the recommended first integration because its official API provides channel and video information for supported public resources.

Potential fields include:

- Channel name
- Channel description
- Channel thumbnail
- Subscriber count, when available
- Total channel views
- Video count
- Public video IDs
- Video titles and thumbnails
- Publication timestamps

### Setup

1. Create a Google Cloud project.
2. Enable YouTube Data API v3.
3. Create an API key.
4. Restrict the key to the necessary API and usage.
5. Add the key to `.env.local`.

### Channel Endpoint

```text
https://www.googleapis.com/youtube/v3/channels
```

Use the `snippet` and `statistics` parts to retrieve supported channel information.

### Video Search Endpoint

```text
https://www.googleapis.com/youtube/v3/search
```

Use the channel ID, `type=video`, a suitable ordering, and a limited result count to retrieve candidate videos.

**Important:** Recently published videos are not necessarily viral videos or YouTube Shorts. Popularity ranking and Shorts classification require additional logic.

## 6.2 Phase Two — Instagram

Potential fields depend on the Instagram API product, account type, authorization, and granted permissions.

Possible features include:

- Eligible account information
- Available follower statistics
- Supported media metadata
- Public post and reel links
- Supported embed references

A username alone does not grant unrestricted access to every public profile or its follower count.

Instagram embeds may use URLs in this form:

```text
https://www.instagram.com/p/SHORTCODE/embed/
```

Use valid content references and supported embedding mechanisms. Do not assume Instagram oEmbed provides follower counts or unrestricted profile information.

## 6.3 Phase Three — Facebook

Potential fields depend on the Page, application permissions, access token, and available API features.

Possible features include:

- Page name and image
- Supported Page statistics
- Public content references
- Eligible video embeds
- Available verification information

Example video plugin URL:

```text
https://www.facebook.com/plugins/video.php?href=ENCODED_VIDEO_URL&show_text=false
```

Construct the URL using `URLSearchParams` and a validated public video URL.

Facebook Page likes, followers, and verification status should not be assumed available for every Page.

## 6.4 Optional Third-Party APIs

Third-party providers may be evaluated for additional integrations. Before adoption, verify their reliability, terms, cost, quotas, permitted data usage, and privacy implications.

Keep third-party providers behind the same internal service interfaces as official integrations.

---

# 7. Unified Creator API

## 7.1 Endpoint

```text
GET /api/creator/[slug]
```

Example:

```text
GET /api/creator/example-creator
```

## 7.2 Data Flow

1. Validate the slug.
2. Retrieve the creator record.
3. Return `404` if the creator does not exist.
4. Fetch connected platform data.
5. Handle each platform independently.
6. Normalize the available responses.
7. Return consistent JSON.
8. Apply the appropriate caching and rate-limiting policies.

## 7.3 Response Contract

```json
{
  "creator": {
    "id": "creator-uuid",
    "slug": "example-creator",
    "name": "Example Creator",
    "bio": "Discover this creator on Vicrix."
  },
  "platforms": {
    "youtube": {
      "connected": true,
      "status": "available",
      "data": {
        "channelName": "Example Creator",
        "subscriberCount": "125000",
        "totalViews": "8500000"
      }
    },
    "instagram": {
      "connected": false,
      "status": "not_connected",
      "data": null
    },
    "facebook": {
      "connected": false,
      "status": "not_connected",
      "data": null
    }
  },
  "generatedAt": "2026-10-14T00:00:00.000Z"
}
```

The sample statistics are illustrative, not live data.

Keep numeric statistics in a machine-readable format. Format them for display in the UI.

## 7.4 Platform Statuses

| Status | Meaning |
|---|---|
| `available` | Data retrieved successfully |
| `not_connected` | No platform identifier is configured |
| `unavailable` | Request failed or the source could not be reached |
| `not_implemented` | Integration is not implemented |
| `restricted` | Access is not permitted |
| `not_found` | No matching source resource was found |

These statuses allow the UI to display accurate information without confusing missing data with a value of zero.

---

# 8. Dynamic Creator Profile Pages

## 8.1 Route

```text
/creator/[slug]
```

Example:

```text
/creator/example-creator
```

## 8.2 Recommended Page Sections

**Creator Header**
- Profile image
- Creator name
- Biography
- Connected platforms
- Official profile links

**Statistics**
- YouTube subscribers
- Total channel views
- Video count
- Available statistics from other connected platforms

**Content Highlights**
- Recent videos
- Supported Instagram embeds
- Supported Facebook embeds

**Discovery**
- Related creators
- Featured creators
- Directory navigation

**SEO**
- Unique title and description
- Canonical URL
- Open Graph metadata
- Appropriate JSON-LD

## 8.3 Rendering Strategy

Load creator data directly from the database and integration services in the Server Component. Use the public API endpoint for client-side refreshes or external consumers.

This avoids unnecessary server-to-server HTTP requests to the application's own API.

---

# 9. Caching and Performance

## 9.1 Starting Cache Policy

| Data | Suggested TTL |
|---|---|
| Creator identity | 1–24 hours |
| YouTube statistics | 6 hours |
| Instagram statistics | 6–12 hours, where available |
| Facebook statistics | 6–12 hours, where available |
| Video candidates | 6–12 hours |
| Featured creator selection | 1–24 hours |

These are initial design choices, not platform requirements. Adjust them according to quotas, source requirements, freshness, and deployment behavior.

## 9.2 Cache Layers

1. **Database:** Canonical creator records.
2. **Data cache:** Cached external API responses.
3. **Page cache:** Cached public pages where suitable.
4. **Optional Redis:** Shared cache across application instances.

## 9.3 Performance Guidelines

- Fetch independent services concurrently.
- Use timeouts for external requests.
- Cache external responses.
- Limit the number of videos retrieved.
- Lazy-load content embeds.
- Optimize images.
- Use pagination for larger directories.
- Avoid loading the entire directory on every request.
- Track cache hit rates and external API latency.

When an API fails, reuse an eligible cached response if available. Otherwise return an unavailable state rather than fabricated statistics.

---

# 10. SEO and Structured Data

## 10.1 Metadata

Each creator page should provide:

- A unique title
- A concise description
- A canonical URL
- Open Graph metadata
- Appropriate robots directives
- Descriptive image alt text

Example title:

```text
Example Creator — Creator Profile | Vicrix
```

Example description:

```text
Explore Example Creator's profile, connected social platforms,
available statistics, and public videos on Vicrix.
```

Use Next.js `generateMetadata()` to generate page metadata from the creator record.

## 10.2 Canonical URLs

Example:

```text
https://vicrix.com/creator/example-creator
```

Replace the example domain with the actual production domain. Canonical URLs should be consistent across the directory and individual profile pages.

## 10.3 Person Schema

Use Schema.org's `Person` type for individual creators where appropriate. Use accurate public identity information and supported official profile links.

Structured data should describe visible, verified information. It does not guarantee a Google Knowledge Panel or enhanced search result.

## 10.4 Additional SEO Requirements

- Generate a sitemap.
- Add `robots.txt`.
- Provide internal links between directory and profile pages.
- Render essential profile details in server-generated HTML.
- Avoid indexing empty or incomplete pages where appropriate.
- Monitor indexing through Google Search Console.

---

# 11. Security and Reliability

## 11.1 API Security

- Keep private API credentials server-side.
- Validate incoming slugs and query parameters.
- Add rate limiting to public endpoints.
- Apply timeouts to external requests.
- Avoid exposing raw provider errors.
- Restrict database access.
- Validate external URLs used in embeds.
- Avoid arbitrary server-side URL fetching.
- Do not expose private account data or access tokens.

## 11.2 External API Failures

A platform integration must fail independently. For example, an unavailable Instagram API must not prevent a creator's YouTube profile from loading.

Display meaningful unavailable states and avoid silently replacing missing data with zero.

## 11.3 Privacy and Compliance

Use only authorized API access and supported embedding methods. Respect platform terms, access restrictions, applicable privacy requirements, and content ownership.

## 11.4 Monitoring

Track:

- API errors and quota usage
- Response latency
- Cache hit rates
- Database failures
- Invalid requests
- Rate-limited requests
- Unexpected response formats

Avoid logging secrets or unnecessary personal information.

---

# 12. Complete Project Structure

```text
vicrix/
├── README.md
├── VICRIX_BLUEPRINT.md
├── package.json
├── next.config.mjs
├── .env.example
├── .env.local
├── .gitignore
│
├── lib/
│   ├── db.js
│   ├── validation.js
│   ├── cache.js
│   └── apis/
│       ├── youtube.js
│       ├── instagram.js
│       └── facebook.js
│
├── components/
│   ├── layout/
│   │   ├── Navbar.jsx
│   │   └── Footer.jsx
│   ├── creators/
│   │   ├── CreatorCard.jsx
│   │   ├── CreatorHeader.jsx
│   │   ├── CreatorStats.jsx
│   │   └── CreatorMedia.jsx
│   └── ui/
│       ├── SearchBar.jsx
│       └── EmptyState.jsx
│
├── app/
│   ├── layout.js
│   ├── page.js
│   ├── loading.js
│   ├── not-found.js
│   ├── error.js
│   ├── globals.css
│   ├── robots.js
│   ├── sitemap.js
│   │
│   ├── creators/
│   │   └── page.js
│   │
│   ├── creator/
│   │   └── [slug]/
│   │       ├── page.js
│   │       ├── schema.js
│   │       └── loading.js
│   │
│   └── api/
│       ├── creators/
│       │   └── route.js
│       └── creator/
│           └── [slug]/
│               └── route.js
│
└── public/
    ├── favicon.ico
    └── images/
```

This is the target structure. Create each module as its functionality is implemented.

---

# 13. Complete Implementation Files

The following files form the initial implementation. YouTube is functional when valid credentials and a valid channel ID are supplied; Instagram and Facebook are placeholders until their respective integrations are implemented.

## 13.1 Database Client — `lib/db.js`

```javascript
import "server-only";

import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.SUPABASE_URL;
const supabaseKey = process.env.SUPABASE_SERVICE_ROLE_KEY;

let client;

function getSupabase() {
  if (!supabaseUrl || !supabaseKey) {
    throw new Error("Supabase environment variables are missing.");
  }

  if (!client) {
    client = createClient(supabaseUrl, supabaseKey, {
      auth: {
        persistSession: false,
        autoRefreshToken: false,
      },
    });
  }

  return client;
}

export async function getCreatorBySlug(slug) {
  const supabase = getSupabase();

  const { data, error } = await supabase
    .from("creators")
    .select(
      "id, slug, name, custom_bio, yt_channel_id, ig_handle, fb_page_id"
    )
    .eq("slug", slug)
    .maybeSingle();

  if (error) {
    console.error("Creator lookup failed:", error.message);
    throw new Error("Unable to retrieve creator.");
  }

  return data;
}
```

## 13.2 YouTube Service — `lib/apis/youtube.js`

```javascript
import "server-only";

const API_BASE = "https://www.googleapis.com/youtube/v3";

export async function fetchYouTubeData(channelId) {
  const emptyResult = {
    status: channelId ? "unavailable" : "not_connected",
    channelName: null,
    avatar: null,
    subscriberCount: null,
    totalViews: null,
    videoCount: null,
    videos: [],
  };

  if (!channelId) {
    return emptyResult;
  }

  const apiKey = process.env.YOUTUBE_API_KEY;

  if (!apiKey) {
    console.error("YOUTUBE_API_KEY is missing.");
    return emptyResult;
  }

  try {
    const channelUrl = new URL(`${API_BASE}/channels`);

    channelUrl.searchParams.set("part", "snippet,statistics");
    channelUrl.searchParams.set("id", channelId);
    channelUrl.searchParams.set("key", apiKey);

    const videosUrl = new URL(`${API_BASE}/search`);

    videosUrl.searchParams.set("part", "snippet");
    videosUrl.searchParams.set("channelId", channelId);
    videosUrl.searchParams.set("order", "date");
    videosUrl.searchParams.set("type", "video");
    videosUrl.searchParams.set("maxResults", "3");
    videosUrl.searchParams.set("key", apiKey);

    const [channelResult, videosResult] = await Promise.allSettled([
      fetch(channelUrl, {
        next: { revalidate: 21600 },
        signal: AbortSignal.timeout(8000),
      }),
      fetch(videosUrl, {
        next: { revalidate: 21600 },
        signal: AbortSignal.timeout(8000),
      }),
    ]);

    let channel = null;
    let channelFailed = false;
    let videos = [];

    if (channelResult.status === "fulfilled") {
      const response = channelResult.value;

      if (response.ok) {
        const data = await response.json();
        channel = data.items?.[0] ?? null;
      } else {
        channelFailed = true;
        console.error("YouTube channel error:", response.status);
      }
    } else {
      channelFailed = true;
    }

    if (videosResult.status === "fulfilled") {
      const response = videosResult.value;

      if (response.ok) {
        const data = await response.json();

        videos = (data.items ?? [])
          .filter((item) => item.id?.videoId)
          .map((item) => ({
            id: item.id.videoId,
            title: item.snippet?.title ?? "Untitled video",
            thumbnail:
              item.snippet?.thumbnails?.high?.url ??
              item.snippet?.thumbnails?.medium?.url ??
              null,
            publishedAt: item.snippet?.publishedAt ?? null,
            embedUrl:
              `https://www.youtube-nocookie.com/embed/${item.id.videoId}`,
            watchUrl:
              `https://www.youtube.com/watch?v=${item.id.videoId}`,
          }));
      } else {
        console.error("YouTube video error:", response.status);
      }
    }

    if (!channel) {
      return {
        ...emptyResult,
        status: channelFailed ? "unavailable" : "not_found",
        videos,
      };
    }

    const stats = channel.statistics ?? {};
    const thumbnails = channel.snippet?.thumbnails ?? {};

    return {
      status: "available",
      channelName: channel.snippet?.title ?? null,
      avatar:
        thumbnails.high?.url ??
        thumbnails.medium?.url ??
        thumbnails.default?.url ??
        null,
      subscriberCount:
        stats.hiddenSubscriberCount === true
          ? null
          : stats.subscriberCount ?? null,
      totalViews: stats.viewCount ?? null,
      videoCount: stats.videoCount ?? null,
      videos,
    };
  } catch (error) {
    console.error("YouTube service failed:", error.message);
    return emptyResult;
  }
}
```

## 13.3 Unified API Route — `app/api/creator/[slug]/route.js`

```javascript
import { NextResponse } from "next/server";

import { getCreatorBySlug } from "@/lib/db";
import { fetchYouTubeData } from "@/lib/apis/youtube";

export const runtime = "nodejs";
export const dynamic = "force-dynamic";

export async function GET(_request, { params }) {
  try {
    const { slug } = await params;

    if (
      typeof slug !== "string" ||
      !/^[a-z0-9]+(?:-[a-z0-9]+)*$/.test(slug)
    ) {
      return NextResponse.json(
        { error: "Invalid creator slug." },
        { status: 400 }
      );
    }

    const creator = await getCreatorBySlug(slug);

    if (!creator) {
      return NextResponse.json(
        { error: "Creator not found." },
        { status: 404 }
      );
    }

    const youtube = await fetchYouTubeData(
      creator.yt_channel_id
    );

    return NextResponse.json(
      {
        creator: {
          id: creator.id,
          slug: creator.slug,
          name: creator.name,
          bio: creator.custom_bio ?? "",
        },

        platforms: {
          youtube: {
            connected: Boolean(creator.yt_channel_id),
            status: youtube.status,
            data: youtube,
          },

          instagram: {
            connected: Boolean(creator.ig_handle),
            status: creator.ig_handle
              ? "not_implemented"
              : "not_connected",
            data: null,
          },

          facebook: {
            connected: Boolean(creator.fb_page_id),
            status: creator.fb_page_id
              ? "not_implemented"
              : "not_connected",
            data: null,
          },
        },

        generatedAt: new Date().toISOString(),
      },
      {
        headers: {
          "Cache-Control": "no-store",
        },
      }
    );
  } catch (error) {
    console.error("Creator API error:", error.message);

    return NextResponse.json(
      { error: "Unable to load creator information." },
      { status: 500 }
    );
  }
}
```

The endpoint disables shared HTTP caching while the external YouTube requests use their own data cache. Add separate endpoint caching only when database invalidation and freshness requirements are defined.

## 13.4 Person Schema — `app/creator/[slug]/schema.js`

```javascript
export function generatePersonSchema(creator, profileUrl) {
  const sameAs = [
    creator.yt_channel_id
      ? `https://www.youtube.com/channel/${encodeURIComponent(
          creator.yt_channel_id
        )}`
      : null,

    creator.ig_handle
      ? `https://www.instagram.com/${encodeURIComponent(
          creator.ig_handle
        )}/`
      : null,
  ].filter(Boolean);

  return {
    "@context": "https://schema.org",
    "@type": "Person",
    name: creator.name,
    description: creator.custom_bio || undefined,
    url: profileUrl,
    sameAs: sameAs.length ? sameAs : undefined,
  };
}
```

Use only verified identity and profile information. Avoid including fabricated statistics or unsupported verification claims.

## 13.5 Dynamic Creator Page — `app/creator/[slug]/page.js`

```jsx
import Image from "next/image";
import Link from "next/link";
import { notFound } from "next/navigation";

import { getCreatorBySlug } from "@/lib/db";
import { fetchYouTubeData } from "@/lib/apis/youtube";
import { generatePersonSchema } from "./schema";

const SITE_URL =
  process.env.NEXT_PUBLIC_SITE_URL ||
  "https://your-vicrix-domain.com";

function formatNumber(value) {
  if (value === null || value === undefined || value === "") {
    return "Unavailable";
  }

  const number = Number(value);

  return Number.isFinite(number)
    ? new Intl.NumberFormat("en-US", {
        notation: "compact",
        maximumFractionDigits: 1,
      }).format(number)
    : "Unavailable";
}

export async function generateMetadata({ params }) {
  const { slug } = await params;
  const creator = await getCreatorBySlug(slug);

  if (!creator) {
    return {
      title: "Creator Not Found | Vicrix",
      robots: { index: false, follow: false },
    };
  }

  const title = `${creator.name} — Creator Profile | Vicrix`;

  const description =
    creator.custom_bio ||
    `Explore ${creator.name}'s social profiles and available public content on Vicrix.`;

  const profileUrl =
    `${SITE_URL}/creator/${creator.slug}`;

  return {
    title,
    description,
    alternates: {
      canonical: profileUrl,
    },
    openGraph: {
      title,
      description,
      url: profileUrl,
      siteName: "Vicrix",
      type: "profile",
    },
  };
}

export default async function CreatorPage({ params }) {
  const { slug } = await params;

  const creator = await getCreatorBySlug(slug);

  if (!creator) {
    notFound();
  }

  const youtube = await fetchYouTubeData(
    creator.yt_channel_id
  );

  const profileUrl =
    `${SITE_URL}/creator/${creator.slug}`;

  const schema = generatePersonSchema(
    creator,
    profileUrl
  );

  return (
    <main className="min-h-screen bg-slate-950 px-4 py-10 text-white sm:px-6 md:py-16">
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(schema).replace(
            /</g,
            "\\u003c"
          ),
        }}
      />

      <div className="mx-auto max-w-5xl space-y-10">
        <nav
          aria-label="Breadcrumb"
          className="text-sm text-slate-400"
        >
          <Link href="/" className="hover:text-white">
            Home
          </Link>

          <span className="mx-2">/</span>

          <Link
            href="/creators"
            className="hover:text-white"
          >
            Creators
          </Link>

          <span className="mx-2">/</span>

          <span className="text-slate-200">
            {creator.name}
          </span>
        </nav>

        <section className="rounded-3xl border border-slate-800 bg-slate-900/70 p-6 sm:p-9">
          <div className="flex flex-col items-center gap-6 sm:flex-row sm:items-start">
            {youtube.avatar ? (
              <Image
                src={youtube.avatar}
                alt={`${creator.name} profile picture`}
                width={128}
                height={128}
                unoptimized
                className="h-28 w-28 rounded-full border border-slate-700 object-cover sm:h-32 sm:w-32"
              />
            ) : (
              <div
                aria-hidden="true"
                className="flex h-28 w-28 items-center justify-center rounded-full border border-slate-700 bg-slate-800 text-3xl font-bold text-slate-300 sm:h-32 sm:w-32"
              >
                {creator.name.charAt(0).toUpperCase()}
              </div>
            )}

            <div className="min-w-0 flex-1 space-y-4 text-center sm:text-left">
              <div>
                <p className="text-xs font-semibold uppercase tracking-[0.2em] text-indigo-300">
                  Vicrix Creator Profile
                </p>

                <h1 className="mt-2 text-3xl font-bold tracking-tight sm:text-4xl">
                  {creator.name}
                </h1>
              </div>

              {creator.custom_bio && (
                <p className="mx-auto max-w-2xl leading-7 text-slate-300 sm:mx-0">
                  {creator.custom_bio}
                </p>
              )}

              {creator.yt_channel_id && (
                <a
                  href={`https://www.youtube.com/channel/${encodeURIComponent(
                    creator.yt_channel_id
                  )}`}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="inline-flex items-center rounded-full border border-slate-700 px-4 py-2 text-sm transition hover:border-indigo-400 hover:bg-slate-800"
                >
                  Visit YouTube Channel ↗
                </a>
              )}
            </div>
          </div>
        </section>

        <section className="space-y-4">
          <h2 className="text-xl font-semibold">
            Available Statistics
          </h2>

          <div className="grid grid-cols-1 gap-4 sm:grid-cols-3">
            {[
              {
                label: "Subscribers",
                value: formatNumber(
                  youtube.subscriberCount
                ),
              },
              {
                label: "Total Channel Views",
                value: formatNumber(youtube.totalViews),
              },
              {
                label: "Published Videos",
                value: formatNumber(youtube.videoCount),
              },
            ].map((stat) => (
              <article
                key={stat.label}
                className="rounded-2xl border border-slate-800 bg-slate-900 p-5"
              >
                <p className="text-sm text-slate-400">
                  {stat.label}
                </p>

                <p className="mt-3 text-2xl font-bold">
                  {stat.value}
                </p>
              </article>
            ))}
          </div>

          <p className="text-xs text-slate-500">
            Statistics depend on source availability and
            caching delays.
          </p>
        </section>

        <section className="space-y-5">
          <div>
            <h2 className="text-xl font-semibold">
              Latest YouTube Videos
            </h2>

            <p className="mt-1 text-sm text-slate-400">
              Recent public videos returned by the YouTube API.
            </p>
          </div>

          {youtube.videos.length > 0 ? (
            <div className="grid grid-cols-1 gap-5 md:grid-cols-3">
              {youtube.videos.map((video) => (
                <article
                  key={video.id}
                  className="overflow-hidden rounded-2xl border border-slate-800 bg-slate-900"
                >
                  <div className="aspect-video bg-slate-800">
                    <iframe
                      src={video.embedUrl}
                      title={video.title}
                      className="h-full w-full"
                      loading="lazy"
                      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
                      referrerPolicy="strict-origin-when-cross-origin"
                      allowFullScreen
                    />
                  </div>

                  <div className="space-y-3 p-4">
                    <h3 className="line-clamp-2 text-sm font-medium leading-6">
                      {video.title}
                    </h3>

                    <a
                      href={video.watchUrl}
                      target="_blank"
                      rel="noopener noreferrer"
                      className="text-sm text-indigo-300 hover:text-indigo-200"
                    >
                      Watch on YouTube ↗
                    </a>
                  </div>
                </article>
              ))}
            </div>
          ) : (
            <div className="rounded-2xl border border-slate-800 bg-slate-900 p-6 text-sm text-slate-400">
              {youtube.status === "not_connected"
                ? "This creator has not connected a YouTube channel."
                : "YouTube videos are currently unavailable."}
            </div>
          )}
        </section>

        <footer className="border-t border-slate-800 pt-6 text-sm text-slate-500">
          Creator profile powered by Vicrix.
        </footer>
      </div>
    </main>
  );
}
```

**Compatibility:** This page uses asynchronous `params`, as in Next.js 15 and later. Adjust the implementation if using an older version. The external image uses `unoptimized`; configure approved remote image hosts if you enable Next.js image optimization.

## 13.6 Package Scripts

Ensure your `package.json` contains the necessary scripts. Keep the dependencies generated by your installed Next.js version.

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint ."
  }
}
```

The lint script requires ESLint to be installed and configured.

## 13.7 Git Ignore

Ensure `.gitignore` contains:

```gitignore
node_modules/
.next/
out/
.env
.env.local
.env.*.local
.vercel/
```

Do not ignore `.env.example`; it should contain placeholders, never secrets.

---

# 14. Development and Testing

## 14.1 Start the Application

```bash
npm install
npm run dev
```

Open:

```text
http://localhost:3000
```

## 14.2 Test the Creator Profile

```text
http://localhost:3000/creator/example-creator
```

## 14.3 Test the API

```text
http://localhost:3000/api/creator/example-creator
```

## 14.4 Test Checklist

- [ ] Supabase connection works.
- [ ] A valid creator slug retrieves the correct record.
- [ ] Unknown creators return a `404`.
- [ ] Invalid slugs return a `400`.
- [ ] YouTube data loads for a valid channel ID.
- [ ] Hidden subscriber counts are not shown as zero.
- [ ] Video retrieval failure does not erase channel statistics.
- [ ] External API failures do not crash the page.
- [ ] API credentials remain server-side.
- [ ] Metadata and JSON-LD contain correct information.
- [ ] Public profiles remain usable when a platform is unavailable.
- [ ] Database errors are logged safely.
- [ ] Caching behavior is tested in the deployed environment.

---

# 15. Deployment Guide

## 15.1 Recommended Hosting

Vercel is a suitable hosting option for the Next.js application, with Supabase providing persistent database storage.

## 15.2 Deployment Steps

1. Push the project to a Git repository.
2. Import the repository into Vercel.
3. Configure production environment variables.
4. Create the Supabase database schema.
5. Verify YouTube API credentials and quota limits.
6. Run a production build.
7. Deploy the application.
8. Test creator profiles and API endpoints.
9. Verify canonical URLs, metadata, and sitemap output.
10. Monitor logs, errors, and API usage.

## 15.3 Production Considerations

- Do not assume local file storage is persistent on serverless hosting.
- Verify caching and revalidation behavior for your deployed Next.js version.
- Use shared external caching when multiple instances require consistent cache reuse.
- Configure rate limiting for public endpoints.
- Restrict access to privileged database operations.
- Test cold starts and external API failures.
- Monitor API quotas to prevent unexpected service interruptions.

---

# 16. Implementation Roadmap

## Phase 1 — Project Foundation

- [ ] Initialize Next.js and Tailwind CSS.
- [ ] Configure Supabase.
- [ ] Create the `creators` table.
- [ ] Add test creator records.
- [ ] Build the global layout and homepage.

**Outcome:** A working directory backed by the database.

## Phase 2 — Creator Profiles

- [ ] Implement `/creator/[slug]`.
- [ ] Load records server-side.
- [ ] Add profile details and official links.
- [ ] Add loading and not-found states.
- [ ] Generate dynamic metadata.

**Outcome:** Crawlable, shareable creator pages.

## Phase 3 — YouTube Integration

- [ ] Configure YouTube Data API v3.
- [ ] Fetch channel statistics.
- [ ] Retrieve recent videos.
- [ ] Normalize responses.
- [ ] Add caching.
- [ ] Display video embeds.

**Outcome:** Creator profiles with working YouTube data.

## Phase 4 — Unified API

- [ ] Implement `/api/creator/[slug]`.
- [ ] Standardize response structures.
- [ ] Handle platform failures independently.
- [ ] Add input validation.
- [ ] Add rate limiting and monitoring.

**Outcome:** A stable backend contract.

## Phase 5 — Instagram and Facebook

- [ ] Review API eligibility and permissions.
- [ ] Configure required authorization.
- [ ] Implement supported account data retrieval.
- [ ] Add permitted content embeds.
- [ ] Handle restricted and unavailable data.

**Outcome:** Additional integrations where supported.

## Phase 6 — Discovery Features

- [ ] Add creator search.
- [ ] Add pagination.
- [ ] Add platform filters.
- [ ] Introduce featured profiles.
- [ ] Improve mobile responsiveness.

**Outcome:** A more useful creator directory.

## Phase 7 — Production Readiness

- [ ] Generate a sitemap and robots file.
- [ ] Audit metadata and structured data.
- [ ] Test database security.
- [ ] Configure monitoring and rate limiting.
- [ ] Validate cache behavior.
- [ ] Run accessibility and performance checks.

**Outcome:** A maintainable production platform.

---

# 17. Future Enhancements

Once the core system is stable, consider adding:

- Creator categories and tags
- Verified creator profiles
- Editorial collections
- Creator submission workflows
- Admin dashboard
- Scheduled data refresh jobs
- Profile update history
- More supported social platforms
- Duplicate creator detection
- API documentation
- Search analytics
- Content popularity ranking
- Profile change notifications
- Shared Redis caching
- Sitemap generation for large directories

Introduce these features when they provide measurable value without unnecessary complexity or data collection.

---

# 18. Production Checklist

Before declaring Vicrix production-ready, confirm the following.

### Architecture

- [ ] Creator records are the source of truth for identity.
- [ ] Platform integrations are isolated into separate services.
- [ ] External responses follow a consistent schema.
- [ ] Unavailable integrations do not break profile pages.

### Database

- [ ] Schema and indexes are configured.
- [ ] Database access is restricted.
- [ ] Private credentials are server-only.
- [ ] Creator slugs are unique and validated.

### API

- [ ] YouTube integration works with valid credentials.
- [ ] Errors and timeouts are handled.
- [ ] API quota usage is monitored.
- [ ] Public endpoints are rate-limited.

### Performance

- [ ] External data is cached appropriately.
- [ ] Media embeds load lazily.
- [ ] Directory queries are paginated.
- [ ] Cache invalidation is tested.

### SEO

- [ ] Titles and descriptions are unique.
- [ ] Canonical URLs are correct.
- [ ] JSON-LD represents verified information.
- [ ] Sitemap and robots files are configured.
- [ ] Important profile content is server-rendered.

### Deployment

- [ ] Production secrets are configured.
- [ ] The production build succeeds.
- [ ] Database connectivity is verified.
- [ ] API endpoints return correct statuses.
- [ ] Error logs are monitored.

---

## Final Architecture Principles

Vicrix should follow eight core rules:

1. Store creator identity and configuration rather than complete social media feeds.
2. Retrieve statistics through supported, authorized integrations.
3. Normalize all platform data into a stable internal model.
4. Cache reusable responses while respecting source restrictions.
5. Render essential profile information on the server.
6. Protect private credentials and database operations.
7. Treat missing data as unavailable rather than inventing values.
8. Introduce new platforms and features incrementally.

**Vicrix — Lightweight storage. Reliable integrations. Discoverable creator profiles.**

