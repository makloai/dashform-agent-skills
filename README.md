# Dashform Skill for Claude Code

Create AI-powered forms and surveys using natural language in Claude Code.

## What is this?

Talk to Claude to create intelligent forms instantly. No coding required.

**Capabilities:**
- 🤖 **Dynamic forms** - AI-driven conversational experiences
- 📋 **Structured forms** - Traditional surveys and questionnaires
- ✨ **Natural language** - Just describe what you want
- 🔗 **Instant sharing** - Get shareable and edit links immediately
- 🔐 **Credential caching** - Authenticate once, create many forms
- 📝 **Complete forms** - Welcome screens, questions, themes, and end screens

## Quick Start

### Prerequisites

Before using this skill, you need:
1. Dashform account at [getaiform.com](https://getaiform.com)
2. Your session token from browser cookies (one-time setup)
3. Local Dashform server running at `https://getaiform.com`

### Setup (2 steps)

**1. Start the Dashform server**

```bash
cd /Users/apple/Workspace/maklo
pnpm dev
```

Server runs at `https://getaiform.com`

**2. Configure Claude Code**

Edit `~/.config/claude-code/mcp.json`:

```json
{
  "mcpServers": {
    "dashform": {
      "url": "https://getaiform.com/api/mcp",
      "transport": "http"
    }
  }
}
```

**That's it!** Claude will guide you through authentication on first use.

## Usage

Just ask Claude to create a form:
- "Create a customer satisfaction survey"
- "Make an event registration form"
- "Build an NPS survey with follow-up questions"

**First time:** Claude will ask for your session token (from browser cookies at `better-auth.session_token`)
**After that:** Credentials are cached, forms are created instantly

## Skill Workflow

When you ask Claude to create a form:

1. **Check credentials** - Reads cached credentials from `dashform/credentials.json`
2. **Authenticate (first time only)** - Asks for session token, runs `setup-credentials.sh`
3. **Read documentation** - Loads `references/SCHEMA.md` and `references/API.md`
4. **Generate config** - Creates complete form JSON with welcome screen, questions, theme, end screen
5. **Save config** - Saves to `dashform/data/{form-name}_{timestamp}.json`
6. **Create form** - Runs `create-form.sh` script → calls MCP `create_form` tool
7. **Return URLs** - Provides share URL (for respondents) and edit URL (for customization)

## Troubleshooting

### MCP Server Not Found
Check server is running at `https://getaiform.com` and verify `mcp.json` configuration.

### Authentication Issues
Delete cached credentials and re-authenticate with a fresh session token:
```bash
rm .claude/skills/dashform/credentials.json
```

### Form Creation Fails
Verify form JSON matches schema in `references/SCHEMA.md` and check server logs.

## Advanced

### Skill Components

**Scripts:**
- `scripts/setup-credentials.sh` - Authenticates and caches credentials
- `scripts/create-form.sh` - Creates forms via MCP

**Data:**
- `dashform/credentials.json` - Cached userId and organizationId
- `dashform/data/` - Generated form configurations

**Documentation:**
- `references/SCHEMA.md` - Form structure and question types
- `references/API.md` - MCP API documentation

**Examples:**
- `examples/customer-survey.json`
- `examples/quiz.json`
- `examples/employee-satisfaction-survey.json`
- `examples/event-registration.json`
- `examples/nps-survey.json`

### Form Configuration Structure

**Required:**
- `name` - Form title

**Recommended:**
- `type` - "structured" (default) or "dynamic"
- `welcomeScreen` - Title, message, CTA button
- `questions` - Array of question objects
- `endScreen` - Thank you message
- `theme` - Colors and fonts

**Example:**
```json
{
  "name": "Customer Survey",
  "type": "structured",
  "welcomeScreen": {
    "title": "We'd love your feedback!",
    "message": "Help us improve",
    "ctaLabel": "Start Survey"
  },
  "questions": [
    {
      "key": "satisfaction",
      "type": "rating",
      "label": "How satisfied are you?",
      "required": true
    }
  ],
  "endScreen": {
    "title": "Thank you!",
    "message": "We appreciate your feedback"
  }
}
```

Note: `userId` and `organizationId` are added automatically by the script.

### Available MCP Tools

**`create_form`**
- `organizationId`, `userId`, `name` (required)
- `type`, `description`, `tone`, `welcomeScreen`, `questions`, `endScreen`, `theme`, `branding` (optional)
- Returns: `formId`, `shareUrl`, `editUrl`

**`create_reply`**
- `formId`, `organizationId` (required)
- `respondentName`, `respondentEmail`, `respondentEmotion`, `data` (optional)
- Returns: `replyId`, `status`

### Question Types

| Type | Key | Description |
|------|-----|-------------|
| Open-ended | `open-ended` | Free text |
| Single Choice | `single-choice` | Radio buttons |
| Multiple Choice | `multiple-choice` | Checkboxes |
| Rating | `rating` | 1-5 stars |

### Form Types

| Type | Use Case |
|------|----------|
| `structured` | Traditional surveys, registrations |
| `dynamic` | AI-powered conversational forms |

---

**Need help?** Check [troubleshooting](#troubleshooting) or open an issue.
