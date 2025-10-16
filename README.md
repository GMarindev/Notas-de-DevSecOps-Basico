# Notas-de-DevSecOps-B-sico
En este repositorio añadiré los comandos más útiles de las tecnologías esenciales para operaciones Dev/Ops, y DevSecOps

PREVIOUS NOTES:

All commands listed here follow current stable releases (Docker 25+, Jenkins 2.x LTS, SonarQube 10+, OWASP Dependency-Check 8+).

CLI syntax can evolve slightly between versions — always confirm with --help or consult the official documentation if you encounter errors.

Some commands may require administrator or root privileges (e.g., sudo docker …) depending on your system configuration.

Be cautious with destructive or cleanup commands like docker system prune -af or docker rm -f; use them only when you understand their impact.

Jenkins CLI operations such as install-plugin or safe-restart require sufficient permissions and access tokens. Ensure that “Allow CLI over Remoting” is enabled if needed.

When using SonarQube, remember that the default credentials are admin / admin; change them after the first login.

SonarQube and the OWASP Dependency-Check CLI both rely on Java, so confirm that a compatible JDK version is installed, or use the provided Docker images to avoid dependency issues.

For Docker Compose, prefer docker compose (with a space) — it’s the integrated plugin supported in modern Docker releases. The legacy docker-compose command (with a hyphen) is deprecated.

Docker Swarm remains supported for small-scale clusters, but newer projects may consider Kubernetes for production orchestration or large deployments.

The OWASP Dependency-Check CLI occasionally updates or deprecates flags; always verify via:
https://jeremylong.github.io/DependencyCheck/dependency-check-cli/arguments.html

#  DOCKER Commands

   - Basics  
  docker --version  
  docker info  
  docker system df  
  docker system prune -af  
  
   - Image management  
  docker image ls  
  docker image build -t <image_name>:<tag> .  
  docker image push <repo>/<image>:<tag>  
  docker rmi <image_name>  
  
   - Container lifecycle  
  docker container run -d --name <name> -p 8000:8080 <image>  
  docker container run -it --name <name> <image>  
  docker container ls -a  
  docker container stop <name>  
  docker container start <name>  
  docker container rm <name>  
  docker container rm -f <name>  
  docker logs -f <name>  
  
   - Volumes  
  docker volume create mydata  
  docker volume ls  
  docker run -d -v mydata:/data redis  
  
   - Useful extras  
  docker exec -it <container_name> /bin/bash  
  docker cp <container>:/path/in/container /path/on/host  
  export DOCKER_HOST=ssh://user@remote-server  
  docker ps  
  

#  DOCKER COMPOSE Commands  

   - Start multi-container app  
  docker compose up -d  
  
   - Stop all services  
  docker compose down  
  
   - Rebuild and restart  
  docker compose up -d --build  
  
   - Logs and status  
  docker compose logs -f  
  docker compose ps  
  
   - Run commands inside service  
  docker compose exec <service_name> /bin/bash  
  
   - Restart service  
  docker compose restart <service_name>  
  
#  DOCKER SWARM — Functional Cluster Commands  

   - Initialize swarm on manager node  
  docker swarm init  
  
   - Join worker node (use token from manager)  
  docker swarm join --token <token> <manager_ip>:2377  
  
   - Show cluster state  
  docker node ls  
  
   - Create and scale services  
  docker service create --name web --replicas 3 nginx  
  docker service scale web=5  
  docker service ls  
  docker service ps web  
  
   - Deploy a stack (Compose file)  
  docker stack deploy -c docker-compose.yml mystack  
  docker stack ps mystack  
  docker stack rm mystack  
  
   - Leave swarm (for workers)  
  docker swarm leave --force  
  

#  JENKINS Commands (CLI + Docker)  

   - Run Jenkins with Docker  
  docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts  
  
   - Retrieve initial admin password  
  docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword  
  
   - Logs and management  
  docker logs -f jenkins  
  docker stop jenkins  
  docker start jenkins  
  docker restart jenkins  
  
   - Backup & restore Jenkins data  
  docker cp jenkins:/var/jenkins_home ./jenkins_backup  
  docker cp ./jenkins_backup jenkins:/var/jenkins_home  
  
   - CLI Operations  
  java -jar jenkins-cli.jar -s http://localhost:8080/ list-plugins  
  java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin <plugin_name>  
  java -jar jenkins-cli.jar -s http://localhost:8080/ safe-restart  
  java -jar jenkins-cli.jar -s http://localhost:8080/ restart  

      To create a Docker image with jenkins:  

      Win  
         docker run -d ^  
           --name my-jenkins ^  
           -p 8080:8080 -p 50000:50000 ^  
           -v jenkins_home:/var/jenkins_home ^  
           jenkins/jenkins:lts  
     
      Linux  
         Replace ^ with \  
  

#  SONARQUBE Commands  

   - Start & Stop SonarQube server  
  ./bin/linux-x86-64/sonar.sh start  
  ./bin/linux-x86-64/sonar.sh stop  
  
   - Logs  
  cat $SONARQUBE_HOME/logs/sonar.log  
  
   - Sonar Scanner (local analysis)  
  sonar-scanner \  
    -Dsonar.projectKey=my_project \  
    -Dsonar.sources=. \  
    -Dsonar.host.url=http://localhost:9000 \  
    -Dsonar.login=<your_token>  
  
   - Using Docker for scanner  
  docker run --rm \  
    -e SONAR_HOST_URL="http://localhost:9000" \  
    -e SONAR_LOGIN="<your_token>" \  
    -v "$(pwd)":/usr/src \  
    sonarsource/sonar-scanner-cli  
  
   - Password reset and admin  
   - Edit sonar.properties or via Web UI under Administration > Security  
  

#  OWASP DEPENDENCY-CHECK Commands  

   - Basic scan  
  dependency-check --project "MyApp" --scan "C:\path\to\project"  
  
   - Define report output  
  dependency-check --project "MyApp" --scan "." --out "C:\reports" --format "HTML"  
  
   - Update vulnerability database  
  dependency-check --updateonly  
  
   - Skip update (faster scans)  
  dependency-check --scan "." --noupdate  
  
   - Use custom data directory  
  dependency-check --data "C:\dependency-check-data" --scan "."  
  
   - Generate JSON report (for automation)  
  dependency-check --project "MyApp" --scan "." --format "JSON" --out "C:\reports"  
  
   - Fail build on high severity  
  dependency-check --scan "." --failOnCVSS 7.0  
  
   - Suppress false positives  
  dependency-check --scan "." --suppression "C:\path\to\suppressions.xml"  

   - Purge local NVD data  
  dependency-check --purge  
  
   - CI/CD example  
  dependency-check --project "MyApp" \  
    --scan "C:\Repos\MyApp" \  
    --format "HTML" \  
    --out "C:\Reports" \  
    --data "C:\dependency-check-data" \  
    --failOnCVSS 7.0 \  
    --noupdate  

