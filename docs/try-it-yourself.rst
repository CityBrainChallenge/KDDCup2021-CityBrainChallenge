.. _tryityourself:

Try it yourself
==================

===============
Installation guide
===============

Installation via Docker
-----------

The engine and gym environment is built in a docker image. You can pull it down to easily build the nessary environment.
For now, the tag of latest image is 0.1.0, we will notify you if a new version is pushed.

.. code-block::

    docker pull citybrainchallenge/cbengine:0.1.0

Then you can clone the code of starter kit.

.. code-block::

    git clone https://github.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge-starter-kit.git

After having pull down the docker image and clone the starter-kit, you could run a docker container and run the code in the starter-kit repo.

.. code-block::

    docker run -it -v /path/to/your/starter-kit:/starter-kit citybrainchallenge/cbengine:0.1.0 bash
    cd starter-kit
    python3 evaluate.py --input_dir agent --output_dir out --sim_cfg cfg/simulator.cfg


===============
Run simulation
===============

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

Here is a simple example of a fixed time agent to coordinate the traffic signal. It use the `current_step` from observation to decide the phase.


.. code-block:: python
    
    # how to import or load local files with absolute path
    import os
    import sys
    path = os.path.split(os.path.realpath(__file__))[0]
    sys.path.append(path)
    import gym_cfg
    with open(path + "/gym_cfg.py", "r") as f:
        pass
    
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


===============
Results
===============



===============
Visualization
===============

Engine could log replay file. You could follow these steps to easily use these files to get visualization of your algorithm. But `mapbox token` and `yarn` is required.


1. Put the ``lightinfo.json``, ``roadinfo.json``, ``time*.json`` from `/log` to `/ui/src/log`
2. modify `/ui/src/index.js`

.. code-block::

    mapboxgl.accessToken = Your_Token;
    this.maxTime = max_value_of_*_of_time*.json

3. cd to `/ui`

.. code-block::

    yarn start

4. open `localhost:3000` with your browser

tips:

1. Sky blue indicates left-turning cars, dark blue indicates straight ahead cars, and dark green indicates right-turning cars.
2. The color of signal is meaningless.
3. Lines indicate roads.the color of the line represents the average speed.
4. Lane is not painted, so the car may not be painted on the line.


===============
Make a submission
===============

1. To submit the models for evaluation, participants need to modify the starter-kit and place all the model-related files (including but not limited to 'agent.py' and deep learing model files) into the 'agent' folder. Compress the agent folder as 'agent.zip' to make the submission. Note that, please make sure you directly compress the 'agent' folder, rather than a group of files.

2. Note that the simulation code will have exactly the same structure as the starter-kit. Hence, please do not modify any file outside the 'agent' folder, except the '.cfg' file (The '.cfg' file can be revised to incorporate different training traffic).

3. Please also make sure to only use the packages in the given docker file, so that your code can be executed at the evaluation platform. 

4. Participants can report the python package required to build the model if these packages are not included in the current docker environment. The support team will evaluate the request and determine whether to add the package to the provided docker environment.

5. Participants are responsible for ensuring that all the submissions can be successfully tested under the given evaluation framework.

