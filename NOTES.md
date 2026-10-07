# Patch Notes

## Summary of Changes
- **SQL Operator Precedence**: Fixed missing parentheses around `LOWER(title) LIKE` and `LOWER(description) LIKE` in `TaskRepository.java`, `db/queries/search_tasks.sql`, and `db/oracle/task_search_package.sql`. This prevents archived tasks from leaking and ensures status filtering works correctly.
- **Backend Latency & Robustness**: Removed artificial `Thread.sleep` delay in `TaskController.java`, added try-catch validation for `TaskStatus` enum parsing to avoid HTTP 500s on bad inputs, and clamped page index parameters (`safePage >= 1`) to prevent subList out-of-bounds crashes.
- **Frontend State & Race Conditions**: Updated `App.jsx` to reset `page` state to `1` when search query or status filter changes. Updated `useTasks.js` with an `isCurrent` effect cancellation flag to prevent out-of-order network response race conditions, and ensured `setLoading(false)` always runs on errors.

## What Was Not Changed & Why
- **In-Memory Pagination**: Kept in-memory subList pagination in Java instead of refactoring to Spring Data `Pageable` database pagination to maintain small patch scope without altering repository method contracts.
- **Oracle PL/SQL Execution**: Maintained as reference SQL without adding test harnesses, updating only the core query logic.

## Biggest Remaining Risk
- **Unbounded Memory & Performance**: Fetching all tasks into JVM memory (`taskRepository.searchTasks`) before paginating will cause high memory consumption and latency degradation as task row count grows in production. Full database-level SQL pagination (`LIMIT`/`OFFSET`) should be implemented next.

## Tools & AI Used
- **Antigravity (Claude)**: Used to analyze query precedence, trace React hook state lifecycles, and generate regression fix patches.
