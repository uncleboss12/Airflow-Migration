### **Task 1. Create a Cloud Composer environment**

In this task, you create a Cloud Composer 3 environment.


On the Google Cloud console title bar, in the Search field, type cloud composer, and then click Composer.

On the Environments page, for Create environment, select Composer 3.

For Name, type ```cepf-dagify-migration-lab```.

For Location, select ```us-east4```.

Scroll to the bottom and click Show Advanced configuration.

For Environment bucket, select Custom bucket.

For Bucket name, click Browse, select ```qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket```, and then click Select.

Leave all other fields as default.

Click Create to create the environment.
NOTE: It can take up to 20 minutes to create the environment. You can continue with the lab and return to recheck this task before you deploy DAG to Cloud Composer.
Click Check my progress to verify the objective.
Please select the desired custom Cloud Storage bucket while creating the Cloud Composer Environment.
Create a Cloud Composer Environment
Please select the desired custom Cloud Storage bucket while creating the Cloud Composer Environment.

### **Task 2. Download and configure DAGify**
In this task, you download and set up the DAGify tool.

In the console, on the Navigation menu (Navigation menu icon), click Compute Engine > VM instances.

For the instance named lab-setup, in the Connect column, click SSH to open an SSH-in-browser terminal window.

In the terminal window, run the following command to clone the DAGify repository:
```bash
git clone https://github.com/GoogleCloudPlatform/dagify
```
![Screenshot 2025-06-21 12 46 08 PM](https://github.com/user-attachments/assets/3116b862-5802-4082-8f8c-0891474561d2)

Navigate to the folder dagify and execute the command below:
```bash
cd ~/dagify
make dagify-clean
``
Wait a minute for the Python Packages to be installed.

![Screenshot 2025-06-21 12 46 20 PM](https://github.com/user-attachments/assets/eeb5bb36-21b9-40f6-855f-c346b684f691)

Once completed, activate the Python virtual environment by running the command:
```bash
source venv/bin/activate
```
Verify DAGify setup by running the following command:
```bash
python ./DAGify.py -h
```
![Screenshot 2025-06-21 12 46 28 PM](https://github.com/user-attachments/assets/7914d09c-90b5-4823-96d1-dc5ef1ec797e)

Click Check my progress to verify the objective.
Download and configure DAGify

Task 3. Run the DAGify command with sample data in default mode
In this task, you use DAGify in default mode to convert the sample Control-M export file in XML format into a Python native DAG.

In the lab-setup terminal, view the Control-M job XML file 001-tfatf.xml by running the following command:
cat ./sample_data/control-m/001-tfatf.xml
Copied!
Output:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<DEFTABLE xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="Folder.xsd">
<SMART_FOLDER FOLDER_NAME="fx_fld_001">
        <!-- Folder 001, Application 001, Sub Application 001, Job 001 -->
        <JOB APPLICATION="fx_fld_001_app_001" SUB_APPLICATION="fx_fld_001_app_001_subapp_001" JOBNAME="fx_fld_001_app_001_subapp_001_job_001" DESCRIPTION="fx_fld_001_app_001_subapp_001_job_001_reports"  TASKTYPE="Command" CMDLINE="echo I am task A" PARENT_FOLDER="fx_fld_001">
                <OUTCOND NAME="fx_fld_001_app_001_subapp_001_job_001_ok" ODATE="ODAT" SIGN="+" />
        </JOB>
        <!-- Folder 001, Application 001, Sub Application 001, Job 002 -->
        <JOB APPLICATION="fx_fld_001_app_001" SUB_APPLICATION="fx_fld_001_app_001_subapp_001" JOBNAME="fx_fld_001_app_001_subapp_001_job_002" DESCRIPTION="fx_fld_001_app_001_subapp_001_job_002_reports"  TASKTYPE="Command" CMDLINE="echo I am task B"  PARENT_FOLDER="fx_fld_001">
                <INCOND NAME="fx_fld_001_app_001_subapp_001_job_001_ok" ODATE="ODAT" AND_OR="A" />
                <OUTCOND NAME="fx_fld_001_app_001_subapp_001_job_002_ok" ODATE="ODAT" SIGN="+" />
        </JOB>

        
</SMART_FOLDER>
```
Run DAGify using the default configuration:
```bash
python ./DAGify.py -s ./sample_data/control-m/001-tfatf.xml -o ./output/lab-output-task-3/
```
This converts the sample Control-M export file in XML format into a Python native DAG and stores it in the output format lab-output-task-3 folder.

This also stores the final DAG Python file within a subfolder of the name of the Control-M project folder, which in this case, is 001-tfatf.

List the output DAG files in the ./output/lab-output-task-3/001-tfatf/ directory by running the command below:
```bash
ls ./output/lab-output-task-3/001-tfatf/
```
Output:

```fx_fld_001.py```
This is your converted Python DAG and it's ready for deployment within Cloud Composer.

View the converted Python DAG file fx_fld_001.py by running the command below:
```bash
cat ./output/lab-output-task-3/001-tfatf/fx_fld_001.py
```
Output:
```python
# Apache Airflow Base Imports
from airflow import DAG
from airflow.decorators import task
from airflow.sensors.external_task import ExternalTaskMarker
from airflow.sensors.external_task import ExternalTaskSensor
import datetime
# Apache Airflow Custom & DAG/Task Specific Imports
from airflow.operators.bash import BashOperator

default_args = {
    'owner': 'airflow',
}

with DAG(
    dag_id="fx_fld_001",
    start_date=datetime.datetime(2024, 1, 1),
    schedule_interval="@daily",  # TIMEFROM not found, default schedule set to @daily,
    catchup=False,
) as dag:

    # DAG Tasks
    fx_fld_001_app_001_subapp_001_job_001 = BashOperator(
        task_id="fx_fld_001_app_001_subapp_001_job_001",
        bash_command="echo I am task A",
        trigger_rule="all_success",
        dag=dag,
    )
```
Click Check my progress to verify the objective.
Run DAGify command with sample data in the default mode

### **Task 4. Deploy DAG to Cloud Composer**
In this task, you check that the Cloud Composer environment is up and then deploy the converted DAG file into the environment.
![Screenshot 2025-06-21 12 56 53 PM](https://github.com/user-attachments/assets/c9e96896-bf8f-4d31-baec-72324bdb0eac)


In the console title bar, type cloud composer in the Search field and then click on Composer.
Validate that your Cloud Composer environment cepf-dagify-migration-lab is up and running and ready for use.

Note: If the Cloud Composer environment is still being created, wait for the creation process to complete.
From the environment list, click cepf-dagify-migration-lab, to open the Environment details page.

Click Open DAGs folder to access the DAGs folder in the Cloud Storage bucket qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket.

Here you can see the example default DAG airflow_monitoring.py that is issued when a composer environment is created.

Return to the terminal window.

Upload the new Python DAG fx_fld_001.py to the Cloud Storage bucket qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket associated with Cloud Composer environment using the gcloud storage cp command:
```bash
gcloud storage cp -r ~/dagify/output/lab-output-task-3/001-tfatf/* gs://qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket/dags/lab-output-task-3/
```
Inspect that the Python DAG file has been uploaded into the correct storage bucket.
```bash
gcloud storage ls gs://qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket/dags/lab-output-task-3/*
```
Output:

gs://qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket/dags/lab-output-task-3/fx_fld_001.py
Go back to the Environment details page for cepf-dagify-migration-lab environment.

Click Open Airflow UI.
![Screenshot 2025-06-21 1 00 41 PM](https://github.com/user-attachments/assets/6a15ef6f-e0c1-431d-a395-7cf693fccc5b)


Authenticate using student-00-18a02f1357d9@qwiklabs.net user.

Verify that ```fx_fld_001``` DAG is visible in the DAGs list.

NOTE: It can take up to 2-3 minutes for uploaded DAGs to Synchronize into Google Cloud Composer
Explore Cloud Composer
In this task you explore the detailed view of fx_fld_001 in Cloud Composer.

Click ```fx_fld_001``` to open the detailed view of the DAG.

Click **Code** to view the original converted Python source code.

Click **Graph** to view the graph of all tasks and dependencies.

Click Check my progress to verify the objective.
Deploy DAG to Cloud Composer

### **Task 5. Run the DAGify command with sample data using a DAG divider**
In this task, you use DAGify with the DAG divider flag ( -d) to divide control-m workflow into multiple DAG files based on the XML Key APPLICATION.

Switch to the lab-setup terminal window.

View Control-M job XML file 002-tftf.xml by running the command:
```bash
cat ./sample_data/control-m/002-tftf.xml
```
![Screenshot 2025-06-21 1 01 21 PM](https://github.com/user-attachments/assets/d572ecc1-ea3b-4df2-95d5-a2496916a1a5)

Output:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<DEFTABLE xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="Folder.xsd">
<SMART_FOLDER FOLDER_NAME="fx_fld_001">
        <!-- Folder 001, Application 001, Sub Application 001, Job 001 -->
        <JOB APPLICATION="fx_fld_001_app_001" SUB_APPLICATION="fx_fld_001_app_001_subapp_001" JOBNAME="fx_fld_001_app_001_subapp_001_job_001" DESCRIPTION="fx_fld_001_app_001_subapp_001_job_001_reports"  TASKTYPE="Command" CMDLINE="" PARENT_FOLDER="fx_fld_001">
                <OUTCOND NAME="fx_fld_001_app_001_subapp_001_job_001_ok" ODATE="ODAT" SIGN="+" />
        </JOB>
        <!-- Folder 001, Application 001, Sub Application 001, Job 002 -->
        <JOB APPLICATION="fx_fld_001_app_001" SUB_APPLICATION="fx_fld_001_app_001_subapp_001" JOBNAME="fx_fld_001_app_001_subapp_001_job_002" DESCRIPTION="fx_fld_001_app_001_subapp_001_job_002_reports"  TASKTYPE="Command" CMDLINE=""  PARENT_FOLDER="fx_fld_001">
                <INCOND NAME="fx_fld_001_app_001_subapp_001_job_001_ok" ODATE="ODAT" AND_OR="A" />
                <OUTCOND NAME="fx_fld_001_app_001_subapp_001_job_002_ok" ODATE="ODAT" SIGN="+" />
        </JOB>

       
</SMART_FOLDER>
```
![Screenshot 2025-06-21 1 02 36 PM](https://github.com/user-attachments/assets/636c8311-10c2-4372-9c29-924334f87bef)


Run the DAGify command using the DAG divider flag:
```bash
python ./DAGify.py -s ./sample_data/control-m/002-tftf.xml -o ./output/lab-output-task-5/ -d APPLICATION
```
This converts the sample Control-M export file in XML format into a Python native DAG and stores it in the output format lab-output-task-5 folder.
![Screenshot 2025-06-21 1 02 43 PM](https://github.com/user-attachments/assets/f4551caf-65cf-46ac-93f2-c28e30be6203)

![Screenshot 2025-06-21 1 02 49 PM](https://github.com/user-attachments/assets/da8407c8-cf99-4ef3-bde4-aca11df6b2fd)


The final DAG Python file is stored within a subfolder in the Control-M project folder. For this case, that folder is 002-tftf.

List the output DAG files by running the command:
```bash
ls ./output/lab-output-task-5/002-tftf/
```
![Screenshot 2025-06-21 1 03 10 PM](https://github.com/user-attachments/assets/0d6ad2dc-815c-47f7-8f74-5e04903037e2)

You should see multiple Python files created in the directory ./output/lab-output-task-5/002-tftf/ similar to following:

Output:

fx_fld_001_app_001.py  fx_fld_001_app_002.py
These are new Python DAGs that are ready to deploy within Cloud Composer.

View a converted Python DAG file, in this case fx_fld_001_app_001.py, by running the command:
```bash
cat ./output/lab-output-task-5/002-tftf/fx_fld_001_app_001.py
```
![Screenshot 2025-06-21 1 03 57 PM](https://github.com/user-attachments/assets/0cb36931-a69d-41cc-973e-2e30d58a453d)

Output:

```python
# Apache Airflow Base Imports
from airflow import DAG
from airflow.decorators import task
from airflow.sensors.external_task import ExternalTaskMarker
from airflow.sensors.external_task import ExternalTaskSensor
import datetime
# Apache Airflow Custom & DAG/Task Specific Imports
from airflow.operators.bash import BashOperator

default_args = {
    'owner': 'airflow',
}

with DAG(
    dag_id="fx_fld_001_app_001",
    start_date=datetime.datetime(2024, 1, 1),
    schedule_interval="@daily",  # TIMEFROM not found, default schedule set to @daily,
    catchup=False,
) as dag:

    # DAG Tasks
    fx_fld_001_app_001_subapp_001_job_001 = BashOperator(
        task_id="fx_fld_001_app_001_subapp_001_job_001",
        bash_command="",
        trigger_rule="all_success",
        dag=dag,
    )
```
    ...
Deploy the converted Python DAG files by running the gcloud storage cp command to move them to the Cloud Storage bucket associated with the Cloud Composer environment:
```bash
gcloud storage cp -r ~/dagify/output/lab-output-task-5/002-tftf/* gs://qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket/dags/lab-output-task-5/
```
Verify that the Python DAG files are uploaded into the correct storage bucket by running the command:
```bash
gcloud storage ls gs://qwiklabs-gcp-01-a52d1a55ade6-dagify-bucket/dags/lab-output-task-5/*
```
![Screenshot 2025-06-21 1 04 34 PM](https://github.com/user-attachments/assets/1ec99a0a-14ac-470d-8ee0-b14a1f28722e)

![Screenshot 2025-06-21 1 05 02 PM](https://github.com/user-attachments/assets/6c9a06dd-7864-4351-8f85-82cba3020a38)

![Screenshot 2025-06-21 1 05 23 PM](https://github.com/user-attachments/assets/2024fb44-4da8-41f2-ae9a-1e78cd8296b5)

![Screenshot 2025-06-21 1 05 43 PM](https://github.com/user-attachments/assets/06f80ee3-185a-4b5a-865c-4dca88d9a728)



