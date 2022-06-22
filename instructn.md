
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
Create a new EC2 instance for testing node exporter:.
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



Exercise: Graph Memory Usage
Add a graph for memory usage to your Prometheus server.

Instructions:
Browse to your Prometheus dashboard on port 9090.
Go to the "Graph tab".
Take a look at all the options in the dropdown labeled "insert metric at cursor".
Create a few graphs to see what kind of information you can get. You can find some examples here.

https://prometheus.io/docs/prometheus/latest/querying/examples/

Create a graph that shows available memory for your EC2 instances with node_exporter running.




=======================
https://github.com/prometheus/alertmanager

-: Sending Alert Messages
Alerting Channels
How do you want to receive alerts?

Email
Chat Tool
Desktop Notifications
Phone Calls

Helpful Alerting
Alerts should, whenever possible, point the way to the source of the problem so that it can be fixed quickly.

Alerts should always include a brief description of the problem (the easier to understand, the better).
For code-related issues like run-time exceptions, a stack trace and source code line number is always appreciated.
When a URL is available to direct the troubleshooting engineer to the problem, it should be included in the alert.

Even though you could get really carried away with all the goodies you could cram into your alerts, we encourage you to keep them simple and to the point. Team member contact information is probably crossing the line. Trying to mix in cross-referenced incident data is probably out-of-scope for an alerting system. Both examples are valid and useful, but just not in the alerting context.

Alerting in Prometheus
Alerts are not available in the core installation of Prometheus. We have to install a utility called Alertmanager alongside Prometheus and configure them to talk to one another. Then, we can create alerting rules that decide when to send out alerts and to whom.

https://prometheus.io/docs/alerting/latest/alertmanager/

Alerting rules actually use the same PromQL expressions that saw in the Expression Browser.

Q: What kinds of information would be appropriate in an alert?
Quick descriptn of d problem
line number of d source code where d issue occured
stack trace from run-time exceptn
url to point to d source of the issue

Set up alert:
add another port to prometheus instance: 9093

create a file in prometheus-2.19.0.linux-amd64 folder name it :
rules.yml paste this


 groups:
   - name: AllInstances
     rules:
     - alert: InstanceDown
     
      <!-- Condition for alerting -->
       expr: [your expression here]
       for: 1m

    <!-- Annotation - addnal info labels to store more info -->
       annotations:
         title: 'Instance {{ $labels.instance}} down'
         description: '{{$labels.instance}} of job {{$labels.job}} has been down '

        <!-- Labels -additional labels to be attached to d alert -->
       labels:
         severity: 'critical'

go into d prometheus file and  the rule_files section and add d rules.ynl
nano prometheus.yml

rule_files:
    - "rules.yml

start ur prometheus server


open another terminal and ssh into d prometheus server

create a new dir inside d server and cd into it
mkdir alertmanager

install alert manager:
https://prometheus.io/download/


wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz


extract d file
tar xvfz alertmanager ......

cd into it
cd alertmanager......

start the alert manager
./alertmanager --config.file=alertmanager.yml

check the alertmanager in browser using the prometheus dns

ec2-0-00-224-18.compute-1.amazonaws.com:9093


connet d alertmanager to prometheus:
nano prometheus.yml

under the alerting section, change or comment out
alertmanager to localhost under - target line
e.g
    - targets:
        - alertmanager:9093

to
 - targets:
        - localhost:9093

start prometheus again

go and stop d alert manager server or terminal

open alertmanager.yml and edit:
nano alertmanager.yml

replace the content with:

global:
    resolve_timeout: 1m
    slack_api_url: 'https://hooks.slack.com/services/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'

route:
    receiver: 'slack-notifications'

receivers:
    - name: 'slack-notifications'
        slack_configs:

        -   channel: '#hugb'
            send_resolved: true


save and exit: ctr x, Y, then ENTER