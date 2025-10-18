README – CI/CD Pipeline for API Gateway Service (FusionIQ Backend)
Overview

This Jenkins pipeline automates the build, test, security scan, containerization, and deployment of the API Gateway Service (FusionIQ backend).
It follows modern DevSecOps best practices by integrating SonarQube, OWASP Dependency Check, Trivy, and AWS ECR publishing.

Pipeline Summary
Stage	Description
Checkout Source Code	Clones the backend code from GitHub using Jenkins credentials.
Read pom.xml	Extracts project version and artifactId from pom.xml to use as Docker image tags.
Quality Checks (Parallel)	Runs Unit Tests, SAST, OWASP Dependency Scan, and SonarQube analysis.
Quality Gate Check	Verifies SonarQube Quality Gate; fails pipeline if not met.
Build Application	Builds and packages the Java application using Maven.
Containerization	Builds a Docker image using the generated JAR file.
Trivy Scan	Scans the built Docker image for CRITICAL and HIGH vulnerabilities.
Publish Docker Image	Pushes the image to AWS ECR with multiple tags (version, build_number, latest).
Post Actions	Cleans up the workspace and sends Microsoft Teams notifications for success/failure.
Tools & Versions
Tool	Version	Purpose
Jenkins	2.426+	CI/CD orchestration
Maven	3.9.8	Build & dependency management
Java	21	Application runtime
SonarQube	9.x or above	Static code analysis (SAST + Code Quality)
OWASP Dependency Check	Plugin / Maven	Dependency vulnerability scanning
Trivy	Latest	Container image vulnerability scanning
Docker	24+	Image build and push
AWS CLI	v2	ECR authentication and image push
Microsoft Teams Webhook	–	Notifications
Git	–	Source control checkout
Required Jenkins Plugins

Make sure the following plugins are installed and configured:

Pipeline

Git Plugin

Pipeline: Maven Integration

HTML Publisher Plugin

SonarQube Scanner for Jenkins

OWASP Dependency Check Plugin

Amazon Web Services SDK / Credentials Plugin

Blue Ocean (optional) for better visualization

Credentials Required
ID	Type	Purpose
git-access	Username with password / Personal Access Token	For cloning private GitHub repository
sonar-token	Secret Text	SonarQube authentication token
aws-ecr-creds	AWS Credentials	Used to authenticate and push images to ECR
teams-webhook	Secret Text	Incoming webhook URL for Microsoft Teams notifications
Environment Variables Used
Variable	Description	Source
APP_VERSION	Extracted from pom.xml (project.version)	Set dynamically
DOCKER_IMAGE	Extracted from pom.xml (project.artifactId)	Set dynamically
CONTAINER_NAME	Container name (same as image name)	Derived
AWS_REGION	AWS region (e.g., ap-south-1)	Hardcoded
ECR_URI	AWS ECR repository URI	Hardcoded
TEAMS_WEBHOOK	MS Teams webhook URL	Jenkins Credential
Pipeline Execution Flow
flowchart TD
    A[Checkout Code] --> B[Read pom.xml]
    B --> C[Quality Checks (Parallel)]
    C --> D[Quality Gate Check]
    D --> E[Build Application]
    E --> F[Docker Build]
    F --> G[Trivy Scan]
    G --> H[Publish to AWS ECR]
    H --> I[Post Actions / Teams Notification]

Quality and Security Stages
Unit Tests

Executes mvn test with failure tolerance.

Marks build as UNSTABLE on test failures (does not stop pipeline).

SAST (Static Application Security Testing)

Placeholder for static analysis (can integrate tools like Checkmarx, SonarCloud, or CodeQL).

OWASP Dependency Check

Scans for vulnerable dependencies and generates reports:

dependency-check-report.html

dependency-check-report.xml

Published in Jenkins build results.

SonarQube Analysis

Uses configured SonarQube server (SonarQube-Server).

Authenticated using a SonarQube token.

Enforces Quality Gate — fails pipeline if gate fails.

Trivy Image Scan

Installs Trivy dynamically if not already available.

Fails the pipeline on HIGH or CRITICAL vulnerabilities.

Docker and AWS ECR Publishing

After successful build:

Docker image is built:

${DOCKER_IMAGE}:${APP_VERSION}


Tagged and pushed to AWS ECR as:

${ECR_URI}:${APP_VERSION}
${ECR_URI}:${APP_VERSION}-${BUILD_NUMBER}
${ECR_URI}:latest


ECR repository example:

361769585646.dkr.ecr.ap-south-1.amazonaws.com/media-content-fusioniq

Notifications
Microsoft Teams Notifications

Notifications are sent after every pipeline run:

Status	Color	Example
Success	Green (2EB886)	SUCCESS: Job #45
Unstable	Orange (FFA500)	UNSTABLE: Job #45
Failed	Red (FF0000)	FAILED: Job #45

Each message includes:

Build number and URL

Version

Docker image name

Container name

Post-Build Cleanup

After every run:

Jenkins workspace is deleted using deleteDir() to avoid leftover files.

Ensures clean state for next build.

How to Run the Pipeline

Open Jenkins → New Item → Pipeline

Under Pipeline definition, choose:

Pipeline script from SCM

SCM: Git

Repository URL:
https://github.com/SurnoiTechnology/API-Gateway-Service-FusionIQ-BE.git

Script path: Jenkinsfile

Add necessary credentials (Git, Sonar, AWS, Teams).

Click Build with Parameters → optionally check “Skip Quality Checks”.

Monitor stages and results in Jenkins console.

Best Practices and Recommendations

Use Jenkins Shared Libraries for reusable functions like Teams notifications.
Configure SonarQube Quality Gate thresholds appropriately.
Automate cleanup of old ECR images using AWS Lifecycle policies.
Integrate Slack/Email alerts in addition to Teams (optional).
Secure secrets using Jenkins Credentials plugin (never hardcode).
Run Trivy scans on a Jenkins node with Docker-in-Docker or host Docker access.

Future Enhancements

Deploy to AWS EKS or ECS after pushing image.

Add Terraform or Helm deployment stage.

Integrate Sonatype Nexus IQ or JFrog Xray for deeper dependency scanning.

Add automatic rollback if deployment fails.
