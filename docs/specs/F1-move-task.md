# F1 — Move a task between columns

## Summary

Allow a user to move an existing task to To do, Doing, or Done. The change is enforced by the API and reflected immediately in the board. The control is keyboard operable, so a task can be moved without a mouse.

## Acceptance criteria

### AC1 — Move a task to another column

- **Given** a board with a task titled "Write tests" in To do
- **When** the user chooses Done in the task's move control
- **Then** "Write tests" appears in the Done column and no longer appears in To do
- **And** the move is persisted in the server for the life of the running process

### AC2 — Moving is available for every task and keyboard accessible

- **Given** a task titled "Write tests" in any column
- **When** the user focuses the `combobox` named `Move "Write tests"` and changes it using the keyboard
- **Then** the task moves to the selected column

### AC3 — Invalid API status is rejected

- **Given** a board containing a task
- **When** `PATCH /api/tasks/:id` is sent with a status other than `todo`, `doing`, or `done`
- **Then** the API returns 400 with `error: "ValidationError"`

### AC4 — Unknown task cannot be moved

- **Given** no task exists with id "missing-task"
- **When** `PATCH /api/tasks/missing-task` is sent with a valid status
- **Then** the API returns 404 with `error: "NotFound"`

### AC5 — UI handles a rejected move

- **Given** a board whose task is currently in To do
- **When** the user selects another status and the API rejects the move
- **Then** the task remains in To do and the UI shows the API error in an accessible `alert`

## API changes (routes, payloads, status codes, validation rules)

Use `PATCH /api/tasks/:id` with JSON payload `{ "status": "todo" | "doing" | "done" }`. This is the REST-style route for changing one field of an existing task and matches the existing `/api/tasks` resource.

- Validate the payload with a zod schema derived from `TASK_STATUSES`.
- Return `200` with the complete updated task.
- Return `400` with `error: "ValidationError"` and zod details for a missing or invalid `status`.
- Return `404` with `error: "NotFound"` when the id is unknown.
- Add a store method to update the task status; preserve the task id, title, and createdAt.
- Unknown extra payload fields should follow the existing zod object behavior and be rejected only if that behavior is configured; no additional F1 fields are accepted by the product UI.

## UI changes (components, accessible names / roles, keyboard behaviour)

- Add a native `combobox` to each `TaskCard`:
  - Accessible name: `Move "<task title>"`
  - Options: `To do`, `Doing`, `Done`
  - The selected option is the task's current status.
- Selecting a different option calls `api.moveTask(task.id, status)`.
- On success, replace the moved task in the shared task state so it appears in the appropriate column.
- On failure, leave the task state unchanged and show the API error in an existing app-level or task-level `alert`.
- The native control must be operable with Tab, Arrow keys, and Enter/Escape according to the browser.
- Preserve visible task title and column semantics.

## Implementation plan (numbered steps, small enough for one commit each, with file paths)

1. Add a zod schema for move payloads and a store update method in [server/src/domain.ts](../../server/src/domain.ts) and [server/src/store.ts](../../server/src/store.ts).
2. Add `PATCH /api/tasks/:id`, status validation, 404 handling, and response serialization in [server/src/app.ts](../../server/src/app.ts).
3. Add `moveTask(id, status)` to [web/src/api.ts](../../web/src/api.ts).
4. Pass a move callback from [web/src/App.tsx](../../web/src/App.tsx) through [web/src/components/Board.tsx](../../web/src/components/Board.tsx) and [web/src/components/Column.tsx](../../web/src/components/Column.tsx) to [web/src/components/TaskCard.tsx](../../web/src/components/TaskCard.tsx).
5. Add the accessible move combobox and move-error handling in [web/src/components/TaskCard.tsx](../../web/src/components/TaskCard.tsx) and [web/src/App.tsx](../../web/src/App.tsx).
6. Add API and store coverage to [server/test/tasks.test.ts](../../server/test/tasks.test.ts), without changing existing tests.
7. Add the F1 Playwright scenarios in **new** [e2e/F1-move-task.spec.ts](../../e2e/F1-move-task.spec.ts).

## Test plan (which acceptance criteria → unit/API test vs. e2e test)

- AC1:
  - API test in `server/test/tasks.test.ts`: PATCH moves a known task and 200 response contains the new status.
  - E2E in `e2e/F1-move-task.spec.ts`: initial board state contains "Write tests" in todo, select Done from the move combobox, and verify it leaves To do and appears in Done.
- AC2:
  - E2E in `e2e/F1-move-task.spec.ts`: initial board state contains "Write tests" in todo; focus `Move "Write tests"`; use keyboard selection to choose Doing; verify the task appears in Doing.
- AC3:
  - API test in `server/test/tasks.test.ts`: PATCH with missing status and unknown status both return 400 `ValidationError`.
- AC4:
  - API test in `server/test/tasks.test.ts`: PATCH for "missing-task" with status done returns 404 `NotFound`.
- AC5:
  - E2E in `e2e/F1-move-task.spec.ts`: initial board state contains "Write tests" in todo; intercept `PATCH /api/tasks/:id` with Playwright `page.route` and return 404 JSON `{ "error": "NotFound" }`; select Done from `Move "Write tests"`; verify "Write tests" remains in To do and a role `alert` reports `NotFound`.

## Out of scope

- Drag-and-drop movement
- WIP limits
- Moving multiple tasks at once
- Persistence across server restarts
- Animation or column transition effects

## Assumptions (decisions taken by default, with a one-line reason each)

- Choosing the task's current status is a no-op from the UI; this avoids an unnecessary PATCH and server update.
- API moves are processed in memory and remain until the server restart or reset; this follows the existing store model.
- Native `<select>` is the control type; it provides an exact combobox role and robust keyboard operation without custom interaction code.
- A rejected move shows the server message in an accessible alert; no rollback is needed because the task state was not optimistically changed.

## Open questions

- None. The explicit status values and UI destination options are defined by the backlog and existing task model.
