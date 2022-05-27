# docs: kubernetes/java
#kubernetes #java
## snippets
#snippets 

### structure java logs
```bash
kubectl logs pod | less -r
```

###  jstat
```bash
ps -ef | grep -i xmx

jstat -heap <pid>
# --> count all numbers ending with C

jstat -gcutil <pid>
```

## Springboot
#springboot

### properties
- override property with a ENV -> e.g. `imgix.api.domain` is `IMGIX_API_DOMAIN`

### actuator
- get monitoring-user credentials

```bash
kubectl exec -it ${pod} -- /bin/bash

# get ${monitoring-user} & ${monitoring-pw} from properties file
cat /config/application-test.properties
```

- curl to actuator backend

```bash
curl -s --user ${moitoring-user}:${monitoring-pw} localhost:8080/actuator

curl -s ... | jq '' | grep mail | sort
```

- open actuator in browser on `localhost:10080/actuator/env` (login with username/password)

```bash
ssh -L 10080:localhost:10080 ${ssh-host}

kubectl port-forward ${pod} 10080:8080
```