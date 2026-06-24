# github-contribution-log

# Contribution 1: Break out request parsing to a standalone function and unit test #1298

**Contribution Number:** 1  
**Student:** Amy Wang  
**Issue:** [GitHub issue link](https://github.com/near/mpc/issues/1298)  
**Status:** Phase 3 Complete

---

## Why I Chose This Issue

I chose this issue because I wanted to work on a beginner-friendly contribution with a clearly defined scope. The maintainer has confirmed that the issue is still relevant, nice-to-have, straightforward to implement, and has added the `good first issue tag`. The issue is also fairly recent, with the latest comments being last month even though the issue was opened 8 months ago. 

I am interested in delving into understanding and working in a large Rust codebase, as I learn the project's architecture, workflow, testing practices, etc. I also want to understand HTTP request handling and parsing in Rust. Lastly, since this issue is a refactoring issue and is labeled as `technical debt`, I want to ensure I learn the best practices for writing code and unit tests.

---

## Understanding the Issue

### Problem Description

The migration service sends and receives key data through a small web API. Request/response parsing lives inside the HTTP handler and client functions, mixed with networking code. That makes it harder to test parsing in isolation without having to set up TLS connections and a full web server.

### Expected Behavior

The parsing logic should live in standalone functions that can be unit tested with plain input/output (no web server or network). Unit tests should contain success and error cases. 

### Current Behavior

Right now, parsing is inline in the HTTP layer. On the server, `handle_request` reads the request, parses it, and builds the response in one place. On the client, `make_keyshare_get_request` build requests, sends them, and parses responses in one place. Integration tests in `web.rs` cover full flows but are slower because  they start a real server and client, and don’t focus on parsing edge cases on their own. 

### Affected Components

`crates/node/src/migration_service/web/server.rs`: server, where incoming requests are handled
`crates/node/src/migration_service/web/client.rs`: client, where outgoing requests are built and responses are read
`crates/node/src/migration_service/web/serialization.rs`: already extracted helpers as reference
`crates/node/src/migration_service/web.rs`: existing full-flow tests (should keep passing)
`docs/engineering-standards.md`: explains why separating parsing from I/O is preferred

---

## Reproduction Process

### Environment Setup

Need to install Rust:

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

and cargo-nextest:

`cargo install cargo-nextest --locked`

Both installations were fast and easy.

### Steps to Reproduce

1. In `crates/node/src/migration_service/web/server.rs`, find handle_request.
2. In `crates/node/src/migration_service/web/client.rs`, find make_keyshare_get_request and make_set_keyshares_request.
3. Observed result: Parsing logic lives in HTTP handlers/client functions.
4. In `crates/node/src/migration_service/web.rs`, find full-flow tests.
5. Observed result: No fast, isolated unit tests for HTTP request/response parsing. Only full-flow tests that require server setup exist.

### Reproduction Evidence

- **Commit showing reproduction:** N/A, no bug to reproduce
- **Screenshots/logs:** Commands ran output some warnings, my computer became very hot, took about 5 minutes to run both
- First integration test run takes 30+ minutes to run, subsequent runs are faster

Migration Web test list command:

`cargo nextest list --cargo-profile=test-release -p mpc-node migration_service::web`

Output:

<img width="793" height="663" alt="Screenshot 2026-06-17 at 12 20 17 AM" src="https://github.com/user-attachments/assets/13c2a42d-6d33-4b6c-a59b-9de921bc20f4" />
<img width="788" height="683" alt="Screenshot 2026-06-17 at 12 20 43 AM" src="https://github.com/user-attachments/assets/a83923e0-89df-41d6-8474-ee5407cb8715" />


Serialization fast unit test command:

`cargo nextest run --cargo-profile=test-release -p mpc-node test_encrypt_decrypt_cycle`

Output:

<img width="831" height="209" alt="Screenshot 2026-06-17 at 12 18 00 AM" src="https://github.com/user-attachments/assets/a65e9e77-b179-4444-a8c5-940e47d9a57b" />

- **My findings:** test_encrypt_decrypt_cycle shows lower-level parsing already has a unit test. No dedicated unit tests exist yet for HTTP request/response body parsing in server.rs / client.rs

---

## Solution Approach

### Analysis

HTTP body parsing/building for the API is mixed with the networking code in `server.rs` and `client.rs`.

### Proposed Solution

Separate parsing logic from the rest of the networking code into its smaller helper functions. Write unit tests for success and error cases.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Request/response body parsing for `/get_keyshares` and `/set_keyshares` should be testable without starting a web server or network connection.

**Match:** Follow the pattern in `serialization.rs` and `encryption.rs`- pure functions + `#[cfg(test)]` module with unit tests. Follow `docs/engineering-standards.md` for separating I/O from logic and test structure (`<system>__should_<assertion>`).

**Plan:** 
1. Add `crates/node/src/migration_service/web/request.rs` (or extend` serialization.rs` if maintainers prefer) with helpers.
2. Register the module in `web.rs`.
3. Refactor `server.rs` `handle_request` to read the HTTP body, then call parse helpers; keep storage/channel/response logic in the handler.
4. Refactor `client.rs` `make_keyshare_get_request` and `make_set_keyshares_request` to call build/parse helpers; keep networking (send request, check status) in the client functions.
5. Add unit tests in `request.rs` for valid input, invalid JSON, bad encryption, empty body.
6. Run existing `migration_service::web` integration tests to confirm no behavior change.

**Implement:** [Link to your branch/commits as you work]

**Review:** 

- [x] Parsing/building logic is in sync helpers with no network/async
- [x] New tests use `Given / When / Then` and `__should_` naming
- [x] No unnecessary comments (per engineering standards)
- [x] `cargo fmt` and `cargo clippy pass`
- [x] Existing integration tests in `web.rs` still pass
- [x] Scope limited to `/get_keyshares` and `/set_keyshares` (no change to /hello)

**Evaluate:** New unit tests pass in isolation; `cargo nextest run --cargo-profile=test-release -p mpc-node migration_service::web` passes; manual spot-check optional (see Testing Strategy).

---

## Testing Strategy

### Unit Tests

- [x] Test case 1: Valid keyset JSON parses successfully (GET /get_keyshares)
- [x] Test case 2: Invalid JSON returns an error
- [x] Test case 3: Valid encrypted keyshares body parses successfully (PUT /set_keyshares)

### Integration Tests

- [x] Existing migration_service::web tests still pass (no new integration tests required)

### Manual Testing

Ran locally:
- [x] new unit tests: `cargo nextest run --cargo-profile=test-release -p mpc-node migration_service::web::request`
- [x] existing integration tests: `cargo nextest run --cargo-profile=test-release -p mpc-node migration_service::web`

---

## Implementation Notes

### Week 2 Progress

- Chose issue #1298 and documented understanding/reproduction
- Set up Rust + cargo-nextest
- Ran `cargo nextest list/run` for `migration_service::web`; confirmed integration tests exist and `serialization.rs` already has a unit test pattern to follow

### Week 3 Progress

- Added `request.rs` with parse/build helpers
- Refactored `server.rs` and `client.rs`
- Added unit tests
- Biggest challenge: understanding Rust syntax, compile time


### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

