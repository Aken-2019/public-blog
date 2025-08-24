---
title: "[Read] Airflow Best Practices"
date: 2022-08-28T11:38:51+08:00
draft: false
---

My team is using airflow to deploy jobs. I found [this official document](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#) when I was searching for test automation in Airflow. After reading this document, I can tell a few bad practices my team is doing：
- Codes are not optimized, especially using external packages such as `pandas` and `numpy`.
- There is no unit test or integration test.
- Airflow variable is misused. We use it to store DAG-specific data but it's not recommended. We should use something else, better if we can put it into the version control system.
- The upgrade/downgrade process is not automated, partly due to the lack of test automation.
- The database is not maintained. We should make backups and clean the operational ones.
- We can use the `pendulum` module to process the time calculation as our jobs are time-zone specific.

## Highlights
I copy and paste my highlights from the document below. You can skip it if you will read the full document.


### Communication
The **tasks should also not store any authentication parameters** such as passwords or token inside them. Where at all possible, **use Connections** to store data securely in Airflow backend and retrieve them using a unique connection id.

https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#communication

### Connections & Hooks
A Connection is essentially set of parameters - such as username, password and hostname - along with the type of system that it connects to, and a unique name, called the conn_id.

https://airflow.apache.org/docs/apache-airflow/stable/concepts/connections.html

### Top Level Python Code
Bad example:
```
import pendulum

from airflow import DAG
from airflow.operators.python import PythonOperator

import numpy as np  # <-- THIS IS A VERY BAD IDEA! DON'T DO THAT!

with DAG(
    dag_id="example_python_operator",
    schedule_interval=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=["example"],
) as dag:

    def print_array():
        """Print Numpy array."""
        a = np.arange(15).reshape(3, 5)
        print(a)
        return a

    run_this = PythonOperator(
        task_id="print_the_context",
        python_callable=print_array,
    )
```

Good example:
```
import pendulum

from airflow import DAG
from airflow.operators.python import PythonOperator

with DAG(
    dag_id="example_python_operator",
    schedule_interval=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=["example"],
) as dag:

    def print_array():
        """Print Numpy array."" The tasks should also not store any authentication parameters such as passwords or token inside them. Where at all possible, use Connections to store data securely in Airflow backend and retrieve them using a unique connection id.
    https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#communication"
        import numpy as np  # <- THIS IS HOW NUMPY SHOULD BE IMPORTED IN THIS CASE

        a = np.arange(15).reshape(3, 5)
        print(a)
        return a

    run_this = PythonOperator(
        task_id="print_the_context",
        python_callable=print_array,
    )
```
https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#top-level-python-code

### Variable
Variables are global, and should **only be used for overall configuration that covers the entire installation**; to pass data from one Task/Operator to another, you should use XComs instead.

https://airflow.apache.org/docs/apache-airflow/stable/concepts/variables.html


### Unit tests
#### Unit test for loading a DAG:
```
import pytest

from airflow.models import DagBag


@pytest.fixture()
def dagbag():
    return DagBag()


def test_dag_loaded(dagbag):
    dag = dagbag.get_dag(dag_id="hello_world")
    assert dagbag.import_errors == {}
    assert dag is not None
    assert len(dag.tasks) == 1

```

note:
The `DagBag`  is:
a collection of dags, parsed out of a folder tree and has high level configuration settings, like what database to use as a backend and what executor to use to fire off tasks. This makes it easier to run distinct environments for say production and development, tests, or for different teams or security profiles. What would have been system level settings are now dagbag level so that one system can run multiple, independent settings sets.

https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/dagbag/index.html

#### The Pendulum Package
```
import pendulum

now = pendulum.now("Europe/Paris")

# Changing timezone
now.in_timezone("America/Toronto")
```

https://pendulum.eustace.io/


#### Self-Checks
You can also implement checks in a DAG to make sure the tasks are producing the results as expected. As an example, if you have a task that pushes data to S3, you can implement a check in the next task. For example, the check could make sure that the partition is created in S3 and perform some simple checks to determine if the data is correct.

```
task = PushToS3(...)
check = S3KeySensor(
    task_id="check_parquet_exists",
    bucket_key="s3://bucket/key/foo.parquet",
    poke_interval=0,
    timeout=0,
)
task >> check

```

### Upgrades and downgrades
- Backup your database
- Disable the scheduler
- Add “integration test” DAGs
- Prune data before upgrading


https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#metadata-db-maintenance