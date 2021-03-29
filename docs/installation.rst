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
        'CBEngine-v0',
        simulator_cfg_file=simulator_cfg_file,
        thread_num=1,
        gym_dict=gym_cfg_instance.cfg
    )

Docker
-----------
1. git clone https://github.com/kddcup2021tester/starter-kit.git
2. docker login -u kddcup2021tester -p 'cbengine!testing2021'
3. docker pull kddcup2021tester/cbengine:0.0.x
4. docker run -it -v /path/to/your/starter-kit:/starter-kit kddcup2021tester/cbengine:0.0.x bash
5. docker exec -it -u root container_id bash
6. python3 /starter-kit/evaluate.py --input_dir agent --output_dir out --sim_cfg simulator.cfg
