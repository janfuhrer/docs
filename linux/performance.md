# docs: linux/performance
#performancetest 

## locust
#locust #python

Github: https://github.com/locustio/locust

install
```bash
pip3 install locust

locust -V
```

use
```bash
locust -f ${FILE}.py
```

## k6
#k6 #grafana

Github: https://github.com/grafana/k6
Samples: https://github.com/grafana/k6/tree/master/samples

install
```bash
brew install k6
```

use
```bash
k6 run ${file}.js
```