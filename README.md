# KAGE (影)

An execution protocol for AI coding assistants that cuts token overhead and enforces production-ready code without conversational filler.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Protocol Version](https://img.shields.io/badge/Protocol-v1.0-green.svg)](skills/kage/SKILL.md)
[![Tokens](https://img.shields.io/badge/Input_Overhead-~70_tokens-brightgreen.svg)](#efficiency-benchmarks)

## Overview

Most system prompts add hundreds or thousands of tokens of process checklists and conversational rules to every request. This wastes context, slows down model responses, and increases API costs.

Models follow direct technical constraints more reliably than lengthy process instructions. KAGE enforces runnable, fully typed, secure code without conversational filler, adding under 70 tokens of input overhead.

## Execution contract

| Rule | Directives |
| :--- | :--- |
| 1. Direct output | Skip conversational filler (`"Sure, I can help"`), intros, and commentary. Return code and facts directly. |
| 2. Production complete | Deliver fully typed, runnable code without placeholders (`// TODO`, `/* implement later */`). |
| 3. Explicit security | Validate and sanitize inputs at trust boundaries. Use no hardcoded secrets or tokens. Handle errors without exposing stack traces. |
| 4. High density | Put code first. Note edge cases and trade-offs in a single line. |

## Example

### Prompt
```text
Create an Express.js middleware to verify Bearer JWT tokens.
```

### Standard assistant response
> *"Sure! I would be happy to help you with that. Securing endpoints is crucial for web applications. Below is an example implementation using the jsonwebtoken library. Make sure you have installed it first..."*
> 
> ```javascript
> function auth(req, res, next) {
>   const token = req.headers['authorization'];
>   // TODO: validate header format properly
>   jwt.verify(token, "mysecretkey", (err, user) => { // Insecure hardcoded secret
>     if (err) return res.send(err); // Leaking internal error details
>     req.user = user;
>     next();
>   });
> }
> ```
> 
> *"I hope this helps! Let me know if you need database lookups or refresh tokens!"*

### KAGE response
```typescript
import type { Request, Response, NextFunction } from 'express';
import jwt, { JwtPayload } from 'jsonwebtoken';

export type AuthRequest = Request & { user?: string | JwtPayload };

export const authenticateToken = (req: AuthRequest, res: Response, next: NextFunction): void => {
  const token = req.headers.authorization?.startsWith('Bearer ')
    ? req.headers.authorization.slice(7)
    : null;

  if (!token) {
    res.status(401).json({ error: 'Unauthorized: Missing or malformed token' });
    return;
  }

  const secret = process.env.JWT_SECRET;
  if (!secret) {
    res.status(500).json({ error: 'Internal configuration error' });
    return;
  }

  try {
    req.user = jwt.verify(token, secret);
    next();
  } catch {
    res.status(403).json({ error: 'Forbidden: Invalid or expired token' });
  }
};
```
*Trade-off: Uses symmetric HMAC (`JWT_SECRET`); upgrade to RS256 if auth provider is externalized.*

## Efficiency benchmarks

| Metric | Traditional prompts | KAGE protocol | Impact |
| :--- | :--- | :--- | :--- |
| Context overhead | 2,500 to 4,000 tokens | ~70 tokens | ~97% reduction |
| Inference latency | High (lengthy tags and multi-step rituals) | Negligible overhead | Faster time to first token |
| Code completeness | Enforced | Enforced | Parity |
| Security boundaries | Theoretical taxonomies | Inline checks | Production-ready |

## Installation and setup

<details>
<summary>Claude Code</summary>

Save to your skills directory:

```bash
mkdir -p ~/.claude/skills/kage
curl -sLo ~/.claude/skills/kage/SKILL.md https://raw.githubusercontent.com/cmsdp/kage/main/skills/kage/SKILL.md
```

Or add to `~/.claude/CLAUDE.md`:

```markdown
## KAGE Protocol (Active on all responses)
- Direct output: Zero filler, no conversational intro/outro.
- Production code: Fully typed, runnable, zero placeholders (`// TODO`).
- Security mandatory: Sanitize inputs, zero hardcoded secrets, no raw trace leaks.
- Density: Code first, trade-offs in one short line.
```

Invoke on demand via `/kage`.

</details>

<details>
<summary>Codex</summary>

```bash
codex plugin marketplace add cmsdp/kage
codex plugin add kage@cmsdp
```

Updates itself at startup.

</details>

<details>
<summary>GitHub Copilot (CLI and VS Code)</summary>

```bash
copilot plugin marketplace add cmsdp/kage
copilot plugin install kage@cmsdp
```

Then add to `~/.copilot/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "cmsdp": { "source": { "source": "github", "repo": "cmsdp/kage" }, "autoUpdate": true }
  }
}
```

In VS Code, run **Chat: Install Plugin From Source** and enter `https://github.com/cmsdp/kage`.

Or append to `.github/copilot-instructions.md` in repository root.

</details>

<details>
<summary>Gemini CLI (manual updates)</summary>

```bash
gemini skills install https://github.com/cmsdp/kage.git --path skills/kage
```

Run the same command to update.

</details>

<details>
<summary>Other agents and custom configs</summary>

```bash
npx skills@latest add cmsdp/kage -a <agent>  # cursor, opencode, devin, windsurf, amp, pi; omit -a to choose
```

To update, run `npx skills@latest update`, and re-run `add` to pick up new updates.

Or append manually to `.cursorrules`, `.windsurfrules`, `.clinerules`, or `.aider.conf.yml`:
```text
Apply KAGE Protocol:
- Direct output: No pleasantries, no conversational preamble.
- Production code: Complete, fully typed, zero placeholders.
- Security: Sanitize inputs, zero hardcoded credentials, safe error handling.
- Density: Code first, single-line trade-off note.
```

</details>

<details>
<summary>REST APIs and system prompts</summary>

Embed directly into the `system` parameter:

```text
Direct answer first. No conversational filler, no placeholders.
Code must be production-ready and complete: no hardcoded secrets, validate all inputs.
```

</details>

## Specification

The reference specification is in [`skills/kage/SKILL.md`](skills/kage/SKILL.md).

## Contributing

Open an issue or pull request to contribute benchmarks or integrations.

## License

[MIT](LICENSE)
