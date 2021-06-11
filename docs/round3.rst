.. _round3:

Final phase
=================

In the final phase, a large-scale cloud computing platform is provided for all qualified teams. Followings are some main guidances for the final round,

1. In this round, participants can submit their own ``CBEngine_round3`` for training or evaluation. Note that **only** `observation` and `reward` could be modified. Please make sure that the dimension of `observation` is aligned with ``gym_cfg.py``. You could continue using the `observations` defined in the qualification round, but the previous `reward` can't be used in `rllib` because `rllib` requires that each agent to be assigned with a `reward`. We provide 2 demo `rewards` definitions, "pressure" and "queue length", along with the old `reward` in the comment of default `CBEngine_round3.py``.
#. Now the current step is not included in ``observation`` by default. It is now included in ``obs['info']['step']``
#. The observation format is modified to align with rllib api. Please look up to the `observation <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_
#. Now the keys (i.e. agent_id) of ``actions``, ``reward``, ``observation``, ``dones`` are `str` instead of `int`.

#. Now `env.reset` return a dict: `observation`.




