# WaveSync

## Overview

WaveSync is a real-time watch party application built with Expo (React Native) on the frontend and Express.js on the backend. Users can create or join rooms to watch videos together in sync, chat in real-time, and send direct messages. The app supports web, iOS, and Android platforms through Expo's cross-platform capabilities.

Key features:
- **Watch Parties**: Create/join rooms to watch YouTube videos together with synchronized playback
- **Real-time Chat**: In-room messaging and direct messaging between users
- **User Profiles**: Registration, login, customizable profiles with avatar colors
- **Room Management**: Public/private rooms with invite codes, categories, and participant limits
- **WebSocket Sync**: Real-time video state synchronization (play/pause/seek) across participants

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (Expo/React Native)
- **Framework**: Expo SDK 54 with expo-router for file-based routing
- **Navigation Structure**: Tab-based layout with 4 tabs (Home, Parties, Chat, Profile), plus modal screens for auth, room creation, and room joining
- **State Management**: TanStack React Query for server state, React Context for auth (`AuthProvider`) and WebSocket (`WSProvider`)
- **Styling**: React Native StyleSheet with a dark theme defined in `constants/colors.ts`
- **Fonts**: Inter font family (400, 500, 600, 700 weights) via `@expo-google-fonts/inter`
- **Key Libraries**: expo-linear-gradient, expo-haptics, expo-clipboard, react-native-webview (for video playback), react-native-gesture-handler, react-native-keyboard-controller

### Backend (Express.js)
- **Runtime**: Node.js with TypeScript (compiled via tsx for dev, esbuild for production)
- **API Layer**: RESTful Express routes defined in `server/routes.ts`
- **WebSocket**: Native `ws` WebSocket server for real-time room state sync and chat, attached to the same HTTP server
- **Authentication**: Token-based auth with Bearer tokens stored in memory map on server, persisted in AsyncStorage on client; bcrypt password hashing
- **CORS**: Dynamic origin allowlist based on Replit environment variables, plus localhost support for dev

### Data Storage
- **Database**: PostgreSQL (required, referenced via `DATABASE_URL` environment variable)
- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Schema Location**: `shared/schema.ts` — shared between frontend and backend
- **Tables**:
  - `users` — id (UUID), username, displayName, password (hashed), avatarColor, bio
  - `rooms` — id (UUID), name, description, hostId (FK→users), isPrivate, inviteCode, video state fields, category, participant limits
  - `messages` — id (UUID), roomId (FK→rooms), userId (FK→users), content, type
  - `directMessages` — referenced in storage imports (schema likely includes sender/receiver pattern)
- **Migrations**: Drizzle Kit with `drizzle-kit push` for schema sync (output in `./migrations`)
- **Validation**: drizzle-zod for generating Zod schemas from Drizzle table definitions

### Real-time Architecture
- WebSocket server manages `activeRooms` map with per-room state (participants, video URL, play state, current time)
- Clients connect via WSProvider context, which handles join/disconnect/message routing
- Room state broadcasts exclude the sender to avoid echo
- Video sync uses timestamp-based state tracking (`lastSyncTime`)

### Project Structure
```
app/                    # Expo Router file-based routing
  (auth)/               # Auth modal group (login, register)
  (tabs)/               # Main tab navigation (index, parties, chat, profile)
  room/[id].tsx         # Dynamic room screen
  create-room.tsx       # Room creation modal
  join-room.tsx         # Join room by invite code modal
server/                 # Express backend
  index.ts              # Server entry, CORS setup, static serving
  routes.ts             # API routes + WebSocket server
  storage.ts            # Database operations (CRUD layer)
  db.ts                 # Drizzle + pg pool setup
  templates/            # Landing page HTML
shared/                 # Shared between client & server
  schema.ts             # Drizzle schema + Zod validators
lib/                    # Frontend utilities
  auth-context.tsx      # Auth state provider
  ws-context.tsx        # WebSocket connection provider
  query-client.ts       # TanStack Query config + API helpers
constants/              # Theme colors
components/             # Reusable UI components
scripts/                # Build scripts for static export
```

### Build & Deployment
- **Dev**: Two processes — `expo:dev` for the Expo bundler and `server:dev` for the Express server
- **Production**: Static web build via custom `scripts/build.js`, server bundled with esbuild to `server_dist/`
- **Environment**: Relies on Replit environment variables (`REPLIT_DEV_DOMAIN`, `REPLIT_DOMAINS`, `DATABASE_URL`)

## External Dependencies

### Database
- **PostgreSQL**: Primary data store, required. Connection via `DATABASE_URL` environment variable. Used for user data, rooms, messages, and session storage.

### Key NPM Packages
- **drizzle-orm** + **drizzle-kit**: ORM and migration tooling for PostgreSQL
- **express** + **express-session**: HTTP server and session management
- **connect-pg-simple**: PostgreSQL session store for express-session
- **ws**: WebSocket server for real-time features
- **bcrypt**: Password hashing
- **@tanstack/react-query**: Server state management on the client
- **expo-router**: File-based routing for React Native
- **react-native-webview**: YouTube video playback in rooms
- **zod** + **drizzle-zod**: Schema validation

### Environment Variables Required
- `DATABASE_URL`: PostgreSQL connection string
- `EXPO_PUBLIC_DOMAIN`: Public domain for API requests from the client
- `SESSION_SECRET`: (optional, has fallback) Secret for session signing
- `REPLIT_DEV_DOMAIN`: Used for CORS and Expo dev server proxy
- `REPLIT_DOMAINS`: Used for CORS allowlisting in production