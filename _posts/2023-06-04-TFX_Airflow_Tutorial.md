---
layout: post
title: "TFX Airflow Tutorial"
date: 2023-06-04 10:11:30
tags: [tensorflow]
---

![TensorFlowExtend](/assets/images/2023-06-04-TFX_Airflow_Tutorial_0.png)

# Tutorial

## 环境要求

- Linux/MacOS
- Python 3.9 及更高版本
- Virtualenv
- Git

## 环境配置

```bash
    cd
    virtualenv -p python3 tfx-env
    source ~/tfx-env/bin/activate
    mkdir tfx; cd tfx



    git clone https://github.com/tensorflow/tfx.git
    cd ~/tfx/tfx/tfx/examples/airflow_workshop/setup
    ./setup_demo.sh
```

## 创建流水线框架

### 1. 打开一个新的终端窗口，在该窗口中执行下面的命令，启动 airflow 管理服务

```bash
    # Open a new terminal window, and in that window ...
    source ~/tfx-env/bin/activate
    cd
    airflow users  create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin
    airflow webserver -p 8080
```

### 2. 打开一个新的终端窗口，在该窗口中执行下面的命令，启动 airflow 调度服务

```bash
    # Open another new terminal window, and in that window ...
    source ~/tfx-env/bin/activate
    airflow scheduler
```

#### 在浏览器中：

打开浏览器，然后转到 http://127.0.0.1:8080

##### DAG 操作

登陆管理平台，用户名：admin, 密码：admin
![login page](/assets/images/2023-06-04-TFX_Airflow_Tutorial_1.png)
运行 taxi 流水线
![DAG](/assets/images/2023-06-04-TFX_Airflow_Tutorial_2.png)

![TensorFlow](/assets/images/2023-06-04-TFX_Airflow_Tutorial_3.png)![dag-button-refresh](/assets/images/2023-06-04-TFX_Airflow_Tutorial_4.png)

### 3. 打开一个新的终端窗口，在该窗口中执行下面的命令，打开对每个组件进行性能分析的 notebook

```bash
    # Open yet another new terminal window, and in that window ...
    # Assuming that you've cloned the TFX repo into ~/tfx
    source ~/tfx-env/bin/activate
    cd ~/tfx/tfx/tfx/examples/airflow_workshop/notebooks
    jupyter notebook
```

![login page](/assets/images/2023-06-04-TFX_Airflow_Tutorial_5.png)
