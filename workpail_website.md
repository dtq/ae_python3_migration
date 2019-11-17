# Workpail.com: Python2 to Python3 App Engine migration
Steps for migrating the [workpail.com](https://www.workpail.com) website from App Engine Python 2 standard environment (using webapp) to Python 3 
standard environment (using Flask).

There are two major parts:
1) First we migrate as many components and dependencies and libraries to Python3 
compatible equivalents (e.g. migrate tests to pytest and migrate the datastore 
emulator to the new Firestore emulator).
2) Then we migrate our code base from Python2 to Python3, and switch to the
App Engine Python3 standard environment.



[✓] Part 1: Migrate components, libraries and dependencies
----------------------------------------------------------


### [✓] Create requirements.txt and requirements_dev.txt
requirements.txt should look like this, plus any other project-specific
dependencies:
```
Flask==1.1.1
Jinja2
```
requirements_dev.txt:
```
pytest
```

### [✓] Setup a python2 virtual environment
- Create a Python2 virtual environment named `venv_python2` or similar, using virtualenv
- Activate the environment: run `source venv_python2/bin/activate`
- Install dependencies: run `pip install -r requirements.txt -r requirements_dev.txt`
- More info on using virtualenv: https://docs.python-guide.org/dev/virtualenvs/


### [✓] Replace `webapp2` with `flask`
- Replace webapp2 with flask in app.yaml libraries
  - This is temporary until we fully migrate code to Python3
- Replace webapp2 with flask in code


### [✓] Run Flask directly (instead of using dev_appserver.py)
Use the following commands to run Flask in debug mode (to enable hot-reload). This assumes app's entry point is
main.py.
```shell script
FLASK_APP=main.py FLASK_ENV=development flask run
```
Or in a Makefile:
```shell script
export FLASK_APP=main.py;        \
export FLASK_ENV=development;    \
flask run
```
- https://cloud.google.com/appengine/docs/standard/python3/testing-and-deploying-your-app
- https://flask.palletsprojects.com/en/1.1.x/quickstart/


### [✓] Replace `logging` with `app.logger`?
- https://flask.palletsprojects.com/en/1.1.x/quickstart/


### [✓] Tests
- Replace `unittest` with `pytest`
  - https://www.patricksoftwareblog.com/testing-a-flask-application-using-pytest/
- Create fixture for flask app
  - https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/appengine/standard/flask/hello_world/main_test.py
- Create fixture for datastore emulator
  - https://cloud.google.com/appengine/docs/standard/python/tools/migrate-cloud-datastore-emulator
- Run `pytest` or `pytest -v` from the project (or tests) directory
- Test manually
- Optional:
  - Enable pytest plugin for IDE (PyCharm has built in support, VS Code has an available plugin). This allows
  you to run single tests directly from the IDE.


### [✓] Deploy
- Deploy to production as a sanity check to ensure everything still works



[✓] Part 2: Migrate from Python2 to Python 3
--------------------------------------------


### [✓] Create a Python3 virtual environment
- Run `python3 -m venv env` to create a new virtual environment named `env`
- Run `source env/bin/activate` to activate the environment
- Run `pip install -r requirements.txt -r requirements_dev.txt` to install dependencies


### [✓] Run Flask using Python3
Use the following commands to run Flask in debug mode (to enable hot-reload). This assumes app's entry point is
main.py.
```shell script
FLASK_APP=main.py FLASK_ENV=development python3 -m flask run
```
Or in a Makefile:
```shell script
export FLASK_APP=main.py;        \
export FLASK_ENV=development;    \
python3 -m flask run
```
- https://cloud.google.com/appengine/docs/standard/python3/testing-and-deploying-your-app
- https://flask.palletsprojects.com/en/1.1.x/quickstart/


### [✓] Migrate project code from Python2 to Python3
- Use python-future's `futurize` module to convert Python2 to Python3
  - https://python-future.org/automatic_conversion.html
  - Migrate tests: `futurize tests/**/*.py -w`
  - Run `pytest` to ensure all tests still pass
  - Migrate rest of code in two stages:
    - Stage 1: `futurize --stage1 mypackage/**/*.py`
    - Run `pytest` to ensure everything still works
    - Stage 2: `futurize --stage2 mypackage/**/*.py`
    - Run `pytest` to ensure everything still works


### [✓] Convert app.yaml from Python2 to Python3 standard environment
- General Python3 app.yaml info: https://cloud.google.com/appengine/docs/standard/python3/config/appref
- Migrating app.yaml: https://cloud.google.com/appengine/docs/standard/python/migrate-to-python3/config-files
  - Change set `runtime: python37`, remove libraries section, etc.
  - Change `url: .*` handler to use `script: auto`


### [✓] Run using dev_appserver.py to ensure that new app.yaml at least works on dev machine
- Switch to python2 virtual environment (e.g. `source venv_python27/bin/activate`)
- Run `dev_appserver.py .`


### [✓] Ensure .gcloudignore is up to date
- Avoid uploading unnecessary directories and files (e.g. virtual environment directories)

### [✓] Deploy
- Deploy to production and ensure everything still works
