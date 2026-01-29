# action-repo

This is the **action repository** that triggers GitHub webhooks on push, pull request, and merge actions. This repository is monitored by the [webhook-repo](https://github.com/YOUR_USERNAME/webhook-repo) which captures and stores these events.

## Purpose

This repository serves as the source for GitHub Actions that trigger webhooks. Any push, pull request, or merge operation on this repository will automatically send event data to the configured webhook endpoint.

## Webhook Configuration

### Events Monitored
- ✅ **Push** - Code pushed to any branch
- ✅ **Pull Request** - PR opened, closed, or synchronized
- ✅ **Merge** - Pull request merged into a branch

### Setup Instructions

#### 1. Configure Webhook in GitHub

1. Go to **Settings** → **Webhooks** → **Add webhook**
2. Configure the webhook:
   - **Payload URL**: `https://your-ngrok-url.ngrok.io/webhook` (or your deployed endpoint)
   - **Content type**: `application/json`
   - **Secret**: (optional, for security)
   - **SSL verification**: Enable SSL verification
   - **Events**: Select individual events
     - ✅ Pushes
     - ✅ Pull requests
   - **Active**: ✅ Checked

3. Click **Add webhook**

#### 2. Webhook Payload Format

When events occur, GitHub sends a JSON payload to the webhook endpoint:

**Push Event:**
```json
{
  "ref": "refs/heads/main",
  "pusher": {
    "name": "username"
  },
  "head_commit": {
    "timestamp": "2021-04-01T21:30:00Z"
  }
}
```

**Pull Request Event:**
```json
{
  "action": "opened",
  "pull_request": {
    "user": {
      "login": "username"
    },
    "head": {
      "ref": "feature-branch"
    },
    "base": {
      "ref": "main"
    },
    "created_at": "2021-04-01T09:00:00Z"
  }
}
```

**Merge Event (Pull Request Closed + Merged):**
```json
{
  "action": "closed",
  "pull_request": {
    "merged": true,
    "user": {
      "login": "username"
    },
    "head": {
      "ref": "feature-branch"
    },
    "base": {
      "ref": "main"
    },
    "merged_at": "2021-04-02T12:00:00Z"
  }
}
```

## Testing the Webhook

### Test Push Event
```bash
# Make a change and push to any branch
echo "Test change" >> test.txt
git add test.txt
git commit -m "Test push event"
git push origin main
```

### Test Pull Request Event
```bash
# Create a new branch
git checkout -b test-branch

# Make changes
echo "Feature changes" >> feature.txt
git add feature.txt
git commit -m "Add feature"
git push origin test-branch

# Create a pull request via GitHub UI
# Go to: https://github.com/YOUR_USERNAME/action-repo/compare/test-branch
```

### Test Merge Event
```bash
# Merge the pull request via GitHub UI
# This will trigger a "pull_request" event with action "closed" and merged=true
```

## Expected Webhook Data Flow

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────┐
│                 │         │                  │         │             │
│  action-repo    │ ──────► │  webhook-repo    │ ──────► │  MongoDB    │
│  (this repo)    │         │  (Flask API)     │         │             │
│                 │         │                  │         │             │
└─────────────────┘         └──────────────────┘         └─────────────┘
     GitHub                      Processes &                  Stores
     Actions                     Formats Data                 Events
```

## Repository Structure

```
action-repo/
├── README.md           # This file
├── test.txt           # Sample files for testing
├── feature.txt        # Sample files for testing
└── .gitignore         # Git ignore file
```

## Webhook Endpoint Repository

The webhook endpoint that receives and processes these events is located at:
- **Repository**: [webhook-repo](https://github.com/anushka16-gitt/webhook-repo.git)
- **Endpoint**: `/webhook` (POST)

## Event Format Expected by webhook-repo

The webhook-repo processes GitHub events and stores them in MongoDB with the following schema:

```json
{
  "request_id": "unique-uuid",
  "author": "username",
  "action": "PUSH | PULL_REQUEST | MERGE",
  "from_branch": "source-branch",
  "to_branch": "target-branch",
  "timestamp": "2021-04-01T21:30:00Z"
}
```

## Display Format in UI

The Streamlit UI displays events in the following formats:

**PUSH:**
```
🚀 Travis pushed to staging on 1st April 2021 - 9:30 PM IST
```

**PULL_REQUEST:**
```
🔀 Travis submitted a pull request from staging to master on 1st April 2021 - 9:00 AM IST
```

**MERGE:**
```
✅ Travis merged branch dev to master on 2nd April 2021 - 12:00 PM IST
```



- [GitHub Webhooks Documentation](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks)
- [webhook-repo Repository](https://github.com/YOUR_USERNAME/webhook-repo)
- [Testing Webhooks Guide](https://docs.github.com/en/developers/webhooks-and-events/webhooks/testing-webhooks)
