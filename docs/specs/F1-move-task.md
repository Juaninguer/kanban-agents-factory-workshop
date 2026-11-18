# F1 — Move a task between columns

## Summary

Allow a user to change a task's status among `todo`, `doing`, and `done`, so the task moves between the **To do**, **Doing**, and **Done** columns. The API must validate both the task ID and the target status. The primary interaction must be operable with a keyboard.

The API design is a `PATCH /api/tasks/:id` partial-update route with a `status` field, consistent with Express, Zod validation, and the existing resource-oriented `/api/tasks` routes.

## Acceptance criteria

### AC1 — Move an existing task

- **Given** a task exists in the To do column
- **When** the user moves it to Doing
- **Then** the task disappears from To do and appears once in Doing
- **And** a subsequent `GET /api/tasks` reports the same task with `status: "doing"`

### AC2 — Move to every supported column

- **Given** an existing task
- **When** it is moved between any pair of To do, Doing, and Done
- **Then** the task is rendered in exactly the selected column and its API representation has the corresponding `todo`, `doing`, or `done` status

### AC3 — Reject an unknown task

- **Given** no task exists with ID `missing-id`
- **When** a client sends `PATCH /api/tasks/missing-id` with a valid status
- **Then** the API responds with `404` and `{ "error": "NotFound" }`
- **And** no task is created or changed

### AC4 — Reject an invalid target status

- **Given** an existing task
- **When** a client sends a target status outside `todo`, `doing`, or `done`
- **Then** the API responds with `400` and `{ "error": "ValidationError", "details": [...] }`
- **And** the task retains its current status

### AC5 — Keyboard-accessible move control

- **Given** a task card is visible
- **When** a keyboard user focuses the card's move control and changes the selected destination
- **Then** the task moves to the selected column without pointer interaction
- **And** the control has an accessible name that identifies the task

### AC6 — Show move failures

- **Given** the API rejects a move
- **When** the move request fails
- **Then** the UI keeps the task in its current column and displays an accessible error
- **And** the move control can be used again

## API changes (routes, payloads, status codes, validation rules)

- Add `PATCH /api/tasks/:id`.
- Request body: `{ "status": "todo" | "doing" | "done" }`.
- Use Zod to require exactly a supported `status` value. An omitted status or unknown status returns `400 ValidationError`.
- Return `200` with the complete updated `Task` object on success.
- Return `404` with `{ "error": "NotFound" }` when the task ID does not exist.
- Preserve existing fields, creation timestamp, and ordering behavior.
- If the destination equals the current status, return `200` with the unchanged task.
- Add a store-level update operation that mutates only the matching task status.
- Existing `POST /api/tasks`, `GET /api/tasks`, and test reset behavior remain unchanged.

## UI changes (components, accessible names / roles, keyboard behaviour)

- Update [web/src/App.tsx](../../web/src/App.tsx) to provide a move-task handler and optimistic state update pattern only if the selected implementation requires it; the final state must be the server-returned updated task.
- Update [web/src/components/Board.tsx](../../web/src/components/Board.tsx) to pass the move callback to columns.
- Update [web/src/components/Column.tsx](../../web/src/components/Column.tsx) to pass the move callback to task cards.
- Update [web/src/components/TaskCard.tsx](../../web/src/components/TaskCard.tsx) to render a native `<select>` with three options labeled To do, Doing, and Done, named for example `Move "Task title"` (the exact accessible name is implementation-specific).
- Changing the select value invokes the move API call; native keyboard navigation and Enter/Space behavior must remain usable.
- On success, replace the task in client state with the returned task so it relocates by status.
- On failure, leave the task where it was and render `role="alert"` feedback. The select value must reflect the task's current status after failure.
- The move control must not be nested in an interactive element or rely on drag-and-drop.
- Existing card title and column region accessible names remain available.

## Implementation plan (numbered steps, small enough for one commit each, with file paths)

1. Add a Zod schema for a task status update in [server/src/domain.ts](../../server/src/domain.ts).
2. Add a store method to find a task by ID and update its status, or a no-op result if it is unchanged, in [server/src/store.ts](../../server/src/store.ts).
3. Add `PATCH /api/tasks/:id` in [server/src/app.ts](../../server/src/app.ts), including Zod `400` and missing-task `404` handling.
4. Add `moveTask(id, status)` to [web/src/api.ts](../../web/src/api.ts).
5. Add a keyboard-operable move select and failure state to [web/src/components/TaskCard.tsx](../../web/src/components/TaskCard.tsx).
6. Thread the move callback through [web/src/components/Board.tsx](../../web/src/components/Board.tsx), [web/src/components/Column.tsx](../../web/src/components/Column.tsx), and [web/src/App.tsx](../../web/src/App.tsx).
7. Add API coverage in [server/test/tasks.test.ts](../../server/test/tasks.test.ts).
8. Add Playwright coverage in [e2e/board.spec.ts](../../e2e/board.spec.ts) using [e2e/fixtures.ts](../../e2e/fixtures.ts) for reset state.
9. Update card and control styling only in [web/src/styles.css](../../web/src/styles.css), if needed.

## Test plan (which acceptance criteria → unit/API test vs. e2e test)

- **AC1:** API test moves an existing task with `PATCH` and verifies `200`, updated status, and subsequent `GET`; E2E test moves a seeded task and verifies column membership.
- **AC2:** API test table-covers all supported status values and store methods; E2E test verifies movement among all three columns.
- **AC3:** API test sends `PATCH /api/tasks/missing-id` and verifies `404` plus `{ "error": "NotFound" }`.
- **AC4:** API test table-covers omitted status and unsupported values and verifies `400 ValidationError` and unchanged status.
- **AC5:** E2E test focuses the named move control with the keyboard, changes its value, and verifies the task region moves without pointer interaction.
- **AC6:** E2E test forces a rejected move or API outage and verifies `role="alert"`, no relocation, and reuse of the control.

## Out of scope

- Drag-and-drop movement.
- Ordering or ranking tasks within a column.
- Moving multiple tasks at once.
- WIP limits.
- Renaming, deleting, filtering, or prioritizing tasks.
- Persistence of moves across server restarts.
- Animations or confirmation dialogs.

## Open questions

- Should dragging cards be added in addition to the keyboard move control? Default: no; native select-only movement is sufficient for F1.
- Should a move in the UI use an optimistic update or wait for the API response? Default: wait for success and replace client state with the returned task, avoiding a false move on failure.
- Which exact `PATCH` response field shape should consumers see? Default: return the complete updated `Task` object.
- Should extra properties in the request body be accepted or rejected? Default: follow Zod's existing object parsing behavior, but do not add new requirements beyond requiring `status`.
- How should rapid repeated move selections be handled while a request is pending? Default: disable or ignore the control for that card until the request completes; final behavior should prevent stale responses.
