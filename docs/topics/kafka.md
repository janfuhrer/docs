tags: #kubernetes #kafka

# Kafka

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

## test producer connection to broker
### download java
- download: https://java.com/en/download/help/linux_x64_install.html
- extract and move to `~/bin/java/` (update $PATH if necessary)

### test with producer and consumer 
1. Download and extract kafka: https://kafka.apache.org/downloads
2. Compile:
```bash
cd kafka-3.2.0-src
./gradlew jar -PscalaVersion=2.13.6
```
  - If a http-proxy is used, this has to be configured in the file `~/.gradle/gradle.properties` 

```properties
systemProp.http.proxyHost=proxy.domain
systemProp.http.proxyPort=8080
systemProp.http.nonProxyHosts=localhost|127.0.0.1
systemProp.https.proxyHost=proxy.domain
systemProp.https.proxyPort=8080
systemProp.https.nonProxyHosts=localhost|127.0.0.1
```

3. Copy cluster CA certificate in a file `ca.crt`
```bash
kubectl get secret ${strimzi-cluster-name}-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt
```
4. Create a truststore with this ca-certificate
```bash
keytool -import -trustcacerts -file ca.crt -keystore truststore.jks -storepass kafka4ever
```
5. create properties file `client.properties`
```properties
security.protocol=SSL
ssl.truststore.location=./truststore.jks
ssl.truststore.password=${truststore-password}
```
6. command to produce:
```bash
./kafka-3.2.0-src/bin/kafka-console-producer.sh --broker-list ${bootstrap-ip}:9094 --topic ${topic-name} --producer.config client.properties
```
7. command to consume
```bash
./kafka-3.2.0-src/bin/kafka-console-consumer.sh --bootstrap-server ${bootstrap-ip}:9094 --topic ${topic-name} --from-beginning --consumer.config client.properties
```
8. create topic (in advance)
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: ${topic-name}
  labels:
    strimzi.io/cluster: ${strimzi-cluster-name}
spec:
  partitions: 1
  replicas: 1
```

### test with scram-sha-512 and authorization simple
1. update properties file `client.properties`
```properties
security.protocol=SASL_SSL
ssl.truststore.location=./truststore.jks
ssl.truststore.password=${truststore-password}
ssl.enabled.protocols=TLSv1.2
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="${username}" password="${password}";
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]