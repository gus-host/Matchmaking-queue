# Matchmaking Queue: A Real-Time Skill-Based System

The Matchmaking Queue is a real-time system that pairs users based on their skill rankings. It:

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

1. Initial Range: ±10 rank points.
2. Expands by 90 Points: If no match is found.
3. Max Rank Difference: 500 points.
4. Fair Matches: Users are paired with the closest available match.

This approach balances match fairness with queue times:

## Implementation Details

### Data Storage: 
Uses ETS tables for in-memory storage (active queue and completed matches).

### Concurrency Handling:
 Managed by GenServer, with Phoenix PubSub for real-time notifications.

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

Tests covers: Tests cover queue operations, match processing, GraphQL mutations, real-time subscriptions, and edge cases.

## Why Adopt This Tool

1. Efficient Pairing: Automates the matching process, ensuring fair pairings and scalability.

2. Customizability: Tailorable for gaming, hiring, social apps, and more.

3. Enhanced User Experience:  Reduced wait times and improved match quality.

4. Supports Advanced Logic: Supports priority queues, dynamic matching, and group matching.

5. Real-Time Notifications: Live updates and insights into matching trends.

6. Integration: Seamlessly integrates with APIs and real-time systems.

7. Open Source: Transparency and community-driven improvements.


**Adoption Scenarios** : Gaming, hiring, social apps, educational platforms, corporate tools.

### Additional Notes:
- Check out the follow_video.txt file to follow [video]()
- For a smooth development process, it’s recommended to run the commands on GitHub Codespaces, particularly for this repo. You can also fork this repository.
