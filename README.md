# BuzzBlog Benchmark
This repository contains the workload generator, scripts, and artifacts to
conduct and analyze the results of system performance experiments in the cloud
using the [BuzzBlog](https://github.com/rodrigoalveslima/BuzzBlogApp)
application.

BuzzBlog Benchmark was developed by Rodrigo Alves Lima (<ral@gatech.edu>) and is
licensed under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

## Notes for running experiment on cloudlab
[TUTORIAL.md](./docs/TUTORIAL.md) shows how to run experiments on cloudlab. This section further records some tricks based on our own experimence in testing our recommendation service on cloudlab.


Same as indicated in [TUTORIAL.md](./docs/TUTORIAL.md). In local machine, first run
```bash
./tutorial_setup.sh \
    --username <cloudlab username> \
    --private_ssh_key_path ~/.ssh/cloudlab \
    --node_0 apt<node0>.apt.emulab.net \
    --node_1 apt<node1>.apt.emulab.net \
    --node_2 apt<node2>.apt.emulab.net \
    --node_3 apt<node3>.apt.emulab.net
```


SSH to node-0, configure workload, and then start controller by pulling the image from `quyuyi/buzzblog:benchmarkcontroller_v0.1`. This is a modified version of `rodrigoalveslima/buzzblog:benchmarkcontroller_v0.1` in order to enable testing our recommendation service.
```bash
ssh <cloudlab username>@apt<node0>.apt.emulab.net

vim workload.yml

# Before run this, start recommendation database first to first load dataset into the database, see next step for details
sudo docker run \
    --env description="My first BuzzBlog experiment." \
    --volume $(pwd):/usr/local/etc/BuzzBlogBenchmark \
    --volume $(pwd):/var/log/BuzzBlogBenchmark \
    --volume $(pwd)/.ssh:/home/$(whoami)/.ssh \
    quyuyi/buzzblog:benchmarkcontroller_v0.1
```


Load database to remote database container
```bash
# In node-3, start recommendation database first
ssh <cloudlab username>@apt<node3>.apt.emulab.net

sudo docker run --name recommendation_database --publish 5436:27017 --volume /data/recommendation/db:/data/db --detach  --cpuset-cpus 0-5 --memory 6g quyuyi/buzzblog:database_v0.1

# In local machine, remember to change the address for the database server in recommendation.js
vim /path/to/recommendation.js

mongo apt<node3>.apt.emulab.net:5436 /path/to/recommendation.js

# Check database
mongo <node3>.apt.emulab.net:5436
> mongo: use myDatabase
> mongo: db.recommendations.count()
```


After the experiment is complete, in node-0, compress results
```bash
tar -czf $(ls . | grep BuzzBlogBenchmark_).tar.gz BuzzBlogBenchmark_*/*
```


In local machine, copy results from node-0
```bash
scp <cloudlab username>@apt<node0>.apt.emulab.net:BuzzBlogBenchmark_*.tar.gz .
```


In order to start another experiment, 
1. delete the loadgen container in node-0 and start over. In node-0,
    ```bash
    sudo docker rm loadgen
    ```
2. remove monotor logs. In each node,
    ```bash
    # remove all monitor data in each node
    sudo rm -rf /tmp/*
    ```
3. optionally stop and remove all containers in each node
    ```bash
    sudo docker container ls -a
    # kill all running containers
    sudo docker kill $(sudo docker ps -q)
    # rm all exited containers
    sudo docker rm $(sudo docker ps -a -q)
    ```


Notes for how to collect information and to generate flame graphs can be found at [FLAMEGRAPH.md](./FLAMEGRAPH.md).