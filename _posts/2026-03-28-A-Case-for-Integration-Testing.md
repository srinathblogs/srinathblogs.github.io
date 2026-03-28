---
layout: post
title: "Integration Testing: Case Over Unit Testing"
date: 2026-03-28 12:00:00 +0000
author: srinath
tags: [design]
toc: true
---

## Tradeoffs in Software Design

Engineers who pick up most of their knowledge from online courses and technical blogs often treat deployment architecture as the main lever for maintaining velocity, scalability, and reliability. This leads to a lot of debate around design choices like:

- Microservices vs. Monoliths
- Clean architecture vs. Three-tier architecture
- REST vs. GraphQL

There is supporting truth behind each of these positions, but they depend heavily on context: your team size, your users, and how fast you actually need to ship.

That said, regardless of the architecture you pick, one problem shows up consistently. The product and engineering team gradually forgets the existing domain they have built, and the testing strategy you choose plays a significant role in how well you manage that.

---

## The Default Approach: Unit Testing Pyramid

Most projects default to the testing pyramid. The idea is straightforward:

- Write many unit tests
- Write some integration tests
- Write few end-to-end tests

This is widely accepted and still the norm across the industry. It keeps tests fast, isolated, and easy to reason about. The general advice is to test units of logic in isolation, mock external dependencies, and only test integrated behavior at higher levels sparingly.

But if you step back and think about it from a different angle, there are two scenarios where this model starts to feel limiting.


## When Integration Tests Make More Sense

### TDD Without Thinking About Internals

If you are developing in a TDD fashion, you often want to stay focused on behavior rather than internal structure. Integration tests let you do that. You write a test against the external interface, and you are free to refactor the internals during the Red, Green, Refactor cycle without rewriting tests every time you move code around.

Unit tests, by contrast, tend to get tied to implementation details. When you restructure a module, the tests break even if the behavior is unchanged.

### Capturing Acceptance Criteria

A technical product manager who cares about what the system does rather than how it does it will find integration tests much more useful. Integration tests describe observable behavior, which maps naturally to acceptance criteria. This means:

- Requirements are documented in code, not a wiki page
- Support teams can use failing tests as a starting point for debugging
- New engineers can understand features without digging into implementation details


## The Mocking Problem

When picking integration tests over unit tests, you have to decide how deep to integrate. This is where mocking decisions matter a lot.

| Mocking Level            | What You Lock In             | Risk                                                                   |
| ------------------------ | ---------------------------- | ---------------------------------------------------------------------- |
| Mock repositories        | Internal data access pattern | Refactoring repositories becomes harder                                |
| Mock the database engine | DB vendor and query behavior | Migrating to a different DB causes test failures without real breakage |
| Use a real test database | Nothing locked               | Slower setup, but tests reflect actual behavior                        |

The rule of thumb: **mock at the boundary you are willing to commit to**. The more you mock, the more your tests assume about the internals, and the harder it becomes to refactor those internals safely.


## Where Integration Tests Shine

### Refactoring with Confidence

A strong set of integration tests gives you the freedom to refactor without worrying about silently breaking the application.

A real example: a background job had a repeated N+1 query problem. Refactoring it down to a single query resulted in a 70% memory reduction and lower infrastructure cost. That refactor was safe because the integration tests covered the acceptance criteria, not the internal query structure. The engineers before me had the foresight to keep tests at the behavior level, which paid off years later, Giving them a refactor within a sprint and time to buid event driven system projected at the current growth rate.

### Debugging Support

When a bug is reported, integration tests that describe user-facing behavior help support and engineering teams quickly figure out where to start. A unit test failure deep in a utility function is often harder to correlate to an actual user problem.


## The Tradeoffs You Should Actually Think About

Integration tests are not free. Here are the real costs:

- **Slower test runs** because they need to spin up a process, connect to services, and tear down after
- **More brittle** especially for frontend tests, where UI changes with every redesign
- **Higher setup complexity** compared to isolated unit tests

This makes the **lifetime of the project** a useful metric. If the frontend gets rewritten with every UX cycle, heavy integration testing at the UI layer is likely wasted effort. If the backend API has been stable for years and teams keep breaking it accidentally, integration tests at the API level pay for themselves.


## Integration Tests in Monoliths

Integration tests are often blamed for slowing down development in monoliths because the full test suite can take a long time to run. This is a genuine tradeoff. But there is a way to manage it.

A modular monolith, where code is organized by domain directory, can run only the tests relevant to the changed module. This gives you most of the benefits of integration testing while keeping feedback loops reasonable. The test suite scales with the project, not against it.


## Summary: Unit vs Integration Tests

|                              | Unit Tests              | Integration Tests     |
| ---------------------------- | ----------------------- | --------------------- |
| Speed                        | Fast                    | Slower                |
| Refactor safety              | Low (tied to internals) | High (tests behavior) |
| Captures acceptance criteria | Rarely                  | Yes                   |
| Debugging usefulness         | Low level               | User-facing behavior  |
| Setup complexity             | Low                     | Higher                |
| Frontend suitability         | Good                    | Brittle with UI churn |
| Backend API suitability      | Partial                 | Very good             |

The right testing strategy depends on what you are building, how long it needs to last, and how often the internals change vs. the behavior.


## References
Some examples of API Testing

- [FastAPI Testing](https://fastapi.tiangolo.com/tutorial/testing/)
- [Integration Testing in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0&pivots=xunit)
