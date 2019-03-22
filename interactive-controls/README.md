# Adding User Interaction to Experiments in the Chaos Toolkit

In this article you're going to walk through how to create a new [Chaos Toolkit][chaostoolkit]
[Control][control] to enable various styles of user interaction where needed when an 
automated chaos experiment is being executed.

[chaostoolkit]: https://chaostoolkit.org/
[control]: https://docs.chaostoolkit.org/reference/extending/create-control-extension/

> ***NOTE:*** This article was originally inspired by a conversation with 
> [Simon Lindroos][simon] on the [Chaos Toolkit Community Slack][slack], and also relates to the 
> [older issue][dangerous-issue] originally raised on the [Chaos Toolkit][chaostoolkit] 
> concerning interaction needed when "dangerous" activities may be executed as part of an 
> experiment's execution.

[slack]: https://join.chaostoolkit.org/
[dangerous-issue]: https://github.com/chaostoolkit/chaostoolkit/issues/94
[simon]: https://twitter.com/SudoQ_

## The Need for User Interaction in Automated Chaos Experiments

By default, [Chaos Toolkit][chaostoolkit] experiments are assumed to be run from beginning to end 
without user interaction. This is still the most desirable strategy, as it makes it 
possible to schedule experiments to be executed automatically without having to 
noitify the team to make people available. Two cases where this is useful are 
when your chaos experiments are being used as chaos tests in a CI/CD system, or 
as part of the [Chaos Platform's][chaosplatform] scheduling capability.

[chaosplatform]: https://github.com/chaostoolkit/chaosplatform

However there are a number of cases when parts of, or all of, an experiment should 
involve user interaction. One example could be a Game Day where an experiment 
is being used to automate the setup and teardown of the turbulent conditions being 
explored and the person running the Game Day and manually progress each automated 
step. As [Simon Lindroos][simon] pointed out in the original 
discussion on the [Community Slack][slack], another example of where this control 
of an experiment's execution could involve an experiment such as:

1. Check steady state hypothesis: Look at some metrics (Automated)
2. Alter some configuration in Legacy HR system (Manual step, waits for user to make change and confirm before proceeding)
3. Check metrics (Automated)
4. Check steady state hypothesis again (Automated)
5. Rollback (Manual step, restore old configuration)

To meet as many cases as possible, here you'll see how the following user 
interaction styles can be implemented using the powerful [Chaos Toolkit Control][control] 
extension point:

* Simple "Click to continue..." style interaction.
* "Yes/No" to running an activity style interaction.
* "This is a Dangerous Activity, skip or proceed" enhancement to the previous style of interaction Shows one strategy to meeting the need specified in [this issue][dangerous-issue].

## Introducing the three new Chaos Toolkit Extensions

This article comes with three extensions for each of the above user interaction styles. 
Each of the modules is available under the [`sources` directory][sources] that accompanies this 
article.

[sources]: sources/

## Set up a new Python Virtual Environment for your new Extensions

It's a good practice to create a separate Python virtual environment when working 
on a new Chaos Toolkit[chaostoolkit] extension. Execute the following to create 
a new virtual environment called `chaosinteract`:

```bash
> python3 -m venv ~/.venvs/chaosinteract
```

Now you can activate your new virtual environment by executing the following:

```bash
> source  ~/.venvs/chaosinteract/bin/activate
```

You know things are working if you see `(chaosinteract)` as a prefix to your 
command prompt.

***NOTE:*** You might also consider installing the [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/), which comes with the convenient `workon` script for switching between Python virtual environments.

Now install the Chaos Toolkit according to the instructions available in the 
[project documentation][install].

[install]: https://docs.chaostoolkit.org/reference/usage/install/#install-the-cli

With your Python virtual environment set up and your `chaos` command working, 
you're now ready to explore and install each of the three different user 
interaction extensions to the Chaos Toolkit.

## The Simple User Interaction Control

The first new Chaos Toolkit Control shows how to provide a simple 
"Press Enter to Continue..." interaction style. The code for this extension is 
available in the `chaossimpleinteract` module's 
[`control.py`](sources/simple/chaossimpleinteract/control.py) source file:

```python
# -*- coding: utf-8 -*-
from typing import Any, Dict, List

from chaoslib.types import Activity, Configuration, Experiment, Hypothesis, \
    Journal, Run, Secrets
import click
from logzero import logger

__all__ = ["before_activity_control"]


def before_activity_control(context: Activity, **kwargs):
    """
    Prompt for Enter key before an activity is executed.
    """
    click.pause()
```

In the above code you are simply providing an implementation of the 
`before_activity_control` Chaos Toolkit Control callback function. The function 
will be called before the execution of any experimental activity that the 
control has been applied to.

### Installing your new Simple user interaction Control

To test your new simple user interaction control you first need to install it 
in your Python virtual environment so that it can be available to your installation 
of the Chaos Toolkit.

You can do this by running the following commands in the `sources/simple` project directory:

```bash
$ pip install -e .
```

You can now see that your new extension is installed as a Python module by 
executing the `pip freeze` command:

```bash
$ pip freeze

...
-e git+git@github.com:chaosiq/chaosiq.git@92da7697cfb40062abe5002f6fdbf70e53c82a92#egg=chaostoolkit_chaossimpleinteract&subdirectory=interactive-controls/sources/simple
...
```

> ***NOTE:*** Your exact path and GIT Hash will be different, but the important 
> thing to note is that you are using your new local `chaostoolkit_chaossimpleinteract` 
> module.

### Adding the "Simple" user interaction Control to an experiment

Once a Chaos Toolkit Control has been installed it can be explicitly used by 
your experiments. The [`simple-interactive-experiment.json`](samples/simple-interactive-experiment.json) experiment is already set to use your new 
simple user interaction control:

```javascript
{
    "version": "1.0.0",
    "title": "A simple sample experiment that can be executed to show controls",
    "description": "Does nothing at all other than be executable by the Chaos Toolkit",
    "tags": [],
    "steady-state-hypothesis": {
        "title": "Assess the Steady State ... (not really in this case)",
        "probes": [
            {
                "type": "probe",
                "name": "hypothesis-activity",
                "tolerance": 0,
                "provider": {
                    "type": "process",
                    "path": "echo",
                    "arguments": "'updated'"
                },
                "controls": [
                    {
                        "name": "prompt",
                        "provider": {
                            "type": "python",
                            "module": "chaossimpleinteract.control"
                        }
                    }
                ]
            }
        ]
    },
    "method": [
		{
            "type": "action",
            "name": "method-activity",
            "provider": {
                "type": "process",
                "path": "echo",
                "arguments": "'updated'"
            },
            "controls": [
                {
                    "name": "prompt",
                    "provider": {
                        "type": "python",
                        "module": "chaossimpleinteract.control"
                    }
                }
            ]
        }
    ],
    "rollbacks": [
        {
            "type": "action",
            "name": "rollback-activity",
            "provider": {
                "type": "process",
                "path": "echo",
                "arguments": "'updated'"
            },
            "controls": [
                {
                    "name": "prompt",
                    "provider": {
                        "type": "python",
                        "module": "chaossimpleinteract.control"
                    }
                }
            ]
        }
    ]
}
```

This experiment contains three activities (all actions): one in the 
`steady-state-hypothesis`, one in the `method`, and one in the `rollbacks` sections. 
This should mean that you are prompted _4_ times when the experiment is executed. 
One when the steady-state hypothesis is executed at the beginning, once when the 
single step in the method is executed, once again when the steady-state hypothesis 
is analysed at the end of the experiment's method execution, and then finally when 
the single step in the rollbacks is executed.

### Seeing the "Simple" user interaction Control in action

Making sure that your `chaosinteract` Python virtual environment is still activated 
(you should see `(chaosinteract)` before your command prompt), you can now execute 
the `chaos run` command and see how your new simple user interaction control is 
applied:

```bash
(chaosinteract) $ chaos run simple-interactive-experiment.json 
[2019-03-21 17:00:54 INFO] Validating the experiment's syntax
[2019-03-21 17:00:54 INFO] Experiment looks valid
[2019-03-21 17:00:54 INFO] Running experiment: A simple sample experiment that can be executed to show controls
[2019-03-21 17:00:54 INFO] Steady state hypothesis: Assess the Steady State ... (not really in this case)
Press any key to continue ...
```
Press any key each time you're prompted to complete your experiment and ... 
perfect! You are being prompted to continue for each of the activities in your 
experiment that the simple user interaction control has been applied to. You can now 
remove the control from any activities where you do not want to be prompted.

> ***NOTE:*** If you wanted the same control applied to all activities, as was the 
> case in this experiment, then you could simply add a `controls` block to the 
> top level of your experiment and that control would be applied to all activities. 
> The `simple-interactive-experiment.json` experiment is showing how you can apply 
> the control specifically to any activity _at the activity level_.

## Extending with a "Yes or No" style user interaction

Let's go one step further. This time you'll explore and apply another Chaos Toolkit control that, this time, asks the user to _confirm_ that they would like to proceed. If they choose not to proceed then the experiment is aborted, and only the rollbacks will be run after that confirmation.

The control's code is provided in the `sources/yesorno` Python module. Once again, the code is contained in a [`control.py`](sources/yesorno/chaosyorn/control.py) file:

```python
# -*- coding: utf-8 -*-
from typing import Any, Dict, List

from chaoslib.types import Activity, Configuration, Experiment, Hypothesis, \
    Journal, Run, Secrets
from chaoslib.exceptions import InterruptExecution
import click
from logzero import logger

__all__ = ["before_activity_control"]


def before_activity_control(context: Activity, **kwargs):
    """
    Prompt for Yes or No to executing an activity.
    """
    logger.info("About to execute activity: " + context.get("name"))
    if click.confirm('Do you want to continue?'):
        logger.info("Continuing: " + context.get("name"))
    else:
        raise InterruptExecution("Experiment manually interrupted")
```

This time your control is using the `click` library's `confirm` feature to prompt the user as to whether they would like to continue and execute the activity, or abort the whole experiment. The experiment is aborted using the 
Chaos Toolkit's [`InterruptException`](https://github.com/chaostoolkit/chaostoolkit-lib/blob/master/chaoslib/exceptions.py#L41).

Install this new Chaos Toolkit extension as before by entering the following command while in the `sources/yesorno` directory, making sure you are still using the `chaosinteract` Python virtual environment:

```bash
(chaosinteract) $ pip install -e .
```

Now you can head over to the `samples` directory and run the `yorn-interactive-experiment.json` experiment to use this new control.

## Removing a control does not stop the experiment executing

One thing that is good to be aware of is that Chaos Toolkit controls are declared as _optional_. This means that if a control is not present, then it is simply ignored and the experiment is still considered valid and executable. This is the opposite of Chaos Toolkit drivers that provide probes and actions. If a probe or an action is declared to use a particular driver module's functions and it cannot be found when the experiment is executed or validated, then the experiment will n ot run and will fail validation. Not so with controls.

To see this in action, try removing your `chaostoolkit-chaosyorn` control's module with the following command:

```bash
(chaosinteract) $ pip uninstall chaostoolkit-chaosyorn
Uninstalling chaostoolkit-chaosyorn-0.1.0:
  Would remove:
    /.../chaostoolkit-chaosyorn.egg-link
Proceed (y/n)? y
  Successfully uninstalled chaostoolkit-chaosyorn-0.1.0
```

Now try running the experiment that uses the `chaostoolkit-chaosyorn` control and you will see the following:

```bash
chaos run yorn-interactive-experiment.json
[2019-03-22 16:09:45 INFO] Validating the experiment's syntax
[2019-03-22 16:09:45 WARNING] Could not find Python module 'chaosyorn.control' in control 'prompt'. Did you install the Python module? The experiment will carry on running nonetheless.
[2019-03-22 16:09:45 INFO] Experiment looks valid
[2019-03-22 16:09:45 INFO] Running experiment: A simple sample experiment that can be executed to show controls
[2019-03-22 16:09:45 INFO] Steady state hypothesis: Assess the Steady State ... (not really in this case)
[2019-03-22 16:09:45 INFO] Probe: hypothesis-activity
[2019-03-22 16:09:45 INFO] Steady state hypothesis is met!
[2019-03-22 16:09:45 INFO] Action: dangerous-method-activity
[2019-03-22 16:09:45 INFO] Steady state hypothesis: Assess the Steady State ... (not really in this case)
[2019-03-22 16:09:45 INFO] Probe: hypothesis-activity
[2019-03-22 16:09:45 INFO] Steady state hypothesis is met!
[2019-03-22 16:09:45 INFO] Let's rollback...
[2019-03-22 16:09:45 INFO] Rollback: rollback-activity
[2019-03-22 16:09:45 INFO] Action: rollback-activity
[2019-03-22 16:09:45 INFO] Experiment ended with status: completed
```

If a Chaos Toolkit Control is referenced, but not available, a warning is issued 
and added to the `chaostoolkit.log`. Controls are considered _optional_ and so 
this is expected. This gives you the power to create experiments that can involve 
human interaction when a human is around, and to turn that off completely when 
the experiment is being executed in an environment when a human being around is 
unlikely (i.e. CI/CD).

## Summary

In this article you saw how to implement two [Control][control] extensions to 
the [Chaos Toolkit][chaostoolkit]. The Control extension is a powerful way of seizing 
control (literally) of the execution of your experiments. The examples here showed how you can 
harness that power to incorporate human interaction styles to your automated 
chaos experiments.
