READ ME 

6)

# **SET–6: Software Engineering Lab Internal II**

Based on the actual SET–6 question paper

**GitHub repository:** [VehicleRentalManagementSystem](https://github.com/nikithamoturi/VehicleRentalManagementSystem.git?utm_source=chatgpt.com)

There are five experiments, each carrying 20 marks.

**Important differences in SET–6:**

- Docker uses a separate WordPress + MySQL vehicle rental information portal. It is independent of the GitHub project.
- Jenkins uses a **Freestyle Project** and must archive the generated WAR file.
- Kubernetes scales from **2 replicas to 4 replicas**.
- Nagios monitors the rental application host, HTTP service and MySQL.
- AWS asks for Ubuntu EC2 deployment and five scenario questions; it does not list detailed expected-output steps.

# **1. Docker – Multi-Container Application [20 Marks]**

**Aim:** Create a WordPress vehicle rental information portal connected to a MySQL database using Docker network and persistent volume.

## **Step 1: Pull official images**

Open PowerShell:

```
docker pull wordpress
docker pull mysql:8.0
```

Check:

```
docker images
```

## **Step 2: Create a dedicated Docker network**

```
docker network create rental-net
```

Verify:

```
docker network ls
```

This network allows WordPress to communicate with MySQL using the container name.

## **Step 3: Create a named volume**

```
docker volume create rental-mysql-data
```

Verify:

```
docker volume ls
```

The named volume stores MySQL database files persistently.

## **Step 4: Run the MySQL container**

```
docker run -d `
--name rental-db `
--network rental-net `
-e MYSQL_ROOT_PASSWORD=RentalRoot@123 `
-e MYSQL_DATABASE=rentaldb `
-e MYSQL_USER=rentaluser `
-e MYSQL_PASSWORD=RentalPass@123 `
-v rental-mysql-data:/var/lib/mysql `
mysql:8.0
```

Explanation:

- `rental-db`: MySQL container name.
- `rental-net`: dedicated Docker network.
- `MYSQL_DATABASE`: database name.
- `MYSQL_USER`: database username.
- `MYSQL_PASSWORD`: database password.
- `rental-mysql-data`: persistent volume.

Check:

```
docker ps
```

Check MySQL logs:

```
docker logs rental-db
```

Wait until MySQL initialization completes before starting WordPress.

## **Step 5: Run WordPress container**

```
docker run -d `
--name rental-wordpress `
--network rental-net `
-p 8080:80 `
-e WORDPRESS_DB_HOST=rental-db:3306 `
-e WORDPRESS_DB_USER=rentaluser `
-e WORDPRESS_DB_PASSWORD=RentalPass@123 `
-e WORDPRESS_DB_NAME=rentaldb `
wordpress
```

Explanation:

- `-p 8080:80`: host port 8080 maps to WordPress port 80.
- `WORDPRESS_DB_HOST`: MySQL container name and port.
- Database name, username and password must match MySQL settings.

## **Step 6: Verify both containers**

```
docker ps
```

Check WordPress logs:

```
docker logs rental-wordpress
```

Check MySQL logs:

```
docker logs rental-db
```

## **Step 7: Open the portal**

Open your browser:

```
http://localhost:8080
```

Complete WordPress installation:

1. Select language.
2. Enter site title: `Vehicle Rental Information Portal`.
3. Create administrator username.
4. Set administrator password.
5. Enter email.
6. Click Install WordPress.
7. Login to the dashboard.

## **Step 8: Add sample rental information**

Go to **Posts → Add New Post**.

Title:

```
Vehicle Rental Information
```

Content:

```
Welcome to Vehicle Rental Information Portal.

Vehicle Categories:

1. Cars
2. Bikes
3. SUVs

Rental Locations:

1. Hyderabad
2. Secunderabad
3. Warangal

Rental Policies:

1. Valid driving license is required.
2. Customers must provide identity proof.
3. Vehicles should be returned on time.

Contact Details:

Email: rental@gmail.com
Phone: 9876543210
```

Click **Publish**.

Open the website and verify that the information is displayed.

## **Step 9: Verify container states and logs**

```
docker ps
docker logs rental-wordpress
docker logs rental-db
```

## **Step 10: Test container restart**

```
docker restart rental-db
docker restart rental-wordpress
```

Check:

```
docker ps
```

Open:

```
http://localhost:8080
```

The portal should still display the published information.

## **Step 11: Demonstrate persistence after database container replacement**

Remove the MySQL container:

```
docker rm -f rental-db
```

Recreate it using the same named volume:

```
docker run -d `
--name rental-db `
--network rental-net `
-e MYSQL_ROOT_PASSWORD=RentalRoot@123 `
-e MYSQL_DATABASE=rentaldb `
-e MYSQL_USER=rentaluser `
-e MYSQL_PASSWORD=RentalPass@123 `
-v rental-mysql-data:/var/lib/mysql `
mysql:8.0
```

Restart WordPress:

```
docker restart rental-wordpress
```

Open:

```
http://localhost:8080
```

The previously published rental information should remain because the named volume was reused.

**Do not delete&#x20;**`rental-mysql-data`**.**

## **Important Docker commands**

| Requirement      | Command                                  |
| ---------------- | ---------------------------------------- |
| Pull image       | `docker pull wordpress`                  |
| Create network   | `docker network create rental-net`       |
| Create volume    | `docker volume create rental-mysql-data` |
| Run container    | `docker run -d ...`                      |
| Check containers | `docker ps`                              |
| View logs        | `docker logs rental-db`                  |
| Restart          | `docker restart rental-wordpress`        |
| Remove container | `docker rm -f rental-db`                 |
| View volumes     | `docker volume ls`                       |

## **Scenario-based questions and answers**

**1. What problem may occur if WordPress starts before MySQL is ready?**

WordPress may fail to establish a database connection and show a database connection error. Start WordPress after MySQL initialization completes, or configure a retry/health-check mechanism.

**2. How can WordPress connect to MySQL using its container name?**

Connect both containers to the same custom Docker network and specify:

```
rental-db:3306
```

Docker's network DNS resolves the container name.

**3. Why must database credentials match?**

WordPress uses the configured database name, username and password to authenticate with MySQL. If they do not match, the connection fails.

**4. What if host port 8080 is already being used?**

Choose another unused host port, such as 8081:

```
docker run -d --name rental-wordpress --network rental-net -p 8081:80 ...
```

Then open:

```
http://localhost:8081
```

**5. How do named volumes retain information after container replacement?**

Named volumes store database files outside the container's writable layer. Recreating the container with the same volume makes the previous database data available again.

# **2. Jenkins – Web Application Build and Continuous Integration [20 Marks]**

**Aim:** Create a Jenkins Freestyle Project to fetch, compile, test and package the Vehicle Rental Management System, then archive the generated WAR file.

Repository:

```
https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
```

## **Step 1: Check Java, Maven and Git**

Open PowerShell:

```
java -version
mvn -version
git --version
```

Install any missing tool and configure its PATH.

## **Step 2: Install Jenkins plugins**

Open Jenkins:

```
http://localhost:8080
```

Go to:

**Manage Jenkins → Plugins → Available plugins**

Install:

1. Git plugin
2. Maven Integration plugin
3. JUnit plugin

Restart Jenkins if required.

## **Step 3: Configure JDK and Maven**

Go to:

**Manage Jenkins → Tools**

Configure:

- JDK installation.
- Maven installation.

Save the configuration.

## **Step 4: Create Freestyle Project**

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter:

```
VehicleRental-CI
```

4. Select **Freestyle Project**.
5. Click OK.

## **Step 5: Configure GitHub repository**

Under **Source Code Management**:

1. Select **Git**.
2. Enter:

```
https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
```

3\. Under Branch Specifier, enter:

```
*/main
```

If the repository uses another default branch, select that actual branch.

## **Step 6: Configure build trigger**

Under **Build Triggers**, select **Poll SCM**.

Enter:

```
H/5 * * * *
```

Jenkins checks for source code changes approximately every five minutes.

Alternatively, use a GitHub webhook if the Jenkins server is reachable by GitHub.

## **Step 7: Configure Maven build goals**

Under **Build Steps**:

1. Click **Add build step**.
2. Select **Invoke top-level Maven targets**.
3. Select the configured Maven installation.
4. Enter:

```
clean test package
```

Meaning:

- `clean` – removes previous build output.
- `test` – executes available tests.
- `package` – generates the configured application package.

## **Step 8: Configure WAR archiving**

Under **Post-build Actions**:

1. Click **Add post-build action**.
2. Select **Archive the artifacts**.
3. Enter:

```
target/*.war
```

Save the job.

This assumes the project actually produces a WAR file. If it does not, check `pom.xml` and the build output rather than changing the artifact name by guesswork.

## **Step 9: Run the Jenkins job**

1. Open `VehicleRental-CI`.
2. Click **Build Now**.
3. Click the build number.
4. Select **Console Output**.

Expected successful output:

```
BUILD SUCCESS
Finished: SUCCESS
```

## **Step 10: Review test results**

Open the build page and check the test results if Maven generated JUnit XML reports.

## **Step 11: Demonstrate a failed build**

For a controlled demonstration:

1. Create a temporary feature branch.
2. Introduce a failing unit test.
3. Push the change.
4. Temporarily configure the Jenkins branch specifier to that feature branch.
5. Run the job.
6. Open Console Output.

Expected:

```
BUILD FAILURE
```

Jenkins should mark the build as failed.

## **Step 12: Archive and verify WAR file**

Run a successful build again.

Open the build page and check **Build Artifacts**.

Expected artifact pattern:

```
target/*.war
```

The generated WAR can be downloaded from the build page.

## **Scenario-based questions and answers**

**1. How does Jenkins help detect errors introduced while changing the vehicle booking process?**

Jenkins automatically builds the project and executes available tests. Compilation or test failures are displayed in the build output.

**2. What should the team check when Maven cannot resolve a dependency?**

Check:

- Dependency name and version in `pom.xml`.
- Internet connectivity.
- Maven repository configuration.
- Repository availability.
- Maven proxy settings, if used.

**3. Why should automated tests be included?**

They help identify errors in booking, vehicle availability and rental workflows before the application is released.

**4. How can the Freestyle Project build a feature branch instead of main?**

Change the branch specifier from:

```
*/main
```

to:

```
*/feature-branch
```

Use the actual feature branch name.

**5. Why archive the WAR file for each successful build?**

It preserves the build artifact for later deployment, testing, rollback and comparison with earlier versions.

# **3. Kubernetes – Container Deployment and Auto Scaling [20 Marks]**

**Aim:** Deploy the Vehicle Rental Management application on Minikube with two replicas, expose it through a Service, scale it to four replicas and demonstrate automatic pod replacement.

## **Step 1: Start Minikube**

Open PowerShell:

```
minikube start
```

Check:

```
minikube status
kubectl get nodes
```

## **Step 2: Clone the repository**

```
git clone https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
```

Enter the project:

```
cd VehicleRentalManagementSystem
```

Check files:

```
dir
```

## **Step 3: Build the application**

If it is a Maven project:

```
mvn clean package
```

Find the generated application artifact:

```
Get-ChildItem -Recurse -Include *.war,*.jar
```

## **Step 4: Build the Docker image**

If the application generates a Tomcat-compatible WAR, create a Dockerfile in the project root.

Template:

```

FROM tomcat:10.1-jdk17

COPY target/VehicleRentalManagementSystem.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

```

Replace the WAR filename with the actual generated file. Use a Tomcat version compatible with the application's Java and web framework.

Configure Minikube's Docker environment:

```
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

Build:

```
docker build -t vehicle-rental:v1 .
```

Verify:

```
docker images
```

## **Step 5: Create Deployment YAML**

Create a file named:

```
deployment.yaml
```

Paste:

```

apiVersion: apps/v1
kind: Deployment

metadata:
  name: rental-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: rental

  template:
    metadata:
      labels:
        app: rental

    spec:
      containers:
        - name: rental-container
          image: vehicle-rental:v1
          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8080

```

This creates two application replicas.

## **Step 6: Create Service YAML**

Create:

```
service.yaml
```

Paste:

```

apiVersion: v1
kind: Service

metadata:
  name: rental-service

spec:
  type: NodePort

  selector:
    app: rental

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080

```

## **Step 7: Deploy the application**

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Check:

```
kubectl get deployments
kubectl get pods
kubectl get services
```

Expected:

```
rental-deployment   2/2
```

Two pods should be running.

## **Step 8: Access the application**

Run:

```
minikube service rental-service
```

This opens the service in your browser.

Alternatively:

```
minikube ip
```

Open:

```
http://MINIKUBE-IP:30080
```

## **Step 9: Scale from two to four replicas**

The question specifically asks for two to four replicas.

Run:

```
kubectl scale deployment rental-deployment --replicas=4
```

Verify:

```
kubectl get deployments
kubectl get pods
```

Expected:

```
rental-deployment   4/4
```

## **Step 10: Delete one running pod**

List pods:

```
kubectl get pods
```

Copy one pod name and run:

```
kubectl delete pod POD-NAME
```

Check again:

```
kubectl get pods
```

Kubernetes automatically creates a replacement pod to maintain four replicas.

## **Step 11: Check updated pod status**

```
kubectl get deployment rental-deployment
kubectl get pods
```

Expected:

```
READY   UP-TO-DATE   AVAILABLE
4/4     4            4
```

## **Step 12: Clean up**

```
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

Stop Minikube:

```
minikube stop
```

## **Important Kubernetes commands**

| Requirement      | Command                                                   |
| ---------------- | --------------------------------------------------------- |
| Start Minikube   | `minikube start`                                          |
| Check nodes      | `kubectl get nodes`                                       |
| Apply Deployment | `kubectl apply -f deployment.yaml`                        |
| Apply Service    | `kubectl apply -f service.yaml`                           |
| View pods        | `kubectl get pods`                                        |
| Scale to four    | `kubectl scale deployment rental-deployment --replicas=4` |
| Delete pod       | `kubectl delete pod POD-NAME`                             |
| Check Deployment | `kubectl get deployment`                                  |
| View services    | `kubectl get services`                                    |

## **Scenario-based questions and answers**

**1. How does Kubernetes keep the rental portal available when a pod terminates?**

The Deployment controller detects that a pod is missing and creates a replacement to maintain the desired replica count.

**2. What is the purpose of specifying replicas in a Deployment?**

It defines how many copies of the application should run. Kubernetes attempts to maintain that number.

**3. Why scale the rental portal during holidays?**

More customers search for vehicles and submit rental requests. Increasing replicas helps handle the higher workload.

**4. How does a Service provide stable access when pod IP addresses change?**

A Service provides a stable network endpoint and forwards traffic to matching available pods.

**5. What can be done if a new application image introduces an error?**

Use a Deployment rollback:

```
kubectl rollout undo deployment/rental-deployment
```

The previous revision must exist in the Deployment's rollout history.

# **4. Nagios – Infrastructure and Application Monitoring [20 Marks]**

**Aim:** Monitor the Vehicle Rental Management application server, CPU, memory, disk, uptime, HTTP availability and MySQL connectivity using Nagios.

## **Step 1: Pull Nagios image**

```
docker pull jasonrivers/nagios:latest
```

## **Step 2: Run Nagios container**

```
docker run -d `
--name nagios `
-p 8081:80 `
jasonrivers/nagios:latest
```

Check:

```
docker ps
```

## **Step 3: Open Nagios dashboard**

Open:

```
http://localhost:8081
```

Common default login for this image:

```
Username: nagiosadmin
Password: nagios
```

## **Step 4: Enter Nagios container**

```
docker exec -it nagios bash
```

Open configuration directory:

```
cd /opt/nagios/etc/objects
```

## **Step 5: Create vehicle rental configuration**

Create:

```
vi rental.cfg
```

Add the host definition:

```

define host {
    use                     linux-server
    host_name               rental-server
    alias                   Vehicle Rental Application Server
    address                 192.168.1.100
    max_check_attempts      3
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}

```

Replace `192.168.1.100` with the actual application server IP.

## **Step 6: Configure HTTP monitoring**

Add:

```

define service {
    use                     generic-service
    host_name               rental-server
    service_description     Rental Portal HTTP
    check_command           check_http
}

```

This checks whether the rental web application responds to HTTP requests.

## **Step 7: Configure MySQL monitoring**

Add:

```

define service {
    use                     generic-service
    host_name               rental-server
    service_description     MySQL Database
    check_command           check_tcp!3306
}

```

This checks whether the MySQL TCP port is accessible. It does not verify successful SQL queries.

## **Step 8: Configure CPU and disk monitoring**

For a remote Ubuntu application server, install NRPE and Nagios plugins:

```
sudo apt update
sudo apt install nagios-nrpe-server nagios-plugins -y
```

Check:

```
sudo systemctl status nagios-nrpe-server
```

Configure NRPE to allow the Nagios server to connect and define the required monitoring commands.

Add the following service definitions to `rental.cfg`:

```

define service {
    use                     generic-service
    host_name               rental-server
    service_description     CPU Load
    check_command           check_nrpe!check_load
}

define service {
    use                     generic-service
    host_name               rental-server
    service_description     Disk Utilization
    check_command           check_nrpe!check_disk
}

```

The NRPE agent must have the `check_load` and `check_disk` commands defined.

For memory and uptime monitoring, configure suitable additional plugins and commands on the remote server and add corresponding service definitions.

## **Step 9: Add configuration file to Nagios**

Open:

```
vi /opt/nagios/etc/nagios.cfg
```

Add:

```
cfg_file=/opt/nagios/etc/objects/rental.cfg
```

Save the file.

## **Step 10: Validate configuration**

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

If no errors are reported, the configuration is valid.

## **Step 11: Restart Nagios**

Exit the container:

```
exit
```

Restart:

```
docker restart nagios
```

Open:

```
http://localhost:8081
```

Check the **Hosts** and **Services** sections.

## **Step 12: Set WARNING and CRITICAL thresholds**

Example thresholds:

| Resource         | WARNING          | CRITICAL           |
| ---------------- | ---------------- | ------------------ |
| CPU usage        | Above 70%        | Above 90%          |
| Memory usage     | Above 75%        | Above 90%          |
| Disk utilization | Above 80%        | Above 95%          |
| HTTP             | Slow response    | Portal unavailable |
| MySQL            | Delayed response | Connection failure |

These are example values. Configure the actual plugin arguments to enforce the thresholds. A service definition alone does not apply them.

## **Scenario-based questions and answers**

**1. How can Nagios detect a resource problem before the rental portal becomes unavailable?**

Nagios periodically monitors CPU, memory, disk and server performance. When a configured threshold is crossed, it generates WARNING or CRITICAL alerts.

**2. When is a service reported as WARNING instead of CRITICAL?**

WARNING indicates a concerning condition that has crossed the warning threshold. CRITICAL indicates a more serious problem or failure.

**3. How can an HTTP check distinguish application failure from complete server failure?**

Nagios checks the host and HTTP service separately. The host may be UP while the HTTP service is CRITICAL.

**4. What monitoring information becomes unavailable if Nagios itself stops?**

Its monitoring dashboard, checks and alert generation stop. A separate monitoring system can be used to monitor Nagios availability.

**5. How can you determine whether a failed vehicle availability request is caused by Tomcat, MySQL or the network?**

- Check Tomcat and HTTP service status.
- Check MySQL service and port 3306.
- Check server network connectivity.
- Compare host status with application and database service status.

# **5. AWS – Cloud Deployment using EC2 [20 Marks]**

**Aim:** Launch an Ubuntu EC2 instance and deploy the Vehicle Rental Management System so customers can access the application remotely.

Repository:

```
https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
```

The SET–6 question paper specifies the EC2 deployment task and scenario questions, but does not provide a separate expected-output checklist for AWS. The following are the practical deployment steps.

## **Step 1: Login to AWS**

1. Open [AWS Console](https://console.aws.amazon.com/?utm_source=chatgpt.com).
2. Search for **EC2**.
3. Click **Launch Instance**.

## **Step 2: Configure EC2 instance**

Enter:

| Setting       | Value                               |
| ------------- | ----------------------------------- |
| Name          | VehicleRental-Server                |
| AMI           | Ubuntu Server                       |
| Instance type | t2.micro or suitable available type |
| Key pair      | Create a new key pair               |
| Storage       | Default                             |

Click **Launch Instance**.

## **Step 3: Configure Security Group**

Add inbound rules:

| Type       | Port | Source    |
| ---------- | ---- | --------- |
| SSH        | 22   | My IP     |
| HTTP       | 80   | 0.0.0.0/0 |
| Custom TCP | 8080 | 0.0.0.0/0 |

Port 8080 is required for direct Tomcat access. For a more secure deployment, use HTTPS and restrict direct access to the application server.

Click **Launch Instance**.

## **Step 4: Connect to EC2**

Select the instance and click:

**Connect → SSH client**

Open PowerShell in the folder containing your `.pem` file.

```
ssh -i "rental-key.pem" ubuntu@YOUR-PUBLIC-IP
```

Replace `YOUR-PUBLIC-IP` with your EC2 public IPv4 address.

## **Step 5: Update Ubuntu**

```
sudo apt update
sudo apt upgrade -y
```

## **Step 6: Install Java**

```
sudo apt install openjdk-17-jdk -y
```

Verify:

```
java -version
```

## **Step 7: Install Maven**

```
sudo apt install maven -y
```

Verify:

```
mvn -version
```

## **Step 8: Install Git**

```
sudo apt install git -y
```

Verify:

```
git --version
```

## **Step 9: Install Tomcat**

For Ubuntu packages:

```
sudo apt install tomcat10 -y
```

Start Tomcat:

```
sudo systemctl start tomcat10
```

Enable automatic startup:

```
sudo systemctl enable tomcat10
```

Check:

```
sudo systemctl status tomcat10
```

## **Step 10: Clone the GitHub repository**

```
git clone https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
```

Enter the project:

```
cd VehicleRentalManagementSystem
```

Check:

```
ls
```

## **Step 11: Build the application**

If the project is Maven-based:

```
mvn clean package
```

Find the generated artifact:

```
find . -type f \( -name "*.war" -o -name "*.jar" \)
```

## **Step 12: Deploy the WAR file**

If the project produces a Tomcat-compatible WAR named `VehicleRentalManagementSystem.war`:

```
sudo cp target/VehicleRentalManagementSystem.war /var/lib/tomcat10/webapps/
```

Use the actual generated filename and location.

Restart Tomcat:

```
sudo systemctl restart tomcat10
```

Check:

```
sudo systemctl status tomcat10
```

## **Step 13: Access the application**

Open:

```
http://YOUR-PUBLIC-IP:8080/VehicleRentalManagementSystem
```

Replace the context path with the actual WAR filename.

If the WAR is named `ROOT.war`, use:

```
http://YOUR-PUBLIC-IP:8080/
```

## **Step 14: Verify deployment**

Check Tomcat:

```
sudo systemctl status tomcat10
```

Check logs:

```
sudo journalctl -u tomcat10
```

Check port:

```
sudo ss -tulnp | grep 8080
```

If the page does not open, check the Security Group, Tomcat service, deployment logs and database configuration.

## **Scenario-based questions and answers**

**1. Which EC2 or Security Group configuration issues could prevent customers from opening the rental portal?**

- Incorrect public IP.
- Port 8080 or HTTP port 80 is blocked.
- Tomcat is stopped.
- Incorrect application URL.
- Application is not deployed.
- Firewall blocks incoming traffic.

**2. Why should SSH access be restricted to trusted administrative addresses?**

It reduces unauthorized access and brute-force attacks. Only authorized administrators should connect through SSH.

**3. What would customers observe if Tomcat stops while EC2 continues running?**

The EC2 instance remains active, but the rental portal becomes unavailable because Tomcat serves the web application.

**4. How can the company handle increased customer requests during holidays?**

Use multiple EC2 instances, an Application Load Balancer, Auto Scaling and health checks.

**5. Why use multiple EC2 instances behind a load balancer?**

The load balancer distributes customer requests among multiple instances. If one instance fails, requests can be routed to healthy instances, improving availability and scalability.

# **SET–6: Last-minute revision**

| Experiment | Exact important points                                                             |
| ---------- | ---------------------------------------------------------------------------------- |
| Docker     | WordPress + MySQL, `rental-net`, `rental-mysql-data`, port 8080                    |
| Jenkins    | Freestyle Project, GitHub URL, branch, Poll SCM, `clean test package`, archive WAR |
| Kubernetes | Two replicas → four replicas, Service, delete pod, automatic replacement           |
| Nagios     | Host, HTTP, MySQL, CPU, memory, disk, uptime, thresholds                           |
| AWS        | Ubuntu EC2, SSH, Java, Maven, Git, Tomcat, WAR deployment                          |

## **Commands you must remember**

**Docker**

```
docker network create rental-net
docker volume create rental-mysql-data
docker ps
docker logs rental-db
docker restart rental-wordpress
```

**Jenkins Maven goals**

```
clean test package
```

**Kubernetes**

```
minikube start
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl scale deployment rental-deployment --replicas=4
kubectl delete pod POD-NAME
```

**AWS**

```
sudo apt update
sudo apt install openjdk-17-jdk maven git -y
git clone https://github.com/nikithamoturi/VehicleRentalManagementSystem.git
mvn clean package
sudo systemctl restart tomcat10
```

**Most important number to remember in SET–6:** Kubernetes starts with **2 replicas and scales to 4 replicas**. Docker Experiment 1 is the separate WordPress vehicle rental information portal, not the GitHub Vehicle Rental Management System.





7)
# **SET–7: Software Engineering Lab Internal II**

Actual SET–7 question paper checked

**GitHub repository:** [SportSystem](https://github.com/sarasrija/SportSystem.git?utm_source=chatgpt.com)

SET–7 has 4 experiments, each carrying 25 marks. It is different from SET–1 to SET–6.

| Experiment | Topic                                                           | Marks |
| ---------- | --------------------------------------------------------------- | ----- |
| 1          | Jenkins – Scripted Pipeline & Build Triggers                    | 25    |
| 2          | AWS – Cloud Deployment using EC2                                | 25    |
| 3          | Kubernetes & Nagios – Minikube Pods & Infrastructure Monitoring | 25    |
| 4          | Docker Compose – Multi-Container Orchestration                  | 25    |

# **1. Jenkins – Scripted Pipeline & Build Triggers [25 Marks]**

**Aim:** Create a Jenkins scripted pipeline to clone the SportSystem repository, clean, compile, package the Maven project and generate a WAR artifact.

Repository:

```
https://github.com/sarasrija/SportSystem.git
```

## **Step 1: Verify Java, Maven and Git**

Open PowerShell:

```
java -version
mvn -version
git --version
```

Install the missing tools if any command is not recognized.

## **Step 2: Install Jenkins plugins**

Open:

```
http://localhost:8080
```

Go to:

**Manage Jenkins → Plugins → Available plugins**

Install:

1. Git plugin
2. Pipeline plugin
3. GitHub plugin

Restart Jenkins if required.

## **Step 3: Configure JDK and Maven**

Go to:

**Manage Jenkins → Tools**

Configure:

- JDK installation.
- Maven installation.

For example:

```
JDK Name: JDK17
Maven Name: Maven3
```

Save the configuration.

The names in the pipeline must exactly match the names configured here.

## **Step 4: Create a Jenkins Pipeline project**

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter:

```
SportSystem-Pipeline
```

4. Select **Pipeline**.
5. Click OK.

## **Step 5: Create Jenkinsfile**

In the SportSystem repository's root folder, create a file named:

```
Jenkinsfile
```

The question paper asks for a scripted pipeline. In Jenkins, `agent` and `tools` are Declarative Pipeline directives, whereas Scripted Pipeline uses `node` and the `tool()` step. The following is a valid Scripted Pipeline implementation that configures the tools and performs all the required stages.

```

node {
    stage('Clean Workspace') {
        cleanWs()
    }

    stage('Configure Tools') {
        def jdkHome = tool 'JDK17'
        def mavenHome = tool 'Maven3'

        env.JAVA_HOME = jdkHome
        env.PATH = "${jdkHome}/bin:${mavenHome}/bin:${env.PATH}"

        echo 'JDK and Maven configured'
    }

    stage('Clone Repository') {
        git branch: 'main',
            url: 'https://github.com/sarasrija/SportSystem.git'
    }

    stage('Clean') {
        sh 'mvn clean'
    }

    stage('Compile') {
        sh 'mvn compile'
    }

    stage('Package') {
        sh 'mvn package'
    }

    stage('Verify WAR') {
        sh 'ls -l target/*.war'
    }
}

```

**Important:**

- Configure the tool names as `JDK17` and `Maven3`, or change the names in the script to match your Jenkins configuration.
- `cleanWs()` requires the Workspace Cleanup plugin. Install it if you use this command.
- This script uses `sh` for a Linux Jenkins agent. For a Windows agent, replace shell commands with suitable `bat` commands.
- If the repository's default branch is not `main`, change the branch name.
- The project must be configured to produce a WAR for the final verification stage to succeed.

## **Step 6: Configure Pipeline in Jenkins**

1. Open the `SportSystem-Pipeline` job.
2. Scroll to **Pipeline**.
3. Select **Pipeline script**.
4. Paste the Jenkinsfile code.
5. Click Save.

If you keep the Jenkinsfile in GitHub, select **Pipeline script from SCM**, select Git, and provide the repository URL and script path `Jenkinsfile`.

## **Step 7: Configure automated build triggers**

Go to:

**Job → Configure → Build Triggers**

For GitHub changes, use either:

**Option A: Poll SCM**

Select Poll SCM and enter:

```
H/5 * * * *
```

Jenkins checks the repository approximately every five minutes.

**Option B: GitHub webhook**

1. Open the GitHub repository.
2. Select **Settings → Webhooks**.
3. Click **Add webhook**.
4. Enter:

```
http://YOUR-JENKINS-PUBLIC-ADDRESS/github-webhook/
```

5. Select content type `application/json`.
6. Enable push events.
7. Save.

The Jenkins server must be reachable by GitHub. A `localhost` URL cannot receive public GitHub webhooks.

## **Step 8: Run the pipeline**

1. Open `SportSystem-Pipeline`.
2. Click **Build Now**.
3. Open the build number.
4. Click **Console Output**.

Expected stages:

```
Clean Workspace
Configure Tools
Clone Repository
Clean
Compile
Package
Verify WAR
```

Expected successful output:

```
BUILD SUCCESS
Finished: SUCCESS
```

## **Step 9: Verify generated WAR**

In the Jenkins console, check:

```
target/*.war
```

The WAR file should be generated inside the project's `target` directory and verified by the final stage.

## **Scenario-based questions and answers**

**1. Your manager asks you to clean the workspace before building. Which stage or command handles it?**

The Clean Workspace stage uses:

```
cleanWs()
```

The Maven command:

```
mvn clean
```

removes the previous Maven build output, especially the `target` directory.

**2. If Maven is not configured globally in Jenkins, which section fails first?**

The Configure Tools stage fails at:

```
def mavenHome = tool 'Maven3'
```

If Jenkins cannot find the configured Maven installation, the pipeline cannot obtain its Maven path.

**3. How can the pipeline trigger only when GitHub changes occur instead of nightly?**

Configure a GitHub webhook in the GitHub repository and enable the GitHub webhook trigger in Jenkins. Alternatively, use Poll SCM to periodically detect changes.

**4. What happens if you replace&#x20;**`mvn clean`**&#x20;with&#x20;**`mvn compile`**?**

- `mvn clean` deletes previous build output.
- `mvn compile` compiles the source code but does not remove old build output.

Using only `mvn compile` may leave old generated files in the project.

**5. What is the difference between polling and webhook triggers?**

| Polling                             | Webhook                                       |
| ----------------------------------- | --------------------------------------------- |
| Jenkins checks GitHub periodically. | GitHub notifies Jenkins when a change occurs. |
| May introduce a delay.              | Usually triggers quickly.                     |
| Generates periodic checks.          | Requires Jenkins to be reachable from GitHub. |

# **2. AWS – Cloud Deployment using EC2 [25 Marks]**

**Aim:** Deploy the Maven-based SportSystem web application on an Ubuntu EC2 instance using Java, Git, Docker and Tomcat.

Repository:

```
https://github.com/sarasrija/SportSystem.git
```

## **Step 1: Launch an EC2 instance**

1. Open [AWS Console](https://console.aws.amazon.com/?utm_source=chatgpt.com).
2. Search for **EC2**.
3. Click **Launch Instance**.
4. Enter:

| Setting       | Value                               |
| ------------- | ----------------------------------- |
| Name          | SportSystem-Server                  |
| AMI           | Ubuntu Server                       |
| Instance type | t2.micro or suitable available type |
| Key pair      | Create a new key pair               |
| Storage       | Default                             |

5\. Click Launch Instance.

## **Step 2: Configure Security Group**

Add inbound rules:

| Type       | Port | Source    |
| ---------- | ---- | --------- |
| SSH        | 22   | My IP     |
| HTTP       | 80   | 0.0.0.0/0 |
| Custom TCP | 8080 | 0.0.0.0/0 |

Port 8080 is needed for direct Tomcat access.

Click Launch Instance.

## **Step 3: Connect using SSH**

Select the EC2 instance.

Click **Connect → SSH client**.

Open PowerShell in the folder containing your `.pem` key.

```
ssh -i "sports-key.pem" ubuntu@YOUR-PUBLIC-IP
```

Replace `YOUR-PUBLIC-IP` with the EC2 public IPv4 address.

## **Step 4: Update Ubuntu**

```
sudo apt update
sudo apt upgrade -y
```

## **Step 5: Install Java**

```
sudo apt install openjdk-17-jdk -y
```

Verify:

```
java -version
```

## **Step 6: Install Git**

```
sudo apt install git -y
```

Verify:

```
git --version
```

## **Step 7: Install Docker**

The question specifically asks to install Docker.

```
sudo apt install docker.io -y
```

Start Docker:

```
sudo systemctl start docker
```

Enable Docker at startup:

```
sudo systemctl enable docker
```

Check:

```
sudo systemctl status docker
```

## **Step 8: Install Maven**

The question requires building a Maven project, so install Maven:

```
sudo apt install maven -y
```

Verify:

```
mvn -version
```

## **Step 9: Install Tomcat**

For Ubuntu packages:

```
sudo apt install tomcat10 -y
```

Start Tomcat:

```
sudo systemctl start tomcat10
```

Enable automatic startup:

```
sudo systemctl enable tomcat10
```

Verify:

```
sudo systemctl status tomcat10
```

## **Step 10: Clone the SportSystem repository**

```
git clone https://github.com/sarasrija/SportSystem.git
```

Enter the project:

```
cd SportSystem
```

Check files:

```
ls
```

## **Step 11: Build the Maven project**

```
mvn clean package
```

Check the generated WAR:

```
ls target/
```

If you want to search specifically:

```
find target -name "*.war"
```

The project must be configured to produce a WAR file.

## **Step 12: Copy WAR into Tomcat webapps**

Suppose the generated file is `SportSystem.war`.

Run:

```
sudo cp target/SportSystem.war /var/lib/tomcat10/webapps/
```

Replace the filename with the actual WAR filename.

## **Step 13: Restart Tomcat**

```
sudo systemctl restart tomcat10
```

Check:

```
sudo systemctl status tomcat10
```

## **Step 14: Access the application**

Open the browser:

```
http://YOUR-PUBLIC-IP:8080/SportSystem
```

Replace `SportSystem` with the actual WAR filename without `.war`.

If the file is named `ROOT.war`, access:

```
http://YOUR-PUBLIC-IP:8080/
```

Expected output

The SportSystem application should open using the EC2 public IP and its Tomcat application path.

## **Step 15: Ensure services start after reboot**

Enable Tomcat:

```
sudo systemctl enable tomcat10
```

Enable Docker:

```
sudo systemctl enable docker
```

Check:

```
sudo systemctl is-enabled tomcat10
sudo systemctl is-enabled docker
```

Both should return:

```
enabled
```

## **Important AWS commands**

| Requirement      | Command                                                     |
| ---------------- | ----------------------------------------------------------- |
| Update packages  | `sudo apt update`                                           |
| Install Java     | `sudo apt install openjdk-17-jdk -y`                        |
| Install Git      | `sudo apt install git -y`                                   |
| Install Docker   | `sudo apt install docker.io -y`                             |
| Install Maven    | `sudo apt install maven -y`                                 |
| Clone repository | `git clone https://github.com/sarasrija/SportSystem.git`    |
| Build project    | `mvn clean package`                                         |
| Copy WAR         | `sudo cp target/SportSystem.war /var/lib/tomcat10/webapps/` |
| Start Tomcat     | `sudo systemctl start tomcat10`                             |
| Enable Tomcat    | `sudo systemctl enable tomcat10`                            |

## **Scenario-based questions and answers**

**1. Where should the generated WAR file be placed for Tomcat execution, and why?**

Place it inside:

```
/var/lib/tomcat10/webapps/
```

Tomcat automatically detects WAR files in its `webapps` directory and deploys them as web applications.

**2. Tomcat is running, but the browser displays 404. What should you check?**

Check:

- Whether the WAR file was copied to `webapps`.
- Whether the WAR deployment completed successfully.
- Whether the context path matches the WAR filename.
- Whether the application has startup errors.
- Whether the WAR is compatible with the Tomcat version.

Check logs:

```
sudo journalctl -u tomcat10
```

**3. The EC2 public IP changes after restarting the instance. Which AWS feature avoids this issue?**

Use an **Elastic IP address**. It provides a static public IPv4 address that can be associated with the EC2 instance.

**4. EC2 has no internet access, but Maven needs external dependencies. What can you use?**

A suitable solution is to provide secure outbound connectivity through a NAT Gateway for a private-subnet instance. A VPC endpoint can provide private access to supported AWS services, and an internal Maven artifact repository can cache dependencies.

**5. What command ensures that a web server service starts automatically after EC2 boots?**

```
sudo systemctl enable tomcat10
```

# **3. Kubernetes & Nagios – Minikube Pods & Infrastructure Monitoring [25 Marks]**

**Aim:** Deploy the SportSystem application using Minikube, scale replicas, expose the application through a Kubernetes Service, and configure Nagios in Docker to monitor server performance and application availability.

This experiment has two parts: Kubernetes and Nagios.

## **Part A: Kubernetes**

### **Step 1: Check Docker**

Open PowerShell:

```
docker --version
```

Make sure Docker Desktop is running.

### **Step 2: Start Minikube using Docker driver**

```
minikube start --driver=docker
```

Check:

```
minikube status
```

Check Kubernetes nodes:

```
kubectl get nodes
```

Expected: The Minikube node should show `Ready`.

### **Step 3: Clone SportSystem repository**

```
git clone https://github.com/sarasrija/SportSystem.git
```

Enter the folder:

```
cd SportSystem
```

Build the application:

```
mvn clean package
```

### **Step 4: Create Dockerfile**

If the project generates a Tomcat-compatible WAR, create a `Dockerfile` in the project root.

```

FROM tomcat:10.1-jdk17

COPY target/SportSystem.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

```

Replace `SportSystem.war` with the actual WAR filename. Ensure that the Tomcat version is compatible with the project.

### **Step 5: Build Docker image inside Minikube**

Configure the Docker environment:

```
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

Build the image:

```
docker build -t sports-app:v1 .
```

Verify:

```
docker images
```

### **Step 6: Create Deployment YAML**

Create a file:

```
deployment.yaml
```

Paste:

```

apiVersion: apps/v1
kind: Deployment

metadata:
  name: sports-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: sports

  template:
    metadata:
      labels:
        app: sports

    spec:
      containers:
        - name: sports-container
          image: sports-app:v1
          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8080

```

This creates two SportSystem pods.

### **Step 7: Create Service YAML**

Create:

```
service.yaml
```

Paste:

```

apiVersion: v1
kind: Service

metadata:
  name: sports-service

spec:
  type: NodePort

  selector:
    app: sports

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080

```

### **Step 8: Deploy the application**

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Check:

```
kubectl get deployments
kubectl get pods
kubectl get services
```

Expected:

```
sports-deployment   2/2
```

Two pods should be running.

### **Step 9: Scale replicas**

Scale to five replicas:

```
kubectl scale deployment sports-deployment --replicas=5
```

Verify:

```
kubectl get pods
kubectl get deployments
```

Expected:

```
sports-deployment   5/5
```

### **Step 10: Expose and access application**

Run:

```
minikube service sports-service
```

Alternatively:

```
minikube ip
```

Open:

```
http://MINIKUBE-IP:30080
```

### **Step 11: Demonstrate pod recovery**

List pods:

```
kubectl get pods
```

Delete one pod:

```
kubectl delete pod POD-NAME
```

Replace `POD-NAME` with an actual pod name.

Check again:

```
kubectl get pods
```

Kubernetes automatically creates a replacement pod because the Deployment maintains five replicas.

### **Step 12: Clean up**

```
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

Stop Minikube:

```
minikube stop
```

## **Part B: Nagios Monitoring**

**Aim:** Install Nagios using Docker and monitor server performance, CPU usage and application availability.

### **Step 1: Pull Nagios image**

Open another PowerShell window:

```
docker pull jasonrivers/nagios:latest
```

### **Step 2: Run Nagios container**

```
docker run -d `
--name nagios `
-p 8081:80 `
jasonrivers/nagios:latest
```

Check:

```
docker ps
```

### **Step 3: Open Nagios dashboard**

Open:

```
http://localhost:8081
```

Common default credentials for this image:

```
Username: nagiosadmin
Password: nagios
```

### **Step 4: Enter Nagios container**

```
docker exec -it nagios bash
```

Open configuration folder:

```
cd /opt/nagios/etc/objects
```

### **Step 5: Create sports host configuration**

Create:

```
vi sports.cfg
```

Add:

```

define host {
    use                     linux-server
    host_name               sports-server
    alias                   Sports Management Server
    address                 192.168.1.100
    max_check_attempts      3
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}

```

Replace the IP address with the actual server IP you want Nagios to monitor.

### **Step 6: Configure HTTP monitoring**

Add:

```

define service {
    use                     generic-service
    host_name               sports-server
    service_description     Sports Application HTTP
    check_command           check_http
}

```

This checks whether the sports application responds to HTTP requests.

### **Step 7: Configure CPU monitoring**

For a remote Ubuntu server, install NRPE and Nagios plugins:

```
sudo apt update
sudo apt install nagios-nrpe-server nagios-plugins -y
```

Check:

```
sudo systemctl status nagios-nrpe-server
```

The NRPE agent must allow the Nagios server to connect and define the monitoring commands.

Add the following service definition in `sports.cfg`:

```

define service {
    use                     generic-service
    host_name               sports-server
    service_description     CPU Load
    check_command           check_nrpe!check_load
}

```

The remote NRPE configuration must define `check_load`.

You can similarly configure memory and disk checks using the installed monitoring plugins and add corresponding service definitions.

### **Step 8: Add sports.cfg to Nagios configuration**

Open:

```
vi /opt/nagios/etc/nagios.cfg
```

Add:

```
cfg_file=/opt/nagios/etc/objects/sports.cfg
```

Save the file.

### **Step 9: Validate configuration**

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

If there are no errors, the configuration is valid.

### **Step 10: Restart Nagios**

Exit:

```
exit
```

Restart:

```
docker restart nagios
```

Open:

```
http://localhost:8081
```

Go to **Hosts** and **Services** to check the monitoring results.

### **Step 11: Verify application availability**

If the SportSystem web application is available at an HTTP endpoint, Nagios can check it using `check_http`.

For the Nagios server to reach the application, the configured host address and port must be accessible from the Nagios container.

### **Important commands**

| Purpose          | Command                                                            |
| ---------------- | ------------------------------------------------------------------ |
| Start Minikube   | `minikube start --driver=docker`                                   |
| View pods        | `kubectl get pods`                                                 |
| Apply Deployment | `kubectl apply -f deployment.yaml`                                 |
| Apply Service    | `kubectl apply -f service.yaml`                                    |
| Scale            | `kubectl scale deployment sports-deployment --replicas=5`          |
| Delete pod       | `kubectl delete pod POD-NAME`                                      |
| Pull Nagios      | `docker pull jasonrivers/nagios:latest`                            |
| Run Nagios       | `docker run -d --name nagios -p 8081:80 jasonrivers/nagios:latest` |
| Restart Nagios   | `docker restart nagios`                                            |

## **Scenario-based questions and answers**

**1. A pod keeps restarting and shows CrashLoopBackOff. How do you troubleshoot it?**

First check:

```
kubectl get pods
```

Then:

```
kubectl describe pod POD-NAME
```

Check application logs:

```
kubectl logs POD-NAME
```

If it restarted previously:

```
kubectl logs POD-NAME --previous
```

Check:

- Application errors.
- Incorrect environment variables.
- Missing configuration.
- Database connection failures.
- Incorrect container startup command.
- Failed health checks.

Correct the issue and redeploy.

**2. A pod is stuck in Pending state. What are the reasons and how do you troubleshoot it?**

Possible reasons:

- Insufficient CPU or memory.
- No suitable node available.
- Persistent volume claim is not bound.
- Incorrect node selector or scheduling constraints.

Run:

```
kubectl describe pod POD-NAME
kubectl get nodes
kubectl get events
```

Check the Events section to identify the scheduling problem.

**3. Some pods are failing health checks. How do you identify the root cause?**

Run:

```
kubectl get pods
kubectl describe pod POD-NAME
kubectl logs POD-NAME
```

Check:

- Readiness probe configuration.
- Liveness probe configuration.
- Application port.
- Application startup time.
- Database connectivity.

Correct the probe configuration or application issue.

**4. Nagios displays "Unable to connect to Nagios process". How do you resolve it?**

Check whether the Nagios process is running:

```
docker ps
docker logs nagios
```

Enter the container:

```
docker exec -it nagios bash
```

Check configuration:

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

Correct configuration errors and restart:

```
docker restart nagios
```

**5. A new host added in Nagios is not visible in the dashboard. What do you check?**

Check:

1. Whether the host definition is correct.
2. Whether the configuration file is included in `nagios.cfg`.
3. Whether the host configuration has errors.
4. Whether Nagios was restarted after changes.

Validate:

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

Restart:

```
docker restart nagios
```

# **4. Docker Compose – Multi-Container Orchestration [25 Marks]**

**Aim:** Create and run three interdependent services using Docker Compose:

1. Flask web microservice.
2. PostgreSQL database.
3. Redis cache.

The question specifically asks for:

- Services: `web`, `db`, `redis`.
- `depends_on`.
- Custom network.
- Persistent volumes.
- Rebuilding and scaling workflows.

## **Step 1: Create project folder**

Open PowerShell:

```
mkdir SportsCompose
cd SportsCompose
```

Create this folder structure:

```
SportsCompose/
│
├── docker-compose.yml
│
└── web/
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```

## **Step 2: Create Flask application**

Inside the `web` folder, create:

```
app.py
```

Paste:

```
    port=6379,    decode_responses=True)@app.route("/")def home():    return "Sports Management System Running"@app.route("/db")def database():    conn = get_db()    cur = conn.cursor()    cur.execute("""        CREATE TABLE IF NOT EXISTS teams (            id SERIAL PRIMARY KEY,            name VARCHAR(100)        )    """)    cur.execute(        "INSERT INTO teams(name) VALUES (%s)",        ("Sports Team",)    )    conn.commit()    cur.execute("SELECT * FROM teams")    result = cur.fetchall()    cur.close()    conn.close()    return str(result)@app.route("/cache")def redis_cache():    cache.set("sport", "Football")    return "Redis Value: " + cache.get("sport")if __name__ == "__main__":    app.run(host="0.0.0.0", port=5000)
```

## **Step 3: Create requirements.txt**

Inside `web`, create:

```
requirements.txt
```

Paste:

```

Flask
psycopg2-binary
redis

```

This installs the Python dependencies.

## **Step 4: Create Dockerfile**

Inside `web`, create:

```
Dockerfile
```

Paste:

```

FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]

```

## **Step 5: Create docker-compose.yml**

Go back to the `SportsCompose` main folder.

Create:

```
docker-compose.yml
```

Paste the following complete code:

```

services:

  web:
    build: ./web
    container_name: sports-web
    ports:
      - "5000:5000"

    environment:
      DB_HOST: db
      POSTGRES_DB: sportsdb
      POSTGRES_USER: sportsuser
      POSTGRES_PASSWORD: sportspass
      REDIS_HOST: redis

    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

    networks:
      - sports-net

  db:
    image: postgres:16
    container_name: sports-db

    environment:
      POSTGRES_DB: sportsdb
      POSTGRES_USER: sportsuser
      POSTGRES_PASSWORD: sportspass

    volumes:
      - postgres-data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U sportsuser -d sportsdb"]
      interval: 5s
      timeout: 5s
      retries: 5

    networks:
      - sports-net

  redis:
    image: redis:7
    container_name: sports-redis

    networks:
      - sports-net

volumes:
  postgres-data:

networks:
  sports-net:
    driver: bridge

```

## **Step 6: Understand the Compose file**

| Section        | Purpose                                       |
| -------------- | --------------------------------------------- |
| `web`          | Flask web microservice                        |
| `db`           | PostgreSQL database                           |
| `redis`        | Redis cache                                   |
| `build: ./web` | Builds Flask image from Dockerfile            |
| `ports`        | Maps host port 5000 to container port 5000    |
| `depends_on`   | Controls service startup order and conditions |
| `networks`     | Connects services using a custom network      |
| `volumes`      | Preserves PostgreSQL data                     |
| `healthcheck`  | Checks whether PostgreSQL is ready            |

**Important:** `depends_on` with `service_healthy` waits for the database health check. It does not automatically guarantee that every database operation will succeed; the application should also handle connection failures.

## **Step 7: Start all services**

Open PowerShell inside the `SportsCompose` folder.

Run:

```
docker compose up -d
```

This builds the Flask image and starts all three services.

Check:

```
docker compose ps
```

Expected containers:

```
sports-web
sports-db
sports-redis
```

## **Step 8: Verify application**

Open:

```
http://localhost:5000
```

Expected:

```
Sports Management System Running
```

## **Step 9: Test PostgreSQL connectivity**

Open:

```
http://localhost:5000/db
```

Expected: A list containing the inserted sports team record.

The first request creates the table and inserts the sample record. Repeated requests insert additional rows.

## **Step 10: Test Redis connectivity**

Open:

```
http://localhost:5000/cache
```

Expected:

```
Redis Value: Football
```

This demonstrates Flask communicating with Redis.

## **Step 11: Check container logs**

```
docker compose logs
```

Check only the Flask service:

```
docker compose logs web
```

Check PostgreSQL:

```
docker compose logs db
```

Check Redis:

```
docker compose logs redis
```

## **Step 12: Rebuild only the web service**

Suppose the Flask application code changes.

Run:

```
docker compose up -d --build web
```

This rebuilds and recreates the web service without unnecessarily rebuilding the database and Redis services.

Check:

```
docker compose ps
```

## **Step 13: Scale the web service**

Run:

```
docker compose up -d --scale web=3
```

**Important:** The Compose file above uses `container_name: sports-web`. Remove that line before scaling, because fixed container names prevent multiple replicas of the same service.

Also, publishing a fixed host port with `"5000:5000"` prevents multiple replicas from all binding to the same host port. For this scaling demonstration, remove the `ports` section from `web` and use:

```
    expose:
      - "5000"
```

Then run:

```
docker compose up -d --scale web=3
```

The three web containers can communicate internally through the Compose network. To access them from the host through one stable port, add a reverse proxy/load balancer service.

Check:

```
docker compose ps
```

## **Step 14: Demonstrate persistent database storage**

Stop the stack:

```
docker compose down
```

Start it again:

```
docker compose up -d
```

Check:

```
docker compose ps
```

Open:

```
http://localhost:5000/db
```

PostgreSQL data should remain because the named volume `postgres-data` is retained.

**Do not use&#x20;**`docker compose down -v` if you want to retain the database volume.

## **Step 15: Stop the application**

```
docker compose down
```

This stops and removes the containers and network but preserves the named PostgreSQL volume.

## **Important Docker Compose commands**

| Requirement               | Command                              |
| ------------------------- | ------------------------------------ |
| Start services            | `docker compose up -d`               |
| Build and start           | `docker compose up -d --build`       |
| List services             | `docker compose ps`                  |
| View logs                 | `docker compose logs`                |
| View web logs             | `docker compose logs web`            |
| Rebuild only web          | `docker compose up -d --build web`   |
| Scale web                 | `docker compose up -d --scale web=3` |
| Stop containers           | `docker compose stop`                |
| Remove containers/network | `docker compose down`                |
| Remove volumes too        | `docker compose down -v`             |

## **Scenario-based questions and answers**

**1. You have two applications: a Node.js backend and a Python scheduled-job script. Should you run both in the same Docker container?**

No. It is generally better to run them in separate containers because each application has its own dependencies, process lifecycle and scaling requirements.

**2. Two containers try to expose port 8080 on the host. What happens?**

A port conflict occurs. Only one container can bind to that host port at a time.

Use different host ports:

```
services:
  app1:
    ports:
      - "8080:8080"

  app2:
    ports:
      - "8081:8080"
```

Now both applications can run.

**3. The frontend has a new release, but you do not want to restart backend or database containers. What should you do?**

Rebuild and recreate only the frontend service:

```
docker compose up -d --build frontend
```

**4. Tomcat is listening on port 8080, but the browser shows Connection Refused. Why?**

Possible reasons:

- Port 8080 is not mapped to the host.
- Firewall blocks the port.
- Tomcat is bound to an inaccessible interface.
- The container is stopped.
- The application is not running.
- Wrong IP address or port is being used.

Check:

```
docker ps
docker logs CONTAINER-NAME
```

**5. How do you identify which process uses port 3000 on Windows?**

In PowerShell:

```
netstat -ano | findstr :3000
```

The last column gives the PID.

Find the process:

```
tasklist | findstr PID
```

Replace `PID` with the actual process ID.

To identify the container using a published port:

```
docker ps
```

# **SET–7: Final revision before the lab**

**Experiment 1 – Jenkins**

- Scripted Pipeline
- `node`, `stage`, `tool`
- Git clone
- `mvn clean`
- `mvn compile`
- `mvn package`
- Poll SCM and GitHub webhook

**Experiment 2 – AWS**

- Ubuntu EC2
- Java, Git, Docker, Maven
- Clone SportSystem
- Build WAR
- Copy to Tomcat `webapps`
- Elastic IP
- `systemctl enable`

**Experiment 3 – Kubernetes + Nagios**

- `minikube start --driver=docker`
- Deployment and Service
- Scale replicas
- Delete pod and observe replacement
- Nagios Docker image
- Host and HTTP service monitoring

**Experiment 4 – Docker Compose**

- Flask
- PostgreSQL
- Redis
- `depends_on`
- Custom network
- Persistent volume
- Rebuild and scale

## **Most important commands to memorize**

**Jenkins / Maven**

```
mvn clean
mvn compile
mvn package
```

**AWS**

```
git clone https://github.com/sarasrija/SportSystem.git
mvn clean package
sudo systemctl enable tomcat10
```

**Kubernetes**

```
minikube start --driver=docker
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl scale deployment sports-deployment --replicas=5
kubectl get pods
```

**Docker Compose**

```
docker compose up -d
docker compose ps
docker compose logs
docker compose up -d --build web
docker compose up -d --scale web=3
docker compose down
```

**Important SET–7 differences to remember:**

- It has **4 experiments**, not 5.
- Jenkins is a **Scripted Pipeline**, not a Freestyle Project.
- AWS requires installing **Java, Git and Docker**.
- Kubernetes uses the **Docker driver** for Minikube.
- Docker Compose uses **Flask + PostgreSQL + Redis**, not WordPress + MySQL.








8)


# **SET–8: Software Engineering Lab Internal II**

Based on SET–8 from SE 19-11-26.zip

**GitHub repository:** [BloodBankSystem](https://github.com/sarasrija/BloodBanksystem.git?utm_source=chatgpt.com)

SET–8 contains 4 experiments, each carrying 25 marks.

| Experiment | Topic                                                           | Marks |
| ---------- | --------------------------------------------------------------- | ----- |
| 1          | Jenkins – Scripted Pipeline & Build Triggers                    | 25    |
| 2          | AWS – Cloud Deployment using EC2                                | 25    |
| 3          | Kubernetes & Nagios – Minikube Pods & Infrastructure Monitoring | 25    |
| 4          | Docker Compose – Multi-Container Orchestration                  | 25    |

# **1. Jenkins – Scripted Pipeline & Build Triggers [25 Marks]**

**Aim:** Create a Jenkins scripted pipeline for the Blood Bank Management System to perform Maven clean, install, test and package stages, with error handling.

Repository:

```
https://github.com/sarasrija/BloodBanksystem.git
```

## **Step 1: Verify Java, Maven and Git**

Open PowerShell:

```
java -version
mvn -version
git --version
```

Install any missing tools before proceeding.

## **Step 2: Install Jenkins plugins**

Open:

```
http://localhost:8080
```

Go to:

**Manage Jenkins → Plugins → Available plugins**

Install:

1. Git plugin
2. Pipeline plugin
3. Maven Integration plugin

Restart Jenkins if required.

## **Step 3: Configure JDK and Maven**

Go to:

**Manage Jenkins → Tools**

Configure:

```
JDK Name: JDK17
Maven Name: Maven3
```

Use the actual installation names configured in your Jenkins.

Save.

## **Step 4: Create a Pipeline project**

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter:

```
BloodBank-Pipeline
```

4. Select **Pipeline**.
5. Click OK.

## **Step 5: Configure GitHub repository**

In the Pipeline configuration, choose **Pipeline script from SCM** if your Jenkinsfile is stored in GitHub.

Enter:

```
SCM: Git
Repository URL: https://github.com/sarasrija/BloodBanksystem.git
Branch: */main
Script Path: Jenkinsfile
```

If the repository uses another default branch, select that branch.

Alternatively, choose **Pipeline script** and paste the script below directly.

## **Step 6: Create Jenkinsfile**

The question specifically asks for a scripted pipeline and Maven goals inside batch steps.

Create a file named:

```
Jenkinsfile
```

Paste:

```

node {
    stage('Configure Tools') {
        def jdkHome = tool 'JDK17'
        def mavenHome = tool 'Maven3'

        env.JAVA_HOME = jdkHome
        env.PATH = "${jdkHome}/bin;${mavenHome}/bin;${env.PATH}"

        echo "Build Number: ${env.BUILD_NUMBER}"
    }

    stage('Clone Repository') {
        git branch: 'main',
            url: 'https://github.com/sarasrija/BloodBanksystem.git'
    }

    stage('Clean') {
        bat 'mvn clean'
    }

    stage('Install') {
        bat 'mvn install -DskipTests'
    }

    stage('Test') {
        bat 'mvn test'
    }

    stage('Package') {
        bat 'mvn package'
    }

    stage('Archive Artifact') {
        archiveArtifacts artifacts: 'target/*.war',
            allowEmptyArchive: false
    }
}

```

**Important corrections for actual execution:**

- `bat` is for a Windows Jenkins agent. Use `sh` on Linux.
- The `JDK17` and `Maven3` names must match the Jenkins Tools configuration.
- `mvn install -DskipTests` skips tests during the install stage so the separate Test stage can demonstrate test execution. If your teacher expects `mvn install` without that option, use it, but remember Maven's install lifecycle already runs earlier lifecycle phases, including tests.
- If the project folder is named `bloodbankapp` rather than `mavenjava`, update any `dir('mavenjava')` reference in the script to `dir('bloodbankapp')`. The script above does not use a `dir()` block because it builds from the repository root.
- The repository must produce a WAR for `target/*.war` archiving to succeed. Check the actual output if it produces a JAR instead.

## **Step 7: Configure automated build trigger**

Go to:

**Job → Configure → Build Triggers**

Select **Poll SCM** and enter:

```
H/5 * * * *
```

This checks for repository changes approximately every five minutes.

Alternatively, configure a GitHub webhook:

1. Open GitHub repository.
2. Select **Settings → Webhooks**.
3. Click **Add webhook**.
4. Enter:

```
http://YOUR-JENKINS-ADDRESS/github-webhook/
```

5. Select `application/json`.
6. Enable push events.
7. Save.

Jenkins must be reachable from GitHub for webhooks to work. A local `localhost` address is not publicly reachable.

## **Step 8: Run the pipeline**

1. Open `BloodBank-Pipeline`.
2. Click **Build Now**.
3. Open the build number.
4. Click **Console Output**.

Expected stages:

```
Configure Tools
Clone Repository
Clean
Install
Test
Package
Archive Artifact
```

Expected successful result:

```
Finished: SUCCESS
```

## **Step 9: Verify WAR file**

Check the Jenkins build page under **Build Artifacts**.

Expected pattern:

```
target/*.war
```

The generated WAR should be available for download.

## **Step 10: Understand error handling**

In a scripted pipeline, if a `bat` command fails, it throws an exception and the pipeline stops by default. Therefore, if the Install stage fails, the Test and Package stages do not run.

To catch a failure and continue deliberately, you can use:

```
stage('Test') {
    def status = bat(
        script: 'mvn test',
        returnStatus: true
    )

    if (status != 0) {
        echo 'Test stage failed'
        currentBuild.result = 'UNSTABLE'
    }
}
```

This is an optional error-handling example, not a requirement to continue after every failure.

## **Scenario-based questions and answers**

**1. The project folder name is&#x20;**`bloodbankapp`**&#x20;instead of&#x20;**`mavenjava`**. Which parts of the script must you edit?**

Change every hardcoded folder reference:

```
dir('mavenjava')
```

to:

```
dir('bloodbankapp')
```

Also update any paths referring to the old folder. If the script builds directly from the workspace root, no folder name needs changing.

**2. If the Install stage fails, will the Test and Package stages run?**

No. By default, a failed `bat` command throws an exception and stops the scripted pipeline. Later stages are skipped unless error handling explicitly catches the failure.

**3. Your test cases failed, but the pipeline continued. How does Jenkins handle this?**

Normally, a failed `bat 'mvn test'` stops the pipeline. If `returnStatus: true`, `catchError`, or a `try-catch` block is used, the failure can be handled and the pipeline may continue.

**4. How do you print the current build number?**

Use:

```
echo "Build Number: ${env.BUILD_NUMBER}"
```

**5. What is the role of Jenkins in Continuous Integration?**

Jenkins automates source-code checkout, compilation, testing and packaging. It helps detect integration errors early and provides feedback to developers.

# **2. AWS – Cloud Deployment using EC2 [25 Marks]**

**Aim:** Deploy static web pages or a Java web application for the Blood Bank Management System on an Ubuntu EC2 instance.

Repository:

```
https://github.com/sarasrija/BloodBanksystem.git
```

The paper specifically mentions `/var/www/html/`, static assets, Java web applications and automatic startup setup. Follow the appropriate deployment path for the application you are given.

## **Step 1: Launch EC2 instance**

1. Open [AWS Console](https://console.aws.amazon.com/?utm_source=chatgpt.com).
2. Search for **EC2**.
3. Click **Launch Instance**.
4. Enter:

| Setting       | Value                               |
| ------------- | ----------------------------------- |
| Name          | BloodBank-Server                    |
| AMI           | Ubuntu Server                       |
| Instance type | t2.micro or suitable available type |
| Key pair      | Create a new key pair               |
| Storage       | Default                             |

5\. Click **Launch Instance**.

## **Step 2: Configure Security Group**

Add inbound rules:

| Type       | Port | Source                     |
| ---------- | ---- | -------------------------- |
| SSH        | 22   | My IP                      |
| HTTP       | 80   | 0.0.0.0/0                  |
| Custom TCP | 8080 | 0.0.0.0/0, if using Tomcat |

For security, restrict SSH to your administrator IP. Port 8080 is only needed if you access Tomcat directly.

## **Step 3: Connect to EC2**

Select the instance.

Click **Connect → SSH client**.

Open PowerShell in the folder containing your `.pem` key:

```
ssh -i "bloodbank-key.pem" ubuntu@YOUR-PUBLIC-IP
```

Replace `YOUR-PUBLIC-IP` with the EC2 public IPv4 address.

## **Step 4: Update Ubuntu**

```
sudo apt update
sudo apt upgrade -y
```

## **Step 5: Install required tools**

For static website hosting using Apache:

```
sudo apt install apache2 git -y
```

For a Java/Maven application, additionally install:

```
sudo apt install openjdk-17-jdk maven -y
```

Verify:

```
java -version
mvn -version
git --version
```

## **Step 6: Deploy a static HTML page**

Create a sample file:

```
nano index.html
```

Paste:

```

<!DOCTYPE html>
<html>
<head>
    <title>Blood Bank Management System</title>
</head>
<body>
    <h1>Welcome to Blood Bank Management System</h1>
    <p>Blood donation saves lives.</p>
    <h2>Available Blood Groups</h2>
    <ul>
        <li>A+</li>
        <li>B+</li>
        <li>O+</li>
        <li>AB+</li>
    </ul>
</body>
</html>

```

Save using `Ctrl + O`, Enter, then `Ctrl + X`.

Copy the page to Apache's web directory:

```
sudo cp index.html /var/www/html/index.html
```

Set suitable permissions:

```
sudo chmod 644 /var/www/html/index.html
```

## **Step 7: Start Apache**

```
sudo systemctl start apache2
```

Enable automatic startup:

```
sudo systemctl enable apache2
```

Check:

```
sudo systemctl status apache2
```

## **Step 8: Access the website**

Open:

```
http://YOUR-PUBLIC-IP
```

Expected output:

```
Welcome to Blood Bank Management System
```

## **Step 9: Deploy the GitHub project**

Clone the repository:

```
git clone https://github.com/sarasrija/BloodBanksystem.git
```

Enter the folder:

```
cd BloodBanksystem
```

Check files:

```
ls
```

If it contains static HTML, CSS and JavaScript assets, copy the appropriate files into:

```
/var/www/html/
```

For example, if the website files are in a folder named `frontend`:

```
sudo cp -r frontend/. /var/www/html/
```

Use the actual folder name in the repository.

If it is a Java Maven web application, build it:

```
mvn clean package
```

Then deploy the generated WAR to a compatible Tomcat installation. The paper does not mandate Tomcat for this experiment; use it only if the project is a Java web application.

## **Step 10: Configure automatic startup after reboot**

For Apache:

```
sudo systemctl enable apache2
```

For Tomcat:

```
sudo systemctl enable tomcat10
```

This ensures the relevant service starts automatically when the EC2 instance boots.

## **Step 11: Use EC2 User Data for automatic setup**

The EC2 feature asked for in the scenario is **User Data**.

When launching an instance, open **Advanced details → User data** and enter:

```
#!/bin/bash
apt update -y
apt install apache2 -y
systemctl enable apache2
systemctl start apache2

cat > /var/www/html/index.html <<'EOF'
<h1>Blood Bank Management System</h1>
<p>EC2 deployment successful</p>
EOF
```

This script runs during the instance's first boot by default. To run deployment scripts on every boot, configure a suitable systemd service or cloud-init configuration.

## **Scenario-based questions and answers**

**1. You deployed&#x20;**`index.html`**&#x20;into&#x20;**`/var/www/html/`**, but the webpage is not loading. What two possible issues would you check?**

- Apache may not be running.
- The EC2 Security Group may not allow inbound HTTP traffic on port 80.

Commands:

```
sudo systemctl status apache2
sudo systemctl restart apache2
```

Check the Security Group's inbound HTTP rule.

**2. The EC2 instance is running, but the public IP does not display your local&#x20;**`index.html`**. What initial steps will you take?**

1. Connect to EC2 using SSH.
2. Install Apache.
3. Copy the HTML file into `/var/www/html/`.
4. Start Apache.
5. Allow HTTP port 80 in the Security Group.
6. Open the public IP in the browser.

**3. You can access the application using&#x20;**`curl localhost:8080`**, but not from an external browser. What AWS setting is likely missing?**

The Security Group may not allow inbound TCP traffic on port 8080. Also check the instance firewall and network ACLs if necessary.

**4. Which EC2 feature runs setup commands automatically during instance startup?**

EC2 **User Data**, which uses cloud-init on Ubuntu.

**5. Why should SSH port 22 be restricted?**

To prevent unauthorized access and brute-force attacks. Allow SSH only from trusted administrator IP addresses rather than opening it to everyone.

# **3. Kubernetes & Nagios – Minikube Pods & Infrastructure Monitoring [25 Marks]**

**Aim:** Deploy the Blood Bank Management System using Minikube, manage pod lifecycles and configure Nagios to monitor server uptime, network devices and application health.

This experiment has two parts.

## **Part A: Kubernetes**

### **Step 1: Start Minikube**

Open PowerShell:

```
minikube start --driver=docker
```

Check:

```
minikube status
kubectl get nodes
```

Expected: Minikube node should show `Ready`.

### **Step 2: Clone the repository**

```
git clone https://github.com/sarasrija/BloodBanksystem.git
```

Enter:

```
cd BloodBanksystem
```

Check:

```
dir
```

### **Step 3: Build the application**

If it is a Maven project:

```
mvn clean package
```

Find the generated artifact:

```
Get-ChildItem -Recurse -Include *.war,*.jar
```

### **Step 4: Create a Docker image**

If the project produces a Tomcat-compatible WAR, create a `Dockerfile` in the project root.

```

FROM tomcat:10.1-jdk17

COPY target/BloodBanksystem.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

```

Replace the WAR filename with the actual generated file. Use a Tomcat version compatible with the application's Java and web framework.

Set Minikube's Docker environment:

```
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

Build the image:

```
docker build -t bloodbank-app:v1 .
```

Verify:

```
docker images
```

### **Step 5: Create Deployment YAML**

Create:

```
deployment.yaml
```

Paste:

```

apiVersion: apps/v1
kind: Deployment

metadata:
  name: bloodbank-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: bloodbank

  template:
    metadata:
      labels:
        app: bloodbank

    spec:
      containers:
        - name: bloodbank-container
          image: bloodbank-app:v1
          imagePullPolicy: IfNotPresent

          ports:
            - containerPort: 8080

```

This creates two pods. The question paper asks for scaled deployments but does not specify an exact replica count.

### **Step 6: Create Service YAML**

Create:

```
service.yaml
```

Paste:

```

apiVersion: v1
kind: Service

metadata:
  name: bloodbank-service

spec:
  type: NodePort

  selector:
    app: bloodbank

  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080

```

### **Step 7: Deploy the application**

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Check:

```
kubectl get deployments
kubectl get pods
kubectl get services
```

### **Step 8: Scale the Deployment**

For example, scale from two to four replicas:

```
kubectl scale deployment bloodbank-deployment --replicas=4
```

Verify:

```
kubectl get deployments
kubectl get pods
```

Expected:

```
bloodbank-deployment   4/4
```

You can use the replica count specified by your instructor if different.

### **Step 9: Access the application**

Run:

```
minikube service bloodbank-service
```

Alternatively:

```
minikube ip
```

Open:

```
http://MINIKUBE-IP:30080
```

### **Step 10: Demonstrate pod lifecycle management**

List pods:

```
kubectl get pods
```

Delete one pod:

```
kubectl delete pod POD-NAME
```

Check again:

```
kubectl get pods
```

Kubernetes creates a replacement pod because the Deployment maintains the desired replica count.

### **Step 11: Check application logs**

```
kubectl logs POD-NAME
```

For detailed information:

```
kubectl describe pod POD-NAME
```

### **Step 12: Clean up**

```
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

Stop Minikube:

```
minikube stop
```

## **Part B: Nagios monitoring**

**Aim:** Run Nagios inside Docker and inspect host status and monitoring dashboards.

### **Step 1: Pull Nagios image**

Open another PowerShell window:

```
docker pull jasonrivers/nagios:latest
```

### **Step 2: Run Nagios container**

```
docker run -d `
--name nagios `
-p 8081:80 `
jasonrivers/nagios:latest
```

Check:

```
docker ps
```

### **Step 3: Open Nagios dashboard**

Open:

```
http://localhost:8081
```

Common default credentials for this image:

```
Username: nagiosadmin
Password: nagios
```

### **Step 4: Check Nagios logs**

```
docker logs nagios
```

### **Step 5: Enter Nagios container**

```
docker exec -it nagios bash
```

Check Nagios process:

```
ps aux | grep nagios
```

Check configuration:

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

Exit:

```
exit
```

### **Step 6: Restart Nagios if necessary**

```
docker restart nagios
```

Check:

```
docker ps
docker logs nagios
```

### **Step 7: Review the dashboard**

Open:

```
http://localhost:8081
```

Check:

- Hosts.
- Services.
- Host status.
- Service status.
- Problems.
- Monitoring events.

For actual remote host and application monitoring, add host and service definitions in Nagios and configure the required plugins. For example, an HTTP check can monitor whether the Blood Bank portal is responding. A dashboard showing Nagios itself is running does not by itself prove that remote hosts are being monitored.

## **Important Kubernetes diagnostic commands**

| Purpose                  | Command                                                   |
| ------------------------ | --------------------------------------------------------- |
| View nodes               | `kubectl get nodes`                                       |
| View pods                | `kubectl get pods`                                        |
| Detailed pod information | `kubectl describe pod POD-NAME`                           |
| View logs                | `kubectl logs POD-NAME`                                   |
| Previous container logs  | `kubectl logs POD-NAME --previous`                        |
| View events              | `kubectl get events`                                      |
| View deployments         | `kubectl get deployments`                                 |
| View rollout history     | `kubectl rollout history deployment/bloodbank-deployment` |
| Rollback                 | `kubectl rollout undo deployment/bloodbank-deployment`    |

## **Scenario-based questions and answers**

**1. How would you debug a failed Deployment using kubectl diagnostic commands?**

First:

```
kubectl get deployments
kubectl get pods
```

Get detailed information:

```
kubectl describe deployment bloodbank-deployment
```

Inspect a failed pod:

```
kubectl describe pod POD-NAME
```

Check application logs:

```
kubectl logs POD-NAME
```

Check events:

```
kubectl get events
```

These commands help identify image-pull errors, scheduling issues, failed probes and application errors.

**2. How do you roll back a faulty Kubernetes Deployment?**

Run:

```
kubectl rollout undo deployment/bloodbank-deployment
```

Verify:

```
kubectl rollout status deployment/bloodbank-deployment
```

The Deployment must have a previous revision recorded.

**3. How do you debug a running pod when application-level errors occur?**

Check logs:

```
kubectl logs POD-NAME
```

Enter the container:

```
kubectl exec -it POD-NAME -- sh
```

Inside the container, inspect configuration, application files, environment variables and connectivity.

**4. How can you check whether the Nagios monitoring daemon is running?**

For a Nagios container:

```
docker ps
docker logs nagios
docker exec -it nagios bash
```

Inside:

```
ps aux | grep nagios
```

Also validate the configuration:

```
/usr/local/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

**5. What are the main advantages of Nagios?**

- Monitors hosts and services.
- Detects failures and resource problems.
- Provides WARNING and CRITICAL alerts.
- Helps identify infrastructure problems.
- Supports proactive monitoring and recovery notifications.

# **4. Docker Compose – Multi-Container Orchestration [25 Marks]**

**Aim:** Create and run a multi-container Blood Bank application stack using Docker Compose with:

1. Web frontend container.
2. PostgreSQL database container.
3. Custom Docker network.
4. Persistent database volume.

## **Step 1: Create project folder**

Open PowerShell:

```
mkdir BloodBankCompose
cd BloodBankCompose
```

Create this structure:

```
BloodBankCompose/
│
└── docker-compose.yml
```

## **Step 2: Create docker-compose.yml**

Open VS Code and create:

```
docker-compose.yml
```

Paste:

```

services:

  web:
    image: nginx:alpine
    container_name: bloodbank-web

    ports:
      - "8080:80"

    depends_on:
      - db

    networks:
      - bloodbank-net

  db:
    image: postgres:16
    container_name: bloodbank-db

    environment:
      POSTGRES_DB: bloodbank
      POSTGRES_USER: blooduser
      POSTGRES_PASSWORD: BloodPass@123

    volumes:
      - postgres-data:/var/lib/postgresql/data

    networks:
      - bloodbank-net

volumes:
  postgres-data:

networks:
  bloodbank-net:
    driver: bridge

```

**Note:** This is a runnable demonstration stack using Nginx as the frontend and PostgreSQL as the database. For the actual Blood Bank application, replace `nginx:alpine` with its frontend/backend Docker image and configure the application to connect to PostgreSQL using hostname `db`, port `5432`, database `bloodbank`, and the configured credentials. `depends_on` controls startup order but does not guarantee database readiness.

## **Step 3: Understand the Compose file**

| Configuration    | Purpose                                    |
| ---------------- | ------------------------------------------ |
| `web`            | Web frontend service                       |
| `db`             | PostgreSQL database service                |
| `image`          | Docker image used by the service           |
| `ports`          | Maps host port 8080 to container port 80   |
| `depends_on`     | Controls service startup order             |
| `environment`    | Configures PostgreSQL database credentials |
| `networks`       | Connects both containers                   |
| `volumes`        | Preserves PostgreSQL data                  |
| `driver: bridge` | Creates a custom bridge network            |

## **Step 4: Start all containers**

Make sure Docker Desktop is running.

From the `BloodBankCompose` folder:

```
docker compose up -d
```

This starts both services.

Check:

```
docker compose ps
```

Expected containers:

```
bloodbank-web
bloodbank-db
```

## **Step 5: Verify frontend in browser**

Open:

```
http://localhost:8080
```

Expected: Nginx welcome page.

This confirms the frontend container is accessible.

## **Step 6: Verify PostgreSQL container**

Enter the database container:

```
docker exec -it bloodbank-db psql -U blooduser -d bloodbank
```

Inside PostgreSQL:

```
CREATE TABLE donors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    blood_group VARCHAR(5)
);

INSERT INTO donors(name, blood_group)
VALUES ('Rahul', 'O+');

SELECT * FROM donors;
```

Exit:

```
\q
```

## **Step 7: Verify network connectivity**

Inspect the custom network:

```
docker network inspect bloodbank-net
```

Both containers should appear in the network.

The database hostname available to another container on this network is:

```
db
```

The PostgreSQL port is:

```
5432
```

## **Step 8: Verify persistent volume**

List volumes:

```
docker volume ls
```

Expected volume:

```
bloodbankcompose_postgres-data
```

The exact name may have a Compose project prefix.

Inspect:

```
docker volume inspect bloodbankcompose_postgres-data
```

## **Step 9: Stop services without removing containers**

```
docker compose stop
```

Check:

```
docker compose ps
```

The containers remain but are stopped.

## **Step 10: Start the same containers again**

```
docker compose start
```

Check:

```
docker compose ps
```

Open:

```
http://localhost:8080
```

## **Step 11: Rebuild or update services**

If you modify your application Dockerfile or source code, rebuild and start:

```
docker compose up -d --build
```

To rebuild only the web service:

```
docker compose up -d --build web
```

## **Step 12: Stop and remove containers and network**

Run:

```
docker compose down
```

This removes:

- Containers.
- Compose-created default network.

It preserves named volumes by default.

Verify:

```
docker ps -a
docker volume ls
```

**Important:** Do not run `docker compose down -v` if you need to preserve PostgreSQL data.

## **Important Docker Compose commands**

| Requirement               | Command                            |
| ------------------------- | ---------------------------------- |
| Start services            | `docker compose up -d`             |
| Check services            | `docker compose ps`                |
| View logs                 | `docker compose logs`              |
| Stop services             | `docker compose stop`              |
| Start stopped services    | `docker compose start`             |
| Rebuild services          | `docker compose up -d --build`     |
| Rebuild only web          | `docker compose up -d --build web` |
| Remove containers/network | `docker compose down`              |
| Remove volumes too        | `docker compose down -v`           |

## **Scenario-based questions and answers**

**1. How do you stop and clean up the running container stack without losing volume data?**

Run:

```
docker compose down
```

It removes containers and the Compose network but preserves named volumes.

**2. You updated code and want to restart frontend, backend and DB containers quickly. What command can you use?**

```
docker compose up -d --build
```

It rebuilds services that need rebuilding and starts the stack. If you only need to restart existing containers without rebuilding:

```
docker compose restart
```

**3. How do you share your application configuration and code as a Docker image bundle?**

First build the image:

```
docker build -t bloodbank-app:v1 .
```

Save it:

```
docker save -o bloodbank-app.tar bloodbank-app:v1
```

Share `bloodbank-app.tar` with your teammate.

The teammate loads it using:

```
docker load -i bloodbank-app.tar
```

Alternatively, push the image to Docker Hub:

```
docker login
docker tag bloodbank-app:v1 YOUR-DOCKERHUB-USERNAME/bloodbank-app:v1
docker push YOUR-DOCKERHUB-USERNAME/bloodbank-app:v1
```

**4. What happens when two containers expose the exact same host port?**

A port conflict occurs. The second container cannot bind to that already-used host port.

Use different host ports:

```
services:
  app1:
    image: nginx
    ports:
      - "8080:80"

  app2:
    image: nginx
    ports:
      - "8081:80"
```

Both containers can now run.

**5. Why specify custom networks in Docker Compose instead of relying only on default bridge networking?**

Custom networks provide:

- Isolated communication between related services.
- Service-name DNS resolution.
- Easier management of interdependent containers.
- Better separation between different application stacks.

# **SET–8: Final revision before the lab**

**Experiment 1 – Jenkins**

- Scripted Pipeline
- `node`, `stage`, `tool`
- GitHub clone
- `mvn clean`
- `mvn install`
- `mvn test`
- `mvn package`
- `env.BUILD_NUMBER`
- `bat` for Windows

**Experiment 2 – AWS**

- Ubuntu EC2
- Apache installation
- `/var/www/html/`
- Public IP
- Security Group
- EC2 User Data
- `systemctl enable`

**Experiment 3 – Kubernetes + Nagios**

- `minikube start --driver=docker`
- Deployment and pod lifecycle
- Scale replicas
- `kubectl describe`
- `kubectl logs`
- `kubectl rollout undo`
- Nagios Docker container and dashboard

**Experiment 4 – Docker Compose**

- Web frontend
- PostgreSQL
- Custom network
- Persistent volume
- `docker compose up`
- `stop`, `start`, `down`
- `docker save`, `docker load`

## **Most important commands to memorize**

**Jenkins / Maven**

```
mvn clean
mvn install
mvn test
mvn package
```

**AWS**

```
sudo apt update
sudo apt install apache2 git -y
sudo cp index.html /var/www/html/
sudo systemctl start apache2
sudo systemctl enable apache2
```

**Kubernetes**

```
minikube start --driver=docker
kubectl get pods
kubectl describe pod POD-NAME
kubectl logs POD-NAME
kubectl rollout undo deployment/bloodbank-deployment
```

**Nagios**

```
docker pull jasonrivers/nagios:latest
docker run -d --name nagios -p 8081:80 jasonrivers/nagios:latest
docker ps
docker logs nagios
docker restart nagios
```

**Docker Compose**

```
docker compose up -d
docker compose ps
docker compose stop
docker compose start
docker compose up -d --build
docker compose down
```

**Important SET–8 differences from SET–7:**

- Jenkins specifically uses Maven goals `clean`, `install`, `test`, and `package`, with batch steps.
- AWS emphasizes static pages in `/var/www/html/` and EC2 User Data.
- Kubernetes questions focus on debugging, rollback and pod lifecycle.
- Docker Compose uses a **web frontend + PostgreSQL**, not Flask + PostgreSQL + Redis.





9)
# **SET–9 – Software Engineering Lab Internal II**

Source: `SE 19-11-26.zip` → `19-11-26/SET-9.docx`

**GitHub Repository:** [ApartmentSystem](https://github.com/sarasrija/ApartmentSystem.git?utm_source=chatgpt.com)

This set contains four experiments, each carrying 25 marks:

1. Jenkins – GitHub Webhook, ngrok and Tomcat deployment
2. AWS EC2 – Docker deployment
3. Kubernetes – 4 replicas, scaling and Nagios monitoring
4. Docker Compose – Nginx/Tomcat and PostgreSQL 15

I'll give the procedure, commands, files, verification and all five scenario answers for each experiment separately.

## **Experiment 1 – Jenkins with GitHub Webhook and Tomcat**

**Objective:** Automatically build the Apartment Management System whenever code is pushed to GitHub, and display the build in Jenkins Pipeline View.

### **A. Start Jenkins and ngrok**

1. Start Jenkins on your computer.
2. Open Jenkins:

```
http://localhost:8080
```

3\. Open PowerShell and start ngrok:

```
ngrok http 8080
```

4\. ngrok displays a forwarding URL similar to:

```
https://abcd-1234.ngrok-free.app
```

Keep the ngrok terminal running. Your webhook URL will be:

```
https://abcd-1234.ngrok-free.app/github-webhook/
```

Replace the sample URL with your actual ngrok URL.

ngrok exposes your local Jenkins server through a public forwarding address so GitHub can reach it.

### **B. Install required Jenkins plugins**

Go to:

`Manage Jenkins → Plugins → Available plugins`

Install:

- Git plugin
- GitHub Integration plugin
- Pipeline plugin
- Pipeline: Stage View (if available)

Restart Jenkins if requested.

### **C. Configure Maven and JDK**

Go to:

`Manage Jenkins → Tools`

Configure:

- JDK: use an installed JDK name, for example `JDK17`
- Maven: add Maven and name it `Maven3`

Use the exact names in your Jenkinsfile.

### **D. Clone the GitHub repository**

Open PowerShell:

```
git clone https://github.com/sarasrija/ApartmentSystem.git
cd ApartmentSystem
```

Check the project files:

```
dir
```

Look for `pom.xml`. The Maven commands below assume the repository has a Maven project with a `pom.xml`.

### **E. Create Jenkins Pipeline**

1. Open Jenkins dashboard.
2. Click **New Item**.
3. Enter:

```
ApartmentSystem
```

4. Select **Pipeline**.
5. Click OK.
6. Under Pipeline, choose **Pipeline script**.
7. Paste the following Scripted Pipeline.

```
node {
    stage('Clean Workspace') {
        cleanWs()
    }

    stage('Clone Repository') {
        git branch: 'main',
            url: 'https://github.com/sarasrija/ApartmentSystem.git'
    }

    stage('Maven Clean') {
        def mvnHome = tool 'Maven3'
        bat "\"${mvnHome}\\bin\\mvn\" clean"
    }

    stage('Compile') {
        def mvnHome = tool 'Maven3'
        bat "\"${mvnHome}\\bin\\mvn\" compile"
    }

    stage('Unit Testing') {
        def mvnHome = tool 'Maven3'
        bat "\"${mvnHome}\\bin\\mvn\" test"
    }

    stage('Package WAR') {
        def mvnHome = tool 'Maven3'
        bat "\"${mvnHome}\\bin\\mvn\" package"
    }

    stage('Archive Artifact') {
        archiveArtifacts artifacts: '**/target/*.war',
                         allowEmptyArchive: false
    }
}
```

This version is for a Windows Jenkins agent. If Jenkins runs on Linux, replace each `bat` with `sh` and use the Linux Maven path.

**Important:** If the project does not have a Maven `pom.xml`, or does not produce a WAR, these Maven/WAR stages must be adjusted to match its actual build configuration.

### **F. Configure GitHub Webhook**

In Jenkins:

1. Open `ApartmentSystem → Configure`.
2. Under **Build Triggers**, select:

```
GitHub hook trigger for GITScm polling
```

3\. Save.

In GitHub:

1. Open the ApartmentSystem repository.
2. Go to `Settings → Webhooks → Add webhook`.
3. Enter your ngrok URL:

```
https://YOUR-NGROK-URL/github-webhook/
```

4\. Content type:

```
application/json
```

5. Select **Just the push event**.
6. Click **Add webhook**.

Now push a change to GitHub:

```
git add .
git commit -m "Test Jenkins webhook"
git push origin main
```

Jenkins should automatically start a build.

### **G. Create Pipeline View**

1. Go to Jenkins Dashboard.
2. Click the `+` icon or **New View**.
3. Enter:

```
ApartmentPipeline
```

4. Select **Pipeline View**.
5. Select the `ApartmentSystem` job as the initial job.
6. Click OK and Save.

Open the view to see the pipeline stages and build status.

### **H. Deploy WAR to Tomcat**

1. Install and start Apache Tomcat on the machine where you are deploying.
2. Open the Jenkins job's latest successful build.
3. Go to **Build Artifacts** and download the generated WAR.
4. Copy the WAR file into Tomcat's `webapps` directory.

For a typical Windows Tomcat installation:

```
C:\Program Files\Apache Software Foundation\Tomcat 10.1\webapps\
```

5. Start or restart Tomcat.
6. Open:

```
http://localhost:8080/ApartmentSystem/
```

The context path depends on the actual WAR filename. For example, `app.war` normally deploys as `/app`.

Tomcat version and Java compatibility must match the application's requirements. The question paper does not specify the exact Tomcat version or WAR filename.

### **Expected output**

- Jenkins build starts after a GitHub push.
- Pipeline stages appear in Pipeline View.
- Maven build and test results appear in Console Output.
- WAR artifact is generated.
- Application is deployed on Tomcat.

### **Scenario-based questions – Jenkins**

**1. Webhook is configured, but Jenkins does not trigger. What should you check in ngrok?**

Check whether the ngrok terminal is still running and whether it shows a forwarding address for port 8080. If ngrok has stopped, restart it using:

```
ngrok http 8080
```

Also check that GitHub has the current forwarding URL.

**2. How does Jenkins display Java compilation errors?**

Jenkins displays compiler errors in the **Console Output**, including the Java filename, line number and error description. The Compile stage becomes failed.

**3. If compilation succeeds but a unit test fails, will Jenkins create the WAR?**

With the pipeline above, the Test stage fails and the later Package stage does not execute. Therefore, the WAR is not generated by that run.

**4. Why is ngrok necessary?**

Jenkins running on `localhost` is accessible only from the local machine/network. GitHub's servers cannot directly access that local address. ngrok provides a public forwarding URL so GitHub can send webhook requests to Jenkins.

**5. Difference between Continuous Integration and Continuous Deployment?**

| Continuous Integration (CI)             | Continuous Deployment (CD)                                                      |
| --------------------------------------- | ------------------------------------------------------------------------------- |
| Developers frequently merge code.       | Changes are automatically deployed to production after passing required checks. |
| Code is automatically built and tested. | Application is released to users automatically.                                 |
| Detects errors early.                   | Makes delivery faster.                                                          |

## **Experiment 2 – AWS EC2 deployment using Docker**

**Objective:** Deploy Apartment Management System on an Ubuntu EC2 instance using Docker and Tomcat.

### **A. Launch an EC2 instance**

[AWS Compute EC2 Demo - KodeKloud](https://images.openai.com/static-rsc-4/68CcIF9Zd-z_dPOp21MB4kfC_6lv_DXiROIyt60_xKJZSfFtRQW1ALy7gdV3A_NR5Xu5YtjoTdgYTZgdQgTxTiC5meCEo2bRL4RSjS85NSGKSfewU5smfUy085rZPEzYEyTazW6CT7A6GF_lT0cLpobZ3egOXZ_PZ1hNSEnB1HM?purpose=inline)

[notes.kodekloud.com](https://notes.kodekloud.com/docs/AWS-Cloud-Practitioner-CLF-C02/Technology-Part-One/AWS-Compute-EC2-Demo/page?utm_source=chatgpt.com)

1. Sign in to AWS Console.
2. Open **EC2 → Instances → Launch instances**.
3. Enter instance name:

```
ApartmentSystem
```

4. Select Ubuntu Server.
5. Choose `t2.micro` if available and eligible under your account's current free-tier offer.
6. Create or select a key pair and download the `.pem` file.
7. Configure Security Group inbound rules:

| Type       | Port | Source    |
| ---------- | ---- | --------- |
| SSH        | 22   | My IP     |
| Custom TCP | 8090 | 0.0.0.0/0 |

8\. Launch the instance.

### **B. Connect to EC2 using SSH**

Open PowerShell in the folder containing your `.pem` file:

```
ssh -i "your-key.pem" ubuntu@YOUR_EC2_PUBLIC_IP
```

Replace the key filename and public IP with your actual values.

If permission errors occur, use the downloaded key with the appropriate Windows file permissions.

### **C. Update Ubuntu and install Docker and Git**

Run these commands inside the EC2 terminal:

```
sudo apt update
sudo apt upgrade -y
```

Install Docker:

```
sudo apt install docker.io git -y
```

Start Docker:

```
sudo systemctl start docker
sudo systemctl enable docker
```

Allow the Ubuntu user to run Docker:

```
sudo usermod -aG docker ubuntu
```

Reconnect to SSH for the group change to take effect.

Check:

```
docker --version
git --version
```

### **D. Clone the repository**

```
git clone https://github.com/sarasrija/ApartmentSystem.git
cd ApartmentSystem
ls
```

Inspect the repository and locate the Maven project and its `pom.xml`.

### **E. Create Dockerfile**

Create a file named `Dockerfile` in the directory containing the WAR file or build configuration.

For a Maven project that produces a WAR compatible with Tomcat 10:

```
FROM tomcat:10.1-jdk17

RUN rm -rf /usr/local/tomcat/webapps/*

COPY target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

Build the WAR first if the repository has a Maven project:

```
sudo apt install maven openjdk-17-jdk -y
mvn clean package
```

Then build the Docker image:

```
docker build -t apartment-app .
```

This Dockerfile assumes the application produces a WAR compatible with Tomcat 10 and Java 17. If it uses a different servlet/Jakarta version or Java version, select a compatible Tomcat base image.

### **F. Run the Docker container on port 8090**

The paper asks to map the EC2 host port to container port 8090. Tomcat's usual internal port is 8080, so map host `8090` to container `8080`:

```
sudo docker run -d \
  --name apartment-container \
  -p 8090:8080 \
  apartment-app
```

Check the container:

```
sudo docker ps
```

View logs:

```
sudo docker logs apartment-container
```

### **G. Verify application**

Open this URL in your browser:

```
http://YOUR_EC2_PUBLIC_IP:8090
```

If it does not open, check:

- EC2 instance is running.
- Docker container is running.
- Security Group allows inbound TCP port 8090.
- Tomcat started successfully.
- The application was deployed inside Tomcat.

### **Expected output**

- Ubuntu EC2 instance running.
- Docker image built successfully.
- Container running in detached mode.
- Application accessible through EC2 public IP on port 8090.

### **Scenario-based questions – AWS**

**1. Container is running on port 8090 inside EC2, but browser uses port 8080. What happens?**

The browser may show a connection refused or timeout error because it is accessing the wrong host port. Update the EC2 Security Group to allow the actual host port, `8090`, and use:

```
http://PUBLIC-IP:8090
```

**2. Why is SSH port 22 open to everyone (**`0.0.0.0/0`**) a security risk?**

Anyone on the internet can attempt to connect to the server. This increases the risk of brute-force attacks and unauthorized access. Restrict SSH access to your own IP address.

**3. Commands to update Ubuntu and install Docker?**

```
sudo apt update
sudo apt upgrade -y
sudo apt install docker.io git -y
sudo systemctl start docker
sudo systemctl enable docker
```

**4. How do you view container logs if Tomcat crashes?**

```
sudo docker logs apartment-container
```

To continuously follow logs:

```
sudo docker logs -f apartment-container
```

**5. How can application availability be improved?**

- Run multiple application replicas.
- Use a load balancer.
- Configure automatic restart and health checks.
- Monitor server and application health.
- Use backups and recovery procedures.

## **Experiment 3 – Kubernetes Deployment, Auto Scaling and Nagios**

**Objective:** Deploy Apartment Management System in Minikube with 4 replicas, expose it using a Service, scale it and monitor system health using Nagios.

### **A. Start Minikube**

Open PowerShell:

```
minikube start --driver=docker
```

Check cluster:

```
minikube status
kubectl get nodes
```

If `kubectl` is not recognized, install Kubernetes CLI or use:

```
minikube kubectl -- get nodes
```

### **B. Build the application Docker image**

From the ApartmentSystem project directory, make sure the Dockerfile from Experiment 2 is available and the WAR is built.

Build the image:

```
docker build -t apartment-app:v1 .
```

Make the image available to Minikube:

```
minikube image load apartment-app:v1
```

### **C. Create Deployment with 4 replicas**

Create a file named `deployment.yaml`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apartment-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: apartment
  template:
    metadata:
      labels:
        app: apartment
    spec:
      containers:
      - name: apartment-container
        image: apartment-app:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
```

Apply:

```
kubectl apply -f deployment.yaml
```

Check:

```
kubectl get deployments
kubectl get pods
```

You should see four application pods once they become ready.

### **D. Expose the application using a Service**

Create `service.yaml`:

```
apiVersion: v1
kind: Service
metadata:
  name: apartment-service
spec:
  type: NodePort
  selector:
    app: apartment
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
```

Apply:

```
kubectl apply -f service.yaml
```

Check:

```
kubectl get services
```

Open the application:

```
minikube service apartment-service
```

Alternatively, use port forwarding:

```
kubectl port-forward service/apartment-service 8081:8080
```

Open:

```
http://localhost:8081
```

### **E. Scale the deployment**

Increase the number of replicas to 6 as an example:

```
kubectl scale deployment apartment-deployment --replicas=6
```

Check:

```
kubectl get deployments
kubectl get pods
```

You should now see six pods.

The question requires 4 replicas initially but does not specify the final scale count; 6 is an example.

### **F. Manage and troubleshoot pods**

List pods:

```
kubectl get pods
```

Detailed pod information:

```
kubectl describe pod POD_NAME
```

View application logs:

```
kubectl logs POD_NAME
```

Delete a pod to test recovery:

```
kubectl delete pod POD_NAME
```

Check again:

```
kubectl get pods
```

Kubernetes Deployment automatically creates a replacement pod to maintain the desired replica count.

### **G. Run Nagios using Docker**

Pull the Nagios image:

```
docker pull jasonrivers/nagios:latest
```

Run it:

```
docker run -d --name nagios -p 8082:80 jasonrivers/nagios:latest
```

Check:

```
docker ps
```

Open the Nagios dashboard:

```
http://localhost:8082/nagios
```

The commonly used default login for this image is:

```
Username: nagiosadmin
Password: nagios
```

If the login does not work, check the image's configured credentials.

### **H. Monitor CPU, memory, disk and uptime**

Nagios monitors hosts and services through configured checks and plugins. A running Nagios dashboard alone does not automatically mean all four resources are being monitored.

For a simple Minikube resource check, run:

```
kubectl top nodes
kubectl top pods
```

These commands require Metrics Server/resource metrics to be available. For disk and uptime monitoring, configure appropriate Nagios host/service checks and plugins on the target machine.

### **Expected output**

- Minikube cluster running.
- Deployment with four replicas.
- Kubernetes Service exposing the application.
- Deployment successfully scaled.
- Nagios dashboard accessible.
- Pod status and resource metrics can be checked.

### **Scenario-based questions – Kubernetes and Nagios**

**1. If an application pod crashes, how does Kubernetes maintain replicas?**

The Deployment controller continuously checks the desired number of replicas. If a pod crashes, Kubernetes creates a replacement pod to maintain the required replica count.

**2. Difference between Pod IP and Service IP?**

| Pod IP                                     | Service IP                                         |
| ------------------------------------------ | -------------------------------------------------- |
| Assigned to an individual pod.             | Assigned to a Kubernetes Service.                  |
| Can change when a pod is recreated.        | Remains stable while the Service exists.           |
| Used to communicate with a particular pod. | Provides a stable address to access matching pods. |

**3. Which command checks the list of pods and their statuses?**

```
kubectl get pods
```

**4. If NodePort does not open in the browser, how can you test locally?**

Use:

```
kubectl port-forward service/apartment-service 8081:8080
```

Then visit:

```
http://localhost:8081
```

**5. How can Nagios identify application failure when the server is running?**

Nagios uses service checks, such as HTTP checks, to test whether the application responds correctly. Even if the server is running, a failed HTTP check can mark the application service as critical.

## **Experiment 4 – Docker Compose with Nginx/Tomcat and PostgreSQL 15**

**Objective:** Run the web service and PostgreSQL database together using Docker Compose, with a custom network and persistent storage.

### **A. Create project folder**

Create a folder named:

```
compose-lab
```

Inside it, create:

```
compose-lab/
│
├── docker-compose.yml
└── index.html
```

### **B. Create index.html**

Paste:

```
<!DOCTYPE html>
<html>
<head>
    <title>Apartment Management System</title>
</head>
<body>
    <h1>Apartment Management System</h1>
    <p>Docker Compose is working successfully!</p>
</body>
</html>
```

### **C. Create docker-compose.yml**

Paste the following:

```
version: "3.9"

services:

  web:
    image: nginx:latest
    container_name: apartment-web
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
    depends_on:
      - db
    networks:
      - apartment-network

  db:
    image: postgres:15
    container_name: apartment-db
    restart: always
    environment:
      POSTGRES_DB: apartmentdb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin123
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - apartment-network

networks:
  apartment-network:
    driver: bridge

volumes:
  postgres-data:
```

**Understand the important parts:**

| Configuration       | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| `web`               | Nginx web service                                  |
| `db`                | PostgreSQL database                                |
| `ports`             | Maps host port 8080 to Nginx port 80               |
| `depends_on`        | Starts the database service before the web service |
| `networks`          | Allows containers to communicate                   |
| `volumes`           | Preserves database records                         |
| `POSTGRES_DB`       | Creates the database                               |
| `POSTGRES_USER`     | Sets the database username                         |
| `POSTGRES_PASSWORD` | Sets the database password                         |

### **D. Run Docker Compose**

Open PowerShell inside `compose-lab`:

```
docker compose up -d
```

Check running containers:

```
docker compose ps
```

Check logs:

```
docker compose logs
```

Open browser:

```
http://localhost:8080
```

You should see:

```
Apartment Management System
Docker Compose is working successfully!
```

### **E. Check PostgreSQL database**

Connect to the database container:

```
docker exec -it apartment-db psql -U admin -d apartmentdb
```

Create a table:

```
CREATE TABLE residents (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    apartment_no VARCHAR(20)
);
```

Insert a record:

```
INSERT INTO residents (name, apartment_no)
VALUES ('Rahul', 'A101');
```

Display records:

```
SELECT * FROM residents;
```

Exit:

```
\q
```

### **F. Check Docker network and volume**

List networks:

```
docker network ls
```

Inspect the custom network:

```
docker network inspect compose-lab_apartment-network
```

List volumes:

```
docker volume ls
```

The actual network name can vary based on the Compose project name.

### **G. Stop and restart containers**

Stop services:

```
docker compose stop
```

Start again:

```
docker compose start
```

Remove containers and network:

```
docker compose down
```

The named database volume remains, so the database records are preserved.

To remove the volume too:

```
docker compose down -v
```



Important

&#x20;Do not use `docker compose down -v` if you need to preserve database records.



### **H. Rebuild after code changes**

After changing application code or configuration:

```
docker compose up -d --build
```

For this particular example, Nginx serves `index.html` through a bind mount, so changes to that file are visible without rebuilding the image. The rebuild option is useful when your service image is built from a Dockerfile.

### **Expected output**

- Web and database containers running.
- Nginx page opens on `localhost:8080`.
- PostgreSQL database and table created.
- Custom network created.
- Database volume preserves records after containers are removed.

This is a working Nginx + PostgreSQL Compose demonstration. The Nginx page is static and does not itself connect to PostgreSQL. A real Apartment Management backend must be configured with the database hostname `db`, port `5432`, and the credentials above.

### **Scenario-based questions – Docker Compose**

**1. What is the main use of&#x20;**`depends_on`**?**

It specifies service startup order. In this example, Compose starts the `db` service before the `web` service. It does not guarantee that PostgreSQL is fully ready to accept connections.

**2. Why are persistent volumes used? What happens without them?**

Volumes preserve database files even when containers are removed. Without a persistent volume, database data stored only in the container's writable layer can be lost when that container is deleted.

**3. Why don't code changes always appear after&#x20;**`docker compose up -d`**? Which flag rebuilds the image?**

The existing image may still be used. Add `--build`:

```
docker compose up -d --build
```

**4. How do custom networks help containers communicate?**

Containers on the same user-defined network can communicate using service names. For example, the backend can connect to PostgreSQL using hostname:

```
db
```

It does not need a fixed IP address.

**5. How can database information be recovered after container deletion?**

If a named volume was used, recreate the containers using:

```
docker compose up -d
```

The database can use the existing volume and its saved records. If the volume was also deleted, recovery requires a separate backup.

## **Important last-minute revision – commands to remember**

| Experiment     | Most important commands                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Jenkins        | `ngrok http 8080`, `git push`, `mvn clean package`                                                                                                     |
| AWS            | `sudo apt update`, `sudo apt install docker.io git -y`, `docker build -t apartment-app .`, `docker logs apartment-container`                           |
| Kubernetes     | `minikube start --driver=docker`, `kubectl apply -f deployment.yaml`, `kubectl get pods`, `kubectl scale deployment apartment-deployment --replicas=6` |
| Docker Compose | `docker compose up -d`, `docker compose ps`, `docker compose logs`, `docker compose down`                                                              |

**One important port clarification for Experiment 2:** Standard Tomcat listens on container port `8080`, so the command I gave uses `-p 8090:8080`. If your faculty specifically expects container port `8090`, Tomcat's Connector must first be configured to listen on `8090`; then use `-p 8090:8090`.

**Remember these five viva points:**

1. Jenkins automates build, test and integration.
2. ngrok exposes local Jenkins to GitHub webhook requests.
3. Docker packages applications and dependencies into containers.
4. Kubernetes manages, scales and recovers application pods.
5. Docker Compose manages multiple related containers using one YAML file.






