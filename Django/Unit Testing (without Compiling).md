# Instructions for Running Unit Tests Without Compiling the Full Application
**Boritrade, LLC**  
**Updated Jan 15, 2025**  

Follow these steps to set up the environment and run unit tests without needing a fully working instance of the application.

---

## 1. Clone the Repository

1. Clone the repository and navigate to the root directory.
   ```bash
   git clone <repository-url>
   cd <repository-root>
   ```

2. Ensure the `djangostuff` directory and a `requirements.txt` file are present.

---

## 2. Create a Virtual Environment

1. Create a virtual environment **above the `djangostuff` directory**:
   ```bash
   python3 -m venv myenv
   source myenv/bin/activate  # For Linux/macOS
   myenv\Scripts\activate     # For Windows
   ```

2. Install dependencies using the provided `requirements.txt`:
   ```bash
   pip install -r requirements.txt
   ```

---

## 3. Configure the Test Settings

1. A file named `test_settings.py` is provided in the `djangostuff` directory. This file contains a minimal Django configuration for running tests. It uses:
   - An in-memory SQLite database (`:memory:`).
   - Basic logging to output test results to the console.
   - A dummy secret key (`test-secret-key`).

2. Set the `DJANGO_SETTINGS_MODULE` environment variable to use the test settings:
   ```bash
   export DJANGO_SETTINGS_MODULE=djangostuff.test_settings  # Linux/macOS
   set DJANGO_SETTINGS_MODULE=djangostuff.test_settings    # Windows
   ```

---


### **Minimal `test_settings.py`**
```python
from pathlib import Path

# Base directory (adjust as necessary for your project structure)
BASE_DIR = Path(__file__).resolve().parent.parent

# Secret Key (use a dummy value)
SECRET_KEY = 'test-secret-key'

# Debug mode enabled for testing
DEBUG = True

# Allowed hosts (empty for testing purposes)
ALLOWED_HOSTS = []

# Installed apps (include only what's necessary for tests)
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'dashboard',  # Your app
]

# Middleware (minimum required for tests)
MIDDLEWARE = [
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

# Root URL configuration (if applicable)
ROOT_URLCONF = 'config.urls'

# Templates (minimal setup)
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'dashboard/templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# Database (use SQLite in-memory database for tests)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',  # In-memory database for tests
    }
}

# Password validation (disable for testing)
AUTH_PASSWORD_VALIDATORS = []

# Static files (optional for tests)
STATIC_URL = '/static/'

# Logging (basic configuration for test output)
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': 'DEBUG',
    },
}

# Channels setup (if used in tests)
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels.layers.InMemoryChannelLayer',
    },
}
```

---


## 4. Run Unit Tests

1. Run all tests using Django’s `manage.py` script:
   ```bash
   python djangostuff/manage.py test
   ```

2. Alternatively, if tests are organized in a `tests` folder, you can use:
   ```bash
   python -m unittest discover -s djangostuff/tests
   ```

---

## 5. Mocking and Dependencies

- **External Dependencies**:
  - All external services (e.g., databases, APIs) are mocked in the test environment.
  - Use libraries like `unittest.mock` or `pytest-mock` for simulating interactions with these services.

- **Database**:
  - The `test_settings.py` uses an in-memory SQLite database, so no PostgreSQL configuration is required.

---




# Running Tests

## Django Native Testing
Typically, you would use the following command to run all tests defined in your Django project:

### **Run All Tests**
```bash
python djangostuff/manage.py test
```

### **Optional Variations**
1. **Run Tests for a Specific App**:
   If you want to run tests only for a specific app (e.g., `dashboard`), use:
   ```bash
   python djangostuff/manage.py test dashboard
   ```

2. **Run a Specific Test Case**:
   If you want to run a specific test case (e.g., a test class or method in `test_views.py`), use:
   ```bash
   python djangostuff/manage.py test dashboard.tests.test_views
   ```

3. **Run Using `unittest` Directly**:
   If your tests are organized in a `tests` folder and follow Python's `unittest` structure, you can use:
   ```bash
   python -m unittest discover -s djangostuff/tests
   ```

---

### **Before Running Tests**
Ensure the following:
1. The virtual environment is activated.
2. The test settings are configured:
   ```bash
   export DJANGO_SETTINGS_MODULE=djangostuff.test_settings  # Linux/macOS
   set DJANGO_SETTINGS_MODULE=djangostuff.test_settings    # Windows
   ```

---

## PYTEST
If you prefer or plan to use **pytest** instead of Django's built-in `manage.py test`, you absolutely can. However, you’ll need to ensure your environment is configured correctly for pytest to work with Django.

Here’s how you can set up and run tests using pytest:

---

### **1. Install pytest and pytest-django**
First, make sure `pytest` and `pytest-django` are installed in your environment. Add these to your `requirements.txt` file if they’re not already included:

```bash
pip install pytest pytest-django
```

---

### **2. Configure pytest for Django**
pytest requires some configuration to know about your Django project settings. Create a file named `pytest.ini` in the root of your project (at the same level as the `djangostuff` directory), and include the following:

```ini
[pytest]
DJANGO_SETTINGS_MODULE = djangostuff.test_settings
python_files = tests.py test_*.py *_tests.py
```

- `DJANGO_SETTINGS_MODULE`: Points to your test settings file.
- `python_files`: Ensures pytest recognizes your test files.

---

### **3. Run Tests with pytest**
To run tests, simply execute:

```bash
pytest
```

- **Run Specific App Tests**: To test only a specific app (e.g., `dashboard`):
  ```bash
  pytest djangostuff/dashboard
  ```

- **Run a Specific Test File**: To test a specific file (e.g., `test_views.py`):
  ```bash
  pytest djangostuff/dashboard/tests/test_views.py
  ```

- **Run a Specific Test Function**: To run a specific test method (e.g., `test_login` in `test_views.py`):
  ```bash
  pytest djangostuff/dashboard/tests/test_views.py::test_login
  ```

---

### **4. Advantages of pytest**
- **Detailed Output**: pytest provides more informative output than `manage.py test`.
- **Plugins**: pytest has an extensive plugin ecosystem (e.g., `pytest-mock` for mocking).
- **Simpler Assertions**: pytest uses simple `assert` statements instead of Django’s `self.assert*` methods.

---

### **Key Considerations**
- **Test Settings**: Ensure `DJANGO_SETTINGS_MODULE` is set correctly in your environment or `pytest.ini`.
- **Test Discovery**: pytest discovers tests by default using the naming conventions in `pytest.ini`.

---

### **Final Command for pytest**
After setup, simply use:
```bash
pytest
```
