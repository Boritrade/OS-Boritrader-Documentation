# Unit Testing (via Github Actions & Locally)
**Boritrade, LLC**   
Last Updated: 1/16/25

# **CI/CD Pipeline Documentation for Running Django Tests**

This document provides an overview of the CI/CD pipeline implemented using GitHub Actions, details the process of running Django tests, and outlines the requirements for successful execution.

---

## **Pipeline Overview**

The GitHub Actions workflow (`test.yml`) automates the following steps:
1. Sets up the Docker Compose environment.
2. Creates the `.env` file dynamically using **GitHub Secrets**.
3. Builds and starts the Docker containers defined in `docker-compose.yml`.
4. Waits for required services (PostgreSQL and Redis) to become ready.
5. Installs Python dependencies, including `requirements.txt` and `requirements_git.txt`.
6. Runs database migrations.
7. Executes Django tests located in `dashboard/tests` using `pytest`.
8. Shuts down containers after the tests complete.

---

## **Pre-requisites**

1. **GitHub Secrets Configuration**  
   The workflow depends on the following secrets configured in your GitHub repository. Ensure these are added under **Settings > Secrets and Variables > Actions**:
   - `DJANGO_SECRET`: The secret key for Django.
   - `READ_ONLY_PAT`: The GitHub personal access token for read-only access.
   - `POSTGRES_DB`: The PostgreSQL database name.
   - `POSTGRES_USER`: The PostgreSQL username.
   - `POSTGRES_PASSWORD`: The PostgreSQL password.
   - `POSTGRES_HOST`: The PostgreSQL host (usually `localhost` or the service name in Docker Compose).
   - `POSTGRES_PORT`: The PostgreSQL port (default is `5432`).

2. **Docker Compose File**
   Ensure `docker-compose.yml` is correctly configured to define the services (e.g., `django`, `postgres`, `redis`) required for the application.

---

## **Running Tests via GitHub Actions**

The CI/CD pipeline is triggered automatically on the following events:
- **Pushes**: Any push to branches in the repository.
- **Pull Requests**: Any pull request targeting a branch in the repository.

Steps performed in the workflow:
1. **Create `.env` File**  
   The `.env` file is dynamically generated during the pipeline execution using the GitHub Secrets.

2. **Build and Start Containers**  
   Docker Compose builds and starts all necessary containers (`django`, `postgres`, `redis`).

3. **Install Dependencies**  
   The pipeline installs the required Python packages listed in `requirements.txt` and `requirements_git.txt`.

4. **Run Migrations**  
   Database migrations are applied using Django's `manage.py migrate` command.

5. **Run Tests**  
   Tests in `dashboard/tests` are executed using `pytest`.

6. **Shutdown Containers**  
   Docker Compose shuts down all containers after the tests complete, regardless of their success or failure.

---

## **How to Manually Run Tests Locally**

To run tests locally, follow these steps:

1. **Clone the Repository**:
   ```bash
   git clone git@github.com:<username>/<repository>.git
   cd <repository>
   ```

2. **Set Up the Environment**:
   Create a `.env` file in the root directory with the following keys:
   ```env
   DJANGO_SECRET=your_django_secret
   READ_ONLY_PAT=your_read_only_pat
   POSTGRES_DB=your_postgres_db
   POSTGRES_USER=your_postgres_user
   POSTGRES_PASSWORD=your_postgres_password
   POSTGRES_HOST=localhost
   POSTGRES_PORT=5432
   ```

3. **Build and Start Containers**:
   ```bash
   docker-compose up -d --build
   ```

5. **Run Migrations & create superuser**:
   ```bash
   docker-compose exec django python djangostuff/manage.py makemigrations dashboard
   docker-compose exec django python djangostuff/manage.py migrate
   docker-compose exec django python djangostuff/manage.py createsuperuser

   ```

6. **Run Tests**:
   ```bash
   docker-compose exec django pytest djangostuff/dashboard/tests
   ```

7. **Shut Down Containers**:
   ```bash
   docker-compose down
   ```

---

## **Notes**
1. The pipeline will fail if the required GitHub secrets are not set.
2. All tests must be placed in the `dashboard/tests` directory.
3. Ensure that the services in `docker-compose.yml` (e.g., `postgres` and `redis`) are correctly named and configured for inter-service communication.
