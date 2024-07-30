Sure, I'll provide the Windows commands and update the script execution accordingly.

### Project Setup Plan

#### 1. Create Project Directory and Initialize Git

```sh
mkdir flask_test_project
cd flask_test_project
git init
```

#### 2. Set Up Virtual Environment

```sh
python -m venv venv
venv\Scripts\activate
```

#### 3. Install Dependencies

```sh
pip install flask gunicorn flask_sqlalchemy flask_migrate python-dotenv pytest
```

#### 4. Create Project Structure

```sh
mkdir app scripts docs
type NUL > app\__init__.py
type NUL > app\models.py
type NUL > app\routes.py
type NUL > config.py
type NUL > .env
type NUL > Procfile
type NUL > README.md
```

#### 5. Define Application and Configuration

1. **`app/__init__.py`**

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from config import DevelopmentConfig, TestingConfig, StagingConfig, ProductionConfig

db = SQLAlchemy()
migrate = Migrate()

def create_app(config_class=DevelopmentConfig):
    app = Flask(__name__)
    app.config.from_object(config_class)

    db.init_app(app)
    migrate.init_app(app, db)

    with app.app_context():
        from app import routes, models
        db.create_all()

    return app
```

2. **`app/models.py`**

```python
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return f'<User {self.username}>'
```

3. **`app/routes.py`**

```python
from flask import Flask, render_template
from app import create_app

app = create_app()

@app.route('/')
def index():
    return "Hello, World!"
```

4. **`config.py`**

```python
import os
from dotenv import load_dotenv

basedir = os.path.abspath(os.path.dirname(__file__))
load_dotenv(os.path.join(basedir, '.env'))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'dev.db')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'test.db')

class StagingConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('STAGING_DATABASE_URL') or \
        'postgresql://yourusername:yourpassword@yourstagingdburl/yourdatabase'

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'postgresql://yourusername:yourpassword@yourproductiondburl/yourdatabase'
```

5. **`.env`**

```plaintext
SECRET_KEY=your_secret_key
DEV_DATABASE_URL=sqlite:///${PWD}/dev.db
TEST_DATABASE_URL=sqlite:///${PWD}/test.db
STAGING_DATABASE_URL=postgresql://yourusername:yourpassword@yourstagingdburl/yourdatabase
DATABASE_URL=postgresql://yourusername:yourpassword@yourproductiondburl/yourdatabase
```

6. **`Procfile`**

```sh
echo web: gunicorn app.routes:app > Procfile
```

#### 6. Initialize Database and Migrations

1. **Initialize Migration Directory**

```sh
venv\Scripts\activate
flask db init
```

2. **Create Initial Migration**

```sh
flask db migrate -m "Initial migration."
```

3. **Apply Initial Migration**

```sh
flask db upgrade
```

#### 7. CI/CD Pipeline Setup

1. **Set Up GitHub Actions (CI/CD)**

Create a `.github/workflows/ci-cd.yml` file:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:

    runs-on: windows-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.8

    - name: Install dependencies
      run: |
        python -m venv venv
        venv\Scripts\activate
        pip install -r requirements.txt

    - name: Run tests
      run: |
        venv\Scripts\activate
        flask db upgrade
        pytest
```

#### 8. Housekeeping Scripts

1. **Backup Script for SQLite**

Create `scripts/backup_sqlite.bat`:

```batch
@echo off
set TIMESTAMP=%date:~10,4%%date:~4,2%%date:~7,2%_%time:~0,2%%time:~3,2%%time:~6,2%
set BACKUP_DIR=backups
set BACKUP_FILE=dev_backup_%TIMESTAMP%.db

:: Create backup directory if it doesn't exist
if not exist %BACKUP_DIR% mkdir %BACKUP_DIR%

:: Copy the database file to the backup directory
copy dev.db %BACKUP_DIR%\%BACKUP_FILE%

echo Backup created: %BACKUP_DIR%\%BACKUP_FILE%
```

Make the script executable:

```sh
attrib +x scripts\backup_sqlite.bat
```

2. **Sanity Check Script**

Create `scripts/sanity_check.py`:

```python
from app import db
from app.models import User

try:
    # Check database connection
    db.session.execute('SELECT 1')
    print("Sanity check passed: Database connection is working")

    # Check model schema
    columns = [column.name for column in User.__table__.columns]
    assert 'username' in columns, "Column 'username' not found in 'User' table"
    print("Sanity check passed: 'username' column is present")
except Exception as e:
    print(f"Sanity check failed: {e}")
```

#### 9. Deploy to Heroku

1. **Create Heroku Apps**

```sh
heroku create your-app-name-staging
heroku create your-app-name
```

2. **Add PostgreSQL Add-Ons**

```sh
heroku addons:create heroku-postgresql:hobby-dev --app your-app-name-staging
heroku addons:create heroku-postgresql:hobby-dev --app your-app-name
```

3. **Deploy to Heroku Staging**

```sh
git remote add heroku-staging https://git.heroku.com/your-app-name-staging.git
git push heroku-staging master
```

4. **Perform Final Testing on Staging**

```sh
heroku run python scripts/sanity_check.py --app your-app-name-staging
```

5. **Deploy to Heroku Production**

```sh
git remote add heroku https://git.heroku.com/your-app-name.git
git push heroku master
```

6. **Perform Final Sanity Check on Production**

```sh
heroku run python scripts/sanity_check.py --app your-app-name
```

### Scenarios in the Life of a Developer

#### Scenario 1: Local Development

**Task:** Add a new feature and test it locally.

**Steps:**
1. **Develop the Feature:**
   - Write code and add new features in the development environment using SQLite.
   
   ```sh
   set FLASK_ENV=development
   flask run
   ```

2. **Run Local Development Server:**
   - Start the local server to test the new feature.
   
   ```sh
   flask run
   ```

3. **Perform Local Sanity Checks:**
   - Verify the new feature works as expected in the development environment.

4. **Commit Changes to Version Control:**
   - Use Git to commit your changes.
   
   ```sh
   git add .
   git commit -m "Add new feature"
   ```

#### Scenario 2: Local Testing

**Task:** Run automated tests in the local testing environment.

**Steps:**
1. **Switch to Testing Configuration:**
   - Set the environment to testing to use the `TestingConfig`.

2. **Run Automated Tests:**
   - Execute your test suite to ensure all tests pass in isolation.
   
   ```sh
   set FLASK_ENV=testing
   pytest
   ```

3. **Perform Local Sanity Checks:**
   - Verify database integrity and check model schemas.
   
   ```python
   # Run sanity checks script
   python scripts/sanity_check.py
   ```

4. **Fix Any Issues:**
   - Address any test failures or issues found during sanity checks.
   
   ```sh
   # Edit code to fix issues
   flask db upgrade
   pytest
   ```

#### Scenario 3: Testing on Staging

**Task:** Test the new feature in a staging environment.

**Steps:**
1. **Push Code to Heroku Staging:**
   - Deploy your changes to the staging environment for further testing.
   
   ```sh
   git push heroku-staging

 master
   ```

2. **Run Sanity Checks on Staging:**
   - Ensure the staging environment is set up correctly and the feature works as expected.
   
   ```sh
   heroku run python scripts/sanity_check.py --app your-app-name-staging
   ```

3. **Perform Final Testing:**
   - Conduct comprehensive testing in the staging environment to catch any issues before going live.
   
   ```sh
   heroku open --app your-app-name-staging
   ```

#### Scenario 4: Deploying to Production

**Task:** Deploy the tested feature to production.

**Steps:**
1. **Final Verification in Staging:**
   - Ensure all tests pass and the feature is verified in staging.

2. **Push Code to Heroku Production:**
   - Deploy the final version of the application to the production environment.
   
   ```sh
   git push heroku master
   ```

3. **Run Final Sanity Checks on Production:**
   - Perform sanity checks to ensure everything is functioning correctly in production.
   
   ```sh
   heroku run python scripts/sanity_check.py --app your-app-name
   ```

4. **Monitor the Production App:**
   - Monitor logs and performance to ensure the deployment is successful.
   
   ```sh
   heroku logs --tail --app your-app-name
   ```

#### Scenario 5: Handling Issues

**Task:** Fix an issue identified in production.

**Steps:**
1. **Identify the Issue:**
   - Diagnose the problem based on logs and error messages.

2. **Fix the Issue Locally:**
   - Develop the fix in the local development environment.
   
   ```sh
   set FLASK_ENV=development
   flask run
   ```

3. **Test the Fix Locally:**
   - Run tests and sanity checks in the local testing environment to ensure the fix works.
   
   ```sh
   set FLASK_ENV=testing
   pytest
   python scripts/sanity_check.py
   ```

4. **Deploy Fix to Staging:**
   - Push the fix to the staging environment and perform thorough testing.
   
   ```sh
   git push heroku-staging master
   heroku run python scripts/sanity_check.py --app your-app-name-staging
   ```

5. **Deploy Fix to Production:**
   - Once verified in staging, deploy the fix to production and perform final checks.
   
   ```sh
   git push heroku master
   heroku run python scripts/sanity_check.py --app your-app-name
   ```

#### Scenario 6: Backup and Recovery

**Task:** Backup the database and restore if necessary.

**Steps:**
1. **Backup SQLite Database Locally:**
   - Run the backup script to create a backup of the local SQLite database.
   
   ```sh
   scripts\backup_sqlite.bat
   ```

2. **Backup PostgreSQL Database on Heroku:**
   - Create a backup of the PostgreSQL database on Heroku.
   
   ```sh
   heroku pg:backups:capture --app your-app-name
   ```

3. **Restore PostgreSQL Database on Heroku:**
   - If needed, restore the database from a backup.
   
   ```sh
   heroku pg:backups:restore b001 DATABASE_URL --app your-app-name
   ```