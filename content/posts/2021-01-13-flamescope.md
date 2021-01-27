g clone git@github.com:Netflix/flamescope.git
docker build -f Dockerfile . -t flamescope
docker run --rm -it -v /tmp/profiles:/profiles:ro -p 5000:5000 flamescope
docker run --rm -it -v /home/toast/perf:/profiles/perf.data:ro -p 5000:5000 flamescope


sudo perf record -F 99 -g -p $(pgrep -f next-start) -- sleep 60
sudo perf script -f --header > perf/stacks.test.$(date --iso-8601)