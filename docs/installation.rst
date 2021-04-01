.. _Installation:

Installation Guide
==========================

Docker
-----------

The engine and gym environment is built in a docker image. You can pull it down to easily build the nessary environment.

.. code-block::

    docker login -u kddcup2021tester -p 'cbengine!testing2021'
    docker pull kddcup2021tester/cbengine:0.0.x

Then you can pull down the code of starter kit

.. code-block::

    git clone https://github.com/kddcup2021tester/starter-kit.git

After having pull down the docker image and clone the starter-kit, you could run a docker container and run the code in starter-kit

.. code-block::

    docker run -it -v /path/to/your/starter-kit:/starter-kit kddcup2021tester/cbengine:0.0.x bash
    docker exec -it -u root container_id bash
    python3 /starter-kit/evaluate.py --input_dir agent --output_dir out --sim_cfg simulator.cfg
