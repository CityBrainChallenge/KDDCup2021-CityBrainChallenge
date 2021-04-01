.. _cbengine:

Environment - CBEngine
==================

CBEngine (City Brain Engine) is a city-scale traffic simulation environment. In this section, we will introduce the basic configurations to define the environment, including road network and traffic flow.


Data format
*******************


Roadnet File Format
''''''''''''''''''''''''''''''''''


Road network data
+++++++++++++++++++++
the road network file contains three datasets

- Intersection dataset
    consists of identification, location and traffic signal installation information about an intersection. A snippet of intersection dataset is shown below.

    .. code-block::

        92344 // total number of intersections
        30.2795476000 120.1653304000 25926073 1
        30.2801771000 120.1664368000 25926074 0
        ...


    The attributes of intersection dataset is described in details as below.

    +--------------------+----------------------+-----------------------------------------------+
    |Attribute Name      |       Example        |Description                                    |
    +====================+======================+===============================================+
    |latitude            |30.279547600          |local latitude                                 |
    +--------------------+----------------------+-----------------------------------------------+
    |longitude           |  120.1653304000      |local longitude                                |
    +--------------------+----------------------+-----------------------------------------------+
    |inter_id            |25926073              |intersection ID                                |
    +--------------------+----------------------+-----------------------------------------------+
    |signalized          |1                     |1 if traffic signal is installed, 0 otherwise  |
    +--------------------+----------------------+-----------------------------------------------+


- Road dataset
    Road datase consists information about road segments in the network. In general, there are two directions on each road segment (i.e., dir1 and dir2). A snippet of road dataset is shown as follows.


    .. code-block::

        2105 // total number of road segments
        28571560 4353988632 93.2000000000 20 3 3 1 2
        1 0 0 0 1 0 0 1 1 // dir1_mov: permissible movements of direction 1
        1 0 0 0 1 0 0 1 1 // dir2_mov: permissible movements of direction 2
        28571565 4886970741 170.2000000000 20 3 3 3 4
        1 0 0 0 1 0 0 1 1
        1 0 0 0 1 0 0 1 1

    The attributes of road dataset is described in details as below


    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |Attribute Name             |       Example         |Description                                                                                                                                                                                                                                |
    +===========================+=======================+===========================================================================================================================================================================================================================================+
    |from_inter_id              |28571560               |upstream intersection ID w.r.t. dir1                                                                                                                                                                                                       |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |to_inter_id                |  4353988632           |downstream intersection ID w.r.t. dir2                                                                                                                                                                                                     |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |length (m)                 |93.2000000000          |length of road segment                                                                                                                                                                                                                     |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |speed_limit (m/s)          |20                     |speed limit of road segment                                                                                                                                                                                                                |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir1_num_lane              |3                      |number of lanes of direction 1                                                                                                                                                                                                             |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir2_num_lane              |3                      |number of lanes of direction 2                                                                                                                                                                                                             |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir1_id                    |1                      |road segment (edge) ID of direction 1                                                                                                                                                                                                      |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir2_id                    |2                      |road segment (edge) ID of direction 2                                                                                                                                                                                                      |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir1_mov                   |1 0 0 0 1 0 0 1 1      |Every three digits form a group to indicate whether left-turn, through and right-turn movements are permissible for a lane of direction 1. For example, '0 1 1' indicate a shared through and right-turn lane.                             |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir2_mov                   |1 0 0 0 1 0 0 1 1      |Every three digits form a group to indicate whether left-turn, through and right-turn movements are permissible for a lane of direction 2.  |                                                                                              |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+



- Traffic signal dataset
    Note that, we assume that each intersection has no more than four exiting approaches. The exiting approaches 1 to 4 starting from the northern one and rotating in clockwise direction. Here, -1 indicates that the corresponding exiting approach is missing, which generally indicates a three-leg intersection.

    .. code-block::

        107 // total number of traffic signals
        1317137908 724 700 611 609 // inter_id, approach_id
        672874599 311 2260 3830 -1
        672879594 341 -1 2012 339


    The attributes of road dataset is described in details as below

    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |Attribute Name             |       Example         |Description                                                                                                                                                                                                                                |
    +===========================+=======================+===========================================================================================================================================================================================================================================+
    |inter_id                   |1317137908             |intersection ID                                                                                                                                                                                                                            |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach1_id               |  724                  |road segment (edge) ID of northern exiting approach                                                                                                                                                                                        |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach2_id               |700                    |road segment (edge) ID of eastern exiting approach                                                                                                                                                                                         |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach3_id               |611                    |road segment (edge) ID of southern exiting approach                                                                                                                                                                                        |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach4_id               |609                    |road segment (edge) ID of southern exiting approach                                                                                                                                                                                        |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

    For a traffic signal, there are at most 8 phases(1 - 8). Every phase allow 2 traffic movement to pass this intersection. Here are illustrations of the traffic movements and signal phase.

    .. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/phases.png
        :align: center

        Phase and lane ordering



Example
+++++++++++++
Here is an example 1x1 roadnet ``roadnet.txt`` .

.. code-block:: c

    5
    30 120 0 1
    31 120 1 0
    30 121 2 0
    29 120 3 0
    30 119 4 0
    4
    0 1 30 20 3 3 1 2
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    0 2 30 20 3 3 3 4
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    0 3 30 20 3 3 5 6
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    0 4 30 20 3 3 7 8
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    1
    0 1 3 5 7


Here is a Illustration of example above

.. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/roadnet.jpg
        :align: center

        Illustration of a 1x1 roadnet

Flow File Format
''''''''''''''''''''''''''''''''''



the first line of flow file is *n*, the number of flow

the following *3n* lines indicating configuration of each flow

the first line of flow configuration indicating *start_time*, *end_time*, *vehicle_interval*.

the second line of flow configuration indicating the length of route of this flow , *k*

the third line of flow configuration indicating the route of this flow. Here flow's route is defined by roads.

.. code-block:: c

    n
    flow_1_start_time	flow_1_end_time	flow_1_interval
    k_1
    flow_1_route_0	flow_1_route_1	...	flow_1_route_k1

    flow_2_start_time	flow_2_end_time	flow_2_interval
    k_2
    flow_2_route_0	flow_2_route_1	...	flow_2_route_k2

    ...

    flow_n_start_time	flow_n_end_time	flow_n_interval
    k_n
    flow_n_route_0	flow_n_route_1	...	flow_n_route_k

Here is an example flow file

.. code-block:: c

    12
    0 100 5
    2
    2 3
    0 100 5
    2
    2 5
    0 100 5
    2
    2 7
    0 100 5
    2
    4 5
    0 100 5
    2
    4 7
    0 100 5
    2
    4 1
    0 100 5
    2
    6 7
    0 100 5
    2
    6 1
    0 100 5
    2
    6 3
    0 100 5
    2
    8 1
    0 100 5
    2
    8 3
    0 100 5
    2
    8 5




Observations
*******************

Participants will be able to get a full observation of the traffic on the road network at every step, including vehicle-level information (e.g., position, speed) and lane-level information (e.g., average speed of each lane, number of vehicles on each lane). These observations will help the participants to make their decisions on the traffic signal for the next time step. Detailed description about accessing these observation features can be found in ``agent/gym_cfg.py``. 

Actions
**********************

The action to take is defined as the traffic signal phase for each intersection in the next 10 seconds. The details about the traffic signal phase setting can be found at the traffic signal dataset section of this page. There are in total eight different phases for a typical four-way intersection. However, to simplify the problem, only the first four signal phases are open to participants at this stage. You can also learn how to set the traffic signals for different intersections in the APIs page.
