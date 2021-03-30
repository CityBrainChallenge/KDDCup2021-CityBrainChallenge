.. _Gym:

Gym api
=======================
The Gym api is the same as the OpenAi Gym.

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

======
step
======


.. code-block:: python

    observation, reward, dones, info = env.step(action)


`step`:
    - env.step(action)
    - return observation, reward, info, dones


`action`:
    - a dict
    - {agent_id: phase}
    - Here phase is a integer between 0 and 8

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
    - {agent_id: reward}

    .. code-block::

        # list of tuple，value indicating (in_number of this step, out_number of this step)
        # The length is 24. The order of their roads is defined in 'signal' part of roadnet file
        # the order is :inroad0lane0, inroad0lane1, inroad0lane2, inroad1lane0 ... inroad3lane2, outroad0lane0, outroad0lane1 ...
        # If there is -1 in signal part of roadnet file, then the lane of this road is filled with three -1.
        0:[(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0)，(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0),(0,0),(0,1),(1,0),(0,0)]

`info`:
    - a dict
    - {vehicle_id: vehicle_info}

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
    - {agent_id: bool_value}
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


===========
gym_cfg.py
===========

.. code-block:: python

    class gym_cfg():
        def __init__(self):
            self.cfg = {
                'observation_features':['lane_speed','lane_vehicle_num']
            }

`self.cfg`:
    - store the configuration of gym
    - 'observation_features' indicates the return observation feature of the gym instance. Currently `lane_speed`, `lane_vehicle_num` is available


=========
Example
=========


Here is a simple usage of the api

.. code-block:: python

    env = gym.make(
        'CBEngine-v0',
        simulator_cfg_file=simulator_cfg_file,
        thread_num=1,
        gym_dict=gym_cfg_instance.cfg
    )

    for i in range(100):
        print("{}/{}".format(i,mx_step))
        obs, rwd, dones, info = env.step({0: 1})
        for k,v in info.items():
            print("{}:{}".format(k,v))

