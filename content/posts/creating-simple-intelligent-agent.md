---
title: "Creating Simple Intelligent Agent: HunterAI"
author: "Adam Jordan"
date: 2019-09-16
description: "Today, I try to make an intelligent agent that can hunt monster. I name this agent: **Hunter**.
    In this devlog, I will write the understanding and design process of a very simple intelligent agent."
tags: [ArtificialConsciousness, ArtificialIntelligence, Devlog, HunterAI]
draft: false
---


## Today's Goal

In this article, I will try to make an intelligent agent that can hunt monster.
I name this agent: **Hunter**.


## Defining "Agent"

How can we build something if we do not what it is?

***What is Agent?***
Agent is anything that can be viewed as: **perceiving** environment through **sensors**; and **acting** upon environment through **actuators**.
It will run in **cycles** of perceiving, thinking, and acting.


***What is an Intelligent Agent?***
An agent that acts in a manner that causes it to be as _successful_ as it can.
This agent acts in a way that is expected to **maximise** to its **performance measure**, given the _evidence_ provided by what it perceived and whatever _built-in knowledge_ it has.
The **performance measure** defines the criterion of success for an agent.


## Designing an Intelligent Agent

Okay, we already understand what an intelligent agent is. Now, how do we start designing the agent?

When designing an intelligent agent, we can describe the agent by defining _(PEAS Descriptors)s_:

* **P**erformance -- how we measure the agent's achievements
* **E**nvironment -- what is the agent is interacting with
* **A**ctuator -- what produces the outputs of the agent
* **S**ensor -- what provides the inputs to the agent


Alternatively, we can desribe the agent by defining _(PAGE Descriptor)_:

* **P**ercepts -- the inputs to our system
* **A**ctions -- the outputs of our system
* **G**oals -- what the agent is expected to achieve
* **E**nvironment -- what the agent is interacting with


Formally, we can define structure of an Agent as:

$$Agent = Architecture + Program$$

**Architecture**: a device with sensors and actuators.

**Agent Program**: a program implementing *Agent Function* $f$ on top an architecture, where **Agent Function**: a function that maps a **sequence of perceptions** into **action**.

$$f(P) = A$$


## Designing Hunter AI

I will try to build **Hunter** as an intelligent agent that will shoot an arrow when it find a monster.

Let's describe **Hunter** with _PEAS Descriptor_:

* **P**erformance: Able to kill all monsters.
* **E**nvironment: A battefield filled with monsters.
* **A**ctuator: Hunter's bow
* **S**ensor: Hunter's vision


## Let's make a Framework

First, let's start by making a simple abstract framework implementing the above concepts.

```python
# framework.py

class Action:
    def __init__(self, id):
        self.id = id


class Perception:
    def __init__(self, id, value):
        self.id = id
        self.value = value


class Environment:
    def __init__(self):
        pass

    def acted_upon(self, action):
        pass

    def step(self):
        pass

    def update(self, state):
        pass


class Architecture:
    def __init__(self, environment):
        self.environment = environment

    def act(self, action):
        '''
        actuator function
        '''
        self.environment.acted_upon(action)

    def perceive(self):
        '''
        sensors function
        '''
        raise NotImplementedError()


class Agent:
    def __init__(self, architecture):
        self.architecture = architecture
    
    def process(self, perceptions):
        '''
        agent function, return Action
        '''
        pass

    def step(self):
        perceptions = self.architecture.perceive()
        action = self.process(perceptions)
        if action:
            self.architecture.act(action)


class World:
    def __init__(self, environment, agents):
        self.time = 0
        self.environment = environment
        self.agents = agents

    def step(self):
        self.time += 1
        self.environment.step()
        for agent in self.agents:
            agent.step()
```

## Let's start simple

Next, let's try implementing our **Hunter** world using this framework.
Because this is our first attempt, let's simplify things:

* There is one monster in battlefield.
* We only care about the monster visibility. We don't care about its position.
* When a monster appears / visible, hunter will react by shooting it.
* When a monster got hit, it will disappear.

I know that this looks oversimplified and seems very easy.
But this is important as our building foundation.


First, Let's define the action: **Shooting action**.

```python
# hunterAI.py

from framework import Action, Perception, Environment, Architecture, Agent, World

ACTION_SHOOT = Action('shoot')
```

Next, let's define the environment.
As with our simplification, there is only one state: `monster_visible`.
If `ACTION_SHOOT` is acted upon this environment, the monster will disappear, thus setting `monster_visible` to `False`.

```python
class HunterEnv(Environment):
    def __init__(self):
        self.monster_visible = False

    def acted_upon(self, action):
        if action == ACTION_SHOOT:
            print('[+] Pew! Hunter shoots the monster')
            self.update({ 'monster_visible': False })

    def update(self, state):
        if 'monster_visible' in state:
            self.monster_visible = state['monster_visible']
```

Next, let's define our agent architecture.
There is nothing changet at actuator function.
For the sensor function, we need to parse the `monster_visible` state from our environment.
This state is the perception that will be passed to our agent program.

```python
class HunterArch(Architecture):
    def perceive(self):
        monster_visible = self.environment.monster_visible
        return [Perception('monster_visible', monster_visible)]
```

Finally, let's define our agent program.
As will our simplification, when the agent perceives that a monster is visible, it will do shooting action.

```python
class HunterAgent(Agent):
    def process(self, perceptions):
        if len(perceptions) == 0:
            return None
        perception = perceptions[0]
        if perception.id == 'monster_visible' and perception.value == True:
            return ACTION_SHOOT
        else:
            return None
```

And some helpers function:
```python
def initiate():
    print('[+] Creating environment')
    env = HunterEnv()

    print('[+] Creating architecture')
    arch = HunterArch(environment=env)

    print('[+] Creating agent')
    hunter = HunterAgent(architecture=arch)

    print('[+] Creating world')
    world = World(environment=env, agents=[hunter])

    return world, hunter


def debug(world):
    print('[+] DEBUG: ({}) monster_visible: {}'.format(world.time, world.environment.monster_visible))
```


## Running our agent

Now that we finished programming our intelligent agent, let's run and play around with it.

Spawn a python interpreter, and import our `hunterAI.py` programs.

```python
$ python3

>>> from hunterAI import *

>>> world, hunter = initiate()
[+] Creating environment
[+] Creating architecture
[+] Creating agent
[+] Creating world

>>> debug(world)
[+] DEBUG: (0) monster_visible: False
```

We step one time unit period in our program by calling `world.step()`.
```python
>>> world.step()

>>> debug(world)
[+] DEBUG: (1) monster_visible: False
```

Let's try updating the environment.
We want to see if our agent react accordingly when a monster appears.

```python
>>> debug(world)
[+] DEBUG: (1) monster_visible: False

>>> world.environment.update({ 'monster_visible': True })

>>> debug(world)
[+] DEBUG: (1) monster_visible: True
```

After running step, we will see that our **Hunter** should shoot the monster.

```python
>>> debug(world)
[+] DEBUG: (1) monster_visible: True

>>> world.step()
[+] Pew! Hunter shoots the monster

>>> debug(world)
[+] DEBUG: (2) monster_visible: False
```


## Conclusion

Today, we learned about Intelligent agent and we have built a very simple intelligent agent.
In the future, we will try to extend this agent to have more advanced features.
I will also try to create a visualization for this agent.
While it seems to be still far, this is my first step towards creating an Artificial Consciousness.


## References

* https://hackernoon.com/rational-agents-for-artificial-intelligence-caf94af2cec5
* https://www.geeksforgeeks.org/agents-artificial-intelligence/
* http://www.cs.bham.ac.uk/~jxb/IAI/w4.pdf
