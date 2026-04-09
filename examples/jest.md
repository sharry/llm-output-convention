# Jest

## Standard output
```
 PASS  src/utils.test.ts
 PASS  src/helpers.test.ts
 PASS  src/format.test.ts
 FAIL  src/auth.test.ts
  ● should reject expired tokens

    expect(received).toBe(expected)

    Expected: 401
    Received: 200

      40 |   const res = await request(app).get('/protected').set('Authorization', expiredToken);
      41 |
    > 42 |   expect(res.status).toBe(401);
         |                      ^
      43 |
      44 | });

      at Object.<anonymous> (src/auth.test.ts:42:22)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

  ● should refresh token on 403

    TypeError: refreshToken is not a function

      13 |   if (response.status === 403) {
      14 |     const newToken = await refreshToken();
    > 15 |     return retry(config, newToken);
         |                         ^
      16 |   }

      at handleResponse (src/api.ts:15:25)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

 FAIL  src/db.test.ts
  ● should rollback on constraint violation

    error: duplicate key value violates unique constraint "users_email_key"

      22 |   await db.query('INSERT INTO users (email) VALUES ($1)', ['dupe@test.com']);
      23 |
    > 24 |   await db.query('INSERT INTO users (email) VALUES ($1)', ['dupe@test.com']);
         |          ^
      25 |

      at Pool.query (node_modules/pg/lib/pool.js:183:19)
      at Object.<anonymous> (src/db.test.ts:24:10)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

  ● should handle connection timeout

    thrown: "Exceeded timeout of 5000 ms for a test.
    Add a timeout value to this test to increase the timeout, if this is a long-running test. See https://jestjs.io/docs/api#testname-fn-timeout."

      30 | test('should handle connection timeout', async () => {
    > 31 |   const pool = createPool({ connectionTimeoutMillis: 1 });
         |                ^
      32 |   await pool.query('SELECT 1');

      at Object.<anonymous> (src/db.test.ts:31:16)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

Test Suites: 2 failed, 3 passed, 5 total
Tests:       4 failed, 48 passed, 52 total
Snapshots:   0 total
Time:        4.217 s
Ran all test suites.
```

## `--llm` output
```
FAIL 4/52

--- src/auth.test.ts:42 "should reject expired tokens"
expected: 401
received: 200

--- src/auth.test.ts:67 "should refresh token on 403"
TypeError: refreshToken is not a function
  at handleResponse (src/api.ts:15)

--- src/db.test.ts:24 "should rollback on constraint violation"
error: duplicate key value violates unique constraint "users_email_key"
  at Pool.query (src/db.test.ts:24)

--- src/db.test.ts:31 "should handle connection timeout"
Exceeded timeout of 5000 ms
  at Object.<anonymous> (src/db.test.ts:31)
```

## Design decisions
| Decision | Rationale |
|---|---|
| No PASS lines | Agent only needs to act on failures |
| No code context windows | Agent can read the source file itself if needed |
| One-level stack traces, no `node_modules` frames | Top user-code frame is almost always sufficient; runtime internals are noise |
| Quoted test name on each failure line | Helps the agent match output to a specific test case |
| `expected`/`received` preserved | Core diagnostic info for assertion failures |
| No summary table (suites, snapshots, time) | A single `FAIL 4/52` line conveys the same signal in fewer tokens |
| Timeout message stripped to essentials | The Jest docs link and suggestion text add nothing for an LLM |
