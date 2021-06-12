.. _tryityourself:

Try it yourself
==================

Here, we provide guidelines for setting up the simulation environment and submitting results.

======================================
Installation
======================================

Installation on your local environment
----------------------------

The simulator engine and the gym environment are incorporated into the docker image. You can pull it down to easily setup the environment.
The latest image version is ``0.1.3``, we will notify you if a new version is updated.


.. code-block::

    docker pull citybrainchallenge/cbengine:0.1.3

Then you can clone the code of the starter-kit.

.. code-block::

    git clone https://github.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge-starter-kit.git

After pulled down the docker image and cloned the starter-kit, you can run a docker container and run the code of the ``starter-kit`` repo.

.. code-block::

    docker run -it -v /path/to/your/starter-kit:/starter-kit citybrainchallenge/cbengine:0.1.3 bash
    cd starter-kit
    # for evaluating single flow
    python3 evaluate.py --input_dir agent --output_dir out --sim_cfg /starter-kit/cfg/simulator_round3_flow0.cfg --metric_period 200 --threshold 1.4

Installation on the computing platform
----------------------------

================
Create sample traffic data
================

The python script for generating sample traffic is in the ``data`` folder. You can create your sample traffic flow data by executing,

.. code-block::

    python3 traffic_generator.py
    
Afterwards, you will find a newly created or updated ``flow_round3.txt`` file. 



================
Check the CBEngine
================

To check your simulation enviroment is ok, you can run ``demo.py`` in the starter-kit, where the ``actions`` are simply fixed. You need to overwrite the function of ``act()`` in ``agent.py`` to define the policy of signal phase selection (i.e., action). Also, participants could modify the CBEngine.


.. code-block:: python

    from agent.CBEngine_round3 import CBEngine_round3 as CBEngine_rllib_class
    import gym
    import agent.gym_cfg as gym_cfg

    # load config
    simulator_cfg_file = '/starter-kit/cfg/simulator_round3_flow0.cfg'
    mx_step = 360
    gym_cfg_instance = gym_cfg.gym_cfg()
    gym_configs = gym_cfg_instance.cfg
    # gym
    env_config = {
        "simulator_cfg_file": simulator_cfg_file,
        "thread_num": 8,
        "gym_dict": gym_configs,
        "metric_period": 200,
        "vehicle_info_path": "/starter-kit/log/"
    }
    env = CBEngine_rllib_class(env_config)
    env.set_info(1)
    for i in range(mx_step):
        print("{}/{}".format(i, mx_step))

        # run one step simulation
        # you can use act() in agent.py to get the actions predicted by agent.
        actions = {0: 1}
        obs, rwd, dones, info = env.step(actions)

        # print observations and infos
        # for k, v in obs.items():
        #     print("{}:{}".format(k, v))
        for k, v in info.items():
            print("{}:{}".format(k, v))



The meaning of ``simulator_cfg_file``, ``gym_cfg``,``metric_period``,``vehicle_info_path`` is explained in `APIs <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-initialization>`_


Here is a simple example of a fixed time (traffic signal is pre-timed) agent implemented at ``agent.py`` to coordinate the traffic signal. It use the `current_step` (i.e., current time step) from info to decide the phase.



.. code-block:: python

    # how to import or load local files
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
            self.green_sec = 2
            self.max_phase = 8
            self.last_change_step = {}
            self.agent_list = []
            self.phase_passablelane = {}
            self.intersections = {}
            self.roads = {}
            self.agents = {}
        ################################
        # don't modify this function.
        # agent_list is a list of agent_id
        def load_agent_list(self,agent_list):
            self.agent_list = agent_list
            self.now_phase = dict.fromkeys(self.agent_list,1)
            self.last_change_step = dict.fromkeys(self.agent_list,0)

        # intersections[key_id] = {
        #     'have_signal': bool,
        #     'end_roads': list of road_id. Roads that end at this intersection. The order is random.
        #     'start_roads': list of road_id. Roads that start at this intersection. The order is random.
        #     'lanes': list, contains the lane_id in. The order is explained in Docs.
        # }
        # roads[road_id] = {
        #     'start_inter':int. Start intersection_id.
        #     'end_inter':int. End intersection_id.
        #     'length': float. Road length.
        #     'speed_limit': float. Road speed limit.
        #     'num_lanes': int. Number of lanes in this road.
        #     'inverse_road':  Road_id of inverse_road.
        #     'lanes': dict. roads[road_id]['lanes'][lane_id] = list of 3 int value. Contains the Steerability of lanes.
        #               lane_id is road_id*100 + 0/1/2... For example, if road 9 have 3 lanes, then their id are 900, 901, 902
        # }
        # agents[agent_id] = list of length 8. contains the inroad0_id, inroad1_id, inroad2_id,inroad3_id, outroad0_id, outroad1_id, outroad2_id, outroad3_id
        def load_roadnet(self,intersections, roads, agents):
            self.intersections = intersections
            self.roads = roads
            self.agents = agents
        ################################


        def act(self, obs):
            """ !!! MUST BE OVERRIDED !!!
            """
            # here obs contains all of the observations and infos

            # observations is returned 'observation' of env.step()
            # info is returned 'info' of env.step()
            observations = obs['observations']
            info = obs['info']
            actions = {}

            now_step = info['step']
            # a simple fixtime agent

            # get actions
            for agent in self.agent_list:
                # select the now_step
                step_diff = now_step - self.last_change_step[agent]
                if(step_diff >= self.green_sec):
                    self.now_phase[agent] = self.now_phase[agent] % self.max_phase + 1
                    self.last_change_step[agent] = now_step
                actions[agent] = self.now_phase[agent]
            # print(self.intersections,self.roads,self.agents)
            return actions


Here `load_roadnet` imports the roadnet file. This infomation is also in `CBEngine_rllib` class.

.. code-block::

    intersections[key_id] = {
        'have_signal': bool,
        'end_roads': list of road_id. Roads that end at this intersection. The order is random.
        'start_roads': list of road_id. Roads that start at this intersection. The order is random.
        'lanes': list, contains the lane_id in. The order is explained in Docs.
    }
    roads[road_id] = {
        'start_inter':int. Start intersection_id.
        'end_inter':int. End intersection_id.
        'length': float. Road length.
        'speed_limit': float. Road speed limit.
        'num_lanes': int. Number of lanes in this road.
        'inverse_road':  Road_id of inverse_road.
        'lanes': dict. roads[road_id]['lanes'][lane_id] = list of 3 int value. Contains the Steerability of lanes.
                  lane_id is road_id*100 + 0/1/2... For example, if road 9 have 3 lanes, then their id are 900, 901, 902
    }
    agents[agent_id] = list of length 8. contains the inroad0_id, inroad1_id, inroad2_id,inroad3_id, outroad0_id, outroad1_id, outroad2_id, outroad3_id


====================================
Training your model with rllib
====================================

We provide example codes for training in `rllib` and evaluating the model from `rllib`.


- rllib_train.py:
    - It's an example code of training model in `rllib`.
    - In ``train.sh`` we provide a simple training command for `/starter-kit/cfg/simulator_warm_up.cfg`. You could use it to check the environment.
    - Note that the training result will be in ``model/$algorithm/$foldername/checkpoint_*/checkpoint-*``.

.. code-block:: python

    from ray import tune
    import gym
    from agent.CBEngine_round3 import CBEngine_round3 as CBEngine_rllib_class
    import citypb
    import ray
    from ray import tune
    import os
    import numpy as np
    import argparse
    import sys
    import subprocess
    parser = argparse.ArgumentParser()



    if __name__ == "__main__":
        # some argument
        parser.add_argument(
            "--num_workers",
            type=int,
            default=30,
            help="rllib num workers"
        )
        parser.add_argument(
            "--multiflow",
            '-m',
            action="store_true",
            default = False,
            help="use multiple flow file in training"
        )
        parser.add_argument(
            "--stop-iters",
            type=int,
            default=10,
            help="Number of iterations to train.")
        parser.add_argument(
            "--algorithm",
            type=str,
            default="A3C",
            help="algorithm for rllib"
        )
        parser.add_argument(
            "--sim_cfg",
            type=str,
            default="/starter-kit/cfg/simulator_round3_flow0.cfg",
            help = "simulator file for CBEngine"
        )
        parser.add_argument(
            "--metric_period",
            type=int,
            default=3600,
            help = "simulator file for CBEngine"
        )
        parser.add_argument(
            "--thread_num",
            type=int,
            default=8,
            help = "thread num for CBEngine"
        )
        parser.add_argument(
            "--gym_cfg_dir",
            type = str,
            default="agent",
            help = "gym_cfg (observation, reward) for CBEngine"
        )
        parser.add_argument(
            "--checkpoint_freq",
            type = int,
            default = 5,
            help = "frequency of saving checkpoint"
        )

        parser.add_argument(
            "--foldername",
            type = str,
            default = 'train_result',
            help = 'The result of the training will be saved in ./model/$algorithm/$foldername/. Foldername can\'t have any space'
        )

        # find the submission path to import gym_cfg
        args = parser.parse_args()
        for dirpath, dirnames, file_names in os.walk(args.gym_cfg_dir):
            for file_name in [f for f in file_names if f.endswith(".py")]:
                if file_name == "gym_cfg.py":
                    cfg_path = dirpath
        sys.path.append(str(cfg_path))
        import gym_cfg as gym_cfg_submission
        gym_cfg_instance = gym_cfg_submission.gym_cfg()
        gym_dict = gym_cfg_instance.cfg
        simulator_cfg_files=[]

        # if set '--multiflow', then the CBEngine will utilize flows in 'simulator_cfg_files'
        if(args.multiflow):
            simulator_cfg_files = [
                '/starter-kit/cfg/simulator_round3_flow0.cfg'
                ]
        else:
            simulator_cfg_files = [args.sim_cfg]
        print('The cfg files of this training   ',format(simulator_cfg_files))
        class MultiFlowCBEngine(CBEngine_rllib_class):
            def __init__(self, env_config):
                env_config["simulator_cfg_file"] = simulator_cfg_files[(env_config.worker_index - 1) % len(simulator_cfg_files)]
                super(MultiFlowCBEngine, self).__init__(config=env_config)


        # some configuration
        env_config = {
            "simulator_cfg_file": args.sim_cfg,
            "thread_num": args.thread_num,
            "gym_dict": gym_dict,
            "metric_period":args.metric_period,
            "vehicle_info_path":"/starter-kit/log/"
        }
        obs_size = gym_dict['observation_dimension']
        OBSERVATION_SPACE = gym.spaces.Dict({
            "observation": gym.spaces.Box(low=-1e10, high=1e10, shape=(obs_size,))
        })
        ACTION_SPACE = gym.spaces.Discrete(9)
        stop = {
            "training_iteration": args.stop_iters
        }
        ################################
        # modify this
        tune_config = {
            # env config
            "env":MultiFlowCBEngine,
            "env_config" : env_config,
            "multiagent": {
                "policies": {
                    "default_policy": (None, OBSERVATION_SPACE, ACTION_SPACE, {},)
                }
            },

            "num_cpus_per_worker":args.thread_num,
            "num_workers":args.num_workers,



            # add your training config

        }
        ########################################
        ray.init(address = "auto")
        local_path = './model'



        def name_creator(self=None):
            return args.foldername


        # train model
        ray.tune.run(args.algorithm, config=tune_config, local_dir=local_path, stop=stop,
                     checkpoint_freq=args.checkpoint_freq,trial_dirname_creator = name_creator)




- rllit_test.py:
    - We provide a script ``rllib_test.py`` to evaluate your model of `rllib`. You could set your own arguments to evaluate the model.
    - Again, the model file is in ``model/$algorithm/$foldername/checkpoint_*/checkpoint-*`` after training. In ``rllib_test.py``, you could set the arguments ``--algorithm``, ``--foldername``, ``--iteration`` to load and evaluate the model. You could refer to ``rllib_evaluate.sh``, which is a simple evaluating bash script to use ``rllib_test.py``.
    - Result will be in ``/log/$flow_number/$folder_name/$iteration``. Here $flow_number is the number of ``simulator_round3_flow*.cfg``.
    - When submission, you could load the ``checkpoint-*`` file in your `agent.py`. We provide an example ``agent_rllib.py`` in the starterkit.
    - Don't open lots of evaluating processes in parallel. It would cause the cloud server shutdown!!!!
    - Here is an example agent of loading the `rllib` model.

.. code-block:: python

    class RLlibTFCheckpointPolicy():
        def __init__(
            self, load_path, algorithm, policy_name, observation_space, action_space
        ):
            self._checkpoint_path = load_path
            self._algorithm = algorithm
            self._policy_name = policy_name
            self._observation_space = observation_space
            self._action_space = action_space
            self._sess = None

            if isinstance(action_space, gym.spaces.Box):
                self.is_continuous = True
            elif isinstance(action_space, gym.spaces.Discrete):
                self.is_continuous = False
            else:
                raise TypeError("Unsupport action space")

            if self._sess:
                return

            if self._algorithm == "PPO":
                from ray.rllib.agents.ppo.ppo_tf_policy import PPOTFPolicy as LoadPolicy
            elif self._algorithm in ["A2C", "A3C"]:
                from ray.rllib.agents.a3c.a3c_tf_policy import A3CTFPolicy as LoadPolicy
            elif self._algorithm == "PG":
                from ray.rllib.agents.pg.pg_tf_policy import PGTFPolicy as LoadPolicy
            elif self._algorithm in ["DQN","APEX"]:
                from ray.rllib.agents.dqn.dqn_tf_policy import DQNTFPolicy as LoadPolicy
            else:
                raise TypeError("Unsupport algorithm")

            self._prep = ModelCatalog.get_preprocessor_for_space(self._observation_space)
            self._sess = tf.Session(graph=tf.Graph())
            self._sess.__enter__()

            with tf.name_scope(self._policy_name):
                # obs_space need to be flattened before passed to PPOTFPolicy
                flat_obs_space = self._prep.observation_space
                self.policy = LoadPolicy(flat_obs_space, self._action_space, {})
                objs = pickle.load(open(self._checkpoint_path, "rb"))
                objs = pickle.loads(objs["worker"])
                state = objs["state"]
                weights = state[self._policy_name]
                list_keys = list(weights.keys())
                for k in list_keys:
                    if(k not in self.policy.get_weights().keys()):
                        weights.pop(k)
                self.policy.set_weights(weights)

        def act(self, obs):
            action = {}
            if isinstance(obs, list):
                # batch infer
                obs = [self._prep.transform(o) for o in obs]
                action = self.policy.compute_actions(obs, explore=False)[0]
            elif isinstance(obs, dict):
                for k,v in obs.items():
                    obs = self._prep.transform(v)
                    action[k] = self.policy.compute_actions([obs], explore=False)[0][0]
            else:
                # single infer
                obs = self._prep.transform(obs)
                action = self.policy.compute_actions([obs], explore=False)[0][0]

            return action


====================================
Customize CBEngine interface
====================================

In the final phase, you can customize the ``CBEngine`` interface to define your own ``observation`` and ``reward``, but you need to submit their customized ``CBEngine``. Here is an example code to customize ``CBEngine`` interface:

.. code-block:: python

    class CBEngine_round3(CBEngine_rllib_class):
    """See CBEngine_rllib_class in /CBEngine_env/env/CBEngine_rllib/CBEngine_rllib.py

    Need to implement reward.

    implementation of observation is optional

    """
    def __init__(self,config):
        super(CBEngine_round3,self).__init__(config)
        self.observation_features = self.gym_dict['observation_features']
        self.custom_observation = self.gym_dict['custom_observation']
        self.observation_dimension = self.gym_dict['observation_dimension']

    def _get_observations(self):

        if(self.custom_observation == False):
            obs = super(CBEngine_round3, self)._get_observations()
            return obs
        else:
            ############
            # implement your own observation
            #
            # Example: lane_vehicle_num
            obs = {}
            lane_vehicle = self.eng.get_lane_vehicles()
            for agent_id, roads in self.agent_signals.items():
                result_obs = []
                for lane in self.intersections[agent_id]['lanes']:
                    # -1 indicates empty roads in 'signal' of roadnet file
                    if (lane == -1):
                        result_obs.append(-1)
                    else:
                        # -2 indicates there's no vehicle on this lane
                        if (lane not in lane_vehicle.keys()):
                            result_obs.append(0)
                        else:
                            # the vehicle number of this lane
                            result_obs.append(len(lane_vehicle[lane]))
                # obs[agent_id] = {
                #     "observation" : your_observation
                # }
                # Here agent_id must be str

                obs[agent_id] = {"observation":result_obs}

            # Here agent_id must be str. So here change int to str
            int_agents = list(obs.keys())
            for k in int_agents:
                obs[str(k)] = obs[k]
                obs.pop(k)

            return obs
            ############

    def _get_reward(self):

        rwds = {}

        ##################
        ## Example : pressure as reward.
        # if(self.observation_features[0] != 'lane_vehicle_num'):
        #     raise ValueError("maxpressure need 'lane_vehicle_num' as first observation feature")
        # lane_vehicle = self.eng.get_lane_vehicles()
        # for agent_id, roads in self.agent_signals.items():
        #     result_obs = []
        #     for lane in self.intersections[agent_id]['lanes']:
        #         # -1 indicates empty roads in 'signal' of roadnet file
        #         if (lane == -1):
        #             result_obs.append(-1)
        #         else:
        #             # -2 indicates there's no vehicle on this lane
        #             if (lane not in lane_vehicle.keys()):
        #                 result_obs.append(0)
        #             else:
        #                 # the vehicle number of this lane
        #                 result_obs.append(len(lane_vehicle[lane]))
        #     pressure = (np.sum(result_obs[12: 24]) - np.sum(result_obs[0: 12]))
        #     rwds[agent_id] = pressure
        ##################

        ##################
        ## Example : queue length as reward.
        v_list = self.eng.get_vehicles()
        for agent_id in self.agent_signals.keys():
            rwds[agent_id] = 0
        for vehicle in v_list:
            vdict = self.eng.get_vehicle_info(vehicle)
            if(float(vdict['speed'][0])<0.5 and float(vdict['distance'][0]) > 1.0):
                if(int(vdict['road'][0]) in self.road2signal.keys()):
                    agent_id = self.road2signal[int(vdict['road'][0])]
                    rwds[agent_id]-=1
        # normalization for qlength reward
        for agent_id in self.agent_signals.keys():
            rwds[agent_id] /= 10

        ##################

        ##################
        ## Default reward, which can't be used in rllib
        ## self.lane_vehicle_state is dict. keys are agent_id(int), values are sets which maintain the vehicles of each lanes.

        # def get_diff(pre,sub):
        #     in_num = 0
        #     out_num = 0
        #     for vehicle in pre:
        #         if(vehicle not in sub):
        #             out_num +=1
        #     for vehicle in sub:
        #         if(vehicle not in pre):
        #             in_num += 1
        #     return in_num,out_num
        #
        # lane_vehicle = self.eng.get_lane_vehicles()
        #
        # for agent_id, roads in self.agents.items():
        #     rwds[agent_id] = []
        #     for lane in self.intersections[agent_id]['lanes']:
        #         # -1 indicates empty roads in 'signal' of roadnet file
        #         if (lane == -1):
        #             rwds[agent_id].append(-1)
        #         else:
        #             if(lane not in lane_vehicle.keys()):
        #                 lane_vehicle[lane] = set()
        #             rwds[agent_id].append(get_diff(self.lane_vehicle_state[lane],lane_vehicle[lane]))
        #             self.lane_vehicle_state[lane] = lane_vehicle[lane]
        ##################
        # Change int keys to str keys because agent_id in actions must be str
        int_agents = list(rwds.keys())
        for k in int_agents:
            rwds[str(k)] = rwds[k]
            rwds.pop(k)
    return rwds

Participants can continue using the old `observation` used in qualification phase by set ``'custom_observation' : False`` in ``gym_cfg.py``. But `reward` should be implemented because `reward` in rllib needs to be single values. We provide 2 rewards , ``pressure`` and ``queue length`` , along with the old rewards.

Note that you are **not allowed** to use ``self.eng.log_vehicle_info()`` (otherwise, your solution will not be accepted), which means that you cannot access to the information about vehicle route and travel time at speed limit. Here is a table of the APIs (e.g., ``self.eng.get_vehicles()``) that are allowable for the final phase:

+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|API                            |Returned value                 |Description                                                                                  |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_vehicle_count()            |int                            |The total number of running vehicle                                                          |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_vehicles()                 |list                           |A list of running vehicles' ids                                                              |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_lane_vehicle_count()       |dict                           |A dict. Keys are lane_id, values are number of running vehicles on this lane.                |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_lane_vehicles()            |dict                           |A dict. Keys are lane_id, values are a list of running vehicles on this lane.                |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_vehicle_speed()            |dict                           |A dict. Keys are vehicle_id of running vehicles, values are their speed                      |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_average_travel_time()      |float                          |The average travel time of both running vehicles and finished vehicles.                      |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+
|get_vehicle_info(vehicle_id)   |dict                           |Input vehicle_id, output the information of the vehicle as a dict.                           |
+-------------------------------+-------------------------------+---------------------------------------------------------------------------------------------+


=================================
Evaluation
=================================

Default evalution method
----------------------------

``evaluate.sh`` is a scoring script that output the scores of your agent in multiple sample traffic flow in parallel.

``evaluate.py`` is a scoring script that evaluate your agent only in single flow. It is similar to ``evaluate.py`` in the qualification phase.

.. code-block::

    # run evaluation on 1 set of traffic flow 
    python3 evaluate.py --input_dir agent --output_dir out --sim_cfg /starter-kit/cfg/simulator_round3_flow0.cfg  --metric_period 120 --threshold 1.4 --vehicle_info_path log


The results for multiple traffic flows will be output at ``/starter-kit/out/scores.json``, while single flow result will be output at ``/starter-kit/out/$flow_number/scores.json``. In qualification phase, your solution is evaluated every 120 seconds for scoring (i.e., metric_period=120).


Efficient evaluation for a learning-based model
----------------------------
For learning-based model, we also provide an extra more efficient evaluation framework. But you can still use the default evaluation method for submission.


===============
Results
===============

Results will be saved as ``/starter-kit/out/scores.json``, the data format of results is exemplified as follows.

.. code-block::

    {
      "success": true,
      "error_msg": "", // if "success" is false, "error_msg" stores the exception
      "data": {
        "total_served_vehicles": 1047, // if "success" is false, here it rethe replay of your intermediate results after your solution being evaluated. Here `mapbox token` and `yarn` are required. You can get a `mapbox token` by registering a mapbox account.turns -1
        "delay_index": 2.3582080966292374 // if "success" is false, here it returns -1
      }
    }
    


===============
Visualization
===============

You can visualize the replay of your intermediate results after your solution being evaluated. Here `mapbox token` and `yarn` are required. You can get a `mapbox token` by registering a mapbox account.


1. The visualization process will run in your local environment (not the docker environment). To prepare for visualization, you need to install yarn (npm is required) in your local environment.

2. open the `/KDDCup2021-CityBrainChallenge-starter-kit` folder. copy the files ``lightinfo.json``, ``roadinfo.json``, ``time*.json`` in `/log` folder and paste into your newly created `/ui/src/log` folder. Here,

- ``lightinfo.json`` records the information of traffic light.
- ``roadinfo.json`` records the information of road network.
- ``time*.json`` files record the intermediate results over all time steps, for example, ``time0.json`` records the results at the first step.

3. modify `/ui/src/index.js`

.. code-block::

    mapboxgl.accessToken = Your_Token; # your mapbox default public key
    this.maxTime = max_of_time*.json # if the last file of your ``time*.json`` files is ``time359.json``, it is 359.

4. cd to `/ui` (make sure run "yarn start" in your local environment instead of docker environment)

.. code-block::

    yarn
    yarn start

the replay of your intermediate results after your solution being evaluated. Here `mapbox token` and `yarn` are required. You can get a `mapbox token` by registering a mapbox account.
5. open `localhost:3000` with your browser (If report "JavaScript heap out of memory", please refer to this `website <https://support.snyk.io/hc/en-us/articles/360002046418-JavaScript-heap-out-of-memory>`_)

Here are some Tips:​
260
5. open `localhost:3000` with your browser (If report "JavaScript heap out of memory", please refer to this `website <https://support.snyk.io/hc/en-us/articles/360002046418-JavaScript-heap-out-of-memory>`_)

- *Sky blue* indicates left-turning vehicles, *dark blue* indicates going straight vehicles, and *dark green* indicates right-turning vehicles.
- Lines indicate roads. The color of the line represents the average speed of the road.
- Here's an example of an intersection in ui. The number in the center (with red background) indicates the current phase number. The number of each road segment help you to identify the permissible movements of current phase, for example, in current phase-1, 0 and 2 left-turn movements are given right-of-way. For more information about signal phase, please refer to `Action <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/cbengine.html#actions>`_.

.. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/ui_example.jpg
    :align: center





==================
Make a submission
==================




Important tips:
    In the final phase, you should also submit ``CBEngine_round3.py``. See `CBEngine_round3 <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/cbengine.html#custom-cbengine>`_. So all participants should submit ``CBEngine_round3.py``, ``agent.py``, ``gym_cfg.py``.

1. To submit the models for evaluation, participants need to modify the starter-kit and place all the model-related files (including but not limited to ``agent.py`` and deep learning model files) into the ``agent`` folder. Compress the agent folder and name it as ``agent.zip`` to make the submission. Note that you need to directly compress the ``agent`` folder, rather than a group of files.

2. Participants need to train their models offline and submit the trained models along with ``agent.py``, which will load them.

3. All submissions should follow the format of our sample code in starter-kit . Hence, please do not modify any file outside the ``agent`` folder, except the ``.cfg`` file (The ``.cfg`` file can be revised to incorporate different training traffic).

4. If your model need to import or load some files, please put them to the ``agent`` folder and make sure to use the absolute path. Examples are shown at the beginning of fixed time ``agent.py``.

5. Please also make sure to only use the packages in the given docker file, so that your code can be executed at the evaluation platform.

6. Participants can report the python package required to build the model if these packages are not included in the current docker environment. The support team will evaluate the request and determine whether to add the package to the provided docker environment.

7. Participants are responsible for ensuring that all the submissions can be successfully tested under the given evaluation framework.


