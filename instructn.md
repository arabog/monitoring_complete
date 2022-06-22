
Instructions:
Create a new EC2 instance. Micro/free tier works great!
If you don't have a key pair created already in your AWS Console, you should do that now. Folllow these instructions to create the key pair and save a copy to your computer in a file named keypair.pem.
SSH into the instance using the console using the key pair.
Install the application "Prometheus" using the instructions in this tutorial.

https://codewizardly.com/prometheus-on-aws-ec2-part1/

Open up port 9090 on the EC2 instance's security groups.
Browse to the Prometheus application using the EC2 instance's hostname and port 9090 to verify it's working.

ssh into the instance e.g named prometheus instance:
ssh -i keypair.pem ubuntu@ec2-18-233-163-68.compute-1.amazonaws.com

To download prometheus:
wget https://github.com/prometheus/prometheus/releases/download/v2.19.0/prometheus-2.19.0.linux-amd64.tar.gz


To exstract d dwloaded file: 
tar xvfz prometheus-2.19.0.linux-amd64.tar.gz

cd into the folder:
prometheus-2.19.0.linux-amd64/

check version:
prometheus --version

cat prometheus.yml 

start prometheus in terminal:
./prometheus --config.file=./prometheus.yml

check prometheus in browser using the public dns with ur port:
ec2-107-20-73-211.compute-1.amazonaws.com:9090

to modify the prometheus.yml file:
nano prometheus.yml


++++++++++++++
https://github.com/prometheus/node_exporter


Exporter
Exercise: Configuring Node Exporter
Configure a simple "exporter" in an EC2 instance so that its data and metrics are available to Prometheus. Also, configure Prometheus to auto-detect EC2 instances with built-in Service Discovery so that the new instance don't need to be added manually.

Instructions:
Create a new EC2 instance for testing.
SSH into and configure the EC2 instance with node_exporter using the instructions in this tutorial.
https://codewizardly.com/prometheus-on-aws-ec2-part2/


Connect again via SSH to your Prometheus server in EC2.
Configure Prometheus to discover EC2 instances automatically following this tutorial.

https://codewizardly.com/prometheus-on-aws-ec2-part3/

View the test EC2 instance in Prometheus.

wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz

tar xvfz node_exporter-1.0.1.linux-amd64.tar.gz

cd into the node_exporter folder

start node_exporter:
./node_exporter

open another terminal:
ssh into the prometheus terminal

cd into the folder:
prometheus-2.19.0.linux-amd64/

to modify the prometheus.yml file:
nano prometheus.yml

add new job:
    - job_name: 'node'
    static_configs:
        - targets: ['dns of node_exporter instance:9100']
        eg:
        - targets: ['ec2-3-87-224-18.compute-1.amazonaws.com:9100']

save and exit d text editor

start prometheus in anoda terminal:
./prometheus --config.file=./prometheus.yml

go back to prometheus in browser and under
status click service discovery

-: Automatic monitoring:
to modify the prometheus.yml file:
nano prometheus.yml

add new job:
replace this:
    - job_name: 'node'
    static_configs:
        - targets: ['dns of node_exporter instance:9100']

with:
    - job_name: 'node'
    ec2_sd_configs:
        - region: eu-west-1
        access_key: your value
        secret_key: your value
        port: 9100

Restart Prometheus service.
sudo systemctl restart prometheus


Ansible : how to create roles and install prometheus, grafana and node-exporter

https://xavier-pestel.medium.com/ansible-how-to-create-roles-and-install-prometheus-grafana-and-node-exporter-28c142904541

https://www.youtube.com/watch?v=Qimp1wPF_zg


 - job_name: 'prometheus'
    static_configs:
    - targets: ['ec2-3-87-224-18.compute-1.amazonaws.com:9100']