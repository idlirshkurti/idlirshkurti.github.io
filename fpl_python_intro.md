---
layout: page
title: FPL python
parent: FPL
description: Using python to get data from the FPL API
nav_order: 1
tags: [python, fpl, api]
---

# Guide: Accessing Data from the Fantasy Premier League (FPL) API for Specific Players, Gameweeks, and Managers
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The **Fantasy Premier League (FPL) API** allows you to programmatically access various types of data about players, matches, and teams. This guide walks you through retrieving data for specific players, gameweeks, and managers.

#### 1. **API Overview**

The FPL API (although slow :D) is a publicly available, undocumented API that does not require authentication. You can query it directly using any HTTP client such as `requests` in Python.

#### 2. **Getting Started**
To begin, install the required libraries if they are not already installed:

```bash
pip install requests
```

Then, import the necessary module in your script:

```python
import requests
```

#### 3. **Base URL**
The base URL for the FPL API is:

```
https://fantasy.premierleague.com/api/
```

All requests will be made to endpoints under this base URL.

---

### 4. **Retrieve Data for Specific Players**

Each player in the FPL has a unique `player_id`. You can use the following endpoint to retrieve data about a specific player:

#### Endpoint:
```
https://fantasy.premierleague.com/api/element-summary/{player_id}/
```

#### Example:
Here’s an example of how to get detailed statistics for a player using their `player_id` (e.g., 302 for Mohamed Salah):

```python
def get_player_data(player_id):
    url = f"https://fantasy.premierleague.com/api/element-summary/{player_id}/"
    response = requests.get(url)
    return response.json()

player_data = get_player_data(302)  # Mohamed Salah's player_id
print(player_data)
```

This response includes various stats for the player across all gameweeks, including goals, assists, minutes played, and more.

---

### 5. **Retrieve Data for Specific Gameweeks**

To access gameweek-specific data, you need to extract information related to a specific **gameweek** from the FPL API. Here's how you can get data on a particular gameweek:

#### Endpoint:
```
https://fantasy.premierleague.com/api/event/{gameweek_id}/live/
```

#### Example:
To get live data for a specific gameweek (e.g., gameweek 10):

```python
def get_gameweek_data(gameweek_id):
    url = f"https://fantasy.premierleague.com/api/event/{gameweek_id}/live/"
    response = requests.get(url)
    return response.json()

gameweek_data = get_gameweek_data(10)
print(gameweek_data)
```

This provides a breakdown of each player's performance for the specific gameweek.

---

### 6. **Retrieve Data for Specific Managers (User Teams)**

Managers or users in the FPL have a unique `manager_id`. You can retrieve a manager’s team details for the current season or a specific gameweek.

#### Endpoint:
```
https://fantasy.premierleague.com/api/entry/{manager_id}/
```

#### Example:
To get details about a specific manager (e.g., manager ID 1234567):

```python
def get_manager_data(manager_id):
    url = f"https://fantasy.premierleague.com/api/entry/{manager_id}/"
    response = requests.get(url)
    return response.json()

manager_data = get_manager_data(1234567)
print(manager_data)
```

This will provide information about the manager’s history, leagues they are in, total points, etc.

#### To Get a Manager's Team for a Specific Gameweek:

#### Endpoint:
```
https://fantasy.premierleague.com/api/entry/{manager_id}/event/{gameweek_id}/picks/
```

#### Example:
To get the team picks for a specific gameweek (e.g., gameweek 10) of a particular manager:

```python
def get_manager_team_for_gameweek(manager_id, gameweek_id):
    url = f"https://fantasy.premierleague.com/api/entry/{manager_id}/event/{gameweek_id}/picks/"
    response = requests.get(url)
    return response.json()

team_picks = get_manager_team_for_gameweek(1234567, 10)
print(team_picks)
```

This returns a list of the players the manager has selected for that gameweek, including which players are on the bench and who the captain is.

---

### 7. **Combining Data for a Player, Gameweek, and Manager**

You can combine the above methods to extract specific data. For example, to retrieve performance stats for players on a manager’s team in a given gameweek:

1. Get the manager’s team picks for the gameweek.
2. Loop through the list of players, retrieving each player’s detailed gameweek stats using the player and gameweek API endpoints.

#### Example:

```python
def get_manager_team_performance(manager_id, gameweek_id):
    team_picks = get_manager_team_for_gameweek(manager_id, gameweek_id)
    player_performance = []
    
    for pick in team_picks['picks']:
        player_id = pick['element']
        player_data = get_player_data(player_id)
        player_performance.append({
            'player_id': player_id,
            'gameweek_performance': player_data['history'][gameweek_id - 1]
        })
    
    return player_performance

manager_team_performance = get_manager_team_performance(1234567, 10)
print(manager_team_performance)

```

This will give you the performance of each player on the manager's team for gameweek 10.

---

### 8. **Additional Data You Can Access**

- **General FPL Game Data**:
    - Use the endpoint `https://fantasy.premierleague.com/api/bootstrap-static/` to retrieve general game data, including player stats, fixtures, and more.

- **Fixtures for a Gameweek**:
    - Use `https://fantasy.premierleague.com/api/fixtures/?event={gameweek_id}` to get fixtures for a specific gameweek.


The specific IDs (like 302 for Mohamed Salah) is available from the **bootstrap-static** endpoint:

### Retrieve Player IDs and Other Data

#### Endpoint:
```
https://fantasy.premierleague.com/api/bootstrap-static/
```

This endpoint returns a large JSON object containing all sorts of FPL data, including a list of players with their corresponding `player_id`s, teams, prices, positions, etc.

#### Example in Python:

```python
import requests

def get_all_players():
    url = "https://fantasy.premierleague.com/api/bootstrap-static/"
    response = requests.get(url)
    data = response.json()
    return data['elements']  # 'elements' contains the list of all players

all_players = get_all_players()

# Loop through players to find Salah
for player in all_players:
    if player['web_name'] == 'Salah':  # 'web_name' is the short name used in the game
        print(f"Player Name: {player['first_name']} {player['second_name']}")
        print(f"Player ID: {player['id']}")
        print(f"Team: {player['team']}")
        print(f"Price: {player['now_cost'] / 10}m")
        break
```

#### Output:

This will return details about Mohamed Salah, including his `player_id`, which is 302 in this case. You can similarly loop through all players to find IDs for any other player.

### Key Information in `bootstrap-static`:

- **`elements`**: Contains player data, including `id`, `web_name`, `team`, and stats.
- **`teams`**: Contains data about all the teams, such as `team_id` and team names.
- **`events`**: Information about gameweeks, deadlines, etc.

---

### Example of Key Fields for a Player (Mohamed Salah):

Here is what the player entry for Salah might look like in the `elements` array:

```json
{
    "id": 302,
    "photo": "302.jpg",
    "web_name": "Salah",
    "team_code": 14,
    "status": "a",
    "code": 123456,
    "first_name": "Mohamed",
    "second_name": "Salah",
    "squad_number": 11,
    "news": "",
    "now_cost": 127,  # Cost in tenths (12.7m)
    "chance_of_playing_this_round": 100,
    "total_points": 100,
    "minutes": 900,
    ...
}
```

This contains Salah's `id`, `web_name`, `team_code`, `now_cost`, and many other stats. You can easily find any player's `id` from this data by searching by `web_name` or `first_name/second_name`.