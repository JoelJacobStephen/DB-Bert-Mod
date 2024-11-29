# DB-BERT MOD

## An attempt to slightly modify DB-BERT as a part of an implementation project

<img align='right' src='https://github.com/itrummer/dbbert/blob/7f8b9914ca4ef1081cfeeb1685c47e93a951ce6e/dbbert.png' width='192'>

# DB-BERT: the Tuning Tool that "Reads" the Manual

DB-BERT extracts hints for database parameter settings from text via natural language analysis. It then optimizes parameter settings for a given workload and performance metric using reinforcement learning.

Here is the updated README file with instructions for setting up and running DB-BERT using Docker:

DB-BERT MOD

An attempt to slightly modify DB-BERT as a part of an implementation project

<img align='right' src='https://github.com/itrummer/dbbert/blob/7f8b9914ca4ef1081cfeeb1685c47e93a951ce6e/dbbert.png' width='192'>

DB-BERT: the Tuning Tool that “Reads” the Manual

DB-BERT extracts hints for database parameter settings from text via natural language analysis. It then optimizes parameter settings for a given workload and performance metric using reinforcement learning.

Running Experiments

To simplify the setup process, DB-BERT now uses Docker to encapsulate its environment. Follow these steps to set up and run DB-BERT experiments:

Prerequisites

1. Install Docker
    - Ensure Docker is installed on your system. You can download and install Docker from Docker’s official website.
2. Clone the Repository
    - Clone the DB-BERT repository:
    
    ```markdown
    git clone https://github.com/JoelJacobStephen/DB-Bert-Mod
    cd dbbert
    ```
    
3. Build the Docker Image
    - From the dbbert directory, build the Docker image:

```markdown
docker build -t dbbert:latest .
```

1. Run the Docker Container
- Start a Docker container using the built image:

```markdown
docker run -it --rm -p 8501:8501 dbbert:latest
```

- This will start the Streamlit GUI and expose it on port 8501. Access it by navigating to <http://localhost:8501> in your browser.
1. Interactive Shell (Optional)
- To access an interactive shell inside the container for custom experiments, use:

```markdown
docker run -it --rm --entrypoint /bin/bash dbbert:latest
```

1. Running Experiments
- Once inside the container, you can run DB-BERT experiments. Example:

```markdown
PYTHONPATH=src python3 src/run/run_dbbert.py demo_docs/postgres100 64000000000 200000000000 8 pg tpch dbbert dbbert "sudo systemctl restart postgresql" /tmp/tpchdata/queries.sql --recover_cmd="sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"
```

- During execution, DB-BERT generates two result files:
    - dbbert_results_performance: Contains performance measurements.
    - dbbert_results_configure: Describes configurations used for each trial run.

Notes

<aside>
💡

```
1. The PostgreSQL and MySQL databases are pre-configured with a user named dbbert and password dbbert.
2. Benchmark databases (e.g., TPC-H and JOB) are installed during the Docker build process.

```

</aside>

# Using DB-BERT: GUI

- To start the GUI, run `streamlit run src/run/interface.py` from the DB-BERT root directory.
- If accessing DB-BERT on a remote EC2 server, make sure to enable inbound traffic to port 8501.
- Enter the URL shown in the console into your Web browser to access the interface.
- You can select settings to read from configuration files in the `demo_configs` folder.
- Select a collection of tuning text documents for extraction (e.g., from the `demo_docs` folder).
- You may change parameter related to database access, learning, and tuning goals.
- Click on the `Start Tuning` button to start the tuning process.

# Using DB-BERT: CLI

Use DB-BERT from the command line using `src/run/run_dbbert.py`.

## Required Parameters

The module requires setting the following parameters:

| Parameter | Explanation |
| --- | --- |
| text_source_path | path to text with tuning hints. Text is stored as .csv file, containing document IDs and text snippets. You find example documents in the `demo_docs` folder. |
| memory | the amount of main memory of the target platform, measured in bytes. |
| disk | the amount of disk space on the target platform, measured in bytes. |
| cores | the number of cores available on the target platform. |
| dbms | whether to tune PostgreSQL (set to `pg`) or MySQL (set to `ms`). |
| db_name | name of database on which target workload is running. |
| db_user | name of database login with access to target database. |
| db_pwd | password of database login. |
| restart_cmd | command for restarting database server from command line. E.g., `"sudo systemctl restart postgresql"` or `"sudo systemctl restart mysql"`. |
| query_path | path to .sql file containing queries of target workload, separated by semicolon (no semicolon after the last query!). |

Note: specifying `recover_cmd` parameter is optional but highly recommended (otherwise, sub-optimal parameter settings may prevent a restart of the database server, preventing DB-BERT from correcting faulty parameter values).

## Optional Parameters

You may also want to set the following parameters:

| Parameter | Explanation |
| --- | --- |
| max_length | divide text into chunks of at most that many characters for extracting tuning hints. |
| filter_params | filter text passages via simple heuristic (set to `1`, recommended) or not (set to `0`). |
| use_implicit | try to detect implicit parameter references (set to `1`, recommended) or not (set to `0`). |
| hint_order | try out tuning hints in document order (set to `0`), parameter order (set to `1`), or optimized order (set to `2`, recommended). |
| nr_frames | maximal number of frames to execute for learning (recommendation: set high, e.g. `100000`, to rely on timeout instead). |
| timeout_s | maximal number of seconds used for tuning (this number is approximate since system finishes trial runs before checking for timeouts). |
| performance_scaling | scale rewards due to performance improvements by this factor. |
| assignment_scaling | scale rewards due to successful parameter value changes by this factor. |
| nr_evaluations | number of trial runs based on the same collection of tuning hints (recommended: `2`). |
| nr_hints | number of hints to consider in combination (recommended: `20`). |
| min_batch_size | batch size used for text analysis (e.g., `8`, optimal settings depend on language model). |
| recover_cmd | command line command to reset database configuration if server restart is impossible. E.g., use `"sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"` for PostgreSQL. |
