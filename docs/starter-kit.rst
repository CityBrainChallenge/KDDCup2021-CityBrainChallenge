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

    # simple demoNote that **only** `observation` and `reward` could be modified. Please make sure that the dimension of `observation` is aligned with ``gym_cfg.py``. You could continue using the `observations` defined in the qualification phase, but the previous `reward` can't be used in `rllib` because `rllib` requires that each agent to be assigned with a `reward`. We provide 2 demo `rewards` definitions, "pressure" and "queue length", along with the old `reward` in the comment of default `CBEngine_round3.py``.
    demo.py

Participants should implement their algorithm in agent.py. In the final phase, custom ``CBEngine_round3`` is available. Participants can **only** revise the observation and reward if they choose to use the rllib interface (Participants are also allowed not to use rllib interface to implement their own algorithm).

1. Participants can submit their own ``CBEngine_round3`` for training or evaluation. Note that **only** `observation` and `reward` could be modified. Please make sure that the dimension of `observation` is aligned with ``gym_cfg.py``. You could continue using the `observations` defined in the qualification phase, but the previous `reward` can't be used in `rllib` because `rllib` requires that each agent to be assigned with a `reward`. We provide 2 demo `rewards` definitions, "pressure" and "queue length", along with the old `reward` in the comment of default ``CBEngine_round3.py``.
#. Now the current step is not included in ``observation`` by default. It is now included in ``obs['info']['step']``
#. The observation format is modified to align with rllib api. For more information, please refer to the `observation <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_
#. Now the keys (i.e. agent_id) of ``actions``, ``reward``, ``observation``, ``dones`` are `str` instead of `int`.

#. Now `env.reset` return a dict: `observation`.
