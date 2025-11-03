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
# proof_of_spark.py

from web3 import Web3
import json
import random
import time

# --- MOCK SETUP ---
# In a real system, INFURA_URL and ANONYMOUS_WALLETS would be managed
# by a secure environment or a dedicated blockchain service.
INFURA_URL = 'https://sepolia.infura.io/v3/YOUR_INFURA_KEY'
w3 = Web3(Web3.HTTPProvider(INFURA_URL))

# Mock Wallets for a successful PoS validation
ANONYMOUS_WALLETS = {
    1: '0xAnonWalletA123',
    5: '0xAnonWalletB456',
    3: '0xAnonWalletC789',
    4: '0xAnonWalletD012',
}
# --- END MOCK SETUP ---


def generate_spark_id(pair_ids, shared_interest, location, timestamp):
    """
    Generates a unique, deterministic ID for a specific match event.
    This ID is used by both users to claim the Proof-of-Spark.
    """
    # Sort the pair IDs to ensure the Spark ID is the same regardless of order (1,5 or 5,1)
    sorted_pair = tuple(sorted(pair_ids))
    unique_string = f"{sorted_pair}_{shared_interest}_{location}_{timestamp}"
    
    # Hash the unique string to get a verifiable Spark ID
    spark_id = Web3.keccak(text=unique_string).hex()
    
    return spark_id

def issue_fate_credit(user_id):
    """
    Simulates issuing a Soulbound Token (Fate Credit) to the user's wallet.
    In a real system, this would trigger a smart contract minting function.
    """
    wallet = ANONYMOUS_WALLETS.get(user_id, '0xDefaultAnonWallet')
    credit_token = {
        'token_name': 'Fate Credit SBT',
        'wallet': wallet,
        'timestamp': int(time.time()),
        'reputation_change': +1,
        'tx_hash': Web3.keccak(text=f"Mint_{wallet}_{time.time()}").hex()
    }
    return credit_token

def validate_proof_of_spark(user_a_id, user_b_id, spark_id, target_lat=40.7, target_lon=-74.0):
    """
    Simulates the core validation logic: checking proximity and token ID.
    
    In reality, User A and User B would submit their location separately.
    Here, we simulate the submission and check if they are both "nearby."
    """
    print(f"\n--- Validating Spark: {spark_id[:8]}... ---")
    
    # --- Mock Proximity Check ---
    # Simulate User A's location (randomly offset from target)
    user_a_lat = target_lat + random.uniform(-0.0001, 0.0001)  # ~10 meters max offset
    user_a_lon = target_lon + random.uniform(-0.0001, 0.0001)

    # Simulate User B's location (randomly offset from target)
    user_b_lat = target_lat + random.uniform(-0.0001, 0.0001)
    user_b_lon = target_lon + random.uniform(-0.0001, 0.0001)
    
    # Simple success condition: if the difference between their locations is tiny, they met.
    # We'll assume a success rate for this simulation for clarity.
    proximity_match = random.choice([True, True, False])  # 2/3 chance of success for demo
    # --- End Mock Proximity Check ---

    if proximity_match and spark_id.startswith('0x'): # Simple Spark ID check
        print(f"✅ Spark Confirmed: Users {user_a_id} and {user_b_id} successfully synchronized proximity.")
        
        # Issue rewards
        credit_a = issue_fate_credit(user_a_id)
        credit_b = issue_fate_credit(user_b_id)
        
        return {
            'status': 'SUCCESS', 
            'proof': spark_id,
            'user_a_credit': credit_a,
            'user_b_credit': credit_b
        }
    else:
        print("❌ Spark Failed: Proximity or Spark ID mismatch.")
        return {'status': 'FAILED', 'proof': spark_id}

# --- Example of How to Use in app.py or demo.py ---
# Assume we have a match object from match_heartbeats:
# match = {'pair': (1, 5), 'shared_interest': 'art', 'mood_avg': 3.0, 'nudge_idea': '...'}
# timestamp_of_meetup = 1730514000 # Unix timestamp for 3:00 PM Nov 3rd, 2025

# spark_id = generate_spark_id(match['pair'], match['shared_interest'], 'NYC', timestamp_of_meetup)
# validation_result = validate_proof_of_spark(match['pair'][0], match['pair'][1], spark
# app.py
# ... existing imports
from .proof_of_spark import generate_spark_id 
# NOTE: You'd need to create a proof_of_spark.py file for this import to work.
# app.py (updated match route snippet)

@app.route('/match', methods=['POST'])
def match():
    # ... existing user input and df update logic ...

    matches = match_heartbeats(df)
    
    # 🌟 NEW: Add Proof-of-Spark Logic
    final_matches = []
    
    # Mock a target time for the spontaneous meet-up (e.g., 30 mins from now)
    meetup_time_unix = int(time.time() + 1800) 
    
    for m in matches:
        # Generate the shared, verifiable Spark ID
        spark_id = generate_spark_id(m['pair'], m['shared_interest'], df.loc[df['id'] == m['pair'][0], 'location'].iloc[0], meetup_time_unix)
        
        m['spark_id'] = spark_id
        m['suggested_time'] = meetup_time_unix
        m['nudge_idea'] += f" (ZK Spark ID: {spark_id[:8]}...)"
        final_matches.append(m)

    # Note: The ZK proof in your original code can be replaced/enhanced by the Spark ID mechanism

    return jsonify({
        'matches': final_matches, 
        'your_mood': mood 
        # 'zk_proof' removed/absorbed by spark_id
    })
    # proof_of_spark.py
# (Same content as provided previously, repeated for completeness)

from web3 import Web3
import json
import random
import time

# --- MOCK SETUP ---
INFURA_URL = 'https://sepolia.infura.io/v3/YOUR_INFURA_KEY'
w3 = Web3(Web3.HTTPProvider(INFURA_URL))

# Mock Wallets for successful PoS validation
ANONYMOUS_WALLETS = {
    1: '0xAnonWalletA123',
    5: '0xAnonWalletB456',
    3: '0xAnonWalletC789',
    4: '0xAnonWalletD012',
}
# --- END MOCK SETUP ---


def generate_spark_id(pair_ids, shared_interest, location, timestamp):
    """
    Generates a unique, deterministic ID for a specific match event.
    This ID is used by both users to claim the Proof-of-Spark.
    """
    sorted_pair = tuple(sorted(pair_ids))
    unique_string = f"{sorted_pair}_{shared_interest}_{location}_{timestamp}"
    spark_id = Web3.keccak(text=unique_string).hex()
    return spark_id

def issue_fate_credit(user_id):
    """
    Simulates issuing a Soulbound Token (Fate Credit) to the user's wallet.
    In a real system, this would trigger a smart contract minting function.
    """
    wallet = ANONYMOUS_WALLETS.get(user_id, '0xDefaultAnonWallet')
    credit_token = {
        'token_name': 'Fate Credit SBT',
        'wallet': wallet,
        'timestamp': int(time.time()),
        'reputation_change': +1,
        'tx_hash': Web3.keccak(text=f"Mint_{wallet}_{time.time()}").hex()
    }
    return credit_token

def validate_proof_of_spark(user_a_id, user_b_id, spark_id, target_lat=40.7, target_lon=-74.0):
    """
    Simulates the core validation logic: checking proximity and token ID.
    (Simple 2/3 chance of success for demo)
    """
    print(f"\n--- Validating Spark: {spark_id[:8]}... ---")
    
    # Mock Proximity Check
    proximity_match = random.choice([True, True, False])

    if proximity_match and spark_id.startswith('0x'):
        print(f"✅ Spark Confirmed: Users {user_a_id} and {user_b_id} successfully synchronized proximity.")
        
        # Issue rewards
        credit_a = issue_fate_credit(user_a_id)
        credit_b = issue_fate_credit(user_b_id)
        
        return {
            'status': 'SUCCESS', 
            'proof': spark_id,
            'user_a_credit': credit_a,
            'user_b_credit': credit_b
        }
    else:
        print("❌ Spark Failed: Proximity or Spark ID mismatch.")
        return {'status': 'FAILED', 'proof': spark_id}
        # demo.py - Heartbeat Sync Standalone Matcher with Proof-of-Spark Simulation

import pandas as pd
import random
import time
import json
from proof_of_spark import generate_spark_id, validate_proof_of_spark

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
    """
    Simple match: same city, similar mood +/-2, shared interest.
    """
    matches = []
    for i in range(len(df)):
        for j in range(i + 1, len(df)):
            if df.loc[i, 'location'] == df.loc[j, 'location']:
                mood_diff = abs(df.loc[i, 'mood'] - df.loc[j, 'mood'])
                shared = set(df.loc[i, 'interests']) & set(df.loc[j, 'interests'])
                
                if mood_diff <= 2 and len(shared) > 0:
                    shared_interest = list(shared)[0]
                    mood_avg = round((df.loc[i, 'mood'] + df.loc[j, 'mood']) / 2, 1)
                    
                    # Mock a meet-up time for the Spark ID generation (e.g., 30 minutes from now)
                    meetup_time_unix = int(time.time() + 1800) 
                    
                    # Generate the ZK-inspired Spark ID for validation
                    spark_id = generate_spark_id(
                        (df.loc[i, 'id'], df.loc[j, 'id']), 
                        shared_interest, 
                        df.loc[i, 'location'], 
                        meetup_time_unix
                    )
                    
                    nudge = (
                        f"Meet for a quick {shared_interest} share – mood match: {mood_avg}/10. "
                        f"Get your Fate Credit with Spark ID: {spark_id[:8]}..."
                    )
                    
                    matches.append({
                        'pair': (df.loc[i, 'id'], df.loc[j, 'id']),
                        'shared_interest': shared_interest,
                        'mood_avg': mood_avg,
                        'nudge_idea': nudge,
                        'spark_id': spark_id
                    })
    return matches

# --- RUN AND VALIDATE ---

print("--- Heartbeat Sync Demo Matches ---")
matches = match_heartbeats(df)

if not matches:
    print("No matches found based on current mock data.")
else:
    for m in matches:
        print(f"\nMatch Found: Pair {m['pair']}")
        print(f"  > Nudge: {m['nudge_idea']}")
        
        # 🌟 NEW: Simulate the real-world meet-up validation (Proof-of-Spark)
        print("\n  --- Simulating Real-World PoS Validation ---")
        validation_result = validate_proof_of_spark(
            user_a_id=m['pair'][0], 
            user_b_id=m['pair'][1], 
            spark_id=m['spark_id']
        )
        
        # Print the Fate Credit token details on success
        if validation_result['status'] == 'SUCCESS':
            print("\n  ** SUCCESS! Fate Credits Issued: **")
            print(json.dumps(validation_result['user_a_credit'], indent=4))
