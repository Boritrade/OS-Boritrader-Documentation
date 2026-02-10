
# Troubleshooting
**Boritrade, LLC**  
**Last Updated:** February 10, 2026   

### Application Won't Start

1. Check if all required environment variables are set in `.env`
2. Verify Docker services are running: `docker-compose ps`
3. Check logs: `docker-compose logs django`
4. Ensure ports 8000, 8001, 5432, 6379 are not already in use
#### Port Already in Use

**Error**: `bind: address already in use`

**Solution**: Another service is using ports 8000, 8001, 5433, or 6379

```bash
# Find what's using the port
lsof -i :8000  # macOS/Linux
netstat -ano | findstr :8000  # Windows

# Stop the conflicting service or change ports in docker-compose.yml
```


### Database Connection Errors

1. Verify PostgreSQL is running: `docker-compose ps db`
2. Check database credentials in `.env`
3. Ensure `POSTGRES_HOST=db` (not `localhost`) when using Docker Compose
#### Database Connection Failed

**Error**: `could not connect to server` or `connection refused`

**Solution**: Database hasn't finished starting

```bash
# Wait 10-15 seconds after docker-compose up
# Check database logs
docker-compose logs db

# Restart Django service
docker-compose restart django
```


### WebSocket Connection Issues

1. Verify Redis is running: `docker-compose ps redis`
2. Check browser console for WebSocket errors
3. Ensure port 8001 is accessible


### Import Errors

1. Rebuild the Docker image: `docker-compose up --build`
2. Verify all dependencies in `requirements.txt`
3. Check Python version compatibility


### Database failure after adjusting `models.py`
 Every time you make a change to models.py you will need to run a new migration. The application handles this automatically on startup via entrypoint.sh run by Dockerfile, but you may need to make additional adjustments.

 Importantly, The Django migrations folder is EXCLUDED in ./gitignore by default, so migrations will only persist LOCALLY unless you make an external backup. 

 **Never push `migrations/` file to Boritrader origin repository.** 
#### Migrations Not Applied

**Error**: `no such table: dashboard_algorithm`

**Solution**: Run migrations manually

```bash
docker-compose exec django python /app/djangostuff/manage.py migrate
```


### Binance API Errors

**Error**: `API key invalid` or `signature invalid`

**Solutions**:
1. Verify API keys in `.env` (no extra spaces!)
2. Use testnet keys from https://testnet.binance.vision/
3. Set `MASTER_B_TESTNET=True` when using testnet
4. Ensure proper TLD for binance region is used in .env
    - `us` for Binance.us
    - `com` for Binance.com
5. Check API key permissions in Binance dashboard


### Static Files Not Loading

**Error**: CSS/JS not working, page looks broken

**Solution**: 
1. Clear browser cache
2. Collect static files
```bash
docker-compose exec django python /app/djangostuff/manage.py collectstatic --noinput
```

