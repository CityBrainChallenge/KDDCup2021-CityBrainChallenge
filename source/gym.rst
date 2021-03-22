.. _gym:

Gym api
=======================
The Gym api is the same as the OpenAi Gym.

.. code-block:: c

    action: {agentid: phase}
    observation: {agentid_feature: agent_data}  (here key is "{}_{}".format(agentid,feature)  where feature is given by gym.cfg. The first item in the agent_data is current step)
    reward: {agentid : lane_out_vehicle_number}  (here indicating the out vehicle number of this step)
    dones: {agent_id: bool_value}

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
