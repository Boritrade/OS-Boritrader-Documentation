# Database Access Guide

This guide explains how the PostgreSQL database works in Boritrader and how to access it.

---

## 🎯 Important: No PostgreSQL Installation Needed!

**Docker handles everything.** You do NOT need to:
- ❌ Install PostgreSQL on your computer
- ❌ Have an existing database
- ❌ Configure PostgreSQL manually

**Docker automatically**:
- ✅ Downloads PostgreSQL (version 13)
- ✅ Creates a new database inside a container
- ✅ Uses your `.env` values to initialize it
- ✅ Persists data in a Docker volume

---

## 🔑 How Database Creation Works

### Step 1: You Choose Credentials

In your `.env` file, you provide credentials that **will be used to CREATE** the database:

```env
POSTGRES_DB=boritrader_db        # ← Creates database with this name
POSTGRES_USER=boritrader_user    # ← Creates user with this username
POSTGRES_PASSWORD=secure-pass    # ← Sets this as the password
POSTGRES_HOST=db                 # ← Container name (don't change)
POSTGRES_PORT=5432               # ← Internal port (don't change)
```

**Important**: These are NOT credentials to an existing database. They tell Docker **what to create**.

### Step 2: Docker Creates Everything

When you run `docker-compose up`, Docker:

1. **Pulls** the `postgres:13` image (if not already downloaded)
2. **Starts** a PostgreSQL container
3. **Reads** your `.env` file
4. **Creates** a new database named `boritrader_db`
5. **Creates** a new user `boritrader_user` with password from `.env`
6. **Grants** full permissions to this user
7. **Saves** all data to a Docker volume named `postgres_data`

### Step 3: Django Connects

Django automatically connects to this newly created database using the same `.env` credentials.

---

## 🗄️ Where is the Database Stored?

### Docker Volume (Persistent Storage)

Your database is stored in a **Docker-managed volume**:

```yaml
volumes:
  postgres_data:  # ← Docker creates and manages this
```

**Location varies by OS:**

- **Linux**: `/var/lib/docker/volumes/boritrader_postgres_data/_data`
- **macOS**: `~/Library/Containers/com.docker.docker/Data/vms/0/`
- **Windows**: `\\wsl$\docker-desktop-data\data\docker\volumes\`

**You don't need to know the exact location** - Docker manages it for you!

### What Persists

✅ **Persists** (survives container restarts):
- All user accounts
- All algorithms
- All settings
- All trading history
- Everything in the database

❌ **Doesn't persist** (if you run `docker-compose down -v`):
- The `-v` flag **deletes volumes**, which deletes all data
- Use `docker-compose down` (without `-v`) to keep data

---

## 🔌 Connection Details

Once Docker creates your database, here's how to connect:

### From Your Host Machine

```bash
# Using psql (if you have it installed separately)
psql -h localhost -p 5433 -U boritrader_user -d boritrader_db

# Port 5433 because docker-compose.yml maps:
# "5433:5432" (host:container)
```

### From Inside Docker Containers

**From Django container:**
```bash
docker-compose exec django bash
psql -h db -p 5432 -U boritrader_user -d boritrader_db
```

**From database container:**
```bash
docker-compose exec db bash
psql -U boritrader_user -d boritrader_db
```

### Connection Parameters Summary

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Host** | `localhost` (from host)<br>`db` (from containers) | Container name is `db` |
| **Port** | `5433` (from host)<br>`5432` (from containers) | Port mapping in docker-compose |
| **Database** | `boritrader_db` | From your `.env` |
| **Username** | `boritrader_user` | From your `.env` |
| **Password** | (from `.env`) | `POSTGRES_PASSWORD` value |

---

## 🔐 Django Superuser vs Database User

### Two Completely Different Things!

#### Django Superuser (Application Level)

**Created with:**
```bash
docker-compose exec django python /app/djangostuff/manage.py createsuperuser
```

**Purpose**:
- ✅ Login to Django admin panel (`/sys-admin/`)
- ✅ Manage application: users, algorithms, settings
- ✅ Application-level administrator

**Stored in**: `auth_user` table in the database (as data, not a database user)

**Analogy**: Like an admin account in a WordPress site

---

#### PostgreSQL User (Database Level)

**Created automatically** when Docker starts (using `.env` values)

**Purpose**:
- ✅ Direct database access via psql, pgAdmin, etc.
- ✅ Run SQL queries
- ✅ Database backups and maintenance
- ✅ Database-level administrator

**Stored in**: PostgreSQL system catalogs (part of PostgreSQL itself)

**Analogy**: Like MySQL root user or database administrator

---

### Which One Do I Need?

| Task | Use This |
|------|----------|
| **Login to web admin** | Django superuser |
| **Add algorithms via web** | Django superuser |
| **Manage users via web** | Django superuser |
| **Direct SQL queries** | PostgreSQL user (from `.env`) |
| **Database backup** | PostgreSQL user (from `.env`) |
| **Connect with pgAdmin** | PostgreSQL user (from `.env`) |

---

## 🔧 Common Database Tasks

### View All Tables

```bash
# Connect to database
docker-compose exec db psql -U boritrader_user -d boritrader_db

# List tables
\dt

# Describe a specific table
\d auth_user
\d algorithms
```

### Query Data

```sql
-- Count users
SELECT COUNT(*) FROM auth_user;

-- List all algorithms
SELECT * FROM algorithms;

-- View recent activity
SELECT username, date_joined 
FROM auth_user 
ORDER BY date_joined DESC 
LIMIT 10;
```

### Backup Database

```bash
# Create backup file
docker-compose exec db pg_dump -U boritrader_user boritrader_db > backup_$(date +%Y%m%d).sql

# With compression
docker-compose exec db pg_dump -U boritrader_user boritrader_db | gzip > backup_$(date +%Y%m%d).sql.gz
```

### Restore Database

```bash
# Stop Django first
docker-compose stop django

# Restore from backup
docker-compose exec -T db psql -U boritrader_user boritrader_db < backup.sql

# Restart Django
docker-compose start django
```

---

## 🗂️ Database Schema Overview

### Core Tables

After running migrations, you'll have these tables:

| Table | Purpose |
|-------|---------|
| `auth_user` | User accounts (Django users) |
| `user_profiles` | Extended user info (phone, timezone, etc.) |
| `algorithms` | Available trading algorithms |
| `api_keys` | User Binance API keys |
| `user_algorithms` | Algorithm configurations per user |
| `trade_signals` | Trading signals generated by algorithms |
| `notifications` | User notifications |
| `system_logs` | System events and errors |
| `login_attempts` | Failed login tracking |

### View Schema

```bash
docker-compose exec db psql -U boritrader_user -d boritrader_db
```

```sql
-- List all tables
\dt

-- Show table structure
\d+ auth_user
\d+ algorithms

-- Show all columns in a table
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'algorithms';
```

---

## 🛠️ GUI Database Clients

If you prefer a graphical interface:

### pgAdmin

1. **Install**: https://www.pgadmin.org/download/
2. **Add Server**:
   - Host: `localhost`
   - Port: `5433`
   - Database: `boritrader_db`
   - Username: `boritrader_user`
   - Password: (from `.env`)

### DBeaver

1. **Install**: https://dbeaver.io/download/
2. **New Connection** → PostgreSQL
3. Enter connection details (same as above)

### TablePlus (Mac)

1. **Install**: https://tableplus.com/
2. **Create Connection** → PostgreSQL
3. Enter connection details

---

## 🔄 Resetting the Database

### Complete Reset (Deletes All Data!)

```bash
# Stop all services and DELETE volumes
docker-compose down -v

# Start fresh (creates new empty database)
docker-compose up --build

# Recreate superuser
docker-compose exec django python /app/djangostuff/manage.py createsuperuser

# Re-add algorithms
# (follow steps in GETTING_STARTED.md)
```

### Soft Reset (Keep Database, Reset Data)

```bash
# Connect to database
docker-compose exec db psql -U boritrader_user -d boritrader_db

# Delete all data but keep tables
TRUNCATE auth_user CASCADE;
TRUNCATE algorithms CASCADE;
# etc...
```

---

## 📊 Monitoring Database

### Database Size

```sql
-- Total database size
SELECT pg_size_pretty(pg_database_size('boritrader_db'));

-- Size per table
SELECT 
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Active Connections

```sql
-- Current connections
SELECT count(*) FROM pg_stat_activity WHERE datname = 'boritrader_db';

-- Detailed connection info
SELECT pid, usename, application_name, client_addr, state, query
FROM pg_stat_activity
WHERE datname = 'boritrader_db';
```

---

## 🐛 Troubleshooting

### "Database boritrader_db does not exist"

**Cause**: First-time setup, Docker hasn't created the database yet

**Solution**:
```bash
# Make sure .env has POSTGRES_* values
cat .env | grep POSTGRES

# Restart database container to trigger initialization
docker-compose restart db

# Check logs for initialization
docker-compose logs db | grep "database system is ready"
```

### "Password authentication failed"

**Cause**: Wrong password in `.env` or password changed

**Solution**:
1. Check `.env` file for correct password
2. Restart database to use new password:
```bash
docker-compose down
docker-compose up
```

### "Could not connect to server"

**Cause**: Database container not running or not ready

**Solution**:
```bash
# Check if database is running
docker-compose ps db

# Check database logs
docker-compose logs db

# Wait for database to be ready
docker-compose logs db | grep "ready to accept connections"
```

### "Table does not exist"

**Cause**: Migrations haven't been run

**Solution**:
```bash
# Run migrations
docker-compose exec django python /app/djangostuff/manage.py migrate

# Check migration status
docker-compose exec django python /app/djangostuff/manage.py showmigrations
```

---

## 🔐 Security Notes

### For Development

Your `.env` credentials are for the **containerized database only**. They're not exposed outside Docker unless you explicitly connect to port 5433.

### For Production

1. **Use strong passwords** (20+ characters, random)
2. **Don't expose port 5433** publicly:
   ```yaml
   # Remove or comment out in docker-compose.yml
   # ports:
   #   - "5433:5432"
   ```
3. **Regular backups** automated via cron
4. **SSL/TLS** for database connections
5. **Separate read-only user** for analytics:
   ```sql
   CREATE USER readonly WITH PASSWORD 'secure-pass';
   GRANT CONNECT ON DATABASE boritrader_db TO readonly;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
   ```

---

## 📚 Quick Reference

### Connection Strings

```bash
# psql from host
psql -h localhost -p 5433 -U boritrader_user -d boritrader_db

# psql from Django container
psql -h db -p 5432 -U boritrader_user -d boritrader_db

# SQLAlchemy format (for scripts)
postgresql://boritrader_user:password@localhost:5433/boritrader_db

# Django settings.py format (already configured)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'boritrader_db',
        'USER': 'boritrader_user',
        'PASSWORD': config('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': '5432',
    }
}
```

### Essential Commands

```bash
# Backup
docker-compose exec db pg_dump -U boritrader_user boritrader_db > backup.sql

# Restore
docker-compose exec -T db psql -U boritrader_user boritrader_db < backup.sql

# Interactive psql
docker-compose exec db psql -U boritrader_user -d boritrader_db

# Django shell
docker-compose exec django python /app/djangostuff/manage.py shell

# Django dbshell
docker-compose exec django python /app/djangostuff/manage.py dbshell
```

---

## 💡 Key Takeaways

1. **No installation needed** - Docker handles PostgreSQL
2. **`.env` credentials CREATE the database** - not connecting to existing one
3. **Django superuser ≠ Database user** - different purposes
4. **Data persists** in Docker volume `postgres_data`
5. **Port 5433 (host)** maps to **5432 (container)**
6. **From host**: use `localhost:5433`
7. **From containers**: use `db:5432`

---

**Need more help?** See the troubleshooting section or open an issue on GitHub.
