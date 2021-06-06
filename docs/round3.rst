.. _round3:

Round3
========

In round3, participants could use the large-scale cloud computing platform. Here we list main change

1. Now participants could submit their own ``CBEngine_round3`` whether in training or in evaluating. Make sure that your dimension of `observation` is aligned in ``gym_cfg.py``. Note that **only** `observation` and `reward` could be modified. You could continue using old `observations`, but the old `reward` can't be used in `rllib` because `rllib` needs single value of each agent as `reward`. Anyway, we provide 2 `rewards`, "pressure" and "queue length", along with the old `reward` in the comment of default `CBEngine_round3.py``.
#. Now the current step is not included in ``observation`` by default. It is now included in ``obs['info']['step']``
#. The observation format is modified to align with rllib api. Please look up to the `observation <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_
#. Now the keys (i.e. agent_id) of ``actions``, ``reward``, ``observation``, ``dones`` are `str` instead of `int`.

#. Now `env.reset` return a dict: `observation`.




Training and Evaluating
================================
We provide example codes of training in `rllib` and evaluating the model from `rllib`.


- rllib_train.py:
    - It's a example code of training model in `rllib`.
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
            default="/starter-kit/cfg/simulator.cfg",
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

        # if set '--multiflow', then the CBEngine will utilize all 6 flows.
        if(args.multiflow):
            simulator_cfg_files = ['/starter-kit/cfg/simulator_round3_flow0.cfg','/starter-kit/cfg/simulator_round3_flow1.cfg','/starter-kit/cfg/simulator_round3_flow2.cfg','/starter-kit/cfg/simulator_round3_flow3.cfg','/starter-kit/cfg/simulator_round3_flow4.cfg','/starter-kit/cfg/simulator_round3_flow5.cfg']
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
            "metric_period":args.metric_period
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
            "batch_mode": "complete_episodes",



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
    - Result will be in ``/log/$flow_number/$folder_name/$iteration``.
    - When submission, you could load the ``checkpoint-*`` file in your `agent.py`.
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
            for k in weights.keys():
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

