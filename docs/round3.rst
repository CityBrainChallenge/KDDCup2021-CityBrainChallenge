.. _round3:

Round3
========

In round3, participants could use the large-scale cloud computing platform. Here we list main change

1. Now participants could submit their own ``CBEngine_round3`` whether in training or in evaluating. Make sure that your dimension of `observation` is aligned in ``gym_cfg.py``. Note that **only** `observation` and `reward` could be modified. You could continue using old `observations`, but the old `reward` can't be used in `rllib` because `rllib` needs single value of each agent as `reward`. Anyway, we provide 2 `rewards`, "pressure" and "queue length", along with the old `reward` in the comment of default `CBEngine_round3.py``.
#. Now the current step is not included in ``observation`` by default. It is now included in ``obs['info']['step']``
#. The observation format is modified to align with rllib api. Please look up to the `observation <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_
#. Now the keys (i.e. agent_id) of ``actions``, ``reward``, ``observation``, ``dones`` are `str` instead of `int`.

#. Now return a dict: `observation`.