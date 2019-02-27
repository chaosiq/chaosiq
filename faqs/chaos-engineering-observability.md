# How important is it to be able to observe a system before running chaos experiments?

The key word in this question is "before". While it is very useful to have good system observability regardless of whether you are embarking on chaos engineering, it is not _exactly_ a pre-requisite.

The two approaches, chaos engineering and observabiltiy, work well together. 

In some cases you will already have some system observability before you do any chaos engineering, and this will definitely help with when your chaos engineering experiments surface deviations that need to be diagnosed into system weaknesses. However it's also the case that by starting to conduct chaos engineering experiments will become a "forcing function" (Reference); it will highlight the need for better system observability if it is not present.

For this reason, in our experience good system observability is less of as strong pre-requisite to chaos engineering. More often as you consider chaos engineering your lack of system observability will come to the fore and both techniques will mutually develop and reinforce one another over time. 

Interestingly this is also why chaos engineering is a good "forcing function" for other non-functional or operationally-focussed system improvements. Chaos engineering often makes such easily-forgotten or de-priotised concerns utterly unignorable, and is a great wasy of helping everyone, even those not responsible and intimate with production systems, aware of these challenges and then able to design and implement system robustness and resilience strategies to meet them.

In a nutshell, chaos engineering makes everyone a better operationally-sensitive system owner, and system observability is one capability that benefits strongly almost immediately.

----

This content is licensed under the [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/) license. If you would like to submit a comment, correction or extend its ideas then please [raise an issue on the accompanying repository](https://github.com/chaosiq/chaosiq).
