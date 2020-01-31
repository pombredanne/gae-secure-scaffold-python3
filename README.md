# Secure GAE Scaffold (Python 3)

## Introduction

Please note: this is not an official Google product.

This is a secure scaffold package designed primarily to work with
Google App Engine (although it is not limited to this).

It is built using Python 3 and Flask.

The scaffold provides the following basic security guarantees by default through
a flask app factory found in `secure_scaffold/factories.py`. This app will:

1. Set assorted security headers (Strict-Transport-Security, X-Frame-Options,
   X-XSS-Protection, X-Content-Type-Options, Content-Security-Policy) with
   strong default values to help avoid attacks like Cross-Site Scripting (XSS)
   and Cross-Site Script Inclusion.
1. Verify XSRF tokens by default on authenticated requests using any verb other
   that GET, HEAD, or OPTIONS.


## Usage

### Installation

This project can be installed via

`pip install https://github.com/davidwtbuxton/gae-secure-scaffold-python/archive/py37-scaffold.zip`


### Starting a new project

Install the Cookiecutter utility. Then create a new empty project:

    pip install cookiecutter
    cookiecutter https://github.com/davidwtbuxton/gae-secure-scaffold-python.git --checkout py37-scaffold

This prompts for the App Engine project name and creates a directory with that name. Inside the directory is the skeleton of a basic app for Python 3 on App Engine standard.


### App Factory

To use the secure scaffold in your app, use our app generator.

```python
from secure_scaffold import factories

app = factories.AppFactory().generate()
```

This will automatically set all the needed CSP headers.


### XSRF

To enable XSRF protection add the decorator to the endpoints you need it for.
This needs to be set *after* the route decorator
e.g.

```python
from secure_scaffold import factories
from secure_scaffold import xsrf

app = factories.AppFactory().generate()

@app.route('/', methods=['GET', 'POST'])
@xsrf.xsrf_protected()
def index():
    return 'Hello World!'
```


We use Flask Sessions for XSRF. A random SECRET_KEY is created and saved to the datastore.


### Configuring Flask and the SECRET_KEY setting

Set the environment variable FLASK_SETTINGS_FILENAME to the name of a Python file. Configuration defaults are loaded from "secure_scaffold.settings". The configuration is loaded when you create the Flask application.

To customize settings, create a Python module that overrides the default settings, and point FLASK_SETTINGS_FILENAME to the file name. For example, here's how you can change the name for the session cookie created by Flask:

    # myappconfig.py
    SESSION_COOKIE_NAME = 'myapp_session'

Then set the FLASK_SETTINGS_FILENAME:

    # app.yaml
    runtime: python37

    env_variables:
      FLASK_SETTINGS_FILENAME = "myappconfig.py"

The SECRET_KEY setting is read from the datastore when the application starts. If there is no setting, a random key is created and saved.


### Authentication for IAP users

This is available at `secure_scaffold.contrib.appengine.users`. It provides a `User`
class which has a few useful methods providing the details of the current user.
It also provides `requires_auth` and `requires_admin` decorators which enforce the need
for authentication and admin rights respectively on the views they are applied to.

These work almost identically to how they do in the first generation App Engine APIs.

To use these you will need to enable IAP on your App Engine instance. This provides the app with the correct headers for this functionality.


## Scaffold Development

### Structure:

`secure_scaffold/`
- Top level directory

`secure_scaffold/contrib`
- Contains non-essential but useful libraries.
- Holds several alternatives to App Engine APIs
which are no longer available in second generation instances

`secure_scaffold/tests`
- Tests for the secure scaffold

`secure_scaffold/factories.py`
- The main Flask app factory that applies the security defaults
- See App Factory below on how to use this

`secure_scaffold/settings.py`
- Security settings
- Defines our CSP headers and other specifics


### Dependency Setup

We recommend setting up a virtual env to install dependencies:

    python3 -m venv env
    source env/bin/activate
    pip install --requirement dev_requirements.txt


### Testing

Install the Google Cloud SDK: https://cloud.google.com/sdk/docs

Once the SDK is installed, install the datastore emulator:

    gcloud components install beta
    gcloud components install cloud-datastore-emulator

To run tests:

    pytest


## Third Party Credits

- Flask - https://github.com/pallets/flask
- flask-seasurf - https://github.com/maxcountryman/flask-seasurf
- flask-talisman - https://github.com/GoogleCloudPlatform/flask-talisman
