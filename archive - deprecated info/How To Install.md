# The Boritrader Documentation: Installation
**Boritrade, LLC**
**Last Updated:** January 30, 2025  

## Prerequisites
- Python 3.7+
- pip (Python package installer)
- A Binance account and API keys (with trading permissions enabled)
- Docker 26.1.4+


## Setup

1. **Clone the repository:**
    ```sh
    git clone https://github.com/Boritrade/Boritrader.git
    cd Boritrader
    ```
    Directory should be:

        app/
            djangostuff/
            docker-compose.yml
            Dockerfile
            README.md
            requirements.txt
            supervisord.conf

1. **Build an Env.list file:**
Create new file 1 level above docker-compose.yml
This will hold all environmental variables, including all API keys & secrets.

    - note: in settings.py the line *from decouple import config* is referencing a function config that pulls env vars from the container into a dictionary *config*.
    - Adding the env.list file one directory above docker-compose will allow docker-compose to load all env vars into container automatically.

    ```sh
    # Directory Tree:

    app/
        djangostuff
        docker-compose.yml
        Dockerfile
        README.md
        requirements.txt
        supervisord.conf
    env.list
    ```

    ```sh
    # env.list file

    # Django required
    DJANGO_SECRET=django-secret123

    # Database credentials
    POSTGRES_DB=deep_stone_crypt
    POSTGRES_USER=
    POSTGRES_PASSWORD=
    POSTGRES_HOST=db  # Note that this is set to 'db' to match the service name in docker-compose
    POSTGRES_PORT=5432  # Internal port of the PostgreSQL container

    ```



# Installation Method 1. Docker-Compose Build (Recommended)

1. **Build Project via docker-compose:**
    ```sh
    docker-compose build
    ```
    or 'docker compose build' depending on docker version

1. **Run via docker-compose:**
    ```sh
    docker-compose up
    ```

Done!

## [additional] Initializing Database (See Database Documentation for more):

### Start the Containers
```sh
docker-compose up -d
```

**Copy the database dump & restore:**

```sh
docker cp database_dump.sql your_db_container:/database_dump.sql
docker-compose exec db bash
psql -U postgres -d deep_stone_crypt -f /database_dump.sql
```

### Apply Migrations

```sh
docker-compose exec django python manage.py makemigrations
docker-compose exec django python manage.py migrate
```

### Create a Superuser (Enables Django Admin)

```sh
docker-compose exec django python manage.py createsuperuser
```


# [DEPRECATED] Installation Method 2. Manual Installation & Execution via Docker
To install manually, you will still need docker. However, you can manually build & run each container separately if needed.

### Step 1. Dependencies (requirements.txt)
Dependencies are installed automatically when program is built & run with Docker, but if you need to install manually use:

```console
pip install -r requirements.txt
```

OR

```console
pip install pyyaml
pip install django

# For Binance API
pip install python-binance

# For Algorithm Analysis
pip install pandas

# For Algorithm Monitoring via Websocket
pip install channels
pip install channels-redis
pip install uvicorn

# For Django config path loading (especially for database)
pip install python-decouple

# For Django database loading
pip install psycopg2-binary

```

### Step 2: BUILD
in project root directory:

```console
sudo docker build -t myapp .
```

### Step 3. Run Redis Server
```console
docker run -d --name redis -p 6379:6379 redis:latest
```

### Step 4. Run the Django container linked to the Redis container with the env file and expose both ports
```console
docker run -d --name django_container --link redis:redis --env-file=env.list -p 8000:8000 -p 8001:8001 $IMAGE_NAME
```

note: the Asgi server runs on 8001, the Django server runs on 8000


### Step 5. Run the database
You will also need to initiate & run the docker database somehow, no longer writing  additional instructions as it is much faster to just use the docker-compose method.
