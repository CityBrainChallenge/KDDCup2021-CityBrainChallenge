.. _Installation:

Installation Guide
==========================

Docker
-----------

1. git clone https://github.com/kddcup2021tester/starter-kit.git
2. docker login -u kddcup2021tester -p 'cbengine!testing2021'
3. docker pull kddcup2021tester/cbengine:0.0.x
4. docker run -it -v /path/to/your/starter-kit:/starter-kit kddcup2021tester/cbengine:0.0.x bash
5. docker exec -it -u root container_id bash
6. python3 /starter-kit/evaluate.py --input_dir agent --output_dir out --sim_cfg simulator.cfg
