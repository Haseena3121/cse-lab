# Software Engineering Lab – Practical Cheat Sheet

**Windows laptop | Sequential steps | Commands + checks + common fixes**

> Commands are separated line-by-line so they are easy to identify and copy. Run commands in the indicated terminal/folder.

---

## Experiment 1 – Software Installation + SRS

### 1. Check installations
Open CMD or Git Bash and run each command separately:

```bash
git --version
java -version
mvn -version
docker --version
kubectl version --client
minikube version
```

### 2. If a command is not recognized
- Install the required tool.
- Add it to `PATH` if required.
- Close and reopen the terminal.
- Retry the version command.

### 3. SRS document sequence
1. Title / Introduction
2. Abstract: project purpose, problem solved, technologies used
3. Functional Requirements (what the system does)
4. Non-Functional Requirements (security, performance, reliability, usability)
5. Users / Actors
6. User workflows (steps performed)
7. Use cases (actors, actions, system boundary)
8. UML diagrams (if asked)

---

## Experiment 2 – Git Local + Remote

### A. Create GitHub repository
1. GitHub → Create **New repository**.
2. Choose Private/Public as required.
3. If pushing an existing project, create it **without a README**.

### B. Enter your project folder (Git Bash)

```bash
cd ~/OneDrive/Desktop/SE
pwd
ls
```

### C. Initialize and configure Git

```bash
git init
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### D. Add and commit files

```bash
git status
git add .
git status
git commit -m "Initial commit"
git branch -M main
```

### E. Add remote and push (replace the URL)

```bash
git remote add origin https://github.com/USERNAME/REPO.git
git remote -v
git push -u origin main
```

### F. Verify
Refresh the GitHub repository page. Confirm your files and folders are visible.

### G. Common Git commands

```bash
git clone URL
git status
git add .
git commit -m "message"
git log
git log --oneline
git diff
git branch
git branch name
git checkout name
git checkout -b name
git merge name
git branch -d name
git push origin main
git pull origin main
git fetch origin
git remote -v
git remote add origin URL
git remote set-url origin URL
git remote remove origin
git reset filename
git revert COMMIT_ID
touch file.txt
cat file.txt
ls
```

### H. Common errors and fixes
- **Not a git repo:** `cd` into the project folder; check `pwd` and `ls`.
- **Origin exists:** check `git remote -v`; update with:
  ```bash
  git remote set-url origin URL
  ```
- **src refspec main:** add and commit files, set branch name, then push:
  ```bash
  git add .
  git commit -m "Initial commit"
  git branch -M main
  git push -u origin main
  ```
- **Rejected / fetch first:** pull, resolve conflicts, commit, then push:
  ```bash
  git pull origin main --allow-unrelated-histories
  git add .
  git commit -m "Merge remote changes"
  git push
  ```
- **Authentication failed:** use browser sign-in or a Personal Access Token (PAT), not your normal GitHub password.

---

## Experiment 3 – Collaboration, Branches, Conflict, Patch

### A. Create GitHub Organization (if asked)
GitHub profile/menu → Organizations → Create → enter organization name → choose free plan → add collaborator.

### B. Share repository
Repository → Settings → Collaborators / Manage Access → invite partner. Partner must **accept** the invitation.

### C. Partner clones the repository

```bash
git config --global user.name "Partner Name"
git config --global user.email "partner@example.com"
git clone https://github.com/OWNER/REPO.git
cd REPO
git remote -v
```

### D. Create branch, edit, commit, push

```bash
git checkout -b feature1
```

Edit or create a file, then:

```bash
git add .
git commit -m "Added feature"
git push origin feature1
```

### E. Update from main

```bash
git pull origin main
```

### F. Conflict practice
Create a branch and edit the same file:

```bash
git checkout -b branch1
```

Edit the file, then:

```bash
git add .
git commit -m "branch change"
git checkout main
```

Edit the **same lines** differently on main, then:

```bash
git add .
git commit -m "main change"
git merge branch1
```

If a conflict occurs:
1. Open the conflicted file.
2. Remove the `<<<<<<<`, `=======`, and `>>>>>>>` markers.
3. Keep the required text and save.

Then:

```bash
git add .
git commit -m "Resolved merge conflict"
```

### G. Patch file

```bash
git diff > changes.patch
cat changes.patch
git apply changes.patch
git status
```

### H. Common errors
- **Conflict:** edit the file, remove conflict markers, add, and commit.
- **Permission denied:** check that invitation was accepted and write access exists.
- **Patch will not apply:** use the correct folder and matching file version.

---

## Experiment 4 – Maven Java + Web Project

### A. Create Maven Java project in Eclipse
1. Eclipse → File → New → Project → Maven → Maven Project → Next.
2. Select `maven-archetype-quickstart` → Next.
3. Group ID: `com.example`
4. Artifact ID: `myapp`
5. Finish.

### B. Maven commands (run in project folder)

```bash
mvn -version
mvn clean
mvn compile
mvn test
mvn package
mvn install
mvn clean install
```

Check the `target/` folder. Expected result: **BUILD SUCCESS**.

### C. Maven command meanings
- `clean` = remove old `target` output
- `compile` = compile source code
- `test` = run tests
- `package` = create JAR/WAR
- `install` = place artifact in local Maven repository

### D. Add a dependency
1. Open `pom.xml`.
2. Add the dependency inside the `<dependencies>` section.
3. Save.
4. Right-click project → Maven → Update Project.
5. Run:

```bash
mvn clean install
```

Example Gson dependency coordinates: `com.google.code.gson:gson:2.10`.

### E. Create Maven Web project
Eclipse → File → New → Maven Project → select `maven-archetype-webapp` → enter Group/Artifact IDs → Finish.

Typical structure:
- `pom.xml`
- `src/main/webapp`
- `index.jsp`
- `WEB-INF`

### F. Servlet dependency
Use the dependency matching the project/Tomcat version. Example coordinates:

- Group: `javax.servlet`
- Artifact: `javax.servlet-api`
- Version: `4.0.1`
- Scope: `provided`

### G. Build WAR

```bash
mvn clean install
```

Check `target/` for the `.war` file.

### H. Deploy using Tomcat in Eclipse
1. Eclipse → Servers view → New Server.
2. Choose Apache Tomcat.
3. Select the Tomcat installation folder.
4. Add the project to the server.
5. Start the server.
6. Open:
   http://localhost:8080/APPNAME/


### J. Docker CLI with Redis (manual exercise)
**Step 1 – Pull Redis**

```bash
docker pull redis
```

**Step 2 – Run Redis container**

```bash
docker run --name my-redis -d redis
```

**Step 3 – Verify it is running**

```bash
docker ps
```

**Step 4 – Open Redis CLI inside container**

```bash
docker exec -it my-redis redis-cli
```

At the Redis prompt, run one command at a time:

```redis
SET name "Alice"
GET name
```
Expected: `OK`, then `"Alice"`.

Exit Redis CLI:

```redis
exit
```

**Step 5 – Stop and restart**

```bash
docker stop my-redis
docker start my-redis
```

**Step 6 – Remove container and image (only when asked/finished)**

```bash
docker rm my-redis
docker rmi redis
```
Container must be stopped before `docker rm`.

### I. Common errors
- **BUILD FAILURE:** read the first error in Console.
- **Dependency missing:** verify dependency coordinates and internet connection.
- **JAVA_HOME:** set JAVA_HOME to the installed JDK.
- **Red marks:** Right-click project → Maven → Update Project.
- **Port 8080 busy:** stop the other service or change the port.
- **WAR absent:** run `mvn package`.

---


### J. Dependency and plugin version conflicts (scenario question)
1. Open `pom.xml` and identify the dependency causing the conflict.
2. Check dependency tree:

```bash
mvn dependency:tree
```

3. If two versions are pulled transitively, use a direct dependency version or an exclusion. Example exclusion pattern:

```xml
<dependency>
  <groupId>example.parent</groupId>
  <artifactId>example-library</artifactId>
  <version>1.0</version>
  <exclusions>
    <exclusion>
      <groupId>example.conflicting</groupId>
      <artifactId>conflicting-library</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

4. For shared version control, use `<dependencyManagement>` in the parent POM and declare the chosen version there.
5. Save, update Maven project in Eclipse, and run:

```bash
mvn clean install
```

Expected: `BUILD SUCCESS` if the conflict is resolved.

### K. Compiler plugin (set Java version)
Add inside `<build><plugins>` in `pom.xml`:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
  <configuration>
    <source>17</source>
    <target>17</target>
  </configuration>
</plugin>
```

Then:

```bash
mvn package
jar tf target/myapp.jar
```

### L. Maven multi-module project (Parent + JAR + WAR child)
**Project structure:**

```text
MultiModule/
├── pom.xml                 (parent, packaging pom)
├── MultiModuleChild1/      (Child 1, JAR)
│   └── pom.xml
└── module2/                 (Child 2, WAR; depends on Child 1)
    └── pom.xml
```

**Create parent:** Eclipse → File → New → Other → Maven → Maven Project → check *Create a simple project* → Group ID `KMIT-5F6` → Artifact ID `MultiModule` → Packaging `pom` → Finish.

**Create Child 1:** Right-click parent → New → Maven Module → simple project → Artifact ID `MultiModuleChild1` → Packaging `jar` → Finish.

**Create Child 2:** Right-click parent → New → Maven Module → Artifact ID `module2` → choose `maven-archetype-webapp` → Finish.

Parent `pom.xml` key sections (keep generated XML header/project wrapper):

```xml
<groupId>KMIT-5F6</groupId>
<artifactId>MultiModule</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>pom</packaging>
<modules>
  <module>MultiModuleChild1</module>
  <module>module2</module>
</modules>
```

Child 1 `pom.xml` key sections:

```xml
<parent>
  <groupId>KMIT-5F6</groupId>
  <artifactId>MultiModule</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>MultiModuleChild1</artifactId>
<packaging>jar</packaging>
<properties>
  <maven.compiler.source>8</maven.compiler.source>
  <maven.compiler.target>8</maven.compiler.target>
</properties>
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.1</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

Child 2 `module2/pom.xml` key sections:

```xml
<parent>
  <groupId>KMIT-5F6</groupId>
  <artifactId>MultiModule</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</parent>
<artifactId>module2</artifactId>
<packaging>war</packaging>
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.1</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>KMIT-5F6</groupId>
    <artifactId>MultiModuleChild1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </dependency>
</dependencies>
<build>
  <finalName>module2</finalName>
  <plugins>
    <plugin>
      <artifactId>maven-war-plugin</artifactId>
      <version>3.4.0</version>
    </plugin>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.13.0</version>
    </plugin>
  </plugins>
</build>
```

**Build the whole parent project:** Right-click `MultiModule` → Run As → Maven build → Goals `clean install` → Run.

```bash
mvn clean install
```

Expected: `BUILD SUCCESS`; WAR at `module2/target/module2.war`; Child 1 JAR under `MultiModuleChild1/target/`.

---

## Experiment 5 – Docker CLI

### A. Start Docker
Open Docker Desktop and wait until the engine is running.

### B. Verify

```bash
docker --version
docker info
```

### C. Pull Nginx and list images

```bash
docker pull nginx
docker images
```

### D. Run Nginx container

```bash
docker run -d --name mynginx -p 8080:80 nginx
docker ps
docker ps -a
```

Open in browser: http://localhost:8080 (Nginx welcome page).

### E. Container commands

```bash
docker stop mynginx
docker start mynginx
docker restart mynginx
docker logs mynginx
docker inspect mynginx
docker stats
docker exec -it mynginx bash
```

If you enter the container shell, exit with:

```bash
exit
```

Remove container (stop it first if it is running):

```bash
docker rm mynginx
```

### F. Remove image
Remove containers using the image first, then:

```bash
docker rmi nginx
```

### G. Network commands

```bash
docker network create mynetwork
docker network ls
docker network connect mynetwork mynginx
docker network disconnect mynetwork mynginx
docker network rm mynetwork
```

Disconnect/remove attached containers before removing a network.

### H. Volume commands

```bash
docker volume create myvolume
docker volume ls
docker volume inspect myvolume
docker volume rm myvolume
```

Remove a volume only when it is not in use.

### I. Common errors
- **Docker daemon error:** start Docker Desktop.
- **Port busy:** try `-p 8081:80`.
- **Name exists:** run `docker ps -a`, then remove the old stopped container.
- **Container exits:** check `docker logs`.
- **Exec error:** the container must be running.

---

## Experiment 6 – Dockerfile, Build, Run, Docker Hub

### A. Create folder and enter it

```bash
mkdir redis
cd redis
```

### B. Create `Dockerfile`
The filename must be exactly `Dockerfile` (not `Dockerfile.txt`).

Contents:

```dockerfile
FROM redis:latest
CMD ["redis-server"]
```

### C. Build image (the dot is required)

```bash
docker build -t redisnew .
docker images
```

### D. Run container and verify

```bash
docker run -d --name myredisnew redisnew
docker ps -a
docker inspect myredisnew
```

### E. Stop and start

```bash
docker stop myredisnew
docker start myredisnew
```

### F. Commit container to another image

```bash
docker commit myredisnew redisnewbackup
docker images
```

### G. Remove

```bash
docker rm myredisnew
docker rmi redisnewbackup
```

Stop the container before removing it.

### H. Docker Hub
Login:

```bash
docker login
```

Tag image (replace username):

```bash
docker tag redisnew YOUR_USERNAME/redisnew:latest
```

Push:

```bash
docker push YOUR_USERNAME/redisnew:latest
```

On another machine, pull and run:

```bash
docker pull YOUR_USERNAME/redisnew:latest
docker run -d --name myredis YOUR_USERNAME/redisnew:latest
```

### I. Common errors
- **Dockerfile missing:** check exact filename and current folder.
- **Build error:** inspect Dockerfile.
- **Invalid image name:** use lowercase.
- **Name exists:** remove the old container.
- **Push denied:** login and verify the tag/username.

---

## Experiment 7 – Docker Compose

### A. Create folder

```bash
mkdir compose-lab
cd compose-lab
```

### B. Create `docker-compose.yml`
Use spaces, not tabs. Paste this YAML exactly:

```yaml
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
```

### C. Start services

```bash
docker compose up -d
```

### D. Verify and access

```bash
docker compose ps
docker compose logs
docker compose logs -f
```

Open browser: http://localhost:8080

Press `Ctrl+C` to stop following live logs.

### E. Stop, start, restart, remove

```bash
docker compose stop
docker compose start
docker compose restart
docker compose down
```

### F. Add Redis service
Add this inside `services:` in the compose file:

```yaml
  redis:
    image: redis:alpine
```

Then run:

```bash
docker compose up -d
docker compose ps
```

### G. Network + persistent volume example
Replace the compose file contents with this version if the task asks for a network and volume:

```yaml
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
```

### H. Run and clean up

```bash
docker compose up -d
docker compose ps
docker compose down
```


### J. Redis cache + `depends_on` (manual Task 2)
In `docker-compose.yml`, add Redis under `services:` and add dependency under web:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - redis
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
  redis:
    image: redis:alpine
```

Restart and verify:

```bash
docker compose up -d
docker compose ps
```
Expected: web, db, redis listed/running.

### K. Transfer Compose setup to another machine
1. Zip the `compose-lab` folder.
2. Transfer and extract it on a machine with Docker Compose.
3. Open terminal in that folder.

```bash
docker compose up -d
docker compose ps
```

Expected: the same services start without changing the Compose file.

### L. Custom network + persistent volume (manual Task 4)
Replace the Compose file with:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - app-net
    depends_on:
      - db
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
```

Run and test persistence:

```bash
docker compose up -d
docker compose ps
docker compose down
docker compose up -d
```
Expected: database data in named volume persists after containers are recreated; services communicate over `app-net` using service names.

### M. Flask app + Docker Compose for quick development (manual Task 5)
Create `app.py` in `compose-lab`:

```python
from flask import Flask
app = Flask(__name__)
@app.route("/")
def home():
    return "Hello from Flask + Docker!"
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Create a `Dockerfile` in the same folder:

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```

Add/replace the `web` service in Compose with:

```yaml
web:
  build: .
  ports:
    - "5000:5000"
  depends_on:
    - db
```

Run:

```bash
docker compose up --build
```
Open `http://localhost:5000`.
Change the return message in `app.py`, save, then rebuild:

```bash
docker compose up --build
```
Refresh browser and confirm the changed text.

### I. Common errors
- **YAML indentation:** align spaces carefully.
- **Port busy:** change the host port, for example `8081:80`.
- **Wrong folder:** `cd` to the folder containing the compose file.
- **Database connection:** use service name `db` as hostname from another Compose service.
- **Data loss:** use a named volume for persistent data.

---

## Experiment 8 – Jenkins Freestyle Maven Jobs

### A. Open Jenkins
Open http://localhost:8080 and log in.

### B. Create Java build job
1. Dashboard → New Item.
2. Name: `MavenJava_Build`.
3. Select Freestyle Project → OK.
4. Source Code Management → Git → Repository URL.
5. Branch: `*/main` (or `*/master`).
6. Build Steps → Invoke top-level Maven targets.
7. Select configured Maven tool.
8. Goals: `clean`.
9. Add another Maven build step with Goals: `install`.
10. Post-build Actions → Archive artifacts: `*/*`.
11. Post-build Actions → Build other projects: `MavenJava_Test` (only if stable).
12. Apply → Save.

### C. Create test job
1. New Item → `MavenJava_Test` → Freestyle.
2. Build Environment → Delete workspace before build.
3. Build Step → Copy artifacts from another project.
4. Project name: `MavenJava_Build`.
5. Artifacts: `*/*`.
6. Build Step → Invoke Maven → Goals: `test`.
7. Post-build → Archive artifacts: `*/*`.
8. Save.

### D. Pipeline View (if asked)
Install Pipeline/Build Pipeline View plugin if needed:
1. Dashboard → add view (+).
2. Name: `MavenJava_Pipeline`.
3. Choose Build Pipeline View.
4. Initial job: `MavenJava_Build`.
5. Save.

### E. Maven Web Build → Test → Deploy to Tomcat (important)
Create/configure these **three Freestyle jobs** in sequence.

#### Job 1: `MavenWeb_Build`
1. Dashboard → New Item → `MavenWeb_Build` → Freestyle → OK.
2. Description: `Web Build demo`.
3. Source Code Management → Git → paste GitMavenWeb repository URL.
4. Branch: `*/main` (or `*/master`, matching repository).
5. Build Steps → Invoke top-level Maven targets → Maven tool `MAVEN_HOME` → Goals `clean`.
6. Add another Maven build step → Goals `install`.
7. Post-build → Archive artifacts → `**/*`.
8. Post-build → Build other projects → `MavenWeb_Test` → Only if build is stable.
9. Apply → Save.

#### Job 2: `MavenWeb_Test`
1. New Item → `MavenWeb_Test` → Freestyle → OK.
2. Description: `Test demo`.
3. Build Environment → check Delete workspace before build starts.
4. Build Step → Copy artifacts from another project:
   - Project: `MavenWeb_Build`
   - Build: Stable build only
   - Artifacts: `**/*`
5. Add Build Step → Invoke top-level Maven targets → Maven tool `MAVEN_HOME` → Goals `test`.
6. Post-build → Archive artifacts → `**/*`.
7. Post-build → Build other projects → `MavenWeb_Deploy`.
8. Apply → Save.

#### Job 3: `MavenWeb_Deploy`
1. New Item → `MavenWeb_Deploy` → Freestyle → OK.
2. Description: `Web Code Deployment`.
3. Build Environment → Delete workspace before build starts.
4. Build Step → Copy artifacts from another project:
   - Project: `MavenWeb_Test`
   - Build: Stable build only
   - Artifacts: `**/*`
5. Post-build Actions → Deploy WAR/EAR to a container.
6. Configure deployment:
   - WAR/EAR file: `**/*.war`
   - Context path: `Webpath`
   - Add container: Tomcat 9.x Remote
   - Credentials: Tomcat username/password configured on Tomcat server
   - Tomcat URL: manual example `http://localhost:8085/` (use your configured Tomcat URL/port)
7. Apply → Save.

#### Tomcat setup required for deployment
1. Open Tomcat's `conf/tomcat-users.xml`.
2. Add a deploy role and user inside the `<tomcat-users>` element (use credentials you will enter in Jenkins):

```xml
<role rolename="manager-script"/>
<user username="admin" password="YOUR_PASSWORD" roles="manager-script"/>
```

3. Save and restart Tomcat.
4. Confirm Tomcat is running and the Manager deployment access is configured.
5. In Jenkins, use the matching saved credentials; don't type real credentials into a public README.

#### Pipeline View for Maven Web
1. Dashboard → click `+` → name `MavenWeb_Pipeline`.
2. Choose Build Pipeline View.
3. Layout: Based on upstream/downstream relationship.
4. Initial job: `MavenWeb_Build`.
5. Apply → Save.

#### Run and verify deployment
1. Start/run `MavenWeb_Build` (Build Now/Run).
2. Check Console Output for success (green status).
3. Confirm test job triggers; check its Console Output.
4. Confirm deploy job triggers; check its Console Output.
5. Open Tomcat homepage/Manager application.
6. Select `/Webpath` (context path configured above) and verify the deployed application.


### F. Run and verify
Click **Build Now** → open build number → **Console Output**. Green status indicates success.

### G. Common errors
- **Git checkout:** check URL, branch, credentials.
- **Maven not found:** Manage Jenkins → Tools.
- **Artifact missing:** check archive pattern.
- **Downstream not triggered:** check job name and trigger setting.
- **Tomcat deploy failure:** check URL, credentials, Manager, port.
- **View empty:** check plugin and upstream job settings.

---

## Experiment 9 – Jenkins Pipeline Script + Triggers

### A. Create Pipeline job
Dashboard → New Item → `MavenScriptPipeline` → Pipeline → OK.

### B. Pipeline script
Pipeline section → Definition: Pipeline script. Paste the script below, replacing the repository URL, project folder, and Maven tool name as required.

```groovy
pipeline {
    agent any

    tools {
        maven 'MAVEN-HOME'
    }

    stages {
        stage('Git Clone and Clean') {
            steps {
                bat 'git clone YOUR_GITHUB_REPOSITORY_URL'
                bat 'mvn clean -f mavenjava'
            }
        }

        stage('Install') {
            steps {
                bat 'mvn install -f mavenjava'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test -f mavenjava'
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package -f mavenjava'
            }
        }
    }
}
```

### C. Replace values
- Repository URL: your GitHub repository URL.
- Project folder: `mavenjava` (change if your cloned folder is different).
- Maven tool: match the name configured in Jenkins Tools.

### D. Save and run
Apply → Save → Build Now → open build number → Console Output. Verify all stages pass.

### E. Build triggers
Job → Configure → Build Triggers. Available options may include:
- Build periodically
- Poll SCM
- GitHub hook trigger
- Build after other projects
- Remote trigger

### F. Schedule examples
Poll SCM, approximately every 5 minutes:

```text
H/5 * * * *
```

Daily around 9:00:

```text
H 9 * * *
```


### H. Add a Tomcat deployment stage after Package (scenario)
If the question asks to deploy to Tomcat after packaging, add a `Deploy` stage **after** `Package`. For a manual deployment task, use the configured Tomcat deployment plugin or secure credentials; do not put passwords directly in the Jenkinsfile.


```groovy
stage('Deploy') {
    steps {
        echo 'Deploy packaged WAR to Tomcat here'
        // Deployment step/plugin configuration goes here
    }
}
```

### I. Pipeline scenario quick answers
- **Clean workspace:** `clean` stage / `mvn clean`.
- **Missing GitHub URL:** add it in the Clone stage's `git`/`git clone` command.
- **Maven not configured:** the `tools { maven 'MAVEN-HOME' }` declaration fails; configure matching name in Manage Jenkins → Tools.
- **No WAR:** `Package` stage runs `mvn package`.
- **Test failure:** `mvn test` returns non-zero; pipeline normally stops and later stages don't run.
- **GitHub-change trigger:** configure job Build Triggers or pipeline `triggers` (e.g., Poll SCM).
- **Different project folder:** update every Maven path reference to the correct folder.
- **Install fails:** subsequent stages won't execute unless pipeline is explicitly configured to catch/ignore failure.
- **Secure private repo:** store credentials in Jenkins Credentials Manager and reference credential ID; never hardcode credentials.
- **Pipeline plugin:** required for pipeline syntax/jobs.
- **Linux agent:** use `sh` instead of Windows `bat`.
- **Clone failure:** command error stops stage/pipeline by default.
- **Welcome message:** use `echo 'Welcome to Jenkins Pipeline!'`.
- **Build number:** use `echo "Build Number: ${BUILD_NUMBER}"`.
- **Checkout latest code:** use `git branch: 'main', url: 'REPOSITORY_URL'` in Checkout stage.
- **Compile Java:** add stage with `bat 'javac Hello.java'` on Windows.
- **Scheduled evening build at 6 PM:** `cron('0 18 * * *')`.
- **Deploy after package:** add Deploy stage immediately after Package.

### G. Common errors
- **Maven tool name mismatch:** check Jenkins Tools.
- **Clone URL wrong:** correct repository URL.
- **Folder incorrect:** fix the `-f` project path.
- **`bat` only works on Windows:** on Linux use `sh`.
- **Syntax error:** check braces and quotes.
- **Agent unavailable:** check Jenkins agent.

---

## Experiment 10 – Minikube/Kubernetes + Nagios

### Part A – Minikube / Kubernetes

#### A. Docker Desktop and version checks
Make sure Docker Desktop is running, then:

```bash
docker --version
minikube version
kubectl version --client
```

#### B. Start cluster

```bash
minikube start --driver=docker
minikube status
```

#### C. Create Nginx deployment

```bash
kubectl create deployment mynginx --image=nginx
kubectl get deployments
kubectl get pods
```

Wait until the Pod is `Running`, then:

```bash
kubectl describe pods
```

#### D. Expose deployment

```bash
kubectl expose deployment mynginx --type=NodePort --port=80 --target-port=80
kubectl get services
```

#### E. Scale to 4 replicas

```bash
kubectl scale deployment mynginx --replicas=4
kubectl get deployments
kubectl get pods
```

Verify 4 Pods are shown.

#### F. Access option 1 – Port forward
Keep this terminal open while using the browser:

```bash
kubectl port-forward svc/mynginx 8081:80
```

Open: http://localhost:8081

Stop port-forwarding with `Ctrl+C`.

#### G. Access option 2 – Minikube service

```bash
minikube service mynginx
```

Or:

```bash
minikube service mynginx --url
```

#### H. Dashboard

```bash
minikube dashboard
```

#### I. Cleanup

```bash
kubectl delete deployment mynginx
kubectl delete service mynginx
kubectl get deployments
minikube stop
minikube delete
```

`minikube stop` and `minikube delete` are optional. `minikube delete` removes the cluster.

#### J. Kubernetes troubleshooting
- **Minikube won't start:** check `minikube status` and confirm Docker Desktop is running.
- **Pod Pending:** run `kubectl describe pod POD`.
- **ImagePullBackOff:** check image name and internet connection.
- **CrashLoopBackOff:** run `kubectl logs POD`.
- **Service issue:** run `kubectl get svc`.
- **Pod not ready:** wait and run `kubectl get pods` again.
- **Wrong replicas:** run:
  ```bash
  kubectl scale deployment mynginx --replicas=4
  ```

### Part B – Nagios Docker

#### A. Pull image

```bash
docker pull jasonrivers/nagios:latest
```

#### B. Run container

```bash
docker run -d --name nagiosdemo -p 8888:80 jasonrivers/nagios:latest
```

#### C. Verify and open

```bash
docker ps
```

Open: http://localhost:8888

Login (if the image's default credentials have not changed):
- Username: `nagiosadmin`
- Password: `nagios`

#### D. Explore
Explore hosts, services, alerts, and status pages.

#### E. Logs and cleanup

```bash
docker logs nagiosdemo
docker logs -f nagiosdemo
docker stop nagiosdemo
docker rm nagiosdemo
docker rmi jasonrivers/nagios:latest
```

Image removal is optional.

#### F. Common errors
- **Port conflict:** map another host port, e.g. `-p 8889:80`, and open http://localhost:8889.
- **Container exits:** inspect logs.
- **Name exists:** remove the old stopped container.
- **Not ready:** allow initialization time.

---

## Experiment 11 – Jenkins CI/CD Webhook + Email

### Part A – GitHub Webhook

#### A. Jenkins
Ensure Jenkins is running at http://localhost:8080.

#### B. Plugin
Install/configure GitHub integration/plugin if required.

#### C. Start ngrok
In a separate terminal:

```bash
ngrok http 8080
```

Copy its HTTPS forwarding URL. **Keep ngrok running.**

#### D. Add webhook in GitHub
Repository → Settings → Webhooks → Add webhook:
- Payload URL: `https://YOUR-NGROK-URL/github-webhook/`
- Content type: `application/json`
- Event: Just the push event
- Click Add webhook.

#### E. Enable trigger in Jenkins
Jenkins job → Configure → Build Triggers → check **GitHub hook trigger for GITScm polling** → Save.

#### F. Test by pushing

```bash
git add .
git commit -m "Webhook test"
git push origin main
```

#### G. Verify
Check whether Jenkins automatically starts a build.

#### H. Webhook troubleshooting
- **No trigger:** check checkbox, plugin, and job SCM.
- **Webhook failed:** check URL and payload settings.
- **Localhost not reachable externally:** use ngrok.
- **ngrok URL changed:** update the webhook URL in GitHub.
- **Build fails:** open Console Output.

### Part B – Email notification

#### A. Gmail setup
Enable 2-Step Verification → create an App Password if available → keep the App Password private.

#### B. Jenkins plugin
Jenkins → Manage Plugins → install **Email Extension Plugin** → restart if requested.

#### C. Configure SMTP
Jenkins → Manage Jenkins → Configure System → E-mail Notification:
- SMTP server: `smtp.gmail.com`
- Authentication: enabled
- Username: your Gmail address
- Password: Gmail App Password
- SSL: enabled
- Port: `465`
- Reply-To: Gmail address
- Send test email and verify it arrives.

#### D. Configure job email
Job → Configure → Post-build Actions → Editable Email Notification:
- Recipient list
- Content type
- Triggers (success/failure)
- Save

#### E. Common errors
- **Authentication:** use Gmail App Password.
- **Email missing:** check spam and recipient.
- **SMTP error:** verify host, port, and network.
- **Plugin missing:** install plugin.

#### F. UML quick definitions
- **Class diagram:** classes, attributes, methods, relationships.
- **Sequence diagram:** interactions over time.
- **Component diagram:** software modules and connections.

---

## Experiment 12 – Ubuntu VM / AWS EC2 Web Deployment

### A. Push project to GitHub (skip if already pushed)

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin main
```

### B. Launch EC2
AWS Console → EC2 → Launch Instance:
1. Name: `ubuntu`
2. Choose Ubuntu Server AMI.
3. Select eligible instance type (example in manual: `t2.micro`).
4. Create/download `.pem` key pair; keep it safe.
5. Security group inbound rules:
   - SSH 22 (restrict source to your own IP)
   - HTTP 80
   - HTTPS 443
   - Custom TCP 6060
6. Storage example: 8 GB.
7. Launch instance.

### C. Wait and copy IP
Wait for instance state **Running** and status checks passed. Copy Public IPv4.

### D. Connect to instance
AWS EC2 → select instance → Connect → EC2 Instance Connect (if available).

Or, in Git Bash in the `.pem` folder:

```bash
ssh -i AWS.pem ubuntu@PUBLIC_IP
```

Type `yes` if the first-connection prompt appears.

### E. Install tools on Ubuntu

```bash
sudo apt update
sudo apt-get install docker.io -y
sudo apt install git -y
sudo apt install nano -y
sudo systemctl start docker
sudo systemctl status docker
docker --version
```

Press `q` to exit the status view.

### F. Clone repository

```bash
git clone https://github.com/USERNAME/REPO.git
cd REPO
ls
```

### G. Create Dockerfile (Tomcat 9 WAR example)
Create/edit Dockerfile:

```bash
nano Dockerfile
```

Paste:

```dockerfile
FROM tomcat:9.0
COPY target/*.war /usr/local/tomcat/webapps/
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

Save in Nano: `Ctrl+O` → Enter → `Ctrl+X`.

### H. Important WAR requirement
`target/*.war` must already exist. If the project does not have a WAR, build it first (install Maven/JDK or use a prebuilt WAR). Check project packaging.

### I. Build image

```bash
sudo docker build -t appdemo .
sudo docker images
```

### J. Run container

```bash
sudo docker run -d --name app-demo -p 6060:8080 appdemo
sudo docker ps
```

### K. Open website
Open:

- `http://PUBLIC_IP:6060`
- `http://PUBLIC_IP:6060/APP_CONTEXT/` (context path often matches the WAR filename)

### L. Common errors
- **SSH timeout:** check security group port 22 and source IP.
- **Key error:** correct `.pem`, username `ubuntu`, and IP.
- **Git/Docker missing:** run the installation commands.
- **Docker permission:** use `sudo`.
- **COPY WAR error:** `target/*.war` is missing; build the project first.
- **Container exited:** check:
  ```bash
  sudo docker ps -a
  sudo docker logs app-demo
  ```
- **Name conflict:** if stopped, remove it:
  ```bash
  sudo docker rm app-demo
  ```
- **Site inaccessible:** check container, port mapping, and security group port 6060.
- **404:** use the correct WAR/context path.
- **Public IP changed:** use the current IP (Elastic IP is needed for a persistent IP).

---

## General Exam Checklist / Quick Troubleshooting

1. Check software versions before starting.
2. Start Docker Desktop before Docker/Minikube commands.
3. Enter the correct project folder; use `pwd` and `ls`.
4. Git: `status` → `add` → `commit` → `push`; verify on GitHub.
5. Maven: confirm `BUILD SUCCESS`; check `target/`.
6. Docker: use `docker ps`, `docker ps -a`, and `docker logs`.
7. Docker build: filename must be exactly `Dockerfile`; include the dot in `docker build`.
8. Compose: YAML spaces/indentation; run from the compose-file folder.
9. Jenkins: save job configuration; check Console Output.
10. Pipeline: correct repo URL, branch, project path, Jenkins tool name.
11. Kubernetes: wait for Pods to be Running; use `describe`/`logs`; keep port-forward terminal open.
12. Webhook: ngrok running; correct HTTPS URL; trigger checked.
13. EC2: instance running; SSH key correct; inbound ports allowed; public IP correct.
14. Take screenshots of successful outputs before cleanup.
15. Do not remove containers/images or delete cluster until screenshots/results are saved.

---

## Quick Command Matcher

| Task | Command sequence / action |
|---|---|
| Git | `git status` → `git add .` → `git commit -m "msg"` → `git push` |
| Maven | `mvn clean install` |
| Docker | `docker pull IMAGE` → `docker run` → `docker ps` |
| Dockerfile | `docker build -t IMAGE .` → `docker run` |
| Compose | `docker compose up -d` → `docker compose ps` → `docker compose down` |
| Jenkins | New Item → configure SCM/build/post-build → Save → Build Now |
| Pipeline | New Pipeline → script → Save → Build Now |
| Minikube | `minikube start` → `kubectl create deployment` → `get pods` → `expose` → `scale` |
| Nagios | `docker pull/run` → `localhost:8888` → login → inspect → stop/remove |
| EC2 | Launch → SSH → install → clone → Dockerfile → build → run → Public IP:port |

---

**End of Cheat Sheet**

---

# EXAM TASKS YOU MAY BE GIVEN (UI-BASED, DO IN THIS ORDER)

## Experiment 8 – Jenkins Freestyle Maven Java Build → Test

### Task A: MavenJava_Build job
1. Open `http://localhost:8080` → **New Item**.
2. Name: `MavenJava_Build` → **Freestyle project** → OK.
3. **Source Code Management** → Git → Repository URL: paste Maven Java GitHub URL.
4. Branch: `*/main` (or `*/master`, as repo uses).
5. **Build Steps** → Add Build Step → **Invoke top-level Maven targets**.
6. Maven Version: select configured `MAVEN_HOME`.
7. Goals: `clean`.
8. Add a second **Invoke top-level Maven targets** build step → Goals: `install`.
9. **Post-build Actions** → Archive the artifacts → Files: `**/*`.
10. Add **Build other projects** → `MavenJava_Test` → trigger only if build is stable.
11. Apply → Save → Build Now → Console Output.

### Task B: MavenJava_Test job
1. New Item → `MavenJava_Test` → Freestyle → OK.
2. Build Environment → check **Delete workspace before build starts**.
3. Build Steps → **Copy artifacts from another project**.
   - Project: `MavenJava_Build`
   - Build: Stable build only
   - Artifacts: `**/*`
4. Add Build Step → **Invoke top-level Maven targets**.
   - Maven Version: `MAVEN_HOME`
   - Goals: `test`
5. Post-build → Archive artifacts → `**/*`.
6. Apply → Save.
7. Run `MavenJava_Build`; verify it triggers `MavenJava_Test` and both are green.

### Task C: Create MavenJava Pipeline View
1. Dashboard → click `+` next to All.
2. View name: `MavenJava_Pipeline`.
3. Select **Build Pipeline View** → Create.
4. Layout: Based on upstream/downstream relationship.
5. Initial job: `MavenJava_Build`.
6. Apply → Save → click Run/trigger and inspect Console Output.

---

## Experiment 8 – Jenkins Maven Web Build → Test → Deploy to Tomcat

### Task A: MavenWeb_Build
1. New Item → `MavenWeb_Build` → Freestyle → OK.
2. Description: `Web Build demo`.
3. Source Code Management → Git → paste Maven Web repo URL.
4. Branch: `*/main` or `*/master`.
5. Build Steps → Add Build Step → **Invoke top-level Maven targets**.
   - Maven: `MAVEN_HOME`
   - Goals: `clean`
6. Add another **Invoke top-level Maven targets**.
   - Maven: `MAVEN_HOME`
   - Goals: `install`
7. Post-build → **Archive the artifacts** → Files: `**/*`.
8. Post-build → **Build other projects** → `MavenWeb_Test` → Only if build is stable.
9. Apply → Save.

### Task B: MavenWeb_Test
1. New Item → `MavenWeb_Test` → Freestyle → OK.
2. Build Environment → **Delete workspace before build starts**.
3. Build Steps → **Copy artifacts from another project**.
   - Project: `MavenWeb_Build`
   - Build: Stable build only
   - Artifacts: `**/*`
4. Add Build Step → **Invoke top-level Maven targets**.
   - Maven: `MAVEN_HOME`
   - Goals: `test`
5. Post-build → Archive artifacts → `**/*`.
6. Post-build → Build other projects → `MavenWeb_Deploy`.
7. Apply → Save.

### Task C: MavenWeb_Deploy (Tomcat)
1. New Item → `MavenWeb_Deploy` → Freestyle → OK.
2. Build Environment → **Delete workspace before build starts**.
3. Build Steps → **Copy artifacts from another project**.
   - Project: `MavenWeb_Test`
   - Build: Stable build only
   - Artifacts: `**/*`
4. Post-build Actions → **Deploy WAR/EAR to a container**.
5. WAR/EAR File: `**/*.war`.
6. Context path: `Webpath` (or task-specified name).
7. Add container → **Tomcat 9.x Remote**.
8. Add/select Tomcat credentials (username/password must match Tomcat configuration).
9. Tomcat URL: use the Tomcat server address and port configured for the lab (manual example: `http://localhost:8085/`).
10. Apply → Save.
11. Run `MavenWeb_Build` and check all 3 jobs turn green.
12. Open Tomcat homepage/Manager → click `/webpath` → log in if prompted → open deployed app.

### Tomcat deployment prerequisites (if Jenkins deploy fails)
1. Start Tomcat and verify Tomcat homepage opens at its configured port.
2. In Tomcat `conf/tomcat-users.xml`, inside `<tomcat-users>`, ensure a deployment user/role is configured. Example (use your own lab credentials):

```xml
<role rolename="manager-script"/>
<user username="admin" password="YOUR_PASSWORD" roles="manager-script"/>
```

3. Save and restart Tomcat.
4. In Jenkins, use matching credentials and the correct Tomcat URL/port.
5. Check the Deploy job's Console Output for errors.

### Jenkins UI questions/tasks that may be asked
- **Which Build Step?** `Invoke top-level Maven targets`.
- **Where to type `clean`?** Build Steps → Invoke top-level Maven targets → Goals.
- **Where to type `install`?** Add another Invoke top-level Maven targets step → Goals `install`.
- **Where to run tests?** Test job → Invoke top-level Maven targets → Goals `test`.
- **Where to copy `target` output/artifacts?** Copy artifacts from another project → Project name + Stable build + artifact pattern (manual uses `**/*`).
- **Where to archive?** Post-build Actions → Archive the artifacts → `**/*`.
- **Where to deploy WAR to Tomcat?** Deploy job → Post-build Actions → Deploy WAR/EAR to a container → `**/*.war`.
- **Where to set Tomcat?** Add container → Tomcat 9.x Remote → credentials → Tomcat URL.
- **How to chain jobs?** Build job → Post-build Actions → Build other projects → next job name → only if stable.
- **How to create visual pipeline?** Dashboard `+` → Build Pipeline View → initial job = Build job.
- **Build failed?** Open job → build number → Console Output; check Git URL/branch, Maven tool, goals, dependencies, WAR file, credentials, Tomcat URL.

---

## Experiment 9 – Jenkins Pipeline Script: likely task variations

1. New Item → name (example `MavenScriptPipeline`) → Pipeline → OK.
2. Configure → Pipeline → Definition: **Pipeline script**.
3. Paste the pipeline from the experiment and change repository URL/project folder as given.
4. If asked to clean before build, use `mvn clean` in the Clean stage.
5. If asked to compile, add a stage with `mvn compile`.
6. If asked to install, add a stage with `mvn install`.
7. If asked to test, add a stage with `mvn test`.
8. If asked to package/create WAR or JAR, add a stage with `mvn package`.
9. If asked to deploy to Tomcat after packaging, add a **Deploy** stage after Package; deployment settings/credentials must be configured for the Tomcat server.
10. If folder name is not `mavenjava`, change every Maven `-f` path (or the `dir('folder')` block) to the correct folder.
11. On Windows agent use `bat`; Linux agent use `sh`.
12. Save → Build Now → check each stage and Console Output.


## Experiment 5/6 – Redis Docker tasks (quick task variants)

### Pull and run Redis image
```bash
docker pull redis
docker images
docker run -d --name myredis -p 6379:6379 redis
docker ps
```

### Open Redis CLI and test it
```bash
docker exec -it myredis redis-cli
```
Inside Redis prompt, enter one command at a time:
```text
PING
SET name Haseena
GET name
EXIT
```
Expected: `PING` returns `PONG`; `SET` returns `OK`; `GET name` returns the stored value.

### Redis container management
```bash
docker stop myredis
docker start myredis
docker restart myredis
docker logs myredis
docker inspect myredis
docker rm -f myredis
```

### Redis Dockerfile build (if task says build an image)
Create a folder and a file named `Dockerfile`:
```dockerfile
FROM redis:latest
CMD ["redis-server"]
```
Then:
```bash
docker build -t redisnew .
docker run -d --name myredisnew redisnew
docker ps -a
docker logs myredisnew
```

### Redis in Docker Compose (if task asks to add Redis service)
Inside `services:` in `docker-compose.yml`, add:
```yaml
  redis:
    image: redis:alpine
```
If asked for `depends_on`, add under the web service:
```yaml
  depends_on:
    - redis
```
Then run:
```bash
docker compose up -d
docker compose ps
```

---

# Experiment 12 – AWS EC2/Tomcat: missing port, access denied, and deployment scenarios

## If you did not add Port 8080 (or the required app port) during EC2 launch

**You do not need to recreate the EC2 instance. Add the rule to its Security Group.**

1. AWS Console → EC2 → Instances → select your instance.
2. Open the **Security** tab → click the attached **Security Group**.
3. Select **Inbound rules** → **Edit inbound rules**.
4. Click **Add rule**.
5. Type: `Custom TCP`.
6. Port range: `8080` (or the port your task requires; lab Docker mapping may use `6060`).
7. Source: for a lab test, use the instructed source; preferably `My IP` for restricting access. Do not leave a management port open to everyone unnecessarily.
8. Click **Save rules**.
9. Retry browser URL using `http://PUBLIC_IP:PORT`.

### Port mapping reminder – know which port to open
- Docker command `-p 6060:8080` means **EC2/host port 6060** is the browser-facing port; add inbound TCP `6060`, open `http://PUBLIC_IP:6060`.
- Docker command `-p 8080:8080` means host port `8080`; add inbound TCP `8080`, open `http://PUBLIC_IP:8080`.
- If Tomcat runs directly on EC2 at port `8080` (not inside Docker), add inbound TCP `8080` and open `http://PUBLIC_IP:8080`.
- `EXPOSE 8080` in Dockerfile does **not** open AWS firewall/security-group access. Security Group rule and Docker `-p` mapping are still required.

## If the EC2 website does not open

Check in this order:
1. EC2 state = Running; status checks passed.
2. Copy the current Public IPv4 address (it may change after stop/start).
3. Security Group inbound includes the **host port** you are opening (e.g., 6060 or 8080).
4. Container running:
```bash
sudo docker ps
sudo docker ps -a
```
5. Logs:
```bash
sudo docker logs app-demo
```
6. Confirm the port mapping:
```bash
sudo docker port app-demo
```
7. Check Tomcat app path: if WAR is `myapp.war`, open `http://PUBLIC_IP:PORT/myapp` (not necessarily just `/`).
8. If port rule was missing, add it using the steps above; wait briefly and retry.

## AWS/EC2 scenario quick answers
- **Forgot inbound port 8080?** Instance → Security tab → Security Group → Edit inbound rules → Add rule → Custom TCP → 8080 → choose source → Save.
- **App is Docker-mapped to 6060 but opened 8080?** Open 6060 and allow inbound 6060; Docker mapping is host:container.
- **Tomcat works locally but not from internet?** Check EC2 Security Group inbound host port, Docker port mapping, and public IP.
- **Browser Connection Refused?** Confirm container/Tomcat is running and the host port mapping is correct.
- **Tomcat 404?** Use the WAR/context name in the URL and inspect Tomcat logs.
- **Public IP changed?** Use current Public IPv4; Elastic IP is needed for a fixed address.
- **Need app to start after EC2 reboot?** Configure Docker restart policy or systemd/User Data if the task asks.
- **Need outbound access from private subnet?** Lab manual notes NAT Gateway or VPC endpoints depending on requirements.

