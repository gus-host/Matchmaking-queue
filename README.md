# Matchmaking Queue

A real-time matchmaking system built with Elixir and GraphQL that pairs users based on their skill rankings.

## Overview

Queue of Matchmaking is a skill-based matchmaking system that:

- Accepts user requests to join a matchmaking queue
- Pairs users with similar skill rankings
- Notifies matched users in real-time through GraphQL subscriptions
- Maintains data integrity during concurrent operations
- Stores all data in-memory for fast processing

## Technical Stack

- **Elixir** - Primary programming language
- **Phoenix** - Web framework
- **Absinthe** - GraphQL implementation
- **ETS** - In-memory storage

#### Note: 
I recommend you run the command on github code space for a smooth development process, more preferrably the codespace for this repo. You could also folk this repository.

## Pre-Setup

1. Install Home Brew:
If you have brew already installed you don't need to run the following commands
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
You can then run the following commands:
```bash
echo >> /home/codespace/.bashrc
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/codespace/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"


sudo apt-get install build-essential
brew update
```

2. Install Elixir:

```bash
brew install elixir
```

## Setup

1. Install dependencies:

```bash
mix deps.get
```

2. Start the Phoenix server:

```bash
mix phx.server
```

The GraphQL endpoint will be available at `https://fictional-sniffle-j69v4xrqr45cpjrq-4000.app.github.dev/graphiql` i.e if you are using the codespace for this repo. If not replace https://fictional-sniffle-j69v4xrqr45cpjrq-4000.app.github.dev with your forwarded address

## API Usage

### Adding a User to Queue

Use the `addRequest` mutation to add a user to the matchmaking queue:

```graphql
mutation {
  addRequest(userId: "Player123", rank: 1500) {
    ok
    error
  }
}
```

Response on success:

```json
{
  "data": {
    "addRequest": {
      "ok": true,
      "error": null
    }
  }
}
```

### Viewing Queue Status

Get the current state of the matchmaking queue:

```graphql
query {
  queueStatus {
    userId
    rank
  }
}
```

Response:

```json
{
  "data": {
    "queueStatus": [
      {
        "userId": "Player123",
        "rank": 1500
      },
      {
        "userId": "Player456",
        "rank": 1550
      }
    ]
  }
}
```

### Viewing Match History

Get the history of completed matches:

```graphql
query {
  matchHistory(limit: 5) {
    timestamp
    users {
      userId
      rank
    }
  }
}
```

Response:

```json
{
  "data": {
    "matchHistory": [
      {
        "timestamp": 1703001234,
        "users": [
          {
            "userId": "Player123",
            "rank": 1500
          },
          {
            "userId": "Player456",
            "rank": 1480
          }
        ]
      }
    ]
  }
}
```

### Subscribing to Match Notifications

Subscribe to match notifications for a specific user:

```graphql
subscription {
  matchFound(userId: "Player123") {
    users {
      userId
      rank
    }
  }
}
```

When a match is found, subscribers receive:

```json
{
  "data": {
    "matchFound": {
      "users": [
        {
          "userId": "Player123",
          "rank": 1500
        },
        {
          "userId": "Player456",
          "rank": 1480
        }
      ]
    }
  }
}
```

## Matchmaking Logic

The system uses an expanding window approach to create fair matches:

1. Initial matching attempts to find users within ±10 rank points
2. If no match is found, the range expands by 90 points each iteration
3. Maximum rank difference allowed is 500 points
4. Users are paired with the closest available match within the current range

This approach balances match fairness with queue times:

- Quick matches for players when similarly ranked opponents are available
- Gradually relaxed requirements to prevent excessive wait times
- Hard limit on rank difference to prevent highly mismatched games

## Implementation Details

### Data Storage

- Uses ETS tables for in-memory storage:
  - `matchmaking_queue`: Ordered set for active queue (sorted by rank)
  - `matches`: Set for completed match history
- Protected access for data integrity
- Efficient rank-based matching using ETS ordered sets

### Concurrency Handling

- GenServer manages state and synchronizes operations
- ETS provides atomic operations for queue management
- Phoenix PubSub handles real-time notifications
- Absinthe manages subscription state

## Configuration

Key configurations (in `lib/queue_of_matchmaking/matchmaking_queue.ex`):

```elixir
@initial_rank_window 10      # Initial search window (±10)
@rank_window_increment 90    # Window expansion per iteration
@max_rank_window 500        # Maximum allowed rank difference
```

These values can be adjusted to balance match quality vs. queue times.

## Testing

Run the test suite:

```bash
mix test
```

Tests cover:

- Queue operations
- Match processing
- GraphQL mutations and queries
- Real-time subscriptions
- Concurrent request handling
- Edge cases and validation

## Reason why users should adopt this tool
The Matchmaking Queue Tool can be a valuable addition for users seeking to optimize and streamline matching processes in various scenarios, such as gaming, hiring, dating, or team-building. Here's why users should consider adopting it:

1. Efficient Pairing of Users
The tool automates the process of matching users based on predefined criteria, eliminating manual efforts. Whether it's matching players in a multiplayer game or pairing participants in a networking event, it ensures:

- Speedy matching.
- Fair and balanced pairings.
- Scalability to handle large user pools.

2. Customizability
The matchmaking queue can be tailored to fit diverse use cases:

- Gaming: Match players based on skill level, region, or latency.
- Hiring: Pair candidates with recruiters based on skill sets and job requirements.
- Social Apps: Facilitate connections based on shared interests, proximity, or preferences.

3. Enhanced User Experience
By reducing wait times and providing quality matches, users will enjoy:

- A more seamless experience.
- Increased engagement due to relevant pairings.
- Reduced frustration from mismatches or delays.

4. Supports Advanced Logic
The tool can incorporate:

- Priority Queues: Users with premium subscriptions or high priority can be matched faster.
- Dynamic Matching: Adapts to user behavior, preferences, or feedback over time.
- Group Matching: Facilitates team formations in collaborative settings.

5. Integration with Existing Systems
The tool is built to integrate seamlessly with platforms that:

- Use APIs for user management.
- Leverage GraphQL for query efficiency and data fetching.
- Need real-time matching through WebSocket or live updates.

6. Open Source and Community-Driven
If built as open-source (like the repository you're working on), users benefit from:

- Transparency of the codebase.
- The ability to modify and improve it for specific needs.
- Contributions from a community of developers, ensuring continuous improvement.

7. Real-Time Analytics and Insights
Users can gain valuable insights, such as:

- Matching trends.
- Bottlenecks in the queue.
- Success rates of matches.
These metrics can drive decision-making and optimize future matches.

### Adoption Scenarios
- Gaming Companies: To provide skill-based and latency-aware matchmaking.
- Educational Platforms: To pair students with mentors or collaborators.
- Social and Dating Apps: To ensure meaningful and satisfying connections.
- Corporate Tools: To streamline hiring or networking processes.
