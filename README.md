DockerPracticeTP1

This project runs a small API for students and departments. It uses Docker to package the app and database, and Docker Compose to run them together.

Install Docker Desktop. From the project folder, run:

docker compose -f backend/simpleapi/docker-compose.yml up --build

The first run builds the images and may take a few minutes. When it is ready, open http://localhost/departments. To stop the containers:

docker compose -f backend/simpleapi/docker-compose.yml down

How the containers work together

Docker images are the packaged instructions and files for a service. A container is a running copy of an image. Compose builds the images and starts three containers on a private network:

database is PostgreSQL. It creates the tables and sample data using the SQL files in databases/.

backend runs the Java API. It connects to the database using the name database.

httpd runs Apache. It accepts web requests on your computer's port 80 and forwards them to the backend container on port 5000.

The containers can find each other by their Compose service names. Only Apache's port 80 is published to your computer, so use http://localhost/departments rather than connecting to the database directly.

Dockerfiles and images

backend/simpleapi/Dockerfile builds the API in two steps. First it uses a Java 21 image and Maven to build the application into a JAR file. Then it copies the JAR into a runtime image and starts the app. This keeps Maven and the source code out of the running application image.

backend/simpleapi/Dockerfile.apache builds the separate Apache image. It copies in Apache's configuration and the web page.

databases/Dockerfile starts from a PostgreSQL image and copies in the scripts that create and fill the database.

Docker Compose reads backend/simpleapi/docker-compose.yml. Its build paths tell Docker where to find each Dockerfile and the files that Dockerfile needs. The backend and Apache use backend/simpleapi as their build context; the database uses databases.

GitHub Actions

.github/workflows/main.yml runs automatically when code is pushed to main or develop, and when a pull request is opened or updated.

The first job installs Java 21, runs Maven's clean verify 
command (mvnw clean verify), and sends a code scan to SonarCloud. The second job runs only if the first job succeeds. It logs in to Docker Hub and builds and uploads the API, database, and Apache images.

The workflow needs Docker Hub username and password secrets named DOCKER_USERNAME and DOCKER_PASSWORD. The SonarCloud scan also needs a SONAR_TOKEN secret.

The Docker Hub image names use the account in DOCKER_USERNAME and the tags in the workflow.
