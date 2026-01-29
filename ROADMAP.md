# TODO App Roadmap

A minimalistic, collaborative TODO application with real-time sync and offline support.

## Tech Stack

- **Frontend:** Next.js (React)
- **Backend:** Vercel Serverless Functions
- **Database:** Neon (PostgreSQL)
- **Real-time:** Server-Sent Events (SSE)

---

## Phase 1: Core Infrastructure

### 1.1 Database Schema Design

Design and implement the PostgreSQL schema in Neon:

- **accounts** — User accounts (nullable for anonymous users)
  - Email, hashed password, OAuth provider info
  - Anonymous token reference
  - List ordering (array of list IDs)
  - Theme preference
  - RSS feed secret token
- **anonymous_tokens** — Tokens for unauthenticated users
  - Token value (UUID)
  - Created timestamp
  - Associated account ID (nullable, populated on auth)
- **lists** — TODO lists
  - Title
  - Color (from preset palette)
  - Owner account ID
  - Item ordering (array of item IDs)
  - RSS feed secret token
  - Deleted at timestamp (for trash)
- **todo_items** — Individual TODO entries
  - Title
  - Body (Markdown, nullable)
  - Due date/time (nullable)
  - Completed date/time (nullable)
  - List ID
  - Deleted at timestamp (for trash)
- **list_shares** — Sharing permissions
  - List ID
  - Account ID
  - Role (view, edit, admin)
  - Invited by account ID

### 1.2 API Foundation

Implement RESTful API endpoints on Vercel:

- CRUD operations for lists and TODO items
- Ordering/reordering endpoints
- Soft delete with trash support
- Input validation and error handling
- Anonymous token generation and validation

### 1.3 Authentication System

Implement multi-provider authentication:

- Anonymous token generation and local storage persistence
- Email/password registration and login
- GitHub OAuth integration
- Google OAuth integration
- Token-to-account migration (transfer anonymous lists to authenticated user)
- Session management with secure cookies/JWT

---

## Phase 2: Core Features

### 2.1 List Management

- Create, read, update, delete lists
- Preset color palette selection (8-12 colors that work with light/dark themes)
- Reorder lists within an account
- View trash (lists deleted within 7 days)
- Restore lists from trash

### 2.2 TODO Item Management

- Create, read, update, delete TODO items
- Title editing (inline)
- Markdown body editing with dedicated editor
- Due date/time picker (optional field)
- Mark as complete/incomplete with timestamp
- Reorder items within a list
- View trash (items deleted within 7 days)
- Restore items from trash

### 2.3 Trash System

- Soft delete for lists and TODO items
- 7-day retention period
- Scheduled cleanup job (Vercel Cron)
- Manual restore functionality
- Permanent delete option

---

## Phase 3: User Interface

### 3.1 Layout and Navigation

- Minimalistic design language
- Responsive layout (mobile, tablet, desktop)
- Sidebar for list navigation
- Main content area for TODO items
- Global header with search, theme toggle, and account menu

### 3.2 List View

- Display list title and color indicator
- Show TODO items (title only)
- Completed items display checkmark and animate to bottom of list
- Quick-complete checkbox/button for each item
- Drag handles for reordering
- Up/down arrow buttons for reordering (accessibility)
- Link to list RSS feed

### 3.3 TODO Detail Modal

- Trigger on TODO title click
- Display all fields: title, body (rendered Markdown), due date, completion status
- Edit mode for all fields
- Markdown editor component for body
- Delete button (moves to trash)
- Close/save actions

### 3.4 Markdown Editor

- Integrate a Markdown editor library (e.g., react-markdown, MDXEditor, or similar)
- Toolbar with common formatting options
- Live preview or split view
- Support for standard Markdown syntax

### 3.5 Drag and Drop

- Implement drag-and-drop for TODO items within lists
- Implement drag-and-drop for lists in sidebar
- Visual feedback during drag operations
- Touch support for mobile devices
- Persist new order to backend on drop

### 3.6 Theme System

- Light theme (default based on system preference)
- Dark theme
- Theme toggle in UI
- Persist preference to account settings
- CSS variables for theme colors
- Ensure preset list colors work with both themes

### 3.7 Global Live Search

- Search input in header
- Search as you type (debounced)
- Search across all lists and TODO items (titles and bodies)
- Display results grouped by list
- Click result to navigate to item/list
- Keyboard navigation support

---

## Phase 4: Real-time and Notifications

### 4.1 Server-Sent Events (SSE)

- SSE endpoint for authenticated connections
- Event types: item created, item updated, item deleted, list created, list updated, list deleted, list shared
- Client-side event handling and state updates
- Reconnection logic with exponential backoff
- Support for anonymous token-based connections

### 4.2 Push Notifications

- Service worker registration
- Push notification permission request flow
- Due date reminder scheduling
  - Configurable reminder time (e.g., 1 hour before, 1 day before)
  - Backend scheduled job to trigger notifications
- Display modal when notification received while app is open
- Native notification when app is in background

### 4.3 Multi-Client Sync

- New items from other clients appear automatically via SSE
- Updates reflect in real-time across all connected clients
- Deleted items removed from view across clients
- List reordering synced across clients

---

## Phase 5: Sharing and Collaboration

### 5.1 Share Management

- Share list dialog/modal
- Invite users by email
- Role selection: view-only, edit, admin
- View current collaborators and their roles
- Owner can modify roles or revoke access
- Admin can invite others and modify non-admin roles
- Remove self from shared list

### 5.2 Permission Enforcement

- API-level permission checks for all operations
- View: read-only access to list and items
- Edit: create, update, delete items; reorder items
- Admin: all edit permissions plus invite/manage collaborators
- Owner: all admin permissions plus delete list, transfer ownership

### 5.3 Shared List UX

- Visual indicator for shared lists
- Show collaborator avatars/initials on shared lists
- Real-time updates from collaborators via SSE (no notifications)

---

## Phase 6: RSS Feeds

### 6.1 Feed Generation

- RSS 2.0 or Atom feed format
- Per-account feed: all TODO items across all owned/accessible lists
- Per-list feed: all TODO items in a specific list
- Include: title, body (as description), due date, completion status
- Secret token in URL for access control (e.g., `/rss/account/{secret}`, `/rss/list/{secret}`)

### 6.2 Feed Management

- Generate/regenerate feed secret tokens
- Display feed URLs in account settings
- Display list feed URL in list settings/menu
- Copy-to-clipboard functionality

### 6.3 Feed Security

- Validate secret token on every request
- Rate limiting
- No authentication required (token is the auth)

---

## Phase 7: Progressive Web App (PWA)

### 7.1 PWA Foundation

- Web app manifest
- Service worker for caching
- Install prompt handling
- Offline-capable asset caching
- App icons and splash screens

### 7.2 Offline Data Storage

- IndexedDB for local data persistence
- Cache lists and TODO items for offline access
- Queue mutations (create, update, delete, reorder) when offline
- Track sync status for each queued operation

### 7.3 Online/Offline Sync

- Detect online/offline status
- Sync queued mutations when connection restored
- Conflict detection (item modified both locally and remotely)
- Conflict resolution modal: display both versions, let user choose
- Field-level diff display for informed decision-making

### 7.4 Offline UX

- Visual indicator of offline status
- Show pending sync count
- Disable features that require connectivity (sharing, search across server data)
- Optimistic UI updates with sync status indicators

---

## Phase 8: Polish and Optimization

### 8.1 Performance

- Optimize database queries with proper indexing
- Implement pagination for large lists
- Lazy load TODO item bodies
- Image optimization for any assets
- Bundle size optimization

### 8.2 Accessibility

- ARIA labels and roles
- Keyboard navigation throughout
- Screen reader testing
- Focus management in modals
- Color contrast compliance (WCAG AA)
- Up/down buttons as alternative to drag-and-drop

### 8.3 Error Handling

- User-friendly error messages
- Retry logic for failed API calls
- Graceful degradation when features unavailable
- Error boundary components

### 8.4 Testing

- Unit tests for utility functions and hooks
- Integration tests for API endpoints
- End-to-end tests for critical user flows
- Accessibility testing automation

---

## Data Model Summary

```
Account
├── id
├── email (nullable for anonymous)
├── password_hash (nullable)
├── oauth_provider (nullable)
├── oauth_id (nullable)
├── anonymous_token_id (nullable)
├── list_order: [list_ids]
├── theme: "light" | "dark" | "system"
├── rss_secret
├── created_at
└── updated_at

AnonymousToken
├── id
├── token (UUID)
├── account_id (nullable)
├── created_at
└── converted_at (nullable)

List
├── id
├── title
├── color (preset key)
├── owner_id (account)
├── item_order: [item_ids]
├── rss_secret
├── deleted_at (nullable)
├── created_at
└── updated_at

TodoItem
├── id
├── list_id
├── title
├── body (Markdown, nullable)
├── due_at (nullable)
├── completed_at (nullable)
├── deleted_at (nullable)
├── created_at
└── updated_at

ListShare
├── id
├── list_id
├── account_id
├── role: "view" | "edit" | "admin"
├── invited_by_id (account)
├── created_at
└── updated_at
```

---

## Milestones

| Milestone | Phases | Description |
|-----------|--------|-------------|
| **M1: Foundation** | 1.1–1.3 | Database, API, and authentication working |
| **M2: Functional MVP** | 2.1–2.3 | Core CRUD operations for lists and items |
| **M3: Usable App** | 3.1–3.7 | Complete UI with all interactions |
| **M4: Real-time** | 4.1–4.3 | Live sync and push notifications |
| **M5: Collaboration** | 5.1–5.3 | Sharing and permissions |
| **M6: RSS** | 6.1–6.3 | Feed generation and access |
| **M7: Offline** | 7.1–7.4 | PWA with offline support |
| **M8: Production Ready** | 8.1–8.4 | Performance, accessibility, testing |

---

## Non-Functional Requirements

- **No monetization:** Free app, no ads, no premium tiers
- **Minimalistic design:** Clean, distraction-free interface
- **Privacy-focused:** Anonymous usage supported, minimal data collection
- **Open source friendly:** Structure code for potential open-sourcing
