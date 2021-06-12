.. _computing resource:

Computing resource
==================================

In the final phase, a large-scale cloud computing platform is provided for all qualified teams. Followings are some details on computing resource allocation and usage.

================================
Computing resource allocation
================================

Basic computing resource
----------------------------

- Each team will be assigned a computing cluster with 248 CPU cores, 640GB memory, and 1 TB hard disk storage as basic computing resource. 
- The basic computing resource will allow each team to run at least 30 simulator instances in parallel. 

Bonus resource allocation
----------------------------

- We will arrange 3 rounds of bonus resource allocation on 06/15, 06/20 and 06/25, respectively.
- Top 10 teams by 2:00 PM UTC-12 of the allocation day can apply for bonus computing resources (up to 144 CPU cores,  384GB memory per team). 
- Between 2:00 PM and 10:00 PM of the allocation day, we will recycle and re-allocate the bonus computing resources. Participants cannot access the computing resources during this period. 

Apply for bonur computing resource
--------------------------------------------------------

- To apply for bonus computing resource, the team leader needs to send an application email to citybrainchallenge@gmail.com before 2:00 PM UTC-12  of the allocation day. Please use the same email address you used for registration.
- The email only needs to contain a title: "Computing resource request - team XX (team name)" (Email content is not needed).
- We will review all applications and grant bonus resources based on the CPU usage of a team in the previous allocation round and send a reponse by 10:00 PM UTC-12 of the allocation day.


=============================================
How to use the computing resource
=============================================

Login
----------------------------

- The top 20 qualified teams' leaders will receive the login credentials along with the confirmation emails.
- The login credential includes IP address, user name, and password, which you can use to login to the assigned computing cluster (A ubuntu system is pre-installed).

Model development
----------------------------

- `Ray library <https://rise.cs.berkeley.edu/projects/ray/>`_ and `RLlib library <https://rise.cs.berkeley.edu/projects/rllib/>`_ are the default packages to support distributed model training. Sample codes of training models using RLlib are provided in ``rllib_train.py``. 
- Participants can also use their own preferred distributed computing packages

Result and submission
----------------------------

- Participants need to download their training log, results and models to their local storage.  It is the responsibility of the participants to ensure the security of the data.
- Participants still need to submit their model via the official website to get their leaderboard scores and official ranking.





