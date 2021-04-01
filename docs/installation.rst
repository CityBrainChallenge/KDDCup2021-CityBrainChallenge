.. _Installation:

Installation Guide
==========================

Docker
-----------

The engine and gym environment is built in a docker image. You can pull it down to easily build the nessary environment. 
For now, the tag of latest image is 0.1.0, we will notify you if a new version is pushed.

.. code-block::

    docker login -u citybrainchallenge -p 'ChallengeBrainCity!'
    docker pull citybrainchallenge/cbengine:0.1.0

Then you can clone the code of starter kit.

.. code-block::

    git clone https://github.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge-starter-kit.git

After having pull down the docker image and clone the starter-kit, you could run a docker container and run the code in the starter-kit repo.

.. code-block::

    docker run -it -v /path/to/your/starter-kit:/starter-kit citybrainchallenge/cbengine:0.1.0 bash
    cd starter-kit
    python3 evaluate.py --input_dir agent --output_dir out --sim_cfg simulator.cfg
