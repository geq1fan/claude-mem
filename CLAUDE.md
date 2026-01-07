# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Claude-Mem: AI Development Instructions

Claude-mem is a Claude Code plugin providing persistent memory across sessions. It captures tool usage, compresses observations using the Claude Agent SDK, and injects relevant context into future sessions.

## Architecture

**5 Lifecycle Hooks**: SessionStart → UserPromptSubmit → PostToolUse → Summary → SessionEnd

**Hooks** (`src/hooks/*.ts`) - TypeScript → ESM, built to `plugin/scripts/*-hook.js`

**Worker Service** (`src/services/worker-service.ts`) - Express API on port 37777, Bun-managed, handles AI processing asynchronously

**Database** (`src/services/sqlite/`) - SQLite3 at `~/.claude-mem/claude-mem.db`

**Search Skill** (`plugin/skills/mem-search/SKILL.md`) - HTTP API for searching past work, auto-invoked when users ask about history

**Chroma** (`src/services/sync/ChromaSync.ts`) - Vector embeddings for semantic search

**Viewer UI** (`src/ui/viewer/`) - React interface at http://localhost:37777, built to `plugin/ui/viewer.html`

## Privacy Tags
- `<private>content</private>` - User-level privacy control (manual, prevents storage)

**Implementation**: Tag stripping happens at hook layer (edge processing) before data reaches worker/database. See `src/utils/tag-stripping.ts` for shared utilities.

## Build Commands

```bash
npm run build-and-sync        # Build, sync to marketplace, restart worker
```

## Configuration

Settings are managed in `~/.claude-mem/settings.json`. The file is auto-created with defaults on first run.

## File Locations

- **Source**: `<project-root>/src/`
- **Built Plugin**: `<project-root>/plugin/`
- **Installed Plugin**: `~/.claude/plugins/marketplaces/thedotmack/`
- **Database**: `~/.claude-mem/claude-mem.db`
- **Chroma**: `~/.claude-mem/chroma/`

## Requirements

- **Bun** (all platforms - auto-installed if missing)
- **uv** (all platforms - auto-installed if missing, provides Python for Chroma)
- Node.js

## Documentation

**Public Docs**: https://docs.claude-mem.ai (Mintlify)
**Source**: `docs/public/` - MDX files, edit `docs.json` for navigation
**Deploy**: Auto-deploys from GitHub on push to main

## Pro Features Architecture

Claude-mem is designed with a clean separation between open-source core functionality and optional Pro features.

**Open-Source Core** (this repository):

- All worker API endpoints on localhost:37777 remain fully open and accessible
- Pro features are headless - no proprietary UI elements in this codebase
- Pro integration points are minimal: settings for license keys, tunnel provisioning logic
- The architecture ensures Pro features extend rather than replace core functionality

**Pro Features** (coming soon, external):

- Enhanced UI (Memory Stream) connects to the same localhost:37777 endpoints as the open viewer
- Additional features like advanced filtering, timeline scrubbing, and search tools
- Access gated by license validation, not by modifying core endpoints
- Users without Pro licenses continue using the full open-source viewer UI without limitation

This architecture preserves the open-source nature of the project while enabling sustainable development through optional paid features.

## Important

No need to edit the changelog ever, it's generated automatically.

## Fork: OpenAI Compatible Provider

This fork adds support for OpenAI-compatible API endpoints (Azure OpenAI, local models like Ollama/LM Studio, etc.).

### Fork-Specific Files

- `src/services/worker/OpenAICompatibleAgent.ts` - New agent implementation (not in upstream)
- Settings: `CLAUDE_MEM_OPENAI_COMPATIBLE_*` (API_KEY, BASE_URL, MODEL, MAX_CONTEXT_MESSAGES, MAX_TOKENS)

### Fork Code Markers

All fork-specific modifications are marked with comments for easy identification during sync:
- `// === Fork: ... ===` - Block markers
- `/* fork */` - Inline markers

Search for these markers to locate all fork modifications:
```bash
grep -r "Fork:" src/ --include="*.ts" --include="*.tsx"
grep -r "/\* fork \*/" src/ --include="*.ts" --include="*.tsx"
```

### Upstream Sync Guidelines

When syncing from upstream (thedotmack/claude-mem):

1. **Before Pull**
   ```bash
   git stash  # Save local changes
   git fetch upstream
   ```

2. **Conflict Resolution Priority**
   - Accept upstream changes first
   - Re-apply fork code at **end of files/arrays** (not middle)
   - Ensure `'openai-compatible'` stays last in type unions

3. **High-Conflict Files** (designed for minimal conflicts)
   | File | Fork Strategy |
   |------|---------------|
   | `SettingsDefaultsManager.ts` | Fork configs at file end |
   | `SessionRoutes.ts` | Fork param is optional, helper at file end |
   | `worker-types.ts` | Type appended with `/* fork */` |
   | `SettingsRoutes.ts` | Array item marked with `/* fork */` |

4. **After Merge**
   ```bash
   bun build src/services/worker-service.ts --outdir=/tmp/test --target=bun  # Verify build
   npm run build-and-sync  # Rebuild plugin
   ```

5. **If OpenAICompatibleAgent.ts Conflicts**
   - This file is fork-only, upstream won't have it
   - If upstream adds similar functionality, consider migrating to their implementation
