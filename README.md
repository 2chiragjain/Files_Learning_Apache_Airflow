# Files_Learning_Apache_Airflow

This script automates the installation, configuration, and management of **Apache Airflow** in a Codespaces environment.  

---

## Set Airflow Home Directory

```bash
export AIRFLOW_HOME="/workspaces/Files_Learning_Apache_Airflow/airflow"
echo "Airflow home is set to: $AIRFLOW_HOME"

Sets the AIRFLOW_HOME environment variable, which tells Airflow where to store DAGs, configs, and logs.
Instructor Note

echo "${BPurple}Note From Instructor: ${NC} This script will install Airflow ${URed}WITH${NC} the example dags. If you want Airflow ${URed}WITHOUT${NC} the example dags please run this command after this script finishes, before starting airflow: sed -i -e '/load_examples =/ s/= .*/= False/' ${AIRFLOW_HOME}/airflow.cfg"
sleep 3
echo "${BRed}This script is also not a substitute for following the Hands on Data Engineering LinkedIn Learning Course directly.${NC}"
sleep 3

Displays important information about example DAGs and course guidance.
Set Airflow Version and Python Version

AIRFLOW_VERSION=2.10.2
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
echo "This script will install Airflow version $AIRFLOW_VERSION, with Python $PYTHON_VERSION."
sleep 3

Defines which Airflow version to install and detects the Python version in use.
Install Airflow with Version Constraints

CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

Installs Airflow with the correct dependencies for the Python version.
Initialize Airflow Database

echo "Running DB Migrate"
airflow db init
airflow info
sleep 5

Initializes the Airflow metadata database and prints Airflow info for verification.
Update airflow.cfg Settings

sed -i -e '/load_examples =/ s/= .*/= False/' ${AIRFLOW_HOME}/airflow.cfg
sed -i -e '/dag_dir_list_interval =/ s/= .*/= 2/' ${AIRFLOW_HOME}/airflow.cfg
sed -i -e '/worker_refresh_batch_size =/ s/= .*/= 0/' ${AIRFLOW_HOME}/airflow.cfg
sed -i -e '/worker_refresh_interval =/ s/= .*/= 0/' ${AIRFLOW_HOME}/airflow.cfg
sed -i -e '/workers =/ s/= .*/= 2/' ${AIRFLOW_HOME}/airflow.cfg
sed -i -e '/expose_config =/ s/= .*/= True/' ${AIRFLOW_HOME}/airflow.cfg

Updates common Airflow settings such as example DAGs loading, scheduler intervals, worker settings, and exposing the config in the UI.
Generate and Configure webserver_config.py

airflow config generate -c ${AIRFLOW_HOME}/webserver_config.py
cp $(python -c "import os, airflow; print(os.path.join(os.path.dirname(airflow.__file__), 'www', 'app.py'))") ${AIRFLOW_HOME}/webserver_config.py

sed -i -e '/WTF_CSRF_ENABLED =/ s/= .*/= False/' ${AIRFLOW_HOME}/webserver_config.py

Generates a webserver configuration file and disables CSRF (for Codespaces forwarding URLs).
Create Admin User

airflow users create --username admin --firstname Firstname --lastname Lastname --role Admin --email admin@example.org --password password

Creates a default admin user for logging into the Airflow UI.
Start Airflow Webserver and Scheduler

airflow webserver -D
airflow scheduler -D
airflow webserver -p 8080 -D

Starts the webserver and scheduler in the background.
Kill Running Airflow Processes

WEBSERVER_PID=$(ps -ef | grep -i webserver | grep master | grep -v grep | awk '{print $2}')
echo "Killing webserver running with pid $WEBSERVER_PID"
kill $WEBSERVER_PID

SCHEDULER_PID=$(ps -ef | grep -i scheduler | grep -v grep | grep -v DagFileProcessorManager | awk '{print $2}')
echo "Killing scheduler running with pid $SCHEDULER_PID"
kill $SCHEDULER_PID
echo "Airflow stopped."

Stops any currently running Airflow webserver and scheduler processes.
Reset Airflow Database

airflow db reset

Resets the Airflow metadata database (removes all DAG runs and task instances).
Configure Airflow Environment for Codespaces

export AIRFLOW__WEBSERVER__ALLOWED_HOSTS="*"
export AIRFLOW__WEBSERVER__ENABLE_PROXY_FIX=True
export AIRFLOW__CORE__LOAD_EXAMPLES=False

Allows Airflow to work behind Codespaces’ forwarded URLs and disables example DAGs.
Restart Airflow Services

airflow webserver -D
airflow scheduler -D

Starts Airflow again with updated settings.
✅ Notes

    This script is for development/testing purposes only.

    CSRF is disabled to allow Codespaces forwarded URLs.

    Example DAGs are disabled unless explicitly enabled.

    Use sh run_airflow.sh to start Airflow instead of running these commands manually.# Files_Learning_Apache_Airflow

export AIRFLOW_HOME="/workspaces/Files_Learning_Apache_Airflow/airflow"

echo "Airflow home is set to: $AIRFLOW_HOME"


echo "${BPurple}Note From Instructor: ${NC} This script will install Airflow ${URed}WITH${NC} the example dags. If you want Airflow ${URed}WITHOUT${NC} the example dags please run this command after this script finishes, before starting airflow: sed -i -e '/load_examples =/ s/= .*/= False/' ${AIRFLOW_HOME}/airflow.cfg"

sleep 3
echo "${BRed}This script is also not a substitute for following the Hands on Data Engineering LinkedIn Learning Course directly.${NC}"
sleep 3

AIRFLOW_VERSION=2.10.2
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
echo "This script will install Airflow version $AIRFLOW_VERSION, with Python $PYTHON_VERSION."
sleep 3
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

echo "Running DB Migrate"

# Init the airflow DB
airflow db init

# Run Airflow Info
airflow info

# Give airflow enough time to finish writing it's config files with sleep 5
sleep 5
echo "Setting value of ${BRed}load_examples${NC} in airflow.cfg to ${BPurple}True${NC}"
sed -i -e '/load_examples =/ s/= .*/= False/' ${AIRFLOW_HOME}/airflow.cfg
echo "Setting value of ${BRed}dag_dir_list_interval${NC} in airflow.cfg to ${BPurple}2${NC}"
sed -i -e '/dag_dir_list_interval =/ s/= .*/= 2/' ${AIRFLOW_HOME}/airflow.cfg
echo "Setting value of ${BRed}worker_refresh_batch_size${NC} in airflow.cfg to ${BPurple}0${NC}"
sed -i -e '/worker_refresh_batch_size =/ s/= .*/= 0/' ${AIRFLOW_HOME}/airflow.cfg
echo "Setting value of ${BRed}worker_refresh_interval${NC} in airflow.cfg to ${BPurple}0${NC}"
sed -i -e '/worker_refresh_interval =/ s/= .*/= 0/' ${AIRFLOW_HOME}/airflow.cfg
echo "Setting value of ${BRed}workers${NC} in airflow.cfg to ${BPurple}2${NC}"
sed -i -e '/workers =/ s/= .*/= 2/' ${AIRFLOW_HOME}/airflow.cfg
echo "Setting value of ${BRed}expose_config${NC} in airflow.cfg to ${BPurple}True${NC}"
sed -i -e '/expose_config =/ s/= .*/= True/' ${AIRFLOW_HOME}/airflow.cfg

airflow config generate -c ${AIRFLOW_HOME}/webserver_config.py
cp $(python -c "import os, airflow; print(os.path.join(os.path.dirname(airflow.__file__), 'www', 'app.py'))") ${AIRFLOW_HOME}/webserver_config.py

# Update CSRF For Webserver
echo "Setting value of ${BRed}WTF_CSRF_ENABLED${NC} in webserver_config.py to ${BPurple}False${NC}"
sed -i -e '/WTF_CSRF_ENABLED =/ s/= .*/= False/' ${AIRFLOW_HOME}/webserver_config.py

# Create User
airflow users create --username admin --firstname Firstname --lastname Lastname --role Admin --email admin@example.org --password password

echo "${BRed}This has only installed airflow, to run it, you will need to run "sh run_airflow.sh"${NC}"

echo "AIRFLOW_HOME is set to: $AIRFLOW_HOME"

airflow webserver -D
airflow scheduler -D


airflow webserver -p 8080 -D

export AIRFLOW_HOME="/workspaces/Files_Learning_Apache_Airflow/airflow"
echo "AIRFLOW_HOME is set to: $AIRFLOW_HOME"

# Get Actively Running PID of webserver
WEBSERVER_PID=$(ps -ef | grep -i webserver | grep master | grep -v grep | awk '{print $2}')
echo "Killing webserver running with pid $WEBSERVER_PID"
kill $WEBSERVER_PID

# Get Actively Running PID of scheduler
SCHEDULER_PID=$(ps -ef | grep -i scheduler | grep -v grep | grep -v DagFileProcessorManager | awk '{print $2}')
echo "Killing scheduler running with pid $SCHEDULER_PID"
kill $SCHEDULER_PID

echo "Airflow stopped."

airflow db reset

export AIRFLOW__WEBSERVER__ALLOWED_HOSTS="*"
export AIRFLOW__WEBSERVER__ENABLE_PROXY_FIX=True
export AIRFLOW__CORE__LOAD_EXAMPLES=False
airflow webserver -D
airflow scheduler -D
