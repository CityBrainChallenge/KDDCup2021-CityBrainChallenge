.. _Installation:

Installation Guide
==========================

Docker
-----------

The engine and gym environment is built in a docker image. You can pull it down to easily build the nessary environment.

.. code-block::

    docker login -u citybrainchallenge -p 'ChallengeBrainCity!'
    docker pull citybrainchallenge/cbengine:0.1.0

Then you can pull down the code of starter kit

.. code-block::

    git clone https://github.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge-starter-kit.git

After having pull down the docker image and clone the starter-kit, you could run a docker container and run the code in starter-kit

.. code-block::

    docker run -it -v /path/to/your/starter-kit:/starter-kit citybrainchallenge/cbengine:0.1.0 bash
    docker exec -it -u root container_id bash
    python3 /starter-kit/evaluate.py --input_dir agent --output_dir out --sim_cfg simulator.cfg
