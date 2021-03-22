# Flow file format

the first line of flow file is *n*, the number of flow

the following *3n* lines indicating configuration of each flow

the first line of flow configuration indicating *start_time*, *end_time*, *vehicle_interval*. 

the second line of flow configuration indicating the length of route of this flow , *k*

the third line of flow configuration indicating the route of this flow. Here flow's route is defined by roads.

```
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
flow_n_route_0	flow_n_route_1	...	flow_n_route_kn
```

Here is a example flow file

```
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
```

