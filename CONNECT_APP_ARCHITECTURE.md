# Connect App — System Architecture & Product Blueprint

## 1) Product Vision
Connect App is a **multi-tenant community platform-of-platforms** where one user account can participate in many community spaces (schools, barangays, organizations, parent groups). Each space is configurable by its leader and can activate only the modules it needs.

---

## 2) Primary Personas & Roles

### Platform-level
- **Platform Super Admin**: manages global policies, abuse handling, billing/subscription (future), and tenant governance.

### Community-level (per space)
- **Space Owner / Community Leader**: creates and configures a space, invites staff, turns modules on/off.
- **Moderator / Staff**: manages day-to-day operations (content, classes, announcements, events, chats).
- **Member**: participates in feed/chat/classes/events and accesses files based on permissions.

---

## 3) Information Architecture (App Structure)

### App Navigation (MVP-first)
1. **Home**
   - Joined Communities
   - Suggested/Invited Communities
2. **Community Space**
   - Feed
   - Announcements
   - Chat
   - Education
   - Events
   - Files
   - Members
3. **Profile**
   - Personal information
   - Linked children
   - Joined spaces
   - Privacy and notification settings

### Community Space Setup Fields
- Space name
- Logo
- Banner
- Theme color
- Join code / invite link
- Module toggles:
  - Education
  - Announcements
  - Chat
  - Events
  - Files
  - Emergency Alerts

---

## 4) MVP Scope (Phase 1)
- Auth: login/signup (email/phone verification)
- Create community space
- Join via code
- Announcements (create, pin, categorize)
- Group chat (space-wide)
- Basic education posting (assignments + files + deadlines)
- Events (create + RSVP)
- User profiles

Out-of-scope for strict MVP but in architecture now:
- Parent-child linking with approvals
- Emergency SMS fallback
- Advanced moderation automation

---

## 5) Functional Blueprint by Module

### A. Education Module
- Classes and subject channels
- Assignment posting with due dates
- Upload files (PDF, Word, Video links/files)
- Student/member submissions
- Deadline tracking and reminders

### B. Communication & Interaction
- Announcements (pinning, categories)
- Space-wide chat
- Role-based chats (e.g., teachers-only, officers-only)
- Direct messaging
- Comments, reactions, threaded replies

### C. Social & Family
- User profiles
- Parent-child linking
- Friend/follow model
- Community feed (posts, photos, stories)

### D. Events & Activities
- Shared calendar
- Event creation + details + RSVP
- Event reminders via push notifications

### E. Emergency & Alerts
- One-tap emergency alert by authorized leaders
- Push notifications to members
- Optional SMS fallback for critical cases

### F. Files & Resources
- Central library per space
- Folder organization + tags
- Download/view tracking

---

## 6) Permission Model (RBAC + Scoped Policies)

Use **role-based access control** with two scopes:
1. **Platform scope** (Super Admin)
2. **Space scope** (Owner, Moderator, Member)

### Core rules
- A user may hold different roles in different spaces.
- Every record carries `space_id` (except platform-global entities).
- Access checks require both role and membership state.

### Permission Matrix (high-level)

| Capability | Super Admin | Space Owner | Moderator/Staff | Member |
|---|---|---|---|---|
| Create space | ✅ | ✅ | ❌ | ❌ |
| Configure space branding/modules | ✅ | ✅ | Limited | ❌ |
| Invite/promote moderators | ✅ | ✅ | ❌ | ❌ |
| Post announcements | ✅ | ✅ | ✅ | Limited (if enabled) |
| Pin announcements | ✅ | ✅ | ✅ | ❌ |
| Send emergency alerts | ✅ | ✅ | Optional | ❌ |
| Create events | ✅ | ✅ | ✅ | Optional |
| Moderate/remove content | ✅ | ✅ | ✅ | ❌ |
| Access private role chat | ✅ | ✅ | By role | ❌ |
| Join via code | ✅ | ✅ | ✅ | ✅ |

### Child safety permissions
- Child accounts require guardian approval for:
  - account activation (optional policy)
  - joining external spaces (policy-driven)
  - DM/chat scope (age policy)

---

## 7) User Flows (MVP + Core Extended)

### Flow 1: New user onboarding
1. User installs app and signs up via email or phone.
2. Verification (OTP/email link) completes account creation.
3. User lands on Home with options: Create Community or Join via Code.

### Flow 2: Create community space (Leader)
1. Tap “Create Space”.
2. Enter name, upload logo/banner, pick theme color.
3. Select modules (checkbox list).
4. System generates join code and invitation link.
5. Leader is assigned `space_owner` role.

### Flow 3: Join via code (Member)
1. User enters join code.
2. Backend validates code, membership policy, and role defaults.
3. User is added as `member` and sees community dashboard.

### Flow 4: Post announcement
1. Owner/Moderator opens Announcements.
2. Creates post, category, optional pin.
3. Notification service sends push to members.

### Flow 5: Education assignment lifecycle
1. Staff creates assignment in class/subject channel.
2. Members submit files or text before deadline.
3. Staff reviews submissions and marks status.

### Flow 6: Event + RSVP
1. Moderator creates event with date/time/location.
2. Members receive notification.
3. Members RSVP; reminders sent pre-event.

### Flow 7: Emergency alert
1. Authorized user taps “Emergency Alert”.
2. Alert fan-outs via push; optional SMS fallback for critical mode.
3. Alert appears pinned in community feed/chat.

---

## 8) High-Level Technical Architecture

## Recommended Stack
- **Mobile Frontend**: React Native (fast delivery, JS ecosystem) or Flutter (strong UI consistency)
- **Backend**: Node.js (NestJS/Express)
- **Primary DB**: PostgreSQL (multi-tenant relational data)
- **BaaS support option**: Supabase (Postgres + auth + storage + realtime)
- **Realtime messaging**: WebSockets (Socket.IO) or Supabase Realtime/Firebase RTDB patterns
- **File Storage**: S3-compatible object storage (Supabase Storage / AWS S3)
- **Notifications**: FCM/APNs + SMS provider (Twilio or regional gateway)

### Logical Components
1. **API Gateway / BFF**
   - Authenticated API entry
   - Rate limiting, request validation, audit tags
2. **Identity & Access Service**
   - Authentication
   - Email/phone verification
   - JWT/session issuing
   - Role/permission resolution
3. **Community Service**
   - Space CRUD
   - Module toggles
   - Membership management
4. **Content Service**
   - Feed posts, announcements, comments, reactions
5. **Chat Service**
   - Channels, DMs, role-based rooms, message history
6. **Education Service**
   - Classes, assignments, submissions, deadlines
7. **Events Service**
   - Calendar entries, RSVP, reminders
8. **File Service**
   - Upload, metadata, foldering, tracking
9. **Alert Service**
   - Emergency broadcasts, escalation policy, delivery logs
10. **Notification Service**
   - Push/email/SMS orchestration and retry queues
11. **Moderation Service**
   - Reports, blocklists, moderation queue, sanctions

### Infrastructure Pattern
- Containerized services (Docker/Kubernetes or managed serverless)
- Message queue for asynchronous tasks (BullMQ/SQS/PubSub)
- Redis for cache/presence/session acceleration
- CDN for media delivery
- Observability: logs + traces + metrics + audit logs

---

## 9) Data Model (Core Entities)

- `users`
- `spaces`
- `space_memberships` (user_id, space_id, role, status)
- `modules_enabled` (space_id + module flags)
- `announcements`
- `posts`, `comments`, `reactions`
- `chat_rooms`, `chat_messages`, `dm_threads`
- `classes`, `subjects`, `assignments`, `submissions`
- `events`, `event_rsvps`
- `files`, `folders`, `downloads`
- `alerts`, `alert_deliveries`
- `reports`, `blocks`, `moderation_actions`
- `parent_child_links`, `guardian_consents`

### Multi-tenant strategy
- Single database, shared schema, strict `space_id` scoping on space-level tables.
- Optional row-level security (especially if using Supabase/Postgres RLS).

---

## 10) Security & Trust Blueprint

- Email/phone verification at signup
- JWT with short-lived access tokens + refresh token rotation
- RBAC with server-side enforcement (never client-only)
- Data encryption in transit (TLS) and at rest
- Content moderation pipeline (report/block/triage)
- Abuse throttling (rate limits, spam detection)
- Parent-approved child accounts and configurable child safety rules
- Audit trails for privileged actions (role changes, alerts, removals)

---

## 11) Non-Functional Requirements

- **Scalability**: support many small-to-medium spaces with bursty events
- **Reliability**: at-least-once delivery for alerts/notifications with retries
- **Performance**: <300ms median API response for common reads
- **Availability target**: 99.9% for core APIs
- **Data retention**: configurable per community/legal policy
- **Backup/DR**: daily snapshots + point-in-time recovery for Postgres

---

## 12) Suggested Delivery Roadmap

### Phase 1 (MVP, 8–12 weeks)
- Auth + profile
- Space creation + join code
- Announcements + group chat
- Basic education posting
- Events + RSVP

### Phase 2
- Parent-child linking
- Advanced moderation and report workflows
- File library enhancements + analytics

### Phase 3
- Emergency SMS fallback and escalation
- Deeper social graph (follow/friends/stories)
- Integrations (LMS, gov systems, SSO)

---

## 13) API Surface (Illustrative)
- `POST /auth/signup`, `POST /auth/verify`, `POST /auth/login`
- `POST /spaces`, `GET /spaces/:id`, `POST /spaces/:id/join`
- `POST /spaces/:id/announcements`, `GET /spaces/:id/announcements`
- `POST /spaces/:id/chat/rooms`, `POST /chat/rooms/:roomId/messages`
- `POST /spaces/:id/classes/:classId/assignments`
- `POST /assignments/:id/submissions`
- `POST /spaces/:id/events`, `POST /events/:id/rsvp`
- `POST /spaces/:id/alerts/emergency`

---

## 14) Product Success Metrics
- Activation: % users joining or creating at least one space in first day
- Weekly active members per space
- Announcement read rate
- Chat response latency/engagement
- Assignment submission rate (education spaces)
- Event RSVP-to-attendance conversion
- Emergency alert delivery success rate
- Safety metrics: report resolution SLA and repeat offender rates

---

## 15) How to Test This Blueprint (Practical Validation Plan)

Since this is currently an architecture/product blueprint, testing means validating that implementation (or prototypes) satisfy the documented requirements.

### A. What to test first (MVP acceptance)
1. **Auth works**: user can sign up, verify email/phone, log in, and persist session.
2. **Space creation works**: leader can create a space with branding and module toggles.
3. **Join via code works**: second user can join the space with a valid code.
4. **Announcements work**: moderator can post and pin announcement; member can read.
5. **Group chat works**: users in same space can exchange real-time messages.
6. **Education post works**: staff can create assignment with deadline + attachment.
7. **Events work**: moderator creates event; member can RSVP.
8. **Profile works**: user can view/edit profile and see joined spaces.

### B. Manual QA checklist by role

#### Super Admin
- Can view/manage spaces and user moderation reports.
- Can enforce global policy actions.

#### Space Owner
- Can configure modules and branding.
- Can promote moderator.
- Can issue emergency alert (if module enabled).

#### Moderator/Staff
- Can create announcements, events, and education posts.
- Cannot change owner-only settings.

#### Member
- Can join via code and consume enabled modules.
- Cannot access restricted role chats or admin configuration pages.

### C. API-level smoke tests (example using cURL)
> Replace placeholders like `<TOKEN>` and `<SPACE_ID>`.

```bash
# 1) Login
curl -X POST http://localhost:3000/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"leader@example.com","password":"Passw0rd!"}'

# 2) Create space
curl -X POST http://localhost:3000/spaces \
  -H 'Authorization: Bearer <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"name":"Barangay 101","themeColor":"#2A7FFF","modules":{"announcements":true,"chat":true,"education":true,"events":true}}'

# 3) Join space by code
curl -X POST http://localhost:3000/spaces/<SPACE_ID>/join \
  -H 'Authorization: Bearer <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"joinCode":"ABC123"}'

# 4) Post announcement
curl -X POST http://localhost:3000/spaces/<SPACE_ID>/announcements \
  -H 'Authorization: Bearer <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"title":"Class Suspension","body":"No classes tomorrow","category":"School","pinned":true}'

# 5) Create event
curl -X POST http://localhost:3000/spaces/<SPACE_ID>/events \
  -H 'Authorization: Bearer <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"title":"Parent Meeting","startsAt":"2026-06-20T09:00:00Z"}'
```

### D. Security and permission tests
- Verify users cannot access records from spaces they are not members of (`space_id` isolation).
- Verify member role cannot perform moderator/owner actions.
- Verify blocked user cannot send DM/chat in the affected scope.
- Verify child account policies block restricted actions unless guardian consent exists.

### E. Non-functional validation
- Load test chat + announcements concurrently (e.g., k6) and confirm p95 latency targets.
- Verify notification retries and dead-letter handling for failed push/SMS deliveries.
- Verify audit logs are recorded for role changes, content removals, and emergency alerts.

### F. Definition of Done for MVP
MVP is considered test-passed when:
- All 8 MVP acceptance scenarios in section A pass.
- Role-based restrictions pass for Owner, Moderator, Member.
- No cross-space data leakage is observed.
- Critical flows (login, join, post announcement, chat, RSVP) meet performance baseline.
