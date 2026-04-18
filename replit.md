# WaveSync

A dark-themed Expo React Native watch party app for synchronized multi-platform video playback.

## Architecture

### Artifacts
- **API Server** (`artifacts/api-server/`) — Express.js + PostgreSQL + WebSocket backend
- **WaveSync Mobile** (`artifacts/wavesync/`) — Expo React Native app

### Tech Stack
- **Frontend**: Expo SDK, expo-router (file-based routing), React Query, Inter fonts
- **Backend**: Express.js, Drizzle ORM, PostgreSQL, `ws` (WebSocket)
- **Auth**: Custom token system (`wsync_{userId}:{timestamp}.{rand}`) — no JWT/Firebase
- **Build**: esbuild for backend bundling

## Key Features
1. **Auth** — Mock social sign-in (Google/Facebook/Twitter/Apple) + email login, creates real DB user
2. **Multi-Platform Watch Parties** — YouTube, Netflix, Prime Video, Twitch — open in native WebView, one-tap Watch Together
3. **Video Play Detection** — JS injected into native WebView detects video play events → auto-prompts Watch Together
4. **Platform-Aware UI** — FAB, modal, room screen, and room cards change color/branding per platform (YouTube red, Netflix red, Prime blue, Twitch purple)
5. **Real-time Sync** — WebSocket for play/pause/seek sync and room chat broadcast
6. **Chat** — Room chat (per-party) + direct messages between users
7. **Profile** — Customizable avatar color (10 color options), bio editing, display name
8. **Browsing History** — AsyncStorage-backed history with favicon grouping by date
9. **Invite System** — 6-char invite codes + native Share sheet / clipboard copy
10. **Friends System** — Send/accept/decline friend requests via email; FriendsContext manages state
11. **Live Presence** — Real-time tracking of who's online and what they're watching via PresenceContext + WebSocket `presence_update` broadcasts
12. **In-App Notifications** — NotificationBanner slides in for room invites and friend requests; accept/decline without leaving current screen
13. **Invite Friends Modal** — From inside any room, tap the user-plus icon to invite friends via WebSocket push notification; friends list shows online/offline/in-room status

## Design
- Dark theme: bg `#0A0A0F`, card `#12121A`, surface `#1A1A28`
- Accent: purple `#9333EA`, pink `#EC4899`
- Tab navigation: Home, Parties, Chat, Profile
- Liquid glass tabs on iOS (falls back to BlurView classic tabs)

## API Endpoints
- `POST /api/auth/login` — upsert user by email, return token
- `GET/POST /api/rooms` — list my rooms / create room
- `POST /api/rooms/join` — join by invite code
- `GET /api/rooms/:id` — room detail
- `GET /api/rooms/:id/members` — room member list
- `POST /api/rooms/:id/messages` — send room chat
- `GET /api/rooms/:id/messages` — fetch room chat history
- `POST /api/rooms/:id/leave` — leave room
- `GET/PUT /api/users/:userId/profile` — view/update profile
- `GET /api/dm/conversations` — DM conversation list
- `GET /api/dm/:userId` — DM thread with user
- `POST /api/dm/:userId` — send DM
- `GET /api/ws` — WebSocket connection (auth via token query param)

## WebSocket Events
- `join_room` / `leave_room` — presence tracking
- `sync_state` — broadcast play/pause/seek to room members
- `chat_message` — room chat broadcast
- `dm_message` — direct message delivery

## Database Schema
- `users` — id, email, displayName, photoUrl, bio, avatarColor, token, createdAt
- `rooms` — id, name, inviteCode, hostId, youtubeVideoId, youtubeVideoTitle, isPlaying, currentTime
- `room_members` — roomId, userId
- `messages` — id, content, senderId, roomId (nullable), recipientId (nullable), createdAt

## App Structure (wavesync)
```
app/
  _layout.tsx          — root layout (AuthProvider, WebSocketProvider, QueryClient)
  (auth)/
    index.tsx          — login/register screen
    _layout.tsx        — auth group layout
  (tabs)/
    _layout.tsx        — tab bar (liquid glass / classic blur)
    index.tsx          — Home screen (stats, quick actions, live rooms)
    parties.tsx        — Watch Parties (create/join modals, room list)
    chat.tsx           — DM conversation list
    profile.tsx        — User profile with avatar color picker
  room/[id].tsx        — Watch party room (player, chat, members)
  dm/[userId].tsx      — Direct message thread
components/
  Avatar.tsx           — Initials avatar with color
  MessageBubble.tsx    — Chat message bubble (own/other)
  RoomCard.tsx         — Room list card
  KeyboardAwareScrollViewCompat.tsx
context/
  AuthContext.tsx      — Auth state + token management (AsyncStorage)
  WebSocketContext.tsx — WebSocket real-time sync
constants/
  colors.ts            — Dark theme palette + AVATAR_COLORS
```

## Environment Variables
- `EXPO_PUBLIC_DOMAIN` — API server domain (set to `$REPLIT_DEV_DOMAIN` in workflow)
- `DATABASE_URL` — PostgreSQL connection string
