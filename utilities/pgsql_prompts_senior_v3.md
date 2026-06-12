# PostgreSQL / Aurora — Senior DB Engineer Prompt Toolkit
### Ground-up rewrite. Production-first. No academic fluff.

---

## HOW TO READ THIS

Every prompt is structured the same way:
- **Fill in** the `[bracketed]` fields — skipping them produces generic output
- **Checkboxes** are signals to the model, not decoration — check the ones that apply
- **"Don't generate"** sections exist because models default to bad habits — be explicit
- **Org constraints** sections are where real projects live; textbook answers often don't apply

The prompts deliberately ask for decisions, not just code. If you find yourself writing
"just give me the function", you're skipping the thinking that prevents 2am incidents.

---
---

## SECTION A — AUTHORING NEW OBJECTS

---

### A1. NEW FUNCTION

```
Write a PostgreSQL function. I'll give you everything you need — don't fill in gaps
with assumptions; ask if anything is ambiguous before writing code.

━━━ IDENTITY ━━━
Schema    : [schema]
Name      : [fn_name]
Purpose   : [One precise sentence. Not "processes data" — what data, what outcome, for whom]

━━━ SIGNATURE ━━━
Parameters (in order):
  [name]  [type]  [NOT NULL | DEFAULT value | OUT | INOUT]
  [name]  [type]  ...

Returns:
  [ ] Scalar    : [type]
  [ ] TABLE(col1 type, col2 type, ...)
  [ ] SETOF     : [composite type or table name]
  [ ] void

━━━ BEHAVIOR CONTRACT ━━━
Volatility — pick one and justify it in a comment:
  [ ] IMMUTABLE  — no DB access, same args always same result (safe for index expressions)
  [ ] STABLE     — reads DB, same args same result within a transaction (safe for planner optimization)
  [ ] VOLATILE   — default; writes, sequences, or non-deterministic reads

Parallel safety — pick one (wrong choice silently breaks parallel query plans):
  [ ] PARALLEL SAFE        — no shared mutable state, safe to run in parallel workers
  [ ] PARALLEL RESTRICTED  — can run in parallel but only in leader process
  [ ] PARALLEL UNSAFE      — default; cannot run in parallel plans at all

Security:
  [ ] SECURITY INVOKER   — default; function runs as the calling user
  [ ] SECURITY DEFINER   — runs as owner; MUST include SET search_path = [schema], pg_catalog
                           inside function body or it's a privilege escalation vector

Language: [ ] sql  [ ] plpgsql  [ ] other: [name]

━━━ LOGIC ━━━
[Describe the logic step by step. Don't make the model guess from vague hints.]
Step 1: [...]
Step 2: [...]
Step 3: [...]

NULL contract:
  [What should happen when each nullable param is NULL? Return NULL? Return empty set?
   Use a default? Raise? Be explicit — "COALESCE to X" or "RETURN NULL" or "RAISE"]

Business rules / constraints that aren't obvious from schema:
  [e.g., "Fiscal year starts April 1st", "deleted_at IS NULL means active",
   "currency must be checked before arithmetic — mixed currency rows exist"]

━━━ DATA CONTEXT ━━━
Tables touched (read): [schema.table — approximate row count — partitioned? y/n]
Tables touched (write): [schema.table — if any]
Indexes that exist and should be exploited: [index name: columns, type]
Sequences or serials consumed: [if any — matters for re-run safety]

━━━ GENERATE THIS ━━━
1. Function DDL with $$ dollar-quoting
2. DECLARE block with all variables typed and initialized — no mid-body declarations
3. Header comment block inside the function:
     /*
      * [fn_name]
      * Purpose      : [one line]
      * Returns      : [describe]
      * Depends on   : [tables, functions, types]
      * Gotchas      : [anything non-obvious]
      * Author       : [placeholder]
      * Created      : [YYYY-MM-DD]
      * Version      : 1.0
      */
4. GRANT EXECUTE ON FUNCTION [schema].[fn_name]([types]) TO [role];
5. COMMENT ON FUNCTION DDL

━━━ DO NOT ━━━
- Do not use RETURN NEXT in a loop to build a result set — use RETURN QUERY
- Do not use SELECT INTO for scalar assignment — use := or perform GET DIAGNOSTICS
- Do not mark IMMUTABLE unless the body has zero database access
- Do not write dynamic SQL unless I've said it's required
- Do not use bare WHEN OTHERS in EXCEPTION blocks — catch specific SQLSTATE codes
- Do not use RAISE EXCEPTION for expected business conditions (e.g., "no rows found")
  — return NULL or empty set; let the caller decide what that means
- Do not hardcode schema names anywhere except in SET search_path
```

---

### A2. NEW STORED PROCEDURE

```
Write a PostgreSQL stored procedure. This is going to production.
Treat every design choice as a decision that needs a justification comment in the code.

━━━ IDENTITY ━━━
Schema    : [schema]
Name      : [proc_name]
Purpose   : [One precise sentence — what load/operation, source, target, when it runs]
Caller    : [Airflow / Lambda / application / cron / manual psql]
Frequency : [every N min / daily / event-triggered]

━━━ TRANSACTION DESIGN ━━━
This is the most important decision. Pick one and understand the consequences:

[ ] SINGLE TRANSACTION (caller manages commit)
    — Safest for correctness. All-or-nothing. Locks held for full duration.
    — Use when: volume is manageable (< ~5M rows), runtime < a few minutes,
      lock contention is not a concern.
    — Procedure must NOT contain COMMIT/ROLLBACK.

[ ] AUTONOMOUS COMMITS (procedure commits mid-work)
    — Required for large loads or when you need checkpoint recovery.
    — Partial success is possible. On failure, some data is already committed.
    — Requires explicit checkpoint design: log progress to a resumption table
      so re-run can skip already-committed batches.
    — Caller cannot wrap this in a transaction that rolls everything back.
    — Use when: volume or lock duration makes single-transaction impractical.

[ ] BATCH LOOP WITH COMMIT
    — Process N rows, COMMIT, loop. Reduces lock hold time.
    — Requires keyset pagination (NOT OFFSET) for large tables.
    — Document the batch size choice and why.

Chosen: [ ] with justification: [...]

━━━ PARAMETERS ━━━
  p_batch_id     BIGINT                  — ties this run to an audit record
  p_run_date     DATE                    — explicit; never use CURRENT_DATE inside logic
  p_dry_run      BOOLEAN  DEFAULT FALSE  — run all logic, skip final COMMIT / DML
  p_debug        BOOLEAN  DEFAULT FALSE  — emit RAISE NOTICE at each significant step
  [others]       [type]   [default]

━━━ SOURCE / TARGET ━━━
Source:
  Object     : [schema.table or VIEW or CTE — explain if it's a staging area]
  Grain      : [one row per what]
  Volume     : [~N rows per run, ~N total rows]
  Key cols   : [watermark col / business key / change indicator]
  Known data quality issues: [nulls in unexpected places, type mismatches, dupes upstream]

Target:
  Object     : [schema.table]
  PK         : [surrogate key — type and generation strategy]
  Business key: [natural key cols used for matching]
  Partitioned: [yes/no — partition key + type — critical for lock and pruning behavior]
  Indexes    : [list — especially those the load path depends on]

━━━ LOAD PATTERN ━━━
[ ] Full refresh — partition swap preferred (zero downtime); TRUNCATE+reload only if acceptable
[ ] Incremental upsert — INSERT ... ON CONFLICT (business_key) DO UPDATE SET ...
    ON CONFLICT target must exactly match an existing UNIQUE constraint or unique index
[ ] CDC apply — source has op_type ('I','U','D'); apply in order, handle out-of-order risk
[ ] SCD Type 1 — overwrite in place, update metadata cols
[ ] SCD Type 2 — expire current row, insert new; see Section E for full SCD2 prompt
[ ] Delete + reload single partition — safer than TRUNCATE on full table
[ ] Merge via writable CTE — document why not ON CONFLICT

━━━ CHANGE DETECTION STRATEGY ━━━
(For incremental / SCD loads — pick one, state trade-off)
[ ] Watermark (updated_at >= last_run_watermark)
    Trade-off: fast, misses rows if source backfills without updating watermark
[ ] Row hash (MD5 of tracked columns)
    Trade-off: catches all data changes, hash column must be maintained or computed on-the-fly
[ ] Column-by-column IS DISTINCT FROM
    Trade-off: explicit and accurate, verbose, brittle when schema changes
[ ] Full outer join diff
    Trade-off: most accurate, expensive at scale

Chosen: [ ] with trade-off acknowledged in a code comment

━━━ AUDIT LOGGING ━━━
Write to [audit_schema.job_log] — schema assumed to exist:
  proc_name, batch_id, run_date, started_at (clock_timestamp()),
  finished_at, rows_read, rows_inserted, rows_updated, rows_skipped,
  rows_errored, status ('RUNNING'→'SUCCESS'/'PARTIAL'/'FAILED'), error_message

Pattern:
  INSERT audit row with status='RUNNING' at start
  UPDATE same row at end — do not INSERT a second row
  On exception: UPDATE with status='FAILED', error_message=SQLERRM, then RE-RAISE

━━━ ERROR HANDLING CONTRACT ━━━
- Catch specific SQLSTATE codes where recovery is possible
- WHEN OTHERS: log to audit table, then RAISE — never swallow
- If mid-batch failure: document exactly what state is left and how to recover
- Validation failures (bad data): log to [error_schema.load_errors], skip the row,
  continue processing — do not abort the whole batch for a few bad rows
  (unless the prompt specifies zero-tolerance)

━━━ ORG CONSTRAINTS ━━━
[ ] No TRUNCATE on live target tables
[ ] No CREATE TEMP TABLE — use pre-declared staging schema; TRUNCATE at proc start
[ ] No DDL inside procedure
[ ] Connection pooled via PgBouncer (transaction mode) — no session-level SET that must persist,
    no PREPARE, no advisory locks that depend on session lifetime
[ ] All timestamps in UTC — no LOCALTIME, CURRENT_TIMESTAMP is fine (returns UTC on this cluster)
[ ] search_path not guaranteed — fully qualify every object: schema.table, schema.function()
[ ] DBA approval required for new indexes — flag as separate recommendation

━━━ CONCURRENCY ━━━
Can multiple instances run simultaneously?
[ ] No — single worker guaranteed
[ ] Yes, different partitions/date ranges — partition-level advisory lock pattern
[ ] Yes, same data — advisory lock with pg_try_advisory_xact_lock; RETURN early if locked

If yes: show the advisory lock block and the "already running" exit path.

━━━ DO NOT ━━━
- Do not use OFFSET for batch pagination — use keyset (WHERE id > last_id LIMIT N)
- Do not consume sequences inside a dry-run path
- Do not write to target table before validating source data
- Do not hold locks longer than necessary — structure for minimal lock window
- Do not use RAISE NOTICE in production path when p_debug is FALSE
```

---

### A3. NEW TRIGGER

```
Write a PostgreSQL trigger and trigger function for [schema.table_name].

━━━ PURPOSE ━━━
[ ] Audit / change log — capture before/after row image with who/when
[ ] CDC feed — write to changelog table for downstream consumers
[ ] Derived column — keep [col] computed without application changes
[ ] Soft-delete interception — rewrite DELETE as UPDATE set deleted_at = clock_timestamp()
[ ] Business rule enforcement — constraint too complex for CHECK
[ ] Cross-table denormalization — maintain summary/count in another table

━━━ TRIGGER SPECIFICATION ━━━
Fires on   : [ ] INSERT  [ ] UPDATE  [ ] UPDATE OF [col1, col2]  [ ] DELETE  [ ] TRUNCATE
Timing     : [ ] BEFORE  [ ] AFTER   [ ] INSTEAD OF (views only)
Level      : [ ] FOR EACH ROW  [ ] FOR EACH STATEMENT
Condition  : WHEN ([expression]) — leave blank if always fires

━━━ TIMESTAMP DECISION ━━━
For event timestamps in audit/CDC tables, use clock_timestamp(), NOT now() or CURRENT_TIMESTAMP.
Reason: now() returns transaction start time — all rows modified in one transaction
        get the same timestamp, which destroys event ordering in audit logs.
        clock_timestamp() returns actual wall-clock time at moment of each row change.

━━━ AUDIT TABLE DDL (generate this if audit trigger) ━━━
CREATE TABLE [audit_schema.table_name_log] (
  audit_id      BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  op            CHAR(1)      NOT NULL CHECK (op IN ('I','U','D')),
  table_name    TEXT         NOT NULL,
  schema_name   TEXT         NOT NULL,
  row_id        [pk_type],                    -- copy of PK for fast lookup
  changed_by    TEXT         NOT NULL DEFAULT current_user,
  app_user      TEXT,                         -- from current_setting('app.current_user', true)
  changed_at    TIMESTAMPTZ  NOT NULL DEFAULT clock_timestamp(),
  client_addr   INET         DEFAULT inet_client_addr(),
  old_data      JSONB,                        -- NULL for INSERT
  new_data      JSONB,                        -- NULL for DELETE
  changed_cols  TEXT[]       -- UPDATE only: list of cols that actually changed
) -- consider UNLOGGED if audit durability can be traded for write performance

━━━ CHANGED COLUMNS (UPDATE triggers) ━━━
For UPDATE: populate changed_cols with only columns whose value changed:
  SELECT array_agg(key)
  FROM jsonb_each(to_jsonb(NEW)) n
  JOIN jsonb_each(to_jsonb(OLD)) o USING (key)
  WHERE n.value IS DISTINCT FROM o.value

━━━ RECURSIVE TRIGGER GUARD ━━━
If the trigger function writes to a table that also has this trigger (or any trigger
that could cycle back), add a guard:
  IF current_setting('app.in_trigger', true) = 'true' THEN RETURN NEW; END IF;
  PERFORM set_config('app.in_trigger', 'true', true); -- true = resets at txn end
  -- ... trigger body ...
  PERFORM set_config('app.in_trigger', 'false', true);

━━━ TRUNCATE GAP ━━━
TRUNCATE is not captured by row-level triggers.
Add a FOR EACH STATEMENT trigger on TRUNCATE that logs a synthetic 'T' op with row count
from pg_class.reltuples — or document this gap explicitly in a comment.

━━━ NEW/OLD NULL RULES ━━━
Always handle:
  INSERT : OLD is NULL — never reference OLD columns without NULL check
  DELETE : NEW is NULL — never reference NEW columns without NULL check
  UPDATE : Both exist — use IS DISTINCT FROM for comparison, not !=

━━━ PERFORMANCE NOTES ━━━
- Trigger fires per row by default — at 10k rows/sec this is 10k function calls/sec
- Benchmark the trigger overhead on bulk loads; document it
- For bulk loads where audit is acceptable to skip: provide a session-level toggle
  IF current_setting('app.skip_audit', true) = 'true' THEN RETURN NEW; END IF;
- UNLOGGED audit tables write ~30% faster — weigh durability vs performance

━━━ GENERATE ━━━
1. Trigger function DDL (CREATE OR REPLACE FUNCTION ... RETURNS TRIGGER)
2. CREATE TRIGGER DDL (all options explicit — no defaults left implicit)
3. Audit table DDL (if applicable)
4. DROP TRIGGER / DROP FUNCTION rollback block (commented out at bottom)
5. Smoke test: INSERT + UPDATE + DELETE against table, SELECT from audit to verify
```

---
---

## SECTION B — REVERSE ENGINEERING EXISTING CODE

---

### B1. UNDERSTAND WHAT THIS CODE ACTUALLY DOES

```
I have a PostgreSQL function/procedure with no documentation. Tell me what it actually does.

━━━ WHAT I NEED ━━━
Not a line-by-line walkthrough. Give me:
1. One paragraph: what is the real-world purpose of this code?
2. Data flow: what tables does it read, what does it write, in what order?
3. The "happy path" in plain English
4. Every branch condition and what it leads to
5. What inputs would cause it to return nothing / null / raise?
6. What upstream state does it assume exists? (staging table populated, sequence seeded, etc.)
7. What downstream code probably calls this and expects from it?
8. Three things that will break this in production that aren't obvious from reading it

━━━ THEN GENERATE ━━━
A COMMENT ON FUNCTION / COMMENT ON PROCEDURE block ready to apply:
  COMMENT ON FUNCTION [schema].[name]([types]) IS
  '...' ;

And a header block to insert at top of function body.

[PASTE CODE]
```

---

### B2. DEPENDENCY MAP

```
Map all dependencies of this PostgreSQL object.

I need to know before I change it what else will break.

━━━ GENERATE ━━━
1. Objects this code depends on:
   - Tables read (schema.name, approximate rows, partitioned?)
   - Tables written (schema.name)
   - Functions/procedures called (schema.name, signature)
   - Sequences used
   - Types / domains referenced
   - Extensions used (e.g., uuid-ossp, pgcrypto)
   - Config settings read via current_setting()

2. Objects that depend on this code (I'll run this separately, but note what to check):
   SELECT p.proname, n.nspname
   FROM pg_proc p
   JOIN pg_namespace n ON p.pronamespace = n.oid
   JOIN pg_depend d ON d.objid = p.oid
   WHERE d.refobjid = '[schema].[fn_name]'::regproc;

3. Breaking change risk:
   - If I change the signature: [what breaks]
   - If I change the return type: [what breaks]
   - If I change the volatility: [what breaks — planner behavior]
   - If I rename it: [what breaks]

4. Safe-change checklist:
   [ ] Can I use CREATE OR REPLACE (signature/return type unchanged)?
   [ ] Do I need DROP + recreate?
   [ ] Do I need a deprecation path (keep old, add new, migrate callers)?

[PASTE CODE]
```

---
---

## SECTION C — CODE REVIEW

---

### C1. SENIOR CODE REVIEW — THE FULL AUDIT

```
Do a production-readiness review of this PostgreSQL code.
This is not a style review. Find things that will fail, corrupt data, or cause incidents.

━━━ CONTEXT ━━━
PostgreSQL / Aurora version : [e.g., Aurora PostgreSQL 15.4]
Called by                   : [Airflow / application / dbt / cron / manual]
Table sizes involved        : [approximate row counts for each table touched]
Existing indexes            : [list, or say "unknown"]
Connection pooling          : [PgBouncer / RDS Proxy / direct]
Transaction isolation       : [READ COMMITTED (default) / REPEATABLE READ / SERIALIZABLE]

━━━ REVIEW DIMENSIONS — ADDRESS EVERY ONE ━━━

[1. CORRECTNESS]
For each of these, find the actual line(s) where the issue exists or confirm it doesn't:

□ NULL propagation
  — Every expression touching a nullable column: what happens when it's NULL?
  — Is a NULL result silently returned where a non-NULL is expected downstream?
  — NOT IN (subquery): does the subquery include rows where the IN column is NULL?
    If yes, the entire NOT IN returns zero rows — always. Confirm or flag.

□ Type coercion
  — Any implicit cast in WHERE, JOIN ON, or ON CONFLICT target?
  — Does any column get cast to match a literal rather than the literal cast to match the column?
    This invalidates index use silently.

□ Date/time arithmetic
  — Timezone assumptions: are timestamps cast to DATE using server timezone? Is that intentional?
  — BETWEEN for date ranges: BETWEEN is inclusive on both ends. Is that the correct semantic?
  — DST boundary risk: does this run across a DST transition? What happens?

□ Aggregation
  — GROUP BY with a nullable key: NULLs group together — is that correct here?
  — COUNT(*) vs COUNT(col): using COUNT(*) where only non-null rows should count?
  — DISTINCT as a band-aid: hiding a cartesian product or bad join?

□ JOIN correctness
  — Every LEFT JOIN: is the nullable side handled downstream or is it silently NULL-spreading?
  — Any join on a column where the type on one side is text and the other is int/uuid?
    Implicit cast prevents index use and can silently return wrong results.

□ Cursor / loop logic
  — FOUND flag: checked immediately after the query that sets it? (A subsequent query resets it)
  — Loop EXIT: is there a guaranteed exit condition, or can this run forever?
  — Row-by-row logic that could be a single set-based statement?

□ EXCEPTION block correctness
  — SQLSTATE codes caught: are they the right ones for the expected error?
  — WHEN OTHERS: does it log before re-raising, or silently eat the error?
  — After catching and handling an error, is the database in a consistent state?
  — Nested BEGIN/EXCEPTION: does an inner catch prevent the outer transaction from aborting
    when it should?

[2. IDEMPOTENCY & RE-RUN SAFETY]
□ Can this be safely re-run after a failure without creating duplicates or incorrect state?
□ Does it write data before validating inputs? (Should validate first, write last)
□ Are sequences consumed even in failure paths or dry-run mode?
□ Is there a race condition if two workers run this simultaneously?
  — Missing SELECT FOR UPDATE?
  — No advisory lock on the batch key?
  — ON CONFLICT target — does it match an actual UNIQUE constraint/index?

[3. PERFORMANCE] (flag each as LOW / MEDIUM / HIGH / BLOCKING)
□ Any function call wrapping a column in WHERE or JOIN ON?
  e.g., WHERE LOWER(email) = ... or WHERE DATE_TRUNC('day', created_at) = ...
  These prevent index use. The fix is an expression index or rewriting the predicate.

□ CTE materialization behavior
  — Pre-PG12: all CTEs are materialized (optimization fence) by default
  — PG12+: CTEs may or may not be materialized; is MATERIALIZED / NOT MATERIALIZED explicit?
  — CTEs that are referenced more than once: should they be MATERIALIZED to avoid re-execution?

□ Implicit type casts in join conditions or ON CONFLICT targets
□ DISTINCT in subqueries that could be EXISTS
□ Unnecessary ORDER BY inside CTEs, subqueries, or views that feed another query
□ Row-by-row cursor loop doing what a single INSERT/UPDATE/DELETE could do
□ Temp table without an index being joined to a large table
□ OFFSET-based pagination on a large table (use keyset instead)
□ Unbounded result set risk — is there any code path with no LIMIT?

[4. SECURITY]
□ SECURITY DEFINER without explicit SET search_path in body
  — Attacker creates a function with same name in their schema, gets elevated execution
  — Fix: SET search_path = [specific_schema], pg_catalog inside function body

□ Dynamic SQL injection surface
  — Any EXECUTE with concatenated strings rather than USING clause?
  — format() usage: %I for identifiers (quote_ident), %L for literals, %s is unsafe for user input
  — User-supplied identifiers (table names, column names): validated against pg_catalog before use?

□ Data exposure
  — Does SECURITY DEFINER give callers access to data their role cannot see directly?
  — Is that intentional and documented?

[5. OPERATIONAL]
□ Long transaction risk
  — How long does the longest realistic run hold locks?
  — Will it block autovacuum? (Autovacuum blocked by long transactions causes table bloat)
  — Is there a statement_timeout or lock_timeout set at the session level?

□ Logging completeness
  — Can an on-call engineer reconstruct what happened from the logs alone?
  — Are row counts captured (inserted / updated / skipped / errored)?
  — Is the batch ID or run timestamp in every log line?

□ Temp object cleanup
  — Temp tables, advisory locks, session settings: all cleaned up on both success and failure paths?
  — In a pooled connection, session state leaks to the next user of the connection.

□ Timeout risks
  — Any query that could run much longer under data skew or unexpected volume?
  — Is there a fallback or timeout handling?

━━━ OUTPUT FORMAT ━━━
1. Issue register:
   | # | Severity | Category | Location (line/block) | Description | Recommended Fix |

   Severity: CRITICAL (data corruption / security) / HIGH (production failure risk) /
             MEDIUM (performance degradation) / LOW (code quality / maintainability)

2. Fixed code: only the changed blocks, clearly marked with -- BEFORE / -- AFTER comments
3. "Merge this safely" checklist: what to verify before deploying this to production

[PASTE CODE]
```

---
---

## SECTION D — DEBUGGING

---

### D1. ROOT CAUSE ANALYSIS — NOT A GUESS LIST

```
This PostgreSQL code is broken. I need a diagnosis, not a list of things to try.

━━━ THE FAILURE ━━━
Error text (copy verbatim — every line including CONTEXT and DETAIL):
  [
  ERROR:  ...
  DETAIL: ...
  CONTEXT: PL/pgSQL function fn_name(uuid,date) line 47 at SQL statement
  ]

When it fails:
  [ ] Always, with any input
  [ ] Only with specific input values — those values: [describe the pattern]
  [ ] Only under concurrent load (N+ simultaneous callers)
  [ ] Intermittently — no clear pattern yet
  [ ] Worked before, broke after: [what changed — deploy? data volume? schema change?]

━━━ ENVIRONMENT ━━━
Version          : Aurora PostgreSQL [version] / RDS PostgreSQL [version]
Connection pool  : [PgBouncer / RDS Proxy / direct / pgpool]
  Pool mode      : [session / transaction / statement] — critical for temp tables and locks
Caller           : [language + driver: e.g., Python psycopg2, Java JDBC, dbt, Airflow]
Isolation level  : [READ COMMITTED / REPEATABLE READ / SERIALIZABLE]

━━━ DATA CONTEXT ━━━
Table sizes      : [schema.table: ~N rows, partitioned by X]
Known data quirks: [e.g., "customer_id is stored as TEXT but the column is UUID",
                    "some rows have NULL in updated_at despite NOT NULL constraint — legacy"]
Recent changes   : [schema migration? statistics reset? data volume spike? pg_upgrade?]

━━━ WHAT I'VE ALREADY RULED OUT ━━━
[e.g., "Not a permissions issue — confirmed same role works on dev",
 "Not a data type mismatch — checked pg_typeof() on both sides of the join",
 "Happens even with single-row input"]

━━━ GIVE ME ━━━
1. MOST LIKELY ROOT CAUSE — one specific diagnosis, not "it could be A or B"
   with the reasoning that leads to that conclusion from the error text and context

2. SECOND MOST LIKELY — only if the primary is genuinely ambiguous
   with what would distinguish between them (diagnostic query or test)

3. THE FIX — minimal change, just the broken section, before/after

4. VERIFICATION — one RAISE NOTICE or diagnostic SELECT I can run in staging to
   confirm the fix before promoting to production

5. IF IT'S A DATA ISSUE not a code issue:
   The query to find and count the bad rows causing this

6. OTHER LANDMINES — issues in this code that aren't causing this error but will
   cause the next incident. Flag them, don't fix them here.

[PASTE FULL CODE]
```

---

### D2. "WORKS ON DEV, FAILS ON PROD" — ENVIRONMENT DIFF

```
This PostgreSQL code works correctly in dev/staging but fails or produces wrong results
in production. Help me isolate what's different.

━━━ BEHAVIOR DIFFERENCE ━━━
Dev result   : [what it returns / how long it takes]
Prod result  : [what it returns / how long it takes / what error]

━━━ KNOWN DIFFERENCES TO CHECK ━━━
Work through each of these and tell me which ones are likely culprits:

[DATA]
□ Row counts: dev [~N rows] vs prod [~N rows] — volume difference changes query plans
□ Data distribution: dev data is clean/uniform; prod has skew, NULLs, edge cases
□ Statistics currency: when was ANALYZE last run on affected tables in each env?
  Query: SELECT relname, last_analyze, n_live_tup FROM pg_stat_user_tables WHERE relname = '[table]';
□ Partition count: same partition structure? Prod may have more/older partitions.

[CONFIGURATION]
□ work_mem: what is it set to in each env?
  SELECT current_setting('work_mem');
□ max_parallel_workers_per_gather: parallel query enabled in prod?
□ enable_seqscan / enable_hashjoin: any planner flags toggled differently?
□ search_path: is it set the same way? Missing schema in search_path is silent in dev,
  breaks in prod if schemas differ.

[CONNECTION / SESSION]
□ Connection pool type: dev direct vs prod PgBouncer transaction mode?
  — Transaction-mode pool resets session state between calls
  — Temp tables from a previous call may or may not exist
  — SET commands don't persist in transaction-mode pool
□ Transaction isolation level: same in both environments?
□ lock_timeout / statement_timeout: set differently in prod?

[SCHEMA]
□ Are column types identical? Check pg_typeof() on joined/filtered columns.
□ Are indexes identical? An index that exists in dev may not exist in prod, or vice versa.
□ Any view or function dependency that resolves to a different object in each env?
  (search_path issue — different schema objects with the same name)

━━━ DIAGNOSTIC QUERIES TO RUN IN BOTH ENVIRONMENTS ━━━
Generate:
1. EXPLAIN (ANALYZE, BUFFERS) for the slow/failing query — compare plans between envs
2. Table statistics comparison query (n_live_tup, last_analyze, correlation)
3. Index existence check for all indexes used by this query
4. current_setting() for all relevant GUCs

[PASTE CODE]
```

---
---

## SECTION E — DATA LOAD PATTERNS

---

### E1. INCREMENTAL LOAD WITH WATERMARK

```
Write an incremental load procedure using a watermark strategy.

━━━ PATTERN CONTEXT ━━━
This is the most common incremental pattern and also the most commonly broken one.
The key failure mode: source rows are updated without bumping the watermark column.
Document this limitation explicitly in the procedure header.

━━━ CONFIGURATION ━━━
Source          : [schema.table] — [~N rows total, ~N rows change per run]
Target          : [schema.table]
Watermark col   : [col_name, data type — e.g., updated_at TIMESTAMPTZ]
Watermark store : [schema.watermark_table (col: proc_name, last_watermark) or p_from_ts param]
Business key    : [cols used to match source to target for upsert]
Batch size      : [N rows per commit cycle, or 0 for single transaction]

━━━ WATERMARK LOGIC ━━━
Last run watermark retrieval:
  SELECT last_watermark INTO v_from_ts
  FROM [schema.watermark_table]
  WHERE proc_name = '[proc_name]'
  FOR UPDATE;  -- lock it so concurrent runs don't read stale value

New watermark capture (do this BEFORE querying source, not after):
  v_to_ts := clock_timestamp();
  -- Reason: if captured after query, rows inserted during the query window are missed

Source query filter:
  WHERE [watermark_col] > v_from_ts
  AND   [watermark_col] <= v_to_ts
  -- Use >= / <= or > / <= depending on whether last_watermark row was already processed
  -- Document the boundary decision in a comment; wrong boundary = duplicates or gaps

━━━ UPSERT PATTERN ━━━
INSERT INTO [target] (...)
SELECT ... FROM [source_cte]
ON CONFLICT ([business_key_cols]) DO UPDATE SET
  [col1] = EXCLUDED.[col1],
  [col2] = EXCLUDED.[col2],
  updated_at = EXCLUDED.updated_at
WHERE [target].[hash_col] IS DISTINCT FROM EXCLUDED.[hash_col];
-- The WHERE clause on DO UPDATE prevents no-op updates (same data, no change)
-- This matters for row versioning, triggers, and replication volume

━━━ KNOWN LIMITATIONS — PUT IN HEADER COMMENT ━━━
1. Rows updated in source WITHOUT bumping [watermark_col] will never be picked up
2. Rows deleted from source are NOT propagated (this is incremental, not CDC)
3. If the procedure fails after v_to_ts is captured but before watermark is saved,
   next run will re-process some rows — ON CONFLICT ensures this is safe (idempotent)
4. Clock skew between source DB and this DB can cause narrow gaps — add a [N second]
   overlap buffer: v_from_ts := v_from_ts - INTERVAL '[N] seconds'
   Document the buffer value and why it was chosen.

━━━ INCLUDE ━━━
1. Full procedure with all parameters
2. Watermark table DDL (CREATE TABLE IF NOT EXISTS)
3. The overlap buffer logic with a configurable parameter
4. Row count tracking: read, inserted, updated, skipped, errored
5. Audit log INSERT/UPDATE pattern
6. Dry-run path: compute counts, ROLLBACK, report — do not advance watermark
```

---

### E2. SCD TYPE 2 — EVERY EDGE CASE

```
Write a production SCD Type 2 implementation. This pattern is broken in most codebases.
Address every edge case explicitly — don't paper over them.

━━━ TABLE DESIGN ━━━
Dimension table: [schema.dim_table]
  dim_key         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
  [business_key_col1]  [type] NOT NULL
  [business_key_col2]  [type] -- if composite key
  [tracked_col1]  [type]  -- changes to these cols trigger a new version
  [tracked_col2]  [type]
  [overwrite_col1] [type] -- changes here update in-place (no new version)
  effective_from  TIMESTAMPTZ NOT NULL DEFAULT now()
  effective_to    TIMESTAMPTZ NOT NULL DEFAULT '9999-12-31 23:59:59.999999+00'
  is_current      BOOLEAN NOT NULL DEFAULT TRUE
  record_hash     TEXT NOT NULL  -- MD5 of tracked columns only
  load_batch_id   BIGINT         -- traceability

REQUIRED CONSTRAINT (generate the DDL):
  CREATE UNIQUE INDEX uix_[dim_table]_current
  ON [schema.dim_table] ([business_key_col1] [, business_key_col2])
  WHERE is_current = TRUE;
  -- This is the most important thing in an SCD2 schema. Without it, nothing prevents
  -- two current rows for the same business key. Queries will silently double-count.

━━━ RECORD HASH ━━━
Use MD5 on tracked columns only (not overwrite cols, not metadata cols):
  MD5(
    COALESCE([tracked_col1]::text, '') || chr(30) ||
    COALESCE([tracked_col2]::text, '') || chr(30) ||
    ...
  )
  -- Use chr(30) (ASCII Record Separator) as delimiter, not '|'
  -- Reason: '|' can appear in data; chr(30) never does in normal text
  -- Document which columns are included in the hash in a code comment

━━━ EDGE CASES — HANDLE ALL OF THEM ━━━

[CASE 1: New record]
Business key not in dimension at all.
→ INSERT as is_current=TRUE, effective_from=v_batch_ts

[CASE 2: No change]
Record exists, record_hash matches.
→ No-op. Do not create a new version. Update overwrite_col1 in-place if changed.
→ Count as "skipped" in audit log.

[CASE 3: Tracked column changed]
Record exists, record_hash differs.
→ Expire current row: UPDATE SET is_current=FALSE, effective_to=v_batch_ts - INTERVAL '1 microsecond'
→ Insert new current row: effective_from=v_batch_ts
→ Do these atomically in a single CTE statement to prevent a window where no current row exists.

[CASE 4: Late-arriving record]
The source row has a change timestamp earlier than the current row's effective_from.
(Data arrived out of order.)
→ Decision required — pick one:
  [ ] Ignore: process as a normal current-record update (lose historical accuracy)
  [ ] Insert into history gap: find the correct position and insert with correct effective dates
      (complex, requires re-sorting history; document the approach chosen)
  [ ] Flag and skip: log to error table, human review required
  State your choice and the business justification in the header comment.

[CASE 5: Delete in source]
Business key disappears from source (hard delete or logical delete upstream).
→ Decision required — pick one:
  [ ] Expire the current row (effective_to = v_batch_ts, is_current = FALSE)
  [ ] Set a is_deleted flag on the current row (keeps it current but marked deleted)
  [ ] Do nothing (ignore source deletes in the dimension)
  State your choice and the business justification.

[CASE 6: Multiple changes for same business key in same batch]
Source has 3 rows for customer_id=123 in this load file, each with a different address.
→ Must process in order of change timestamp, not source row order
→ Only the last version becomes the new is_current row
→ Intermediate versions are inserted as expired historical rows
→ Use ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY change_ts) to sequence them

[CASE 7: Re-run safety]
The procedure runs twice with the same batch_id and data.
→ Result must be identical to a single run. No duplicate rows.
→ Mechanism: skip INSERT if (business_key, effective_from, record_hash) already exists

━━━ ATOMIC EXPIRY + INSERT ━━━
Use a single CTE-based DML statement:
  WITH
  to_expire AS (
    SELECT dim_key FROM [schema.dim_table]
    WHERE is_current = TRUE
    AND [business_key_col] IN (SELECT [business_key_col] FROM changed_records)
    AND record_hash != (SELECT record_hash FROM changed_records
                        WHERE [business_key_col] = [dim_table].[business_key_col])
  ),
  expired AS (
    UPDATE [schema.dim_table]
    SET is_current = FALSE,
        effective_to = v_batch_ts - INTERVAL '1 microsecond'
    WHERE dim_key IN (SELECT dim_key FROM to_expire)
    RETURNING [business_key_col]
  )
  INSERT INTO [schema.dim_table] (...)
  SELECT ... FROM changed_records
  WHERE [business_key_col] IN (SELECT [business_key_col] FROM expired);

-- This is preferred over two separate statements because it eliminates the window
-- between expiry and insert where a concurrent reader sees no current row.

━━━ POST-LOAD INTEGRITY CHECK ━━━
Generate queries to run after each load to verify SCD2 health:
  -- Duplicate current rows (should return zero):
  SELECT [business_key_col], COUNT(*) FROM [schema.dim_table]
  WHERE is_current = TRUE GROUP BY [business_key_col] HAVING COUNT(*) > 1;

  -- Overlapping date ranges (should return zero):
  SELECT a.dim_key, b.dim_key FROM [schema.dim_table] a
  JOIN [schema.dim_table] b
    ON a.[business_key_col] = b.[business_key_col]
    AND a.dim_key != b.dim_key
    AND a.effective_from < b.effective_to
    AND a.effective_to > b.effective_from;

  -- Gaps in history (no row covering a specific date):
  -- [generate as parameterized query with p_check_date DATE]
```

---

### E3. CDC APPLY — PROCESSING CHANGE EVENTS

```
Write a procedure to apply CDC change events from a changes table to a target table.

━━━ CONTEXT ━━━
CDC source  : DMS / Debezium / custom / logical replication
Changes table: [schema.cdc_changes] containing:
  op_type      CHAR(1)      'I' = insert, 'U' = update, 'D' = delete
  captured_at  TIMESTAMPTZ  source database timestamp
  received_at  TIMESTAMPTZ  when row landed in changes table
  lsn          BIGINT       log sequence number (ordering)
  payload      JSONB        full row image (after for I/U, before for D)
  processed    BOOLEAN      DEFAULT FALSE

Target table: [schema.target_table]

━━━ ORDERING CRITICAL ━━━
Process events in LSN order, not received_at order.
Reason: rows can arrive out of order due to replication lag.
  ORDER BY lsn ASC — not captured_at, not received_at

━━━ IDEMPOTENCY ━━━
Each event must be processable multiple times with same result.
Mark processed=TRUE after applying. Check processed=FALSE before selecting.
If the same LSN is processed twice (due to retry), it should be a no-op.

━━━ OPERATION HANDLING ━━━
'I' — INSERT ... ON CONFLICT ([pk]) DO NOTHING
      (row may already exist if a previous partial run inserted it)

'U' — UPDATE ... SET [cols from payload] WHERE [pk] = [payload pk]
      Check that target row exists; if not, convert to INSERT (late-arriving insert)
      Document the out-of-order handling decision.

'D' — Decision required:
  [ ] Hard DELETE from target
  [ ] Soft delete: UPDATE SET deleted_at = captured_at, is_active = FALSE
  [ ] Insert a tombstone record
  State the choice and its implications for downstream queries.

━━━ ERROR HANDLING FOR BAD EVENTS ━━━
Some events will have:
  - Malformed JSON in payload
  - References to rows already deleted (for updates)
  - Type mismatch between payload and current table schema

Do NOT abort the whole batch for a bad event.
→ Log to [error_schema.cdc_errors]: lsn, op_type, payload, error_message, failed_at
→ Mark the event as processed=TRUE (or processed_error=TRUE)
→ Continue with next event
→ Report error count in audit log
→ Raise NOTICE if p_debug=TRUE

━━━ SCHEMA DRIFT RISK ━━━
CDC payload reflects the source schema at capture time.
Target schema may have drifted (columns added/removed).
→ Use payload->'column_name' with NULL default for columns that may not exist in older events
→ Do NOT use SELECT * from payload — extract columns explicitly
→ Document schema version assumption in header comment

━━━ INCLUDE ━━━
1. Full procedure
2. Batch size control (p_batch_size INT DEFAULT 10000)
3. Maximum LSN per batch (to limit blast radius on large backlogs)
4. Metrics: events_applied, events_skipped, events_errored per op_type
5. A query to check backlog depth: unprocessed event count and oldest unprocessed LSN
```

---
---

## SECTION F — PERFORMANCE

---

### F1. EXPLAIN PLAN ANALYSIS & QUERY REWRITE

```
Analyze this PostgreSQL EXPLAIN plan and rewrite the query.

━━━ PERFORMANCE PROFILE ━━━
Current runtime     : [e.g., 3m 20s]
Target runtime      : [e.g., under 20s]
Runs                : [frequency — affects whether caching helps]
Was it fast before? : [yes/no — if yes, what changed?]

━━━ EXPLAIN OUTPUT ━━━
[Paste EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) output]
If too long, paste:
  — Root node and its actual vs estimated rows
  — Any node where actual rows >> estimated rows (statistics problem)
  — Any Hash Join with "Batches: N" where N > 1 (hash join spilled to disk)
  — Any Sort node with "Sort Method: external merge" (sort spilled to disk)
  — Any node with "Rows Removed by Filter: N" where N is large (bad index selectivity)
  — The single slowest node by actual time

━━━ TABLE CONTEXT ━━━
For each table in the query:
  [schema.table]
    Rows             : ~[N]
    Partitioned      : [yes/no — key?]
    Indexes          : [name: (cols) TYPE, e.g., "ix_orders_customer: (customer_id) BTREE"]
    Last ANALYZE     : [date — or run: SELECT last_analyze FROM pg_stat_user_tables WHERE relname='...']
    Table correlation : [run: SELECT attname, correlation FROM pg_stats WHERE tablename='...' AND attname='...']
    Bloat            : [known? vacuum status?]

━━━ CONSTRAINTS ON WHAT I CAN CHANGE ━━━
[ ] Cannot add indexes without DBA ticket (flag all index recommendations separately)
[ ] Cannot change table schema or add columns
[ ] Cannot change calling signature
[ ] work_mem cap: [X MB] for this role
[ ] max_parallel_workers_per_gather: [0 / N]
[ ] Cannot use read replica
[ ] Cannot use materialized views
[ ] Cannot change query outside of this function (external callers use it)

━━━ WHAT I NEED ━━━
1. DIAGNOSIS: what specific plan nodes are causing the slowness and why?
   — Statistics problem (bad row estimate) vs structure problem (wrong join type) vs
     resource problem (work_mem, sort spill) vs index problem — these need different fixes

2. REWRITTEN QUERY: the minimal change with the highest impact. Not a full rewrite.
   One change at a time, explained.

3. PLAN PREDICTION: which EXPLAIN nodes will change after the rewrite and what they
   should look like (new join type, new estimated rows, no spill)

4. INDEX RECOMMENDATIONS (separate section, for DBA):
   CREATE INDEX statement
   Estimated build time at current volume
   Estimated size
   Expected impact on this query's plan

5. STATISTICS FIX if the issue is bad estimates:
   — Is this a statistics target problem? (Default is 100 — increase with ALTER TABLE ... ALTER COLUMN ... SET STATISTICS N)
   — Is there data skew? (CREATE STATISTICS for correlated columns)
   — Is the planner ignoring a valid index due to cost miscalibration?

6. QUICK WIN vs PROPER FIX — if a proper fix requires DBA involvement, give me
   a temporary query rewrite that reduces runtime now, and the permanent fix separately.

[PASTE QUERY / FUNCTION]
```

---

### F2. INDEX STRATEGY REVIEW

```
Review the index strategy for this table and query workload.

━━━ TABLE ━━━
[schema.table]
Row count         : ~[N]
Row size (avg)    : [bytes, or "unknown"]
Partitioned       : [yes/no — key]
Write rate        : [~N inserts/updates/deletes per second]
Read patterns     : [describe the 3-5 most common query patterns against this table]

━━━ EXISTING INDEXES ━━━
[Run and paste: SELECT indexname, indexdef FROM pg_indexes WHERE tablename = '[table]' AND schemaname = '[schema]';]
[Run and paste: SELECT indexname, idx_scan, idx_tup_read, idx_tup_fetch FROM pg_stat_user_indexes WHERE relname = '[table]';]

━━━ QUERY WORKLOAD ━━━
[Paste or describe the 3-5 queries that matter most for performance]

━━━ ANALYZE FOR ━━━

[UNUSED INDEXES]
Flag any index with idx_scan = 0 (or very low) for a long time.
Every unused index costs write overhead and storage.
Provide DROP INDEX CONCURRENTLY statement.
Note: idx_scan resets on statistics reset / pg_upgrade — check pg_stat_reset_shared_time.

[MISSING INDEXES]
For each query pattern: is there an index that would convert a Seq Scan to an Index Scan?
Generate CREATE INDEX CONCURRENTLY statements (non-blocking on PG, must be run outside a transaction).

[BLOATED INDEXES]
Flag indexes where index size is disproportionate to table size.
Query: SELECT pg_size_pretty(pg_relation_size(indexrelid)) FROM pg_stat_user_indexes WHERE relname='[table]';
Recommend REINDEX CONCURRENTLY if bloat is significant.

[EXPRESSION INDEXES]
If queries use functions on columns in WHERE/JOIN:
  WHERE LOWER(email) = $1  → CREATE INDEX ON table (LOWER(email))
  WHERE DATE(created_at) = $1 → CREATE INDEX ON table (DATE(created_at))
  WHERE payload->>'status' = $1 → CREATE INDEX ON table ((payload->>'status'))
Generate the expression index and the matching query rewrite.

[PARTIAL INDEXES]
If queries have consistent WHERE clause filters:
  WHERE is_current = TRUE → index on (business_key) WHERE is_current = TRUE
  WHERE status = 'PENDING' → index on (created_at) WHERE status = 'PENDING'
Partial indexes are smaller and faster than full indexes for selective conditions.

[COVERING INDEXES]
If EXPLAIN shows "Index Scan" followed by heap fetches (not "Index Only Scan"):
  Add INCLUDE (col1, col2) to include non-key columns needed by the query
  Turns Index Scan → Index Only Scan, eliminating heap access entirely

[MULTI-COLUMN INDEX COLUMN ORDER]
For index on (col_a, col_b):
  — Equality predicates before range predicates
  — Most selective column first (for composite equality only — range breaks this rule)
  — Index is only usable if leftmost prefix is present in the query
  Review every multi-column index for correct column order.
```

---
---

## SECTION G — CONCURRENCY & LOCKING

---

### G1. LOCKING DESIGN

```
Design the locking strategy for this concurrent write operation.

━━━ OPERATION ━━━
[Describe: what writes to what, in what sequence, how often, how many concurrent workers]

━━━ CONCURRENCY PROFILE ━━━
Concurrent readers       : [yes/no — read replica available?]
Concurrent writers       : [N workers / N app instances — same table? same rows?]
Tolerable lock wait      : [< 1s / < 5s / zero tolerance — fail fast]
Transaction isolation    : [READ COMMITTED / REPEATABLE READ / SERIALIZABLE]
Connection pool          : [PgBouncer transaction mode / session mode / direct]

━━━ LOCK HIERARCHY ANALYSIS ━━━
For every table this operation touches, determine:
  — Lock type required: ROW EXCLUSIVE (DML) / SHARE (read with no concurrent writes) /
    ACCESS EXCLUSIVE (DDL, TRUNCATE — blocks all readers) / ...
  — Duration: how long is each table locked?
  — Can the lock acquisition order cause deadlock?
    Rule: all transactions must acquire locks on tables in the SAME order.
    If Transaction A locks orders then customers, Transaction B must also lock in that order.

━━━ PATTERNS TO EVALUATE ━━━

[ADVISORY LOCKS — for job-level mutual exclusion]
  -- Prevents two workers from processing the same logical unit simultaneously
  SELECT pg_try_advisory_xact_lock(hashtext(p_batch_key::text)::bigint)
  -- Returns FALSE immediately if already held (non-blocking)
  -- Released automatically at transaction end (xact advisory lock)
  -- Use pg_try_advisory_lock (session-level) only if you need persistence across statements
  --   but then MUST explicitly release with pg_advisory_unlock

  Recommended pattern:
    IF NOT pg_try_advisory_xact_lock(hashtext(p_batch_key::text)::bigint) THEN
      RAISE NOTICE 'Batch % already locked by another worker — skipping', p_batch_key;
      RETURN;  -- exit cleanly, not an error
    END IF;

[SELECT FOR UPDATE — for row-level coordination]
  Use FOR NO KEY UPDATE when you only need to prevent concurrent updates, not key changes
  Use FOR UPDATE only when updating the PK or referencing it in a foreign key chain
  Use SKIP LOCKED for queue-style processing (multiple workers consuming a work queue)
  Use FOR SHARE when you need to read-lock rows to prevent deletion while you process them

[BATCH SIZE TO REDUCE LOCK CONTENTION]
  Large single-statement updates hold row locks for the full duration.
  Batching with LIMIT N + COMMIT releases locks after each batch.
  Cost: multiple round trips, partial commit means partial failure is possible.
  Keyset pagination: WHERE pk > v_last_pk ORDER BY pk LIMIT N — not OFFSET.

━━━ DEADLOCK ANALYSIS ━━━
Describe the scenario where a deadlock could occur with this code:
  Worker A: locks row X in table 1, then tries to lock row Y in table 2
  Worker B: locks row Y in table 2, then tries to lock row X in table 1
  → Deadlock

Give me:
1. The specific deadlock scenario for this code (if any)
2. The fix (consistent lock ordering, or reduce to single table, or advisory lock)
3. The pg_locks query to run during testing to observe lock behavior:
   SELECT pid, locktype, relation::regclass, mode, granted
   FROM pg_locks l JOIN pg_stat_activity a USING (pid)
   WHERE relation IN ('[schema.table1]'::regclass, '[schema.table2]'::regclass);

━━━ AUTOVACUUM INTERACTION ━━━
Long-running transactions block autovacuum on all tables they've touched.
Bloated tables → slower queries → longer transactions → more bloat → spiral.
For operations expected to run > 5 minutes:
  — Document the autovacuum impact
  — Consider pg_stat_activity monitoring to detect bloat accumulation
  — Consider splitting into shorter transactions with intermediate commits
```

---
---

## SECTION H — SCHEMA & MIGRATION

---

### H1. SAFE PRODUCTION MIGRATION

```
Write a production-safe migration for this PostgreSQL change.

━━━ CHANGE DESCRIPTION ━━━
[e.g., "Function fn_get_order_total is being replaced with a new version that adds
 a p_currency TEXT parameter with DEFAULT 'USD' and changes internal logic.
 The return type is unchanged. Callers in the application do not need to change."]

━━━ CHANGE CLASSIFICATION ━━━
(This determines the deployment approach)

[ ] Non-breaking — CREATE OR REPLACE is safe:
    - New function (no existing callers)
    - Add parameter WITH DEFAULT (plpgsql only — check for overload conflicts)
    - Change body logic only (signature and return type unchanged)
    - Change COMMENT, volatility, parallel safety

[ ] Breaking — requires DROP + recreate, and caller coordination:
    - Remove parameter
    - Change parameter type
    - Change return type (even compatible types need DROP)
    - Rename object

[ ] Breaking with zero-downtime path:
    - If callers cannot be updated simultaneously with the DB change:
      Phase 1: Deploy new function under name [fn_name]_v2
      Phase 2: Update callers to use [fn_name]_v2
      Phase 3: Drop [fn_name] (separate migration, separate deploy)
      This adds complexity but prevents downtime. Required if callers are
      in a separate deployment pipeline.

━━━ PRE-MIGRATION CHECKS ━━━
(Include these as executable SQL, not comments — run them and verify before proceeding)

-- 1. Confirm current version exists and matches what we expect to replace:
SELECT oid::regprocedure, prosrc
FROM pg_proc
WHERE proname = '[fn_name]'
AND pronamespace = '[schema]'::regnamespace;

-- 2. Check for dependents (views, rules, other functions calling this):
SELECT classid::regclass, objid, deptype
FROM pg_depend
WHERE refobjid = '[schema].[fn_name]([arg_types])'::regprocedure;

-- 3. Check for active sessions using this function right now:
SELECT pid, query, state, query_start
FROM pg_stat_activity
WHERE query ILIKE '%[fn_name]%'
AND state != 'idle';

━━━ MIGRATION SCRIPT STRUCTURE ━━━
-- HEADER
-- Migration: [short description]
-- Ticket: [ID]
-- Author: [placeholder]
-- Date: [YYYY-MM-DD]
-- Type: [non-breaking / breaking / breaking-with-zero-downtime-path]
-- Pre-conditions: [what must be true before this runs]
-- Post-conditions: [what to verify after this runs]

BEGIN;

-- PRE-CHECK (fail fast if assumptions are wrong)
DO $check$
BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM pg_proc
    WHERE proname = '[fn_name]' AND pronamespace = '[schema]'::regnamespace
  ) THEN
    RAISE EXCEPTION 'Pre-condition failed: [fn_name] does not exist in [schema]';
  END IF;
END $check$;

-- DROP IF BREAKING CHANGE
-- [DROP FUNCTION IF EXISTS [schema].[fn_name]([old_arg_types]);]

-- MAIN DDL
[CREATE OR REPLACE FUNCTION ...]

-- PERMISSIONS
GRANT EXECUTE ON FUNCTION [schema].[fn_name]([arg_types]) TO [app_role];
REVOKE EXECUTE ON FUNCTION [schema].[fn_name]([arg_types]) FROM PUBLIC;

-- SMOKE TEST (fail the transaction if the function doesn't behave as expected)
DO $smoke$
DECLARE v_result [return_type];
BEGIN
  -- Test with known inputs and expected output:
  SELECT [schema].[fn_name]([test_args]) INTO v_result;
  IF v_result IS NULL THEN  -- or specific assertion
    RAISE EXCEPTION 'Smoke test failed: expected [X], got NULL';
  END IF;
  RAISE NOTICE 'Smoke test passed: %', v_result;
END $smoke$;

COMMIT;

-- ============================================================
-- ROLLBACK SCRIPT (run this if post-deploy monitoring shows issues)
-- ============================================================
-- BEGIN;
-- DROP FUNCTION IF EXISTS [schema].[fn_name]([new_arg_types]);
-- [Previous version DDL here]
-- GRANT EXECUTE ON FUNCTION [schema].[fn_name]([old_arg_types]) TO [app_role];
-- COMMIT;
-- ============================================================

━━━ ORG CONSTRAINTS ━━━
[ ] Migrations run through [Flyway / Liquibase / manual psql / custom tool]
    Naming convention: [V[version]__description.sql / YYYYMMDD_description.sql]
[ ] All migrations go through PR review before prod — this script will be reviewed
[ ] No DDL transaction wrap (some tools run migrations outside explicit transactions)
    If this applies: replace BEGIN/COMMIT with idempotency guards per statement
[ ] Deployment window: [maintenance window / zero-downtime required]
```

---
---

## SECTION I — TESTING

---

### I1. PRODUCTION-QUALITY TEST SUITE

```
Write tests for this PostgreSQL function/procedure that would catch real bugs.
Not toy tests. Tests that would have caught issues before they hit production.

━━━ FUNCTION UNDER TEST ━━━
[PASTE SIGNATURE AND FULL CODE]

━━━ TEST FRAMEWORK ━━━
[ ] pgTAP — preferred if available; generates TAP output
[ ] Plain SQL assertions — DO blocks with ASSERT / RAISE EXCEPTION
[ ] Both — pgTAP for the test runner, raw SQL for complex setup

Test isolation: wrap every test in a SAVEPOINT / ROLLBACK TO SAVEPOINT block.
  Note: this doesn't work for procedures with autonomous COMMIT.
  For those: document the cleanup approach (DELETE by batch_id, etc.)

━━━ SEED DATA PHILOSOPHY ━━━
- All test data is created inline — no dependency on existing rows
- Use a dedicated test schema: test_[fn_name] — created and dropped per test run
- Data should represent production-realistic values, not id=1 toy rows
- Include at least one edge case value per column type:
    TEXT: empty string, NULL, very long string, unicode, characters that break LIKE patterns
    NUMERIC: zero, negative, very large, fractional
    TIMESTAMPTZ: min date, max date, DST boundary, end of month
    BOOLEAN: both TRUE and FALSE — don't test only the happy path value

━━━ TEST CATEGORIES ━━━

[CORRECTNESS — happy paths]
Test [N] representative real-world input combinations and assert exact output.
Do not assert "returned something" — assert the specific value returned.

[NULL CONTRACT]
For every nullable parameter: what is the documented behavior with a NULL input?
Write a test asserting that exact behavior (return NULL / return empty / raise / use default).
If behavior is undocumented: flag it and write the test as a spec for what it SHOULD do.

[BOUNDARY CONDITIONS]
  Dates      : first/last day of month, Feb 29 on leap year, DST transition day
  Numeric    : zero, negative, max/min for the type, division by zero path
  Strings    : empty string (''), single space, NULL, 256+ chars, SQL special chars (%,_,')
  Result set : zero rows returned, exactly one row, N rows where N is a known value

[IDEMPOTENCY]
Call the function/procedure with identical inputs twice.
Assert: row counts in target tables are identical after run 2 as after run 1.
Assert: no duplicate rows exist for the business key.
Assert: audit log shows run 2 as a no-op (zero inserted, zero updated) if no source changes.

[BUSINESS RULES]
[Describe 2-3 specific business rules this code must enforce]
Write one test per rule that asserts the rule holds even at boundary conditions.
e.g.: "If order_status = 'CANCELLED', the function must return 0 rows, not an error"
      "The effective_to of an expired SCD2 row must always be < effective_from of the next"

[ERROR CONDITIONS]
Input that should raise an exception: write it and assert the specific SQLSTATE.
Input that is syntactically valid but semantically wrong:
  e.g., p_to_date < p_from_date — does the function handle this gracefully?

[PERFORMANCE REGRESSION TEST]
Generate a test that inserts [N] rows of realistic data and measures runtime.
Assert runtime < [threshold] on the test environment.
This catches accidental N+1 patterns introduced in code changes.

━━━ OUTPUT FORMAT ━━━
1. Single .sql file, runnable with:  psql -v ON_ERROR_STOP=1 -f test_[fn_name].sql
2. Each test clearly labeled: -- TEST: [description] / -- EXPECTS: [outcome]
3. Summary at end: SELECT 'N tests passed, N failed' with counts
4. Exit code: non-zero if any test fails (for CI integration)
```

---
---

## SECTION J — AURORA-SPECIFIC

---

### J1. AURORA POSTGRESQL — PLATFORM-AWARE CODE

```
Review / write this code with Aurora PostgreSQL-specific behavior in mind.
Aurora is not vanilla PostgreSQL. Differences matter.

━━━ AURORA CONTEXT ━━━
Version              : Aurora PostgreSQL [version] (engine version [X.Y.Z])
Instance class       : [db.r6g.2xlarge / db.serverless — matters for memory/parallel limits]
Serverless v2        : [yes/no — if yes, ACU range: min [N] max [N]]
RDS Proxy            : [yes/no — if yes, pool mode: transaction / session]
Read replicas        : [yes/no — can this workload be routed to replica?]
Multi-AZ             : [yes/no]
Performance Insights : [enabled/disabled — if enabled, we can use pg_stat_statements]

━━━ AURORA-SPECIFIC BEHAVIORS TO ADDRESS ━━━

[STORAGE ENGINE]
Aurora uses a distributed storage engine (not local disk).
  — I/O is billed by IOPS. Queries that generate unnecessary I/O are a cost problem, not just perf.
  — Sequential scans on large tables are more expensive than on local SSD.
  — Index-only scans are important here: they avoid heap access (= no storage I/O).
  — Recommendation: flag every Seq Scan on tables > 10M rows as a cost concern.

[FAILOVER BEHAVIOR]
Aurora failover promotes a replica to writer in ~30 seconds.
  — Code that holds long transactions will fail mid-transaction on failover.
  — Retry logic in the caller is required; the procedure cannot handle this internally.
  — pg_postmaster_start_time() is NOT reliable on Aurora for detecting failover.
  — Use connections strings with cluster endpoint (not instance endpoint) for automatic failover.
  — Document in procedure header: "Not restart-safe; caller must handle connection errors and retry."

[SERVERLESS V2 SPECIFIC]
  — ACU (Aurora Capacity Units) scale up in ~tens of seconds, not instantly.
  — First query after scale-up may be slow (cold start at new capacity).
  — Long transactions prevent ACU scale-down (cost risk).
  — Set statement_timeout appropriate to workload; runaway queries hold ACUs open.
  — max_connections scales with ACU; at min ACU, connection count is low — size pool accordingly.
  — Avoid long idle transactions: they hold ACUs and prevent scale-down.

[RDS PROXY (TRANSACTION MODE)]
If RDS Proxy is in use with transaction mode (most common):
  — Session-level state is NOT preserved between transactions.
  — This breaks:
      SET search_path = ... (resets after transaction)
      PREPARE / EXECUTE (prepared statements lost)
      Temp tables (may not exist in next transaction on same logical connection)
      pg_advisory_lock (session-level advisory locks; use xact-level instead)
      SET LOCAL works (scoped to transaction) but SET does not persist.
  — Use SET LOCAL for any per-transaction settings.
  — Use pg_try_advisory_xact_lock, never pg_try_advisory_lock in RDS Proxy environments.
  — Temp tables: either use pre-declared staging tables or accept that temp tables
    may need to be created on each call (check existence first).

[PARAMETER GROUPS]
Aurora parameter group defaults differ from community PostgreSQL in some versions.
  — max_parallel_workers_per_gather may default to 0 (no parallel query)
  — shared_buffers is managed by Aurora, not user-configurable
  — effective_cache_size should be ~75% of instance RAM for planner cost estimates
  — work_mem default (4MB) is often too low for analytical queries — may need ALTER ROLE
  — statement_timeout: not set by default — recommend setting at role level for production

[PERFORMANCE INSIGHTS]
If enabled: queries are tracked in pg_stat_statements.
  — Use $1, $2, etc. (bind parameters) in queries so plans cluster by normalized query form.
  — Queries with literal values (WHERE id = 12345) appear as separate statements each time.
  — For high-frequency procedures, this inflates pg_stat_statements — use query normalization.
  — To inspect top queries: SELECT query, calls, total_exec_time/calls AS avg_ms
    FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 20;

━━━ GENERATE ━━━
For the code provided:
1. Flag every issue that is Aurora-specific (not a vanilla PostgreSQL issue)
2. Add Aurora-specific defensive comments in the code where behavior differs
3. Add a "Platform: Aurora PostgreSQL [version]" note in the header comment block
4. Note any parameters that should be set via ALTER ROLE or SET LOCAL for this workload

[PASTE CODE OR DESCRIBE THE OPERATION]
```

---
---

## SECTION K — ANTI-PATTERN SCANNER

---

### K1. FIND EVERY ANTI-PATTERN IN THIS CODE

```
Scan this PostgreSQL code for anti-patterns. Be thorough — I want to find everything
before it finds me in production.

For each anti-pattern found: location, why it's dangerous, the fix.

━━━ SCAN LIST ━━━

[CORRECTNESS TRAPS]
□ NOT IN (subquery) where subquery column is nullable
  — If subquery returns any NULL, entire NOT IN evaluates to FALSE for all rows → zero results
  — Fix: NOT EXISTS (...) or LEFT JOIN ... WHERE right_side IS NULL

□ DISTINCT masking a cartesian product or bad join
  — DISTINCT does not fix the join; it just hides the result. Fix the join.

□ COALESCE to an empty string instead of handling NULL semantically
  — COALESCE(col, '') in a GROUP BY groups empty string with NULL — usually wrong

□ Implicit timezone conversion: DATE(timestamptz_col) uses server timezone
  — Safe only if server timezone is verified UTC. Use DATE(col AT TIME ZONE 'UTC') explicitly.

□ BETWEEN on timestamps: BETWEEN '2024-01-01' AND '2024-01-31' misses Jan 31 data after midnight
  — Use: >= '2024-01-01' AND < '2024-02-01'

□ Integer division returning wrong results: 1/2 = 0 in PostgreSQL (integer division)
  — Fix: 1::numeric/2 or 1.0/2

□ String comparison on UUID columns stored as TEXT vs native UUID
  — Text comparison is case-sensitive; UUID comparison is not. Normalize or use proper type.

[TRANSACTION / STATE TRAPS]
□ WHEN OTHERS THEN NULL or WHEN OTHERS THEN RAISE NOTICE (without RE-RAISE)
  — Silently swallows errors. The caller doesn't know the procedure failed.

□ GET DIAGNOSTICS after more than one statement
  — ROW_COUNT reflects only the immediately preceding statement. One statement between
    the DML and GET DIAGNOSTICS resets it.

□ Sequence consumed inside a dry-run / rollback path
  — Sequences are non-transactional; consumed values are lost even on rollback.
  — Test: does this code call NEXTVAL in a path that gets rolled back?

□ now() for event timestamps in audit tables
  — All rows in the same transaction get the same timestamp.
  — Fix: clock_timestamp()

□ Temp table assumed to exist from a previous call
  — In PgBouncer transaction mode, the session may be handed to a different connection.
  — Fix: always check existence before use; or use pre-declared staging tables.

[PERFORMANCE KILLERS]
□ Function call on indexed column in WHERE clause
  — WHERE LOWER(email) = $1 prevents index use unless expression index exists
  — WHERE DATE(created_at) = $1 prevents partition pruning
  — WHERE id::text = $1 implicit cast prevents index use

□ OFFSET on large tables
  — OFFSET 1000000 reads and discards 1M rows every time
  — Fix: keyset pagination: WHERE id > $last_id ORDER BY id LIMIT N

□ ORDER BY in a CTE or subquery that feeds a query with its own ORDER BY
  — Inner ORDER BY is ignored by the planner (and costs a sort)
  — Exception: ORDER BY inside a window function is intentional

□ SELECT * from a table when only 2 columns are needed
  — Pulls all columns through the executor; prevents Index Only Scan

□ Cursor loop doing what a single SQL statement could do
  — "For each row in source: UPDATE target" → one UPDATE ... FROM instead

□ CTE used as optimization fence (no MATERIALIZED / NOT MATERIALIZED explicit)
  — Pre-PG12: always materialized (can hurt)
  — PG12+: planner decides — make it explicit to document intent

□ ROW_NUMBER() / RANK() in a subquery, then WHERE on the row_number, then outer DISTINCT
  — Usually indicates a windowed dedup that could be simpler DISTINCT ON

[SECURITY VULNERABILITIES]
□ SECURITY DEFINER without SET search_path = [schema], pg_catalog
  — Search path hijack: attacker creates a function with the same name in their schema

□ EXECUTE 'SELECT ... FROM ' || table_name || ' WHERE ...'
  — SQL injection. Fix: EXECUTE format('SELECT ... FROM %I WHERE ...', table_name) USING value

□ EXECUTE format(..., user_input, ...) using %s instead of %L for values or %I for identifiers
  — %s does no quoting. Use %L for literal values, %I for identifiers.

□ Granting EXECUTE to PUBLIC on a SECURITY DEFINER function
  — Any database user, including unprivileged ones, can call it

[OPERATIONAL LANDMINES]
□ RAISE EXCEPTION inside a trigger
  — Aborts the transaction in the calling application. In some ORMs this is uncatchable.
  — Fix: use RETURN OLD / RETURN NULL (for BEFORE trigger) to silently reject the change,
    or document that callers must handle this exception explicitly.

□ IMMUTABLE on a function that reads from a table
  — Planner will inline and cache the result. Function sees stale data.
  — Test: does this function contain any SELECT from a table? If yes, it cannot be IMMUTABLE.

□ Missing ON CONFLICT target — or target doesn't match an actual unique constraint
  — Silent failure: the INSERT does nothing without error, and you don't know why.
  — Verify: SELECT * FROM pg_constraint WHERE contype = 'u' AND conrelid = '[table]'::regclass;

□ Hardcoded CURRENT_DATE or CURRENT_TIMESTAMP inside load logic
  — Backfills and reruns process the wrong date range.

□ Long transaction blocking autovacuum
  — Any transaction open > autovacuum_vacuum_cost_delay * N seconds prevents vacuum.
  — Tables accumulate bloat → slower queries → longer transactions → spiral.
  — Document max expected runtime; set statement_timeout accordingly.

━━━ REPORT FORMAT ━━━
| # | Anti-pattern | Line/Block | Danger Level | Impact | Fix |

Danger Level: CRITICAL / HIGH / MEDIUM / LOW
CRITICAL = data corruption or security vulnerability
HIGH = production failure or silent wrong results
MEDIUM = performance degradation under load
LOW = technical debt / maintainability

[PASTE CODE]
```

---
---

## SECTION L — QUICK CONTEXT INJECTORS

Append to any prompt. Mix and match. These tell the model the constraints it cannot assume.

```
── CONNECTION & SESSION ──
Connection pool: PgBouncer transaction mode.
  Consequences: no session-level SET, no PREPARE, no temp tables with session lifetime,
  advisory locks must be xact-level (pg_try_advisory_xact_lock), not session-level.

RDS Proxy in use.
  Same consequences as PgBouncer transaction mode for session state.

── AURORA ──
Target: Aurora PostgreSQL [version].
  Note Aurora-specific behavior differences from community PostgreSQL where relevant.
  Note: rds_superuser only (no actual superuser). No pg_read_binary_file, no COPY to/from file.

Target: Aurora Serverless v2.
  Avoid long idle transactions (prevent ACU scale-down).
  Set statement_timeout. No reliance on constant max_connections.

── PARTITIONING ──
Target table is range-partitioned on [col] ([type]).
  Ensure partition pruning works: filter must reference [col] directly,
  no function wrapping, no implicit cast. Generate the filter correctly.
  Do not use TRUNCATE on the parent table.

── MULTI-SCHEMA / SEARCH PATH ──
search_path is not set and should not be relied on.
  Fully qualify every object reference: schema.table, schema.function().
  In SECURITY DEFINER functions: SET search_path = [schema], pg_catalog in the function body.

── CONCURRENT WORKERS ──
This procedure runs with [N] concurrent workers processing different [date ranges / batch IDs].
  Show the advisory lock block using pg_try_advisory_xact_lock(hashtext(p_batch_key::text)::bigint).
  Show the "already locked" early exit path. This is not an error; it is normal operation.

── DBT CONTEXT ──
This function is called from a dbt macro, post-hook, or on-run-end hook.
  Must be STABLE or IMMUTABLE (no writes, no sequences, no INSERT/UPDATE/DELETE).
  No RAISE EXCEPTION — signal errors via return value or raise with SQLSTATE.
  No explicit COMMIT or ROLLBACK — dbt manages the transaction.
  search_path will be set to dbt target schema — fully qualify all non-target objects.

── TIMESTAMPS ──
All timestamps are UTC. CURRENT_TIMESTAMP returns UTC on this cluster.
  Do NOT use LOCALTIME, LOCALTIMESTAMP, AT TIME ZONE [server timezone].
  For event time in audit: clock_timestamp(). For transaction time: now() / CURRENT_TIMESTAMP.

── INFRA RESTRICTIONS ──
No TRUNCATE on live production tables. Use DELETE with WHERE, or partition swap.
No CREATE INDEX CONCURRENTLY inside a transaction block.
No ad-hoc CREATE TEMP TABLE — use pre-declared staging tables in [staging_schema].
No dynamic SQL unless explicitly approved in this prompt.
No superuser features: no COPY to/from server filesystem, no pg_read_file, no pg_ls_dir.

── OUTPUT CONTROL ──
Return only the changed blocks — before/after — not the full function.
Return only executable SQL. No explanatory prose. No markdown fences in the SQL output.
Include a one-line comment at the top of each changed block explaining what changed and why.
```

---
---

## QUICK REFERENCE — DECISIONS THAT MATTER

> For each one: get it wrong once and you'll never forget it.

| Decision | Wrong default | Right approach |
|---|---|---|
| Volatility | Mark everything VOLATILE "to be safe" | STABLE for functions that only read; IMMUTABLE only if truly zero DB access |
| Audit timestamps | `now()` | `clock_timestamp()` — `now()` is transaction start; all rows in same txn get same timestamp |
| SECURITY DEFINER | No `search_path` set | Always: `SET search_path = [schema], pg_catalog` in function body |
| NULL in NOT IN | `WHERE id NOT IN (SELECT id FROM ...)` | `WHERE NOT EXISTS (...)` — NULL in subquery silently returns zero rows with NOT IN |
| ON CONFLICT target | Column name (works sometimes) | Must exactly match an existing UNIQUE CONSTRAINT or unique index |
| Advisory locks | `pg_advisory_lock` (session) | `pg_try_advisory_xact_lock` (transaction-scoped, auto-released, non-blocking) |
| Pagination | `OFFSET N` | Keyset: `WHERE pk > $last_pk ORDER BY pk LIMIT N` |
| CTE fence | No annotation | Explicit `MATERIALIZED` or `NOT MATERIALIZED` in PG12+; never assume |
| Temp tables in pools | Create per-call | Use pre-declared staging tables; `TRUNCATE` at procedure start |
| Watermark boundary | `WHERE ts >= last_run` | Capture `v_to_ts := clock_timestamp()` BEFORE the source query; filter `> last AND <= v_to_ts` |
| SCD2 integrity | Trust the logic | Enforce with `UNIQUE (business_key) WHERE is_current = TRUE` partial index |
| Dynamic SQL values | `'WHERE col = ' \|\| val` | `EXECUTE format('WHERE col = $1') USING val` |
| Date range | `BETWEEN '2024-01-01' AND '2024-01-31'` | `>= '2024-01-01' AND < '2024-02-01'` |
| Row count after DML | `SELECT COUNT(*)` after the statement | `GET DIAGNOSTICS v_n = ROW_COUNT` immediately after the DML |
| Long transaction | Hope autovacuum catches up | Set `statement_timeout`; document max expected runtime; batch + commit for large loads |
| Index on expression | No index, wrap column in function | Expression index: `CREATE INDEX ON t (LOWER(col))` and rewrite query to match |
