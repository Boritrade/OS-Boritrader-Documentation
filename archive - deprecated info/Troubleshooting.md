# Troubleshooting Django Application
**Boritrade, LLC**
**Last Updated:** January 30, 2025  

## ASGI Server Error

Most problems, especially syntax errors or misconfigurations, will typically cause the ASGI server to crash first. When this happens, further information is needed to pinpoint the issue. The first step in troubleshooting is to check the ASGI error logs, which will provide details about what went wrong and where the error occurred.

To access the logs:

1. Navigate to project directory, or open a shell inside the Docker container:
   ```bash
   docker-compose exec django bash
   ```

2. Navigate to the logs directory and view the ASGI error log:
   ```bash
   cd logs      # Enter the logs directory, located 1 level above 'app' default working directory
   cat asgi_err.log  # View the ASGI error log
   ```

Check the logs for the exact error message and traceback, which will help in identifying and fixing the root cause of the crash.

3. Additional logs can be found at: https://boritrade.sentry.io/

---

## Database

### 1. Switch to PostgreSQL User
To perform database operations, switch to the PostgreSQL user:
```bash
sudo -i -u postgres
```

### 2. Restart PostgreSQL Server
If the database server needs to be restarted:
```bash
sudo systemctl restart postgresql
```

### 3. Connect to the Database
To connect to the PostgreSQL database via the command line:
```bash
psql -h localhost -p 5432 -U postgres -d deep_stone_crypt  # Default connection
psql -h localhost -p 5433 -U cayde6 -d deep_stone_crypt  # Alternative port or user
```

### 4. Restore the Database
To restore a database from a SQL dump file:

1. Copy the dump file into the Docker container:
   ```bash
   docker cp database_dump.sql your_db_container:/database_dump.sql
   ```

2. Run the restore command inside the container:
   ```bash
   docker-compose exec db bash
   psql -U postgres -d deep_stone_crypt -f /database_dump.sql
   ```


### 5. Applying Migrations and Create Superuser

Apply any pending migrations to keep your database schema up to date:
```bash
docker-compose exec django python manage.py makemigrations
docker-compose exec django python manage.py migrate
```

### 6. Creating a Superuser
Create a superuser to access Django’s admin interface:
```bash
docker-compose exec django python manage.py createsuperuser
```

**Note:** 5 and 6 may need to be repeated after rebuilding your project or modifying your database schema.

---

## Additional Tips

- Django admin can be accessed at: [http://0.0.0.0:8000/admin/](http://0.0.0.0:8000/admin/)

- If you've made changes to `models.py` but `makemigrations` detects no changes, specify the app name explicitly:
  ```bash
  docker-compose exec django python manage.py makemigrations [app_name]
  ```


## AWS 

### 1. Basic Docker Cleanup
To clear up memory and free up disk space on an EC2 instance running Docker, you can remove unused Docker objects (containers, images, volumes, and networks) as well as temporary files. Follow these steps:

Docker provides commands to clean up unused resources:

```bash
docker container prune # Remove Stopped Containers
docker image prune # Remove Unused Images
docker network prune # Remove Unused Networks
docker volume prune ## Remove Unused Volumes

docker system prune # Clean Up All Unused Resources at once
docker system prune -a --volumes # For even more aggressive cleaning (including unused volumes)
```
> **Warning**: The `-a` flag removes all unused images, not just dangling ones. Be cautious if you have images you want to keep.

---

#### **Clear Docker Build Cache**
Docker builds can accumulate cached layers over time. You can clear them using:
```bash
docker builder prune

docker builder prune --all # More aggressive cleanup
```


#### **Check for Orphaned Containers**
Sometimes, dangling or "orphaned" containers consume memory or disk space. List all containers, including stopped ones:
```bash
docker ps -a
docker rm <container_id> # Remove any unwanted containers:
```

---

### 2. **Clearing Temporary Files**
Temporary files on the EC2 instance can also take up space. Use the following commands:

#### Clear Temporary Directory
```bash
sudo rm -rf /tmp/*
```

#### Clear Unused Package Files (for Ubuntu/Debian-based instances)
```bash
sudo apt-get clean
```
---




### 3. **Restarting the Server:**
#### Soft Reset (Weekly Reset)
Currently, every 7 days AWS RDS secret keys rotate, causing the application to lose access to RDS. 
The credentials are reloaded automatically using the following script:
```bash
source /etc/profile.d/set_env_vars.sh
```
which runs automatically on every new ssh connection, but can be run manually as well.
Once run, simply re-initialize the server. 
```bash
docker compose up
```

#### Hard Reset
If You run into ghost problems, where everything is seemingly fine but users just aren't recieving updates:
```bash
docker compose down # stop active server
docker system prune # clear all docker images
git pull # pull latest update
docker compose build 
docker compose up -d 
docker compose exec django python djangostuff/manage.py collectstatic # make sure ngnix server has latest staticfiles
```


## User-End Troubleshooting:

### 1. Can connect to /login & /settings, But not /dashboard.
That likely means API keys are invalid, update in settings.

### 2. Other
Clear browser cache, refresh page twice.
