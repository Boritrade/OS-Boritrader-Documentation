# Getting Started with Boritrader
**Boritrade, LLC**  
**Last Updated:** February 10, 2026   

This guide walks you through setting up Boritrader from a fresh clone to a fully operational trading platform.

---

## 📋 Prerequisites

Before you begin, ensure you have:

- **Docker** (version 20.10+) and **Docker Compose** (version 2.0+)
- **Git**
- **Binance API Keys** (testnet recommended for first setup)
  - Get testnet keys: https://testnet.binance.vision/
- **Email SMTP Credentials** (AWS SES recommended, or any SMTP server)

---

## 🚀 Quick Start (5 Minutes)

### Step 1: Clone and Configure

```bash
# Clone the repository
git clone https://github.com/yourusername/boritrader.git
cd boritrader

# Create environment file from template
cp .env.example .env

# Edit with your credentials
nano .env  # or use your preferred editor
```

**Minimum required in `.env`:**
```env
# Generate with: python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
DJANGO_SECRET=your-random-secret-key-here

# Database (these defaults work with docker-compose)
POSTGRES_DB=boritrader_db
POSTGRES_USER=boritrader_user
POSTGRES_PASSWORD=choose-secure-password
POSTGRES_HOST=db
POSTGRES_PORT=5432

# Binance (use testnet for learning!)
MASTER_B_KEY=your-binance-api-key
MASTER_B_SECRET=your-binance-secret
MASTER_B_TLD=com
MASTER_B_TESTNET=True

# Email (for password reset - can skip initially)
SES_SMTP_USERNAME=your-ses-username
SES_SMTP_PASSWORD=your-ses-password
```

### Step 2: Start the Application

```bash
# Build and start all services
docker-compose up --build

# First run takes 2-3 minutes (downloading images, building)
# You'll see logs from django, db, and redis services
```

**Wait for these messages:**
```
django_1  | Finished migrate.
django_1  | Finished collecting static files.
django_1  | Starting supervisord...
```

The application is now running at:
- **Main App**: http://localhost:8000
- **WebSocket**: http://localhost:8001

### Step 3: Create Your Admin Account

Open a **new terminal** (keep the first one running):

```bash
# Create superuser
docker-compose exec django python /app/djangostuff/manage.py createsuperuser

# Follow the prompts:
# Username: (choose one)
# Email: your-email@example.com
# Password: (choose secure password)
# Password (again): (confirm)
```

### Step 4: Populate Trading Algorithms

This is a **required step** - the app needs algorithms in the database to function.

#### Option A: Admin Panel (Recommended)

1. Navigate to http://localhost:8000/sys-admin/
2. Login with your superuser credentials
3. Click on **"Algorithms"**
4. Click **"Add Algorithm"** for each algorithm below

**Add these 6 algorithms** (copy exact values):

| Display Name | Internal Name | Description |
|--------------|---------------|-------------|
| Moving Average Crossover | macs | Identifies uptrends using MA9/MA50 crossover with RSI confirmation |
| Catching The Bottom | catching_the_bottom | Buys oversold conditions with RSI below 40 and EMA100 > EMA50 |
| Maximized RSI | max_rsi | Long-term strategy buying RSI < 35 below MA100, exiting at RSI > 65 |
| Ride The Trend | ride_the_trend | Participates in strong uptrends when RSI > 70 on 4-hour chart |
| Dollar Cost Averaging | dca | Spreads purchases over time when RSI < 50 and price below MA50 |
| Pump and Dump | pump_and_dump | Targets rapid price movements using volume spikes and moving averages |

**Detailed algorithm descriptions** are in `docs/algorithms/overview.md`


TODO; Fix Internal_name field
Create algorithm documentation - Adding your own algorithm
Create algorithm documentation - display vs internal name




#### Option B: Django Shell (Advanced)

```bash
docker-compose exec django python /app/djangostuff/manage.py shell
```

Then paste this code:

```python
from dashboard.models import Algorithm

algorithms = [
    {
        'display_name': 'Moving Average Crossover',
        'internal_name': 'macs',
        'description': 'Identifies uptrends using MA9/MA50 crossover with RSI confirmation'
    },
    {
        'display_name': 'Catching The Bottom',
        'internal_name': 'catching_the_bottom',
        'description': 'Buys oversold conditions with RSI below 40 and EMA100 > EMA50'
    },
    {
        'display_name': 'Maximized RSI',
        'internal_name': 'max_rsi',
        'description': 'Long-term strategy buying RSI < 35 below MA100, exiting at RSI > 65'
    },
    {
        'display_name': 'Ride The Trend',
        'internal_name': 'ride_the_trend',
        'description': 'Participates in strong uptrends when RSI > 70 on 4-hour chart'
    },
    {
        'display_name': 'Dollar Cost Averaging',
        'internal_name': 'dca',
        'description': 'Spreads purchases over time when RSI < 50 and price below MA50'
    },
    {
        'display_name': 'Pump and Dump',
        'internal_name': 'pump_and_dump',
        'description': 'Targets rapid price movements using volume spikes and moving averages'
    }
]

for algo in algorithms:
    Algorithm.objects.create(**algo)
    print(f"✓ Created: {algo['display_name']}")

print("\n✅ All algorithms added!")
exit()
```

### Step 5: Start Trading! 🎉

1. **Visit** http://localhost:8000
2. **Register** a new user account (or login with superuser)
3. **Go to Settings** to configure:
   - Your Binance API keys (if different from master key)
   - Trading preferences
   - Notification settings
4. **Visit Dashboard** to:
   - Enable/disable algorithms
   - Monitor trading signals
   - View real-time market data

---


## 📊 Understanding Your Setup

### What Just Happened?

When you ran `docker-compose up`, three services started:

1. **PostgreSQL Database** (`db` service)
   - Stores: users, algorithms, preferences, trading history
   - Port: 5433 (host) → 5432 (container)
   - Data persists in Docker volume `postgres_data`

2. **Redis** (`redis` service)
   - Handles: WebSocket channels for real-time updates
   - Port: 6379
   - In-memory only (resets on restart)

3. **Django Application** (`django` service)
   - Runs: Web server (port 8000) and WebSocket server (port 8001)
   - Automatically runs migrations and collects static files on startup



## 🔧 Common Tasks

### Viewing Logs

```bash
# All services
docker-compose logs -f

# Just Django
docker-compose logs -f django

# Just database
docker-compose logs -f db

# Last 100 lines
docker-compose logs --tail=100
```

### Stopping the Application

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (DELETES DATABASE!)
docker-compose down -v
```

### Restarting After Changes

```bash
# If you changed .env or code
docker-compose restart django

# If you changed dependencies
docker-compose up --build

# If you changed database schema
docker-compose exec django python /app/djangostuff/manage.py migrate
```


### Database Migrations
- Every time you make a change to database schema or models.py you will need to run a new migration. The application handles this automatically on startup via entrypoint.sh run by Dockerfile, but use the following if you may need to make additional adjustments.

- Importantly, The Django migrations folder is EXCLUDED in ./gitignore by default, so migrations will only persist LOCALLY unless you make an external backup. 

- Never push migrations/ file to Boritrader origin repository.  

```bash
# Create new migrations
docker-compose exec django python /app/djangostuff/manage.py makemigrations

# Apply migrations
docker-compose exec django python /app/djangostuff/manage.py migrate

# View migration status
docker-compose exec django python /app/djangostuff/manage.py showmigrations
```
This is executed automatically on startup by entry


### Accessing Django Shell

```bash
# Python shell with Django models loaded
docker-compose exec django python /app/djangostuff/manage.py shell

# Example: Check user count
>>> from django.contrib.auth.models import User
>>> User.objects.count()
>>> exit()
```

### Running Tests

```bash
# All tests
docker-compose exec django pytest

# Specific test file
docker-compose exec django pytest djangostuff/dashboard/tests/test_views.py

# With coverage report
docker-compose exec django pytest --cov=dashboard --cov-report=html
```

---

## 🐛 Troubleshooting
see Documentation/troubleshooting.md for common errors and troubleshooting guides.

---

## 🔐 Security Checklist

Before deploying to production:

- [ ] Set `DEBUG=False` in `.env`
- [ ] Use strong, unique `DJANGO_SECRET` (50+ characters)
- [ ] Use strong database password
- [ ] Enable HTTPS/SSL
- [ ] Configure `ALLOWED_HOSTS` in `settings.py`
- [ ] Use production Binance keys (not testnet)
- [ ] Restrict Binance API key to specific IPs
- [ ] Set up proper backup system
- [ ] Review `SECURITY.md` for full checklist

---
### Database Access

Want to query the database directly? See:
- `Documentation/database/DATABASE-ACCESS.md` - PostgreSQL connection guide

### Optional Features

- **Cloudflare Turnstile** (CAPTCHA): `docs/security/TURNSTILE_SETUP.md`
- **Email Notifications**: Configured via SES credentials in `.env`
- **Custom Algorithms**: See `docs/development/custom-algorithms.md`

