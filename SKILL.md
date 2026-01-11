---
name: smtp-dev
description: SMTP.dev API integration for temporary email management. List/create/delete email accounts, fetch emails and messages. Use for email testing, verification code retrieval, and automated email workflows. Triggers on "smtp", "temp email", "get email", "check inbox", "verification code".
allowed-tools: Bash, AskUserQuestion, Read, Write
---

# SMTP.dev Email Management Skill

Manage temporary email accounts via SMTP.dev API for testing and automation workflows.

## Quick Start

Ask me:
- "List all email accounts" → Show existing mailboxes
- "Create email test@domain.xyz" → Create new account
- "Check inbox for user@domain.xyz" → Fetch recent emails
- "Get verification code from user@domain.xyz" → Extract code from latest email
- "Delete account user@domain.xyz" → Remove email account

**Tag Management**:
- "Tag user@domain.xyz as project1" → Add tag to account
- "List accounts with tag project1" → Filter by tag
- "Remove tag project1 from user@domain.xyz" → Remove tag
- "Show all tags" → List all defined tags

## Configuration

### API Key Setup

The skill requires an API key from SMTP.dev. Set it via environment variable or pass directly:

```bash
# Environment variable (recommended)
export SMTP_DEV_API_KEY="smtplabs_your_key_here"

# Or pass in skill invocation
/smtp-dev --key smtplabs_xxx list
```

### Base URL

```
https://api.smtp.dev
```

## API Reference

### Authentication

All requests require `X-API-KEY` header:

```bash
curl -H "X-API-KEY: $SMTP_DEV_API_KEY" https://api.smtp.dev/accounts
```

### Endpoints

| Operation | Method | Endpoint | Description |
|-----------|--------|----------|-------------|
| List Accounts | GET | `/accounts` | Get all email accounts |
| Get Account | GET | `/accounts/{id}` | Get specific account |
| Create Account | POST | `/accounts` | Create new email account |
| Delete Account | DELETE | `/accounts/{id}` | Delete email account |
| List Messages | GET | `/accounts/{accountId}/mailboxes/{mailboxId}/messages` | Get emails in mailbox |
| Get Message | GET | `/accounts/{accountId}/mailboxes/{mailboxId}/messages/{id}` | Get full email content |

## Operations

### 1. LIST ACCOUNTS

List all email accounts with optional filtering.

**Parameters**:
- `address` (string) - Filter by email address
- `isActive` (boolean) - Filter by active status
- `page` (int) - Pagination

**Command**:
```bash
curl -s -X GET "https://api.smtp.dev/accounts" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY"
```

**Response**:
```json
{
  "member": [{
    "id": "67fcc8ed5737a4772603ceeb",
    "address": "user@domain.xyz",
    "quota": 0,
    "used": 709708,
    "isActive": true,
    "mailboxes": [
      {"id": "...", "path": "INBOX"},
      {"id": "...", "path": "Sent"},
      {"id": "...", "path": "Trash"}
    ],
    "createdAt": "2025-04-14T08:35:57+00:00"
  }],
  "totalItems": 495
}
```

### 2. CREATE ACCOUNT

Create a new email account.

**Command**:
```bash
curl -s -X POST "https://api.smtp.dev/accounts" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"address": "newuser@domain.xyz", "password": "SecurePass123"}'
```

**Response**:
```json
{
  "id": "new_account_id",
  "address": "newuser@domain.xyz",
  "isActive": true,
  "mailboxes": [...]
}
```

### 3. DELETE ACCOUNT

Delete an email account permanently.

**Warning**: This action cannot be undone!

**Command**:
```bash
curl -s -X DELETE "https://api.smtp.dev/accounts/{account_id}" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY"
```

### 4. GET MESSAGES

Fetch emails from a mailbox (INBOX by default).

**Parameters**:
- `page` (int, required) - Page number (max 30 per page)

**Command**:
```bash
# First get account ID and INBOX mailbox ID
ACCOUNT_ID="67fcc8ed5737a4772603ceeb"
MAILBOX_ID="67fcc8ed5737a4772603ceec"

curl -s -X GET "https://api.smtp.dev/accounts/${ACCOUNT_ID}/mailboxes/${MAILBOX_ID}/messages?page=1" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY"
```

**Response**:
```json
{
  "member": [{
    "id": "message_id",
    "from": {"address": "sender@example.com", "name": "Sender"},
    "to": [{"address": "user@domain.xyz"}],
    "subject": "Your verification code",
    "isRead": false,
    "hasAttachments": false,
    "size": 1234,
    "receivedAt": "2025-04-14T10:00:00+00:00"
  }],
  "totalItems": 10
}
```

### 5. GET MESSAGE CONTENT

Fetch full email content including body.

**Command**:
```bash
curl -s -X GET "https://api.smtp.dev/accounts/${ACCOUNT_ID}/mailboxes/${MAILBOX_ID}/messages/${MESSAGE_ID}" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY"
```

**Response**:
```json
{
  "id": "message_id",
  "from": {"address": "sender@example.com"},
  "subject": "Your verification code",
  "text": "Your code is: 123456",
  "html": "<p>Your code is: <strong>123456</strong></p>",
  "attachments": []
}
```

## Workflow Examples

### Get Verification Code

```bash
# 1. Find account by email address
EMAIL="user@domain.xyz"
ACCOUNT=$(curl -s "https://api.smtp.dev/accounts?address=${EMAIL}" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY" | jq -r '.member[0]')

ACCOUNT_ID=$(echo $ACCOUNT | jq -r '.id')
INBOX_ID=$(echo $ACCOUNT | jq -r '.mailboxes[] | select(.path=="INBOX") | .id')

# 2. Get latest message
MESSAGE_ID=$(curl -s "https://api.smtp.dev/accounts/${ACCOUNT_ID}/mailboxes/${INBOX_ID}/messages?page=1" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY" | jq -r '.member[0].id')

# 3. Get message content and extract code
curl -s "https://api.smtp.dev/accounts/${ACCOUNT_ID}/mailboxes/${INBOX_ID}/messages/${MESSAGE_ID}" \
  -H "X-API-KEY: $SMTP_DEV_API_KEY" | jq -r '.text' | grep -oE '[0-9]{4,6}'
```

### Batch Create Accounts

```bash
for i in {1..5}; do
  curl -s -X POST "https://api.smtp.dev/accounts" \
    -H "X-API-KEY: $SMTP_DEV_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{\"address\": \"test${i}@domain.xyz\", \"password\": \"Pass${i}123\"}"
done
```

## Rate Limits

- **General quota**: 2048 QPS (queries per second)
- **Exceeded**: Returns HTTP 429

## Error Handling

| Status | Meaning | Resolution |
|--------|---------|------------|
| 401 | Invalid API key | Check X-API-KEY header |
| 404 | Account/Message not found | Verify ID exists |
| 429 | Rate limit exceeded | Wait and retry |
| 500 | Server error | Retry later |

## Implementation Guide

### Entry Point

When user invokes `/smtp-dev`:

1. Parse command: `list`, `create`, `delete`, `inbox`, `message`
2. Check for API key (env var or --key flag)
3. Execute appropriate API call
4. Format and display results

### Command Patterns

```javascript
// Parse user input
const commands = {
  'list': () => listAccounts(),
  'create <email>': (email) => createAccount(email),
  'delete <email|id>': (target) => deleteAccount(target),
  'inbox <email>': (email) => getInbox(email),
  'message <email> [index]': (email, idx) => getMessage(email, idx || 0),
  'code <email>': (email) => extractVerificationCode(email)
};
```

### Helper Functions

```bash
# Get account by email address
get_account_by_email() {
  local email=$1
  curl -s "https://api.smtp.dev/accounts?address=${email}" \
    -H "X-API-KEY: $SMTP_DEV_API_KEY" | jq -r '.member[0]'
}

# Get INBOX mailbox ID
get_inbox_id() {
  local account_json=$1
  echo $account_json | jq -r '.mailboxes[] | select(.path=="INBOX") | .id'
}

# Extract verification code from text
extract_code() {
  local text=$1
  echo "$text" | grep -oE '[0-9]{4,8}' | head -1
}
```

## Tag Management

SMTP.dev API does not natively support tags. This skill implements local tag management via `tags.json`.

### Data Structure

**File**: `~/.claude/skills/smtp-dev/tags.json`

```json
{
  "version": "1.0",
  "tags": {
    "project1": {
      "description": "Project 1 accounts",
      "color": "blue",
      "createdAt": "2025-01-11T00:00:00Z"
    }
  },
  "accounts": {
    "67fcc8ed5737a4772603ceeb": {
      "address": "user@domain.xyz",
      "tags": ["project1", "testing"],
      "updatedAt": "2025-01-11T00:00:00Z"
    }
  }
}
```

### Tag Operations

#### 1. ADD TAG TO ACCOUNT

```bash
# Read current tags.json
TAGS_FILE="$HOME/.claude/skills/smtp-dev/tags.json"

# Add tag to account (using jq)
jq --arg id "ACCOUNT_ID" --arg tag "TAG_NAME" --arg addr "EMAIL" '
  .accounts[$id].tags = ((.accounts[$id].tags // []) + [$tag] | unique) |
  .accounts[$id].address = $addr |
  .accounts[$id].updatedAt = now | todate
' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
```

#### 2. REMOVE TAG FROM ACCOUNT

```bash
jq --arg id "ACCOUNT_ID" --arg tag "TAG_NAME" '
  .accounts[$id].tags = (.accounts[$id].tags | map(select(. != $tag)))
' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
```

#### 3. LIST ACCOUNTS BY TAG

```bash
jq --arg tag "TAG_NAME" '
  .accounts | to_entries | map(select(.value.tags | contains([$tag]))) |
  map({id: .key, address: .value.address, tags: .value.tags})
' "$TAGS_FILE"
```

#### 4. LIST ALL TAGS

```bash
jq '.tags | keys' "$TAGS_FILE"
```

#### 5. CREATE TAG

```bash
jq --arg tag "TAG_NAME" --arg desc "Description" '
  .tags[$tag] = {
    description: $desc,
    createdAt: (now | todate)
  }
' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
```

#### 6. DELETE TAG

```bash
# Remove tag definition and from all accounts
jq --arg tag "TAG_NAME" '
  del(.tags[$tag]) |
  .accounts = (.accounts | map_values(.tags = (.tags | map(select(. != $tag)))))
' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
```

### Sync with API

Sync local tags.json with remote accounts:

```bash
API_KEY="$SMTP_DEV_API_KEY"
TAGS_FILE="$HOME/.claude/skills/smtp-dev/tags.json"

# Get all accounts from API
accounts=$(curl -s "https://api.smtp.dev/accounts" -H "X-API-KEY: $API_KEY")

# Update tags.json with current accounts (preserve existing tags)
echo "$accounts" | jq -r '.member[] | "\(.id),\(.address)"' | while IFS=',' read -r id addr; do
  # Add account if not exists, preserve tags if exists
  jq --arg id "$id" --arg addr "$addr" '
    if .accounts[$id] then
      .accounts[$id].address = $addr
    else
      .accounts[$id] = {address: $addr, tags: [], updatedAt: (now | todate)}
    end
  ' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
done
```

### Workflow Examples

#### Tag accounts by domain

```bash
# Tag all accounts from specific domain
jq -r '.accounts | to_entries[] | select(.value.address | endswith("@740888.xyz")) | .key' "$TAGS_FILE" | \
while read -r id; do
  jq --arg id "$id" --arg tag "domain-740888" '
    .accounts[$id].tags = ((.accounts[$id].tags // []) + [$tag] | unique)
  ' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
done
```

#### Delete accounts by tag

```bash
# Get account IDs with specific tag
ids=$(jq -r --arg tag "to-delete" '.accounts | to_entries | map(select(.value.tags | contains([$tag]))) | .[].key' "$TAGS_FILE")

# Delete from API
for id in $ids; do
  curl -s -X DELETE "https://api.smtp.dev/accounts/$id" -H "X-API-KEY: $API_KEY"
done

# Remove from tags.json
jq --arg tag "to-delete" '
  .accounts = (.accounts | to_entries | map(select(.value.tags | contains([$tag]) | not)) | from_entries)
' "$TAGS_FILE" > /tmp/tags.json && mv /tmp/tags.json "$TAGS_FILE"
```

## Related Skills

- None currently

## Changelog

### v1.1.0
- Added local tag management system
- Multi-tag support per account
- Tag CRUD operations
- Sync with remote API
- Filter accounts by tag

### v1.0.0
- Initial release
- Support for account CRUD operations
- Email listing and content retrieval
- Verification code extraction helper
