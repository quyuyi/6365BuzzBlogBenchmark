## TODO
1. Check `analysis/notebooks/BuzzBlogExperimentAnalysis.ipynb`

2. Use the tutorial topology or create our own topology?


## Done
1. Update Dockerfile for `controller/` to download BuzzBlogApp from our modified version.

2. Build and upload container images to docker hub, including
- apigateway: `quyuyi/buzzblog:apigateway_v0.1`
- recommendation service: `quyuyi/buzzblog:recommendation_v0.1`
- recommendation database: `quyuyi/buzzblog:database_v0.1`
- controller: `quyuyi/buzzblog:benchmarkcontroller_v0.1`
- loadgen: `quyuyi/buzzblog:loadgen_v0.1`
```bash
# build from Dockerfile
cd controller/
docker build -t benchmarkcontroller .
docker images
docker tag <IMAGE ID> quyuyi/buzzblog:benchmarkcontroller_v0.1
docker push quyuyi/buzzblog:benchmarkcontroller_v0.1

# build from container
docker commit <CONTAINER ID>
docker images
docker tag <IMAGE ID> quyuyi/buzzblog:database_v0.1
docker push quyuyi/buzzblog:database_v0.1
```

3. Change `controller/conf/workload.yml`

4. Change docker parameters in `controller/conf/system.yml` \
- node0
    - containers: loadgen
    - monitors: collectl
- node1
    - containers: loadbalancer, apigateway
    - monitors: collectl, tcplife-bpfcc, radvisor
- node2
    - containers: recommendation_service, {original services}
    - monitors: collectl, tcplife-bpfcc, radvisor
- node3:
    - containers: recommendation_database, {original databases}
    - monitors: collectl, tcplife-bpfcc, radvisor

5. Add function for requesting recommendation service in `loadgen/loadgen.py`. \
Constructed a list of keywords for search job: 100 top keywords + 50 keywords that do not occur in the keywords field of the dataset.


## Run Notes
In local machine,
```bash
./tutorial_setup.sh \
    --username quyuyi \
    --private_ssh_key_path ~/.ssh/cloudlab \
    --node_0 apt171.apt.emulab.net \
    --node_1 apt169.apt.emulab.net \
    --node_2 apt184.apt.emulab.net \
    --node_3 apt188.apt.emulab.net
```

SSH to node-0,
```bash
ssh quyuyi@apt171.apt.emulab.net

sudo docker run \
    --env description="My first BuzzBlog experiment." \
    --volume $(pwd):/usr/local/etc/BuzzBlogBenchmark \
    --volume $(pwd):/var/log/BuzzBlogBenchmark \
    --volume $(pwd)/.ssh:/home/$(whoami)/.ssh \
    quyuyi/buzzblog:benchmarkcontroller_v0.1
```

After the experiment is complete, in node-0,
```
tar -czf $(ls . | grep BuzzBlogBenchmark_).tar.gz BuzzBlogBenchmark_*/*
```

In local machine,
```
scp quyuyi@apt171.apt.emulab.net:BuzzBlogBenchmark_*.tar.gz .
```


## Questions
- monitor tools
    - bpfcc: toolkit for efficient kernel tracing and manipulation programs
    - collectl: collect performance data
    - radvisor
- In `controller/conf/system.yml`, what does `-c max_connections=128` mean?
```
xxx_database:
    image: "postgres:13.1 -c max_connections=128"
    options: ...
```

