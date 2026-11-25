SOFTWARE ENGINEERING LAB - PRACTICAL CHEAT SHEET
Short, sequential steps | Windows laptop | Commands + checks + common fixes

============================================================
EXPERIMENT 1 - SOFTWARE INSTALLATION + SRS
============================================================
1. Check installations (open CMD/Git Bash):
   git --version
   java -version
   mvn -version
   docker --version
   kubectl version --client
   minikube version
2. If command not recognized: install tool -> add to PATH if required -> close/reopen terminal -> retry.
3. SRS document sequence:
   a) Title / Introduction
   b) Abstract: project purpose + problem solved + tech used
   c) Functional Requirements (what system does)
   d) Non-Functional Requirements (security, performance, reliability, usability)
   e) Users/Actors
   f) User workflows (steps performed)
   g) Use cases (actors + actions + system boundary)
   h) UML diagrams if asked

============================================================
EXPERIMENT 2 - GIT LOCAL + REMOTE
============================================================
A. GitHub: Create NEW repository -> choose Private/Public as asked -> create WITHOUT README if pushing existing project.
B. Git Bash: enter project folder:
   cd ~/OneDrive/Desktop/SE
   pwd
   ls
C. Initialize + configure:
   git init
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
D. Add + commit:
   git status
   git add .
   git status
   git commit -m "Initial commit"
   git branch -M main
E. Add remote + push (replace URL):
   git remote add origin https://github.com/USERNAME/REPO.git
   git remote -v
   git push -u origin main
F. Verify on GitHub: refresh repo page; confirm files/folders appear.
G. Common Git commands:
   git clone URL | git status | git add . | git commit -m "message"
   git log | git log --oneline | git diff
   git branch | git branch name | git checkout name
   git checkout -b name | git merge name | git branch -d name
   git push origin main | git pull origin main | git fetch origin
   git remote -v | git remote add origin URL
   git remote set-url origin URL | git remote remove origin
   git reset filename | git revert COMMIT_ID
   touch file.txt | cat file.txt | ls
H. Errors:
   - not a git repo: cd into project folder; check pwd/ls.
   - origin exists: git remote -v; update URL with git remote set-url origin URL.
   - src refspec main: git add . -> commit -> git branch -M main -> push.
   - rejected/fetch first: git pull origin main --allow-unrelated-histories; resolve conflicts; git add .; git commit; push.
   - auth failed: use browser sign-in/PAT, not normal GitHub password.

============================================================
EXPERIMENT 3 - COLLABORATION, BRANCHES, CONFLICT, PATCH
============================================================
A. Create GitHub Organization (if task asks): profile/menu -> Organizations -> Create -> name -> free plan -> add collaborator.
B. Repository sharing: Repo Settings -> Collaborators/Manage Access -> invite partner -> partner ACCEPTS.
C. Partner clone:
   git config --global user.name "Partner Name"
   git config --global user.email "partner@example.com"
   git clone https://github.com/OWNER/REPO.git
   cd REPO
   git remote -v
D. Branch and edit:
   git checkout -b feature1
   (edit/create file)
   git add .
   git commit -m "Added feature"
   git push origin feature1
E. Update from main:
   git pull origin main
F. Conflict practice:
   git checkout -b branch1
   (edit same file) -> git add . -> git commit -m "branch change"
   git checkout main
   (edit SAME lines differently) -> git add . -> git commit -m "main change"
   git merge branch1
   If conflict: open file; remove <<<<<<<, =======, >>>>>>> markers; keep desired text; save.
   git add .
   git commit -m "Resolved merge conflict"
G. Patch:
   git diff > changes.patch
   cat changes.patch
   git apply changes.patch
   git status
H. Errors: conflict -> edit/remove markers/add/commit; permission denied -> check invite accepted/write access; patch won't apply -> correct folder/version.

============================================================
EXPERIMENT 4 - MAVEN JAVA + WEB PROJECT
============================================================
A. Eclipse Java Maven project:
   Eclipse -> File -> New -> Project -> Maven -> Maven Project -> Next
   choose maven-archetype-quickstart -> Next
   Group ID: com.example | Artifact ID: myapp -> Finish
B. Maven commands (terminal in project folder):
   mvn -version
   mvn clean
   mvn compile
   mvn test
   mvn package
   mvn install
   mvn clean install
   Check target/ folder; expect BUILD SUCCESS.
C. Maven meanings: clean=remove old target; compile=compile source; test=run tests; package=create JAR/WAR; install=put artifact in local repository.
D. Dependency: open pom.xml -> add dependency inside <dependencies> -> save -> Maven Update Project -> mvn clean install.
   Example Gson:
   <dependency>
     <groupId>com.google.code.gson</groupId>
     <artifactId>gson</artifactId>
     <version>2.10</version>
   </dependency>
E. Maven Web project:
   Eclipse -> File -> New -> Maven Project -> maven-archetype-webapp -> Group/Artifact IDs -> Finish.
   Structure: pom.xml; src/main/webapp; index.jsp; WEB-INF.
F. Servlet dependency (use matching project/Tomcat version):
   <dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>javax.servlet-api</artifactId>
     <version>4.0.1</version>
     <scope>provided</scope>
   </dependency>
G. Build WAR: mvn clean install -> check target/*.war.
H. Tomcat deploy: Eclipse Servers view -> New Server -> Apache Tomcat -> select install folder -> add project -> Start -> browser http://localhost:8080/APPNAME/
I. Errors: BUILD FAILURE -> Console first error; dependency missing -> verify coordinates/internet; JAVA_HOME -> set JDK; red marks -> Right-click project -> Maven -> Update Project; port 8080 busy -> stop other service/change port; WAR absent -> mvn package.

============================================================
EXPERIMENT 5 - DOCKER CLI
============================================================
A. Open Docker Desktop; wait until engine running.
B. Verify:
   docker --version
   docker info
C. Pull/list:
   docker pull nginx
   docker images
D. Run Nginx:
   docker run -d --name mynginx -p 8080:80 nginx
   docker ps
   docker ps -a
   Browser: http://localhost:8080 (Nginx welcome page)
E. Container commands:
   docker stop mynginx
   docker start mynginx
   docker restart mynginx
   docker logs mynginx
   docker inspect mynginx
   docker stats
   docker exec -it mynginx bash (if bash exists)
   exit
   docker rm mynginx (stop first if running)
F. Remove image: docker rmi nginx (remove containers using it first).
G. Network:
   docker network create mynetwork
   docker network ls
   docker network connect mynetwork mynginx
   docker network disconnect mynetwork mynginx
   docker network rm mynetwork (disconnect/remove attached containers first)
H. Volume:
   docker volume create myvolume
   docker volume ls
   docker volume inspect myvolume
   docker volume rm myvolume (not in use)
I. Errors: daemon -> Docker Desktop start; port busy -> use -p 8081:80; name exists -> docker ps -a then remove old stopped container; exits -> docker logs; exec -> container must be running.

============================================================
EXPERIMENT 6 - DOCKERFILE, BUILD, RUN, DOCKER HUB
============================================================
A. Create folder and enter:
   mkdir redis
   cd redis
B. Create file named EXACTLY Dockerfile (not Dockerfile.txt). Contents:
   FROM redis:latest
   CMD ["redis-server"]
C. Build image (dot is required):
   docker build -t redisnew .
   docker images
D. Run container:
   docker run -d --name myredisnew redisnew
   docker ps -a
   docker inspect myredisnew
E. Stop/start:
   docker stop myredisnew
   docker start myredisnew
F. Commit container to another image:
   docker commit myredisnew redisnewbackup
   docker images
G. Remove:
   docker rm myredisnew (stop first)
   docker rmi redisnewbackup
H. Docker Hub:
   docker login
   docker tag redisnew YOUR_USERNAME/redisnew:latest
   docker push YOUR_USERNAME/redisnew:latest
   On other machine: docker pull YOUR_USERNAME/redisnew:latest
   docker run -d --name myredis YOUR_USERNAME/redisnew:latest
I. Errors: Dockerfile missing -> correct filename/current folder; build error -> check Dockerfile; invalid image name -> lowercase; name exists -> remove old container; push denied -> login/check tag.

============================================================
EXPERIMENT 7 - DOCKER COMPOSE
============================================================
A. Create folder:
   mkdir compose-lab
   cd compose-lab
B. Create docker-compose.yml (indent with spaces; no tabs):
   services:
     web:
       image: nginx:latest
       ports:
         - "8080:80"
     db:
       image: postgres:15
       environment:
         POSTGRES_USER: demo
         POSTGRES_PASSWORD: demo
         POSTGRES_DB: demo_db
C. Start:
   docker compose up -d
D. Check:
   docker compose ps
   Browser http://localhost:8080
   docker compose logs
   docker compose logs -f (live logs)
E. Stop/start/remove:
   docker compose stop
   docker compose start
   docker compose restart
   docker compose down
F. Add Redis service inside services:
     redis:
       image: redis:alpine
   Then docker compose up -d -> docker compose ps.
G. Network + persistent volume example:
   services:
     web:
       image: nginx:latest
       ports:
         - "8080:80"
       networks:
         - app-net
     db:
       image: postgres:15
       environment:
         POSTGRES_USER: demo
         POSTGRES_PASSWORD: demo
         POSTGRES_DB: demo_db
       volumes:
         - db-data:/var/lib/postgresql/data
       networks:
         - app-net
   networks:
     app-net:
   volumes:
     db-data:
H. Run: docker compose up -d -> docker compose ps -> docker compose down.
I. Errors: YAML indentation -> align spaces; port busy -> change host port; wrong folder -> cd to compose file folder; DB connection -> use service name db; data loss -> named volume.

============================================================
EXPERIMENT 8 - JENKINS FREESTYLE MAVEN JOBS
============================================================
A. Open Jenkins: http://localhost:8080 -> login.
B. Java build job:
   Dashboard -> New Item -> name MavenJava_Build -> Freestyle -> OK.
   Source Code Management -> Git -> Repository URL -> Branch */main (or */master).
   Build Steps -> Invoke top-level Maven targets -> Maven configured tool -> Goals clean.
   Add second Maven step -> Goals install.
   Post-build -> Archive artifacts: **/*
   Post-build -> Build other projects: MavenJava_Test -> only if stable.
   Apply -> Save.
C. Test job:
   New Item -> MavenJava_Test -> Freestyle.
   Build Environment -> Delete workspace before build.
   Build Step -> Copy artifacts from another project -> MavenJava_Build -> **/*
   Build Step -> Invoke Maven -> test
   Post-build -> Archive artifacts **/* -> Save.
D. Pipeline View (if asked): install Pipeline/Build Pipeline View plugin if needed -> dashboard add view (+) -> name MavenJava_Pipeline -> Build Pipeline View -> initial job MavenJava_Build -> Save.
E. Web build/test/deploy chain:
   MavenWeb_Build (Git -> Maven clean + install -> archive **/* -> trigger MavenWeb_Test)
   MavenWeb_Test (delete workspace -> copy artifacts from MavenWeb_Build **/* -> Maven test -> archive -> trigger MavenWeb_Deploy)
   MavenWeb_Deploy (delete workspace -> copy artifacts from MavenWeb_Test **/* -> Deploy WAR/EAR plugin -> WAR pattern **/*.war -> context path -> Tomcat remote details/credentials -> Save)
F. Run: click Build Now -> open build number -> Console Output. Green = success.
G. Errors: Git checkout -> URL/branch/credentials; Maven not found -> Manage Jenkins > Tools; artifact missing -> archive pattern; downstream not triggered -> job name/trigger; Tomcat deploy -> URL/credentials/manager/port; view empty -> plugin and upstream settings.

============================================================
EXPERIMENT 9 - JENKINS PIPELINE SCRIPT + TRIGGERS
============================================================
A. Dashboard -> New Item -> MavenScriptPipeline -> Pipeline -> OK.
B. Pipeline section -> Definition: Pipeline script -> paste (edit URL, folder and Maven tool name):
   pipeline {
       agent any
       tools { maven 'MAVEN-HOME' }
       stages {
           stage('Git Clone and Clean') {
               steps {
                   bat 'git clone YOUR_GITHUB_REPOSITORY_URL'
                   bat 'mvn clean -f mavenjava'
               }
           }
           stage('Install') {
               steps { bat 'mvn install -f mavenjava' }
           }
           stage('Test') {
               steps { bat 'mvn test -f mavenjava' }
           }
           stage('Package') {
               steps { bat 'mvn package -f mavenjava' }
           }
       }
   }
C. Replace URL, cloned folder name mavenjava (if different), Maven tool name.
D. Apply -> Save -> Build Now -> build number -> Console Output. Stages should pass.
E. Build triggers (Job Configure -> Build Triggers):
   - Build periodically
   - Poll SCM
   - GitHub hook trigger
   - Build after other projects
   - Remote trigger
F. Schedule examples:
   Poll SCM: H/5 * * * * (approx every 5 minutes)
   Daily around 9:00: H 9 * * *
G. Errors: Maven tool name mismatch -> Tools; clone URL wrong -> correct URL; folder incorrect -> fix -f path; bat only Windows -> on Linux use sh; syntax -> check braces/quotes; agent unavailable -> check agent.

============================================================
EXPERIMENT 10 - MINIKUBE/KUBERNETES + NAGIOS
============================================================
PART A: MINIKUBE
A. Docker Desktop ON. Verify:
   docker --version
   minikube version
   kubectl version --client
B. Start cluster:
   minikube start --driver=docker
   minikube status
C. Create Nginx deployment:
   kubectl create deployment mynginx --image=nginx
   kubectl get deployments
   kubectl get pods (repeat until Running)
   kubectl describe pods
D. Expose:
   kubectl expose deployment mynginx --type=NodePort --port=80 --target-port=80
   kubectl get services
E. Scale to 4:
   kubectl scale deployment mynginx --replicas=4
   kubectl get deployments
   kubectl get pods (verify 4)
F. Access option 1 (keep terminal open):
   kubectl port-forward svc/mynginx 8081:80
   Browser http://localhost:8081
   Stop forwarding with Ctrl+C.
G. Access option 2:
   minikube service mynginx
   OR minikube service mynginx --url
H. Dashboard:
   minikube dashboard
I. Cleanup:
   kubectl delete deployment mynginx
   kubectl delete service mynginx
   kubectl get deployments
   minikube stop (optional)
   minikube delete (optional; deletes cluster)
J. Kubernetes troubleshooting:
   minikube won't start -> minikube status; Docker Desktop running?
   Pending -> kubectl describe pod POD
   ImagePullBackOff -> image name/internet
   CrashLoopBackOff -> kubectl logs POD
   service issue -> kubectl get svc
   pod not ready -> wait and kubectl get pods again
   wrong replicas -> kubectl scale deployment mynginx --replicas=4

PART B: NAGIOS DOCKER
A. Pull:
   docker pull jasonrivers/nagios:latest
B. Run (detached):
   docker run -d --name nagiosdemo -p 8888:80 jasonrivers/nagios:latest
C. Verify:
   docker ps
   Browser http://localhost:8888
   Login: nagiosadmin / nagios (if image default credentials unchanged)
D. Explore hosts, services, alerts/status.
E. Logs/cleanup:
   docker logs nagiosdemo
   docker logs -f nagiosdemo
   docker stop nagiosdemo
   docker rm nagiosdemo
   docker rmi jasonrivers/nagios:latest (optional)
F. Errors: port conflict -> -p 8889:80 and use localhost:8889; container exit -> logs; existing name -> remove old stopped container; allow initialization time.

============================================================
EXPERIMENT 11 - JENKINS CI/CD WEBHOOK + EMAIL
============================================================
PART A: GITHUB WEBHOOK
A. Jenkins running at http://localhost:8080.
B. Install/configure GitHub integration/plugin if required.
C. Start ngrok in separate terminal:
   ngrok http 8080
   Copy its HTTPS forwarding URL. KEEP ngrok running.
D. GitHub repo -> Settings -> Webhooks -> Add webhook:
   Payload URL = https://YOUR-NGROK-URL/github-webhook/
   Content type = application/json
   Event = Just the push event
   -> Add webhook.
E. Jenkins job -> Configure -> Build Triggers -> check "GitHub hook trigger for GITScm polling" -> Save.
F. Test in project terminal:
   git add .
   git commit -m "Webhook test"
   git push origin main
G. Verify Jenkins automatically starts a build.
H. Webhook errors: no trigger -> checkbox/plugin/job SCM; failed webhook -> URL/payload; localhost won't work externally -> ngrok; ngrok URL changes -> update webhook; build fails -> Console Output.

PART B: EMAIL
A. Gmail: enable 2-Step Verification -> create App Password (if available) -> keep it private.
B. Jenkins -> Manage Plugins -> install Email Extension Plugin -> restart if requested.
C. Manage Jenkins -> Configure System -> E-mail Notification:
   SMTP server smtp.gmail.com
   authentication enabled; username Gmail address; password App Password
   SSL enabled; port 465; Reply-To Gmail
   Send test email and verify.
D. Job -> Configure -> Post-build Actions -> Editable Email Notification:
   recipient list -> content type -> choose triggers (success/failure) -> Save.
E. Errors: auth -> App Password; missing email -> spam/recipient; SMTP -> host/port/network; plugin missing -> install plugin.
F. UML if asked: Class = classes/attributes/methods/relationships; Sequence = interactions over time; Component = software modules and connections.

============================================================
EXPERIMENT 12 - UBUNTU VM / AWS EC2 WEB DEPLOYMENT
============================================================
A. Push project to GitHub (skip if already pushed):
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/USERNAME/REPO.git
   git push -u origin main
B. AWS Console -> EC2 -> Launch Instance:
   Name ubuntu -> Ubuntu Server AMI -> eligible instance type (manual example t2.micro)
   Create/download .pem key (keep safe)
   Security group inbound: SSH 22 (restrict to own IP), HTTP 80, HTTPS 443, custom TCP 6060
   Storage example 8 GB -> Launch.
C. Wait instance Running + status checks passed. Copy Public IPv4.
D. Connect:
   AWS EC2 -> select instance -> Connect -> EC2 Instance Connect (if available)
   OR Git Bash in .pem folder:
   ssh -i AWS.pem ubuntu@PUBLIC_IP
   Type yes if first connection prompt.
E. On Ubuntu server:
   sudo apt update
   sudo apt-get install docker.io -y
   sudo apt install git -y
   sudo apt install nano -y
   sudo systemctl start docker
   sudo systemctl status docker (press q to exit)
   docker --version
F. Clone:
   git clone https://github.com/USERNAME/REPO.git
   cd REPO
   ls
G. Create Dockerfile (Tomcat 9 WAR example):
   nano Dockerfile
   Contents:
   FROM tomcat:9.0
   COPY target/*.war /usr/local/tomcat/webapps/
   EXPOSE 8080
   CMD ["catalina.sh", "run"]
   Save: Ctrl+O -> Enter -> Ctrl+X
H. IMPORTANT: target/*.war must already exist. If project has no WAR, build it first (install Maven/JDK or use prebuilt WAR); check project packaging.
I. Build image:
   sudo docker build -t appdemo .
   sudo docker images
J. Run:
   sudo docker run -d --name app-demo -p 6060:8080 appdemo
   sudo docker ps
K. Browser:
   http://PUBLIC_IP:6060
   or http://PUBLIC_IP:6060/APP_CONTEXT/
   (Context path often matches WAR name.)
L. Troubleshooting:
   SSH timeout -> security group port 22/source
   key error -> correct .pem, username ubuntu, IP
   git/docker missing -> install commands above
   docker permission -> use sudo
   build COPY WAR error -> target/*.war missing/build project first
   container exited -> sudo docker ps -a; sudo docker logs app-demo
   name conflict -> sudo docker rm app-demo (if stopped)
   site inaccessible -> container/port mapping/security group 6060
   404 -> correct WAR/context path
   public IP changed -> use current IP (Elastic IP if persistent needed)

============================================================
GENERAL EXAM CHECKLIST / QUICK TROUBLESHOOTING
============================================================
1. Check software versions before starting.
2. Start Docker Desktop before Docker/Minikube commands.
3. cd to correct project folder; pwd + ls.
4. Git: status -> add -> commit -> push; verify GitHub.
5. Maven: run build and confirm BUILD SUCCESS; check target output.
6. Docker: check docker ps, docker ps -a, docker logs.
7. Docker build: Dockerfile exact name; dot at end of docker build command.
8. Compose: YAML spaces/indentation; run commands from compose file folder.
9. Jenkins: Save job configuration; check Console Output.
10. Pipeline: use correct repository URL, branch, project path and Jenkins tool name.
11. Kubernetes: wait for Pods Running; use describe/logs for errors; keep port-forward terminal open.
12. Webhook: ngrok running; correct HTTPS URL; webhook trigger checked.
13. EC2: instance running; SSH key correct; inbound port allowed; public IP correct.
14. Take screenshots of successful outputs before cleanup.
15. Do not remove containers/images or delete cluster until required screenshots/results are saved.

QUICK COMMAND MATCHER
Git:       git status -> git add . -> git commit -m "msg" -> git push
Maven:     mvn clean install
Docker:    docker pull IMAGE -> docker run -> docker ps
Dockerfile:docker build -t IMAGE . -> docker run
Compose:   docker compose up -d -> docker compose ps -> docker compose down
Jenkins:   New Item -> configure SCM/build/post-build -> Save -> Build Now -> Console
Pipeline:   New Pipeline -> script -> Save -> Build Now
Minikube:  minikube start -> kubectl create deployment -> get pods -> expose -> scale
Nagios:    docker pull/run -> localhost:8888 -> login -> inspect -> stop/remove
EC2:       launch -> SSH -> install -> clone -> Dockerfile -> build -> run -> Public IP:port

END OF CHEAT SHEET
