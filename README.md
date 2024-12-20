# DB-BERT MOD

## An attempt to slightly modify DB-BERT as a part of an implementation project

<img align='right' src='https://github.com/itrummer/dbbert/blob/7f8b9914ca4ef1081cfeeb1685c47e93a951ce6e/dbbert.png' width='192'>

# DB-BERT: the Tuning Tool that "Reads" the Manual

DB-BERT extracts hints for database parameter settings from text via natural language analysis. It then optimizes parameter settings for a given workload and performance metric using reinforcement learning.

# Running Experiments

The following instructions have been tested on a p3.2xlarge EC 2 instance with Deep Learning Base GPU AMI (Ubuntu 20.04) 20230811 AMI. On that system, run them from the ubuntu home directory.

1. Install PostgreSQL and MySQL, e.g., run:

```
sudo apt-get update
sudo apt install postgresql-12
sudo apt install mysql-server-8.0
```

2. Create a user named "dbbert" with password "dbbert" for PostgreSQL and MySQL:

```
sudo adduser dbbert # Type "dbbert" as password twice, then press enter repeatedly, finally "Y" to confirm
sudo -u postgres psql -c "create user dbbert with password 'dbbert' superuser;"
sudo mysql -e "create user 'dbbert'@'localhost' identified by 'dbbert'; grant all privileges on *.* to 'dbbert'@'localhost'; flush privileges;"
```

3. Download this repository, e.g., run:

```
git clone https://github.com/itrummer/dbbert
```

4. Install dependencies:

3. **Interactive Shell (Optional)**
- To access an interactive shell inside the container for custom experiments, use:
     ```bash
     docker run -it --rm --entrypoint /bin/bash dbbert:latest
     ```
4. **Start Postgres and MySQL server (if not started automatically)**

- After running the container, access it using the following command:
     ```bash
     docker exec -it <container_name> /bin/bash
     ```
     Replace `<container_name>` with the actual name of the container (e.g., `nice_noyce`).

- Inside the container, modify the `max_wal_size` parameter in the PostgreSQL configuration file:
     ```bash
     Modify max_wal_size from '1GB' to '2GB' in /etc/postgresql/15/main/postgresql.conf
     ```

- Start the necessary services:
     ```bash
     sudo service postgresql start
     sudo service mysql start
     ```

- Ensure the services are running properly after starting them.
    
### Running Experiments

```
sudo scripts/installtpch.sh
sudo scripts/installjob.sh
```

6. You can now run experiments with DB-BERT, e.g.:

```
PYTHONPATH=src python3.9 src/run/run_dbbert.py demo_docs/postgres100 64000000000 200000000000 8 pg tpch dbbert dbbert "sudo systemctl restart postgresql" /tmp/tpchdata/queries.sql --recover_cmd="sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"
```

During execution, DB-BERT generates two result files:

- dbbert_results_performance: contains tab-separated rows describing performance measurements for each trial run. The columns represent (from left to right) the run counter, the evaluation counter within the run, the time since the start of the current run in milliseconds, the optimal performance achieved over all evaluations within the run (e.g., if minimizing run time, this is the minimal query execution time in milliseconds), and the performance measured for the current trial run.
- dbbert_results_configure: contains tab-separated rows describing configurations used for each trial run. The columns represent (from left to right) the run counter, the evaluation counter within the run, the time since the start of the current run in milliseconds, the optimal configuration over all evaluations within the run, and the current configuration. Each configuration is represented by a dictionary, mapping parameter names to their values (it only contains parameters with non-default settings).

See next section for explanations on DB-BERT's command line parameters.

# Using DB-BERT: CLI

Use DB-BERT from the command line using `src/run/run_dbbert.py`.

## Required Parameters

The module requires setting the following parameters:

| Parameter        | Explanation                                                                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| text_source_path | path to text with tuning hints. Text is stored as .csv file, containing document IDs and text snippets. You find example documents in the `demo_docs` folder. |
| memory           | the amount of main memory of the target platform, measured in bytes.                                                                                          |
| disk             | the amount of disk space on the target platform, measured in bytes.                                                                                           |
| cores            | the number of cores available on the target platform.                                                                                                         |
| dbms             | whether to tune PostgreSQL (set to `pg`) or MySQL (set to `ms`).                                                                                              |
| db_name          | name of database on which target workload is running.                                                                                                         |
| db_user          | name of database login with access to target database.                                                                                                        |
| db_pwd           | password of database login.                                                                                                                                   |
| restart_cmd      | command for restarting database server from command line. E.g., `"sudo systemctl restart postgresql"` or `"sudo systemctl restart mysql"`.                    |
| query_path       | path to .sql file containing queries of target workload, separated by semicolon (no semicolon after the last query!).                                         |

Note: specifying `recover_cmd` parameter is optional but highly recommended (otherwise, sub-optimal parameter settings may prevent a restart of the database server, preventing DB-BERT from correcting faulty parameter values).

## Optional Parameters

You may also want to set the following parameters:

| Parameter           | Explanation                                                                                                                                                                  |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| max_length          | divide text into chunks of at most that many characters for extracting tuning hints.                                                                                         |
| filter_params       | filter text passages via simple heuristic (set to `1`, recommended) or not (set to `0`).                                                                                     |
| use_implicit        | try to detect implicit parameter references (set to `1`, recommended) or not (set to `0`).                                                                                   |
| hint_order          | try out tuning hints in document order (set to `0`), parameter order (set to `1`), or optimized order (set to `2`, recommended).                                             |
| nr_frames           | maximal number of frames to execute for learning (recommendation: set high, e.g. `100000`, to rely on timeout instead).                                                      |
| timeout_s           | maximal number of seconds used for tuning (this number is approximate since system finishes trial runs before checking for timeouts).                                        |
| performance_scaling | scale rewards due to performance improvements by this factor.                                                                                                                |
| assignment_scaling  | scale rewards due to successful parameter value changes by this factor.                                                                                                      |
| nr_evaluations      | number of trial runs based on the same collection of tuning hints (recommended: `2`).                                                                                        |
| nr_hints            | number of hints to consider in combination (recommended: `20`).                                                                                                              |
| min_batch_size      | batch size used for text analysis (e.g., `8`, optimal settings depend on language model).                                                                                    |
| recover_cmd         | command line command to reset database configuration if server restart is impossible. E.g., use `"sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"` for PostgreSQL. |

# Using DB-BERT: GUI

- To start the GUI, run `streamlit run src/run/interface.py` from the DB-BERT root directory.
- If accessing DB-BERT on a remote EC2 server, make sure to enable inbound traffic to port 8501.
- Enter the URL shown in the console into your Web browser to access the interface.
- You can select settings to read from configuration files in the `demo_configs` folder.
- Select a collection of tuning text documents for extraction (e.g., from the `demo_docs` folder).
- You may change parameter related to database access, learning, and tuning goals.
- Click on the `Start Tuning` button to start the tuning process.

# Resources

A video talk introducing the vision behind this project is [available online](https://youtu.be/Spa5qzKbJ4M). Please cite:

```
@inproceedings{Trummer2022,
author = {Trummer, Immanuel},
booktitle = {SIGMOD},
pages = {190--203},
title = {{DB-BERT: a Database Tuning Tool that ``Reads the Manual''}},
url = {https://doi.org/10.1145/3514221.3517843},
year = {2022}
}

@article{Trummer2021nlp,
author = {Trummer, Immanuel},
journal = {PVLDB},
number = {7},
pages = {1159--1165},
title = {{The Case for NLP-Enhanced Database Tuning: Towards Tuning Tools that “Read the Manual”}},
url = {https://doi.org/10.14778/3450980.3450984},
volume = {14},
year = {2021}
}
```
