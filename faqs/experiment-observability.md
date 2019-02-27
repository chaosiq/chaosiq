# How important is it to be able to observe a chaos experiment itself?

Chaos engineering can fail in an organisation for a number of reasons. From not understanding the mindset right through to chaos engineers subjecting other teams to chaos in a conflict-style relationship, chaos engineering done badly can easily end up with the discipline being rejected.

We at [ChaosIQ](https://chaosiq.io/) believe that _everyone_ in an organisation will eventually begin to practice chaos engineering. Rather than being a technique just for the chosen few, chaos engineering is a useful mindset and discipline for everyone who owns mission-critical business systems. Understandable, this means that, even for a moderately sized organisation, there is the challenge of _chaos engineering at scale_. And at scale the opportunities for failure of the chaos engineering approach increase.

One such opportunity for the failure of chaos engineering, which appears at all scales of adoption, is when chaos engineering is practices as a _surprise_. What this means is that if anyone is surprised when an experiment executes then that's usually a bad moment for everyone and can frequently lead to requests not to run those experiments again.

Sometimes it is useful to surprise your team with a chaos experiment itself, say as part of an organised Game Day. In order to explore how your team responds to turbulent conditions, perhaps to explore any people/practices/process weaknesses, you may not tell your colleagues the exact nature of the conditions being applied, and so it could be argued that those conditions are a surprise. However the Game Day itself is not a surprise in and of itself, and this is an important point.

When executing chaos experiments it is also useful for those experiments to not be a surprise either. Everyone potentially affected by chaos experiment execution should be able to know that chaos is being executed, when and against what and, ideally, why.

The combination of the [Chaos Toolkit and Platform's](https://chaostoolkit.org/) [experiment format](https://docs.chaostoolkit.org/reference/api/experiment/) _and_ pluggable chaos engineering observability makes this level of cross-team and organisation visibility possible, avoiding the dangers of surprise chaos and the adverse reactions we've seen that can happen at that point.

So in many respects, regardless of the size of your teams and chaos engineering capability, bringing your chaos experiment execution into your overall system observability picture is crucial to avoid these pitfalls. Chaos engineering experiments are also citizens within your system, and so should be good citizens by making their activitiers known and observable alongside all the other aspects of your runtime systems.

----

This content is licensed under the [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/) license. If you would like to submit a comment, correction or extend its ideas then please [raise an issue on the accompanying repository](https://github.com/chaosiq/chaosiq).