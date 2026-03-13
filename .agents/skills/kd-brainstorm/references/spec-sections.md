# Spec Design Sections — Conditional Templates

This file contains detailed templates for conditional spec sections.
Read only the sections relevant to the feature being designed.

## Table of Contents
- [API Contract](#api-contract)
- [Data Model & Migration](#data-model--migration)
- [UX/UI Flow](#uxui-flow)
- [Environment & Configuration](#environment--configuration)

---

## API Contract
**Include when**: new endpoints are added, existing endpoints change request/response/auth behavior, or external API integrations are introduced.

```markdown
## API Contract

### New / Changed Endpoints

#### `{METHOD} {/path}`
- **Purpose**: {what this endpoint does}
- **Auth**: {auth model — bearer token, API key, public, role-based}
- **Permissions**: {required roles/scopes}

**Request:**
| Parameter | Location | Type | Required | Description | Validation |
|-----------|----------|------|----------|-------------|------------|
| {param} | body/query/path/header | {type} | yes/no | {description} | {rules} |

**Response** (`{status_code}`):
```json
{
  "field": "value",
  "nested": { "field": "value" }
}
```

**Error Responses:**
| Code | Condition | Response Body |
|------|-----------|---------------|
| 400 | {when} | `{"error": "..."}` |
| 401 | {when} | `{"error": "..."}` |
| 404 | {when} | `{"error": "..."}` |
| 409 | {when} | `{"error": "..."}` |

**Behavior Notes:**
- Idempotency: {yes/no — how}
- Rate limiting: {rules}
- Pagination: {strategy — cursor/offset}
- Versioning: {strategy — URL/header}
- Caching: {strategy — ETags/Cache-Control}

**Example:**
```http
POST /api/v1/resource HTTP/1.1
Host: api.example.com
Authorization: Bearer {token}
Content-Type: application/json

{"field": "value"}
```

### Backward Compatibility
- Breaking changes: {yes/no — details}
- Deprecated fields: {list with removal timeline}
- Migration path for consumers: {steps}
```

---

## Data Model & Migration
**Include when**: database tables/models change, new tables are added, columns change, or indexes need updating.

```markdown
## Data Model & Migration

### Schema Changes

#### Table: `{table_name}`
| Column | Type | Nullable | Default | Index | Notes |
|--------|------|----------|---------|-------|-------|
| {column} | {type} | yes/no | {default} | {type or none} | {new/changed/dropped} |

#### New Table: `{table_name}`
| Column | Type | Nullable | Default | Index | Notes |
|--------|------|----------|---------|-------|-------|
| id | uuid | no | gen_random_uuid() | PK | |
| {column} | {type} | yes/no | {default} | {type} | |
| created_at | timestamptz | no | now() | | |
| updated_at | timestamptz | no | now() | | |

### Relationships
- `{table_a}.{column}` → `{table_b}.{column}` ({one-to-many/many-to-many})
- Cascade behavior: {on delete/update}

### Migration Plan
1. **Migration name**: `{YYYYMMDDHHMMSS}_{description}`
2. **Up**: {what the migration does}
3. **Down**: {how to revert — or "irreversible" with justification}
4. **Backfill**: {needed? strategy for existing data}
5. **Zero-downtime**: {yes/no — how to ensure no outage during migration}

### Compatibility During Partial Deploy
- Can old code run against new schema? {yes/no — why}
- Can new code run against old schema? {yes/no — why}
- Deploy order: {migrate first, then deploy code — or vice versa}

### Data Integrity
- Constraints: {unique, check, foreign key}
- Validation rules: {application-level vs DB-level}
- Seed data needed: {yes/no — what}
```

---

## UX/UI Flow
**Include when**: frontend screens/components change, new routes/pages are added, or user-facing workflows change.

```markdown
## UX/UI Flow

### Entry Points
- Route: `{/path}` — {description}
- Trigger: {button click, navigation, deep link, notification}
- Access: {who can see this — roles/permissions}

### Screen / Component Changes
| Screen/Component | Change Type | Description |
|------------------|-------------|-------------|
| {name} | new/modified/removed | {what changes} |

### User Flow
```
{Step 1} → {Step 2} → {Step 3} → {Outcome}
                ↓ (error)
            {Error handling}
```

### State Matrix
| State | UI Behavior | Data Source | Transition |
|-------|-------------|-------------|------------|
| Loading | {skeleton/spinner/placeholder} | {API call in progress} | → Success or Error |
| Empty | {empty state message + CTA} | {API returned empty} | → Loading (on action) |
| Success | {data rendered} | {API response} | → Loading (on refresh) |
| Error | {error message + retry} | {API error} | → Loading (on retry) |
| Partial | {partial data + warning} | {some items failed} | → Loading (on retry) |
| Unauthorized | {access denied message} | {403/401} | → Login or back |
| Offline | {cached data + banner} | {no network} | → Loading (on reconnect) |

### Validation Rules
| Field | Rule | Error Message |
|-------|------|---------------|
| {field} | {required/min/max/pattern} | {user-facing message} |

### Copy / Content
- {Heading text}
- {Button labels}
- {Error messages}
- {Success messages}
- {Placeholder text}

### Analytics Events
| Event | Trigger | Properties |
|-------|---------|------------|
| {event_name} | {user action} | {key: value pairs} |

### Accessibility
- Keyboard navigation: {tab order, shortcuts}
- Screen reader: {ARIA labels, announcements}
- Color contrast: {any concerns}

### Responsive Behavior
- Desktop: {layout}
- Tablet: {changes}
- Mobile: {changes}
```

---

## Environment & Configuration
**Include when**: new environment variables, feature flags, secrets, or infrastructure dependencies are introduced.

```markdown
## Environment & Configuration

### New Environment Variables
| Variable | Required | Default | Description | Secret? |
|----------|----------|---------|-------------|---------|
| {VAR_NAME} | yes/no | {default} | {what it controls} | yes/no |

### Feature Flags
| Flag | Default | Description | Rollout Plan |
|------|---------|-------------|--------------|
| {flag_name} | false | {what it enables} | {canary → 10% → 50% → 100%} |

### Environment Differences
| Setting | Local | Staging | Production |
|---------|-------|---------|------------|
| {setting} | {value} | {value} | {value} |

### Infrastructure Dependencies
- {New service/queue/cache needed}
- {Version requirements}
- {Scaling considerations}

### Secrets Management
- {Where secrets are stored — vault, env, config service}
- {Rotation policy}
- {Access control}
```
