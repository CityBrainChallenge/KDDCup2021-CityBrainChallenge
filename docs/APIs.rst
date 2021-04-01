.. _APIs:

API functions
=======================
The Gym api is the same as the OpenAi Gym. We build out environment based on OpenAI Gym API.


===============
Initialize
===============
.. code-block:: python

    env = gym.make(
            'CBEngine-v0',
            simulator_cfg_file=simulator_cfg_file,
            thread_num=1,
            gym_dict=gym_cfg_instance.cfg
        )

`simulator_cfg_file`:
    - the path of simulator.cfg
    - be used for initialize engine

`thread_num`:
    - the thread number used for engine

`gym_dict`:
    - the configuration used for initialize gym
    - a dict
    - stored in /agent/gym_cfg.py

===========
gym_cfg.py
===========

gym_cfg.py is in the ``agent`` folder. It defines the configuration of gym environment. Currently it contains `observation space`. There are 2 options in `observation space`: `lane_speed` , `lane_vehicle_num`. You must write at least one of them. The observation that you get from ``env.step()`` is defined by this configuration.

.. code-block:: python

    class gym_cfg():
        def __init__(self):
            self.cfg = {
                'observation_features':['lane_speed','lane_vehicle_num']
            }


`self.cfg`:
    - store the configuration of gym
    - 'observation_features' indicates the return observation feature of the gym instance. Currently `lane_speed`, `lane_vehicle_num` is available


======
step
======


.. code-block:: python

    observation, reward, dones, info = env.step(action)


``step``:
    - env.step(action)
    - return observation, reward, info, dones


`action`:
    - a dict
    - set `agent_id` to some `phase`
    - {`agent_id`: `phase`}
    - Here phase is a integer between 1 and 8
    - The initial phase of all agents are 1
    - If action didn't set an agent's phase, it will be the phase of last step.

`observation`:
    - a dict
    - Here key is "{}_{}".format(agentid,feature)  where feature is given by *gym_cfg.py*.

    .. code-block::

        # For lane_speed [13, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2]
        # The first value is current step
        # There are 24 lanes left. The order of their roads is defined in 'signal' part of roadnet file
        # the order is :inroad0lane0, inroad0lane1, inroad0lane2, inroad1lane0 ... inroad3lane2, outroad0lane0, outroad0lane1 ...
        # If there is -1 in signal part of roadnet file, then the lane of this road is filled with three -1.
        # -2 indicating there's no vehicle on this lane

        # for lane_vehcile_num [13, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,]
        # The first value is current step
        # There are 24 lanes left. The order of their roads is defined in 'signal' part of roadnet file
        # the order is :inroad0lane0, inroad0lane1, inroad0lane2, inroad1lane0 ... inroad3lane2, outroad0lane0, outroad0lane1 ...
        # If there is -1 in signal part of roadnet file, then the lane of this road is filled with three -1.

        "12530758427_lane_speed": [13, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2],
        "12530758427_lane_vehicle_num" : [13, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,],
        ...


`reward`:
    - a dict
    - {`agent_id`: `reward`}

    .. code-block::

        # list of tuple,value indicating (in_number of this step, out_number of this step)
        # The length is 24. The order of their roads is defined in 'signal' part of roadnet file
        # the order is :inroad0lane0, inroad0lane1, inroad0lane2, inroad1lane0 ... inroad3lane2, outroad0lane0, outroad0lane1 ...
        # If there is -1 in signal part of roadnet file, then the lane of this road is filled with three -1.
        0:[(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0), (0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0)]

    here is a illustration of the lane order in observation and reward.

        .. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/roadnet_lanes.jpg
            :align: center

            lane order


`info`:
    - a dict
    - {`vehicle_id`: `vehicle_info`}

    .. code-block::

        "info": {
        0: {
            "distance": [259.0],
            "drivable": [29301.0],
            "road": [293.0],
            "route": [293.0, 195.0, 207.0, 5.0, 67.0, 70.0, 88.0, 92.0, 76.0, 18.0],
            "speed": [0.0],
            "start_time": [73.0],
            "t_ff": [112.716]
            },
        ...
        }

`dones`:
    - a dict
    - {`agent_id`: `bool_value`}
    - indicating whether an agent is end

========
reset
========

.. code-block:: python

    observation , info = env.reset()



`reset`:
    - env.reset()
    - return (observation, info)
    - reset the engine

==================
Other interface
==================

We offer 2 extra interface .

``set_warning(flag)``:
    - flag == False to turn off the warning of invalid phase

``set_log(flag)``:
    - flag == False to turn off log to debug if you need. If you do so, the score function won't work.


