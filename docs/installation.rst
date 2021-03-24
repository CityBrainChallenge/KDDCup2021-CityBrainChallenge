.. installation:

Installation Guide
==========================

Windows
--------

1. make sure you have installed cmake and pybind11
.. code-block::

    sudo apt install -y build-essential cmake
    pip install pybind11

2. install citypb and kdd_gym
.. code-block::

    cd ./CBEngine/core
    pip install -e .
    cd ../env
    pip install -e .

3. use gym environment
.. code-block::

    env = gym.make(
        'kdd_gym-v0',
        cfg_file=cfg_file,
        thread_num=1
    )