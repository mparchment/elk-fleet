# elk-fleet

# Elastic Stack with Fleet Integration

This repository contains the Docker configuration for setting up an **Elastic Stack** (Elasticsearch, Logstash, Kibana) environment, including the integration of **Fleet** for centralized management of Elastic Agents.

## Overview

- **Elastic Stack**: A set of tools for searching, analyzing, and visualizing data in real time. It includes Elasticsearch for search and analytics, Logstash for data processing, and Kibana for visualization.
- **Elastic Agent**: A single, unified agent that collects data from various sources, eliminating the need for multiple Beats installations.
- **Fleet**: Centralized management of Elastic Agents, simplifying deployments and updates.

## Key Features

- **Centralized Management with Fleet**: Simplifies the management of Elastic Agents, allowing for easier deployment and updates.
- **Unified Data Collection with Elastic Agent**: Replaces the need for multiple Beats on client machines, streamlining data collection with less overhead.
- **Docker Configuration**: Provides a Docker-based setup for deployment and scaling of the Elastic Stack components.

## Structure

- **Elasticsearch**: The search and analytics engine at the core of the Elastic Stack.
- **Logstash**: Processes and transforms incoming data before indexing it in Elasticsearch.
- **Kibana**: Provides visualization and exploration capabilities for the data indexed in Elasticsearch.
- **Elastic Agent**: Deployed on clients to collect data from various sources within the environment.
- **Fleet**: Manages Elastic Agents from a central location.

## Credits

This repository was forked from [Eddie Mitchell's Getting Started with Elastic Stack and Docker Compose](https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose-part-2).

## Prerequisites

- Docker [not installed with `snap`]

## ELK + Fleet Setup

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/mparchment/elk-fleet.git
   cd elk-fleet
   ```

2. **Start the Containers:**

   ```bash
   docker compose up -d
   ```

   Access Kibana at https://localhost:5601. Use HTTPS and accept any certificate warnings.

3. **Acess Fleet & Output Setup**:

   - Navigate to Fleet via the Hamburger menu -> Management -> Fleet in Kibana -> Settings ->  Edit in the Outputs section.
   - Retrieve CA certificate from the cluster: `docker cp es-cluster-es01-1:/usr/share/elasticsearch/config/certs/ca/ca.crt /tmp/.`
   - Retrieve fingerprint of the CA certificate: `openssl x509 -fingerprint -sha256 -noout -in /tmp/ca.crt | awk -F"=" {' print $2 '} | sed s/://g`
   - Print CA certificate: `cat /tmp/ca.crt`
   - Update the "Elasticsearch CA trusted fingerprint" field with the obtained fingerprint.
   - Add the certificate text to the "Advanced YAML configuration" field in Fleet Settings. Begin with:

     ```yaml
       ssl:
         certificate_authorities:
           - |
     ```

     Place the printed CA certificate on the next line, ensuring correct indentation.

## Agent Setup

1. **Create Agent Policy:**

   - Navigate to Fleet in Kibana.
   - Go to the Policies tab and click on "Create policy".
   - Configure the policy according to your requirements.
   - Save the policy with a descriptive name.
   - Add *Kubernetes* as an integration.

2. **Add Elastic Agent:**

   - In Fleet, go to the Agents tab.
   - Click on "Add agent" and select the Kubernetes option.
   - Download or copy the Kubernetes manifest to your local machine.
  
3. **Apply YAML to Kubernetes:**

   Before applying the YAML configuration to Kubernetes, ensure you are logged in to Azure and have connected to the AKS cluster. Use the following commands:
   
   - **Login to Azure:** `az login`
   - **Connect to the AKS cluster:** `az aks get-credentials --name cluster-name --resource-group rg-name`

   After connecting to the AKS cluster, deploy the Agent configurations using `kubectl apply -f elastic-agent-managed-kubernetes.yml`. 

## Useful Commands

### Docker Compose

- Create and start all contains within the docker-compose.yml: `docker compose up`
- Stops all containers and removes any networks relatied to the docker-compose.yml: `docker compose down`
- Stops all containers, removes any networks along with any volumes related to the docker-compose.yml: `docker compose down -v` - use this to "start over"
- Stops all containers, but leaves environment in tact: `docker compose stop` - same result as `CTRL+C` in a running Docker terminal
- Start all containers which were stopped: `docker compose start` - use this for faster deployment 

### Kubernetes

- Create files in Kubernetes cluster from local machine: `kubectl create -f file-name.yml`
- Delete files in Kubernetes cluster: `kubectl delete -f file-name.yml`
- Get pods across all namespaces: `kubectl logs pod-name -n name-space`
- Check logs of a specific pod: `kubectl logs pod-name -n name-space`

## Troubleshooting ELK + Docker 

These are some common problems when dealing with ELK and Docker in general:

- **Docker Installation:** Ensure Docker is installed correctly. Avoid using snap for installation as it runs Docker in a sandbox with insufficient permissions.
- **Memory Allocation:** Elasticsearch requires sufficient memory to run. Allocate more than 2GB to prevent crashes.
- **Connection Issues:** If you face issues connecting to Kibana through the browser, try changing the URL scheme from HTTP to HTTPS or vice versa.




