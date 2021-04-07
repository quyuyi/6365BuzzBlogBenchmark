## TODO
1. Update Dockerfile for `controller/` and `loadgen`
Download BuzzBlogApp from our modified version.

2. <s>Upload container images to docker registery</s> \
Build image for controller, loadgen, apigateway, recommendation service
```bash
# build from Dockerfile
cd controller/
docker build -t benchmarkcontroller .
docker images
docker tag <IMAGE ID> quyuyi/buzzblog:benchmarkcontroller_v0.1
docker push quyuyi/buzzblog:benchmarkcontroller_v0.1

# build from container
docker commit <container name: recommendation_database>
docker images
docker tag <IMAGE ID> quyuyi/buzzblog:database_v0.1
docker push quyuyi/buzzblog:database_v0.1
```
- `quyuyi/buzzblog:apigateway_v0.1`
- `quyuyi/buzzblog:recommendation_v0.1`
- `quyuyi/buzzblog:database_v0.1`
- `quyuyi/buzzblog:benchmarkcontroller_v0.1`
- `quyuyi/buzzblog:loadgen_v0.1`

3. <s>Change `controller/conf/workload.yml`</s>

4. <s>Change docker parameters in `controller/conf/system.yml`</s> \
- node0
    - containers: loadgen
    - monitors: collectl
- node1
    - containers: loadbalancer, apigateway
    - monitors: collectl, tcplife-bpfcc, radvisor
- node2
    - containers: recommendation_service
    - monitors: collectl, tcplife-bpfcc, radvisor
- node3:
    - containers: recommendation_database
    - monitors: collectl, tcplife-bpfcc, radvisor

5. <s>Add function for requesting recommendation service in `loadgen/loadgen.py`.</s> \
Constructed a list of keywords for search job: 100 top keywords + 50 keywords that do not occur in the keywords field of the dataset.

6. <s>Add `setup_mongo_database` in `run_experiment.py`.</s>

7. Check `analysis/notebooks/BuzzBlogExperimentAnalysis.ipynb`

8. Use the tutorial topology or create our own topology?

### Run Notes
In local machine,
```bash
./tutorial_setup.sh \
    --username quyuyi \
    --private_ssh_key_path ~/.ssh/cloudlab \
    --node_0 apt132.apt.emulab.net \
    --node_1 apt136.apt.emulab.net \
    --node_2 apt130.apt.emulab.net \
    --node_3 apt131.apt.emulab.net
```

SSH to node-0,
```bash
ssh quyuyi@apt132.apt.emulab.net

sudo docker run \
    --env description="My first BuzzBlog experiment." \
    --volume $(pwd):/usr/local/etc/BuzzBlogBenchmark \
    --volume $(pwd):/var/log/BuzzBlogBenchmark \
    --volume $(pwd)/.ssh:/home/$(whoami)/.ssh \
    quyuyi/buzzblog:benchmarkcontroller_v0.1
```

From local,
```bash
# use our own workload.yml and system.yml instead of the ones pulled by tutorial_setup.sh
scp ../controller/conf/* ssh quyuyi@apt132.apt.emulab.net:~
```


## Questions
1. `BuzzBlogBenchmark/docs/CLOUDLAB.md` set bash as default linux shell in *emulab*?

2. monitor tools
    - bpfcc: toolkit for efficient kernel tracing and manipulation programs
    - collectl: collect performance data
    - radvisor

