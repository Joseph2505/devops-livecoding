# Discover Github Action

# 1) What are testcontainers?

Testcontainers are a Java library used for facilitating integration testing by providing lightweight, disposable instances of databases (like PostgreSQL in your case) or other services. These instances run in Docker containers, ensuring a consistent and isolated testing environment without the need for complex setup or maintenance. This makes it easier to test your application under real conditions during your continuous integration process.

## Workflow Name

- `name: CI devops 2023`
- This defines the name of the workflow as "CI devops 2023".

## Trigger Conditions

- `on:` Specifies when the workflow should be executed.
- `push:` and `pull_request:` The workflow is triggered on push or pull request events.
- `branches:` Limits the trigger to pushes and pull requests on the `main` and `develop` branches.

## Jobs

- `jobs:` This section defines the jobs to be run by the workflow.
- `test-backend:` Names the job 'test-backend'.

## Run Environment

- `runs-on: ubuntu-22.04`
- Specifies that the job should run on an Ubuntu 22.04 runner provided by GitHub Actions.

## Steps

The actions to be performed in the job are defined under `steps:`.

- `- uses: actions/checkout@v2.5.0`
  - Checks out the repository code into the runner, allowing the workflow to access it.

### Set up JDK 17

- `- name: Set up JDK 17`
- This step sets up Java Development Kit version 17 using the `actions/setup-java@v3` action.
- `java-version: '17'` and `distribution: 'temurin'` specify the JDK version and distribution.

### Build and Test with Maven

- `- name: Build and test with Maven`
- Executes the Maven command to build and test the application.
- `working-directory: ./simple-api` sets the working directory to the `simple-api` folder in your repository.
- `run: mvn clean install --batch-mode --show-version --errors`
  - Runs the Maven command to clean, install, and test the application.
  - `--batch-mode` runs Maven in non-interactive mode.
  - `--show-version` displays the version of Maven.
  - `--errors` makes Maven show error messages.
  
```yaml
name: CI devops 2023

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
      - develop

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build and test with Maven
        working-directory: ./simple-api  # Set the working directory to your Java project directory
        run: mvn clean install --batch-mode --show-version --errors
```

# 2) Document your Github Actions configurations.

# Build your docker images inside your GitHub Actions pipeline
## Workflow Name
- `name: CI/CD Pipeline`
- Defines the name of the workflow.

## Trigger Conditions
- `on:` Specifies when the workflow should be executed.
- `push:` The workflow is triggered on push events.
- `branches:` Limits the trigger to pushes on the `main` branch.

## Jobs
- `jobs:` Defines the jobs to be executed.

## Test Backend Job
- `test-backend:` This job is responsible for testing the backend.
- `runs-on: ubuntu-22.04`: Specifies that the job should run on an Ubuntu 22.04 runner.
- `steps:` Lists the steps to be executed in this job.
  - Check out the code, set up JDK 17, and build and test with Maven, similar to the previous explanation.

## Build and Push Docker Image Job
- `build-and-push-docker-image:` This job handles building and pushing Docker images.
- `needs: test-backend`: Indicates that this job depends on the successful completion of the `test-backend` job.
- `runs-on: ubuntu-22.04`: Specifies the runner environment.
- `steps:` Lists the steps in this job.
  - **Checkout Code**: Checks out the code for access in this job.
  - **Login to DockerHub**: Logs into DockerHub using secrets stored in the GitHub repository. These secrets (`DOCKERHUB_TOKEN` and `DOCKERHUB_USERNAME`) should be set in the repository settings.
  - **Build and Push Docker Images**:
    - There are three separate steps for building and pushing images for the backend, database, and HTTP server. Each uses the `docker/build-push-action@v3` action.
    - `context:` Specifies the directory of the Docker context (e.g., `./simple-api`, `./database`, `./http-server`).
    - `tags:` Sets the tag for the Docker image, using a secret for the DockerHub username and appending the repository-specific tag.
    - `push:` Determines when to push the image. Here, it's set to push only when the `main` branch is updated.
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build and test with Maven
        working-directory: ./simple-api  # Set the working directory to your Java project directory
        run: mvn clean install --batch-mode --show-version --errors
  
  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Login to DockerHub
        run: echo ${{ secrets.DOCKERHUB_TOKEN }} | docker login --username ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./database
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-httpd:latest
          push: ${{ github.ref == 'refs/heads/main' }}

```

# Publish your docker images when there is a commit on the main branch.

## Java CI with Maven and Docker Build

This workflow is designed for continuous integration using Maven and Docker in a Java project.

## Trigger Conditions

- `on:` Defines when the workflow is triggered.
  - `push:` Triggers the workflow on pushes to `main` and `test` branches.
  - `pull_request:` Triggers the workflow on pull requests to `main` and `test` branches.

## Jobs

### Test Backend

- `test-backend:` Job to test the backend of the application.
- `runs-on: ubuntu-latest`: Specifies the latest Ubuntu runner provided by GitHub Actions.
- Steps involved:
  - Checks out the repository code.
  - Sets up JDK 20 with Maven cache.
  - Builds with Maven and runs SonarQube analysis.
    - `run:` Executes Maven with parameters for SonarQube, specifying the project key, organization, Sonar host URL, and Sonar login token (from secrets).

### Build and Push Docker Image

- `build-and-push-docker-image:` Job to build and push Docker images.
- `needs: test-backend`: This job runs after the successful completion of the `test-backend` job.
- `runs-on: ubuntu-22.04`: Specifies the runner environment as Ubuntu 22.04.
- `if:` Conditional execution, only if the event is a push to the `main` branch.
- Steps involved:
  - Checks out the code.
  - Logs in to DockerHub using DockerHub username and token from secrets.
  - Builds and pushes Docker images for simple-api, database, and http-server.
    - For each component, it specifies the context, Dockerfile location, image tags, and enables pushing to DockerHub.

```yaml
name: Java CI with Maven and Docker Build

on:
  push:
    branches: [ "main", "test" ]  # Trigger on push to main and test (develop in your terms)
  pull_request:
    branches: [ "main", "test" ]  


jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 20
      uses: actions/setup-java@v3
      with:
        java-version: '20'
        distribution: 'temurin'
        cache: maven
    - name: Build with Maven
      run: mvn -B verify sonar:sonar -Dsonar.projectKey=live-coding_joseph-live-coding -Dsonar.organization=live-coding -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./simple-api/pom.xml

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v2.5.0
    - name: Log in to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - name: Build and push simple-api Docker image
      uses: docker/build-push-action@v3
      with:
        context: ./simple-api
        file: ./simple-api/Dockerfile
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/simple-api:latest
        push: true
    - name: Build and push database Docker image
      uses: docker/build-push-action@v3
      with:
        context: ./database
        file: ./database/Dockerfile
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/database:latest
        push: true
    - name: Build and push http-server Docker image
      uses: docker/build-push-action@v3
      with:
        context: ./http-server
        file: ./http-server/Dockerfile
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/http-server:latest
        push: true
```
# Setup Quality Gate
Set up your pipeline to use SonarCloud analysis while testing.
![image](https://github.com/Joseph2505/devops-livecoding.git/assets/Capture.PNG)
