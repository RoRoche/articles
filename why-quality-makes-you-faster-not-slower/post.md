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

One important caveat: the multipliers in Tables 5-1 and 5-2 are explicitly illustrative.
They are not a universal law that every production bug costs thirty or one hundred times more to fix.

The mechanism is more useful than a dramatic number.
An error found locally may need a code change.
The same error in production may also require diagnosis, a patch, support, and data repair.

Eric Elliott explores this broader cost in [The Outrageous Cost of Skipping TDD & Code Reviews](https://medium.com/javascript-scene/the-outrageous-cost-of-skipping-tdd-code-reviews-57887064c412).
His discussion highlights interruptions, context switching, and handoffs: a production defect can disrupt work well beyond the fix itself.
His January 2019 update also acknowledges that the widely repeated 100× repair-cost claim lacks a reliable basis.
I take the article as an argument for examining the full cost of rework, rather than as proof of a universal return on investment.

To make the case in your own team, examine recent incidents.
How much time went into finding the cause, restoring service, handling support, and repairing the consequences?
What feedback would have helped earlier?

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
It does not mean that deploying more often will, by itself, cause better quality, or that every local speed–risk trade-off disappears.

The mechanism makes sense to me: small changes are easier to understand and diagnose, and useful tests catch mistakes.
Repeatable deployments reduce manual variation.
Observability helps us detect and recover from failures.

Those practices reinforce one another.
A decoupled architecture can reduce coordination, but introducing microservices without the ability to operate them may create more friction than it removes.

Measure one application's trends in context.
Use the findings to investigate constraints with the team.
Ranking individuals or imposing deployment quotas would change the incentives without necessarily improving delivery.

And keep asking whether the product is useful.
Delivery performance measures how effectively changes reach production; it cannot tell us whether we chose the right changes.

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
They do not guarantee them.
It is possible to write brittle tests around a poorly designed system while following the TDD cycle.

I favor small fakes when they make a dependency's behavior clear.
But a fake database does not establish that real SQL, transactions, or constraints work.
Those boundaries need integration tests as well.

The test suite should make internal refactoring easier while detecting changes in observable behavior.
If every refactoring requires rewriting dozens of mocks, I would question what the tests are coupled to.

## Challenge the tests themselves

This is the theme of my [mutation testing talk at Clermont'ech](https://www.clermontech.org/talks/api-hour-72/romain-rochegude-mutation-testing-quand-vos-tests-sont-mis-a-lepreuve.html).
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

This is not a demand for a perfect mutation score.
Some mutants preserve behavior, and repeated test execution can be expensive.
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

A release script still depends on a developer's machine if it uses their local JDK, cached dependencies, working tree, and undeclared configuration.
Automating the commands is one step; controlling the environment makes the process repeatable.

**ChatOps brings operational commands into the project's conversation**, where the request, execution, and result are visible to the team.
[Rultor](https://www.rultor.com/) applies this approach to merge, release, and deployment operations through GitHub comments.
For example, an authorized collaborator can request a release with:

```text
@rultor release, tag is `1.4.0`
```

Rultor executes the workflow defined in the repository's `.rultor.yml`.
Its [Docker configuration](https://doc.rultor.com/reference.html#docker-image) defines the build environment, keeping execution separate from a developer's laptop.
The release starts in a fresh, isolated container, with the required toolchain and explicitly supplied configuration.
The environment is disposable: leftover files from a previous local build should never be an undeclared input to publication.

For my Java libraries, the intended flow is concrete:

1. Check out the source state selected for release.
2. Prepare the requested version through the versioned release script.
3. Build and run the required quality checks in the isolated environment.
4. Sign and publish the resulting artifacts to Maven Central using explicitly supplied credentials.
5. Have Rultor tag the resulting release commit and push the tag after the script succeeds.

Rultor's [release documentation](https://doc.rultor.com/basics.html#release) describes this division of responsibilities between the project's script and the bot.
Publishing a library and deploying an application are distinct operations; Rultor also supports a separately configured deployment script.

The human decides which state to release and which version to assign.
Automation performs the ceremony and leaves logs attached to the conversation.
That evidence helps answer practical questions: who requested the release, what ran, what failed, and which commit corresponds to the published version?

A fresh container removes accidental local state.
Reproducibility also requires controlled inputs: pinned tool versions, resolved dependencies, and explicit configuration.
Secrets belong outside the source code and should be supplied only where needed.

This is how release automation supports quality and speed together: fewer manual steps, fewer hidden prerequisites, and less dependence on the one person whose laptop can publish.
The same properties can be built with other CI/CD tools; Rultor makes the ChatOps approach tangible.

Production still needs observability, controlled rollout, and a recovery strategy.
Tests cannot predict every interaction with real traffic, data, and external services.

A successful deployment is a technical event.
Whether users benefit from the change is a separate question that requires product feedback.

## Remove the small frictions that accumulate

Rework is not the only hidden cost.
Searching for a command, reconstructing test setup, or deciphering a silent CI job also consumes attention.
These irritations can become so familiar that the team stops questioning them.

Consider an illustrative calculation: two minutes lost, ten times a day, across ten developers and 220 working days amounts to about **733 hours a year**.
That is arithmetic based on assumed repetition, not measured savings or capacity we can automatically recover.
It is a reason to investigate frequent small costs alongside spectacular bottlenecks.

The first fix may be simple: use an existing test fixture, give a build command an obvious name, or improve an assertion's failure message.
If every test repeats the same setup and comparison logic, a focused helper can make the next hundred tests easier to write and diagnose.
Check what the standard tools already provide before introducing another abstraction.

Some improvements make work more understandable without making execution faster.
A long-running test that reports progress is easier to distinguish from a hung process.
Preserving the relevant diagnostic artifacts after a failure can save the next person from rerunning the entire investigation.

The same principle applies to knowledge.
When a release requires one particular colleague, that person has become a dependency in the delivery process.
Encode repeatable steps in versioned commands and explain their intent in documentation, so colleagues can learn without needing a permanent intermediary.
ChatOps and reproducible release environments address this problem at the publication boundary.

Keep a small friction backlog: what recurs, how often, who encounters it, and what is the smallest useful fix?
Compare the cost of implementing and maintaining the fix with its likely reuse.
Observe whether it actually helps before expanding it.

This also sets a boundary for automation.
Understanding a domain rule or debating a consequential design decision takes effort that produces learning.
Repeating a mechanical procedure rarely does. Removing that repetition gives the team more attention for the work that needs judgment.

## Manage technical debt through its consequences

Technical debt can appear even when a team makes sensible decisions.
We learn more about the domain, dependencies age, and yesterday's architecture may no longer fit today's constraints.

Some debt is deliberate: accepting a bounded shortcut to test an assumption can be reasonable.
The decision needs an owner, visible consequences, and a condition for revisiting it.

The useful question is rarely "how much debt do we have?" in the abstract.
It is "which debt is making the next important change slower or riskier?"

For each significant item, capture:

- The friction or failure risk it creates, with a recent example.
- The product work it affects and the likely consequence of waiting.
- The smallest useful improvement, its owner, and a review date.

This makes prioritization possible without pretending that every code smell has an exact financial value.

[Renovate](https://docs.renovatebot.com/) and [Dependabot](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-version-updates) automate dependency update proposals.
They open pull requests that make outdated dependencies visible and bring maintenance into the team's normal review workflow.
I use Renovate on my open-source projects.
Dependabot is another valid option; the choice depends on the project's stack.

Small, frequent updates are easier to assess than an occasional migration across many versions.
Each proposal should pass the same quality gates as other changes: compilation, tests, static analysis, and the relevant architecture and security checks.
Choose a schedule and grouping strategy that the team can sustain. Automation should reduce maintenance friction rather than create an unmanageable review queue.

[SonarQube](https://docs.sonarsource.com/sonarqube-server/2025.4/user-guide/code-metrics/metrics-definition) makes another part of technical debt visible through static analysis.
It reports maintainability issues and estimates the effort required to remediate them.
Its technical debt measure sums those remediation estimates, giving teams a starting point for sizing cleanup work and tracking progress.
Integrated into CI/CD, it can also enforce agreed quality gates, including coverage conditions.

Those estimates help make debt actionable, but still need context: a rule-based estimate does not capture every architectural constraint or the full cost of a change.
I would combine SonarQube's findings with the team's knowledge of the affected code and product priorities before committing to a repayment plan.
Coverage and static analysis provide useful signals; meaningful tests and human review establish what those signals mean for the product.

AI-assisted changes deserve the same scrutiny.
Generating a patch faster does not establish that its behavior, compatibility, or design is correct.
Verification capacity matters as much as generation speed.

I would address nearby debt as part of feature work when practical, and plan larger improvements against explicit product risks.
Waiting for a mythical cleanup phase makes repayment unlikely.

## Adaptation requires engineering and leadership

VUCA describes a world that is **volatile, uncertain, complex, and ambiguous**: conditions change, predictions are limited, dependencies interact, and situations admit several interpretations.

Jamais Cascio's [BANI framework](https://medium.com/@cascio/facing-the-age-of-chaos-b00687b1f51d) offers another lens: **brittle, anxious, nonlinear, and incomprehensible**.
Systems may fail abruptly, uncertainty can paralyze decisions, effects can be disproportionate to causes, and more information may still leave us struggling to understand events.

These are ways of framing uncertainty.
I would not treat BANI as a scientifically established replacement for VUCA.
For engineering teams, their value lies in the questions they prompt.

Can we detect a mistaken assumption quickly?
Can we change direction without rebuilding everything?
Can someone raise a risk before it becomes an incident?

As responsibility grows, quality becomes a matter of shaping the conditions in which people work:

- **Developers** make behavior explicit through tests and keep changes understandable.
- **Tech leads** share reasoning, coach through pairing, and help the team own its standards.
- **Architects** reduce unnecessary coupling and make consequential trade-offs explicit.
- **Technical managers** protect improvement capacity, remove recurring obstacles, and support learning without blame.

A team needs room to say that a test is unreliable, a deadline is unrealistic, or a decision has stopped working.
Punishing those signals delays feedback precisely when we need it most.

Continuous improvement can be modest and concrete.
Choose one recurring source of friction, agree on a small intervention, assign an owner, and inspect the result at the next retrospective.

If reviews are the bottleneck, adding more tests may not help.
If production incidents consume the week, faster code generation may simply feed more work into an unstable system.
Follow the constraint.

## Put the claim to work

Choose one recent change that took longer than expected.
Trace it from the initial need to production, including review queues, failed checks, rework, and recovery.

Then ask the team: **what earlier feedback or smaller decision would have saved us the most time?**
Try one improvement and observe whether it helps the next comparable change.

Quality does require investment.
The aim is to spend that effort where it reduces uncertainty, rework, and the cost of changing our minds.

My own extension of Martin's sentence would be: *to go fast, we need to go well; to keep going well, we need fast feedback.*

That is the speed I want to build: the ability to deliver useful changes repeatedly, with a team that can sustain the pace.
