---
layout: post
title: "System Design - Chat System"
date: 2021-09-08 22:00:00 +0800
tags: Server DesignPattern
---

Design Chat System

# Requirement clarification

## functional requirement

- one-to-one chatting message(sent/delivered/read)
- user(online/offline/last seen time)
- picture, video sharing in the message

## non-functional requirement

## extend requirement

- group chat
- push notification for offline user
- end-to-end encryption(Whatsapp)

# Scale of the system - data size

- 500 million DAU(daily active users)
- each user sends 40 messages per day
- `message count per day = 500M * 40 = 20 billion`
- each message take 100bytes

## Traffic estimates

## Storage estimates

- `per day = 20 billion * 100 bytes = 2 TB/day`
- `storage = 2 TB * 265 days * 5 years = 3.6 PB`

## Bandwidth estimates

- `bandwidth = 2 TB / (24 hours * 3600 sec) = 25 MB/s`

## Memory cache estimates

# API interface

- `sendMessage(User from, User to, String message, Boolean userActivity)`
- `isUserOnline(String userID)`

# Database data-model

- User

  - String userID
  - TimeStamp time

- Message
  - String messageID
  - String content

# High-level design

![Chat System](/assets/images/2021-09-08-SystemDesign_ChatSystem_1.png)

![Chat System](/assets/images/2021-09-08-SystemDesign_ChatSystem_2.png)

# Detail design

- use websocket to timely nofify

  - heartbeat to keep alive

- authentication

- group table

- MQ for keep message order(userID hash)

# Bottlenecks(follow up)

- loadbalance
- consistent hashing to solve single point failure; vertual host to solve hash slide
- idempotence, every message has one ID
- groupID hash or userID hash for database sharding
- use UDP to send when big event; client pull message
- multi-layer message block, to avoid deserialize/serialize all package
