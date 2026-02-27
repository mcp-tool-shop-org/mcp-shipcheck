# Error Contract

Structured errors across all MCP Tool Shop products.

Two tiers: the **shape** is mandatory everywhere, the **base type** is optional for larger projects.

---

## Tier 1: Structured Error Shape (mandatory)

Every user-facing error — whether thrown, returned, or printed — must carry these fields:

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `code` | string | yes | Machine-readable, namespaced (see registry rules below) |
| `message` | string | yes | What went wrong |
| `hint` | string | yes | What the user should do about it |
| `cause` | string/error | no | Original error, for chaining |
| `retryable` | boolean | no | Can the caller safely retry? |

**Simple scripts** don't need a class. Just emit the shape:

```json
{
  "code": "IO_READ_FAILED",
  "message": "Could not read config.json",
  "hint": "Check file permissions or run with --debug",
  "retryable": false
}
```

```python
# Minimal: return a dict
{"code": "IO_READ_FAILED", "message": str(e), "hint": "Check file permissions"}
```

The contract is the shape, not the implementation.

---

## Tier 2: Base Error Type + Exit Codes (CLI / MCP / Desktop)

For repos with a CLI, MCP server, or desktop UI — formalize the shape into a typed error class with two output modes.

### Output modes

| Mode | Audience | Stack traces? | When |
|------|----------|---------------|------|
| **safe** | End users, LLMs, MCP | Never | Default |
| **debug** | Developers | Yes, if available | `--debug` flag |

### Exit codes (CLI only)

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | User error (bad input, missing args, invalid config) |
| 2 | Runtime error (crash, IO failure, dependency missing) |
| 3 | Partial success (some items ok, some failed) |

### TypeScript reference

```typescript
export class ProductError extends Error {
  readonly code: string;
  readonly hint: string;
  readonly cause?: Error;
  readonly retryable: boolean;

  constructor(
    code: string,
    message: string,
    hint: string,
    opts?: { cause?: Error; retryable?: boolean },
  ) {
    super(message);
    this.name = 'ProductError'; // rename per-product
    this.code = code;
    this.hint = hint;
    this.cause = opts?.cause;
    this.retryable = opts?.retryable ?? false;
  }

  /** Safe output — no stack traces. For MCP responses + UI. */
  toSafeText(): string {
    const lines = [
      `Error [${this.code}]: ${this.message}`,
      `Hint: ${this.hint}`,
    ];
    if (this.cause) lines.push(`Cause: ${this.cause.message}`);
    return lines.join('\n');
  }

  /** Debug output — includes stack. For CLI --debug. */
  toDebugText(): string {
    const lines = [
      `[${this.code}] ${this.message}`,
      `  Hint: ${this.hint}`,
    ];
    if (this.cause) {
      lines.push(`  Cause: ${this.cause.message}`);
      if (this.cause.stack) lines.push(`  Stack: ${this.cause.stack}`);
    }
    return lines.join('\n');
  }
}

export function wrapError(
  err: unknown,
  code: string,
  hint: string,
): ProductError {
  if (err instanceof ProductError) return err;
  const cause = err instanceof Error ? err : new Error(String(err));
  return new ProductError(code, cause.message, hint, { cause });
}
```

### Python reference

```python
class ProductError(Exception):
    """Structured error with code, hint, and optional cause."""

    def __init__(
        self,
        code: str,
        message: str,
        hint: str,
        *,
        cause: Exception | None = None,
        retryable: bool = False,
    ):
        super().__init__(message)
        self.code = code
        self.hint = hint
        self.__cause__ = cause
        self.retryable = retryable

    def to_safe_text(self) -> str:
        """Safe output — no stack traces. For MCP responses + UI."""
        lines = [f"Error [{self.code}]: {self}", f"Hint: {self.hint}"]
        if self.__cause__:
            lines.append(f"Cause: {self.__cause__}")
        return "\n".join(lines)

    def to_debug_text(self) -> str:
        """Debug output — includes stack. For CLI --debug."""
        import traceback

        lines = [f"[{self.code}] {self}", f"  Hint: {self.hint}"]
        if self.__cause__:
            lines.append(f"  Cause: {self.__cause__}")
            lines.append(
                "".join(traceback.format_exception(self.__cause__))
            )
        return "\n".join(lines)


def wrap_error(err: Exception, code: str, hint: str) -> ProductError:
    """Wrap any exception into a ProductError."""
    if isinstance(err, ProductError):
        return err
    return ProductError(code, str(err), hint, cause=err)
```

### .NET pattern

```csharp
public class ProductException : Exception
{
    public string Code { get; }
    public string Hint { get; }
    public bool Retryable { get; }

    public ProductException(
        string code, string message, string hint,
        Exception? cause = null, bool retryable = false)
        : base(message, cause)
    {
        Code = code;
        Hint = hint;
        Retryable = retryable;
    }

    public string ToSafeText() =>
        $"Error [{Code}]: {Message}\nHint: {Hint}"
        + (InnerException != null ? $"\nCause: {InnerException.Message}" : "");
}
```

### Naming convention

Each product renames the class: `GuardianError`, `AttestiaError`, `SoundboardError`, etc.

---

## Error Code Registry Rules

Three rules.

### 1. Codes are namespaced

Use a prefix from this table (extend as needed):

| Prefix | Domain |
|--------|--------|
| `IO_` | File system, streams, paths |
| `CONFIG_` | Configuration, settings, env vars |
| `PERM_` | Permissions, access control |
| `DEP_` | Dependencies, external services |
| `RUNTIME_` | Unexpected failures, OOM, timeouts |
| `PARTIAL_` | Some items succeeded, some failed |
| `INPUT_` | Bad user input, validation failures |
| `STATE_` | Corrupt or stale internal state |

Examples: `IO_READ_FAILED`, `CONFIG_MISSING_KEY`, `STATE_CORRUPT`, `DEP_OLLAMA_UNREACHABLE`

### 2. Codes are stable once released

A code that appears in a shipped version is permanent. You can add new codes. You can deprecate old ones (map to a successor). You cannot change what an existing code means.

### 3. Every code maps to an exit code + default hint

Maintain a table in your repo (in the error file or a separate `ERROR_CODES.md`):

```
IO_READ_FAILED      → exit 2  → "Check file exists and permissions are correct"
CONFIG_MISSING_KEY   → exit 1  → "Add the required key to your config file"
STATE_CORRUPT        → exit 2  → "Run --reset to rebuild state"
PARTIAL_BATCH        → exit 3  → "Check logs for individual failures"
```

This table is your error contract with users. Treat it like an API.
