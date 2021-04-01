.. _Tutorial:

Tutorial
============

Minimal Example
---------------

To run this enviroment, you just need to run `demo.py` in starter-kit after installation, where the `action` is empty.

.. code-block:: python

    import CBEngine
    import gym
    import agent.gym_cfg as gym_cfg
    
    # load config
    simulator_cfg_file = './cfg/simulator.cfg'
    mx_step = 3600
    gym_cfg_instance = gym_cfg.gym_cfg()

    # gym
    env = gym.make(
        'CBEngine-v0',
        simulator_cfg_file=simulator_cfg_file,
        thread_num=1,
        gym_dict=gym_cfg_instance.cfg
    )

    for i in range(mx_step):
        print("{}/{}".format(i,mx_step))
        
        # run one step simulation
        obs, rwd, dones, info = env.step({0: 1})
        
        # print observations and infos
        for k,v in obs.items():
            print("{}:{}".format(k,v))
        for k,v in info.items():
            print("{}:{}".format(k,v))



Fixed Time
--------------

Here is a simple example of fixed time agent. It use the `current_step` from observation to decide the phase.


.. code-block:: python

    class TestAgent():
        def __init__(self):
            self.now_phase = {}
            
            # pre-define time of green light
            self.green_sec = 40
            
            self.max_phase = 8
            self.last_change_step = {}
            self.agent_list = []
            self.phase_passablelane = {}
            
        ################################
        # load agent list
        # not suggest to modify this function.
        # agent_list is a list of agent_id (intersection id)
        def load_agent_list(self,agent_list):
            self.agent_list = agent_list
            self.now_phase = dict.fromkeys(self.agent_list,1)
            self.last_change_step = dict.fromkeys(self.agent_list,0)

        ################################


        def act(self, obs):
            """ !!! MUST BE OVERRIDED !!!
            """
            # here obs contains all of the observations and infos
            observations = obs['observations']
            info = obs['info']
            actions = {}


            # preprocess observations
            # get a dict observations_for_agent that contains the features of all agents.
            observations_for_agent = {}
            for key,val in observations.items():
                observations_agent_id = int(key.split('_')[0])
                observations_feature = key[key.find('_')+1:]
                if(observations_agent_id not in observations_for_agent.keys()):
                    observations_for_agent[observations_agent_id] = {}
                observations_for_agent[observations_agent_id][observations_feature] = val

            for agent in self.agent_list:
                # select the now_step
                # change phase for a certain period of time
                for k,v in observations_for_agent[agent].items():
                    now_step = v[0]
                    break
                step_diff = now_step - self.last_change_step[agent]
                if(step_diff >= self.green_sec):
                    self.now_phase[agent] = self.now_phase[agent] % self.max_phase + 1
                    self.last_change_step[agent] = now_step

                # construct actions
                actions[agent] = self.now_phase[agent]
            return actions
