**Chaos Engineering Fundamentals: The Steady State**

In chaos engineering, the Steady State is the core idea surrounding each experiment. Without this, running disruptions against resources is actually chaos, instead of controlled experiments. 

**What is Steady State?**

The Steady State defines what you believe to be the status of your environment before, after, and potentially during a chaos experiment execution. Any change to this hypothesis results in a deviation (a result) is a candidate for investigation and (potentially) a place for improvements to be applied. In other words, if your system does not return to its expected Steady State after running a chaos experiment, that should be a red flag that you need to investigate your system’s robustness strategies.

**How do I declare my Steady State Hypothesis?**

You can determine which deviations from Steady State get flagged by defining the tolerance range. The probe tolerance can be defined by a number of factors based on the probe itself. The most rigid tolerances check for a boolean or count value, while more flexible probes check for a value in a json path or regular expression. The last type of tolerance can be defined using another probe. These tolerances should be used carefully as they can cause an experiment to fail if not defined correctly.

It is recommended that you make your steady-state-hypothesis as detailed as possible to get the most accurate results. To define your Steady State using Chaos Toolkit, for example, you need to use probes that scan your environment before and after the experiment runs. Below is an example snippet for defining a probe.

```json
{
 "steady-state-hypothesis": {
   "title": "asg is healthy with no scaling in progress",
   "probes": [
     {
       "type": "python",
       "name": "asg is healthy",
       "tolerance": true,
       "provider": {
         "module": "chaosaws.asg.probes",
         "func": "desired_equals_healthy",
         "arguments": {
           "asg_names": [
             "my-first-auto-scaling-group",
             "my-second-auto-scaling-group"
           ]
         }
       }
     },
     {
       "type": "python",
       "name": "no scaling in progress",
       "tolerance": false,
       "provider": {
         "module": "chaosaws.asg.probes",
         "func": "is_scaling_in_progress",
         "arguments": {
           "asg_names": [
             "my-first-auto-scaling-group",
             "my-second-auto-scaling-group"
           ]
         }
       }
     }
   ]
 }
}
```

Let’s break down what’s happening in this hypothesis:

- First, we have to define what type of probe we are going to use. For all probes related to checking the value of a cloud resource, the type will (generally) be “python”. For probes that will interact internally to a resource (ex: a path exists on an instance or a specific process is running) the type will vary. We also give a “name” to the probe, this is just a way to easily define what the probe will do.
   
   Additional information on probe types can be found in the [chaostoolkit docs page](https://docs.chaostoolkit.org/reference/concepts/)

- Next, we are declaring our tolerance. Both the probes above return a boolean value. The first probe “asg is healthy” is checking to see that the “desired” number of instances matches the number of “healthy” instances in the ASG. The second probe is validating that the ASG is not currently scaling up or down.

If both of these probes are true, the steady-state passes and the experiment can begin. If one or both of the probes are not true, the experiment fails before moving onto the actions.

If a probe fails after the experiment, this provides a useful result that you can research to find the cause of the deviation. Once you have corrected for this deviation, run the experiment again to prove your correction works. 

As you can see, Steady State analysis becomes most useful when it is run frequently and as part of the experiment. Knowing your Steady State is the key to successful and meaningful chaos engineering experiment implementation. 
