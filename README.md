# Dashform Skill for Claude Code

Create and manage AI-powered forms through natural conversation with Claude.

## 📖 What is this?

The Dashform Skill lets you create intelligent forms and surveys directly in Claude Code using natural language. Just tell Claude what kind of form you want, and it will create it for you.

**What you can do:**
- Create dynamic forms (AI-driven conversational forms)
- Create structured forms (traditional surveys)
- Customize form name, description, and tone
- Get shareable links and edit links instantly

## 🚀 Quick Start

### Step 1: Start the Dashform Server

Run in your terminal:

```bash
cd /Users/apple/Workspace/maklo
pnpm dev
```

The server will start at `http://localhost:3000`.

### Step 2: Configure Claude Code

Add the Dashform MCP server to your Claude Code configuration:

**Config file location**: `~/.config/claude-code/mcp.json`

```json
{
  "mcpServers": {
    "dashform": {
      "url": "http://localhost:3000/api/mcp",
      "transport": "http"
    }
  }
}
```

### Step 3: Get Your Credentials

Visit https://getaiform.com/dashboard/settings/developer to find:
- **Organization ID**
- **User ID**

## 🚀 How It Works

### Architecture

```
User Request
    ↓
Claude Code (with Dashform Skill)
    ↓
MCP Protocol
    ↓
Dashform MCP Server (http://localhost:3000/api/mcp)
    ↓
Database (Neon PostgreSQL)
    ↓
Form Created ✅
```

### Available MCP Tools

The skill provides access to these MCP tools:

#### 1. `create_form`
Creates a new form in Dashform.

**Parameters:**
- `organizationId` (required): User's organization ID
- `userId` (required): User's user ID
- `name` (required): Form name
- `type` (optional): "dynamic" or "structured" (default: "dynamic")
- `description` (optional): Form description
- `tone` (optional): Form tone (e.g., "friendly", "professional")

**Returns:**
- Form ID
- Share URL (for respondents)
- Edit URL (for customization)

#### 2. `create_reply`
Creates a response/reply for an existing form.

**Parameters:**
- `formId` (required): Form ID
- `organizationId` (required): Organization ID
- `respondentName` (optional): Respondent's name
- `respondentEmail` (optional): Respondent's email
- `respondentEmotion` (optional): "neutral", "positive", or "negative"
- `data` (optional): Initial form data

## 📦 Installation

### Option 1: Use Locally (Recommended for Development)

The skill is already in your project at `.claude/skills/dashform-skill/`. Claude Code will automatically discover it.

### Option 2: Package and Share

To package the skill for sharing:

```bash
cd .claude/skills/dashform-skill
zip -r dashform-skill.zip SKILL.md README.md examples/ references/
```

## 🎓 Usage Examples

### Example 1: Create a Customer Survey

**User:** "Create a customer satisfaction survey"

**Claude's Actions:**
1. Asks for Organization ID and User ID
2. Calls MCP tool `create_form` with:
   ```
   organizationId: <user's org ID>
   userId: <user's user ID>
   name: "Customer Satisfaction Survey"
   type: "dynamic"
   description: "Collect feedback from customers"
   tone: "friendly"
   ```
3. Returns share URL and edit URL

### Example 2: Create a Quiz

**User:** "Create a personality quiz about work styles"

**Claude's Actions:**
1. Gets credentials
2. Calls `create_form` with appropriate parameters
3. Provides URLs for sharing and editing

## 🔧 Development

### Project Structure

```
.claude/skills/dashform-skill/
├── SKILL.md           # Main skill definition (MCP-based)
├── README.md          # This file
├── examples/          # Example forms and use cases
└── references/        # Additional documentation
```

### MCP Server Implementation

The MCP server is implemented at:
- **File**: `src/app/api/mcp/route.ts`
- **Endpoint**: `http://localhost:3000/api/mcp`
- **Protocol**: HTTP-based MCP using `mcp-handler`

### Adding New Tools

To add a new MCP tool:

1. Edit `src/app/api/mcp/route.ts`
2. Register a new tool with `server.registerTool()`
3. Update `SKILL.md` to document the new tool
4. Test the tool through Claude Code

## 🔐 Security Notes

### Current Implementation

⚠️ **Important**: The current MCP server implementation does NOT include authentication. It accepts `organizationId` and `userId` as parameters without verification.

### For Production Use

Before deploying to production, you should:

1. **Add Authentication**: Implement proper authentication in the MCP server
2. **Validate Permissions**: Verify users have permission to access organizations
3. **Rate Limiting**: Add rate limiting to prevent abuse
4. **Audit Logging**: Log all MCP tool calls for security auditing

### Recommended Security Enhancements

```typescript
// Example: Add authentication middleware
async (input) => {
  // Verify the user has access to the organization
  const hasAccess = await verifyOrganizationAccess(
    input.userId,
    input.organizationId
  );

  if (!hasAccess) {
    return {
      content: [{ type: "text", text: "Access denied" }],
      isError: true
    };
  }

  // Proceed with form creation...
}
```

## 🧪 Testing

### Manual Testing

1. Start the development server:
   ```bash
   pnpm dev
   ```

2. Test the MCP server directly:
   ```bash
   curl -X POST http://localhost:3000/api/mcp \
     -H "Content-Type: application/json" \
     -d '{
       "jsonrpc": "2.0",
       "method": "tools/call",
       "params": {
         "name": "create_form",
         "arguments": {
           "organizationId": "org_123",
           "userId": "user_123",
           "name": "Test Form",
           "type": "dynamic"
         }
       },
       "id": 1
     }'
   ```

3. Test through Claude Code:
   - Open Claude Code
   - Say: "Create a test form using Dashform"
   - Provide credentials when asked
   - Verify the form is created

## 📊 Comparison: v1.0 vs v2.0

| Feature | v1.0 (HTTP API) | v2.0 (MCP) |
|---------|-----------------|------------|
| **Integration** | Python HTTP requests | Native MCP tools |
| **Code in Skill** | ~300 lines of Python | ~50 lines of instructions |
| **Type Safety** | Manual validation | Zod schema validation |
| **Error Handling** | Manual try/catch | Built-in MCP error handling |
| **Extensibility** | Add new Python code | Register new MCP tools |
| **Authentication** | Bearer token in code | Managed by MCP server |
| **Maintenance** | Update Python code | Update MCP tool definitions |

## 🐛 Troubleshooting

### Issue 1: MCP Server Not Found

**Error**: "Cannot connect to MCP server"

**Solution**:
1. Verify dev server is running: `pnpm dev`
2. Check MCP endpoint: `curl http://localhost:3000/api/mcp`
3. Verify MCP configuration in Claude Code

### Issue 2: Form Creation Fails

**Error**: "Failed to create form"

**Solution**:
1. Check Organization ID and User ID are valid
2. Verify database connection in `.env.local`
3. Check server logs for detailed error messages

### Issue 3: Skill Not Discovered

**Error**: Skill doesn't appear in Claude Code

**Solution**:
1. Verify skill is in `.claude/skills/` directory
2. Check `SKILL.md` has valid YAML frontmatter
3. Restart Claude Code to refresh skill discovery

## 📚 Additional Resources

- **Dashform Website**: https://getaiform.com
- **MCP Specification**: https://modelcontextprotocol.io
- **Agent Skills Docs**: https://agentskills.io
- **Project CLAUDE.md**: See `/CLAUDE.md` for project architecture

## 🤝 Contributing

To improve this skill:

1. Edit `SKILL.md` for instruction changes
2. Edit `src/app/api/mcp/route.ts` for new MCP tools
3. Test thoroughly with Claude Code
4. Update this README with any changes

## 📄 License

MIT License - See project root for details

## 🎉 Changelog

### v2.0.0 (2024-01-29)
- **BREAKING**: Migrated from HTTP API to MCP protocol
- Added `create_form` MCP tool implementation
- Simplified skill instructions (removed Python code)
- Improved type safety with Zod schemas
- Updated documentation for MCP usage

### v1.0.0 (Initial Release)
- HTTP API-based form creation
- Python code for API calls
- Bearer token authentication
