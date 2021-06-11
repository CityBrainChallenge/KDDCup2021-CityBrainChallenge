.. _citybrainchallenge:

City Brain Challenge
=========================

In this challenge, we will provide you with a city-scale road network and its traffic demand derived from real traffic data. You will be in charge of coordinating the traffic signals to maximize number of vehicles served while maintaining an acceptable delay. We will increase the traffic demand and see whether your coordination model can still survive.


===============
Problem
===============

Traffic signals coordinate the traffic movements at the intersection and a smart traffic signal coordination algorithm is the key to transportation efficiency. For a four-leg intersection (see figure below), 1 of 8 types of signal phases can be selected each period of time step, serving a pair of non-conflict traffic movements (e.g., phase-1 gives right-of-way for left-turn traffic from northern and southern approaches). In this competition, participants need to develop a model to select traffic signal phases at intersections of a road network to improve road network traffic performance.



  .. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/intersection.png
        :align: center



In the final round, a city-scale road network and 1-hour sample traffic data is provided. We use exactly the same road network but different traffic data for scoring your submissions. Participants are encouraged to use the python script to generate your own sample traffic data for training and testing since the traffic settings for evaluation is not revealed. 

===============
Evaluation
===============

Total number of vehicles served (i.e., total number of vehicles entering the network) and delay index will be computed every 120 seconds to evaluate your submissions. The evaluation process will be terminated once the delay index reaches the predefined threshold 1.40. 

The trip delay index is computed as actual travel time divided by travel time at free-flow speed. For an uncompleted trip, the free-flow speed is used to estimate the travel time of rest of the trip. The delay index is computed as average trip delay index over all vehicles served: :math:`D = \frac{1}{N}\sum_{i=1}^{N}{D_{i}}`.

The trip delay :math:`D_{i}` of vehicle :math:`i` is defined as :math:`D_{i} = \frac{TT_{i} + TT_{i}^{r}}{TT_{i}^{f}}`, where, 
 - :math:`TT_i`: travel time of vehicle :math:`i`;
 - :math:`TT_{i}^{r}`: rest of trip travel time, estimated with free-flow speed;
 - :math:`TT_{i}^{f}`: full trip travel time at free-flow speed 

We will evaluate your solution on multiple traffic flow settings. The overall `total number of served vehicles' is computed as the summation of `total number of served vehicles' over all evaluation scenarios. The overall `delay index' is computed as the average delay index among all vehicles over all scenarios.

The submission scoring and ranking process follows three principles:

 - Solutions that served more vehicles will rank higher.
 - If two solutions served the same number of vehicles, the one with lower delay index will rank higher.
 - If two solutions served the same number of vehicles with same delay index, the one submitted earlier will rank higher.

======================
Qualfication round Starter-kit
======================

.. code-block::

    # Participants must implement
    agent/agent.py

    # Participants could modify
    agent/gym_cfg.py
    cfg/simulator.cfg

    # A simple demo of the environment
    demo.py

    # Scoring and simulation program
    evaluate.py

    # Other files and directory
    data/flow_1x1_txt
    data/flow_warm_up_1000.txt
    data/flow_round2.txt
    data/roadnet_1x1.txt
    data/roadnet_warm_up.txt
    data/roadnet_round2.txt
    agent/agent_DQN.py
    train_dqn_example.py
    log/
    out/

Participants should implement their algorithm in ``agent.py``. And then execute ``evaluate.py`` to get scores. Participants could modify ``simulator.cfg`` and  ``gym_cfg.py``.

======================
Final round Starter-kit
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
    data/flow_round3_flow0.txt
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

Participants should implement their algorithm in agent.py. In final phase, custom ``CBEngine_round3`` is available. Participants can **only** revise the observation and reward if they choose to use the rllib interface. Participants can also choose not to use rllib interface and implement their own algorithm. 



