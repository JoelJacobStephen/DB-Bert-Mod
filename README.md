# DB-BERT MOD

## An attempt to slightly modify DB-BERT as a part of an implementation project

<img align="right" src="https://github.com/itrummer/dbbert/blob/7f8b9914ca4ef1081cfeeb1685c47e93a951ce6e/dbbert.png" width="192" alt="DB-BERT Logo">

# DB-BERT: The Tuning Tool that "Reads" the Manual

DB-BERT extracts hints for database parameter settings from text via natural language analysis. It then optimizes parameter settings for a given workload and performance metric using reinforcement learning.

---

## Running Experiments

To simplify the setup process, DB-BERT now uses Docker to encapsulate its environment. Follow these steps to set up and run DB-BERT experiments:

### Prerequisites

1. **Install Docker**
   - Ensure Docker is installed on your system. You can download and install Docker from [Docker’s official website](https://www.docker.com/products/docker-desktop).

2. **Clone the Repository**
   - Clone the DB-BERT repository:
     ```bash
     git clone https://github.com/JoelJacobStephen/DB-Bert-Mod
     cd dbbert
     ```

### Setup with Docker

1. **Build the Docker Image**
   - From the `dbbert` directory, build the Docker image:
     ```bash
     docker build -t dbbert:latest .
     ```

2. **Run the Docker Container**
   - Start a Docker container using the built image:
     ```bash
     docker run -it --rm -p 8501:8501 dbbert:latest
     ```
   - This will start the Streamlit GUI and expose it on port `8501`. Access it by navigating to [http://localhost:8501](http://localhost:8501) in your browser.

3. **Interactive Shell (Optional)**
   - To access an interactive shell inside the container for custom experiments, use:
     ```bash
     docker run -it --rm --entrypoint /bin/bash dbbert:latest
     ```

### Running Experiments

- Once inside the container, you can run DB-BERT experiments. Example:
  ```bash
  PYTHONPATH=src python3 src/run/run_dbbert.py demo_docs/postgres100 64000000000 200000000000 8 pg tpch dbbert dbbert "sudo systemctl restart postgresql" /tmp/tpchdata/queries.sql --recover_cmd="sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"
  ```

- During execution, DB-BERT generates two result files:
  - **dbbert_results_performance**: Contains performance measurements.
  - **dbbert_results_configure**: Describes configurations used for each trial run.

### Notes

1. The PostgreSQL and MySQL databases are pre-configured with a user named `dbbert` and password `dbbert`.
2. Benchmark databases (e.g., TPC-H and JOB) are installed during the Docker build process.

---

## Using DB-BERT: GUI

- To start the GUI, run:
  ```bash
  streamlit run src/run/interface.py
  ```
  from the DB-BERT root directory.

- If accessing DB-BERT on a remote EC2 server, make sure to enable inbound traffic to port `8501`.

- Enter the URL shown in the console into your web browser to access the interface.

- You can:
  - Select settings to read from configuration files in the `demo_configs` folder.
  - Select a collection of tuning text documents for extraction (e.g., from the `demo_docs` folder).
  - Change parameters related to database access, learning, and tuning goals.
  - Click the **Start Tuning** button to start the tuning process.

---

## Using DB-BERT: CLI

You can also use DB-BERT from the command line using `src/run/run_dbbert.py`.

### Required Parameters

The module requires the following parameters:

| Parameter        | Explanation                                                                                                                                                   |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `text_source_path` | Path to text with tuning hints. Text is stored as a `.csv` file, containing document IDs and text snippets. Example documents can be found in the `demo_docs` folder. |
| `memory`         | Amount of main memory of the target platform, measured in bytes.                                                                                              |
| `disk`           | Amount of disk space on the target platform, measured in bytes.                                                                                               |
| `cores`          | Number of cores available on the target platform.                                                                                                             |
| `dbms`           | Whether to tune PostgreSQL (set to `pg`) or MySQL (set to `ms`).                                                                                              |
| `db_name`        | Name of the database on which the target workload is running.                                                                                                 |
| `db_user`        | Name of the database login with access to the target database.                                                                                                |
| `db_pwd`         | Password of the database login.                                                                                                                               |
| `restart_cmd`    | Command for restarting the database server from the command line. E.g., `"sudo systemctl restart postgresql"` or `"sudo systemctl restart mysql"`.           |
| `query_path`     | Path to `.sql` file containing queries of the target workload, separated by semicolons (no semicolon after the last query).                                    |

> **Note**: Specifying the `recover_cmd` parameter is optional but highly recommended. Otherwise, sub-optimal parameter settings may prevent a database server restart, making it difficult for DB-BERT to correct faulty parameter values.

### Optional Parameters

You may also want to set the following parameters:

| Parameter            | Explanation                                                                                                                                                                  |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `max_length`         | Divide text into chunks of at most that many characters for extracting tuning hints.                                                                                         |
| `filter_params`      | Filter text passages via a simple heuristic (set to `1`, recommended) or not (set to `0`).                                                                                   |
| `use_implicit`       | Try to detect implicit parameter references (set to `1`, recommended) or not (set to `0`).                                                                                   |
| `hint_order`         | Try out tuning hints in document order (`0`), parameter order (`1`), or optimized order (`2`, recommended).                                                                  |
| `nr_frames`          | Maximal number of frames to execute for learning (set high, e.g., `100000`, to rely on timeout instead).                                                                     |
| `timeout_s`          | Maximal number of seconds used for tuning (this is approximate since the system finishes trial runs before checking for timeouts).                                           |
| `performance_scaling`| Scale rewards due to performance improvements by this factor.                                                                                                                |
| `assignment_scaling` | Scale rewards due to successful parameter value changes by this factor.                                                                                                      |
| `nr_evaluations`     | Number of trial runs based on the same collection of tuning hints (recommended: `2`).                                                                                        |
| `nr_hints`           | Number of hints to consider in combination (recommended: `20`).                                                                                                              |
| `min_batch_size`     | Batch size used for text analysis (e.g., `8`; optimal settings depend on the language model).                                                                                 |
| `recover_cmd`        | Command line command to reset database configuration if server restart is impossible. E.g., `"sudo rm /var/lib/postgresql/12/main/postgresql.auto.conf"` for PostgreSQL.      |
