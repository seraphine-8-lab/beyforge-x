# BeyForge X — App Specification & Implementation Workflow

## CONCEPT

BeyForge X is a Beyblade X companion app that combines:
1. **Combo Builder** — browse parts, see images, get competitive viability data (already built as web v1)
2. **Match Tracker** — log 1v1 and 3v3 matches, track win/loss stats over time
3. **Tournament System** — create/share tournaments, judge mode with live split-screen scoring
4. **Deck Builder** — save your combos, build 3v3 decks, drag-and-drop ordering

---

## USER FLOWS

### Flow 1: Solo Testing (Simplest)
1. Player opens app → taps "Solo Test"
2. Selects 1v1 or 3v3
3. Picks their combo(s) from saved deck or builds new
4. Picks opponent combo(s) (optional — can leave blank)
5. Plays match → taps result button (Spin/Over/Burst/Extreme) for winner
6. App logs result, updates stats dashboard
7. Repeat as long as they want → session summary at end

### Flow 2: Friendly Match
1. Same as solo but two players each have their phone
2. Player 1 creates match → gets share code
3. Player 2 joins with code
4. Both pick their combos on their phones
5. One phone acts as judge → approves combos → enters play mode
6. Judge taps results → both phones see live score

### Flow 3: Tournament (Full)
1. **Organizer creates tournament:**
   - Name, format (Swiss/Single Elim/Double Elim/Round Robin)
   - 1v1 or 3v3
   - **Best of 1 or First to 4** (WBO standard match formats)
   - Max players
   - Private (passphrase) or Public
   - WBO rules by default or custom
2. **Players join:**
   - Enter passphrase or browse public list
   - Build/select their 3v3 deck (drag-and-drop order)
   - Confirm → deck locked for tournament
3. **Match setup:**
   - System pairs players based on format (Swiss pairing, bracket, etc.)
   - Players arrange combo order on their phones (drag-and-drop)
   - Confirm → pushes to tournament
   - Judge (organizer or assigned) approves on their phone
4. **Play mode (judge screen):**
   - Clean split-screen: Player 1 left, Player 2 right
   - Combo names + images on each side
   - Score counter
   - 4 result buttons per side: Spin, Over, Burst, Extreme
   - Penalty button per side (assigns -1 or custom penalty)
   - Tap winner's result → point awarded → round logged
   - Best of 3 → repeats until match decided
5. **Bracket advancement:**
   - System auto-advances winners based on format
   - Next round pairings generated
   - Repeat until champion
6. **Post-tournament:**
   - Final standings
   - Player stats updated (wins, losses, finish types, etc.)
   - Export results

### Flow 4: Stats Dashboard
- Win rate (overall, by combo, by format)
- Most used combos
- Most common finish types
- Win type breakdown (spin vs burst vs over vs extreme)
- Match history (scrollable list)
- Performance vs specific opponent combos

---

## FINISH TYPES & SCORING (WBO Standard)
| Finish Type | Points | Description |
|-------------|--------|-------------|
| Spin Finish | 1 | Opponent stops spinning first |
| Over Finish | 2 | Opponent knocked out of stadium |
| Burst Finish | 2 | Opponent's bey bursts apart |
| Extreme Finish | 3 | Opponent knocked out AND bursts (X-Dash) |

Best of 1 = single round
Best of 3 = first to 4 points (WBO standard)

---

## TOURNAMENT FORMATS

### Swiss (WBO Default)
- Players paired by record (winners vs winners, losers vs losers)
- Fixed number of rounds (usually 5-6)
- Top X players advance to finals (single elimination)
- Best for large groups

### Single Elimination
- Bracket style — lose once, eliminated
- Simple, fast
- Best for small groups or finals

### Double Elimination
- Winners bracket + losers bracket
- Must lose twice to be eliminated
- More forgiving, takes longer

### Round Robin
- Everyone plays everyone
- Best for small groups (4-8 players)
- Most accurate rankings

---

## TECHNICAL ARCHITECTURE

### Recommended: Progressive Web App (PWA)
**Why PWA over native:**
- One codebase, works on iOS + Android + desktop
- No app store review (instant updates)
- Free to host (Vercel/Cloudflare Pages)
- Works offline (service workers)
- Installable on phone home screen
- Can use push notifications, camera, etc.

**Trade-offs:**
- Slightly slower than native (not noticeable for this use case)
- iOS has some PWA limitations (no push notifications pre-iOS 16)
- App store discoverability lost (mitigated by SEO + community marketing)

**Recommendation: Start PWA, publish to stores later via TWA (Trusted Web Activity) wrapper for Android and Capacitor for iOS if needed.**

### Tech Stack
| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | React + Vite | Fast, huge ecosystem, easy to hire for |
| Styling | Tailwind CSS | Rapid, consistent, mobile-first |
| State | Zustand | Lightweight, no boilerplate |
| Backend | Supabase | Free tier, auth + database + realtime, PostgreSQL |
| Realtime | Supabase Realtime | Live tournament updates, judge sync |
| Hosting | Vercel | Free, auto-deploy from Git, edge CDN |
| Auth | Supabase Auth | Email/password, Google, Apple sign-in |

### Database Schema (Supabase/PostgreSQL)
```
users: id, email, display_name, created_at
combos: id, user_id, blade_id, ratchet_id, bit_id, name, created_at
decks: id, user_id, name, combo_ids[], created_at
matches: id, user_id, opponent_combo_id, my_combo_id, format, result, finish_type, created_at
tournaments: id, organizer_id, name, format, type, status, passphrase, rules_json, created_at
tournament_players: tournament_id, user_id, deck_id, joined_at
tournament_rounds: id, tournament_id, round_num, player1_id, player2_id, status, winner_id
tournament_matches: id, round_id, player1_combo_id, player2_combo_id, p1_score, p2_score, finish_log_json, status
```

### Realtime Features
- Tournament lobby: live player list, deck lock status
- Match judge: live score sync to both players' phones
- Tournament bracket: live updates as matches finish

---

## IMPLEMENTATION PHASES

### Phase 0: Foundation (Week 1)
- Set up Vite + React + Tailwind + Supabase
- User auth (email/password + Google)
- Basic routing
- Port combo builder from v1 web app
- Deploy to Vercel (free)

### Phase 1: Combo Builder + Deck Manager (Week 1-2)
- Browse all parts with images and tier data (already built)
- Save combos to account
- Build 3v3 decks (drag-and-drop ordering)
- Edit/delete saved combos and decks

### Phase 2: Solo Match Tracker (Week 2-3)
- Select 1v1 or 3v3
- Pick your combo(s) and opponent combo(s)
- Result buttons: Spin (1pt), Over (2pt), Burst (2pts), Extreme (3pts)
- Session tracking (consecutive matches)
- Stats dashboard (win rate, finish breakdown, combo performance)
- Match history

### Phase 3: Friendly Match (Week 3-4)
- Create match → share code
- Join match with code
- Both players pick combos
- Judge mode (one phone) → approve → play mode
- Split-screen play mode with live scoring
- Sync results to both phones

### Phase 4: Tournament System (Week 4-6)
- Create tournament (format, rules, passphrase)
- Join with passphrase
- Deck lock system
- Swiss pairing algorithm
- Bracket generation (single/double elim)
- Judge play mode (split-screen, same as friendly)
- Live bracket updates
- Final standings + export

### Phase 5: Polish + Publish (Week 6-7)
- PWA install prompt + offline support
- Performance optimization (lazy load images)
- SEO (meta tags, sitemap, structured data)
- Social sharing (combo cards, tournament results)
- Android: wrap as TWA → publish to Google Play ($25 one-time fee)
- iOS: wrap with Capacitor → publish to App Store ($99/year)
- Community marketing (Reddit, Discord, YouTube)

---

## APP PUBLISHING GUIDE

### Google Play Store
- **Cost:** $25 one-time developer registration fee
- **Requirements:** Google account, verify identity
- **Process:** Create developer account → upload AAB → fill store listing → review (1-3 days)
- **TWA approach:** Wrap PWA in Trusted Web Activity → behaves like native app, uses web tech

### Apple App Store
- **Cost:** $99/year Apple Developer Program
- **Requirements:** Apple ID, two-factor auth, verify identity
- **Process:** Create developer account → use Capacitor/Cordova wrapper → upload via Xcode or Transporter → review (1-7 days)
- **Note:** Apple is stricter — may reject if it looks too much like a website without native features

### Alternative: Just PWA (Free)
- Host on Vercel/Cloudflare Pages (free)
- Users install via "Add to Home Screen" on mobile
- No review process, instant updates
- Can be discovered via Google search (SEO)
- **This is the starting point — get users, then decide if app stores are worth it**

---

## COSTS
| Item | Cost | When |
|------|------|------|
| Vercel hosting | Free | Now |
| Supabase (50k users, 500MB DB) | Free | Now |
| Domain name | ~$12/year | Phase 0 |
| Google Play dev account | $25 one-time | Phase 5 |
| Apple dev account | $99/year | Phase 5 |
| **Total to start** | **$12** | |

---

## COMPETITIVE ADVANTAGE
- **Only living app** with competitive tier data + images + match tracking
- BeyCoachx (dead) had competitive data but no tracking, no images, no app
- BeyBuilderX (live) has basic tracking but no competitive data, no tournament system
- WBO has tournament rules but no digital tracking tool
- **We combine all three: competitive intelligence + tracking + tournaments**

---

## NEXT STEPS
1. You confirm the spec looks right
2. I set up the project (Vite + React + Supabase + Tailwind)
3. Start Phase 0 + 1 (foundation + combo builder + deck manager)
4. Deploy to Vercel (free, live in minutes)
5. Iterate from there