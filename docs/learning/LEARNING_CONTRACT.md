# LEARNING_CONTRACT

**Goal:** by the end of the pilot you can explain, debug, modify and defend every major BiteCampus feature in a technical interview, without Claude in the room.

## What Claude does
| Before a milestone | During | After |
|---|---|---|
| One-screen plan: problem, approach + one alternative, request/data flow, DB/API changes, failure & security cases, tests, what you'll learn. **Waits for "approved".** | Small steps. Explains each important file in 1–2 lines. Shows real command output. Explains debugging when something fails. | Walkthrough of the real code, edge cases, test results, one improvement, 3–5 interview questions, one teach-back task, a learning-log entry. |

## What you do
1. Read the plan and ask at least one "why?" before approving.
2. Open the main files Claude names and read them yourself (10–15 minutes).
3. Run the tests yourself once: `npm test`.
4. Answer the teach-back question **in your own words** before reading the model answer.
5. Write or approve the learning-log entry.

## Depth 5 for every feature
1. **What** does it do?
2. **How** does the request flow through the code?
3. **Why** this design rather than the alternative?
4. **What breaks?** Retries, concurrency, timeouts, bad input, bad actors.
5. **How is it tested, and how would you improve it?**

## Simplicity rule
If a plan or explanation takes more than one screen, Claude must shorten it. Diagrams and numbered flows beat paragraphs.

## C++/DSA bridges Claude may use
| Web concept | Think of it as |
|---|---|
| Unique constraint | A `std::set` insert that atomically fails on a duplicate |
| Transaction | All-or-nothing block; like a lock-guarded critical section with rollback |
| Idempotency key | Memoisation keyed by request ID: same input → stored output |
| Middleware chain | A pipeline of functions, each can stop or pass the request on |
| Interface + simulator | A pure virtual base class with a mock implementation |
| Index | A B-tree over a column so lookups are O(log n) instead of a scan |
