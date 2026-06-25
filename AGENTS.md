## Taste

- Simple, efficient, elegant.
- Code should be digestible in one pass.
- Every line should earn its place.
- Prefer boring primitives, clear names, and explicit control flow.
- Prefer code that explains itself over code that impresses.
- Avoid cleverness, speculative abstractions, and broad rewrites unless required.

## Systems

- Think in systems.
- Prefer one owner, one state, one source of truth.
- Keep responsibilities narrow and boundaries explicit.
- Preserve invariants.
- Make state transitions obvious.
- Make failure modes visible and recoverable.
- Fix at the right boundary.

## Change

- Prefer deletion, consolidation, and fewer moving parts before adding code.
- Small, verifiable changes beat impressive large changes.
- Weak evidence means no-change is valid.
- Read the real code, config, logs, data, or provider output before claims about behavior.
