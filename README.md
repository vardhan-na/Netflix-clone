<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project</title>
<style>
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
    max-width: 900px;
    margin: 40px auto;
    padding: 0 20px;
    line-height: 1.6;
    color: #1a1a1a;
    background: #ffffff;
  }
  h1 { font-size: 2em; border-bottom: 2px solid #eaecef; padding-bottom: 0.3em; }
  h2 { font-size: 1.5em; border-bottom: 1px solid #eaecef; padding-bottom: 0.3em; margin-top: 2em; }
  h3 { font-size: 1.2em; margin-top: 1.5em; }
  code {
    background: #f6f8fa;
    padding: 0.2em 0.4em;
    border-radius: 4px;
    font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
    font-size: 0.9em;
  }
  pre {
    background: #f6f8fa;
    padding: 16px;
    border-radius: 6px;
    overflow-x: auto;
  }
  pre code {
    background: none;
    padding: 0;
  }
  blockquote {
    border-left: 4px solid #dfe2e5;
    margin: 1em 0;
    padding: 0 1em;
    color: #6a737d;
  }
  a { color: #0366d6; text-decoration: none; }
  a:hover { text-decoration: underline; }
  hr { border: none; border-top: 1px solid #eaecef; margin: 2em 0; }
  ul, ol { padding-left: 2em; }
  li { margin-bottom: 0.3em; }
</style>
</head>
<body>

<h1>Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!</h1>

<h2>Phase 1: Initial Setup and Deployment</h2>

<h3>Step 1: Launch EC2 (Ubuntu 22.04)</h3>
<ul>
  <li>Provision an EC2 instance on AWS with Ubuntu 22.04.</li>
  <li>Connect to the instance using SSH.</li>
</ul>

<h3>Step 2: Clone the Code</h3>
<ul>
  <li>Update all the packages and then clone the code.</li>
  <li>Clone your application's code repository onto the EC2 instance:</li>
</ul>
<pre><code>git clone https://github.com/vardhan-na/DecSecOps.git</code></pre>

<h3>Step 3: Install Docker and Run the App Using a Container</h3>
<p>Set up Docker on the EC2 instance:</p>
<pre><code>sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock</code></pre>

<p>Build and run your application using Docker containers:</p>
<pre><code>docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

# to delete
docker stop &lt;containerid&gt;
docker rmi -f netflix</code></pre>

<blockquote>It will show an error because you need an API key.</blockquote>

<h3>Step 4: Get the API Key</h3>
<ul>
  <li>Open a web browser and navigate to the TMDB (The Movie Database) website.</li>
  <li>Click on "Login" and create an account.</li>
  <li>Once logged in, go to your profile and select "Settings."</li>
  <li>Click on "API" from the left-side panel.</li>
  <li>Create a new API key by clicking "Create" and accepting the terms and conditions.</li>
  <li>Provide the required basic details and click "Submit."</li>
  <li>You will receive your TMDB API key.</li>
</ul>
<p>Now recreate the Docker image with your API key:</p>
<pre><code>docker build --build-arg TMDB_V3_API_KEY=&lt;your-api-key&gt; -t netflix .</code></pre>

<hr>

<h2>Phase 2: Security</h2>

<h3>1. Install SonarQube and Trivy</h3>
<p>Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.</p>

<p><strong>SonarQube:</strong></p>
<pre><code>docker run -d --name sonar -p 9000:9000 sonarqube:lts-community</code></pre>
<p>Access it at: <code>publicIP:9000</code> (default username &amp; password: <code>admin</code>)</p>

<p><strong>Trivy:</strong></p>
<pre><code>sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy</code></pre>

<p>To scan an image using Trivy:</p>
<pre><code>trivy image &lt;imageid&gt;</code></pre>

<h3>2. Integrate SonarQube and Configure</h3>
<ul>
  <li>Integrate SonarQube with your CI/CD pipeline.</li>
  <li>Configure SonarQube to analyze code for quality and security issues.</li>
</ul>

<hr>

<h2>Phase 3: CI/CD Setup</h2>

<h3>1. Install Jenkins for Automation</h3>
<p>Install Jenkins on the EC2 instance to automate deployment.</p>

<p><strong>Install Java:</strong></p>
<pre><code>sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version</code></pre>
<p>Expected output:</p>
<pre><code>openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)</code></pre>

<p><strong>Install Jenkins:</strong></p>
<pre><code>sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list &gt; /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins</code></pre>

<p>Access Jenkins in a web browser using the public IP of your EC2 instance: <code>publicIp:8080</code></p>

<h3>2. Install Necessary Plugins in Jenkins</h3>
<p>Go to <strong>Manage Jenkins &rarr; Plugins &rarr; Available Plugins</strong>, then install:</p>
<ol>
  <li>Eclipse Temurin Installer (Install without restart)</li>
  <li>SonarQube Scanner (Install without restart)</li>
  <li>NodeJs Plugin (Install without restart)</li>
  <li>Email Extension Plugin</li>
</ol>

<h3>Configure Java and Node.js in Global Tool Configuration</h3>
<p>Go to <strong>Manage Jenkins &rarr; Tools</strong> &rarr; install JDK(17) and NodeJs(16) &rarr; <strong>Apply and Save</strong>.</p>

<h3>SonarQube Setup</h3>
<ul>
  <li>Create the token: <strong>Jenkins Dashboard &rarr; Manage Jenkins &rarr; Credentials &rarr; Add Secret Text</strong>.</li>
  <li>After adding the sonar token, click <strong>Apply and Save</strong>.</li>
</ul>
<blockquote>
  <strong>Configure System</strong> is used to configure different servers.<br>
  <strong>Global Tool Configuration</strong> is used to configure tools installed via plugins.
</blockquote>
<p>Install a sonar scanner in the tools, then create a Jenkins webhook.</p>

<h3>3. Configure CI/CD Pipeline in Jenkins</h3>
<pre><code>pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/vardhan-na/DecSecOps.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}</code></pre>

<h3>Install Dependency-Check and Docker Tools in Jenkins</h3>

<p><strong>Install Dependency-Check Plugin:</strong></p>
<ul>
  <li>Go to <strong>Manage Jenkins &rarr; Manage Plugins &rarr; Available</strong>, search for "OWASP Dependency-Check."</li>
  <li>Check the box and click "Install without restart."</li>
</ul>

<p><strong>Configure Dependency-Check Tool:</strong></p>
<ul>
  <li>Go to <strong>Manage Jenkins &rarr; Global Tool Configuration</strong>.</li>
  <li>Find the "OWASP Dependency-Check" section, add a tool name (e.g., <code>DP-Check</code>), and save.</li>
</ul>

<p><strong>Install Docker Tools and Docker Plugins:</strong></p>
<ul>
  <li>Go to <strong>Manage Jenkins &rarr; Manage Plugins &rarr; Available</strong>, search for "Docker."</li>
  <li>Install: Docker, Docker Commons, Docker Pipeline, Docker API, docker-build-step.</li>
</ul>

<p><strong>Add DockerHub Credentials:</strong></p>
<ul>
  <li>Go to <strong>Manage Jenkins &rarr; Manage Credentials &rarr; System &rarr; Global credentials (unrestricted)</strong>.</li>
  <li>Click "Add Credentials" &rarr; choose "Secret text."</li>
  <li>Enter your DockerHub credentials and give them an ID (e.g., <code>docker</code>).</li>
  <li>Click "OK" to save.</li>
</ul>

<h3>Full Pipeline</h3>
<pre><code>pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/vardhan-na/DecSecOps.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . &gt; trivyfs.txt"
            }
        }
        stage("Docker Build &amp; Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build --build-arg TMDB_V3_API_KEY=&lt;yourapikey&gt; -t netflix ."
                        sh "docker tag netflix &lt;your-dockerhub-username&gt;/netflix:latest"
                        sh "docker push &lt;your-dockerhub-username&gt;/netflix:latest"
                    }
                }
            }
        }
        stage("TRIVY") {
            steps {
                sh "trivy image &lt;your-dockerhub-username&gt;/netflix:latest &gt; trivyimage.txt"
            }
        }
        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 &lt;your-dockerhub-username&gt;/netflix:latest'
            }
        }
    }
}</code></pre>

<blockquote>
  <strong>If you get a Docker login failed error:</strong>
  <pre><code>sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins</code></pre>
</blockquote>

<hr>

<h2>Phase 4: Monitoring</h2>

<h3>1. Install Prometheus and Grafana</h3>

<p><strong>Installing Prometheus:</strong></p>
<pre><code>sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/</code></pre>

<p>Create a systemd unit file:</p>
<pre><code>sudo nano /etc/systemd/system/prometheus.service</code></pre>

<pre><code>[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target</code></pre>

<pre><code>sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus</code></pre>

<p>Access Prometheus at: <code>http://&lt;your-server-ip&gt;:9090</code></p>

<p><strong>Installing Node Exporter:</strong></p>
<pre><code>sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*</code></pre>

<pre><code>sudo nano /etc/systemd/system/node_exporter.service</code></pre>

<pre><code>[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target</code></pre>

<pre><code>sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter</code></pre>

<h3>2. Configure Prometheus Plugin Integration</h3>
<p>Update <code>prometheus.yml</code>:</p>
<pre><code>global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['&lt;your-jenkins-ip&gt;:&lt;your-jenkins-port&gt;']</code></pre>

<pre><code>promtool check config /etc/prometheus/prometheus.yml
curl -X POST http://localhost:9090/-/reload</code></pre>

<p>View targets at: <code>http://&lt;your-prometheus-ip&gt;:9090/targets</code></p>

<h3>Grafana</h3>

<p><strong>Step 1: Install Dependencies</strong></p>
<pre><code>sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common</code></pre>

<p><strong>Step 2: Add the GPG Key</strong></p>
<pre><code>wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -</code></pre>

<p><strong>Step 3: Add Grafana Repository</strong></p>
<pre><code>echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list</code></pre>

<p><strong>Step 4: Update and Install Grafana</strong></p>
<pre><code>sudo apt-get update
sudo apt-get -y install grafana</code></pre>

<p><strong>Step 5: Enable and Start Grafana Service</strong></p>
<pre><code>sudo systemctl enable grafana-server
sudo systemctl start grafana-server</code></pre>

<p><strong>Step 6: Check Grafana Status</strong></p>
<pre><code>sudo systemctl status grafana-server</code></pre>

<p><strong>Step 7: Access Grafana Web Interface</strong></p>
<p>Open <code>http://&lt;your-server-ip&gt;:3000</code> in a browser. Default login: <code>admin</code> / <code>admin</code>.</p>

<p><strong>Step 8: Change the Default Password</strong></p>
<p>Follow the prompts on first login.</p>

<p><strong>Step 9: Add Prometheus Data Source</strong></p>
<ul>
  <li>Gear icon (&#9881;&#65039;) &rarr; Data Sources &rarr; Add data source &rarr; Prometheus.</li>
  <li>Set URL to <code>http://localhost:9090</code>.</li>
  <li>Click "Save &amp; Test."</li>
</ul>

<p><strong>Step 10: Import a Dashboard</strong></p>
<ul>
  <li>"+" icon &rarr; Dashboard &rarr; Import.</li>
  <li>Enter dashboard code <code>1860</code> &rarr; Load.</li>
  <li>Select the Prometheus data source &rarr; Import.</li>
</ul>

<hr>

<h2>Phase 5: Notification</h2>
<h3>1. Implement Notification Services</h3>
<ul>
  <li>Set up email notifications in Jenkins or another notification mechanism.</li>
</ul>

<hr>

<h2>Phase 6: Kubernetes</h2>

<h3>Create Kubernetes Cluster with Nodegroups</h3>
<p>Set up a Kubernetes cluster with node groups for a scalable environment to deploy and manage your applications.</p>

<h3>Monitor Kubernetes with Prometheus</h3>

<p><strong>Install Node Exporter using Helm:</strong></p>
<pre><code>helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace prometheus-node-exporter
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter</code></pre>

<p>Add a scrape job to <code>prometheus.yml</code>:</p>
<pre><code>  - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']</code></pre>

<p>Reload or restart Prometheus to apply these changes.</p>

<h3>Deploy Application with ArgoCD</h3>
<ol>
  <li><strong>Install ArgoCD</strong> — follow the <a href="https://archive.eksworkshop.com/intermediate/290_argocd/install/">EKS Workshop guide</a>.</li>
  <li><strong>Set your GitHub repository as a source</strong> — configure the connection to your repo and define it as the deployment source.</li>
  <li><strong>Create an ArgoCD Application</strong>, specifying:
    <ul>
      <li><code>name</code></li>
      <li><code>destination</code></li>
      <li><code>project</code></li>
      <li><code>source</code> (repo URL, revision, path)</li>
      <li><code>syncPolicy</code> (auto-sync, pruning, self-healing)</li>
    </ul>
  </li>
  <li><strong>Access your application</strong> — open port <code>30007</code> in your security group, then visit <code>NodeIP:30007</code>.</li>
</ol>

<hr>

<h2>Phase 7: Cleanup</h2>
<h3>1. Cleanup AWS EC2 Instances</h3>
<ul>
  <li>Terminate AWS EC2 instances that are no longer needed.</li>
</ul>

</body>
</html>