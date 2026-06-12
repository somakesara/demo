# Behaviour Specification for Agile Development Teams

> A practical guide for microservices and data engineering teams working in SAFe and Kanban environments on .NET and AWS.

---

## Who This Course Is For

You are a developer. You build .NET microservices, Kafka consumers, REST APIs, ETL pipelines, and PostgreSQL-backed data systems on AWS. You work in sprints, PI planning sessions, and Kanban flows. You have probably been in a refinement session where a story was "clear enough" — and then spent three days building the wrong thing.

This course is not about theory. It is about writing stories that prevent real problems your team has already experienced or will experience soon.

---

## How This Course Is Structured

Each module opens with a **pain story** — something that has gone wrong, written in the language your team actually uses. The concept follows. Then real examples. Then an exercise using patterns from your stack.

By the end you will be able to:
- Write behaviour scenarios for REST APIs, Kafka events, ETL pipelines, and data transformations
- Identify missing scenarios before they become production incidents
- Speak a common behaviour language across your team and with product owners
- Seed a prompt library and eventually an agent that generates stories from your codebase

---

# Module 1: Why We Need This at All

## The Pain 😅

### Generic training version:

> *"The feature wasn't what the stakeholder expected."*

Everyone nods. Nobody relates.

### Your team's version:

> It's 2am. PagerDuty fires. The on-call engineer opens their laptop.
>
> The Kafka consumer for `order-placed` events is throwing `NullReferenceException` in production. It has been doing so silently for 4 hours. The dead-letter queue has 12,000 messages in it.
>
> Why? The upstream .NET service started publishing events where `customerId` is null — because a new registration flow was added that doesn't require account creation. Nobody wrote a scenario for it. Nobody asked "what happens when customerId is missing?" The consumer assumed it would always be there. The assumption was written in code, not in a story.
>
> The fix is a one-line null check. The cost was 4 hours of lost orders, a Saturday morning incident review, and 12,000 replayed events that created duplicate notifications.
>
> **The story was "done." The behaviour was never defined.**

---

## What Behaviour Specification Actually Solves

Given/When/Then (GWT) is not a documentation format. It is a **thinking tool** that forces a team to answer three questions before writing a single line of code:

1. **Given** — what state is the system in when this happens?
2. **When** — what event or action triggers the behaviour?
3. **Then** — what observable outcome must occur?

When these three questions are answered for the happy path *and* every edge case, the null check gets written. The dead-letter queue behaviour gets defined. The duplicate event gets handled. Not because a developer is clever, but because the behaviour was specified.

---

## The Behaviour Specification Format

```gherkin
As a [who]
I want to [what]
So that [why / value]

Scenario: [name of this specific behaviour]
  Given [precondition — state of the system]
  When  [event or action that triggers behaviour]
  Then  [observable, verifiable outcome]
```

And for the null case that caused the 2am incident:

```gherkin
Scenario: order-placed event received with missing customerId
  Given the order-placed Kafka topic receives a message
  And the customerId field is null
  When the order consumer processes the message
  Then the message should be moved to the dead-letter queue
  And an alert should be raised with the message payload
  And no order should be created
  And the consumer should continue processing subsequent messages
```

Four lines. Written in refinement. Would have prevented the incident entirely.

---

## Why Developers Resist Writing Stories

Be honest. You have thought at least one of these:

> *"I already know what to build."*
> *"Stories are for product owners, not engineers."*
> *"This slows me down."*
> *"The acceptance criteria are good enough."*

All fair. Here is the counter:

| Resistance | Reality |
|---|---|
| "I know what to build" | Your assumptions are not your colleague's assumptions |
| "Stories are for product owners" | No one else will specify what happens when Kafka lags 3 hours |
| "This slows me down" | One missed edge case costs more time than writing 10 scenarios |
| "AC is good enough" | AC written as bullets has no Given — the context is always missing |

Behaviour specification is not about process. It is about making your assumptions explicit so they can be challenged before they become bugs.

---

## Module 1 Key Takeaways

- Behaviour specification is a thinking tool, not a documentation ritual
- The real cost of missing scenarios shows up in production incidents
- Three questions: what state? what trigger? what outcome?
- Distributed systems have more edge cases than UIs — the need is greater, not smaller

---

# Module 2: Before You Write Scenarios — Example Mapping

## The Pain 😅

### Generic training version:

> *"Hold a Three Amigos session to align the team."*

Nobody knows how to run it. It becomes a meeting about having a meeting.

### Your team's version:

> Sprint planning. A story arrives: *"As a data engineer, I want to replicate customer records from PostgreSQL to the data warehouse so that reports are always current."*
>
> Developer 1 assumes "always current" means within 5 minutes. Developer 2 assumes it means end-of-day batch. The product owner assumed real-time. The QA engineer asks what "current" means when replication lag spikes. Nobody has an answer.
>
> Four people, four interpretations. One story. Zero scenarios written. Development starts on Monday.
>
> Three weeks later: the replication job runs every hour. Reports show data from 59 minutes ago. The business says it is broken. The developer says it is working as built. Both are right. Neither is helpful.

---

## What Is Example Mapping?

Example Mapping is a structured conversation technique created by Matt Wynne. It runs before refinement, typically 25–30 minutes, using four types of cards:

| Card | Colour | Represents |
|---|---|---|
| **Story** | Yellow | The user story headline |
| **Rule** | Blue | A business rule that governs behaviour |
| **Example** | Green | A concrete example that illustrates a rule |
| **Question** | Red | An unanswered question or assumption |

You do not need physical cards. A whiteboard, Miro, or even a markdown table works.

---

## Running Example Mapping for the Replication Story

**Yellow — Story:**
> As a data engineer, I want to replicate customer records from PostgreSQL to the data warehouse so that reports always reflect current data.

**Blue — Rules (what must always be true):**
- Replication must complete within an agreed SLA
- Failed replication must not silently succeed
- Deleted records in source must be handled
- Schema changes in source must not break replication

**Green — Examples (one per rule, concrete):**

*For "must complete within SLA":*
- Example: 10,000 new records arrive — replication completes within 5 minutes ✅
- Example: Replication lag exceeds 5 minutes — an alert fires ✅

*For "failed replication must not silently succeed":*
- Example: PostgreSQL source is unreachable — job fails with an error, does not report success ✅
- Example: 500 records fail validation — job logs failures, reports partial success with count ✅

*For "deleted records":*
- Example: A customer is deleted in source — the record is soft-deleted (not removed) in the warehouse ✅

**Red — Questions (unresolved):**
- What is the agreed SLA? 5 minutes? 1 hour? Not defined yet 🔴
- What happens to records that fail schema validation — skip or halt? 🔴
- Who owns the alert — data team or platform team? 🔴

---

## From Example Map to Scenarios

Each green example becomes a behaviour scenario directly. The red questions must be answered before development starts.

```gherkin
Scenario: Replication completes within SLA
  Given the PostgreSQL source has 10,000 new customer records
  When the replication job runs
  Then all 10,000 records should appear in the data warehouse
  And the job should complete within 5 minutes
  And the job status should be reported as "Success"

Scenario: Replication source is unreachable
  Given the PostgreSQL source database is unavailable
  When the replication job is triggered
  Then the job should fail with status "Failed - Source Unreachable"
  And an alert should be sent to the data platform channel
  And the job should NOT report "Success"
  And no partial data should be written to the warehouse

Scenario: Records fail schema validation during replication
  Given the replication job is running
  And 500 records have a null value in a non-nullable column
  When the validation step runs
  Then the 500 invalid records should be written to a quarantine table
  And the remaining valid records should be replicated successfully
  And the job status should report "Completed with 500 validation failures"
```

Notice: these scenarios came from a conversation, not from someone filling in a template. The format captures what the team agreed on.

---

## The Red Cards Are the Most Valuable Output

A completed Example Map with 8 red cards is a success. It means the team found 8 assumptions before writing code. Red cards that survive to development become production bugs.

---

## Module 2 Key Takeaways

- Example Mapping happens before scenario writing — it surfaces rules and questions
- Green examples become behaviour scenarios directly
- Red questions must be resolved before development starts
- A map full of red cards is not failure — it is the point
- Run it in 25–30 minutes maximum; time-box ruthlessly

---

# Module 3: Defining the System State

## The Pain 😅

### Generic training version:

> ```
> Given I am on the login page
> ```

### Your team's version:

> ```
> Given the customer-profile-updated event has been published to the Kafka topic
> And the enrichment service has consumed and processed it
> And the PostgreSQL read replica has caught up with the primary
> And the Redis cache has NOT yet been invalidated
> When the API receives a GET /customers/{id} request
> Then... which version of the data does it return?
> ```
>
> Nobody wrote this scenario. The cache returned stale data for 4 minutes after every profile update. Three microservices downstream made decisions on stale data. The product owner filed it as a bug. The developer said "cache invalidation is hard." Both were right. Neither wrote a scenario for it.

---

## What the System State Clause Really Means

Given describes the **state of the system** before the action occurs. In distributed systems this is harder than it sounds because state is spread across:

- Database records (PostgreSQL, DynamoDB)
- Cache layers (Redis, ElastiCache)
- Message queues (Kafka topics, SQS queues)
- External service states (upstream APIs, third-party services)
- In-flight events (messages in transit, pending jobs)

A well-written Given clause makes the relevant state explicit. It does not describe everything — only what matters for this specific scenario.

---

## System State Patterns for Your Stack

### REST API — resource state

```gherkin
Given a customer record with id "C-12345" exists in the database
And the customer status is "Active"
```

### Kafka consumer — message state

```gherkin
Given a valid order-placed event is on the "orders" Kafka topic
And the event payload contains a customerId, orderId, and itemList
```

### Cache state

```gherkin
Given the customer profile for id "C-12345" is cached in Redis
And the cached version is 3 minutes old
```

### ETL pipeline — source data state

```gherkin
Given the PostgreSQL source table "customers" contains 50,000 records
And 1,200 records have been updated since the last replication run
And 15 records have been deleted since the last replication run
```

### Queue depth / lag

```gherkin
Given the order-placed Kafka topic has a consumer lag of 0
And the enrichment service is healthy and processing normally
```

### AWS infrastructure state

```gherkin
Given the ECS task for the enrichment service is running with 2 healthy instances
And the RDS PostgreSQL instance is available and accepting connections
```

---

## Single vs Multiple State Conditions

Use **And** to chain conditions. Each condition should be genuinely necessary for the scenario to make sense.

```gherkin
✅ Given a customer record exists with id "C-12345"
   And the customer has placed 3 previous orders
   And the customer's account status is "Suspended"
```

Strip conditions that are always true — do not state them in every scenario:

```gherkin
❌ Given the AWS infrastructure is running
   And the Kafka cluster is healthy
   And the database is available
   And the application has started successfully
```

These are environmental assumptions, not scenario-specific state. They belong in your test environment setup, not in every Given clause.

---

## Common State Definition Mistakes in Distributed Systems

**Missing the queue state:**
```gherkin
❌ Given a payment is being processed
✅ Given a payment-initiated event is on the payments Kafka topic
   And the event has not yet been consumed by the payment processor service
```

**Missing the cache state:**
```gherkin
❌ Given a customer exists
✅ Given a customer record exists in the database
   And the customer's profile is NOT present in the Redis cache
```

**Describing infrastructure instead of state:**
```gherkin
❌ Given the Lambda function is deployed to us-east-1
✅ Given the customer-events Lambda function is active
   And it is subscribed to the customer-updated SNS topic
```

---

## Module 3 Key Takeaways

- The state clause describes system state — not just user state
- In distributed systems: database, cache, queue, and service states all matter
- Use And freely for genuinely relevant conditions
- Strip environmental assumptions — they are not state conditions
- Missing cache or queue state in Given is a common source of flaky scenarios

---

# Module 4: Identifying the Trigger

## The Pain 😅

### Generic training version:

> ```
> When I click Submit
> ```

### Your team's version:

> ```
> When... what exactly?
>
> When the Kafka consumer reads the message?
> When the Lambda is triggered by the SNS event?
> When the ECS task polls the SQS queue?
> When the ETL job is triggered by the cron schedule?
> When the API receives the HTTP request?
> When the retry mechanism fires for the 3rd time?
> ```
>
> A story arrives: *"Process customer data enrichment."* The When clause is missing entirely. Three developers implement three different trigger mechanisms. One polls. One subscribes. One waits to be called. The product owner asks why enrichment sometimes happens in real-time and sometimes takes 20 minutes. Nobody can answer because nobody wrote down what "triggered" meant.

---

## What the Trigger Clause Really Means in Distributed Systems

When describes the **single event or action** that triggers the behaviour being specified. In your stack, triggers are rarely a button click. They are:

- An HTTP request hitting a .NET API endpoint
- A Kafka message arriving on a topic
- An SQS message being consumed by an ECS task
- A scheduled EventBridge rule firing
- A PostgreSQL trigger or CDC event
- An S3 file landing in a bucket
- A Lambda being invoked by SNS

Being specific about the trigger is what allows developers to build the right mechanism and testers to simulate it correctly.

---

## Trigger Patterns for Your Stack

### REST API trigger

```gherkin
When a POST request is made to /api/v1/orders with a valid order payload
When a GET request is made to /api/v1/customers/C-12345
When a DELETE request is made to /api/v1/customers/C-12345 by an admin user
```

### Kafka trigger

```gherkin
When the order-consumer service reads an order-placed event from the "orders" topic
When a customer-deleted event arrives on the "customer-events" topic
When the Kafka consumer processes the message for the third time after two failures
```

### Scheduled / batch trigger

```gherkin
When the nightly ETL job is triggered at 02:00 UTC
When the replication job runs after a 6-hour idle period
When EventBridge triggers the data-enrichment Lambda at the scheduled time
```

### SQS / async trigger

```gherkin
When the payment-processing ECS task picks up a message from the payments SQS queue
When the dead-letter queue handler processes a failed order-placed message
```

### File / S3 trigger

```gherkin
When a CSV file is uploaded to the s3://data-ingestion/customers/ bucket
When an S3 PutObject event triggers the file-processing Lambda
```

---

## One Trigger Per Scenario

The trigger clause describes one trigger. If you find yourself writing two triggers, you have two scenarios.

```gherkin
❌ When the Kafka consumer reads the message and enriches the data and writes to PostgreSQL

✅ Scenario 1:
   When the Kafka consumer reads the customer-updated event

✅ Scenario 2 (separate):
   When the enrichment service processes the customer record

✅ Scenario 3 (separate):
   When the enriched record is written to PostgreSQL
```

Each scenario tests one piece of behaviour. When it fails, you know exactly where.

---

## Retry and Failure Triggers

These are frequently missing from stories and are where production incidents live.

```gherkin
Scenario: Message processing fails and is retried
  Given a valid order-placed message is on the orders Kafka topic
  And the enrichment service is temporarily unavailable
  When the consumer attempts to process the message
  Then the message should be retried after 30 seconds
  And the retry attempt should be logged with the original message id

Scenario: Message exceeds retry limit
  Given an order-placed message has been retried 3 times
  And each retry has failed with a timeout error
  When the consumer attempts the 4th retry
  Then the message should be moved to the dead-letter queue
  And an alert should be raised with the message payload and failure reason
  And the consumer should continue processing the next message
```

---

## Module 4 Key Takeaways

- The trigger clause is the single event that initiates behaviour — be specific about the mechanism (HTTP, Kafka, SQS, cron, S3)
- One trigger per scenario — split if you find yourself writing two
- Retry and failure triggers are scenarios too — specify them explicitly
- The trigger determines how QA simulates the scenario — vague triggers produce untestable scenarios

---

# Module 5: Specifying the Verifiable Outcome

## The Pain 😅

### Generic training version:

> ```
> Then the data should be saved
> Then the user should be notified
> Then it should work correctly
> ```

### Your team's version:

> The ETL job has been running for three months. Every night it completes with status "Success." One morning the data team notices customer counts in reports are 8% lower than expected.
>
> Investigation finds: the transformation step has been silently dropping records where the `phone_number` field contains non-numeric characters — a data quality issue introduced by a new source system six weeks ago. The job succeeded. The records were lost. Nobody noticed.
>
> The original story's Then clause: *"Then the data should be loaded into the warehouse."*
>
> It was loaded. Just not all of it.
>
> **"The data should be loaded" is not a Then clause. It is a wish.**

---

## What the Outcome Clause Really Means

Then describes an **observable, verifiable outcome** — something that can be checked by a human tester or an automated test. In distributed systems, outcomes happen in multiple places simultaneously:

- HTTP response (status code, body, headers)
- Database state (record created, updated, deleted, unchanged)
- Cache state (invalidated, updated, unchanged)
- Message published (to Kafka topic, SQS queue, SNS topic)
- Alert raised (PagerDuty, CloudWatch, Slack)
- Log entry written (CloudWatch Logs, structured log)
- File written (S3, local)
- No action taken (for negative scenarios)

A good Then clause specifies *which* of these happened and *what* the observable value is.

---

## Outcome Patterns for Your Stack

### REST API response

```gherkin
Then the response status should be 201 Created
And the response body should contain the created order id
And the Location header should point to /api/v1/orders/{orderId}

Then the response status should be 400 Bad Request
And the response body should contain "customerId is required"
And no order should be created in the database
```

### Database state

```gherkin
Then a new customer record should be created in the customers table
And the record should have status "Active"
And the created_at timestamp should be within 1 second of the request time

Then the customer record should NOT be deleted from the database
And the deleted_at field should be set to the current timestamp
And the status should be updated to "Deleted"
```

### Kafka message published

```gherkin
Then a customer-updated event should be published to the "customer-events" Kafka topic
And the event payload should contain the customerId, updatedFields, and timestamp
And the event should be published within 500ms of the API response

Then NO message should be published to the "customer-events" topic
```

### Cache state

```gherkin
Then the customer profile cache entry for id "C-12345" should be invalidated
And the next GET request should fetch fresh data from the database

Then the Redis cache entry should be updated with the new customer data
And the TTL should be reset to 300 seconds
```

### ETL / pipeline outcome

```gherkin
Then all 50,000 records should be present in the warehouse customers table
And the record count in the warehouse should match the source count exactly
And the job completion status should be "Success" with a record count of 50,000

Then 49,850 records should be successfully loaded
And 150 records with invalid phone_number format should be written to the quarantine table
And the job status should be "Completed with 150 validation failures"
And an alert should be sent with the quarantine table location
```

### Alert / notification outcome

```gherkin
Then a CloudWatch alarm should be triggered
And a PagerDuty alert should be sent to the on-call data engineer
And the alert should include the job name, failure reason, and timestamp

Then NO alert should be raised
And the job should complete silently with status "Success"
```

### Negative outcome (nothing should happen)

```gherkin
Then the order should NOT be created in the database
Then NO Kafka message should be published
Then the existing record should remain unchanged
Then the consumer should NOT acknowledge the message
```

---

## The "Silent Success" Anti-Pattern

The most dangerous Then clause in data engineering is the one that isn't written.

```gherkin
❌ Then the pipeline should complete successfully

✅ Then the pipeline should complete with status "Success"
   And the records_processed count should equal the records_read count
   And 0 records should be in the quarantine table
   And the completion log should include source count, target count, and duration
```

The first passes when 8% of records are silently dropped. The second does not.

**Rule: if your Then clause would pass when data is lost, rewrite it.**

---

## Specifying Non-Functional Outcomes

Performance, reliability, and observability are not afterthoughts — they are Then clauses.

```gherkin
Scenario: API response time under load
  Given the customer API is handling 500 concurrent requests
  When each request is a GET /customers/{id}
  Then each response should be returned within 200ms at the 95th percentile
  And no request should return a 5xx response

Scenario: Replication job completes within SLA
  Given the PostgreSQL source has 100,000 records to replicate
  When the replication job runs
  Then all records should be present in the warehouse within 10 minutes
  And the job completion should be logged with duration in CloudWatch

Scenario: Consumer recovers from ECS task restart
  Given the Kafka consumer ECS task is restarted mid-processing
  When the new task starts and resumes consuming
  Then no messages should be processed twice
  And no messages should be skipped
  And the consumer lag should return to 0 within 2 minutes
```

---

## Module 5 Key Takeaways

- The outcome clause must be observable — specify status codes, record counts, field values, topic names
- Cover outcomes in all affected layers: API, database, cache, queue, logs, alerts
- The silent success anti-pattern: an outcome that passes when data is lost is not a valid outcome
- Negative outcomes (nothing should happen) are equally important to specify
- Non-functional outcomes — performance, SLAs, observability — are outcome clauses too

---

# Module 6: Edge Cases — Where Production Incidents Are Born

## The Pain 😅

### Generic training version:

> ```
> What if the input is invalid?
> ```

### Your team's version:

> **Incident log, 11:47pm Saturday:**
>
> *"The order enrichment Lambda is timing out. Consumer lag on the orders topic is 45,000 and climbing. Revenue impact confirmed."*
>
> Root cause (found at 2:14am): a new merchant onboarded that afternoon had a product catalogue with 4,000 line items per order. The Lambda was designed and tested for orders with up to 50 items. Nobody wrote a scenario for large payloads. The timeout was set to 30 seconds. Enrichment of a 4,000-item order took 4 minutes. Every message from that merchant caused a timeout. Every timeout caused a retry. Retries caused more timeouts. Kafka lag compounded.
>
> The fix: 45 minutes of work. Two configuration changes and a batch size limit.
>
> The missing scenario:
> ```
> Scenario: Order event with unusually large item count
>   Given an order-placed event contains 1,000 or more line items
>   When the enrichment Lambda processes the event
>   Then... nobody had asked the question.
> ```

---

## The Edge Case Taxonomy for Distributed Systems

Use this taxonomy in every refinement session to find missing scenarios.

### 1. Missing / Null / Empty

| Question | Example |
|---|---|
| What if a required field is null? | `customerId` is null in Kafka event |
| What if an optional field is missing entirely? | No `metadata` key in the payload |
| What if a collection is empty? | Order with 0 line items |
| What if the source table has 0 records? | ETL runs against an empty table |

```gherkin
Scenario: Kafka event received with null customerId
  Given an order-placed event arrives on the orders topic
  And the customerId field is null
  When the enrichment service processes the event
  Then the message should be moved to the dead-letter queue
  And an alert should be raised with the message payload
  And no order enrichment should be attempted
```

---

### 2. Boundary Conditions

| Question | Example |
|---|---|
| What is the maximum payload size? | Kafka message > 1MB |
| What is the maximum record count? | ETL batch of 1,000,000 records |
| What happens at exactly the limit? | File size exactly 10MB |
| What happens 1 unit over the limit? | File size 10MB + 1 byte |

```gherkin
Scenario: Kafka message exceeds maximum payload size
  Given an order-placed event has a payload size of 1.1MB
  When the producer attempts to publish the event
  Then the publish should fail with "Message too large"
  And the order details should be stored in S3
  And a reference event with the S3 location should be published instead

Scenario: ETL batch processes maximum record count
  Given the source table contains exactly 1,000,000 records
  When the ETL job runs
  Then all 1,000,000 records should be processed
  And the job should not exceed the 4-hour SLA
  And memory usage should not exceed the configured ECS task limit
```

---

### 3. Concurrency and Duplicate Events

| Question | Example |
|---|---|
| What if the same event is processed twice? | Kafka offset replay |
| What if two services process the same record simultaneously? | Race condition on update |
| What if the same API call is made twice in quick succession? | Double-click, retry |

```gherkin
Scenario: Duplicate order-placed event processed (idempotency)
  Given an order-placed event with orderId "ORD-999" was successfully processed
  When the same event is consumed again due to a Kafka offset reset
  Then no duplicate order should be created
  And the consumer should acknowledge the message
  And a duplicate-event log entry should be written with the orderId

Scenario: Concurrent API requests for the same resource
  Given two PATCH requests for customer "C-12345" arrive simultaneously
  When both requests attempt to update the customer's address
  Then one request should succeed with status 200
  And the other should receive status 409 Conflict
  And the final database state should reflect exactly one of the updates
```

---

### 4. Downstream Service Unavailability

| Question | Example |
|---|---|
| What if the downstream API is down? | Payment service unavailable |
| What if the database is unreachable? | RDS failover in progress |
| What if the cache is unavailable? | ElastiCache node failure |
| What if Kafka is unavailable? | Broker outage |

```gherkin
Scenario: Downstream payment service unavailable during order processing
  Given an order-placed event is being processed
  And the payment service is returning 503 Service Unavailable
  When the enrichment service attempts to validate payment status
  Then the message should not be acknowledged
  And it should be retried after 60 seconds
  And after 3 failed retries it should be moved to the dead-letter queue
  And the order status in the database should be set to "Pending - Payment Validation Failed"

Scenario: Redis cache unavailable during API request
  Given the ElastiCache Redis cluster is unavailable
  When a GET request is made to /api/v1/customers/C-12345
  Then the API should fall back to fetching data directly from PostgreSQL
  And the response should be returned successfully with status 200
  And a warning should be logged indicating cache bypass
```

---

### 5. Data Quality and Schema

| Question | Example |
|---|---|
| What if the data doesn't match the expected schema? | New field added upstream |
| What if a value is in an unexpected format? | Date as string not ISO8601 |
| What if the data contains characters that break processing? | Special characters in names |
| What if the schema has changed without notice? | Upstream contract broken |

```gherkin
Scenario: Kafka event received with unrecognised schema version
  Given a customer-updated event arrives with schemaVersion "v3"
  And the consumer only supports "v1" and "v2"
  When the consumer attempts to deserialise the event
  Then the message should be moved to the dead-letter queue
  And an alert should be raised indicating unsupported schema version
  And the consumer should continue processing subsequent messages

Scenario: ETL source record contains special characters in name field
  Given the PostgreSQL source contains a customer record
  And the customer_name field contains "O'Brien & Associates <Ltd>"
  When the transformation step processes the record
  Then the record should be cleaned and loaded successfully
  And the stored name should be "O'Brien & Associates Ltd"
  And no exception should be thrown during transformation
```

---

### 6. Ordering and Sequencing

| Question | Example |
|---|---|
| What if events arrive out of order? | Late event after a newer one |
| What if a delete event arrives before a create event? | Event ordering not guaranteed |
| What if a batch job runs twice in the same window? | Overlapping schedules |

```gherkin
Scenario: Customer-deleted event arrives before customer-created event
  Given a customer-deleted event for customerId "C-99999" is on the topic
  And no customer record with id "C-99999" exists in the database
  When the consumer processes the delete event
  Then no action should be taken (record does not exist to delete)
  And the event should be acknowledged
  And a warning log should be written noting the orphaned delete event

Scenario: ETL job triggered twice within the same scheduling window
  Given the ETL job for the current day has already run successfully
  When a second trigger fires within the same day
  Then the job should detect the completed run and exit gracefully
  And it should log "Skipped - run already completed for this window"
  And no data should be duplicated in the warehouse
```

---

## The Edge Case Checklist

Run through these for every story in refinement:

- [ ] What if a required field is null or missing?
- [ ] What happens at the payload size / record count limit?
- [ ] What if the same event or request is processed twice?
- [ ] What if a downstream service is unavailable?
- [ ] What if the data doesn't match the expected schema?
- [ ] What if events arrive out of order?
- [ ] What if the job runs twice in the same window?
- [ ] What does the consumer do when it cannot process a message?
- [ ] What does the pipeline do when source data is partially invalid?
- [ ] What happens during a retry — is it idempotent?

---

## Module 6 Key Takeaways

- Edge cases in distributed systems are not "nice to have" — they are where incidents happen
- Use the taxonomy: null/empty, boundary, concurrency, unavailability, data quality, ordering
- Run the checklist in every refinement session
- Each edge case is a separate scenario — do not squeeze multiple into one
- Idempotency is a behaviour — specify it explicitly

---

# Module 7: Behaviour Patterns for Your Stack

## 7.1 REST API Patterns

### CRUD operations

```gherkin
Scenario: Create a new customer — success
  Given no customer with email "jane@example.com" exists
  When a POST request is made to /api/v1/customers with valid customer data
  Then the response status should be 201 Created
  And the response body should contain the new customerId
  And a customer record should be created in the database with status "Active"
  And a customer-created event should be published to the customer-events Kafka topic

Scenario: Create a customer with duplicate email
  Given a customer with email "jane@example.com" already exists
  When a POST request is made to /api/v1/customers with the same email
  Then the response status should be 409 Conflict
  And the response body should contain "A customer with this email already exists"
  And no new customer record should be created
  And no Kafka event should be published

Scenario: Get a customer that does not exist
  Given no customer with id "C-99999" exists in the database
  When a GET request is made to /api/v1/customers/C-99999
  Then the response status should be 404 Not Found
  And the response body should contain "Customer not found"
```

### Authorisation patterns

```gherkin
Scenario: Standard user attempts admin action
  Given I am authenticated as a standard user
  When a DELETE request is made to /api/v1/customers/C-12345
  Then the response status should be 403 Forbidden
  And the response body should contain "Insufficient permissions"
  And the customer record should remain unchanged in the database

Scenario: Request with expired JWT token
  Given the request contains a JWT token that expired 5 minutes ago
  When any authenticated API endpoint is called
  Then the response status should be 401 Unauthorized
  And the response body should contain "Token expired"
```

---

## 7.2 Kafka / Event-Driven Patterns

### Producer patterns

```gherkin
Scenario: Publish event on successful resource creation
  Given a valid customer creation request has been processed
  When the customer record is successfully saved to the database
  Then a customer-created event should be published to the customer-events topic
  And the event should contain customerId, email, status, and created_at
  And the event key should be the customerId for partition routing

Scenario: Do not publish event when creation fails
  Given a customer creation request fails validation
  When the API returns a 400 response
  Then NO event should be published to the customer-events topic
```

### Consumer patterns

```gherkin
Scenario: Successfully process a valid event
  Given a customer-updated event is on the customer-events topic
  And the event schema is valid and all required fields are present
  When the enrichment service consumes the event
  Then the customer record should be updated in the enrichment database
  And the event should be acknowledged (offset committed)
  And a processing-complete log entry should be written

Scenario: Message processing fails — retry behaviour
  Given a valid event is on the topic
  And the enrichment database is temporarily unavailable
  When the consumer attempts to process the event
  Then the event should NOT be acknowledged
  And the consumer should pause and retry after 30 seconds
  And the failure should be logged with the event payload and error reason

Scenario: Event schema is invalid — dead-letter queue
  Given a malformed event arrives on the customer-events topic
  And the payload cannot be deserialised
  When the consumer attempts to process the event
  Then the event should be published to the customer-events-dlq topic
  And an alert should be raised with the raw payload
  And the consumer should acknowledge the original message and continue
```

---

## 7.3 ETL / Data Pipeline Patterns

### Extraction patterns

```gherkin
Scenario: Full extraction from PostgreSQL source
  Given the source PostgreSQL table has 200,000 customer records
  When the full-load ETL job runs
  Then all 200,000 records should be extracted
  And the extracted record count should be logged
  And extraction should complete within the 30-minute SLA

Scenario: Incremental extraction — changed records only
  Given the last successful run timestamp is stored as "2024-01-15 02:00:00 UTC"
  And 5,000 records have been updated since that timestamp
  When the incremental ETL job runs
  Then only the 5,000 changed records should be extracted
  And the new run timestamp should be stored on completion
  And the extraction log should show "5,000 records extracted (incremental)"
```

### Transformation patterns

```gherkin
Scenario: Transform and enrich customer record — success
  Given a raw customer record has been extracted from PostgreSQL
  And the record has valid values in all required fields
  When the transformation step runs
  Then the phone number should be normalised to E.164 format
  And the email address should be lowercased
  And a derived full_name field should be created from first_name and last_name
  And the transformed record should be written to the staging table

Scenario: Record fails transformation — quarantine
  Given a raw customer record has been extracted
  And the date_of_birth field contains "not-a-date"
  When the transformation step attempts to parse the date
  Then the record should be written to the quarantine table with reason "Invalid date_of_birth format"
  And the record should NOT be written to the staging table
  And the transformation job should continue processing remaining records

Scenario: High-volume transformation — performance
  Given the staging table contains 1,000,000 records to transform
  When the transformation job runs
  Then all records should be processed within 2 hours
  And memory usage should remain below 80% of the ECS task allocation
  And the job should log progress every 100,000 records
```

### Loading patterns

```gherkin
Scenario: Load transformed records — upsert behaviour
  Given the warehouse customers table already contains a record for customerId "C-12345"
  And the staging table contains an updated version of the same record
  When the load step runs
  Then the existing warehouse record should be updated (not duplicated)
  And the updated_at timestamp should reflect the current load time
  And the record count in the warehouse should remain unchanged

Scenario: Load job detects record count anomaly
  Given the previous successful load processed 50,000 records
  And the current staging table contains only 1,000 records
  When the load job runs
  Then the job should pause and raise an alert
  And the alert should state "Record count anomaly: expected ~50,000, found 1,000"
  And no records should be loaded until the anomaly is acknowledged
```

---

## 7.4 PostgreSQL High-Volume Replication Patterns

```gherkin
Scenario: Replication lag exceeds threshold
  Given replication from the PostgreSQL primary to the read replica is running
  When the replication lag exceeds 5 minutes
  Then a CloudWatch alarm should be triggered
  And an alert should be sent to the data platform Slack channel
  And the alert should include current lag in seconds and affected tables

Scenario: Replication resumes after primary failover
  Given the PostgreSQL primary instance has failed over to the standby
  And replication was interrupted for 8 minutes during failover
  When the new primary is available and replication resumes
  Then all records written during the failover window should be replicated
  And the replication lag should return to under 30 seconds within 15 minutes
  And a recovery-complete log entry should be written

Scenario: Schema migration on source — column added
  Given a new nullable column "loyalty_tier" is added to the source customers table
  When the next replication cycle runs
  Then the replication should complete successfully
  And the new column should appear in the replicated table with null values
  And no existing records should be affected
  And an alert should be sent noting the schema change detected
```

---

## Module 7 Key Takeaways

- REST API stories: always cover creation success, duplicate/conflict, not-found, and authorisation
- Kafka producer stories: specify when to publish AND when not to publish
- Kafka consumer stories: success, retry behaviour, and dead-letter queue are all mandatory
- ETL stories: cover extraction, transformation (including quarantine), and load — separately
- Replication stories: lag thresholds, failover recovery, and schema changes need explicit scenarios

---

# Module 8: SAFe and Kanban — Stories at the Right Level

## 8.1 The Story Hierarchy in SAFe

In SAFe your team works at multiple levels. Behaviour specification applies differently at each:

```
Epic
  └── Feature
        └── Story (scenarios live here)
              └── Task (implementation steps — no scenarios needed)
```

**Epic** — a large initiative, often spanning multiple PI cycles. Too big for scenarios directly.
> *"Modernise the customer data platform"*

**Feature** — a deliverable capability within a PI. Described with a benefit hypothesis and acceptance criteria.
> *"Real-time customer profile enrichment from Kafka events"*

**Story** — a specific, independently deliverable behaviour. This is where scenarios are written.
> *"As a downstream service, I want to receive an enriched customer profile event so that I can personalise the customer experience without a synchronous API call"*

**Task** — implementation steps inside a story. No scenarios needed.
> *"Create Kafka consumer class, configure dead-letter queue, write unit tests"*

---

## 8.2 Writing Stories That Are INVEST-Ready

In a SAFe/Kanban hybrid, stories need to be ready to flow. The INVEST criteria help:

| Letter | Criterion | What it means for your stack |
|---|---|---|
| **I** | Independent | Deliverable without depending on another incomplete story |
| **N** | Negotiable | Details can be discussed — not a contract |
| **V** | Valuable | Delivers value to a consumer (service, user, or downstream system) |
| **E** | Estimable | Team can size it — not too vague, not too large |
| **S** | Small | Completable within one sprint or Kanban cycle |
| **T** | Testable | Has behaviour scenarios that can be verified |

**Common INVEST failures in microservices teams:**

```
❌ Story: Build the entire order processing pipeline
   (Not Independent, Not Small, Not Estimable)

✅ Story 1: Publish order-placed event to Kafka on successful order creation
✅ Story 2: Consume order-placed event and validate order data
✅ Story 3: Enrich order with customer profile data from enrichment service
✅ Story 4: Write enriched order to the warehouse staging table
```

Four independent stories. Each has its own scenarios. Each can be deployed separately.

---

## 8.3 Enabler Stories

SAFe includes Enabler stories — technical work that supports future features. These also need behaviour scenarios.

```gherkin
Story: Set up dead-letter queue handling for the order consumer

Scenario: Dead-letter queue handler processes a failed message
  Given a message has been moved to the orders-dlq queue
  When the DLQ handler Lambda is triggered
  Then the message payload should be logged to CloudWatch with full context
  And an alert should be sent to the on-call engineer
  And the message should be moved to an S3 archive bucket for investigation
  And the original message should be deleted from the DLQ

Scenario: DLQ exceeds threshold message count
  Given the orders-dlq queue contains more than 100 messages
  When the CloudWatch metric alarm evaluates
  Then a P2 PagerDuty alert should be raised
  And the alert should include current message count and oldest message age
```

Enabler stories without scenarios get built once and never validated. The behaviour gets assumed. The assumption becomes a bug.

---

## 8.4 Definition of Ready — Scenarios as a Gate

Before a story enters development in your Kanban flow, it should meet a Definition of Ready. Testable scenarios are the gate:

**Definition of Ready checklist:**
- [ ] Story has a "so that" — the value is clear
- [ ] At least one scenario for the happy path
- [ ] At least one scenario for each identified error path
- [ ] All red questions from Example Mapping are resolved
- [ ] Dependencies on other services or teams are identified
- [ ] Non-functional requirements (SLA, performance) have outcome clauses
- [ ] Story is independently deliverable (INVEST check)

A story without scenarios is not ready. It is an assumption waiting to become a bug.

---

## 8.5 PI Planning — Using Outcome Clauses to Surface Cross-Team Dependencies

In PI Planning, teams identify dependencies between Agile Release Trains. Well-specified outcome clauses help surface these before they become blockers.

**Example: Order enrichment story depends on customer profile service**

```gherkin
Scenario: Enrich order with customer profile
  Given an order-placed event has been validated
  When the enrichment service calls GET /api/v1/customers/{customerId}
  Then the response should include loyalty_tier, preferred_channel, and segment
```

This Then clause immediately reveals a dependency: the customer profile API must return `loyalty_tier`, `preferred_channel`, and `segment`. If the customer team's PI plan does not include adding these fields, this story cannot be completed. Discovered in PI Planning: a conversation. Discovered in sprint week 3: a blocker.

---

## Module 8 Key Takeaways

- Behaviour scenarios live at the Story level — not Epic or Feature, and not at Task level
- INVEST criteria are the flow gate — stories without scenarios fail the Testable criterion
- Enabler stories need behaviour scenarios too — technical behaviour must be specified and verifiable
- Definition of Ready should require at least happy path + error path scenarios
- PI Planning: use outcome clauses to surface cross-team API and event contract dependencies early

---

# Module 9: Accelerating with AI Assistance

## 9.1 Why a Prompt Library

Your team now understands behaviour specification. The next question is: how do you apply it consistently across every story, every sprint, every team?

A prompt library is a set of reusable, curated prompts that generate, validate, and expand scenarios. Used with any AI assistant (Claude, Copilot, Cursor, or a future agent), they encode your team's standards into the generation process itself.

Think of it as a **shared brain for story writing** — one that knows your stack, your patterns, and your anti-patterns.

---

## 9.2 Prompt Design Principles

Every prompt in your library should:

1. **Specify the stack context** — .NET, AWS, Kafka, PostgreSQL, ECS
2. **State the output format** — behaviour scenarios in Gherkin syntax
3. **Name the patterns to follow** — happy path, error paths, edge cases
4. **Name the anti-patterns to avoid** — vague outcomes, silent success, implementation detail
5. **Be independently useful** — runnable without other prompts

---

## 9.3 Starter Prompts

### Prompt 1: Generate behaviour scenarios from a feature brief

```
You are a senior developer and BDD practitioner working on a .NET microservices 
platform on AWS using Kafka for event-driven communication and PostgreSQL for 
data storage.

Given the following feature brief:
[PASTE FEATURE BRIEF HERE]

Generate behaviour scenarios in Gherkin syntax covering:
1. The happy path
2. All error paths (invalid input, missing data, downstream unavailability)
3. Edge cases specific to distributed systems (null payloads, duplicate events, 
   retry behaviour, dead-letter queue handling)

Follow these rules:
- Given: describe system state (database, cache, queue) — not infrastructure assumptions
- When: one specific trigger (HTTP request, Kafka message, scheduled job, S3 event)
- Then: observable outcomes only (status codes, record counts, published events, 
  alerts raised) — no vague outcomes like "data should be saved"
- Include a scenario for what happens when the operation is performed twice (idempotency)
- Include a scenario for what happens when a downstream service is unavailable
```

---

### Prompt 2: Expand edge cases from a happy path

```
You are a QA engineer and BDD practitioner reviewing user stories for a .NET 
microservices team using Kafka, SQS, PostgreSQL, and AWS ECS.

Given this happy path scenario:
[PASTE HAPPY PATH SCENARIO]

Generate additional scenarios covering:
1. Null or missing required fields in the input/payload
2. Boundary conditions (maximum payload size, maximum record count)
3. Duplicate event or request processing (idempotency)
4. Downstream service unavailable (timeout, 503, connection refused)
5. Invalid or unexpected schema/format
6. Out-of-order event arrival
7. The operation running twice within the same time window

For each scenario, ensure the Then clause specifies:
- What is observable (status code, record count, event published, alert raised)
- What does NOT happen (no duplicate created, no data lost, consumer continues)
```

---

### Prompt 3: Validate a story for anti-patterns

```
Review the following user story and behaviour scenarios for anti-patterns.

[PASTE STORY AND SCENARIOS]

Check for and report on:
1. Vague Then clauses — "data should be saved", "user should be notified"
2. Silent success anti-pattern — Then passes even when data is lost or dropped
3. Implementation detail in scenarios — API endpoints, database column names, 
   infrastructure config
4. Missing error paths — only happy path covered
5. Missing idempotency scenario
6. Missing downstream unavailability scenario
7. When clause with multiple actions
8. Given clause with environmental assumptions (infrastructure running, etc.)
9. Missing "so that" on the story headline
10. Story too large — more than 8 scenarios suggests splitting

For each issue found, provide the corrected version.
```

---

### Prompt 4: ETL / data pipeline story generator

```
You are a data engineer and BDD practitioner working on ETL pipelines that 
extract from PostgreSQL, transform data, and load into a data warehouse.

Given the following pipeline requirement:
[PASTE PIPELINE REQUIREMENT]

Generate behaviour scenarios covering:
2. Successful incremental load — only changed records processed
3. Source unavailable — job fails explicitly, does not report success
4. Records fail validation — quarantined, not silently dropped
5. Record count anomaly — significantly fewer records than expected
6. Duplicate job trigger — second run skipped or handled safely
7. Schema change detected in source — handled gracefully
8. Performance scenario — completes within defined SLA

Rules:
- Then clauses must include record counts, not just status
- "Pipeline completed successfully" is NOT a valid Then clause unless 
  accompanied by a record count assertion
- Every data loss path must have an explicit Then clause stating what 
  was preserved and what was quarantined
```

---

### Prompt 5: Kafka consumer story generator

```
You are a senior .NET developer and BDD practitioner building Kafka consumers 
on AWS ECS.

Given the following consumer requirement:
[PASTE CONSUMER REQUIREMENT]

Generate behaviour scenarios covering:
2. Invalid schema — moved to dead-letter queue, consumer continues
3. Null required field — explicit handling, not NullReferenceException
4. Duplicate message (replay) — idempotent handling
5. Downstream service unavailable — retry with backoff, eventual DLQ
6. Consumer restarts mid-processing — no messages lost or duplicated
7. Message exceeds processing time limit — timeout handling
8. DLQ threshold exceeded — alert raised

Rules:
- Always specify whether the message is acknowledged or not acknowledged
- Always specify what happens to the consumer after handling (continues / pauses / stops)
- Dead-letter queue behaviour must always be an explicit scenario
- "Processing failed" is not a Then clause — specify what observable state results
```

---

## Module 9 Key Takeaways

- A prompt library encodes your team's standards into AI-assisted story writing
- Each prompt should specify stack context, output format, patterns, and anti-patterns
- Start with five prompts: general generation, edge case expansion, validation, ETL, Kafka
- Prompts are living documents — update them when new patterns or anti-patterns are found
- These prompts are the foundation for a behaviour specification agent

---

# Course Summary

## The Behaviour Specification Format

```gherkin
As a [who — a person or consuming service]
I want to [what — a specific behaviour]
So that [why — the value or outcome]

Scenario: [name of this specific behaviour]
  Given [system state — database, cache, queue, service health]
  When  [one specific trigger — HTTP, Kafka, cron, S3, SQS]
  Then  [observable outcome — status code, record count, event published, alert raised]
```

## The Thought Process

```
1. Start with the pain — what has gone wrong without this scenario?
2. Run Example Mapping — find the rules, examples, and questions
3. Resolve red questions — before development starts
4. Define the state — what is the system state before the trigger?
5. Identify the trigger — what is the single, specific event or action?
6. Specify the outcome — what can be observed and verified?
7. Run the edge case checklist — null, boundary, concurrency, unavailability, schema, ordering
8. Check for anti-patterns — silent success, vague outcomes, missing idempotency
9. Validate INVEST — is this story independently deliverable and testable?
```

## The Edge Case Checklist

- [ ] Required field is null or missing
- [ ] Payload at / over / under size limit
- [ ] Same event or request processed twice
- [ ] Downstream service unavailable
- [ ] Data does not match expected schema
- [ ] Events arrive out of order
- [ ] Job or process runs twice in same window
- [ ] Consumer restarts mid-processing
- [ ] Source data is partially invalid
- [ ] Retry — is the operation idempotent?

## The Anti-Pattern Red Flags

| Red Flag | Fix |
|---|---|
| Outcome: "data should be saved" | Specify table, record count, field values |
| Outcome: "pipeline completed successfully" | Add record count assertion |
| Outcome: "user should be notified" | Specify channel, content, recipient |
| Trigger: two actions in one clause | Split into two scenarios |
| State: "infrastructure is running" | Remove — it is an assumption not a condition |
| No error path scenarios | Add unavailability, invalid input, duplicate |
| No idempotency scenario | Add "given already processed, when processed again" |
| Story has 15 scenarios | Split the story |

## The Prompt Library (Starting Five)

1. **General scenario generator** — from any feature brief
2. **Edge case expander** — from a happy path scenario
3. **Anti-pattern validator** — review and fix existing stories
4. **ETL / pipeline generator** — data engineering patterns
5. **Kafka consumer generator** — event-driven patterns

---

*This course is a living document. Add new patterns as your stack evolves. Update the prompt library when new anti-patterns are found. The goal is a shared language for behaviour — one that prevents incidents, not just describes features.*
