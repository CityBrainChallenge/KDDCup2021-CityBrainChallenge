.. _gym:

Gym api
=======================
The Gym api is the same as the OpenAi Gym.


.. code-block:: python

    observation, reward, dones, info = env.step(action)




`observation`:
    - a dict
    - {agentid_feature: agent_data}
    - Here key is "{}_{}".format(agentid,feature)  where feature is given by *gym.cfg*.
    - The first item in the agent_data is current step.

`reward`:
    - a dict
    - {agent_id: lane_out_vehicle_number}
    - here indicating the out vehicle number of this step

`dones`:
    - a dict
    - {agent_id: bool_value}
    - indicating whether an agent is end

`info`:
    - empty

`action`:
    - a dict
    - {agent_id: phase}
    - Here phase is a integer between 0 and 8


Here is a simple usage of the api

.. code-block:: python

    env = gym.make(
    'kdd_gym-v0',
    cfg_file=cfg_file,
    thread_num=1
    )
    mx_step = 100
    for i in range(mx_step):
        print("{}/{}".format(i,mx_step))
        obs, rwd, dones, info = env.step({0: 1})
        for k,v in obs.items():
            print("{}:{}".format(k,v))
