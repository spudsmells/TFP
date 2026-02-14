# The Gayborhood Discord Bot - Claude Code Initial Prompt

## Introduction

I'm building a comprehensive Discord bot in Python that consolidates functionality from multiple existing bots for "The Gayborhood" Discord server. This bot will handle onboarding, tickets, age verification, and various community features with a distinctive tone and personality.

**Current Bots Being Replaced:**
- Arcane (XP/levelling system) - will be fully replaced with built-in XP tracking
- Custom intro bot (JavaScript, currently hosted on team member's PC) - will be replaced with new intro system
- Various ticketing bots - replaced with custom ticket system
- Auto-threading bots - replaced with configurable threading system
- Other partial-use bots as identified

**IMPORTANT:** We have an existing intro bot written in JavaScript that we're replacing. Before building the new intro system (Release 2), please analyse the existing bot's code to understand:
- Current validation rules and field requirements
- Staff review workflow
- User experience patterns
- Edge cases already handled
- Any special business logic

The existing intro bot repository is at: [GitHub URL - to be provided]

The bot includes a one-time migration command to import existing XP data from Arcane so no user progress is lost.

## Technical Stack

**Required Stack:**
- Python 3.10+
- discord.py 2.x (latest stable)
- Database: SQLite for development, PostgreSQL ready for production
- Environment variables via `.env` file
- Async/await architecture throughout

**API Integrations Needed:**
- Discord Bot API (obviously)
- Spotify API (for music link conversion)
- YouTube Data API (for music link conversion)

**XP System:**
- Build XP tracking internally (replacing Arcane bot)
- Track XP per user in database
- Award XP for: messages (with cooldown), voice chat time, reactions received
- Calculate levels from XP totals
- Fire milestone events at key levels (15, 25, 50, 69, 100+)

## Project Structure Request

Before we start coding, please:
1. **Analyse the existing JavaScript intro bot** (if repo URL provided)
   - Review validation logic, field requirements, workflows
   - Document edge cases and special handling
   - Note what works well and what could be improved
   - Extract any validation rules, regex patterns, or business logic
2. Review the full requirements document (pasted below)
3. Propose a modular project structure that supports phased releases
4. Recommend a database schema that covers all releases
5. Suggest which features to build first for a solid foundation
6. Identify any technical challenges or dependencies I should know about
7. Create a `config.yaml` template for role IDs, channel IDs, and settings

## Key Architectural Requirements

**Modularity:**
- Each release should be a separate module/cog where possible
- Features should be toggleable via config
- Code must be maintainable by future team members

**Error Handling:**
- Robust logging of all events (joins, ticket actions, DM failures, etc.)
- Graceful degradation when Discord API is unavailable
- Timer persistence (if bot restarts, timers should resume from database state)

**Database Requirements:**
- Store: users, intros, tickets, logs, verification status, XP levels
- Track: all staff actions, all user interactions, all DM successes/failures
- Support: querying for analytics (Release 9)

**Tone Requirements:**
All bot messages must sound like:
- A tired gay moderator
- Mildly sarcastic but never mean
- Concise and witty
- Absolutely never corporate or robotic
- Examples: "Use your words. Twenty characters minimum." / "Alright babe, ticket created."

## Configuration Requirements

Create a `config.yaml` that includes:
- Discord role IDs (Pending, Gaybor, all regional roles, staff roles)
- Channel IDs (ticket category, staff review, welcome channel, fallback channel, etc.)
- Timeout values (2 hours for member nudge, 12 hours for staff nudge)
- Rate limits for commands
- XP settings:
  - XP per message (with cooldown period)
  - XP per minute in voice chat
  - XP per reaction received
  - XP thresholds for each level
  - Milestone levels that trigger DMs (15, 25, 50, 69, 100)
  - Channels where XP is disabled (if any)
- Threading settings:
  - Channels where auto-threading is enabled
  - Thread name format templates (configurable per channel)
  - What triggers thread creation (media files, links, embeds, etc.)
- Feature flags for each release
- Channel restrictions for /bully and music conversion

## Database Schema Sketch

Please design tables for:
- Users (Discord IDs, roles, verification status, XP, current level, total messages, VC time)
- XP History (tracking XP gains for analytics and debugging)
- Intros (all submission data, approval status, staff who approved)
- Tickets (type, owner, opener, claimed status, timestamps, nudge tracking)
- Logs (event types, user IDs, staff IDs, details, timestamps)
- Bully insults (text, active status, who added it)
- Music conversions (tracking for rate limiting and analytics)
- Auto threads (channel configurations, thread name formats, triggers)

---

# FULL REQUIREMENTS DOCUMENT

## Overall Bot Concept

**THE NEW GAYBOURHOOD BOT EXTRAVAGANZA**

ONE BOT TO RULE THEM ALL.

This bot handles:
- Member onboarding with rule acknowledgment
- Introduction system with staff approval
- Support ticket system (member-led and staff-led)
- Age verification for NSFW access
- Built-in XP tracking and levelling system (replacing Arcane)
- XP milestone notifications and rewards
- Automatic thread creation on media posts (configurable per channel)
- Fun commands (/bully)
- Automatic music link conversion (Spotify → YouTube)

**Phased Development Approach:**
The bot will be built across 9+ releases, with each release adding specific features.

---

## RELEASE 1 - The Prologue to Penetration
### Core Bot and Basic Onboarding DM

**Goal:** Create a stable foundation. On join, assign Pending role, attempt DM with rules, prepare for full onboarding flow.

### 1. Core Bot Setup

**1.1 Structure**
- Single project folder
- `.env` file for all secrets (bot token, API keys)
- No hardcoded credentials

**1.2 Logging**
- Log: startup, shutdown, reconnects, errors
- Log: join events, DM attempts, fallback triggers
- Log: button interactions

**1.3 Diagnostic Slash Commands**
- `/ping` → basic response
- `/version` → bot version/release ID  
- `/status` → health check

### 2. On Member Join → Onboarding DM

**2.1 Assign Pending**
- Give Pending role immediately on join
- Pending hides all main channels
- Pending allows DM interaction

**2.2 Attempt DM**
Send structured onboarding DM containing:
- Greeting
- Full server rules (clean embed/webhook)
- Explanation of Pending stage
- What happens next
- Button: "I agree to server rules"
- Button: "Start intro" (disabled until rules agreed)
- Placeholder disabled buttons for: Support ticket, Age verification

**2.3 Successful DM**
- Log success
- User stays in DM flow until they press rules button

### 3. Fallback Behaviour (DMs Off)

**3.1 Detect Failure**
- Catch "Missing Access / Cannot send message" error

**3.2 Send Fallback Message**
- Post in designated onboarding channel visible only to Pending + staff
- Include: notice about locked DMs, rules link, instructions to enable DMs, retry button
- Tone: "You've got your DMs locked down tighter than a nun's thigh..."

**3.3 No Dead Ends**
- Users must never become stuck

### 4. Rule Acknowledgement Button

**4.1 When User Clicks "I Agree to Server Rules"**
Record:
- User ID, timestamp, rule version
- Whether DM or fallback was used

**4.2 Confirmation**
- Send short confirmation message

**4.3 Unlock Start Intro**
- "Start intro" button becomes enabled
- No intro modal yet (Release 2)

### Out of Scope for Release 1
No intro modal, staff approvals, public posts, role changes beyond Pending, ticketing, age verification, or public interactions.

---

## RELEASE 1.5 - The Backbone Infrastructure
### XP Tracking System and Auto-Threading

**Goal:** Build the XP tracking foundation and auto-threading system that other releases depend on. This replaces Arcane and provides core engagement mechanics.

### Part A: XP Tracking System

**1.1 Database Schema**

**Users Table (XP columns):**
- `xp_total` - Total XP earned (integer)
- `xp_level` - Current level (calculated from total XP)
- `xp_last_message` - Timestamp of last XP-earning message (for cooldown)
- `xp_messages_sent` - Total message count
- `xp_vc_minutes` - Total voice chat minutes
- `xp_reactions_received` - Count of reactions received on messages

**XP History Table:**
- `history_id` - Primary key
- `user_id` - Discord user ID
- `xp_amount` - XP gained/lost
- `xp_source` - Source of XP (message, vc, reaction, bonus, penalty)
- `timestamp` - When XP was awarded
- `details` - JSON field for additional context

**1.2 XP Gain Mechanics**

**Messages:**
- Award XP for each message (configurable amount, default: 10-15 XP)
- Cooldown: 60 seconds between XP-earning messages per user
- Ignore messages in: ticket channels, staff channels, bot commands
- Configurable XP multipliers per channel (e.g., 2x XP in #events)

**Voice Chat:**
- Award XP per minute in voice (configurable, default: 5 XP/min)
- Track VC time in background task
- Don't award XP if user is muted/deafened (optional toggle)
- Configurable VC channels (exclude AFK, staff-only channels)

**Reactions:**
- Award XP when user's message receives reactions (configurable, default: 2 XP per unique reaction)
- Cap: max 20 XP per message from reactions
- Track unique reactors (same person reacting multiple times = 1 XP)

**Bonus/Penalty XP:**
- Staff can award bonus XP: `/xp-give @user [amount] [reason]`
- Staff can deduct XP: `/xp-take @user [amount] [reason]`
- Bonus for completing intro: +50 XP
- Bonus for age verification: +100 XP
- All manual adjustments logged in XP History

**1.3 Level Calculation**

Use a progressive XP curve (gets harder as you level up):
```
Level 1: 0 XP
Level 2: 100 XP
Level 3: 250 XP
Level 4: 450 XP
Level 5: 700 XP
...
Formula: XP_needed = 50 * (level^2) + 50 * level
```

Or use a simpler linear progression if preferred (configurable in code).

**1.4 Level Up Events**

When user levels up:
- Update user's `xp_level` in database
- Check if level is a milestone (15, 25, 50, 69, 100)
- If milestone: fire milestone event for other systems to hook into
- Log the level up event

**1.5 XP Commands**

**User Commands:**
- `/rank` - Show your own XP, level, progress to next level, server rank
- `/rank @user` - Show another user's stats
- `/leaderboard [page]` - Show top 10 users by XP

**Staff Commands:**
- `/xp-give @user [amount] [reason]` - Award bonus XP
- `/xp-take @user [amount] [reason]` - Deduct XP
- `/xp-set @user [amount]` - Set exact XP value (dangerous, confirmation required)
- `/xp-reset @user` - Reset user's XP to 0 (confirmation required)
- `/xp-import` - Import XP data from Arcane bot (one-time migration command)

**1.6 XP Import from Arcane**

**One-time migration command:**
- Staff run `/xp-import`
- Bot provides instructions for exporting Arcane data
- Staff upload CSV/JSON file with user IDs and levels
- Bot calculates equivalent XP totals
- Bot imports and updates all user records
- Bot posts summary of import (how many users, any errors)
- All imports logged

**1.7 Integration Points for Other Releases**

**Milestone DMs (Release 6):**
- When user hits level 15: trigger age verification eligibility message
- When user hits other milestones: trigger congratulatory DMs

**Age Verification Gate (Release 6):**
- Check user's level before allowing age verification button
- Block if level < 15

**Staff Context (Tickets):**
- Include user's level in ticket embeds
- Helps staff assess if user is established or brand new

### Part B: Auto-Threading System

**2.1 Thread Triggers**

Bot monitors configured channels for:
- Media attachments (images, videos, audio)
- Embedded links (YouTube, Twitter, etc.)
- Specific file types (configurable)
- Optional: messages over certain length

**2.2 Thread Configuration (per channel)**

Store in database or config:
```yaml
auto_threads:
  - channel_id: "123456789"
    enabled: true
    trigger_on:
      - media_files
      - youtube_links
    thread_name_format: "{username}'s {file_type}"
    archive_after: 60  # minutes
    auto_archive_duration: 1440  # Discord's auto-archive setting in minutes
    
  - channel_id: "987654321"
    enabled: true
    trigger_on:
      - all_messages  # thread every message
    thread_name_format: "{username} - {timestamp}"
    archive_after: 1440
```

**2.3 Thread Name Format Variables**

Support these placeholders in thread name formats:
- `{username}` - User's display name
- `{mention}` - User mention (@user)
- `{user_id}` - User ID
- `{timestamp}` - HH:MM format
- `{date}` - DD/MM/YYYY
- `{file_type}` - Type of file (image, video, link)
- `{message_content}` - First 50 chars of message (sanitised)
- `{level}` - User's XP level

**Examples:**
- `"{username}'s artwork"` → "Smokie's artwork"
- `"{username} ({level}) - {timestamp}"` → "Smokie (Level 23) - 14:30"
- `"Discussion: {message_content}"` → "Discussion: Check out this amazing..."

**2.4 Thread Creation Behaviour**

When trigger detected:
1. Create thread on the triggering message
2. Apply configured thread name format
3. Set auto-archive duration from config
4. Optional: Post initial bot message in thread:
   - "Thread created for {username}'s post. Feel free to discuss here!"
   - Tone: casual, welcoming, matches Gayborhood vibe
5. Log thread creation

**2.5 Thread Management Commands**

**Staff Commands:**
- `/thread-setup [channel]` - Interactive setup wizard for a channel's thread config
- `/thread-enable [channel]` - Enable auto-threading in a channel
- `/thread-disable [channel]` - Disable auto-threading in a channel
- `/thread-format [channel] [format]` - Update thread name format
- `/thread-list` - Show all channels with auto-threading enabled
- `/thread-test [channel]` - Test thread creation (doesn't actually create, just shows preview)

**2.6 Safety & Rate Limiting**

- Don't create threads on bot messages
- Don't create threads on messages in ticket channels
- Rate limit: max 10 threads per channel per minute (prevent spam/raids)
- If rate limit hit, log warning and notify staff channel
- Gracefully handle permission errors (log and notify staff)

**2.7 Integration with Other Features**

- Don't auto-thread in ticket channels (even if media posted)
- Don't auto-thread staff-only channels unless explicitly configured
- Thread names respect Gayborhood tone where appropriate

### Out of Scope for Release 1.5

This release does not include:
- XP leaderboard embeds (basic command only)
- XP role rewards (added later if needed)
- XP-based channel unlocks beyond level 15 age verification gate
- Advanced threading features (polls in threads, auto-pins, etc.)
- Thread moderation tools (separate from general mod tools)

---

## RELEASE 2 - Operation Just the Tip
### Full Onboarding Gate: Pending → Gaybor

**Goal:** Complete entry control. Every new user must acknowledge rules, submit intro, pass staff review, then receive Gaybor role and public welcome.

### 2.1 Roles

**Pending:**
- Assigned on join
- Cannot see main server
- Can DM bot

**Gaybor:**
- Assigned after staff approval
- Full channel access

**Regional Roles:**
- Auto-assigned based on location dropdown in intro
- Roles must exist or log warning

### 2.2 Intro Modal

**Triggered:** When user presses "Start intro"

**Required Fields:**
- Age (must be 18+)
- Preferred name
- Pronouns
- Location (dropdown)
- Bio (30-400 characters, links stripped)

**Bio Requirements:**
Must meaningfully describe: hobbies, pastimes, interests, fun facts.
No low-effort filler ("idk", "ask me"), no slurs, no invites, no links.

**Validation Rules:**
- Age < 18 → reject
- Bio under 30 characters → reject
- Bio with links/unsafe content → reject
- Pronouns required
- Location dropdown mandatory
- All fields complete

### 2.3 Automatic Regional Role Assignment

Assign based on location:
- North America, South America, Europe, UK & Ireland, Asia, Australia & Oceania, Africa, Middle East, Other
- If role missing/misconfigured, log error but continue

### 2.4 DM Confirmation

After successful submission:
"Your intro's been received. A staff member will review it shortly."

### 2.5 Staff Review Channel

Post embed with:
- Username + user ID
- Age, preferred name, pronouns, location, bio
- Timestamp
- Rules acknowledged
- Regional role assigned

**Staff Action Buttons:**
- Approve
- Reject
- Reject & Request Info
- Kick user
- Ban user

Reject, Kick, Ban require confirmation.

### 2.6 Approve Behaviour

**A. Roles:**
- Remove Pending
- Assign Gaybor
- Keep regional role

**B. DM User:**
Friendly confirmation of admission

**C. Public Welcome Message:**
Post in #your-new-gaybors:
- Witty, personalised welcome generated from bio
- Tone: human, gay, slightly cheeky, massive power bottom vibes

Examples:
- "Everyone say hi to [name] who already sounds like future chaos in the best way."
- "[Name] has entered the Gayborhood and honestly... we're not prepared."

**D. Post the Intro:**
Under welcome message:
- Preferred name
- Pronouns
- Location
- Full bio

**E. Log Everything:**
- Approving staff, timestamp, all intro fields, DM success/failure, roles assigned

### 2.7 Decline Behaviour

- User stays Pending
- Staff embed updates showing who declined
- Bot DMs polite refusal
- No public post
- Full event logged

### 2.8 Reject & Request Info

**Flow:**
1. Staff click "Reject & Request Info"
2. Modal for staff to type message
3. Bot DMs message to user
4. User can submit one corrected intro
5. Bot replaces old staff review card with new one
6. Staff proceed to approve/decline

### 2.9 Kick Behaviour

- Strip all roles
- Kick user
- DM them if possible with explanation
- Log action

### 2.10 Ban Behaviour

- Ban user
- Delete 24 hours message history (toggle configurable)
- DM them if possible
- Log action

### 2.11 Anti-Abuse Safeguards

- User can only submit intro once (or twice if staff request info)
- Block repeat submissions
- Troll detection: auto-reject slurs, spam, sexual content, nonsense
- DM failure triggers fallback
- All errors logged

### 2.12 Tone Requirements

Bot must communicate like:
- Tired gay mod
- Mildly sarcastic
- Concise
- Witty without trying too hard
- Never corporate
- Never robotic

### Out of Scope for Release 2
No ticketing, age verification, reminders, member close buttons, XP logic, panels.

---

## RELEASE 2.5 - Fun Community Features
### /bully Command and Music Link Conversion

### A. /bully Command

**Goal:** Lighthearted way for members to "bully" each other with ridiculous random insults.

**Command:** `/bully @user`

**Requirements:**
- Only works in designated channels (config: bully_enabled_channels)
- Cannot target Pending users
- Cannot target staff (optional toggle)
- Rate limited: 1 use per user per 60 seconds

**Insult Generation:**
- Pull from database of pre-written insults
- Mix and match sentence templates
- Tone: absurd, camp, over the top, never genuinely mean

**Examples:**
- "Babe, [user] dresses like they're allergic to mirrors"
- "[user]'s playlist is what plays in Hell's waiting room"
- "[user] has the energy of a deflated bouncy castle"

**Output Format:**
```
💅 [caller] has decided to read [target] for filth:
'[random insult]'
```

**Logging:**
- Log: who used it, who was targeted, timestamp
- Track for abuse: if someone targeted 10+ times in an hour, notify staff

**Safety:**
- No slurs, NSFW, or genuinely harmful content
- Staff can disable per-channel or entirely
- Staff can add/remove insults via slash command

**Staff Commands:**
- `/bully-add [insult text]` - Add new insult to pool
- `/bully-remove [insult_id]` - Remove insult
- `/bully-list` - View all active insults
- `/bully-toggle [channel]` - Enable/disable in channel

### B. Music Link Auto-Conversion

**Goal:** Auto-convert Spotify/streaming links to YouTube so everyone can watch/listen.

**Supported Platforms:**
- Spotify (tracks, albums, playlists)
- Apple Music
- SoundCloud
- Tidal
- Deezer

**Behaviour:**
1. User posts streaming link
2. Bot detects link
3. Extract: artist + track name
4. Search YouTube for best match
5. Reply: "🎵 Here's that on YouTube: [YouTube link]"

**Requirements:**
- Only works in music-friendly channels (config: music_conversion_channels)
- Rate limited: max 3 conversions per user per 10 minutes
- If no YouTube match found: "Couldn't find that on YouTube, babe"
- Optional: Include album art thumbnail
- Optional: Support playlists (post top 5 tracks with YT links)

**API Requirements:**
- Spotify API (metadata extraction)
- YouTube Data API (searching)
- Both require API keys in .env

**Logging:**
- Log all conversions
- Track success rate
- Alert staff if API quota low

**Safety:**
- Don't convert in tickets or staff channels
- Don't convert if original message deleted within 10 seconds

---

## RELEASE 3 - The Whiny-Fag-In-Distress Mechanism
### Member-Led Support Tickets (DM-Based Ticket Creation)

**Goal:** Members privately open support tickets through bot DMs after explaining what they need.

### 3.1 Ticket Trigger (DM Panel)

Button in bot's DM interface:
"I need help / Open support ticket"

**Rules:**
- Only visible to non-Pending users
- Works only in DMs
- Disabled in server channels

### 3.2 Ticket Modal Submission

Modal field: "What do you need help with?"

**Validation:**
- Min 20 characters, max 1000
- No links, invites, self-promo, Discord handles
- No slurs or NSFW
- No low-effort ("help", "idk", "someone was mean")

Invalid submissions rejected with short message:
"Use your words. Twenty characters minimum."

### 3.3 Ticket Creation (Server Side)

**A. New Channel:**
- Create in designated ticket category
- Only staff + ticket opener + bot can see

**B. Permissions:**
Private, isolated, clean

**C. Naming Format:**
Examples: `ticket-username-1234`, `support-0231`
Consistent, incrementing, logged

**D. Stored Metadata:**
- Ticket ID, owner, opener, reason text, timestamp
- Whether DM-led
- Number of attempts to submit valid reason

**E. First Message:**
Clean embed with:
- Member's submitted reason
- Username + ID
- Timestamp
- "Ticket opened by member via DM"

### 3.4 DM Confirmation to User

"Alright babe, ticket created. Someone will be with you soon. Don't panic, don't spam, and hydrate."

Include:
- Ticket ID/channel name
- Direct link to ticket
- What to expect
- Don't bump or spam reminder
- Rules link (sent only once here)

### 3.5 Anti-Abuse Controls

- Only one open ticket per user
- If user tries another: "You already have an open ticket. Use that one."
- DM failure → fallback in onboarding channel
- All submissions cleaned for links/garbage

### Out of Scope for Release 3
No staff-initiated tickets, claiming, closure, reminders, escalation, age verification, XP/role integration, adding members to tickets, staff DM triggers.

---

## RELEASE 4 - The Dominant Daddy Override
### Staff Claiming + Add To Ticket DM + First Response Nudges

**Goal:** Staff can claim tickets, members get notified when added, first response nudge if member goes quiet.

### 4.1 Claim Ticket Button

Staff-only button in every open ticket: "Claim Ticket"

**When clicked:**
- Store: staff who claimed, timestamp, ticket ID, ticket type
- Rename channel (optional): `ticket-username-claimed-by-smokie` or append `[claimed]`
- Post in ticket: "This ticket has been claimed by Smokie. They'll be with you shortly."
- Lock claiming: no one else can claim, button replaced with disabled "Claimed" indicator

### 4.2 Member Added to Ticket Behaviour

When staff add a member to existing ticket:

**Bot immediately DMs them:**
- Ticket link
- Why they're being added: "This ticket concerns an issue involving you..."
- Expectations: respond in ticket (not DMs), be civil, don't derail, staff will direct flow
- Rules reminder if appropriate
- "Keep it together babe" style message

**Bot posts in ticket:**
"[member] has been added to this ticket. Please keep communication in this channel only."

### 4.3 Staff Opened Tickets

Staff can open ticket for a member (from server panel introduced in Release 8, but logic defined here).

**When staff open ticket:**
- Bot creates ticket
- Adds member
- Posts internal message noting staff opened it
- Bot DMs member: "A staff-initiated ticket has been opened for you. Someone from the team will explain shortly."

Different messages depending on who triggers ticket.

### 4.4 Two Hour Nudge (Member Reminder)

If member doesn't respond within 2 hours of claim message:

**Bot sends ONE DM:**
"Hey babes, staff have responded in your ticket and are ready for you. Pop back in when you can."

**Rules:**
- Only for member-opened tickets
- Fires once
- Does not repeat
- Does not trigger for staff-opened tickets
- Does not trigger if member was added later

**Tracking:**
- Log whether nudge sent
- Store timestamp

### 4.5 Anti-Abandonment Safeguards

- If member left server, no nudge attempted
- If DMs closed, log failure
- If staff delete claim message, timers not restarted
- If staff unclaim (not allowed), bot blocks it

### Out of Scope for Release 4
No ticket closure, feedback, multi-stage nudges, staff reminders, auto-escalation, archiving, XP changes, age verification.

---

## RELEASE 5 - Naughty Bottom Notification Service
### Staff Opened Tickets + Mandatory 12 Hour Member Reminder

**Goal:** Staff can open tickets for members, bot delivers correct DMs, member gets timed reminder if they ignore ticket.

### 5.1 Staff Ticket Button (Server Panel)

Staff-only button in public support panel: "Staff Ticket"

**When clicked:**
- **A. Member Selector:** Choose user (must be in server, cannot be Pending)
- **B. Optional Reason Field:** Staff can enter short reason (not shown to member unless staff share it)
- **C. Ticket Creation:**
  - Opened by: staff member
  - Ticket owner: selected member
  - Type: staff-initiated
  - Reason: staff text (optional)
  - Channel permissions: member + staff + bot only
  - Metadata logged: opener, owner, timestamp, staff memo
- **D. First Message:** "This ticket has been opened by X for Y. A staff explanation will follow."

### 5.2 Immediate DM to Member

"A staff ticket has been opened for your attention. A member of the team will explain shortly. Please check the channel when you can."

Tone: calm, direct, serious, no fluff.

**If DMs fail:**
- Post fallback in ticket: "DMs could not be delivered. Please check this ticket as soon as possible."
- Log as DM failure

### 5.3 Twelve Hour Reminder (One Time Only)

If member doesn't speak in ticket within 12 hours of creation:

**Bot sends ONE reminder DM:**
"Hi, you still have an open staff ticket waiting for your response. Please reply in the channel as soon as possible."

**Rules:**
- Only sent once
- Only for staff-opened tickets
- Does not activate for member-opened tickets
- Does not activate if member leaves server
- Does not activate if DMs fail (log failure, don't retry)
- Reminder window resets only if ticket reopened in future

### 5.4 Anti-Avoidance Safeguards

- If staff delete first message, reminder timer continues (no restarts)
- If staff open by mistake, must close manually (Release 6)
- Member must post in ticket, not DM staff
- If member replies in DM: "Please respond in the ticket channel so staff can track everything."

### 5.5 Mute Button (NEW REQUIREMENT)

**Staff-only button in staff-opened tickets:**
"Mute User"

**When clicked:**
- Confirmation modal: "Are you sure you want to mute [username]?"
- If confirmed:
  - Apply server mute/timeout role
  - Duration: configurable (default 24 hours)
  - Post in ticket: "[username] has been muted by [staff] for not responding."
  - DM member if possible: "You've been muted for not responding to your staff ticket. Please check the ticket when your mute expires."
  - Log action

### 5.6 Logging

Log:
- Staff who created ticket
- Member it concerns
- Time created
- Initial DM success/failure
- Whether 12-hour reminder sent
- Any DM errors
- Member's first response timestamp
- Mute actions (who, when, duration)

### Out of Scope for Release 5
No ticket closure buttons, feedback, archiving, claiming (handled Release 4), escalation after 12 hours, multi-stage nudges, ticket merging, categorisation beyond origin.

---

## RELEASE 6 - Bareback Access Validation
### Age Verification (Member-Led Only)

**Goal:** Clean, controlled, fully member-led age verification starting in DMs. Includes XP-triggered milestone message at level 15.

### 6.1 Age Verify Trigger (DM Panel)

Button in bot's DM interface: "Age verify"

**Rules:**
- Only visible to Gaybors
- Hidden from Pending
- Only works in DMs
- Staff cannot trigger for a member

### 6.2 Pre-Screen Message

Outline:
- Must be level 15 or higher (self-confirmed, enforced by bot)
- Partnered server route: screenshot of roles showing valid 18+ badge
- Direct route: clear selfie required
- Staff may request second selfie or additional gestures
- Staff may escalate to official ID (only face + DOB visible)
- All images deleted by member + staff after verification
- All verification in ticket only
- Behaviour and privacy expectations

Tone: firm, clear, not cutesy.

### 6.3 Confirmation Button

"I confirm I am level 15 and want to age verify."

- Must click before ticket created
- Prevents accidental verifications
- Log confirmation timestamp

If not clicked: nothing happens.

### 6.4 Ticket Creation

Create: `ageverify-username-XXXX`

**Permissions:**
- Member + staff + bot only
- Type: Age Verify
- Always member-led
- Store: type, timestamps, member ID, partnered vs selfie route

Age verification cannot be staff-initiated.

### 6.5 First Ticket Message

**A. Selfie rules:**
- Clear, full face
- No filters
- Good lighting
- No AI images
- No NSFW
- Additional gesture requests may follow

**B. ID rules (only if escalated):**
- Face + DOB visible
- Everything else covered
- Must hold ID in photo
- No digital editing other than redaction

**C. Deletion rules:**
- All images deleted by staff + member post-verification

**D. Behaviour:**
- Wait for staff before sending images
- Remain polite
- No arguing about verification requirements
- No sending images in DMs

End with: "Staff will be with you as soon as they can."

### 6.6 DM Confirmation

"Your age verification ticket has been opened. Follow the guidance inside the ticket and wait for staff to request images before sending anything."

### 6.7 XP Integration Trigger

**When member reaches Level 15:**

Bot's built-in XP system (from Release 1.5) fires a level-up milestone event.

Bot automatically sends DM:

**Message includes:**
- Congratulations on reaching level 15
- Voice chat access now unlocked
- VC behaviour expectations
- Invitation to begin age verification for Downtown (NSFW) access
- Link/button to "Age verify" DM workflow
- Tone: warm, cheeky, camp, celebratory, but clear

Example:
"Congratulations babe, you finally hit level 15. Voice chats are now open for you. If you'd like access to Downtown, you can start the age verification whenever you're ready."

**Requirements:**
- Hook into the milestone event system from Release 1.5
- Must not fire twice (tracked in user's milestone history)
- Log who received milestone DM
- If DMs closed, post fallback in correct channel
- Award +100 XP bonus upon successful age verification completion

**Level 15 Gate Enforcement:**
- When user clicks "Age verify" button, bot checks their current level
- If level < 15: show error message "You must be level 15 to age verify. Keep participating and you'll get there!"
- If level >= 15: proceed with pre-screen message

### Out of Scope for Release 6
No ticket closure, staff closure, member closure, feedback, escalations, logging of staff actions, automated NSFW access (staff must manually assign role after verification).

---

## RELEASE 7 - The "Don't Move, I'll Get a Towel" Protocol
### Member Close Button, Staff Close Button, and Lifecycle Polish

**Goal:** Complete ticket lifecycle. Members get limited close option for accidental tickets; staff get full close control. Quality of life improvements.

### 7.1 Member Close Button

In every member-opened ticket: user-facing "Close Ticket" button.

**Rules:**
- Only works if:
  - Member themselves opened ticket
  - Within first 2 hours of creation
  - Ticket is "member-opened" type
- Age Verify, support tickets: yes
- Staff-opened tickets: always no

**If invalid:**
"This ticket can't be closed by you. A staff member will handle it."

**If valid:**
- Close ticket
- Log closure
- Post: "Ticket closed by the member."

### 7.2 Staff Close Button

Staff-only "Close Ticket" button in every ticket.

**When clicked:**
- Ticket closes immediately
- Archived or moved to archive category
- All reminder timers cancelled
- Log which staff closed
- Optional DM to owner: "Your ticket has been closed by staff. If you need anything else, feel free to open a new one."

**Always Available:**
- Regardless of: origin, age, status, failed member close, nudges fired

Staff have ultimate authority.

### 7.3 General Polish & System Behaviour

**A. Rules Link Only in First Ticket Confirmation DM**
- Sent once only in initial DM
- No repeats in future reminders

**B. All Timers Fire Once**
- 2-hour member nudge: once ever
- 12-hour staff ticket nudge: once ever
- No repeats, no loops, no spam
- Cancelled when member responds

**C. Member Replies Cancel Reminders**
- If user posts in ticket: all pending nudges cancelled
- Log updates accordingly

**D. Unified Tone & Formatting**
- Consistent formatting for embeds
- Consistent timestamp formatting
- Consistent role naming
- Consistent capitalisation
- Gayborhood tone throughout (dry, camp, unbothered, slightly bitchy)
- No corporate language

**E. Logging Requirements**
Log:
- Ticket creation
- Ticket type
- Who opened it
- Claim events
- Nudges (2-hour, 12-hour)
- Add to ticket events
- Ticket closure (member or staff)
- Timer cancellations
- DM failures
- Permission errors

All logs feed future analytics (Release 9).

### Out of Scope for Release 7
No feedback forms, auto-reopening, escalation levels, server-side analytics/dashboards, weekly/monthly stats, moderation scoring, multi-step close reasons, multi-ticket linking.

---

## RELEASE 8 - The GayOS Touch-To-Thrust Menu
### Ticket Panel in Server

**Goal:** Centralised, on-server control panel where members (and staff) can open tickets without DMing bot. First proper graphical interface.

### 8.1 Ticket Panel Message (Server-Based)

Post single, permanent panel in channel (e.g., #ticket-booth).

**Panel should be:**
- Custom embed or fully rendered graphic
- Lock-protected (no member replies)
- Auto-reposted if deleted
- Edited only by bot

**Panel includes:**

**A. Explanation of Ticket Types:**
- Support Ticket: For help with harassment, questions, reporting, assistance
- Age Verify: Unlock Downtown (NSFW). Only if level 15+
- Staff Ticket: Staff-only tool for reaching out to a member

**B. Behaviour Expectations:**
- Don't spam staff
- Don't open multiple tickets
- Be polite
- Don't escalate private matters to public channels
- Don't close ticket without reading staff replies
- Anything fake, rude, or chaotic will be handled accordingly

**C. Buttons:**
- Raise Support Ticket
- Age Verify
- Staff Ticket (staff only)

### 8.2 Button Behaviour (Mirrors DM Logic Exactly)

All buttons call same backend logic as DM flows for consistency.

**A. Support Ticket Button:**
- Opens modal: "What do you need help with?"
  - Min 20 characters, max 1000
- If too short, reject
- If valid, create ticket (same as Release 3)
- First message: user's reason
- DM confirmation sent (rules link included once, per Release 7)

**B. Age Verify Button:**
- If user below level 15 (bot checks): "You must be level 15 to age verify. Please use the DM panel once you reach the requirement."
- Otherwise: trigger same pre-screen + confirmation + ticket creation as Release 6
- Identical flow whether DM or #ticket-booth

**C. Staff Ticket Button (Staff Only):**
- Staff choose member ticket concerns
- Optional reason field
- Create ticket
- DM member: "A staff ticket has been opened for your attention."
- 12-hour nudge logic begins
- Metadata stored as Release 5
- Identical behaviour whether DM or panel

### 8.3 Additional Requirements

**Permanent Panel Protection:**
- If deleted (accidental/intentional), bot instantly reposts

**Role-Based Visibility:**
- Support Ticket: everyone
- Age Verify: Gaybors only, hidden from Pending
- Staff Ticket: staff only

**Accessibility:**
- Members can use either DM or panel
- Logic unified, not duplicated

### Out of Scope for Release 8
No categorisation beyond defined, multi-panel systems, alternative themes, dynamic personalised panels, multi-language support, analytics for button usage.

---

## RELEASE 9 - The TurboFag Expansion Pack
### Optional Future Enhancements

**Goal:** Define long-term optional upgrades once core system stable. Not guaranteed for launch but future-proofs the bot.

### 9.1 XP Triggered DMs (Extended Milestones)

Since the bot now has built-in XP tracking (Release 1.5), additional milestone features can be added.

**Extended Milestone Levels:**
- Level 25, 50, 69 (nice), 100, 150, 200+

**Messages can include:**
- Humorous congratulations tailored to the level
- Server statistics (how many people at this level, percentile rank)
- Tips, server highlights or reminders
- Unlocks (if any later attach XP to unlockables, e.g., custom roles)
- Encouragement to join voice chats or events
- Prompts to engage with community content
- Special rewards (temporary custom role, profile badge, recognition in #announcements)

**Example Level 69 Message:**
"Nice. You hit level 69, which is somehow both predictable and iconic for this server. You're in the top 5% of members by activity. Keep being fabulous."

**Example Level 100 Message:**
"Congratulations on level 100! You're officially part of the Gayborhood elite. The council has been notified of your achievement and may or may not be planning something."

**Bot must:**
- Log which users have received each milestone
- Avoid duplicate notifications
- Respect DM privacy
- Provide fallback if DMs off
- Track milestone history in database (separate from XP history)

**Optional Enhancements:**
- Staff can configure custom milestones and messages
- Milestones can trigger role assignments (e.g., "Level 50 Elite" role)
- Public announcements in #milestones channel (opt-in per user)
- XP multiplier weekends/events (2x XP for special occasions)

### 9.2 Post-Ticket Satisfaction Surveys

After ticket closes:
- DM member
- 1-5 rating or emoji-based "How did we do?"
- Optional free-text feedback
- Log results in staff-only feedback channel
- Tag staff who handled ticket (optional)
- Track satisfaction trends

**No survey for:**
- Age verify tickets
- Tickets closed due to rule-breaking
- Tickets closed without member interaction

### 9.3 Staff Performance Metrics

Optional analytics for moderation oversight:

**Metrics per staff member:**
- Tickets claimed
- Average response time
- Tickets closed
- Tickets escalated
- Satisfaction scores (if enabled)
- Unanswered nudges
- Hours between claim and resolution
- % staff ticket cases acknowledged within 12 hours

**Outputs:**
- Weekly/monthly summaries
- Leaderboards (internal only)
- Trend charts
- Discord admin channel exports

Helps identify gaps, burnout, improve consistency.

### 9.4 Duplicate Ticket Detection

Bot scans new ticket reasons for similarity to open tickets by same user.

**If detected:**
- "You already have a ticket open about this. Please use that one."
- "It looks like you've raised something similar. Want me to send you to the older ticket?"

**Options:**
- Cancel creation
- Merge tickets (future stretch)
- Redirect user
- Notify staff

**Similarity checks:**
- Keyword matching
- Fuzzy matching
- Titling patterns

### 9.5 Community Feedback Forms

Optional server-wide forms for:
- Suggestions
- Event ideas
- Rule improvement
- Queer cultural feedback
- Venting disguised as "feedback"
- Anonymous honesty box submissions

**Slash commands:**
`/suggest`, `/feedback`, `/idea`, `/confess` (optional, dangerous)

Submissions go to staff review channels.

**Forms can be:**
- Anonymous
- Semi-anonymous (user visible to bot, not staff)
- Public (posted to suggestions channel)

Results feed dashboards or monthly summaries.

### Out of Scope Unless Activated

These enhancements not built by default. Require:
- Explicit approval
- Separate development time
- Inclusion in future releases (10+)
- Confirmation core system stable

---

## Summary of Requirements

This bot needs to be:
1. **Modular** - Each release is a separate component
2. **Robust** - Extensive error handling and logging
3. **Maintainable** - Well-commented, clear structure
4. **Personality-driven** - Specific tone requirements throughout
5. **Database-backed** - All actions and states persisted
6. **Timer-resilient** - Can resume timers after restart
7. **Permission-aware** - Graceful degradation when permissions missing
8. **Rate-limited** - Respects Discord API limits

## What I Need From You (Claude Code)

1. Propose a project structure
2. Design the database schema
3. Create config template files
4. Identify technical challenges
5. Recommend build order for releases
6. Ask clarifying questions about anything unclear

Then we'll start building, beginning with Release 1 as the foundation.
