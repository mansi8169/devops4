**Docker compose for multi containe app:**

app/app.py

from flask import Flask

import psycopg2

app = Flask(\_\_name\_\_)

@app.route('/')

def index():

&nbsp;   try:

&nbsp;       conn = psycopg2.connect(host='db', database='mydb', user='myuser',  password='mypassword')

&nbsp;       cur = conn.cursor()

&nbsp;       cur.execute('SELECT version();')

&nbsp;       db\_version = cur.fetchone()

&nbsp;       cur.close()

&nbsp;       conn.close()

&nbsp;       return f'Connected to PostgreSQL: {db\_version}'

&nbsp;   except Exception as e:

&nbsp;       return f'Failed to connect to PostgreSQL: {e}'

if \_\_name\_\_ == '\_\_main\_\_':

&nbsp;   app.run(debug=True, host='0.0.0.0')

app/requirements.txt

flask

psycopg2-binary

Dockerfile:

FROM python:3.9-slim

WORKDIR /app

RUN apt-get update \&\& \\

&nbsp;   apt-get install -y postgresql-client \&\& \\

&nbsp;   rm -rf /var/lib/apt/lists/\*

COPY app/ .

RUN pip install --no-cache-dir -r requirements.txt

COPY wait-for-postgres.sh /wait-for-postgres.sh

RUN chmod +x /wait-for-postgres.sh

CMD \["/wait-for-postgres.sh", "db", "python", "app.py"]

docker-compose.yml:

version: '3.8'

services:

&nbsp; web:

&nbsp;   build: .

&nbsp;   ports:

&nbsp;     - "5001:5000"

&nbsp;   depends\_on:

&nbsp;     - db

&nbsp; db:

&nbsp;   image: postgres:14

&nbsp;   environment:

&nbsp;     POSTGRES\_DB: mydb

&nbsp;     POSTGRES\_USER: myuser

&nbsp;     POSTGRES\_PASSWORD: mypassword

&nbsp;   volumes:

&nbsp;     - pgdata:/var/lib/postgresql/data

volumes:

&nbsp; pgdata:

wait-for-postgres.sh:

\#!/bin/bash

set -e

host="$1"

shift

cmd="$@"

until PGPASSWORD=mypassword psql -h "$host" -U "myuser" -d "mydb" -c '\\q'; do

&nbsp; >\&2 echo "Postgres is unavailable - sleeping"

&nbsp; sleep 1

done



>\&2 echo "Postgres is up - executing command"

exec $cmd

run: docker-compose down -v-->docker-compose up --build-->open http://localhost:5001





**CI/CD:**

Create Jenkins Project Folder

mkdir jenkins-docker

cd jenkins-docker

This folder will contain the docker-compose.yml file which defines how Jenkins runs in Docker.

docker-compose.yml

version: '3.8'

services:

&nbsp; jenkins:

&nbsp;   image: jenkins/jenkins:lts

&nbsp;   container\_name: jenkins

&nbsp;   restart: unless-stopped

&nbsp;   ports:

&nbsp;     - "8080:8080"      # Jenkins Web UI

&nbsp;     - "50000:50000"    # For Jenkins agents

&nbsp;   volumes:

&nbsp;     - jenkins\_home:/var/jenkins\_home

&nbsp;     - /var/run/docker.sock:/var/run/docker.sock  # Jenkins can run Docker commands

volumes:

&nbsp; jenkins\_home:
Start Jenkins
docker compose up -d

docker ps

docker compose up -d → Runs Jenkins container in the background.
Access Jenkins
Open browser: http://localhost:8080
Initial Setup:
Admin Username: Admin
Get initial password:
docker exec jenkins cat /var/jenkins\_home/secrets/initialAdminPassword
Unlock Jenkins → Install suggested plugins → Create your admin user.
Part 2: Prepare Your Flask/FastAPI App
myapp/

│── app.py

│── requirements.txt

│── Dockerfile

│── tests/

│   └── test\_sample.py

│── Jenkinsfile

1\. app.py (Flask example)

from flask import Flask

app = Flask(\_\_name\_\_)

@app.route("/")

def home():

&nbsp;   return "Hello from Flask + Jenkins!"

if \_\_name\_\_ == "\_\_main\_\_":

&nbsp;   app.run(host="0.0.0.0", port=5000)

2\. requirements.txt

flask

pytest

3\. Dockerfile

FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD \["python", "app.py"]

4\. Test file (tests/test\_sample.py)

def test\_example():

&nbsp;   assert 2 + 2 == 4

Simple pytest test to verify Jenkins CI can run tests.

Part 3: Jenkins Pipeline (CI/CD)

Objective: Automate build → test → deploy using Jenkinsfile.

Jenkinsfile:
pipeline {

&nbsp;   agent any

&nbsp;   environment {

&nbsp;       DOCKER\_IMAGE = "myflaskapi:latest"

&nbsp;       CONTAINER\_NAME = "flaskapi-container"

&nbsp;   }

&nbsp;   stages {

&nbsp;       stage('Checkout') {

&nbsp;           steps {

&nbsp;               git branch: 'main', url: 'https://github.com/your-repo/myapp.git'

&nbsp;           }

&nbsp;       }

&nbsp;       stage('Build') {

&nbsp;           steps {

&nbsp;               sh 'docker build -t $DOCKER\_IMAGE .'

&nbsp;           }

&nbsp;       }

&nbsp;       stage('Test') {

&nbsp;           steps {

&nbsp;               sh 'docker run --rm $DOCKER\_IMAGE pytest'

&nbsp;           }

&nbsp;       }

&nbsp;       stage('Deploy') {

&nbsp;           steps {

&nbsp;               sh 'docker rm -f $CONTAINER\_NAME || true'

&nbsp;               sh 'docker run -d --name $CONTAINER\_NAME -p 5000:5000 $DOCKER\_IMAGE'

&nbsp;           }

&nbsp;       }

&nbsp;   }

&nbsp;   post {

&nbsp;       success {

&nbsp;           echo "✅ App deployed at http://localhost:5000"

&nbsp;       }

&nbsp;       failure {

&nbsp;           echo "❌ Build/Test/Deploy failed"

&nbsp;       }

&nbsp;   }

}
Part 4: Create Pipeline Job in Jenkins

Jenkins → New Item → Name: FlaskAPI-CI-CD

Choose Pipeline → OK

Configure:

Pipeline script from SCM

SCM: Git

Repo URL: Your GitHub repo

Script Path: Jenkinsfile

Save → Click Build Now

Part 5: Test Deployment

Open browser: http://localhost:5000

Should display: Hello from Flask + Jenkins!









**RESTAPI:**

from flask import Flask,jsonify,request

app=Flask(\_\_name\_\_)

todos=\[]

@app.route('/',methods=\['POST'])

def create\_todo():

&nbsp;   todo=request.get\_json()

&nbsp;   todos.append(todo)

&nbsp;   return jsonify(todo),201

@app.route('/',methods=\['GET'])

def get\_all\_todos():

&nbsp;   return jsonify(todos)

@app.route('//<int:todo\_id>',methods=\['GET'])

def get\_todo(todo\_id):

&nbsp;   for todo in todos:

&nbsp;       if todo\['id']==todo\_id:

&nbsp;           return jsonify(todo)

&nbsp;   return jsonify({'error':'To-Do not found'}),404

@app.route('//<int:todo\_id>',methods=\['PUT'])

def update\_todo(todo\_id):

&nbsp;   updated\_data=request.get\_json()

&nbsp;   for i,todo in enumerate(todos):

&nbsp;       if todo\['id']==todo\_id:

&nbsp;           todos\[i]=updated\_data

&nbsp;           return jsonify(updated\_data)

&nbsp;   return jsonify({'error':'To-Do not found'}),404

@app.route('//<int:todo\_id>',methods=\['DELETE'])

def delete\_todo(todo\_id):

&nbsp;   for i,todo in enumerate(todos):

&nbsp;       if todo\['id']==todo\_id:

&nbsp;           del todos\[i]

&nbsp;           return jsonify({'message':'Deleted success'})

&nbsp;   return jsonify({'error':'To-Do not found'}),404

if \_\_name\_\_=='\_\_main\_\_':

&nbsp;   app.run(debug=True)

cmd: pip install flask -> python main.py -->will get link --> open powershell --> curl.exe -X POST http://127.0.0.1:5000 -H "Content-Type: application/json" -d '{\\"id\\":      

>> 1, \\"title\\": \\"Learn Flask\\", \\"completed\\": false}' --> curl.exe http://127.0.0.1:5000 







**dockerize flask /fastapi:**

app.py:

from flask import Flask  

app = Flask(\_\_name\_\_)     

@app.route('/')           

def hello():

&nbsp;   return 'Hello from Dockerized Flask!' 

if \_\_name\_\_ == '\_\_main\_\_': 

&nbsp;   app.run(host='0.0.0.0', port=5000) 

requirements.txt:

flask

Dockerfile:

FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

CMD \["python", "app.py"]

then: docker build -t \[username]/myapp:latest .  --> docker run -d -p 5050:5000 mansi8169/myapp:latest --> docker login(automatically)-->docker push mansi8169/myapp:latest



Ansible:
pip install ansible
pip install docker docker-compose
Verify:
ansible --version
docker --version
docker-compose --version
mkdir ansible-deploy-demo &amp;&amp; cd ansible-deploy-demo
ssh-keygen -t rsa -b 4096 -f ./ansible_demo_key -N ""
Dockerfile.node
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y openssh-server python3 python3-pip python3-venv && mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
docker-compose.yml:
version: "3"
services:
  node1:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node1
    ports:
      - "2222:22"

  node2:
    build:
      context: .
      dockerfile: Dockerfile.node
    container_name: node2
    ports:
      - "2223:22"
then --> docker-compose up -d --build
docker exec -i node1 bash -lc 'mkdir -p /root/.ssh && cat >> /root/.ssh/authorized_keys' < ./ansible_demo_key.pub
docker exec -i node2 bash -lc 'mkdir -p /root/.ssh && cat >> /root/.ssh/authorized_keys' < ./ansible_demo_key.pub
docker exec node1 chmod 700 /root/.ssh && docker exec node1 chmod 600 /root/.ssh/authorized_keys
docker exec node2 chmod 700 /root/.ssh && docker exec node2 chmod 600 /root/.ssh/authorized_keys

ssh-keygen -R "[127.0.0.1]:2222"
ssh -i ./ansible_demo_key -p 2222 root@127.0.0.1 "python3 --version"
ssh-keygen -R "[127.0.0.1]:2223"
ssh -i ./ansible_demo_key -p 2223 root@127.0.0.1 "python3 --version"
inventory.ini:
[node_servers]
node1 ansible_host=127.0.0.1 ansible_port=2222 ansible_user=root ansible_ssh_private_key_file=./ansible_demo_key ansible_python_interpreter=/usr/bin/python3
node2 ansible_host=127.0.0.1 ansible_port=2223 ansible_user=root ansible_ssh_private_key_file=./ansible_demo_key ansible_python_interpreter=/usr/bin/python3
app.py:
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Ansible-deployed Flask app!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
requirements.txt:
flask
deploy-python-app.yml:
- name: Deploy Python Flask app
  hosts: node_servers
  become: yes

  tasks:
    - name: Ensure /opt/myapp exists
      file:
        path: /opt/myapp
        state: directory

    - name: Copy application files
      copy:
        src: ./app.py
        dest: /opt/myapp/app.py

    - name: Copy requirements.txt
      copy:
        src: ./requirements.txt
        dest: /opt/myapp/requirements.txt

    - name: Create virtualenv
      command: python3 -m venv /opt/myapp/venv
      args:
        creates: /opt/myapp/venv

    - name: Install dependencies
      command: /opt/myapp/venv/bin/pip install -r /opt/myapp/requirements.txt

    - name: Run Flask app (systemd service)
      copy:
        dest: /etc/systemd/system/myapp.service
        content: |
          [Unit]
          Description=Flask App
          After=network.target

          [Service]
          ExecStart=/opt/myapp/venv/bin/python /opt/myapp/app.py
          WorkingDirectory=/opt/myapp
          Restart=always
          User=root

          [Install]
          WantedBy=multi-user.target

    - name: Reload systemd
      command: systemctl daemon-reload

    - name: Enable and start service
      systemd:
        name: myapp
        enabled: yes
        state: started
run playbook--> ansible-playbook -i inventory.ini deploy-python-app.yml
test app-->curl http://localhost:5000 should see--> Hello from Ansible-deployed Flask app!







1st Pract:
What is DevOps?
DevOps is a cultural and technical movement that unifies software development (Dev) and
IT operations (Ops). It emphasizes collaboration, automation, continuous delivery, and
rapid feedback loops to deliver high-quality software faster and more reliably.
Why DevOps Matters in Modern Software Development
1. Faster Time to Market
 DevOps practices like Continuous Integration (CI) and Continuous Deployment
(CD) automate code integration, testing, and deployment.
 This reduces manual intervention, accelerates release cycles, and ensures that updates
and new features are delivered faster.
Example: Amazon deploys code every 11.7 seconds using DevOps pipelines.
2. Improved Collaboration and Communication
 DevOps breaks down silos between development, QA, and operations teams.
 Encourages shared responsibility, transparency, and cross-functional
collaboration.
 Teams work toward a common goal: delivering value to the end user.
3. Higher Quality and Reliability
 Automated testing, monitoring, and deployment reduce human errors.
 Early bug detection in CI/CD pipelines ensures issues are caught before production.
 Promotes practices such as Infrastructure as Code (IaC) and automated rollback
for system stability.
4. Scalability and Flexibility
 Cloud-based DevOps tools allow teams to scale infrastructure dynamically.
 Microservices and containerization (e.g., Docker, Kubernetes) enable modular,
scalable software architecture.
5. Continuous Feedback and Improvement
 Real-time monitoring, logging, and analytics provide insights into system
performance and user behavior.
 Enables teams to make data-driven decisions and improve user experience iteratively.
6. Cost Efficiency

 Automation minimizes repetitive manual tasks and reduces downtime.
 Efficient resource utilization leads to lower infrastructure and maintenance costs.
7. Security Integration (DevSecOps)
 DevOps encourages embedding security practices into the development lifecycle.
 Continuous security testing (static and dynamic analysis) ensures compliance and
reduces vulnerabilities.
Key DevOps Tools
Purpose Tools
Version Control Git, GitHub, GitLab
CI/CD Jenkins, CircleCI, Travis CI, GitHub Actions
Containerization Docker, Podman
Orchestration Kubernetes, OpenShift
Configuration Management Ansible, Chef, Puppet
Monitoring &amp; Logging Prometheus, Grafana, ELK Stack, Datadog
Cloud Platforms AWS, Azure, Google Cloud Platform
DevOps Lifecycle Stages
1. Plan – Define and design product roadmap.
2. Develop – Code, build, and review.
3. Integrate – Merge and test code regularly.
4. Test – Automated and manual testing.
5. Release – Deploy applications to production.
6. Operate – Monitor infrastructure and applications.
7. Monitor – Collect metrics and logs for performance analysis.
8. Feedback – Learn and iterate based on performance and user input.
Real-World Impact of DevOps
 Netflix uses DevOps to handle massive traffic while ensuring zero downtime.
 Etsy reduced deployment time from hours to minutes with DevOps practices.
 NASA uses DevOps for reliable deployments in its Jet Propulsion Laboratory.
Conclusion
DevOps is no longer just a trend—it&#39;s a fundamental shift in how software is developed,
tested, and delivered. In today’s fast-paced digital world, organizations that adopt DevOps
are better positioned to deliver value quickly, innovate continuously,

1] https://www.docker.com/products/docker-desktop/ - download docker desktop for
windows/Mac/linux (accordingly)

app.py-->print
Dockerfile CODE:
# Use the official Python image
FROM python:3.11-slim

# Set display (required for GUI apps to work in Docker, may need extra setup on
Mac)
ENV DISPLAY=:0

# Install Tkinter
RUN apt-get update &amp;&amp; apt-get install -y python3-tk

# Set the working directory in the container
WORKDIR /app

# Copy your Python script into the container

COPY DockerDemo.py .

# Run the Python script
CMD [&quot;python&quot;, &quot;DockerDemo.py&quot;]
run: docker build -t &lt;virtual Direcory name&gt; .
: build image of your code. NOTE: -t is tagging the build with the ‘python_main’ name
and ‘.’ Symbolises your current working directory I think
docker run &lt;virtual Directory Name&gt;: finally run the container using the tagged
name
















