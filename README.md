# benthos-umh
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![GitHub Actions](https://github.com/united-manufacturing-hub/benthos-umh/workflows/main/badge.svg)](https://github.com/united-manufacturing-hub/benthos-umh/actions)

Welcome to the benthos-umh repository! This is a version of benthos maintained by the United Manufacturing Hub (UMH) to provide seamless OPC-UA integration with the [Unified Namespace](https://learn.umh.app/lesson/introduction-into-it-ot-unified-namespace/) (MQTT/Kafka). Our goal is to enhance the integration of IT and OT tools for engineers while avoiding vendor lock-in and streamlining data management processes.

## Description

`benthos-umh` is a Docker container designed to facilitate seamless OPC-UA integration with the Unified Namespace (MQTT/Kafka). It is part of the United Manufacturing Hub project and offers the following features:

- Simple deployment in Docker, docker-compose, and Kubernetes
- Connects to an OPC-UA server, browses selected nodes, and forwards all sub-nodes in 1-second intervals
- Supports a wide range of outputs, from the Unified Namespace (MQTT and Kafka) to HTTP, AMQP, Redis, NATS, SQL, MongoDB, Cassandra, or AWS S3. Check out the official [benthos output library](https://benthos.dev/docs/components/outputs/about)
- Fully customizable messages using the benthos processor library: implement Report-by-Exception (RBE) / message deduplication, modify payloads and add timestamps using bloblang, apply protobuf (and therefore SparkplugB), and explore many more options
- Integrates with modern IT landscape, providing metrics, logging, tracing, versionable configuration, and more
- Entirely open-source (Apache 2.0) and free-to-use

We encourage you to try out `benthos-umh` and explore the broader [United Manufacturing Hub](https://www.umh.app) project for a comprehensive solution to your industrial data integration needs.

## Usage

### Standalone

To use benthos-umh in standalone mode with Docker, follow the instructions in the main article provided.

1. Create a new file called benthos.yaml with the provided content
    ```yaml
    ---
    input:
      opcua:
        endpoint: 'opc.tcp://localhost:46010'
        nodeIDs: ['ns=2;s=IoTSensors']

    pipeline:
      processors:
        - bloblang: |
            root = {
              meta("opcua_path"): this,
              "timestamp_unix": timestamp_unix()
            }

    output:
      mqtt:
        urls:
          - 'localhost:1883'
        topic: 'ia/raw/opcuasimulator/${! meta("opcua_path") }'
        client_id: 'benthos-umh'
    ```
2. Execute the docker run command to start a new benthos-umh container
    `docker run --rm --network="host" -v '<absolute path to your file>/benthos.yaml:/benthos.yaml' ghcr.io/united-manufacturing-hub/benthos-umh:latest`
        
### With the United Manufacturing Hub (Kubernetes & Kafka)

To deploy benthos-umh with the United Manufacturing Hub and its OPC-UA simulator, use the provided Kubernetes manifests in UMHLens/OpenLens.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: benthos-1-config
  namespace: united-manufacturing-hub
  labels:
    app: benthos-1
data:
  benthos.yaml: |-
    input:
      umh_input_opcuasimulator: {}
    pipeline:
      processors:
        - bloblang: |
            root = {
              meta("opcua_path"): this,
              "timestamp_unix": timestamp_unix()
            }
    output:
      umh_output: 
        topic: 'ia.raw.${! meta("opcua_path") }'
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: benthos-1-deployment
  namespace: united-manufacturing-hub
  labels:
    app: benthos-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: benthos-1
  template:
    metadata:
      labels:
        app: benthos-1
    spec:
      containers:
        - name: benthos-1
          image: "ghcr.io/united-manufacturing-hub/benthos-umh:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 4195
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /ping
              port: http
          readinessProbe:
            httpGet:
              path: /ready
              port: http
          volumeMounts:
            - name: config
              mountPath: "/benthos.yaml"
              subPath: "benthos.yaml"
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: benthos-1-config
```

## Development and Testing

### Quickstart
Follow the steps below to set up your development environment and run tests:
```
$ git clone https://github.com/united-manufacturing-hub/benthos-umh.git
$ cd serverless-stack
$ nvm install
$ npm install
$ sudo apt-get install zip 
$ echo 'deb [trusted=yes] https://repo.goreleaser.com/apt/ /' | sudo tee /etc/apt/sources.list.d/goreleaser.list
$ sudo apt update
$ sudo apt install goreleaser
$ make
$ npm test
```

## License

All source code is distributed under the APACHE LICENSE, VERSION 2.0. See LICENSE for more information.

## Contact

Feel free to provide us feedback on our [Discord channel](https://discord.gg/F9mqkZnm9d).

For more information about the United Manufacturing Hub, visit [UMH Systems GmbH](https://www.umh.app). If you haven't worked with the United Manufacturing Hub before, [give it a try](https://umh.docs.umh.app/docs/getstarted/installation/)! Setting it up takes only a matter of minutes.

