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

6. Add variety of requests to recommendation microservice
    - search size
    - return size

