.. _cbengine:

Environment - CBEngine
============================

CBEngine is a microscopic traffic simulation engine that can support city-scale road network traffic simulation. CBEngine can support fast simulation of road network traffic with thousands of intersections and hundreds of thousands of vehicles. CBEngine is developed by the team from Yunqi Academy of Engineering. This team will provide timely support for this competition. The safety distance car following and lane-changing models used in CBEngine are similar to SUMO (Simulation of Urban Mobility). The following sections describe input data format, observations and actions for the simulation engine.

Customize CBEngine Interface
**********************
In the final phase, you can customize the 'CBEngine' interface to define their own `observation` and `reward`, but you need to submit their customized `CBEngine`. Here is an example customized `CBEngine`:

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

Note that you are **not allowed** to use `self.eng.log_vehicle_info()` (otherwise, your solution will not be accepted). Here is a table of the APIs (`self.eng`) that are allowable for the final phase:

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




Data format
*******************


Roadnet File Format
''''''''''''''''''''''''''''''''''


Road network data
+++++++++++++++++++++
The road network file contains the following three datasets.

- Intersection dataset
    Intersection data consists of identification, location and traffic signal installation information about each intersection. A snippet of intersection dataset is shown below.

    .. code-block::

        92344 // total number of intersections
        30.2795476000 120.1653304000 25926073 1 //latitude, longitude, inter_id, signalized
        30.2801771000 120.1664368000 25926074 0
        ...


    The attributes of intersection dataset are described in details as below.

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
    Road dataset consists information about road segments in the network. In general, there are two directions on each road segment (i.e., dir1 and dir2). A snippet of road dataset is shown as follows.


    .. code-block::

        2105 // total number of road segments
        28571560 4353988632 93.2000000000 20 3 3 1 2
        1 0 0 0 1 0 0 1 1 // dir1_mov: permissible movements of direction 1
        1 0 0 0 1 0 0 1 1 // dir2_mov: permissible movements of direction 2
        28571565 4886970741 170.2000000000 20 3 3 3 4
        1 0 0 0 1 0 0 1 1
        1 0 0 0 1 0 0 1 1

    The attributes of road dataset are described in details as below.
    Direction 1 is <from_inter_id,to_inter_id>. Direction 2 is <to_inter_id,from_inter_id>.

    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |Attribute Name             |       Example         |Description                                                                                                                                                                                                                                |
    +===========================+=======================+===========================================================================================================================================================================================================================================+
    |from_inter_id              |28571560               |upstream intersection ID w.r.t. dir1                                                                                                                                                                                                       |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |to_inter_id                |4353988632             |downstream intersection ID w.r.t. dir1                                                                                                                                                                                                     |
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
    |dir1_mov                   |1 0 0 0 1 0 0 1 1      |every 3 digits form a permissible movement indicator for a lane of direction 1, 100 indicates a left-turn only inner lane, 010 indicates through only middle lane, 011 indicates a shared through and right-turn outer lane.               |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |dir2_mov                   |1 0 0 0 1 0 0 1 1      |every 3 digits form a lane permissible movement indicator for a lane of direction 2.                                                                                                                                                       |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+



- Traffic signal dataset
    This dataset describes the connectivity between intersection and road segments. Note that, we assume that each intersection has no more than four approaches. The exiting approaches 1 to 4 starting from the northern one and rotating in clockwise direction. Here, -1 indicates that the corresponding approach is missing, which generally indicates a three-leg intersection.

    .. code-block::

        107 // total number of signalized intersections
        1317137908 724 700 611 609 // inter_id, approach1_id, approach2_id, approach3_id, approach4_id
        672874599 311 2260 3830 -1 // -1 indicates a three-leg intersection without western approach
        672879594 341 -1 2012 339


    The attributes of road dataset is described in details as below

    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |Attribute Name             |       Example         |Description                                                                                                                                                                                                                                |
    +===========================+=======================+===========================================================================================================================================================================================================================================+
    |inter_id                   |1317137908             |intersection ID                                                                                                                                                                                                                            |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach1_id               |  724                  |road segment (edge) ID of northern exiting approach (Road_1 in example)                                                                                                                                                                    |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach2_id               |700                    |road segment (edge) ID of eastern exiting approach (Road_3 in example)                                                                                                                                                                     |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach3_id               |611                    |road segment (edge) ID of southern exiting approach (Road_5 in example)                                                                                                                                                                    |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |approach4_id               |609                    |road segment (edge) ID of western exiting approach (Road_7 in example)                                                                                                                                                                     |
    +---------------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+





Example
+++++++++++++
Here is an example 1x1 roadnet ``roadnet.txt`` .

.. code-block:: c

    5 // intersection data
    30 120 0 1 // latitude, longitude, inter_id, signalized
    31 120 1 0
    30 121 2 0
    29 120 3 0
    30 119 4 0
    4 // road data
    0 1 30 20 3 3 1 2
    1 0 0 0 1 0 0 0 1 // dir1_mov: permissible movements of direction 1
    1 0 0 0 1 0 0 0 1 // dir2_mov: permissible movements of direction 2
    0 2 30 20 3 3 3 4
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    0 3 30 20 3 3 5 6
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    0 4 30 20 3 3 7 8
    1 0 0 0 1 0 0 0 1
    1 0 0 0 1 0 0 0 1
    1 // traffic signal data
    0 1 3 5 7 // inter_id, approach1_id, approach2_id, approach3_id, approach4_id


Here provides an Illustration of example above.

.. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/roadnet.jpg
        :align: center

        Illustration of a 1x1 roadnet

Flow File Format
''''''''''''''''''''''''''''''''''

Flow file is composed by flows. Each flow is represented as a tuple (*start_time*, *end_time*, *vehicle_interval*, *route*), which means from *start_time* to *end_time*, there will be a vehicle with *route* every *vehicle_interval* seconds. The format of flows contains serval parts:


* The first row of flow file is *n*, which means the number of flow.

* The following *3n* rows indicating configuration of each flow. Each flow have 3 configuration lines.

    * The first row consists of *start_time*, *end_time*, *vehicle_interval*.

    * The second row is the number of road segments of route for this flow : *k*.

    * The third row describes the `route` of this flow. Here flow's route is defined by `roads` not `intersections`.

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

    12 // n = 12
    0 100 5 // start_time, end_time, vehicle_interval
    2 // number of road segments
    2 3 // road segment IDs
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

Participants will be able to get a full observation of the traffic on the road network at every 10 seconds, including vehicle-level information (e.g., position, speed) and lane-level information (e.g., average speed of each lane, number of vehicles on each lane). These observations will be helpful for decision-making on the traffic signal phase selection. Detailed description the features of `observation` can be found in ``agent/gym_cfg.py``.

The format of observations could be found at annotation in code blocks in `observation format <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_.

Actions
**********************

For a traffic signal, there are at most 8 phases (1 - 8). Each phase allows a pair of non-conflict traffic movement to pass this intersection. Here are illustrations of the traffic movements and signal phase.

    .. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/phases.png
        :align: center

        Phase and lane ordering

For example, if an agent is at phase 1, `lane_1` and `lane_7` along with all right turning lanes are passable. The index of the lanes in `observation` and `reward` could be found in `observation format <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_.

There are a total of 8 different types of phases for a standard four-way intersection. You can also learn how to set the traffic signals with the information given on the `APIs <https://kddcup2021-citybrainchallenge.readthedocs.io/en/latest/APIs.html#simulation-step>`_ page.

The action is defined as the traffic signal phase for each intersection to be selected at next 10 seconds. If an agent is switched to a different phase, there will be a 5 seconds period of 'all red' at the beginning of the next phase, which means all vehicles could not pass this intersection. We fix `env.step()` as 10 seconds for practical implementation consideration, which means the decision can be made every 10 seconds.


