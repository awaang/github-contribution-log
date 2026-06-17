# github-contribution-log

# Contribution 1: Break out request parsing to a standalone function and unit test #1298

**Contribution Number:** 1  
**Student:** Amy Wang  
**Issue:** [GitHub issue link](https://github.com/near/mpc/issues/1298)  
**Status:** Phase 2 Complete

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

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

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

