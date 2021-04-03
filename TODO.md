## TODO
1. Check `analysis/notebooks/BuzzBlogExperimentAnalysis.ipynb`

2. <s>Upload container images to docker registery</s> \
`quyuyi/buzzblog:apigateway_v0.1`, `quyuyi/buzzblog:recommendation_v0.1`

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

5. <s>Add function for requesting recommendation service in `loadgen/loadgen.py`</s>

6. Use the tutorial topology or create our own topology?



## Questions
1. `BuzzBlogBenchmark/docs/CLOUDLAB.md` set bash as default linux shell in *emulab*?

2. monitor tools
    - bpfcc: toolkit for efficient kernel tracing and manipulation programs
    - collectl: collect performance data
    - radvisor

