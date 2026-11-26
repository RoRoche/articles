---
title: "AI Is Eating Software. Who Will Be Responsible for the Code?"
description: "Exploring the implications of AI-generated code on software ownership and accountability."
tags: ["Software Development", "Software Engineering", "AI", "Ownership", "Accountability"]
status: draft
category: ai
slug: ai-is-eating-software-who-will-be-responsible-for-the-code
image: header.png
published_at: null
dev_to_url: null
medium_com_url: null
---

## From "Software is eating the world" to "AI is eating software"

In August 2011, Marc Andreessen wrote an article in the *Wall Street Journal* titled "Software is eating the world."

Andreessen is a Silicon Valley entrepreneur and investor, and a co-founder of Netscape.

He argued that software was becoming central to every industry, transforming fields from music and retail to finance, healthcare, and education.

He pointed to examples such as:

- Netflix replacing traditional video rental stores,
- Amazon reshaping retail,
- Uber disrupting transportation.

The phrase captures how **technology companies, especially software companies, are coming to dominate the global economy**. Software has become a major driver of innovation, growth, and disruption across nearly every sector.

Why does this matter?

- **Digital transformation:** Companies must adapt or risk falling behind.
- **Competitiveness:** Software enables unmatched scale and efficiency.
- **Faster innovation:** Shorter development cycles make it easier to adapt.

Today, a new idea is taking hold: "AI is eating software."
AI no longer just assists developers. It can generate, test, analyze, refactor, and help maintain software. Generative AI tools such as ChatGPT, GitHub Copilot, and Claude Code help developers produce code faster and at lower cost.
If AI can write code for us, why would companies still need engineers?

The answer is simple: **you are responsible for what AI builds.**

## If something breaks, who is responsible for the code?

This is where software engineering begins.

By 2030, what will be the most important skill for a software engineer?

- Writing code quickly
- Guiding an AI
- Designing a sound architecture
- Ensuring quality and reliability
- I don't know yet

The goal is to show why an engineer's value goes beyond writing code.

No single answer is right. Architecture, quality, business understanding, and the ability to work with AI all matter.

Consider this scenario:

> We work for a bank and ask an AI to build a bank transfer service. It generates 2,000 lines of code in minutes. The project compiles, and all the tests it wrote pass.
>
> We deploy the service to production.
>
> The next day, some transfers are processed twice.
>
> Who is responsible: the AI, us, or our company?

This raises a few questions:

- Why didn't the tests catch the problem?
- How can we prevent the same transaction from running twice?
- How do we design a system that can withstand failures?
- How can we tell whether generated code is maintainable?
- How do we stop the problem from happening again?

These are questions of software architecture and quality.

I think of engineering skills in three levels:

- **Level 1 — Producing code:** Know a language, write algorithms, and implement features. AI can greatly reduce the effort required for some of this work.
- **Level 2 — Designing software:** Structure a system, manage dependencies and complexity, plan for change, and make sound trade-offs.
- **Level 3 — Taking responsibility for quality:** Define requirements, verify expected behavior, identify risks, justify decisions, and build confidence appropriate to the stakes.

AI already contributes at all three levels. Architecture and quality are not uniquely human activities.

But delegating work to AI does not transfer responsibility for the result. Engineers and their organizations still need to define expectations, evaluate the output, and stand behind it.

## Knowledge bases and quality gates: an engineer's responsibility

In spec-driven development, AI generates code from a specification. Engineers must be able to define and validate that specification, then check that the generated code meets its requirements.

A useful foundation is a knowledge base that brings together best practices, patterns, security rules, and business requirements. Keep it up to date and make it accessible to the whole team. Its contents can live in version-controlled text formats such as Markdown and Gherkin. This gives both AI and engineers a shared reference for the project's standards and requirements.

Quality gates are also essential. These checkpoints verify that code meets agreed standards before it is integrated into a project. Static analysis, automated tests, and code reviews can catch problems early and reduce the risk of introducing bugs or vulnerabilities.

In recent projects where I used AI agents, I consistently relied on tools such as:

- Linters and formatters to keep code consistent and readable.
- Unit and integration tests to verify expected behavior.
- Mutation testing to check whether existing tests catch unintended changes.
- Static analysis to find vulnerabilities and poor practices.
- Renovate to keep dependencies up to date and secure.
- Visual regression tests, when appropriate, to ensure the interface stays consistent after changes.

## Final thoughts

Writing code is becoming cheaper. Ensuring its quality and reliability still takes sustained effort. AI can speed up production, but it cannot take an engineer's responsibility away.

Engineers must continue to build strong skills in software architecture, quality, and accountability. That is how we make sure AI-generated code is reliable and meets the project's needs.
