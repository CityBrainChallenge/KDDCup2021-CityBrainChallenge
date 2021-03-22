# Roadnet File Format

### introduction

Roadnet file defines the roadnet strcture. Citypb's roadnet mainly consists of intersections and roads.

- *Intersection* is where roads intersects. **Here some intersections have signal (要不要告诉他们有的intersection有signal, 有的没有)**。 Users can set the phase of signal to control the traffic flow of this *intersection*.

- *Road* represent a directional road from one intersection to another intersection with some property, including 
  - id
  - length
  - number of lanes 
  - speed limit

- *Lane* is the component of *Road*. Each lane could have multiple turning direction. For example, vehicles on a particular lane could go straight or turn left. The turning direction of each lane is defined in roadnet file.



### file format

Let's say there are *n* intersections, *k* roads, *m* intersections with signal, the roadnet file format is as follow

The first line is *n* ,the number of intersections in the roadnet 

And there are *n* lines follows, indicating latitude, longitude, intersection_id, boolean value of have_signal

```
n
latitude longitude intersection_id have_signal
latitude longitude intersection_id have_signal
...
latitude longitude intersection_id have_signal
```
the next line is *k*, the number of roads

the next *3k* lines contains the road configuration. Every road have 2 lines. 

The first line indicating start_intersection, end_intersection, length speed_limit, forward_road_lanes, reverse_road_lanes, forward_road_id, reverse_road_id. 

The second line indicating the lane configuration of every lane in forward road.

The third line indicating the lane configuration of every lane in forward road.




```
k
start_intersection end_intersection length speed_limit forward_road_lanes  reverse_road_lanes forward_road_id reverse_road_id
lane_0_of_forward_road_could_turnleft lane_0_of_forward_road_could_gostraight lane_0_of_forward_road_could_turnright lane_1_of_forward_road_could_turnleft lane_1_of_forward_road_could_gostraight lane_1_of_forward_road_could_turnright...

...

start_intersection end_intersection length speed_limit forward_road_lane reverse_road_lanes forward_road_id reverse_road_id
lane_0_of_forward_road_could_turnleft lane_0_of_forward_road_could_gostraight lane_0_of_forward_road_could_turnright lane_1_of_forward_road_could_turnleft lane_1_of_forward_road_could_gostraight lane_1_of_forward_road_could_turnright...

```
the next line indicating *m*, the number of intersections with signal.

the following *m* lines indicating *intersection_id*, *road_0*, *road_1*, *road_2*, *road_3*. Here these 4 roads starts at the intersection. If the road is not 4-road intersection, we set -1 as a empty road.

```
m
intersection_id road0 road1 road2 road3
...
intersection_id road0 road1 road2 road3
```

Here's a sample roadnet file

```
5
30 120 0 1
31 120 1 0
30 121 2 0
29 120 3 0
30 119 4 0
4
0 1 30 20 3 3 1 2
1 0 0 0 1 0 0 1 1
1 0 0 0 1 0 0 1 1
0 2 30 20 3 3 3 4
1 0 0 0 1 0 0 1 1
1 0 0 0 1 0 0 1 1
0 3 30 20 3 3 5 6
1 0 0 0 1 0 0 1 1
1 0 0 0 1 0 0 1 1
0 4 30 20 3 3 7 8
1 0 0 0 1 0 0 1 1
1 0 0 0 1 0 0 1 1
1
0 1 3 5 7
```

