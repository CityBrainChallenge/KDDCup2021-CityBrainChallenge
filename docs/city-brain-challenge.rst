.. _citybrainchallenge:

City Brain Challenge
========================

===============
Problem
===============

Traffic signal is a commonly used method to coordinate traffic movements at intersections. For a four-leg intersection (see figure below), 1 of 8 types of signal phases can be selected each time step, serving a pair of non-conflict traffic movements (e.g., phase-A gives right-of-way for eastbound and westbound through traffic). In this competition, participants need to develop a model to select signal phases at intersections of a road network to improve road network traffic performance.

===============
Evaluation
===============

Delay index will be computed to measure road network traffic performance. The solutions with lower delay index will rank higher. For a completed trip, the delay is computed as actual travel time divided by travel time at free-flow speed. For an uncompleted trip, the free-flow speed is used to estimate actual travel time of rest of the trip. The delay index is computed as average trip delay over all vehicles served.

The trip delay :math:`D_{i}` of vehicle :math:`i` is defined as :math:`D_{i} = \frac{TT_{i} + TT_{i}^{r}}{TT_{i}^{f}}`.
where 
 - :math:`TT_i`: travel time of vehicle :math:`i`;
 - :math:`TT_{i}^{r}`: rest of trip travel time, estimated with free-flow speed;
 - :math:`TT_{i}^{f}`: full trip travel time at free-flow speed 
The delay index over all vehicles served: :math:`D = \frac{1}{N}\sum_{i=1}^{N}{D_{i}}`.
