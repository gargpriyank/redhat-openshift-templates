# Sample Red Hat OpenShift templates

- [How to use templates](#how-to-use-templates)
    - [java-spring-mongodb-kafka-template](#java-spring-mongodb-kafka-template)
        - [Prerequisites](#prerequisites)
        - [Deploy](#deploy)
        - [Test](#test)

## How to use the templates

The sample Red Hat OpenShift templates are available in templates folder in the form of `.yaml` files. Each `.yaml` constitutes Service, Route, ImageStream, BuildConfig, DeploymentConfig, etc. objects to deploy the application provided as Github URL.

### java-spring-mongodb-kafka-template

This sample `yaml` template deploys the spring boot application at https://github.com/gargpriyank/java-spring-mongodb-kafka-example to connect to IBM Event Streams (Kafka) and IBM Databases for MongoDB. For more information, visit https://github.com/gargpriyank/java-spring-mongodb-kafka-example/blob/master/README.md.

#### Prerequisites

1. Install OpenShift CLI plugin to run `oc` commands.

2. Either generate a new JKS or copy `/jre/lib/security/cacerts` from JDk installation directory and rename it to `keystore.jks`.

3. Update the JKS password and import the SSL certificate of IBM Databases for MongoDB. IBM Event Streams (Kafka) will use the default JDK certificate.
```bash
keytool -storepasswd -keystore keystore.jks
keytool -importcert -trustcacerts -file mongodb.pem -keystore keystore.jks -alias mongodb -storepass <secure_password>
```

#### Deploy

1. Login to OpenShift account and point to your selected namespace.

```bash
oc login --token=<private_token> --server=<server_url>
oc project dev
```

2. Create application secrets.

```bash
oc create secret generic java-spring-mongodb-kafka-keystore-cert-secret --from-file=keystore.jks
oc create secret generic java-spring-mongodb-kafka-secret --from-literal=jvm-secret="-Djavax.net.ssl.trustStore=/etc/keystore/cert/keystore.jks -Djavax.net.ssl.trustStorePassword=<secure_password>" --from-literal=mongo-db-url="<database_url starting with mongodb://>" --from-literal=mongo-db-name="<database_name>" --from-literal=es_kafka_topic_name="<kafka_topic_name>" --from-literal=es_kafka_service_name='<es_kafka_service_creds>'
```

3. Execute the below command to deploy the sample spring boot application in OpenShift cluster.

```bash
oc process -f templates/java-spring-mongodb-kafka-template.yaml | oc apply -f -
```

#### Test

The application saves and retrieves employee data and can be accessed through the endpoint `<OpenShift Generated Route URL>/employee`.
1. Send a POST request with following JSON to save employee data. The POST request sends the data to Kafka topic. Kafka consumer listens to the message and save it into MongoDB.
```bash
{
	"name": "..",
	"address": "...",
	"deptName": "..."
}
```
2. Send a GET request to retrieve all the employees.