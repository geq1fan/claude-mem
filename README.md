# Claude-Mem Fork: Extended Provider Support

This fork adds **extended provider support** to [claude-mem](https://github.com/thedotmack/claude-mem).

## What's Different

| Feature | Original | This Fork |
|---------|----------|-----------|
| AI Providers | Claude, Gemini, OpenRouter | + **OpenAI Compatible** |
| Gemini Custom Endpoint | ❌ | ✅ Custom Base URL |
| Folder CLAUDE.md Toggle | Always ON | ✅ Configurable |
| Non-SDK Agent FK Fix | ❌ | ✅ Fixed |

---

## New Feature 1: Gemini Custom Base URL

Allows using Gemini API proxies or alternative endpoints.

### Configuration

```json
{
  "CLAUDE_MEM_PROVIDER": "gemini",
  "CLAUDE_MEM_GEMINI_API_KEY": "your-api-key",
  "CLAUDE_MEM_GEMINI_BASE_URL": "https://your-proxy.com/v1beta/models",
  "CLAUDE_MEM_GEMINI_MODEL": "gemini-2.5-flash"
}
```

**Note**: Model validation is removed - any model name is accepted for proxy compatibility.

---

## New Feature 2: OpenAI Compatible Provider

Supports any OpenAI-compatible API endpoint:

- **OpenAI Official** (GPT-4o, GPT-4o-mini)
- **Azure OpenAI**
- **Local Models** (Ollama, LM Studio, vLLM)
- **Other Providers** (Groq, Together AI, Anyscale)

### Configuration

Settings in `~/.claude-mem/settings.json`:

```json
{
  "CLAUDE_MEM_PROVIDER": "openai-compatible",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_API_KEY": "your-api-key",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_BASE_URL": "https://api.openai.com/v1/chat/completions",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_MODEL": "gpt-4o-mini",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_MAX_CONTEXT_MESSAGES": "20",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_MAX_TOKENS": "100000"
}
```

### Example Configurations

**Ollama (Local)**:
```json
{
  "CLAUDE_MEM_PROVIDER": "openai-compatible",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_API_KEY": "ollama",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_BASE_URL": "http://localhost:11434/v1/chat/completions",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_MODEL": "llama3.2"
}
```

**Azure OpenAI**:
```json
{
  "CLAUDE_MEM_PROVIDER": "openai-compatible",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_API_KEY": "your-azure-key",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_BASE_URL": "https://your-resource.openai.azure.com/openai/deployments/your-deployment/chat/completions?api-version=2024-02-15-preview",
  "CLAUDE_MEM_OPENAI_COMPATIBLE_MODEL": "gpt-4o"
}
```

---

## New Feature 3: Folder CLAUDE.md Toggle

Controls automatic generation of `CLAUDE.md` files in subdirectories.

```json
{
  "CLAUDE_MEM_FOLDER_CLAUDEMD_ENABLED": "false"
}
```

- `"false"` (default): Disabled - no scattered CLAUDE.md files
- `"true"`: Enabled - generates CLAUDE.md in modified folders

---

## Bug Fix: Non-SDK Agent FK Constraint

Fixed foreign key constraint errors for Gemini/OpenRouter/OpenAI-compatible agents.

**Problem**: Non-SDK agents failed to store observations because `memory_session_id` was NULL.

**Solution**: Set `memory_session_id = contentSessionId` at session start for non-SDK agents.

---

## Installation

Same as original:

```
> /plugin marketplace add thedotmack/claude-mem
> /plugin install claude-mem
```

Then configure the OpenAI Compatible provider in settings.

## Upstream Sync

This fork is designed for easy sync with upstream:

- All fork code marked with `// === Fork: ... ===` or `/* fork */`
- Fork-specific configs at end of files (not middle)
- New file: `src/services/worker/OpenAICompatibleAgent.ts`

### Sync Commands

```bash
# Fetch upstream changes
git fetch upstream

# Merge upstream
git merge upstream/main

# Resolve conflicts (fork markers help locate changes)
grep -r "Fork:" src/ --include="*.ts"

# Verify build
bun build src/services/worker-service.ts --outdir=/tmp/test --target=bun

# Rebuild plugin
npm run build-and-sync
```

## Links

- **Original Repository**: https://github.com/thedotmack/claude-mem
- **Original Documentation**: https://docs.claude-mem.ai
- **This Fork**: https://github.com/geq1fan/claude-mem

## License

Same as original: **AGPL-3.0**
