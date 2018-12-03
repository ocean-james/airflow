
# 一、airflow安装   
## 1.1、MAC安装
Airflow依赖于python，所以先安装python<建议使用python2.7>。   
python下载地址：   
https://www.python.org/downloads/   
也可使用pyenv进行python的版本管理   
```
--安装pyenv
brew install pyenv
--安装相应版本的python
pyenv install 2.7.10
--版本管理 切换版本
pyenv versions
pyenv global 2.7.10

```
先配置airflow环境变量，默认会在当前目录下生成airflow文件夹，也可自定义目录路径数据库初始化。     
```
export AIRFLOW_HOME=～/airflow
```
直接采用pip安装即可。   
```
pip install airflow
```
---
* 如碰到pip版本问题的先更新pip版本，其他问题：python3.7版本可能有问题，关键字冲突，如出现错误   
***Command python setup.py egg_info failed with error code 1 in /private/var/folders/zn/8b5xh_pn0y5cmjr0sr15tnhm0000gn/T/pip-install-OQFTZY/mysqlclient/***  
建议使用python2.7版本进行安装。   
* 如本机有多个版本python并遇到上述问题，可试试 
```
python2 -m pip install airflow
```
* 除此之外，可对Airflow额外依赖关系的依赖包进行条件导入。
具体可见：   
http://airflow.apache.org/installation.html#initiating-airflow-database
-------
其他可能需要的环境  
安装mysql数据库  
```
brew install mysql
```
安装RedisServer   
```
brew install redis
```

## 1.2 Centos7安装   
```
#安装gcc
yum install gcc
#安装pip工具及setuptools
yum install pip 
pip install setuptools==33.1.1
#安装环境依赖包
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```
* 可能问题：
如果错误提示类似缺少kernel-headers     
安装kernel-headers:
```
wget http://vault.centos.org/5.7/os/x86_64/CentOS/kernel-headers-2.6.18-274.el5.x86_64.rpm   
rpm -ivh kernel-headers-2.6.18-274.el5.x86_64.rpm
```

* Cannot uninstall 'Markdown'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.   
--cd 到相应目录下/usr/lib/python2.7/site-packages/ 手动删除 markdown相关文件夹及文件



* EnvironmentError: mysql_config not found   
```
  yum install mysql-devel gcc gcc-devel python-devel
```


  airflow安装语句   
```
sudo SLUGIFY_USES_TEXT_UNIDECODE=yes  pip install apache-airflow
```    
### centos7 Mysql安装  
以下是网上一般的安装办法，但会有错误，因为CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了
```
#yum install mysql
#yum install mysql-server
#yum install mysql-devel
```
解决办法，安装mariadb数据库即可
```
# yum install mariadb-server mariadb

systemctl start mariadb  #启动MariaDB
systemctl stop mariadb  #停止MariaDB
systemctl restart mariadb  #重启MariaDB
systemctl enable mariadb  #设置开机启动
```

##  初始化Airflow
安装成功后,初始化airflow  
```
airflow initdb
```
初始化完成后在～/airflow文件夹下会生成  
airflow.cfg  
airflow.db  
unittest.cfg  
这几个文件，airflow.cfg是配置文件，包含各种配置，airflow.db是初始化的数据库文件，默认使用的是SQLite

如需更改后台数据库为mysql可登陆mysql，创建用户和数据库进行连接   
```
mysql -u用户名 -p密码 
create user airflow identified by 'airflow';
create database airflow; 
grant all privileges on airflow.* to 'airflow'@'%' identified by 'airflow'; 
flush privileges;
```

因为使用MySQL作为元数据库，所以还需要配置数据库的连接参数。在～/airflow下airflow.cfg中设置如下选项 
```
executor = LocalExecutor
sql_alchemy_conn = mysql://root:root@localhost:3306/airflow
```
改为本地执行模式和配置数据库连接的相应连接参数。

重新初始化airflow 
```
airflow initdb
```
这样可在mysql数据库中airflow数据库下看到数据表建立  
![](dag.png)  

开启airflow UI界面  
```
airflow webserver
```
出现如下图标安装成功  
![](启动成功.png)  
启动监测程序  
```
airflow scheduler
```
可登录网址
localhost:8080
进行查看   
![](webUI.png)  


# 二、airflow连接配置
管理连接
打开Admin->Connections可看到配置的连接  
![](connections.png)  

新建连接
打开Admin->ConnectionUI的部分    
![](建conn.png)  
单击Create链接以创建新连接  
![](编辑conn.png)  
ConnId：使用所需的连接ID。  
ConnType：选择连接类型。 
单击Save按钮以创建连接。

修改连接
打开Admin->ConnectionUI        的部分。单击连接列表中要编辑的连接旁边的铅笔图标。  
![](修改conn.png)  

修改连接属性，然后单击Save按钮以保存更改。

# 三、Dags测试
编写一个python脚本，保存为hello-world.py文件   
```
# -*- coding: utf-8 -*-
import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from datetime import timedelta
#-------------------------------------------------------------------------------
# these args will get passed on to each operator
# you can override them on a per-task basis during operator initialization
default_args = {
    'owner': 'james.wang',
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(2),
    'email': ['961862251@qq.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
    # 'wait_for_downstream': False,
    # 'dag': dag,
    # 'adhoc':False,
    # 'sla': timedelta(hours=2),
    # 'execution_timeout': timedelta(seconds=300),
    # 'on_failure_callback': some_function,
    # 'on_success_callback': some_other_function,
    # 'on_retry_callback': another_function,
    # 'trigger_rule': u'all_success'
}
#-------------------------------------------------------------------------------
# dag
dag = DAG(
    'example_hello_world_dag',
    default_args=default_args,
    description='my first DAG',
    schedule_interval='*/30 * * * *')
#-------------------------------------------------------------------------------
# first operator
date_operator = BashOperator(
    task_id='date_task',
    bash_command='date',
    dag=dag)
#-------------------------------------------------------------------------------
# second operator
sleep_operator = BashOperator(
    task_id='sleep_task',
    depends_on_past=False,
    bash_command='sleep 5',
    dag=dag)
#-------------------------------------------------------------------------------
# third operator
def print_hello():
    return 'Hello world!'
hello_operator = PythonOperator(
    task_id='hello_task',
    python_callable=print_hello,
    dag=dag)
#-------------------------------------------------------------------------------
# dependencies
sleep_operator.set_upstream(date_operator)
hello_operator.set_upstream(date_operator)
```
在～/airflow环境变量文件夹下新建dags文件夹，将hello-world.py保存在dags下，airflow会自动在该目录下查找。  
测试python脚本  
```
python ~/airflow/dags/hello-world.py
```
如果没有抛出异常，则脚本正确，环境健全。
可进一步测试
```
# print the list of active DAGs
airflow list_dags
# prints the list of tasks the "tutorial" dag_id
airflow list_tasks tutorial
# prints the hierarchy of tasks in the tutorial DAG
airflow list_tasks tutorial --tree
```
在webUI界面能看到新增的dags  example_hello_world_dag

测试dags  
```
$ airflow test example_hello_world_dag hello_task 2018-11-12
```   
![](测试dag成功.png)  
输出 ‘Hello world!’

将DAG设置为“ON”状态，点击右侧link第一个启动按钮，开始running，执行一定时间后可浏览记录。
Web UI界面可看到执行成功记录，深绿色代表执行成功标志。   
![](深绿色成功.png)  

Log日志中记录   
![](log日志.png)  

