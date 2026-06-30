# Knowledge Summary Index

This repository is a reusable SSOT for frontend engineering decisions.

## High Priority Principles

- State ownership: choose local, form, global, server, URL, optimistic, or derived state by owner and lifecycle.
- Component boundary: components render domain state and emit events; hooks/model/api layers own policy, data access, and transformation.
- Abstraction: extraction is not abstraction. A good abstraction exposes intent, stable inputs/outputs, and one change reason.
- Error handling: API detects failures, use-case layer interprets failure meaning, UI layer presents feedback.
- useEffect: use it only for real external side effects, not derived state or event logic.
- React Hook Form: use it for multi-field forms with submit/reset/validation/payload concerns.

## Decision Documents

- `docs/ADR/[ADR-001] 프론트엔드 E2E 테스트 도입.md`: Playwright E2E is accepted for migration regression coverage.

## Routing Entry Point

Always start with:

- `indexes/routing-index.md`

Then read only the SSOT files selected by the current task.

