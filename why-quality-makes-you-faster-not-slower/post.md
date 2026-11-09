---
title: "When Quality Makes You Faster"
description: "How meaningful tests, shared ownership, and technical leadership reduce rework and make fast, reliable delivery sustainable."
tags: ["Software Development", "Software Engineering", "Test Driven Development", "DevOps", "Technical Leadership"]
status: draft
category: software-engineering
slug: why-quality-makes-you-faster-not-slower
image: header.png
published_at: null
dev_to_url: null
medium_com_url: null
---

> "The only way to go fast, is to go well."
>
> — Robert C. Martin, [Going Fast](https://sites.google.com/site/unclebobconsultingllc/going-fast)

This sentence has stayed with me throughout my career, and beyond work.
In French, we have a related saying: *ne pas confondre vitesse et précipitation*.
Do not confuse speed with haste.

I see it in everyday life: doing something carelessly often means doing it, undoing it, and doing it again.
Software adds another complication: other people build on our mistakes before we return to fix them.

Yet "quality makes us faster" deserves more than a slogan.
Tests take time.
Reviews can become queues.
Automation needs maintenance.
A beautifully engineered feature can still solve the wrong problem.

My argument is about the time between identifying a need and delivering a useful, reliable response, including everything we must redo along the way.
Quality improves speed when it reduces that total time.

A developer can shorten a feedback loop in code.
A technical leader must help the whole team do so.

## Start by negotiating scope

"We do not have time to test. We need to ship."

I understand the pressure behind that sentence.
Early in my career, the familiar project triangle framed cost, deadlines, and quality as competing constraints.
Under pressure, quality became negotiable.

Agile thinking helped me make scope explicit in that discussion.
Scope was always a project variable; agility made it easier to revisit it frequently rather than treat the original specification as fixed.

When a date matters, I would first ask: **what is the smallest useful outcome we can deliver safely by then?**
Can we serve one user group, support one workflow, or postpone an optional integration?

Some dates can move; others cannot.
The conversation must make those constraints visible, along with the consequences of each choice.
Silently removing verification hides risk instead of resolving it.

The [Agile Manifesto's principles](https://agilemanifesto.org/principles.html) connect technical excellence and good design with agility.
That connection matters: code we struggle to change limits our ability to respond to users.

Quality must also fit the context.
A disposable experiment and a payment service need different levels of assurance.
Both need an explicit understanding of what can fail and who bears the consequences.

## Make the cost of rework visible

When someone says we cannot afford automated tests, I often ask: "Can we afford not to test?"

That is a starting point for an economic discussion.
The relevant comparison includes the cost of writing and maintaining tests, the failures they can prevent, and the impact of those failures.

A useful historical reference is RTI's [2002 report for NIST](https://www.nist.gov/system/files/documents/director/planning/report02-3.pdf#page=94).
Section 5 models repair costs according to where defects originate and where they are detected.
It also separates developer costs from losses borne by users.

The mechanism is more useful than a dramatic number.
An error found locally may need a code change.
The same error in production may also require diagnosis, a patch, support, and data repair.

Eric Elliott explores this broader cost in [The Outrageous Cost of Skipping TDD & Code Reviews](https://medium.com/javascript-scene/the-outrageous-cost-of-skipping-tdd-code-reviews-57887064c412).
His discussion highlights interruptions, context switching, and handoffs: a production defect can disrupt work well beyond the fix itself.
His January 2019 update also acknowledges that the widely repeated 100× repair-cost claim lacks a reliable basis.

On a web platform I worked on, a bug in the notification workflow reached production.
One person spent at least two days maintaining a workaround while the issue was reproduced.
A developer needed a few hours to correct the code; QA spent several hours validating the fix, and Ops spent several more deploying it.
The response spanned several roles and days, far beyond the time needed to write the fix.

Trace incidents in your own team across diagnosis, support, workaround, validation, and redeployment.
What earlier feedback could have shortened that path?

That gives stakeholders something concrete to weigh against the next investment in prevention.

## What Accelerate changed for me

When I read *Accelerate* by Nicole Forsgren, Jez Humble, and Gene Kim in 2018, I wanted evidence to support the case for agile and DevOps practices.
It helped turn convictions into a more testable argument.

The research challenged the assumption that faster delivery necessarily means lower stability. Its findings support investing in capabilities that improve both. [DORA's current guidance](https://dora.dev/guides/dora-metrics/) retains that central conclusion.

The four historical measures associated with *Accelerate* were:

- **Deployment frequency:** how often changes reach production.
- **Lead time for changes:** time from a code commit to successful production deployment.
- **Time to restore service:** time to restore service after a failure or degradation.
- **Change failure rate:** proportion of production changes that cause failure and require remediation.

These definitions have evolved. DORA now uses five measures, adding deployment rework rate and narrowing recovery measurement to failed deployments.
Rework rate tracks the share of deployments prompted by production incidents.
DORA documents these changes in its [metrics history](https://dora.dev/insights/dora-metrics-history/).

I read this evidence as a reason to improve the delivery system.

The mechanism makes sense to me: small changes are easier to understand and diagnose, and useful tests catch mistakes.
Repeatable deployments reduce manual variation.
Observability helps us detect and recover from failures.

Those practices reinforce one another.
A decoupled architecture can reduce coordination, but introducing microservices without the ability to operate them may create more friction than it removes.

Measure one application's trends in context.
Use the findings to investigate constraints with the team.
Ranking individuals or imposing deployment quotas would change the incentives without necessarily improving delivery.
And keep asking whether the product is useful.

## Adapt quickly in a changing world

VUCA (volatile, uncertain, complex, ambiguous) and BANI (brittle, anxious, nonlinear, incomprehensible) are lenses for describing uncertainty ([BANI framework](https://medium.com/@cascio/facing-the-age-of-chaos-b00687b1f51d)).
[DORA's guidance](https://dora.dev/guides/dora-metrics/) and *Accelerate* point toward the same response: build fast feedback loops and reliable delivery so teams can adapt without trading stability for speed.

Can the team detect a mistaken assumption quickly, change direction without rebuilding everything, and raise risks before they become incidents?
Developers and leaders both shape the conditions that make this possible.

## Use a real bug to introduce test-first development

I like introducing automated testing through a defect the team already wants to fix:

1. Write a test that reproduces the defect and states the expected behavior.
2. Run it and confirm that it fails for the expected reason.
3. Fix the implementation and confirm that the test passes.
4. Refactor while keeping the suite green.

The initial failure matters.
A test that already passes has not demonstrated that it detects this bug.

This approach addresses a familiar frustration: "I thought we had fixed that!"
It adds a regression check for the behavior we now understand.
Over time, those checks form a useful safety net.

It is also an accessible route toward test-driven development.
Test-first describes the ordering; TDD adds a repeated red–green–refactor cycle in which tests help shape the design.

Writing a test first forces useful questions.
What behavior do I expect?
Which dependencies are essential?
Can I express this rule without starting the entire application?

Those questions can encourage cohesive responsibilities, loose coupling, and composition.

I favor small fakes when they make a dependency's behavior clear.
But a fake database does not establish that real SQL, transactions, or constraints work.
Those boundaries need integration tests as well.

The test suite should make internal refactoring easier while detecting changes in observable behavior.
If every refactoring requires rewriting dozens of mocks, I would question what the tests are coupled to.

## Challenge the tests themselves

This is the theme of my [mutation testing talk at Clermont'ech (in French)](https://www.clermontech.org/talks/api-hour-72/romain-rochegude-mutation-testing-quand-vos-tests-sont-mis-a-lepreuve.html).
Executing code and checking its behavior are different things.

Suppose a rule accepts strictly positive numbers:

```java
return number > 0;
```

A test using `1` executes that line. It will also pass if someone changes the rule to:

```java
return number >= 0;
```

Coverage alone cannot distinguish those implementations.
A test of the boundary value `0` can.

Mutation testing deliberately introduces changes like this and checks whether the tests detect them.
A surviving mutant gives us a question to investigate: is an assertion missing, or is the changed behavior irrelevant?
Tools include [PIT](https://pitest.org/) for Java, [Stryker](https://stryker-mutator.io/) for JavaScript and .NET, and [Infection](https://infection.github.io/) for PHP.

I would start with important business rules and use the findings to improve assertions.

The practical question is: **would our tests notice if this behavior became wrong?**

## Make review a learning mechanism

Pair programming, mob programming, and asynchronous code review let us examine code together at different moments.
Each can expose assumptions, share knowledge, and improve decisions.

I learned a great deal through code review when I discovered Java 8 and its Streams API.
Seeing colleagues traverse, transform, and collect data changed how I approached collections.

That learning goes both ways.
An experienced developer may recognize a concurrency issue; a newcomer may reveal that a supposedly obvious abstraction is difficult to understand.

Review should strengthen collective ownership.
It should also help us distinguish a correctness problem, a design concern, and a personal preference.
Those comments deserve different levels of urgency.

Review can also slow delivery when changes are large or one expert must approve everything.
A pull request waiting for two days is part of delivery time, even if the review takes only ten minutes.

Keep changes small, explain their intent, and make review a team responsibility.
Use pairing when a discussion is becoming a long exchange of comments.
Automate formatting so people can focus on behavior and risk.

A recurring review comment is worth examining: can we turn an agreed convention into an executable check?
With [ArchUnit](https://www.archunit.org/userguide/html/000_Index.html#_package_dependency_checks), for example, a Java team can check that domain packages do not depend on infrastructure packages.
The diagram explains the intended boundary; the architecture test detects when a change crosses it.

The agreement must come first.
A design preference does not become universal merely because we can encode it in a tool.
Reviewers still need to discuss whether an abstraction fits the domain, whether a trade-off is acceptable, and whether a rule itself should change.

The tech lead's contribution is to make reasoning visible and develop others' judgment. Becoming everyone's mandatory checkpoint limits that impact.

## Build a pipeline that earns trust

I think of CI/CD as automating trust.
Every change should produce evidence that the team can understand and act on.

Start with fast checks: formatting, linting, and compilation where relevant.
Then run tests that verify behavior and integrations.
Add security, performance, and end-to-end checks according to the risks of the product.

Automating more checks is useful only if the feedback remains actionable.
A slow, flaky suite trains people to ignore failures.
A fast suite with weak assertions creates false confidence.

[DORA's guidance on test automation](https://dora.dev/capabilities/test-automation/) emphasizes reliable suites and fast feedback.
I would treat the health of the pipeline as part of the product's engineering work.

Choose a place for each check.
Fast, deterministic tests belong close to every change.
Expensive load tests or broad mutation analysis may need a separate schedule or a narrower scope.

Build a versioned artifact once and promote it through environments.
Script deployment so success does not depend on someone remembering a sequence of manual steps.

Continuous delivery means keeping software deployable.
Continuous deployment goes further by automatically deploying changes that pass the required checks.
A team can benefit from the former while retaining a release decision.

### Design quality gates developers can trust

A quality gate combines checks into an explicit decision about whether a change can proceed.
Its purpose is to protect the team's attention by catching known problems consistently.
Each check should answer a clear question and help the developer act when it fails.

"Quality gate failed" gives us another problem to investigate.
A useful failure identifies the affected file or class, the violated rule, why it matters, and how to reproduce the check.
For a dependency boundary, name both classes and show the forbidden dependency.
For a failed assertion, report the expected and actual behavior.

Provide a documented entry point, such as `make check` or `mvn verify`, for running the relevant checks locally with consistent tool versions.
Keep environment-specific checks explicit.
Developers should know what they can verify before pushing and what requires the CI environment.
Run cheap, useful checks first and parallelize independent checks when resources allow it.

Decide deliberately which findings block progress:

- **Block on understood, reliable checks:** compilation failures, failing behavior tests, or a violation of an agreed architecture boundary.
- **Start new or uncertain signals in informative mode:** an experimental rule or an uncalibrated performance threshold needs observation before it can justify blocking delivery.
- **Record exceptions:** capture the reason, an owner, and a review or expiry date, so a temporary compromise stays visible.

On an existing codebase, assess historical violations before enforcing a new rule.
Fix them or establish an explicit baseline, then prevent new violations while reducing the backlog incrementally.
A useful improvement should not require an unrelated feature to absorb the entire cleanup cost.

Treat the gate as an internal product with developers as its users.
Give its rules owners, inspect duration and false positives, and fix intermittent failures.
If a check repeatedly wastes attention, improve or replace it.
A permanently red pipeline eventually loses its meaning.

### Make releases repeatable with ChatOps and Rultor

For my Java libraries, [Rultor](https://www.rultor.com/) is a ChatOps bot that runs the release script in an isolated environment, including signing and publishing artifacts to Maven Central. Its [release documentation](https://doc.rultor.com/basics.html#release) explains the workflow.

The broader lesson is tool-independent: make release inputs explicit, keep credentials out of source, and preserve a visible execution log.

Production still needs observability, controlled rollout, and a recovery strategy.
Tests cannot predict every interaction with real traffic, data, and external services.

A successful deployment is a technical event.
Whether users benefit from the change is a separate question that requires product feedback.

## Remove the small frictions that accumulate

Small recurring frictions—reconstructing test setup, finding commands, or diagnosing silent CI failures—consume attention and delay feedback.
Fix the most frequent source with the smallest useful change, then check whether it helped.
Better diagnostics and repeatable commands make feedback easier to use, even when they do not make execution faster.

## Manage technical debt through its consequences

Technical debt matters to speed when it makes an important change slower or riskier.
Prioritize it by concrete friction and likely rework, not abstract code smells or static-analysis estimates.
Tools such as [Renovate](https://docs.renovatebot.com/), [Dependabot](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-version-updates), and [SonarQube](https://docs.sonarsource.com/sonarqube-server/user-guide/code-metrics/metrics-definition) can surface candidates, but product context should drive repayment.
Address nearby debt with feature work, and plan larger improvements against explicit product risks.

## What quality does not promise

Quality does not guarantee that every verification investment pays for itself, that TDD produces a good design, or that deploying more often alone causes better quality or removes every trade-off.
The NIST repair-cost multipliers are illustrative, not universal.
Some mutants preserve behavior, and mutation testing can be expensive, so treat its score as a signal rather than a target.
Delivery metrics show how changes reach production, not whether they were the right changes for users.

## Put the claim to work

Choose one recent change that took longer than expected.
Trace it from the initial need to production, including review queues, failed checks, rework, and recovery.

Then ask the team: **what earlier feedback or smaller decision would have saved us the most time?**
Try one improvement and observe whether it helps the next comparable change.

Quality does require investment.
The aim is to spend that effort where it reduces uncertainty, rework, and the cost of changing our minds.

My own extension of Martin's sentence would be: *to go fast, we need to go well; to keep going well, we need fast feedback.*

That is the speed I want to build: the ability to deliver useful changes repeatedly, with a team that can sustain the pace.
