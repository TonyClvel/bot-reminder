# Technical Plan: Personal Telegram Reminders MVP

## Purpose and scope

This document describes a small, testable design for the behavior in `spec.md`. It is a plan, not an implementation. It defines module responsibilities, the Telegram-to-application flow, conceptual pseudocode, technical decisions, and a test strategy. Every functional requirement (RF) is mapped to the parts that cover it.

The MVP has one authorized Telegram account. It accepts natural-language requests for one-time and recurring reminders, interprets dates and times in UTC, asks for clarification when required information is missing or ambiguous, persists reminders across restarts, and retries failed deliveries.

## Proposed module structure

Keep the implementation in one language and one deployable application. Organize modules by responsibility rather than creating separate services.

| Module | Responsibility | RF coverage |
|---|---|---|
| Telegram adapter | Receive Telegram updates, extract sender and message, send Spanish replies and reminders. It must not contain reminder rules or interpret schedules. | RF-1, RF-5, RF-7 |
| Access policy | Compare the update's sender account identifier with the configured authorized identifier before passing user content into the reminder flow. | RF-1 |
| Conversation coordinator | Route authorized messages, maintain any pending clarification context, invoke parsing and domain operations, and render Spanish responses. | RF-2, RF-3, RF-4, RF-5 |
| Request interpreter | Convert natural-language input into a proposed reminder intent, including task, one-time or recurring schedule, and UTC date/time. Return missing or ambiguous fields for clarification; do not persist data. | RF-2, RF-3, RF-4, RF-5 |
| Reminder domain | Validate complete intents and define reminder state transitions, due occurrences, and recurrence behavior independent of Telegram. | RF-2, RF-3, RF-4, RF-7 |
| Reminder repository | Store and retrieve reminder definitions, delivery state, and retry state in durable storage. | RF-2, RF-3, RF-6, RF-7 |
| Delivery worker | Find due occurrences, send them through a delivery interface, record outcomes, and schedule retry attempts according to the agreed retry policy. | RF-6, RF-7 |
| Configuration | Provide the authorized Telegram account identifier and integration credentials to the application without embedding secrets in source or user-facing messages. | RF-1 |

The delivery worker and conversation coordinator use interfaces for persistence and Telegram delivery. The reminder domain and request interpreter must be testable without the Telegram adapter, in accordance with the constitution.

## Telegram and application interaction

1. The user starts a private conversation with the Telegram bot. Telegram bots cannot initiate a conversation with an account that has not first interacted with them; therefore, the authorized user must start the bot before it can send reminders.
2. The Telegram adapter receives an update and extracts the sender identifier and message text.
3. The access policy checks the sender identifier before processing the message. An unauthorized sender is not passed to the coordinator; no reminder is created and no reminder is sent to that sender.
4. For the authorized sender, the coordinator checks whether the message is an answer to a pending clarification. Otherwise it sends the message to the request interpreter.
5. The interpreter returns either a complete proposed intent or a description of the missing/ambiguous information. The coordinator asks a Spanish clarification question when needed and does not save an incomplete intent.
6. For a complete intent, the coordinator validates it through the reminder domain and saves it through the repository. It responds in Spanish. Whether a confirmation is required remains an open specification question.
7. Independently of incoming user messages, the delivery worker reads due occurrences from durable storage and sends them only to the configured authorized account through the Telegram adapter.
8. The worker records each outcome. On failure it retries and, when retrying has been exhausted, sends a failure notice to the authorized account. The exhaustion condition and retry timing must be decided before implementation.

## Conceptual bot pseudocode

The following is descriptive pseudocode only; it does not prescribe a programming language or implementation syntax.

### Incoming Telegram update

    WHEN a Telegram update arrives:
        extract sender identifier and message
        IF sender identifier is not the configured authorized identifier:
            deny reminder functionality
            do not create or send a reminder for this sender
            stop processing this update

        IF the authorized user has a pending clarification:
            combine the reply with the pending request
            interpret the resulting request
        ELSE:
            interpret the message

        IF required task or schedule details are missing or ambiguous:
            keep the request uncommitted as clarification context
            ask a concise clarification question in Spanish
            stop processing this request

        validate the complete intent using reminder-domain rules
        save the reminder and its schedule durably
        respond in Spanish

### Due reminder processing

    REPEATEDLY, including after application restart:
        load due reminder occurrences from durable storage
        FOR each due occurrence:
            IF the occurrence is already recorded as delivered:
                skip it
            ELSE:
                attempt delivery through the Telegram adapter
                IF delivery succeeds:
                    record the occurrence as delivered
                    advance or complete its schedule
                ELSE:
                    record the failure and retry state
                    IF the agreed retry policy permits another attempt:
                        make the occurrence eligible for that retry
                    ELSE:
                        record final failure
                        notify the authorized user in Spanish

The persisted occurrence and delivery state are intended to prevent a restart from silently losing work and to reduce duplicate sends. Exact delivery guarantees depend on Telegram's response semantics and remain to be specified.

## Technical decisions

| Decision | Rationale | Alternative considered and rejected |
|---|---|---|
| Use one language and one deployable application for the MVP. | Minimizes operational and cognitive complexity in line with the constitution's simple-stack principle. The exact language remains undecided because the repository has no established stack. | Separate services or multiple languages; rejected for adding deployment and integration complexity without a demonstrated need. |
| Keep reminder rules and schedule interpretation outside the Telegram adapter. | Allows functional behavior to be tested without a real Telegram conversation and keeps the interface focused on receiving input and presenting output. | Put parsing, scheduling, and delivery rules in Telegram handlers; rejected because it couples core behavior to the interface and makes isolated tests harder. |
| Use durable storage as the source of truth for reminders and delivery state. | The bot must preserve reminders and configuration over restarts (RF-6); due work and retries must also survive restarts (RF-7). | Keep pending reminders only in process memory; rejected because all pending work would be lost on restart. |
| Store times and evaluate schedules in UTC. | This is the explicit time interpretation rule in RF-5 and avoids implicit dependence on the host's local timezone. | Use the host's local timezone or infer a timezone from Telegram; rejected because the specification explicitly selects UTC. |
| Gate all reminder operations and destinations by one configured Telegram account identifier. | Enforces the single-user scope at both inbound and outbound boundaries (RF-1). | Allow any private-chat user or select the first user automatically; rejected because either would change the specified authorization policy. |
| Treat an interpreted request as a proposal until all required fields are unambiguous. | Prevents incomplete or guessed schedules from being persisted, as required by RF-4. | Save a best-effort interpretation and ask for correction afterward; rejected because that can create unintended reminders. |
| Process due work from persistent state and record delivery outcomes. | Makes pending work recoverable after restart and provides a basis for retrying failures (RF-6, RF-7). | Use only an in-memory timer per reminder; rejected because timers do not survive process restarts. |
| Keep user-facing bot messages in Spanish. | Follows the language requirement in RF-5 and the constitution. | Reply in the language of each incoming request; rejected because the defined user-facing language is Spanish. |

## Test strategy

All tests are automated. Core tests must not depend on a live Telegram conversation. Use a fake clock, an in-memory test repository, and a fake delivery interface where appropriate; these are test doubles, not production persistence.

| Test level | Scope and verification | RF coverage |
|---|---|---|
| Unit: access policy | Authorized sender is accepted; any other sender cannot invoke reminder operations or become a delivery destination. | RF-1 |
| Unit: request interpretation and domain validation | Valid one-time and recurring intents are represented in UTC; missing or ambiguous task/schedule fields produce clarification requirements and cannot be persisted. | RF-2, RF-3, RF-4, RF-5 |
| Unit: conversation coordinator | Clarification flow does not commit an incomplete reminder; messages generated by the bot are Spanish; complete intents reach persistence. | RF-2, RF-3, RF-4, RF-5 |
| Unit: persistence contract | A saved reminder, its schedule, and delivery state can be loaded after creating a new repository/application instance. | RF-2, RF-3, RF-6 |
| Unit: delivery worker | Due occurrences are sent only to the authorized account; success is recorded; failure updates retry state; final failure produces a Spanish notice. | RF-1, RF-6, RF-7 |
| Integration: application with fake Telegram and real test storage | Exercise inbound update → access check → interpretation/clarification or save → due delivery, without connecting to Telegram's live service. | RF-1 through RF-7 |
| Restart/recovery test | Persist a pending reminder, recreate the application, and verify the pending occurrence remains eligible for delivery at the correct UTC time. | RF-6, RF-7 |
| Contract test: Telegram adapter | Verify update fields are mapped to the application boundary and outbound messages use the supplied authorized destination. Keep this separate from core-domain tests. | RF-1, RF-5, RF-7 |

Before integrating a change, run the relevant automated tests and require them to pass, as required by the constitution. Add a regression test for each defect fix.

## Requirement coverage summary

| Requirement | Planned coverage |
|---|---|
| RF-1 — Exclusive access | Access policy, configured account identifier, destination restriction, and unit/adapter contract tests. |
| RF-2 — One-time reminder | Request interpreter, domain validation, durable repository, coordinator, and acceptance/integration tests. |
| RF-3 — Recurring reminder | Request interpreter, recurring domain schedule, durable repository, delivery worker, and acceptance/integration tests. |
| RF-4 — Clarify incomplete or ambiguous request | Interpreter returns missing/ambiguous fields; coordinator asks before persistence; unit and integration tests assert no premature save. |
| RF-5 — UTC and Spanish communication | Interpreter/domain time handling, coordinator/adapter user-facing messages, and UTC/language tests. |
| RF-6 — Persistence over restart | Durable repository and restart/recovery test for reminder definitions, configuration, and pending delivery state. |
| RF-7 — Send and retry | Due worker, Telegram delivery boundary, recorded outcome/retry state, final-failure notice, and delivery-worker tests. |

## Decisions and specification items required before implementation

The following behavior is unresolved in `spec.md`; this plan intentionally does not invent product rules:

- [NEEDS CLARIFICATION] How the authorized Telegram account identifier is provisioned and linked.
- [NEEDS CLARIFICATION] Which recurring patterns, end conditions, and recurrence boundaries are supported.
- [NEEDS CLARIFICATION] What happens when a requested date/time is already in the past.
- [NEEDS CLARIFICATION] Retry limit, retry interval, and the condition that constitutes final delivery failure.
- [NEEDS CLARIFICATION] Whether reminders can be listed, edited, cancelled, or paused, and how users request those actions.
- [NEEDS CLARIFICATION] Whether successful creation or delivery requires a confirmation message.
- [NEEDS CLARIFICATION] Whether reminder text must be preserved exactly as submitted.
- [NEEDS CLARIFICATION] Whether the plan's assumed prerequisite that the authorized user starts the Telegram bot is acceptable for the MVP.
- [NEEDS CLARIFICATION] The exact language/runtime and durable storage technology; choose the simplest option consistent with the existing project once a stack is established.

Until these items are settled, their related behavior must not be treated as implementation-ready or as a fully verifiable acceptance condition.
