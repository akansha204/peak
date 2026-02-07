# Peak Server
Real-time Leaderboard Service using gRPC and Redis.
## Overview
**Peak Server** is a backend service that provides a **real-time leaderboard** using **gRPC bidirectional streaming** and **Redis Sorted Sets**.

Clients can:  
- Join the leaderboard stream
- Send score updates
- Receive updated leaderboard snapshots and rankings in real-time

The server is **stateless** with redis acting as the source of truth.

---
## Tech Stack
- Java
- Spring Boot
- gRPC (bidirectional streaming)
- Redis (Sorted Sets)

## gRPC API (Simplified)

### Client Events
- `JOIN` – Request current leaderboard snapshot
- `SCORE_UPDATE` – Update user score

### Server Events
- `SNAPSHOT` – Leaderboard snapshot with ranked entries

## Redis Data Model

- **Key**: `leaderboard:global`
- **Type**: Sorted Set (ZSET)
- **Member**: `userId` (String)
- **Score**: `score` (Double)


## How Ranking Works

1. Scores are stored in Redis ZSET
2. Leaderboard is fetched using `ZREVRANGE`
3. Ranks are assigned sequentially in memory
4. Snapshot is streamed back to the client


## Running the Server
1. Ensure Redis is running locally on default port (6379) by docker.  
```bash
 docker run -d --name redis -p 6379:6379 redis
 ```
2. Run the spring-boot application
```bash
 ./mvnw spring-boot:run
 ```

## Testing the Server with grpcurl 
- List services:
```bash
docker run --rm --network=host \
  fullstorydev/grpcurl \
  -plaintext localhost:9090 list
```
- Join Leaderboard Stream:
```bash
docker run --rm --network=host \
  fullstorydev/grpcurl \
  -plaintext \
  -d '{"join":{}}' \
  localhost:9090 \
  peak.LeaderboardService/streamLeaderboard
```
- Update Score:
```bash
docker run --rm --network=host \
  fullstorydev/grpcurl \
  -plaintext \
  -d '{"scoreUpdate":{"userId":"alice","score":500}}' \
  localhost:9090 \
  peak.LeaderboardService/streamLeaderboard
```