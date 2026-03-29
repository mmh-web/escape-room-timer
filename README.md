# Escape Room Countdown Timer

A dark, military-themed escape room countdown timer with code entry, sound effects, and a cloud leaderboard. Pure HTML/CSS/JS — no build step. Designed for GitHub Pages.

## Setup

### 1. Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add Project**, name it (e.g., `escape-room-timer`), and create it
3. In the project dashboard, click **Build → Realtime Database → Create Database**
4. Choose your region and start in **test mode** (you'll lock it down in step 3)
5. Go to **Project Settings** (gear icon) → **General** → scroll to **Your apps** → click the **Web** icon (`</>`)
6. Register a web app (any nickname), and copy the `firebaseConfig` object

### 2. Paste Firebase Config

Open `index.html` and find this block near the top of the `<script>`:

```js
const FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Replace every `YOUR_*` value with the real values from your Firebase config.

### 3. Set Firebase Security Rules

In Firebase Console → Realtime Database → **Rules**, paste:

```json
{
  "rules": {
    "escape-room-leaderboard": {
      "events": {
        "$eventId": {
          "scores": {
            ".read": true,
            "$scoreId": {
              ".write": "!data.exists()",
              ".validate": "newData.hasChildren(['teamName', 'timeRemaining', 'totalDuration', 'elapsedTime', 'completed', 'lettersFound', 'timestamp'])
                && newData.child('teamName').isString()
                && newData.child('teamName').val().length > 0
                && newData.child('teamName').val().length <= 50
                && newData.child('timeRemaining').isNumber()
                && newData.child('timeRemaining').val() >= 0
                && newData.child('timeRemaining').val() <= 3600
                && newData.child('totalDuration').isNumber()
                && newData.child('totalDuration').val() >= 0
                && newData.child('totalDuration').val() <= 3600
                && newData.child('elapsedTime').isNumber()
                && newData.child('elapsedTime').val() >= 0
                && newData.child('elapsedTime').val() <= 3600
                && newData.child('completed').isBoolean()
                && newData.child('lettersFound').isNumber()
                && newData.child('lettersFound').val() >= 0
                && newData.child('lettersFound').val() <= 4"
            }
          }
        }
      }
    }
  }
}
```

This allows:
- Anyone can **read** scores (leaderboard is public)
- Anyone can **write new** entries (no auth — controlled environment)
- No one can **update or delete** existing entries from the client
- Writes are validated for correct data types and ranges

### 4. Add Music Tracks (Optional)

Place up to three MP3 files in the same directory as `index.html`:
- `track1.mp3`
- `track2.mp3`
- `track3.mp3`

Any missing tracks are automatically grayed out in the sound menu. The timer works fine with no music files.

### 5. Deploy to GitHub Pages

1. Create a GitHub repo and push the `timer/` folder contents (at minimum `index.html`)
2. Go to **Settings → Pages**
3. Set source to **Deploy from a branch**, select `main` (or `master`), root `/`
4. Your timer will be live at `https://yourusername.github.io/your-repo/`

## Usage

1. **Admin panel** loads on first visit — set duration, exit code, operation name
2. Hit **Start Mission** → team enters their name → **Begin Mission** starts the clock
3. Students tap code boxes and type letters to crack the exit code
4. On victory, the score auto-submits to Firebase and the leaderboard appears
5. **Next Team** (PIN-protected) resets for the next session — all settings preserved
6. Admin access mid-session: tap the 🔒 icon → enter PIN → pause/add time/reset boxes

All settings persist in `localStorage` — browser crashes preserve your config.
