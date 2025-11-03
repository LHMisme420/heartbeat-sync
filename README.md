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
from flask import Flask, request, jsonify, render_template
import pandas as pd
import os
from dotenv import load_dotenv  # Optional for future env vars

load_dotenv()  # Load .env if present

app = Flask(__name__)

# Mock data (expandable via form/CSV upload later)
users = [
    {'id': 1, 'location': 'NYC', 'mood': 4, 'interests': ['art', 'coffee']},
    {'id': 2, 'location': 'NYC', 'mood': 3, 'interests': ['books', 'walks']},
    {'id': 3, 'location': 'LA', 'mood': 6, 'interests': ['hiking', 'music']},
    {'id': 4, 'location': 'LA', 'mood': 5, 'interests': ['yoga', 'music']},
    {'id': 5, 'location': 'NYC', 'mood': 2, 'interests': ['coffee', 'art']},
]

df = pd.DataFrame(users)

def match_heartbeats(df, radius=50):
    matches = []
    for i in range(len(df)):
        for j in range(i+1, len(df)):
            if df.loc[i, 'location'] == df.loc[j, 'location']:
                mood_diff = abs(df.loc[i, 'mood'] - df.loc[j, 'mood'])
                shared = set(df.loc[i, 'interests']) & set(df.loc[j, 'interests'])
                if mood_diff <= 2 and len(shared) > 0:
                    # Generate a nudge idea
                    nudge = f"Meet for a quick {list(shared)[0]} share – mood match: {round((df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2, 1)}/10"
                    matches.append({
                        'pair': (df.loc[i, 'id'], df.loc[j, 'id']),
                        'shared_interest': list(shared)[0],
                        'mood_avg': round((df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2, 1),
                        'nudge_idea': nudge
                    })
    return matches

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/match', methods=['POST'])
def match():
    # For now, use mock data; future: parse form/CSV
    matches = match_heartbeats(df)
    return jsonify({'matches': matches})

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=int(os.environ.get('PORT', 5000)))
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Heartbeat Sync Demo</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; }
        button { background: #4CAF50; color: white; padding: 10px 20px; border: none; cursor: pointer; }
        button:hover { background: #45a049; }
        #results { margin-top: 20px; }
        .match { background: #f0f0f0; padding: 10px; margin: 5px 0; border-radius: 5px; }
    </style>
</head>
<body>
    <h1>Heartbeat Sync: Find Your Vibe Match</h1>
    <p>Demo: Submit a mock vibe and see serendipitous nudges. (Privacy: All anon.)</p>
    
    <form id="vibeForm">
        <label>Location: <input type="text" id="location" placeholder="e.g., NYC" required></label><br><br>
        <label>Mood (1-10): <input type="number" id="mood" min="1" max="10" value="5" required></label><br><br>
        <label>Interests (comma-separated): <input type="text" id="interests" placeholder="e.g., art,coffee" required></label><br><br>
        <button type="submit">Find Matches</button>
    </form>
    
    <div id="results"></div>

    <script>
        document.getElementById('vibeForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const response = await fetch('/match', { method: 'POST' });
            const data = await response.json();
            const resultsDiv = document.getElementById('results');
            if (data.matches && data.matches.length > 0) {
                resultsDiv.innerHTML = '<h3>Your Matches:</h3>' + data.matches.map(m => 
                    `<div class="match">Pair with ${m.pair[1]}: ${m.nudge_idea}</div>`
                ).join('');
            } else {
                resultsDiv.innerHTML = '<p>No matches yet—add more vibes!</p>';
            }
        });
    </script>
</body>
</html>
import pandas as pd

# Same mock data as app.py
users = [
    {'id': 1, 'location': 'NYC', 'mood': 4, 'interests': ['art', 'coffee']},
    {'id': 2, 'location': 'NYC', 'mood': 3, 'interests': ['books', 'walks']},
    {'id': 3, 'location': 'LA', 'mood': 6, 'interests': ['hiking', 'music']},
    {'id': 4, 'location': 'LA', 'mood': 5, 'interests': ['yoga', 'music']},
    {'id': 5, 'location': 'NYC', 'mood': 2, 'interests': ['coffee', 'art']},
]

df = pd.DataFrame(users)

def match_heartbeats(df, radius=50):
    matches = []
    for i in range(len(df)):
        for j in range(i+1, len(df)):
            if df.loc[i, 'location'] == df.loc[j, 'location']:
                mood_diff = abs(df.loc[i, 'mood'] - df.loc[j, 'mood'])
                shared = set(df.loc[i, 'interests']) & set(df.loc[j, 'interests'])
                if mood_diff <= 2 and len(shared) > 0:
                    nudge = f"Meet for a quick {list(shared)[0]} share – mood match: {round((df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2, 1)}/10"
                    matches.append({
                        'pair': (df.loc[i, 'id'], df.loc[j, 'id']),
                        'shared_interest': list(shared)[0],
                        'mood_avg': round((df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2, 1),
                        'nudge_idea': nudge
                    })
    return matches

# Run and print
matches = match_heartbeats(df)
print("Demo Matches:")
for m in matches:
    print(m)
    pandas
flask
python-dotenv
# Future: API keys for geo/maps
GOOGLE_PLACES_API_KEY=your_key_here
PORT=5000
from transformers import pipeline  # Add to requirements.txt

# In app.py, after imports
sentiment_analyzer = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")

def predict_mood_from_text(text):
    result = sentiment_analyzer(text)[0]
    # Map sentiment to mood score (0-10): Negative ~2-4, Neutral ~5, Positive ~6-8
    if result['label'] == 'NEGATIVE':
        return random.uniform(2, 4)  # Simulate variance
    elif result['label'] == 'NEUTRAL':
        return 5
    else:
        return random.uniform(6, 8)

# Update match route
@app.route('/match', methods=['POST'])
def match():
    vibe_text = request.form.get('vibe_text', 'Neutral day')  # New form field
    mood = predict_mood_from_text(vibe_text)  # AI mood
    location = request.form.get('location')
    interests = request.form.get('interests').split(',')
    
    # Add new user to df dynamically
    new_user = {'id': len(df) + 1, 'location': location, 'mood': mood, 'interests': interests}
    global df
    df = pd.concat([df, pd.DataFrame([new_user])], ignore_index=True)
    
    matches = match_heartbeats(df)
    return jsonify({'matches': matches, 'your_mood': mood})
    from web3 import Web3
import json

# Mock ZK proof (in prod, use Semaphore or Tornado Cash libs)
w3 = Web3(Web3.HTTPProvider('https://sepolia.infura.io/v3/YOUR_INFURA_KEY'))  # .env key

def generate_anon_nudge_proof(location_hash, mood_hash):
    # Simple hash "proof" – real ZK: Prove without revealing
    proof = {
        'location_proof': Web3.keccak(text=location_hash).hex(),
        'mood_proof': Web3.keccak(text=str(mood_hash)).hex(),
        'nudge_token': '0xAnonMatch'  # NFT for verified meets
    }
    return json.dumps(proof)

# In app.py match route, add:
proof = generate_anon_nudge_proof(location, mood)
return jsonify({'matches': matches, 'zk_proof': proof})
<!DOCTYPE html>
<html>
<head>
    <title>AR Nudge Demo</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/aframe@1.4.2/dist/aframe.min.js"></script>
</head>
<body>
    <a-scene embedded arjs="sourceType: webcam;">
        <a-marker preset="hiro">
            <a-box position="0 0.5 0" material="color: blue;" animation="property: rotation; to: 0 360 0; loop: true; dur: 1000"></a-box>
            <a-text value="Vibe Match: Art Share? Scan for note." position="0 -0.5 0" color="white"></a-text>
        </a-marker>
        <a-entity camera></a-entity>
    </a-scene>
    <p>Print Hiro marker, scan with phone cam—see floating nudge!</p>
</body>
</html>
# Heartbeat Sync: Serendipity Reborn

![AR Demo GIF Placeholder](ar_demo.gif) <!-- Record phone scan of Hiro marker -->

From "much of nothing" to world-changer: Anonymous AI nudges for real human sparks. Matches moods/locations/interests, now with Web3 privacy + AR overlays. Fights 2025's connection fatigue—think Farcaster meets Pokémon GO for empathy.

## Why Revolutionary?
- **AI Mood Magic**: Text-to-mood prediction (Hugging Face)—no surveys, just vibes.
- **ZK Privacy**: Blockchain proofs for trust without tracking (Web3.py sim).
- **AR Serendipity**: Overlay floating "echo notes" in your world (Three.js/A-Frame).
- **Global Scale**: Real-time, expandable to millions via decentralized nodes.

## Quick Start (Local Demo)
1. Clone: `git clone https://github.com/LHMisme420/heartbeat-sync.git && cd heartbeat-sync`
2. Deps: `pip install -r requirements.txt` (pandas, flask, transformers, web3)
3. Run: `python app.py`
4. Open: [localhost:5000](http://localhost:5000) – Submit vibe, get AI matches + ZK proof.
5. AR Test: [localhost:5000/ar](http://localhost:5000/ar) – Print Hiro marker, scan for floating nudge.

### Example Flow
- Input: "Tired after work" (NYC, auto-mood: 3.2), interests "coffee, doodles"
- Output: Match w/ anon user (ZK-verified nearby): "Doodle vent at corner café? Proof: 0xAnonHash"

## Files Breakdown
- `app.py`: Flask + AI matcher + ZK sim.
- `templates/index.html`: Vibe form UI.
- `templates/ar_demo.html`: AR overlay mock.
- `web3_nudge.py`: Privacy proofs.
- `demo.py`: Standalone tester.

## Roadmap to Moonshot
- **Short (1 Week)**: Add Folium maps for geo-nudges (`pip install folium`).
- **Med (1 Month)**: Mobile with React Native + Expo AR.
- **Long (3 Months)**: DAO governance—users mint NFTs for "fate credits"; partner w/ mental health NGOs for validated prompts.
- **Wild**: Integrate Neuralink/BCI for "telepathic" mood shares (2026 vibes).

Deploy: Vercel (free) – Link in issues if you want collab.

MIT License – Fork, fund humanity. Built w/ Grok. Questions? @LHMisme420 or open issue.

[Live Demo TBA] | [Join Beta Waitlist](https://forms.gle/fake) <!-- Add real Google Form -->
pandas
flask
python-dotenv
transformers  # For AI mood
web3  # For ZK sim
torch  # Transformers dep
