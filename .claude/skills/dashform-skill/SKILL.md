---
name: dashform
description: Create and manage AI-powered smart forms, surveys, and quizzes through the Dashform MCP server. Supports open-ended, single-choice, multiple-choice, and rating questions with AI-powered conversational experiences.
compatibility: Requires Dashform MCP server running locally at http://localhost:3000/api/mcp
metadata:
  author: Dashform
  version: "2.0.0"
  website: https://getaiform.com
---

# Dashform Form Creation Skill

Create intelligent forms through the Dashform MCP (Model Context Protocol) server. This skill enables you to build surveys, quizzes, and feedback forms using MCP tools.

## When to Use This Skill

Use this skill when the user needs to:
- Create surveys, feedback forms, or questionnaires
- Design quizzes or personality tests
- Build conversational AI-powered forms
- Collect structured data from respondents

## Prerequisites

**IMPORTANT**: This skill requires:

1. **Dashform MCP Server**: The local development server must be running at `http://localhost:3000`
2. **Organization ID**: You need the user's organization ID from their Dashform account
3. **User ID**: You need the user's ID from their Dashform account
4. **MCP Request Headers**: The MCP server requires proper Accept headers:
   - `Content-Type: application/json`
   - `Accept: application/json, text/event-stream`

### Getting Organization and User IDs

Ask the user to provide their organization ID and user ID. They can find these by:

1. Sign in to https://getaiform.com
2. Go to **Dashboard** → **Settings** → **Developer**
3. Copy the Organization ID and User ID

## Available MCP Tools

The Dashform MCP server provides the following tools:

### 1. `create_form` - Create a New Form

Creates a new form with full configuration support including questions, screens, theme, and branding.

**Basic Parameters:**
- `organizationId` (string, required): The organization ID to create the form in
- `userId` (string, required): The user ID creating the form
- `name` (string, required): The name of the form (1-255 characters)
- `type` (string, optional): Form type - `"structured"` (traditional, default) or `"dynamic"` (AI-powered conversational)
- `description` (string, optional): Description of the form
- `tone` (string, optional): Tone for the form (e.g., "friendly", "professional")

**Advanced Configuration Parameters:**
- `welcomeScreen` (object, optional): Welcome screen configuration (title, message, CTA)
- `endScreen` (object, optional): End screen configuration
- `endScreenEnabled` (boolean, optional): Whether to enable the end screen
- `endings` (array, optional): Multiple quiz endings for conditional selection
- `questions` (array, optional): List of questions for structured forms
- `snippets` (array, optional): Information snippets for AI to reference (dynamic forms)
- `maxFollowUpQuestions` (number, optional): Max follow-up questions (0-10, for dynamic forms)
- `theme` (object, optional): Visual theme (colors, fonts)
- `branding` (object, optional): Branding settings (logo, watermark)
- `backgrounds` (array, optional): Array of backgrounds (max 10)

**Returns:**
```json
{
  "success": true,
  "formId": "uuid",
  "name": "Form Name",
  "type": "structured",
  "description": "Form description",
  "tone": "friendly",
  "questionsCount": 5,
  "createdAt": "2024-01-29T...",
  "shareUrl": "https://getaiform.com/r/abc123",
  "editUrl": "https://getaiform.com/forms/uuid"
}
```

### 2. `create_reply` - Create a Form Response

Creates a new reply/response for an existing form.

**Parameters:**
- `formId` (string, required): The ID of the form to create a reply for
- `organizationId` (string, required): The organization ID the form belongs to
- `respondentName` (string, optional): Name of the person filling out the form
- `respondentEmail` (string, optional): Email of the person filling out the form
- `respondentEmotion` (string, optional): Initial emotional state - `"neutral"`, `"positive"`, or `"negative"`. Defaults to `"neutral"`
- `data` (object, optional): Initial form data as key-value pairs (question key → answer)

**Returns:**
```json
{
  "success": true,
  "replyId": "uuid",
  "formVersionId": "uuid",
  "status": "partial",
  "respondentName": "John Doe",
  "respondentEmail": "john@example.com",
  "respondentEmotion": "neutral",
  "data": {},
  "createdAt": "2024-01-29T..."
}
```

## How to Use This Skill

### Step 1: Get User Credentials

When the user asks to create a form, first ask for their credentials:

```
"To create a Dashform form, I need your:
1. Organization ID
2. User ID

You can find these at: https://getaiform.com/dashboard/settings/developer

Please provide your Organization ID and User ID:"
```

### Step 2: Create the Form

Use the `create_form` MCP tool with the provided credentials.

**CRITICAL RULES FOR FORM TYPE:**

1. **DEFAULT BEHAVIOR**: DO NOT specify the `type` parameter - let it default to `"structured"`
2. **ONLY use `type: "dynamic"`** when the user EXPLICITLY mentions:
   - "conversational form"
   - "AI-powered"
   - "dynamic"
   - "follow-up questions"
   - "adaptive questions"
3. **ALWAYS use `type: "structured"`** (or omit type) when:
   - User provides specific questions
   - User wants a traditional survey/form
   - User does NOT mention conversational/AI features
   - You include a `questions` array in the parameters

**IMPORTANT**: Always create complete, production-ready forms with:
- ✅ Welcome screen (title, message, CTA)
- ✅ Questions (at least 2-3 relevant questions)
- ✅ End screen (thank you message)
- ✅ Theme (colors and fonts)

**DO NOT create minimal forms with only name and description. Always include full configuration.**

**Reference Complete Examples:**

See the `examples/` directory for full form templates. Use these as references when creating forms:

1. **customer-survey.json** - Customer satisfaction survey (structured, 4 questions)
2. **quiz.json** - Personality quiz (dynamic, multiple endings)
3. **employee-satisfaction-survey.json** - Employee engagement (structured, 7 questions)
4. **event-registration.json** - Event registration (structured, 8 questions)
5. **nps-survey.json** - NPS survey (dynamic, AI follow-ups)

**Schema Reference:**

See `references/SCHEMA.md` for complete form structure documentation including:
- Question types (open-ended, single-choice, multiple-choice, rating)
- Welcome/end screens, quiz endings, themes, backgrounds, branching logic

**API Reference:**

See `references/API.md` for API endpoint documentation.

### Step 3: Inform the User

After creating the form, provide the user with:
- ✅ Success confirmation
- 📋 Share URL (for distributing to respondents)
- ✏️ Edit URL (for customizing the form)

## Form Types

| Type | Description | Use Case |
|------|-------------|----------|
| `structured` | Traditional fixed-question form (default) | Contact forms, registrations, surveys with fixed questions |
| `dynamic` | AI-powered conversational form | Complex surveys, interviews, personality quizzes |

**Recommendation**: Use `"structured"` for most cases with predefined questions. Use `"dynamic"` only when you need AI-powered conversational experiences with follow-up questions.

## Question Types Supported

| Type | Key | Description |
|------|-----|-------------|
| Open-ended | `open-ended` | Free text input |
| Single Choice | `single-choice` | Radio buttons (one answer) |
| Multiple Choice | `multiple-choice` | Checkboxes (multiple answers) |
| Rating | `rating` | 1-5 star rating |

## Tips for Best Results

1. **Always include complete configuration**: Don't create forms with just name and type
2. **Use appropriate form type**: Dynamic for conversational, structured for traditional
3. **Add welcome screens**: Set expectations and engage users
4. **Include end screens**: Thank users and provide next steps
5. **Apply themes**: Make forms visually appealing
6. **Reference examples**: Use the examples/ directory as templates

## Troubleshooting

### MCP Server Not Running

If you get connection errors:
1. Check that the server is running at `http://localhost:3000`
2. Verify the MCP endpoint is accessible at `/api/mcp`

### Invalid Credentials

If you get authentication errors:
1. Verify the Organization ID and User ID are correct
2. Check that the user has permission to create forms

### Form Creation Fails

If form creation fails:
1. Check that all required fields are provided
2. Verify the form configuration matches the schema
3. Review the error message for specific issues
