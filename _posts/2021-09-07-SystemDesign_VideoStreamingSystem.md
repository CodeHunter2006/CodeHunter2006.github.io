---
layout: post
title: "System Design - Video Streaming System"
date: 2021-09-07 22:00:00 +0800
tags: Server DesignPattern
---

Design Video Streaming System

# Requirement clarification

## functional requirement

- upload video
- share video
- search video
- change resolution
- record: likes/dislikes, total views, comments

## non-functional requirement

- reliable, video will not lost
- available, watch with poor network
- real-time, limited video lag

## extend requirement

- recommend
- TopK popular
- subscribe channel
- favorate
- subtitle translate
- advertisement

# Scale of the system - data size

## Traffic estimates

- youtube have 2billion users, DAU 1billion = 1000M，every user watch 5 hours per day,
  `1000M * 5 / (24 * 3600) = 57K videos/sec`
- `upload:view = 1:200`，`upload TPS = 57K/200 = 285 videos/sec`

## Storage estimates

- videosize: 50MB/minutes; 10minutes per video
- `300(285)videos/sec * 60sec * 10min / 60min = upload 3000 hours videos/hours` =>
  `3000hours * 60min * 50MB = 9TB/min = 150G/sec`

## Bandwidth estimates

- input(upload) bandwidth: `3000hours * 60min * 10MB = 1.8TB/min = 30GB/sec`
- outgoing bandwidth: `30GB/sec * 200 = 6TB/sec`

- Amazon CDN per GB $0.02
- `150GB/sec * 0.02$ * 24hours * 3600sec = $259,200 per day`

## Memory cache estimates

- just use CDN to accelerate video streaming

# API interface

- `uploadVideo(dev_key, video_title, video_description, tags[], category_id, default_language, recording_details, video_contents)`，return 202
- `searchVideo(dev_key, search_query, user_location, maximum_to_return, page_token`，return json
- `streamVideo(dev_key, video_id, offset, codec, resolution`, return stream

# Database data-model

- MySQL(ACID)

  - billing information
  - user information
  - transaction information

- NoSQL(Cassandra)

  - heavy write and read(9:1)
  - smaller storage footprint
  - save viewing history, not delete, huge size
  - Live Viewing History(LiveVH): save detail information
  - Compressed Viewing History(CompressedVH): a large amount of older viewing records with rare updates.

- Pure video chunks

  - saved in Amazon S3

- Video

  - VideoID
  - Title
  - Description
  - Size
  - Thumbnail
  - Uploader/User
  - likesCount
  - dislikesCount
  - viewsCount

- VideoComment

  - CommentID
  - VideoID
  - UserID
  - Comment
  - TimeOfCreation

- User

  - UserID
  - Name
  - Email
  - Address
  - Sex
  - Age

# High-level design

![High-level design](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_1.jpeg)

# Detail design

- Read Heavy System
  partition(for write) and replication(for read)
- Compress 480p first, and compress other 1080p, 1440p after 1 hour

## Video Encoding

- why encoding ?
  - raw video is too big, like 3GB
  - different device support different format
  - different network condition need different resolution
  - format: avi, mov, mp4, dash, hls, m3u8
  - codecs: H.264, VP9, HEVC

![encoding](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_2.jpeg)

![DAG](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_3.jpeg)

![MQ](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_4.jpeg)

![Split](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_5.jpeg)

# Bottlenecks(follow up)

- flash upload(same hash code)
- split parellel upload
- upload to nearby datacenter or cdn
- scale up everything，loosely coupled system(MQ)
- cost-saving optimization
  - only use CDN with popular videos
  - different area not need distribute
  - self-build CDN

![CDN](/assets/images/2021-09-07-SystemDesign_VideoStreamingSystem_6.jpeg)

- error handling

  - encode retry
  - can not recognized file
  - API server down, consistent hashing
  - DB server down，failover(slave become a master)

- Metadata Sharding
  distribute out data onto multiple machines wisely

  - Sharding based on UserID
    - Pros
      - fast for one user's video
    - Cons
      - hard to search video name
      - famous person will cause bottleneck
      - uneven distribution
  - Sharding based on VideoID
    - Pros
      - fast for one video
    - Cons
      - hard to search video belong to one user
      - popular video will cause bottleneck, solve with cache
  - ElasticSearch

- upload breakpoint resume
  - offset
