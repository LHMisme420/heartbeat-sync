# heartbeat-sync
"Prototype for serendipitous human connections via mood/location matching."
git clone https://github.com/LHMISME420/heartbeat-sync.git
import random
import pandas as pd

# Mock data for users: location, mood score (0-10), interests
users = [
    {'id': 1, 'location': 'NYC', 'mood': 4, 'interests': ['art', 'coffee']},
    {'id': 2, 'location': 'NYC', 'mood': 3, 'interests': ['books', 'walks']},
    {'id': 3, 'location': 'LA', 'mood': 6, 'interests': ['hiking', 'music']},
    {'id': 4, 'location': 'LA', 'mood': 5, 'interests': ['yoga', 'music']},
    {'id': 5, 'location': 'NYC', 'mood': 2, 'interests': ['coffee', 'art']},
]

df = pd.DataFrame(users)

def match_heartbeats(df, radius=50):  # Simple match: same city, similar mood +/-2, shared interest
    matches = []
    for i in range(len(df)):
        for j in range(i+1, len(df)):
            if df.loc[i, 'location'] == df.loc[j, 'location']:
                mood_diff = abs(df.loc[i, 'mood'] - df.loc[j, 'mood'])
                shared = set(df.loc[i, 'interests']) & set(df.loc[j, 'interests'])
                if mood_diff <= 2 and len(shared) > 0:
                    matches.append({
                        'pair': (df.loc[i, 'id'], df.loc[j, 'id']),
                        'shared_interest': list(shared)[0],
                        'mood_avg': (df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2
                    })
    return matches

# Run matcher
matches = match_heartbeats(df)
print(matches)
# Heartbeat Sync

An opt-in app prototype for anonymous, serendipitous human connections based on mood, location, and shared interests. Fights "connection fatigue" in a digital world.

## Features
- Simple matching engine using pandas for vibe-based pairs.
- Privacy-first: No profiles, just low-key nudges.
- Expandable: Add geo, voice notes, etc.

## Setup
1. Clone the repo.
2. Install deps: `pip install pandas`
3. Run: `python matcher.py`

## Output Example

Built with Grok – let's iterate!
pandas
random
__pycache__/
*.pyc
.DS_Store
.ipynb_checkpoints/
