# CBSbest App — AI Agent Instructions

## Project Overview
A web app replacing text-based PGA tournament tracking. Members draft 4 golfers per event, enter round scores, and compete across seasons. The app enforces strict role-based permissions (Commissioner vs Member) and complex scoring logic including CUT/WD substitution.

## Critical Architecture Knowledge

### Permission Model (Must Implement Correctly)
- **Commissioner**: Full CRUD on golfers, seasons/events; can edit any team/score at any time; controls event status transitions; accesses Admin area
- **Member**: Opt-in during DRAFT; edit own picks (DRAFT only); enter scores for own team (IN_PROGRESS only); view all leaderboards
- **Enforcement Pattern**: Check role + event status + ownership (user_id) at both API and UI layers

### Event Status State Machine
```
DRAFT → IN_PROGRESS → FINAL
```
- Only commissioners transition states
- Transition to FINAL requires Round 4 complete (every opted-in participant rankable)
- No rollback in V1; commissioner can still edit after FINAL as override

### Scoring Computation (The "CUT/WD Substitution" Rule)
This is non-obvious and critical:
1. **CutScore(R)** = highest score_to_par from drafted golfers with result_status=PLAYED in round R
2. **Team round total** = best 3 scores from team's 4 golfers; if < 3 eligible, substitute CutScore(R) for each missing slot
3. **Round completeness** = CutScore computable AND every opted-in team has a GolferRoundScore (PLAYED/CUT/WD) for all 4 golfers in that round
4. **Cumulative total** = sum of team round totals through that round (null if any prior round incomplete)

**Example**: Team drafted [Golfer A, B, C, D] for an event. Round 1 CutScore=+5. If only A(+2), B(CUT), C(+1) have scores:
- Eligible scores: A(+2), C(+1)
- Team round total = +2 + +1 + (+5 substitution) = +8

### Season Points System
- Points awarded **only when event FINAL**
- Base STANDARD: 1st=10, 2nd=9, ..., 10th=1, 11th+=0
- Apply tier multiplier: ELEVATED=2×, MAJOR=3×
- Ties: competition ranking (multiple players at rank 1 both get rank 1 points)

## Key Tables & Foreign Key Constraints
```
User (id, display_name, role)
Season (id, name)
SeasonPointsConfig (season_id PK/FK, standard_points_json, multipliers)
Event (id, season_id FK, name, status, tier)
Golfer (id, full_name, nationality nullable) — global/shared
EventParticipant (event_id FK, user_id FK, unique(event_id, user_id))
TeamPick (event_id FK, user_id FK, golfer_id FK, pick_number 1-4)
  - unique(event_id, golfer_id) ← enforces no duplicate golfers across teams
  - unique(event_id, user_id, pick_number)
GolferRoundScore (event_id FK, golfer_id FK, round_number 1-4, result_status, score_to_par nullable, updated_by_user_id, updated_at)
  - unique(event_id, golfer_id, round_number)
  - result_status in [PLAYED, CUT, WD]
  - score_to_par required when PLAYED
```

## Common Implementation Patterns

### Leaderboard Ranking Logic
```
1. Filter to GolferRoundScore for current round with result_status records
2. Compute each team's round total (apply CUT/WD substitution if needed)
3. If round incomplete: return ranks=null, show "Pending" state
4. If complete: sort by (cumulative_total asc, round_total asc, user display_name asc)
5. Apply competition ranking (1,1,3,4...)
```

### Member Edit Restrictions
- Member can edit own picks **only if event status = DRAFT**
- Member can edit own scores **only if event status = IN_PROGRESS**
- Commissioner overrides both (can edit any team/score any time)
- Enforce at API layer with explicit status/permission checks

### Opt-in Workflow
- Members opt-in creates EventParticipant row (unique constraint prevents duplicates)
- Can only opt-in during DRAFT status
- Commissioner can add/remove participants in DRAFT (admin override)

## Development Workflow
- Test against `/docs/CBSBest-app-MVP.txt` specifications, not assumptions
- Create test documentation under `/docs` as needed
- Run full test suite before marking round/event complete
- Always verify CUT/WD substitution logic in scoring tests

## API Endpoints (High-Level Contract)
- `/api/golfers` — GET/POST/PATCH (commissioner-only write)
- `/api/seasons/{seasonId}/overview` — GET with standings + events
- `/api/events/{eventId}/status` — POST (commissioner; enforce transition rules)
- `/api/events/{eventId}/teams/{userId}/picks` — PUT (member own DRAFT, commissioner override)
- `/api/events/{eventId}/scores?round=R` — GET/PUT (member own IN_PROGRESS, commissioner override)
- `/api/events/{eventId}/leaderboard?round=R` — GET (computed with completeness flags)

## References
- **AI guidelines**: `/docs/ai.md`
-
