.. _citybrainchallenge:

City Brain Challenge
========================

In this challenge, we will provide you with a city-scale road network and its traffic demand derived from real traffic data. You will be in charge of coordinating the traffic at the signalized intersections while maintaining the delay index below a predefined threshold. We will increase the traffic demand and see whether your coordination model can still survive.


===============
Problem
===============

Traffic signals coordinate the traffic movements at the intersection and a smart traffic signal coordination algorithm is the key to transportation efficiency. For a four-leg intersection (see figure below), 1 of 8 types of signal phases can be selected each period of time step, serving a pair of non-conflict traffic movements (e.g., phase-1 gives right-of-way for eastbound and westbound through traffic). In this competition, participants need to develop a model to select traffic signal phases at intersections of a road network to improve road network traffic performance.



  .. figure:: https://raw.githubusercontent.com/CityBrainChallenge/KDDCup2021-CityBrainChallenge/main/images/intersection.png
        :align: center


===============
Evaluation
===============

**Delay index** will be calculated to measure road network traffic performance. The solutions with a lower delay index will be ranked higher. For a completed trip, the delay is computed as actual travel time divided by travel time at free-flow speed. For an uncompleted trip, the free-flow speed is used to estimate the travel time of rest of the trip. The delay index is computed as average trip delay over all vehicles served: :math:`D = \frac{1}{N}\sum_{i=1}^{N}{D_{i}}`.

The trip delay :math:`D_{i}` of vehicle :math:`i` is defined as :math:`D_{i} = \frac{TT_{i} + TT_{i}^{r}}{TT_{i}^{f}}`, where, 
 - :math:`TT_i`: travel time of vehicle :math:`i`;
 - :math:`TT_{i}^{r}`: rest of trip travel time, estimated with free-flow speed;
 - :math:`TT_{i}^{f}`: full trip travel time at free-flow speed 
