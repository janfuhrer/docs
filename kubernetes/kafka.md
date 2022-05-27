# docs: kubernetes/kafka
#kubernetes #kafka
## test producer connection to broker
### download java
- download: https://java.com/en/download/help/linux_x64_install.html
- extract and move to `~/bin/java/` (update $PATH if necessary)

### test with producer and consumer 
1. Download and extract kafka: https://kafka.apache.org/downloads
2. Compile:
```bash
cd kafka-3.1.0-src
./gradlew jar -PscalaVersion=2.13.6
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
ssl.truststore.password=kafka4ever
```
6. command to produce:
```bash
./kafka-3.1.0-src/bin/kafka-console-producer.sh --broker-list ${bootstrap-ip}:9094 --topic ${topic-name} --producer.config client.properties
```
7. command to consume
```bash
./kafka-3.1.0-src/bin/kafka-console-consumer.sh --bootstrap-server ${bootstrap-ip}:9094 --topic ${topic-name} --from-beginning --consumer.config client.properties
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
ssl.truststore.password=kafka4ever
ssl.enabled.protocols=TLSv1.2
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="${username}" password="${password}";
```