# MachineLog - Replit Agent Guide

## Overview

MachineLog is a mobile-first time tracking and billing application for industrial machine operations. It allows operators to log machine usage with precise time intervals, calculate costs based on configurable rates, and generate receipts/invoices. The app tracks multiple machines, supports different interval types (active, idle, maintenance, breakdown), and handles complex billing scenarios including overtime rates, minimum billable time, tax calculations, and time rounding rules.

The project uses an Expo (React Native) frontend with file-based routing via expo-router, and an Express.js backend server. Data is currently stored client-side using AsyncStorage, with a PostgreSQL database configured via Drizzle ORM for server-side storage (though not yet fully utilized for app data).

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture
- **Framework**: Expo SDK 54 with React Native 0.81, targeting iOS, Android, and Web
- **Routing**: expo-router v6 with file-based routing. Tab navigation with 4 tabs (Dashboard, Jobs, Machines, Settings) plus modal/stack screens for job details, receipts, and creation forms
- **State Management**: React Context API (`lib/context.tsx`) provides a centralized `AppProvider` that manages all app state (machines, jobs, receipts, settings) and exposes CRUD operations
- **Data Persistence (Client)**: AsyncStorage (`lib/storage.ts`) stores all data locally on-device as JSON. Keys are prefixed with `@machinelog_`
- **Styling**: Dark theme throughout using a centralized color constants file (`constants/colors.ts`). Both light and dark themes are defined but currently identical (dark-only). Uses amber/gold accent color scheme
- **Fonts**: DM Sans (Google Fonts) loaded via `@expo-google-fonts/dm-sans`
- **Animations**: react-native-reanimated for UI animations (e.g., pulsing dot indicator for running timers)
- **Haptics**: expo-haptics for tactile feedback on user actions
- **Data Fetching**: TanStack React Query is set up with an API client (`lib/query-client.ts`) for server communication, though the app currently operates primarily offline with AsyncStorage

### Backend Architecture
- **Framework**: Express.js v5 running on Node.js
- **Server Entry**: `server/index.ts` sets up CORS, middleware, and serves static files in production
- **Routes**: `server/routes.ts` - currently minimal, structured for `/api` prefixed routes
- **Storage Layer**: `server/storage.ts` implements an `IStorage` interface with an in-memory implementation (`MemStorage`). Currently only has user CRUD operations
- **CORS**: Dynamic CORS handling that supports Replit dev/deployment domains and localhost for Expo web development

### Database
- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Schema**: `shared/schema.ts` defines a `users` table with id (UUID), username, and password. Uses `drizzle-zod` for schema validation
- **Migrations**: Output to `./migrations` directory
- **Connection**: Uses `DATABASE_URL` environment variable
- **Note**: The database schema is minimal (just users). The main app data (machines, jobs, receipts) is stored client-side in AsyncStorage. Future work may involve syncing this data to the server database

### Core Domain Model (`lib/types.ts`)
- **Machine**: Configurable billing entity with rates, overtime, minimum billable time, tax percentage, and machine specs
- **Job**: Links to a machine, contains multiple time intervals, tracks status (idle/running/paused/completed), and stores operator notes
- **TimeInterval**: Individual start/stop records with type classification (active/idle/maintenance/breakdown)
- **Receipt**: Generated from completed jobs with full cost breakdown including tax, duration, and integrity hash
- **AppSettings**: Global config for rounding rules, default tax, idle charging toggle, and currency symbol

### Key Utility Functions (`lib/utils.ts`)
- UUID generation via `expo-crypto`
- Duration formatting (HH:MM:SS and short format)
- Cost calculation with overtime, minimum billing, tax, and rounding rules
- Receipt hash generation for integrity verification

### Build & Deployment
- **Development**: Two processes - `expo:dev` for the Expo dev server and `server:dev` for the Express backend
- **Production Build**: Static web export via custom build script (`scripts/build.js`), server bundled with esbuild
- **Proxy**: In development, the Express server proxies requests to the Expo bundler via `http-proxy-middleware`

### Screen Structure
- `app/(tabs)/index.tsx` - Dashboard with live timers and active job overview
- `app/(tabs)/jobs.tsx` - Job list with filtering (all/running/paused/completed)
- `app/(tabs)/machines.tsx` - Machine management with delete protection
- `app/(tabs)/settings.tsx` - Global app configuration
- `app/job/[id].tsx` - Job detail with interval controls and live timer
- `app/receipt/[id].tsx` - Receipt view with share functionality
- `app/new-machine.tsx` - Machine creation form (presented as form sheet)
- `app/new-job.tsx` - Job creation with machine selection

## External Dependencies

### Core Runtime
- **Expo SDK 54**: Mobile app framework and build tooling
- **React Native 0.81**: Cross-platform UI framework
- **Express.js 5**: Backend HTTP server

### Database & ORM
- **PostgreSQL**: Database (via `pg` driver), connected through `DATABASE_URL` environment variable
- **Drizzle ORM**: TypeScript-first ORM for database queries and schema management
- **drizzle-zod**: Zod schema generation from Drizzle schemas

### Data & State
- **@react-native-async-storage/async-storage**: Client-side persistent key-value storage
- **@tanstack/react-query**: Server state management and data fetching cache

### UI & UX
- **expo-router**: File-based navigation
- **react-native-reanimated**: Declarative animations
- **react-native-gesture-handler**: Touch gesture handling
- **react-native-safe-area-context**: Safe area inset management
- **react-native-screens**: Native screen containers
- **expo-haptics**: Haptic feedback
- **expo-blur / expo-glass-effect**: Visual effects for tab bar
- **@expo/vector-icons / Ionicons**: Icon library
- **@expo-google-fonts/dm-sans**: Typography
- **expo-linear-gradient**: Gradient backgrounds
- **expo-image-picker**: Image selection capability
- **react-native-keyboard-controller**: Keyboard-aware scroll views

### Build & Dev Tools
- **esbuild**: Server-side bundling for production
- **tsx**: TypeScript execution for development server
- **http-proxy-middleware**: Dev server proxy to Expo bundler
- **patch-package**: Post-install patching for dependencies
- **drizzle-kit**: Database migration tooling

### Utilities
- **expo-crypto**: UUID generation and cryptographic operations
- **zod**: Runtime type validation
- **@stardazed/streams-text-encoding**: Text encoding polyfill
- **@ungap/structured-clone**: Structured clone polyfill