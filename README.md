# KomikStation Manhwa Scraper

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-18%2B-339933?style=flat-square&logo=node.js&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/JavaScript-ESM-F7DF1E?style=flat-square&logo=javascript&logoColor=black" alt="JavaScript">
  <img src="https://img.shields.io/badge/Scraper-Web-111827?style=flat-square" alt="Web Scraper">
  <img src="https://img.shields.io/badge/Status-Active-22c55e?style=flat-square" alt="Active">
</p>---

Overview

KomikStation Manhwa Scraper is a lightweight Node.js scraper designed to interact with publicly accessible KomikStation pages and endpoints.

It provides a simple programmatic interface for:

- Searching manhwa
- Retrieving manga metadata
- Listing available chapters
- Reading chapter information
- Retrieving available download links
- Downloading chapter ZIP archives
- Tracking download progress

The project is designed to be easily integrated into:

- CLI applications
- Telegram bots
- Discord bots
- REST APIs
- Web applications
- Automation systems
- Personal content-management tools

«This project does not host or redistribute manga content.»

---

Features

<table>
<tr>
<td width="50%">Search

- WordPress REST search
- Configurable result limit
- Manga URL filtering
- Clean normalized results

</td>
<td width="50%">Manga

- Manga title
- Cover image
- Synopsis
- Chapter count
- Latest chapter
- Chapter URLs

</td>
</tr><tr>
<td>Downloads

- Chapter ZIP downloads
- Streaming support
- Download progress
- File-size protection
- ZIP validation

</td>
<td>Reliability

- Request timeout
- Automatic retry
- "403" retry
- "429" retry
- "5xx" retry
- Network-error recovery

</td>
</tr>
</table>---

Tech Stack

Technology| Purpose
Node.js| Runtime
JavaScript| Core implementation
ES Modules| Module system
Native Fetch| HTTP requests
WordPress REST API| Manga search
HTML parsing| Manga/chapter extraction
KlikCDN| Chapter file delivery

No scraping framework is required.

No external HTTP client is required.

---

Requirements

- Node.js "18+"
- Internet connection
- Access to the target website

Check your Node.js version:

node --version

Recommended:

Node.js 20+

---

Installation

Clone the repository:

git clone <repository-url>

Enter the project:

cd komikstation-scraper

Install dependencies:

npm install

Run your application:

npm start

---

Quick Start

Import the scraper:

import {
  searchManhwa,
  getMangaDetail,
  downloadChapter
} from './index.js'

Search:

const results = await searchManhwa(
  'Solo Leveling'
)

console.log(results)

Retrieve manga details:

const manga = await getMangaDetail(
  results[0].url
)

console.log(manga)

Download a chapter:

const chapter = manga.chapters[0]

const file = await downloadChapter(
  chapter.downloadUrl
)

console.log(file.filename)

---

API Reference

"searchManhwa()"

Searches for manga through the public WordPress REST endpoint.

Signature

searchManhwa(
  query: string,
  limit?: number
): Promise<MangaSearchResult[]>

Parameters

Parameter| Type| Required| Default
"query"| "string"| Yes| —
"limit"| "number"| No| "5"

Example

const results = await searchManhwa(
  'Omniscient Reader',
  10
)

Response

[
  {
    "id": 123,
    "title": "Example Manhwa",
    "url": "https://komikstation.org/manga/example/"
  }
]

---

"getMangaDetail()"

Retrieves manga information and chapter metadata.

Signature

getMangaDetail(
  mangaUrl: string
): Promise<MangaDetail>

Example

const manga = await getMangaDetail(
  'https://komikstation.org/manga/example/'
)

Response

{
  "title": "Example Manhwa",
  "cover": "https://example.com/cover.jpg",
  "synopsis": "Manga synopsis...",
  "pageUrl": "https://komikstation.org/manga/example/",
  "chapterCount": 120,
  "latestChapter": "Chapter 120",
  "chapters": [
    {
      "num": 120,
      "title": "Chapter 120",
      "date": "2026-01-01",
      "readUrl": "https://komikstation.org/manga/example-chapter-120/",
      "downloadUrl": "https://klikcdn.com/ddl?id=123456"
    }
  ]
}

---

"downloadChapter()"

Downloads a chapter archive from an available download URL.

Signature

downloadChapter(
  ddlUrl: string,
  options?: DownloadOptions
): Promise<DownloadResult>

Options

Option| Type| Default
"maxBytes"| "number"| "100 MB"
"timeoutMs"| "number"| "120000"
"onProgress"| "function"| "undefined"

Example

const file = await downloadChapter(
  chapter.downloadUrl,
  {
    onProgress(current, total) {
      if (!total) return

      const percent =
        (current / total) * 100

      console.log(
        `${percent.toFixed(1)}%`
      )
    }
  }
)

Result

{
  filename: 'chapter-120.zip',
  size: 52428800,
  buffer: Buffer
}

---

Architecture

                         ┌───────────────────┐
                         │   Your Application │
                         │ CLI / Bot / API   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                    ┌──────────────────────────┐
                    │   KomikStation Scraper   │
                    ├──────────────────────────┤
                    │                          │
                    │  searchManhwa()          │
                    │  getMangaDetail()        │
                    │  downloadChapter()       │
                    │                          │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
          ┌─────────────────┐       ┌─────────────────┐
          │ WordPress REST  │       │ Manga HTML     │
          │ Search API      │       │ Page           │
          └─────────────────┘       └────────┬────────┘
                                             │
                                             ▼
                                   ┌─────────────────┐
                                   │ Chapter Data    │
                                   │ + Download URL  │
                                   └────────┬────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │ Download Flow   │
                                   │                 │
                                   │ Nonce           │
                                   │ Init            │
                                   │ Polling         │
                                   │ File Download   │
                                   └────────┬────────┘
                                            │
                                            ▼
                                   ┌─────────────────┐
                                   │ ZIP Buffer      │
                                   └─────────────────┘

---

Internal Request Layer

All requests are handled through a shared request function.

Request
   │
   ▼
Timeout Protection
   │
   ▼
HTTP Request
   │
   ├── Success ───────────────► Return Response
   │
   ├── 403 ───────────────────► Retry
   │
   ├── 429 ───────────────────► Retry
   │
   ├── 5xx ───────────────────► Retry
   │
   └── Network Error ─────────► Retry

The retry layer prevents temporary network failures from immediately breaking the operation.

---

Download Pipeline

The chapter download process is separated into several stages.

01. Resolve Download ID

The scraper extracts the download identifier from:

/ddl?id=<ID>

02. Request Download Page

The scraper requests the download page and retrieves the current nonce.

03. Initialize File

The file-generation endpoint is called using:

{
  "post_id": 123456,
  "nonce": "..."
}

04. Handle Generation State

If the server reports that the file is still being generated:

busy

the scraper waits before checking again.

05. Retrieve Download URL

Once the file is ready, the generated download URL is returned.

06. Stream ZIP

The ZIP is downloaded using a stream.

07. Validate

The response is checked for:

- Non-empty content
- Maximum file size
- ZIP signature

---

Download State Machine

                  ┌─────────────┐
                  │    START    │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │ Get Nonce   │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │ Initialize  │
                  └──────┬──────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
        ┌───────────┐        ┌────────────┐
        │   Ready   │        │    Busy    │
        └─────┬─────┘        └──────┬─────┘
              │                     │
              │                     ▼
              │              Wait 5 Seconds
              │                     │
              │                     └──────┐
              │                            │
              └────────────────────────────┘
                           │
                           ▼
                    Download ZIP
                           │
                           ▼
                    Validate ZIP
                           │
                           ▼
                         DONE

---

Progress Tracking

The scraper exposes download progress through a callback.

await downloadChapter(url, {
  onProgress(current, total) {
    if (!total) return

    const percent =
      (current / total) * 100

    process.stdout.write(
      `\rDownloading ${percent.toFixed(1)}%`
    )
  }
})

This makes the downloader suitable for:

- CLI progress bars
- Telegram progress messages
- Discord status updates
- WebSocket events
- REST job tracking
- Dashboard monitoring

---

Error Handling

Recommended pattern:

try {
  const manga = await getMangaDetail(url)

  console.log(manga)
} catch (error) {
  console.error(
    error.message
  )
}

Possible errors include:

Kata kunci kosong.
Pencarian gagal.
Respons pencarian bukan daftar.
Bukan link manga KomikStation.
Halaman manga gagal.
Daftar chapter tidak ditemukan.
Link download tidak valid.
Nonce download tidak ditemukan.
Init download gagal.
Server masih generate file.
Unduhan gagal.
File kebesaran.
File melebihi batas.
File kosong.
Bukan file ZIP valid.

---

Safety Limits

Request Timeout

Default request timeout:

25_000

Download timeout:

120_000

---

Maximum ZIP Size

Default:

100 * 1024 * 1024

Equivalent to:

100 MB

You can change it:

await downloadChapter(url, {
  maxBytes: 200 * 1024 * 1024
})

---

ZIP Validation

The downloader verifies the ZIP signature before returning the file.

if (
  !buffer
    .subarray(0, 2)
    .equals(Buffer.from('PK'))
) {
  throw new Error(
    'Bukan file ZIP valid.'
  )
}

This helps prevent an HTML error page or unexpected response from being treated as a valid download.

---

Recommended Project Structure

For a modular production project:

komikstation-scraper/
│
├── src/
│   ├── index.js
│   ├── request.js
│   ├── search.js
│   ├── manga.js
│   ├── download.js
│   └── utils.js
│
├── examples/
│   ├── search.js
│   ├── detail.js
│   └── download.js
│
├── tests/
│   ├── search.test.js
│   ├── manga.test.js
│   └── download.test.js
│
├── package.json
├── README.md
├── LICENSE
└── .gitignore

For a small project:

komikstation-scraper/
├── index.js
├── package.json
├── README.md
└── LICENSE

---

Integration Examples

Telegram Bot

A Telegram bot can expose the scraper through commands such as:

/search <title>
/manga <title>
/chapters <manga>
/download <chapter>

Possible flow:

User
 │
 ▼
/search Solo Leveling
 │
 ▼
Search Results
 │
 ▼
Select Manga
 │
 ▼
Chapter List
 │
 ▼
Select Chapter
 │
 ▼
Download
 │
 ▼
ZIP File

---

Discord Bot

Possible command structure:

/komik search <title>
/komik detail <title>
/komik chapters <title>
/komik download <chapter>

---

REST API

The scraper can also be wrapped in Express, Fastify, Hono, or another HTTP framework.

Example:

GET /search?q=solo+leveling
GET /manga?url=<url>
GET /chapters?url=<url>
GET /download?url=<url>

Architecture:

Client
  │
  ▼
REST API
  │
  ├── Search
  ├── Manga
  ├── Chapters
  └── Download
          │
          ▼
   Scraper Engine
          │
          ▼
   External Website

---

Full Example

import {
  searchManhwa,
  getMangaDetail,
  downloadChapter
} from './index.js'

import {
  writeFile
} from 'node:fs/promises'

async function main() {
  console.log(
    'Searching KomikStation...'
  )

  const results =
    await searchManhwa(
      'Solo Leveling',
      5
    )

  if (!results.length) {
    console.log(
      'No manga found.'
    )

    return
  }

  console.table(results)

  const manga =
    await getMangaDetail(
      results[0].url
    )

  console.log(
    `\nTitle: ${manga.title}`
  )

  console.log(
    `Chapters: ${manga.chapterCount}`
  )

  const chapter =
    manga.chapters[0]

  if (!chapter.downloadUrl) {
    console.log(
      'Download is not available.'
    )

    return
  }

  console.log(
    `\nDownloading ${chapter.title}...`
  )

  const file =
    await downloadChapter(
      chapter.downloadUrl,
      {
        onProgress(
          current,
          total
        ) {
          if (!total) return

          const percent =
            (current / total) * 100

          process.stdout.write(
            `\rProgress: ${percent.toFixed(1)}%`
          )
        }
      }
    )

  await writeFile(
    file.filename,
    file.buffer
  )

  console.log(
    `\n\nSaved: ${file.filename}`
  )

  console.log(
    `Size: ${file.size} bytes`
  )
}

main().catch(error => {
  console.error(
    '\nError:',
    error.message
  )

  process.exit(1)
})

---

Performance Notes

The scraper intentionally uses lightweight requests.

Search

Uses the public WordPress REST API rather than scraping search-result HTML.

Manga Details

Uses the manga HTML page to extract:

- Metadata
- Chapter list
- Reading URLs
- Download URLs

Downloads

Uses streaming instead of loading an external file through multiple intermediate requests.

For very large files or high-concurrency applications, consider changing the downloader to stream directly to disk instead of keeping the entire ZIP in memory.

---

Scalability

For applications that need multiple simultaneous requests, consider adding:

Request Queue
      │
      ▼
Concurrency Limit
      │
      ▼
Scraper Workers
      │
      ▼
Download Queue
      │
      ▼
Storage

Recommended components for a larger deployment:

- Queue system
- Concurrency limiter
- Persistent storage
- Job status tracking
- Structured logging
- Retry policy
- Rate control
- Cache layer

Avoid sending unnecessary or excessive requests to the target service.

---

Caching Strategy

A simple cache can reduce repeated requests.

Search Query
     │
     ▼
Cache?
 ┌───┴────┐
 │        │
YES      NO
 │        │
 ▼        ▼
Return   Request
          │
          ▼
        Cache
          │
          ▼
        Return

Useful cache targets:

- Search results
- Manga metadata
- Chapter lists

Download files should generally be handled separately from metadata caching.

---

Limitations

The scraper depends on the current structure and behavior of the external services.

Changes to the website may affect:

- WordPress REST API
- Manga page structure
- Chapter HTML
- Metadata tags
- Download URLs
- Nonce generation
- Download initialization
- ZIP delivery

For example, changes to selectors such as:

id="chapterlist"

or:

class="chapternum"

may require parser updates.

---

CAPTCHA & Access Controls

This project does not attempt to solve or bypass CAPTCHA systems.

It also should not be modified to circumvent:

- CAPTCHA
- Authentication requirements
- Paywalls
- Access restrictions
- Anti-bot challenges
- Other technical access controls

The scraper is designed around publicly accessible endpoints and pages.

---

Responsible Use

This project is intended as a technical scraping and automation component.

Users are responsible for complying with:

- Website terms of service
- Applicable laws
- Copyright requirements
- Content licenses
- Rate limits
- Service usage policies

Only download or process content when you have the necessary authorization or rights to do so.

The project author does not host or provide the scraped manga content.

---

Disclaimer

KomikStation Manhwa Scraper is an independent project.

It is not affiliated with, sponsored by, or officially endorsed by KomikStation or KlikCDN.

External websites and APIs can change without notice.

The software is provided for technical and educational purposes and is used at the user's own responsibility.

---

Project Metadata

Project       : KomikStation Manhwa Scraper
Author        : SCRIPETEREN
Language      : JavaScript
Runtime       : Node.js
Module       : ES Modules
API Key       : Not Required
CAPTCHA       : Not Handled
Output        : JSON + ZIP
Architecture  : Modular
Status        : Active

---

Roadmap

- [x] Manga search
- [x] Manga metadata
- [x] Chapter extraction
- [x] Chapter download URLs
- [x] ZIP downloading
- [x] Download progress
- [x] Retry handling
- [x] Timeout protection
- [x] ZIP validation
- [ ] CLI interface
- [ ] Built-in caching
- [ ] Configurable concurrency
- [ ] Structured logging
- [ ] Unit tests
- [ ] REST API wrapper
- [ ] Telegram integration
- [ ] Discord integration

---

Contributing

Contributions are welcome.

Recommended workflow:

git clone <repository-url>

cd komikstation-scraper

git checkout -b feature/your-feature

npm install

Make your changes, test them, and submit a pull request.

Please keep changes:

- Focused
- Readable
- Modular
- Backward-compatible where possible

---

License

This project is released under the MIT License.

MIT License

Copyright (c) 2026 SCRIPETEREN

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files, to deal in the Software
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
Software, subject to the conditions of the MIT License.

---

Author

<p align="center"><strong>SCRIPETEREN</strong>

<br>Node.js Developer · System Developer · Automation

<br><br>

JavaScript · Node.js · Web Scraping · API Integration · Automation · Backend Systems

</p>---

<p align="center">
  Built with JavaScript and Node.js.
</p>