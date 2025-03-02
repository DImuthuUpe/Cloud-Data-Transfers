### This directory contains instructions to transfer HLS Datasets to any Cloud Data Storage

### Note: 
For the Egress cross reduction, NASA S3 buckets only provide data for clients live in US-West-2 region. Because of that, this transfer platform should be installed in US-West-2 VMs. All the egress transfer cost is billed to your account

### Transfer Architecture
Below diagram shows the top level transfer architecture to move data from NASA S3 buckets to any cloud resource endpoint you select

![Transfer Architecture](transfer-architecture.jpg)

### Deployment

Deployment of the platform includes installation of Data Transfer Layer mentioned in the above daigram and registering source and destination storages in the Data Transfer Layer

#### Prerequisites

To have a minimal transfer layer, there should be 2 EC2 VMS (one for MFT Master and one for MFT Agent) created inside us-west-2 region. These VMs should have following features
* Both should have a public host name / ip assigned to ssh from outside
* Both should share a private VPC with private ips for each VM
* Should be initialized with Ubuntu Server 22.04 LTS (HVM) image
* You need to be able to ssh into both VMs as ubuntu user

#### Installation

* Make a copy of scrips/inventories/example/hosts.example to scrips/inventories/example/hosts and update public ip, private ip and agent id fields accordingly. Private ip is the ip VM gets through the private VPC. 
* Update security groups of master vm as below. Update 172.31.32.0/20 with the subnet of your VPC

![Security Groups](security-groups.png)

* Next we install Master, Agent and Endpoint Configurations through set of ansible scripts
* To setup the ansible environment, you need to have python3 installed in your laptop. For that, run following commands

```
cd scripts
python3 -m venv ENV
source ENV/bin/activate
pip install -r requirements.txt
```

* Install transfer orchestration components into the master node by running the following command

```
ansible-playbook -i inventories/example install-master.yml
```

* Install the transfer agent into the agent node
```
ansible-playbook -i inventories/example install-agents.yml
```
* To make sure that all installed properly, go to http://<master-public-ip>:8080 from the browser and you should be able to see Airflow dashboard. Username is airflow and password is airflow. Make sure that you update the password immediately to secure the access

* Finally, install and setup transfer orchestration workflow by running following. For that, you need to have an account created at https://urs.earthdata.nasa.gov and installation script will ask for those credentials. This is required to generate access credentials for NASA S3 endpoint
```
ansible-playbook -i inventories/example configure-endpoints.yml
```

* Then go back to Airflow dashboard, and you should be able to see a workflow named "transfer-workflow". Enable it and click the trigger workflow button in the top right corner. It will start transferring sample dataset mentioned in the scripts/roles/catalog/files/sample-data.csv

![Transfer Workflow](transfer-workflow.png)