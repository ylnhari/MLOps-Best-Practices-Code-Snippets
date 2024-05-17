# Apache Airflow Installation Guide

Follow these steps to install Apache Airflow:

1. **Create a new isolated environment (optional but recommended)**: This is to avoid mixing Airflow's dependencies with your other Python projects. You can use `venv` for this.
    ```bash
    python3 -m venv airflow-env
    source airflow-env/bin/activate
    ```

2. **Install Airflow**: Install Airflow with necessary extras like mssqlserver.
    ```bash
    pip install apache-airflow[mssql]
    pip install pyodbc
    pip install apache-airflow-providers-microsoft-mssql
    ```

3. **Configure Airflow**: You need to set a few environment variables to configure the Airflow installation.
    ```bash
    export AIRFLOW_HOME=~/airflow
    ```

4. **Initialize the database**: Airflow uses a database to store metadata. By default, it uses SQLite, but for production use, you should use a more robust database like PostgreSQL/MSSQLServer.
    ```bash
    airflow db init
    ```

5. **Create MS SQLServer Database:**: If SqlServer was already installed create a new Database else ignore this step, In such case SQLite database can be used to run Airflow for development purpose as it does not require any database server (the database is stored in a local file). There are many limitations of using the SQLite database (for example it only works with Sequential Executor) and it should NEVER be used for production. :
    ```sql
    CREATE DATABASE airflow_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

    CREATE USER 'airflow_user' IDENTIFIED BY 'airflow_pass';

    GRANT ALL PRIVILEGES ON airflow_db.* TO 'airflow_user';
    ```

6. **Edit the airflow.cfg file:**: The airflow.cfg file should be located in the ~/airflow directory. Open the airflow.cfg file in an editor:
    ```bash
    nano ~/airflow/airflow.cfg
    ```

7. **Update airflow.cfg:**: Update the [core] section to use 'LocalExecutor' for MS SQL SERVER  else use 'SequentialExecutor' , edit the dags folder to the proper location and configure the connection to MS SQL Server (ignore MS SQL Server configuration if you are using sqllite):
    ```ini
    [core]
    executor = <RequiredExecutor>

    dags_folder = /path/to/your/dags
    
    sql_alchemy_conn = mssql+pyodbc://airflow_user:<YourStrong!Passw0rd>@localhost/airflow_db?driver=ODBC+Driver+17+for+SQL+Server
    ```


8. **Create Admin User account**: Use following command to create a user with admin previlidges.
    ```bash
    airflow users create --username admin --firstname Admin --lastname User --role Admin --email admin@example.com
    ```

9. **Start the web server**: This starts the Airflow web interface.
    ```bash
    airflow webserver --port 8080
    ```


10. **Start the scheduler**: In a new terminal, start the Airflow scheduler.
    ```bash
    airflow scheduler
    ```

Now, you should be able to access the Airflow web interface at [http://localhost:8080](http://localhost:8080). Use credentials created in step 8 to login. From there, you can start defining your workflows as DAGs.

11. **Access airlow ui**: If you need to connect to a remote Airflow instance from your local machine, you can do so once the server is up and running. This involves forwarding your local machine's port 8080 to the remote server's port. After setting up this connection, you can access the Airflow user interface by navigating to localhost:8080 in your web browser.
    ```bash
    ssh -L 8080:localhost:8080 username@ipaddress
    ```

Please note that this is a basic setup and there are many other considerations for a production deployment of Airflow, such as setting up redundancy for high availability, securing the Airflow web interface with SSL, and setting up monitoring and logging.

## Additional Considerations for Production Deployment

### High Availability and Redundancy
For a production setup, you would want to ensure that your Airflow instance is highly available. This means setting up multiple instances of the scheduler and web server, so that if one goes down, the others can continue to operate. You can achieve this by running multiple Airflow instances on separate machines and using a load balancer to distribute traffic between them.

### Securing the Web Interface
The Airflow web interface should be secured with SSL to protect sensitive data. You can use a tool like Let's Encrypt to obtain a free SSL certificate. Once you have the certificate, you can configure the web server to use it by setting the `web_server_ssl_cert` and `web_server_ssl_key` options in the `airflow.cfg` file.

### Monitoring and Logging
Airflow logs can be configured to be sent to a centralized location for easier access and analysis. You can use services like Elasticsearch, Logstash, and Kibana (ELK stack) for this purpose. For monitoring, you can use tools like Prometheus and Grafana to track Airflow metrics.

Here's a high-level overview of how to set up these features:

#### High Availability and Redundancy
This involves setting up multiple instances of Airflow and a load balancer. This is highly dependent on your infrastructure, but typically involves setting up additional machines and configuring a load balancer to distribute traffic.

#### Securing the Web Interface
To secure the web interface with SSL, you would first need to obtain an SSL certificate. You can use Let's Encrypt for this. Once you have the certificate, you can configure the web server to use it by setting the following options in the `airflow.cfg` file:

```ini
[webserver]
web_server_ssl_cert = /path/to/cert.pem
web_server_ssl_key = /path/to/key.pem
```
### Monitoring and Logging
To set up centralized logging, you can configure Airflow to send logs to a service like Elasticsearch, and then use Kibana to view and analyze the logs. For monitoring, you can use Prometheus to collect Airflow metrics and Grafana to visualize them. This involves installing and configuring these tools, and then setting the appropriate options in the airflow.cfg file.

Please note that these are complex topics that involve many steps and are highly dependent on your specific infrastructure and requirements. For detailed instructions, you should refer to the documentation for each tool and the Airflow documentation.
