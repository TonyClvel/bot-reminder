# MVP Implementation Tasks

Tasks are ordered by dependency. Each task is estimated to take 15–25 minutes; split a task further if it exceeds 30 minutes. Resolve the specification blockers before implementing behavior they govern. Technical task descriptions are in English to follow the project constitution; the requested `Hecho cuando:` marker is retained.

## Specification decisions and preparation

- [ ] **T01 (20 min) — Define authorized-account onboarding.** RF: RF-1. Record how the authorized Telegram account identifier is supplied and verified.
  **Hecho cuando:** The specification states a single, testable account-linking procedure and the rejection behavior for every other account.
- [ ] **T02 (20 min) — Define supported recurring schedules.** RF: RF-3, RF-4. Specify supported recurrence patterns, boundaries, and end conditions.
  **Hecho cuando:** The specification lists accepted recurrence examples and identifies incomplete or unsupported patterns that require clarification.
- [ ] **T03 (15 min) — Define handling of past schedules.** RF: RF-2, RF-4, RF-5. Decide how dates and times already in the past are handled.
  **Hecho cuando:** The specification gives an observable expected result for both a past date and a time that has passed today.
- [ ] **T04 (20 min) — Define delivery retry policy.** RF: RF-7. Specify retry limit, timing, and final-failure condition.
  **Hecho cuando:** The specification defines when another attempt occurs and exactly when a final-failure notice is sent.
- [ ] **T05 (20 min) — Decide reminder management and confirmations.** RF: RF-2, RF-3, RF-7. Resolve whether users can list, edit, cancel, or pause reminders, and whether creation or delivery confirmations are required.
  **Hecho cuando:** The specification explicitly includes or excludes each action and states which confirmations the bot sends.
- [ ] **T06 (15 min) — Decide reminder-text preservation.** RF: RF-2, RF-3, RF-5. Define whether reminder text is preserved as submitted or normalized.
  **Hecho cuando:** The specification states the expected delivered text for an input containing punctuation, capitalization, or non-Spanish text.
- [ ] **T07 (20 min) — Select the simplest project stack and durable storage.** RF: RF-1, RF-6. Inspect the repository and select one implementation language and a persistence option consistent with the constitution.
  **Hecho cuando:** The plan records one language and one durable storage choice, each with a brief reason and no unnecessary additional dependency.
- [ ] **T08 (25 min) — Make acceptance criteria implementation-ready.** RF: RF-1–RF-7. Update the specification with the decisions from T01–T06 and make each criterion objectively testable.
  **Hecho cuando:** No behavior needed by an RF depends on an unresolved `[NEEDS CLARIFICATION]` item, and each RF has at least one observable acceptance check.
- [ ] **T09 (20 min) — Establish the minimal application and test baseline.** RF: RF-1–RF-7. Create the selected-language project entry point and configure a runnable automated-test command.
  **Hecho cuando:** The application starts in its minimal form and the automated test command completes successfully with at least one passing test.

## Reminder logic and user interaction

- [ ] **T10 (20 min) — Define and validate reminder intents.** RF: RF-2, RF-3, RF-4, RF-5. Represent task text and a complete one-time or recurring schedule independently of Telegram.
  **Hecho cuando:** Automated tests accept complete intents and reject intents missing a required task or schedule field without saving them.
- [ ] **T11 (20 min) — Implement UTC schedule interpretation.** RF: RF-2, RF-3, RF-5. Convert accepted date/time input into an unambiguous UTC schedule according to the finalized specification.
  **Hecho cuando:** Automated tests verify the expected UTC instant for each supported date/time form and the defined past-time behavior.
- [ ] **T12 (25 min) — Interpret one-time reminder requests.** RF: RF-2, RF-4, RF-5. Convert supported natural-language one-time requests into proposed intents without persistence.
  **Hecho cuando:** Tests cover valid task/date/time input and demonstrate that missing or ambiguous fields produce a clarification result, not a saved reminder.
- [ ] **T13 (25 min) — Interpret recurring reminder requests.** RF: RF-3, RF-4, RF-5. Convert the finalized recurring forms into proposed intents without persistence.
  **Hecho cuando:** Tests cover each supported recurrence form and verify that unsupported or incomplete patterns require clarification.
- [ ] **T14 (25 min) — Handle clarification conversations.** RF: RF-2, RF-3, RF-4, RF-5. Keep an incomplete request pending and apply the authorized user's clarification response to it.
  **Hecho cuando:** A test proves an incomplete request is not persisted, a sufficient answer completes it, and an insufficient answer prompts for clarification in Spanish.
- [ ] **T15 (20 min) — Enforce the single-user access policy.** RF: RF-1. Check the sender before processing user content and restrict all reminder destinations to the authorized account.
  **Hecho cuando:** Tests prove the authorized sender is accepted and an unauthorized sender cannot create, change, or receive reminders.
- [ ] **T16 (20 min) — Define Spanish response rendering.** RF: RF-4, RF-5, RF-7. Render clarification, creation, and final-failure messages in Spanish according to the settled confirmation policy.
  **Hecho cuando:** Automated tests assert the required Spanish text for each response type and ensure no response is generated for unauthorized senders.

## Telegram integration and persistence

- [ ] **T17 (20 min) — Add the inbound Telegram boundary.** RF: RF-1, RF-2, RF-3, RF-4. Map received Telegram updates to sender identity and message input without putting reminder rules in the adapter.
  **Hecho cuando:** Adapter contract tests show that updates reach the application boundary with the correct sender and text, while domain tests run without Telegram.
- [ ] **T18 (20 min) — Add the outbound Telegram boundary.** RF: RF-1, RF-5, RF-7. Provide a delivery interface that sends Spanish messages to the authorized account only.
  **Hecho cuando:** Contract tests verify the destination is always the configured authorized account and the core delivery tests use a fake sender.
- [ ] **T19 (25 min) — Persist reminder definitions and schedules.** RF: RF-2, RF-3, RF-6. Store complete reminder content and schedule using the selected durable storage.
  **Hecho cuando:** A test can save a one-time and a recurring reminder, create a new application/repository instance, and load both with their schedules intact.
- [ ] **T20 (25 min) — Calculate due occurrences.** RF: RF-2, RF-3, RF-5, RF-6. Determine which one-time and recurring reminder occurrences are due using UTC and the finalized recurrence rules.
  **Hecho cuando:** Fake-clock tests return exactly the expected due occurrences, including across a restart boundary, without returning future occurrences.
- [ ] **T21 (25 min) — Persist delivery and retry state.** RF: RF-6, RF-7. Record pending, delivered, and retry-eligible occurrence state durably.
  **Hecho cuando:** Tests prove delivery state and next eligible retry survive a new application/repository instance and delivered occurrences are not selected again.
- [ ] **T22 (25 min) — Implement reminder delivery and retry transitions.** RF: RF-1, RF-6, RF-7. Send due occurrences through the outbound boundary and apply the finalized retry policy on failure.
  **Hecho cuando:** Fake-delivery tests verify successful delivery is recorded once and failures become eligible for attempts at exactly the specified retry times.
- [ ] **T23 (20 min) — Notify the user of final delivery failure.** RF: RF-1, RF-5, RF-7. Send the required final-failure notice after the retry policy is exhausted.
  **Hecho cuando:** A test verifies one Spanish notice goes only to the authorized account after final failure, and no final notice is sent before then.
- [ ] **T24 (25 min) — Connect Telegram input, application logic, storage, and delivery.** RF: RF-1–RF-7. Wire the adapter, access policy, coordinator, interpreter, domain, repository, and delivery worker in the single application.
  **Hecho cuando:** An automated integration test with fake Telegram input and delivery creates a valid reminder and later delivers it without a live Telegram conversation.

## Verification

- [ ] **T25 (20 min) — Add domain and schedule regression tests.** RF: RF-2, RF-3, RF-4, RF-5. Cover valid one-time and recurring schedules, UTC interpretation, and clarification paths.
  **Hecho cuando:** Tests assert each supported schedule path and verify ambiguous or incomplete requests never reach persistence.
- [ ] **T26 (20 min) — Add access and conversation regression tests.** RF: RF-1, RF-4, RF-5. Cover authorized/unauthorized senders and clarification responses.
  **Hecho cuando:** Tests prove unauthorized updates cannot invoke reminder operations and authorized clarification interactions follow the specified Spanish responses.
- [ ] **T27 (20 min) — Add restart and persistence tests.** RF: RF-2, RF-3, RF-6. Restart the application with pending one-time and recurring reminders.
  **Hecho cuando:** Tests prove all pending definitions and due schedules remain available after restart and retain their UTC times.
- [ ] **T28 (20 min) — Add retry and delivery regression tests.** RF: RF-1, RF-5, RF-7. Cover transient failure, exhausted retries, and final notice behavior.
  **Hecho cuando:** Tests verify the configured number and timing of attempts, one final notice to the authorized account, and no duplicate successful delivery.
- [ ] **T29 (20 min) — Verify the complete RF acceptance matrix.** RF: RF-1–RF-7. Run the automated suite against the acceptance criteria and record the result for every RF.
  **Hecho cuando:** Every RF has a passing automated acceptance check, the full test command passes, and no acceptance check requires a live Telegram conversation.
