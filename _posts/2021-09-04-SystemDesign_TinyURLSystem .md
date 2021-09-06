---
layout: post
title: "System Design - Tiny URL System"
date: 2021-09-04 22:00:00 +0800
tags: Server DesignPattern
---

![Tiny URL System](/assets/images/2021-09-04-SystemDesign_TinyURLSystem_1.jpeg)
Design Tiny URL System
(Using TinyURL-System to explain System-Design process.)

# Requirement clarification(2min)

## functional requirement

1. user1: long url ->(response) short url
2. user1: share short url to others(user2)
3. user2: short ulr ->(http 301 redirect to) long url

- user can set short url TTL(Time To Live)

## non-functional requirement

- system should be highly available
- URL redirection should happen in real-time
- short links should not be predictable

## extend requirement

- Analytics; e.g., how may time a redirection happened?
- may use http 302 to collect parameters，like: who/when/by whom cliked the short URL
- be accessible through REST/RPC APIs by other services

# Scale of the system - data size(5min)

## Traffic estimates

- generate 500M/month
- read/write ratio 100:1

- `write QPS = 500M/(30(d) * 24(h) * 3600(s)) = 200 URLs/s`
- `read QPS = 200 * 100 = 20 K/s`

## Storage estimates

- save data for 5 years
- 500 byte/url
- `storage = 30 billion * 500 bytes = 15 TB`

## Bandwidth estimates

- `incoming bandwidth = 200 * 0.5KB = 100KB/s`
- `outgoing bandwidth = 100 * 100KB/s = 10MB/s`

## Memory cache estimates

- cache 20% top frequent url per day
- `total count per day = 20K * 3600 seconds * 24 hours = 1.7 billion`
- `cache size = 0.2 * 1.7 billion * 0.5KB = 170GB`
- use LRU evict policy

# API interface(1min)

```
POST api/v1/data/shorten
param {
    longURL: string
    expireTime: long(optional)
}
return shortURL

GET api/v1/shortUrl
return longURL for HTTP redirection

DELETE api/v1/data/longURL
```

# Database data-model(5min)

- URL

  - Hash:varchar(16), PK(PrimaryKey)
  - OriginalURL: varchar(512)
  - CreationDate: datetime
  - ExpirationDate: datetime
  - UserID: int

- User

  - UserID: int, PK
  - Name: varchar(20)
  - Email: varchar(32)
  - CreationDate: datetime
  - LastLogin: datetime

- we don't need much relation between object, so we choose **nosql** like MongoDB
- nosql also can scale up easily、high throughput

# High-level design(3min)

1. use "base 62" to present a shortURL
   - base 64 include "+ -"
2. if hash conflict happend, append string harsh again
3. Generate algorithm: UUID、distributed uinque id + hash

# Detail design(5min)

- how much characters we need for shorURL?
  total count is "30 billion" as mentioned before.
  base 62, so `character count = log(30 billion)/log(62) = 6`。
  we use 7 characters to avoid Brute-Force attack.

- Encoding actual URL
  1. hash
     1. choose one hash function to generate unique id, like MD5, SHA-1, HAVAL, CRC32.
     2. choose pre 6 characters.
     3. if conflict, append "userID/timestamp/logID" and rehash.
     4. use bloom filter to check existence.
  2. base62 conversion
     1. use "unique id generator" get a number < 30 billion.
     2. recursively divide 62 and get base62 character by remainder.
     3. no need to consider collision for unique id.

| string hash                  | base 62                                |
| ---------------------------- | -------------------------------------- |
| length fixed                 | lengh may shorter                      |
| not need unique id generator | need unique id generator               |
| collision、rehash            | no collision                           |
| unpredictable                | predictable if incr 1, security issure |

# Bottlenecks(follow up, 5min)

- how to solve "base 62 predictable" problem ?
  use a inner mapping table to avoid.
- how to limit client operation ?
  set a rate count in client session/cookie. ban if reached some degree.
- how to realize URL redirection ?
  config apache/nginx redirect function.
- purging or DB cleanup
  - lazy cleanup, like redis
  - clean service run once every day
  - default expire date like 2 years. or do not delete "space is cheap"
  - reuse deleted short url, put into pool(rebuild bloom filter)
- telemetry, data collection
  use redirect 302(temporal redirect) to collect more information, like:
  country of the visitor, date and time of access, web page that referred the click,
  brower or platform where the page was accessed.
- 1-1 or 1-n (longURL-shortURL)
  - 1-1
    hash(longURL) generate one shortURL. save storage, but cannot distinguish different user/time/location.
  - 1-n(recommended)
    every time a user generate a different shortURL, save more information at the same time, like:
    userID, time, location...
    so that we can do data mining or sell the information.
    "space is cheap".
- if one server down and loose 1000 id...
  just abandon the ids
- Cache
  space 20% per day, LRU
- Data Partitioning and Replication
  user different db instance according to cap of the shortURL, to scale out DB.
  Range-Based Partitioning / Hash-Based Partitioning.
