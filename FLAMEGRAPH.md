This tutorial shows how to generate flamegraph for your service run on benchmark.

# install perf to any node you want to monitor, beaware that the linux-tools version is dependent on your linux version.
```
uname -r
sudo apt-get update
sudo apt-get install linux-tools-5.4.0-67-generic
```

# start monitor for 60 seconds or forever until you interrupt with ctl+c
```
sudo perf record -F 99 -a -g -- sleep 60
sudo perf record -F 99 -a -g
```

# run your service
```
sudo docker run \
    --env description="My first BuzzBlog experiment." \
    --volume $(pwd):/usr/local/etc/BuzzBlogBenchmark \
    --volume $(pwd):/var/log/BuzzBlogBenchmark \
    --volume $(pwd)/.ssh:/home/$(whoami)/.ssh \
    rodrigoalveslima/buzzblog:benchmarkcontroller_v0.1
```

# prepare the flamegraph package
```
git clone https://github.com/brendangregg/FlameGraph
```

# generate the flamegraph
```
sudo rm -rf ./FlameGraph/perf.data
mv perf.data ./FlameGraph
cd FlameGraph
sudo perf script | ./stackcollapse-perf.pl > out.perf-folded
./flamegraph.pl out.perf-folded > node0perf-kernel.svg
cd ..
```

# copy the flame graph to your local directory
```
scp -i {ssh-key} -o StrictHostKeyChecking=no {remote-machine}:{file-path}/node0perf-kernel.svg {local-machine-path}/original-13min-tp20
```

# copy data to your local directory
```
sudo chmod a+rwx perf.data
mv perf.data node0perf.data

scp -i {ssh-key} -o StrictHostKeyChecking=no {remote-machine}:{file-path}/node0perf-kernel.svg {local-machine-path}/node0perf.data
```

# clean everything for service rerun
```
sudo rm -rf BuzzBlogBenchmark*
sudo docker rm loadgen
sudo rm -rf /tmp/*

sudo docker container ls -a
## kill all running containers
sudo docker kill $(sudo docker ps -q)
## rm all exited containers
sudo docker rm $(sudo docker ps -a -q)
```
