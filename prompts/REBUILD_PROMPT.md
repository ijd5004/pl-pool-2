# Premier League Prediction Pool - Modern Rebuild Instructions

## Project Overview

Build a modern, real-time Premier League prediction tracking application that replaces an existing Python/Streamlit app. This is a prediction pool where 5 participants predict the final EPL table order, and the app tracks scores against actual standings throughout the season.

## Current Application Context

**Live Dashboard**: https://diamond-dawgs-pl-pool.streamlit.app/
**Participants**: 5 users with team-themed names
- IncogNeto (Chelsea fan)
- Sonny's Soldiers (Tottenham fan) 
- Wolf Cola (Wolves fan)
- Vardy Party (Leicester fan)
- Championship Playoff Champions (Southampton fan)

## Exact Requirements

### Scoring System (Critical - Must Match Exactly)
```python
def score_prediction(predicted_pos, actual_pos):
    if predicted_pos == actual_pos:
        return 10  # Exact position match
    elif abs(predicted_pos - actual_pos) <= 3:
        return 5   # Within 3 positions
    elif abs(predicted_pos - actual_pos) <= 5:
        return 2   # Within 5 positions
    elif (predicted_pos <= 10 and actual_pos <= 10) or (predicted_pos > 10 and actual_pos > 10):
        return 1   # Correct half of table
    return 0       # No points
```

### Data Structures

**Predictions CSV Format**:
```csv
Index,IncogNeto,Sonny's Soldiers,Wolf Cola,Vardy Party,Championship Playoff Champions
1,Manchester City FC,Manchester City FC,Arsenal FC,Manchester City FC,Arsenal FC
2,Arsenal FC,Arsenal FC,Manchester United FC,Liverpool FC,Manchester City FC
...
20,Sheffield United FC,Sheffield United FC,Burnley FC,Sheffield United FC,Burnley FC
```

**EPL API**: `https://api.football-data.org/v4/competitions/PL/standings`
- Requires `X-Auth-Token` header
- Returns JSON with `standings[0].table` array
- Each team has: `team.name`, `playedGames`, `won`, `draw`, `lost`, `goalsFor`, `goalsAgainst`, `goalDifference`, `points`

**Team Logos** (use these exact URLs):
```javascript
const TEAM_LOGOS = {
  'IncogNeto': 'https://upload.wikimedia.org/wikipedia/en/c/cc/Chelsea_FC.svg',
  'Sonny\'s Soldiers': 'https://upload.wikimedia.org/wikipedia/en/b/b4/Tottenham_Hotspur.svg',
  'Vardy Party': 'https://upload.wikimedia.org/wikipedia/hif/a/ab/Leicester_City_crest.png',
  'Wolf Cola': 'https://upload.wikimedia.org/wikipedia/en/f/fc/Wolverhampton_Wanderers.svg',
  'Championship Playoff Champions': 'https://upload.wikimedia.org/wikipedia/en/c/c9/FC_Southampton.svg'
}
```

## Technical Stack Requirements

**Frontend**: React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui
**Backend**: Vercel API Routes + Node.js + TypeScript
**Database**: Supabase (PostgreSQL + real-time + auth)
**Deployment**: Vercel (free tier)

## Required Database Schema (Supabase)

```sql
-- Teams table
CREATE TABLE teams (
  id SERIAL PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  logo_url TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Participants table  
CREATE TABLE participants (
  id SERIAL PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  logo_url TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Predictions table
CREATE TABLE predictions (
  id SERIAL PRIMARY KEY,
  participant_id INTEGER REFERENCES participants(id),
  team_name TEXT NOT NULL,
  predicted_position INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- EPL Standings table (current actual standings)
CREATE TABLE epl_standings (
  id SERIAL PRIMARY KEY,
  team_name TEXT NOT NULL,
  position INTEGER NOT NULL,
  played INTEGER NOT NULL,
  won INTEGER NOT NULL,
  drawn INTEGER NOT NULL,
  lost INTEGER NOT NULL,
  goals_for INTEGER NOT NULL,
  goals_against INTEGER NOT NULL,
  goal_difference INTEGER NOT NULL,
  points INTEGER NOT NULL,
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Scores table (historical scoring data)
CREATE TABLE scores (
  id SERIAL PRIMARY KEY,
  participant_id INTEGER REFERENCES participants(id),
  total_score INTEGER NOT NULL,
  scored_at TIMESTAMP DEFAULT NOW()
);

-- Individual prediction scores
CREATE TABLE prediction_scores (
  id SERIAL PRIMARY KEY,
  participant_id INTEGER REFERENCES participants(id),
  team_name TEXT NOT NULL,
  predicted_position INTEGER NOT NULL,
  actual_position INTEGER NOT NULL,
  points_earned INTEGER NOT NULL,
  scored_at TIMESTAMP DEFAULT NOW()
);
```

## Required Dashboard Features

### Tab 1: Prediction Scores (Main Leaderboard)
- Display participants ranked by total score
- Show participant name, total points, and team logo
- Real-time updates when EPL standings change
- 3-column layout: Name | Score | Logo

### Tab 2: Prediction Details
- Table showing all predictions with individual scores
- Columns: Position | IncogNeto | Sonny's Soldiers | Wolf Cola | Vardy Party | Championship Playoff Champions
- Each cell shows: "Team Name (X pts)" where X is points earned for that prediction
- Color coding: 10pts=green, 5pts=blue, 2pts=yellow, 1pt=orange, 0pts=red

### Tab 3: Current EPL Table
- Live EPL standings table
- Columns: Position | Team | Played | Won | Drawn | Lost | GF | GA | GD | Points
- Auto-updates when new data is fetched

### Tab 4: Score History
- Line chart showing score progression over time
- One line per participant
- X-axis: Date, Y-axis: Total Score
- Interactive chart with hover details

## API Endpoints Required

**GET /api/epl/standings** - Fetch current EPL standings
**POST /api/epl/update** - Update standings from Football-Data.org API (cron job)
**GET /api/predictions** - Get all predictions with current scores
**GET /api/scores/history** - Get historical score data for charts
**GET /api/participants** - Get all participants with logos

## Implementation Steps

1. **Project Setup**
   - Initialize Next.js/Vite project with TypeScript
   - Setup Tailwind CSS and shadcn/ui
   - Configure Supabase client

2. **Database Setup**
   - Create Supabase project
   - Run SQL schema creation
   - Setup Row Level Security policies
   - Seed initial data (participants, predictions)

3. **API Layer**
   - Create Vercel API routes for EPL data fetching
   - Implement scoring logic exactly as specified
   - Setup cron job for weekly EPL data updates
   - Add error handling and validation

4. **Frontend Components**
   - Create dashboard layout with 4 tabs
   - Build leaderboard component with real-time updates
   - Create prediction details table with color coding
   - Implement EPL standings table
   - Add score history chart with Recharts

5. **Real-time Features**
   - Setup Supabase real-time subscriptions
   - Connect components to live data updates
   - Add loading states and error boundaries

6. **Styling & Polish**
   - Apply consistent Tailwind styling
   - Add animations and transitions
   - Ensure mobile responsiveness
   - Add proper error handling

## Environment Variables Needed

```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
FOOTBALL_DATA_API_KEY=your_football_data_api_key
```

## Initial Data to Seed

Use the exact participant names and current predictions from the existing CSV. The new Claude should read the provided predictions.csv and sample_epl_table.csv files to understand the exact data format and current state.

## Success Criteria

- Dashboard matches current functionality exactly
- Real-time updates work via Supabase subscriptions  
- Scoring algorithm produces identical results
- All 4 dashboard tabs implemented
- Mobile responsive design
- Deployed on Vercel free tier
- Zero monthly hosting costs

Create this as a production-ready application with proper error handling, TypeScript types, and modern React patterns.