# Knowledge Routing Index

Read this file first. It maps task intent to the smallest useful SSOT document set.

## Global Rules

- Use project-local documentation first when it exists.
- Use this repository for reusable principles and decision criteria.
- Do not read every SSOT document by default.
- For implementation or review output, cite the selected SSOT file paths.
- If selected SSOT conflicts with project-local rules, report the conflict and follow project-local rules unless the user asks for general guidance.

## React State Placement

Use when:

- Deciding whether state belongs in local state, URL, React Query, React Hook Form, or global store.
- Reviewing duplicate state, derived state, URL sync, or server cache misuse.

Read:

- `docs/roadmap/1-프론트엔드-상태관리-전략.md`
- `docs/roadmap/8-프론트엔드-useEffect-사용-전략.md`
- `docs/roadmap/9-프론트엔드-React-Hook-Form-사용-전략.md`

## Component Boundary And UI Responsibility

Use when:

- Deciding whether logic belongs in component, hook, model, mapper, or API layer.
- Reviewing props boundaries, UI/domain separation, and feature component design.

Read:

- `docs/roadmap/2-프론트엔드-컴포넌트-설계-원칙.md`
- `docs/roadmap/4-프론트엔드-추상화-레벨.md`

## API, DTO, Mapper, And Error Boundary

Use when:

- Reviewing API response validation, Zod schemas, DTO leakage, request serialization, or mapper boundaries.
- Designing error handling across API, use-case hook, and UI feedback.

Read:

- `docs/roadmap/5-프론트엔드-에러-핸들링-전략.md`
- `docs/roadmap/6-프론트엔드-에러-모델링.md`
- `docs/roadmap/7-프론트엔드-디자인-패턴.md`

## Rendering Strategy

Use when:

- Choosing CSR, SSR, SSG, or ISR.
- Reviewing Next.js page rendering decisions for logged-in operational systems.

Read:

- `docs/roadmap/3-프론트엔드-렌더링-전략.md`

## Form Design

Use when:

- Designing search, create, update, or validation-heavy forms.
- Deciding where default values, reset, serializer, and request payload creation live.

Read:

- `docs/roadmap/9-프론트엔드-React-Hook-Form-사용-전략.md`
- `docs/roadmap/1-프론트엔드-상태관리-전략.md`
- `docs/roadmap/2-프론트엔드-컴포넌트-설계-원칙.md`

## useEffect And Side Effects

Use when:

- Reviewing `useEffect` usage.
- Deciding whether logic belongs in render, event handler, React Query, or a real effect.

Read:

- `docs/roadmap/8-프론트엔드-useEffect-사용-전략.md`
- `docs/roadmap/1-프론트엔드-상태관리-전략.md`

## Abstraction And Refactoring

Use when:

- Refactoring duplicated logic.
- Deciding whether extraction is a real abstraction.
- Naming domain helpers or separating change reasons.

Read:

- `docs/roadmap/4-프론트엔드-추상화-레벨.md`
- `docs/roadmap/7-프론트엔드-디자인-패턴.md`
- `docs/roadmap/2-프론트엔드-컴포넌트-설계-원칙.md`

## E2E And Regression Testing

Use when:

- Introducing or reviewing Playwright E2E tests.
- Deciding what migration flows need browser-level regression coverage.

Read:

- `docs/ADR/[ADR-001] 프론트엔드 E2E 테스트 도입.md`

