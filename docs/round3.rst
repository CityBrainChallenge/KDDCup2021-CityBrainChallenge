.. _round3:

Final phase
=================

In the final phase, a large-scale cloud computing platform is provided for all qualified teams. Followings are some details on computing resource and starter-kit for the final phase

===============
Computing resource allocation
===============

Basic computing resource
- Each team will be assigned a computing cluster with 248 CPU cores, 640GB memory, and 1 TB hard disk storage as basic computing resource. 
- The basic computing resource will allow each team to run at least 30 simulator instances in parallel. 

Bonus resource allocation
- We will arrange 3 rounds of bonus resource allocation on 06/15, 06/20 and 06/25, respectively.
- Top 10 teams by 2:00 PM UTC-12 of the allocation day can apply for bonus computing resources (up to 144 CPU cores,  384GB memory per team). 
- Between 2:00 PM and 10:00 PM of the allocation day, we will recycle and re-allocate the bonus computing resources. Participants cannot access the computing resources during this period. 

Bonus resource application
- To apply for bonus computing resource, the team leader needs to send an application email to citybrainchallenge@gmail.com before 2:00 PM UTC-12  of the allocation day. Please use the same email address you used for registration.
- The email only needs to contain a title: "Computing resource request - team XX (team name)" (Email content is not needed).
- We will review all applications and grant bonus resources based on the CPU usage of a team in the previous allocation round and send a reponse by 10:00 PM UTC-12 of the allocation day.


===============
How to use the computing resource
===============

Login
- The top 20 qualified teams' leaders will receive the login credentials along with the confirmation emails.
- The login credential includes IP address, user name, and password, which you can use to login to the assigned computing cluster (A ubuntu system is pre-installed).

Model development
- `Ray library <https://rise.cs.berkeley.edu/projects/ray/>`_ and `RLlib library <https://rise.cs.berkeley.edu/projects/rllib/>`_ are default packages to support distributed model training. Sample codes of training models using RLlib are provided. 
- Participants can also use their own preferred distributed computing packages

Result and submission
- Participants need to download their training log, results and models to their local storage.  It is the responsibility of the participants to ensure the security of the data.
- Participants still need to submit their model via the official website to get their leaderboard scores and official ranking.

======================
Starter-kit
======================

Participant will get a ``starter-kit``. It contains::

    # The examples of agent
    agent/agent.py
    agent/agent_MP.py
    agent/agent_rllib.py
    agent/checkpoint-25

    # CBEngine config file
    agent/gym_cfg.py

    # Your custom CBEngine
    agent/CBEngine_round3.py

    # sample traffic flow data and road network data
    data/flow_round3.txt
    ...
    data/roadnet_round3.txt
    ...

    # demo script for generating sample traffic flow data
    data/traffic_generator.py

    # where you store your model
    model/

    # scoring script for single flow
    evaluate.py

    # summarize the result of evaluation your solution on multiple traffic flow settings
    summarize.py

    # evaluation and scoring script
    evaluate.sh

    # rllib train example
    rllib_train.py

    # example script for using rllib_train.py
    train.sh

    # rllib testing example
    rllib_test.py

    # script for parallel evaluating the model
    rllib_evaluate.sh

    # simple demo
    demo.py

Participants should implement their algorithm in agent.py. In the final phase, custom ``CBEngine_round3`` is available. Participants can **only** revise the observation and reward if they choose to use the rllib interface (You are also allowed not to use rllib interface to implement their own algorithm). 

1. In this phase, participants can submit their own ``CBEngine_round3`` for training or evaluation. Note that **only** `observation` and `reward` could be modified. Please make sure that the dimension of `observation` is aligned with ``gym_cfg.py``. You could continue using the `observations` defined in the qualification phase, but the previous `reward` can't be used in `rllib` because `rllib` requires that each agent to be assigned with a `reward`. We provide 2 demo `rewards` definitions, "pressure" and "queue length", along with the old `reward` in the comment of default `CBEngine_round3.py``.
#. Now the current step is not included in ``observation`` by default. It is now included in ``obs['info']['step']``
#. The observation format is modified to align with rllib api. Please look up to the `observation <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_
#. Now the keys (i.e. agent_id) of ``actions``, ``reward``, ``observation``, ``dones`` are `str` instead of `int`.

#. Now `env.reset` return a dict: `observation`.




