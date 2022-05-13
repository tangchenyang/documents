# 初始化 postgres 数据库

## Docker 安装 Postgres

## 创建 airflow 和 airflow_ctl 数据库

```postgresql
CREATE USER airflow WITH PASSWORD 'airflow';
CREATE DATABASE airflow OWNER airflow;
ALTER DATABASE airflow OWNER TO airflow;
GRANT ALL PRIVILEGES ON DATABASE airflow to airflow;
CREATE DATABASE airflow_ctl;
```
![xxx](https://raw.githubusercontent.com/tangchenyang/images/master/img20220307172028.png)
## 创建 control_tbl 表

用 airflow 用户登录 airflow_ctl 数据库, 执行一下SQL语句

```postgresql
CREATE TABLE control_tbl
(
    control_id     BIGSERIAL PRIMARY KEY NOT NULL,
    airflow_dag_id varchar(250)          NOT NULL,
    airflow_run_id varchar(250)          NOT NULL,
    start_time     BIGINT,
    end_time       BIGINT,
    status         varchar(25),
    has_errors     boolean,
    errors         TEXT,
    extra_info     TEXT,
    created_at     TIMESTAMP DEFAULT (NOW() AT TIME ZONE 'UTC'),
    last_updated   TIMESTAMP DEFAULT (NOW() AT TIME ZONE 'UTC')
);

CREATE VIEW sourcing_job_stats
AS
SELECT airflow_dag_id,
       airflow_run_id,
       to_timestamp(start_time / 1000) AT TIME ZONE 'UTC' as time,
       extra_info::json -> 'application_id'               as application_id,
       extra_info::json -> 'Request' ->> 'file'           as file,
       extra_info::json -> 'Request' ->> 'args'           as args,
       last_updated                                       as actual_run_time
FROM control_tbl
WHERE airflow_dag_id IN
      ('sourcing_billing_events_dag', 'sourcing_profile_api_dag', 'sourcing_signup_api_dag',
       'sourcing_user_preference_dag');

GRANT ALL PRIVILEGES ON DATABASE "airflow_ctl" to airflow;
GRANT ALL ON control_tbl TO airflow;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO airflow;
```

# 安装 Airflow
## 安装 python 环境
### 安装 python 3.7.8
```shell
pyenv install 3.7.8
```

### 设 python 主版本为 3.7.8
```shell
pyenv global 3.7.8
```

### 安装 virtualenv
```shell
pip install virtualenv
```

## 为 Airflow 创建虚拟 Python 环境
```shell
mkdir -p ~/Software/airflow-1.10.13
virtualenv --python=3.8 ~/Software/airflow-1.10.13
```

## 安装 Airflow
### 安装 Airflow 所需的 package
```shell
cd ~/Software/airflow-1.10.13 && . bin/activate
   
cat > requirements.txt <<EOF
apache-airflow == 1.10.13
newrelic-telemetry-sdk == 0.3.0
SQLAlchemy == 1.3.23
WTForms == 2.3.3
psycopg2-binary == 2.9.3
python-gnupg == 0.4.8 
pysftp == 0.2.9
pyarrow == 6.0.1 
scikit-learn
ssh-tunnel
PGPy
awscli
boto3
google-api-python-client
oauth2client
EOF

pip install -r requirements.txt
```

### 创建 .env
```shell
cat > .env <<EOF
export AIRFLOW_HOME=$HOME/airflow
EOF
```

### 初始化airflow 配置项
将生成 ~/airflow 目录，包含 airflow.cfg 配置文件
```shell
airflow initdb
```

### 修改数据库连接为 postgres, 并重新初始化数据库
```shell
vim ~/airflow/airflow.cfg
sql_alchemy_conn = postgresql://airflow:airflow@localhost/airflow
airflow initdb
```

### 启动airflow
```shell
cd ~/Software/airflow-1.10.13 && . bin/activate 
# 启动 scheduler 
source .env && airflow scheduler 
# 启动 webserver 
source .env && airflow webserver 
```

### copy 项目中的dags
```shell
cd ~/airflow 
cp -r /Users/chenyangtang-foxtel/IdeaProjects/foxtel/streamotion-data-airflow-pipelines/dags .
cp -r /Users/chenyangtang-foxtel/IdeaProjects/foxtel/streamotion-data-airflow-pipelines/plugins .
```

## 卸载 Airflow 

```shell
rm -rf ~/airflow
rm -rf ~/Software/airflow-1.10.13 
```
    

## 修改配置 
- sql_alchemy_conn = postgresql://airflow:airflow@localhost/airflow
- hostname_callable = socket:gethostname
- executor = LocalExecutor
- workers = 4


    