---
layout:     post
title:      解决 poi-fusion 服务存在远程调试安全风险（JDWP）
subtitle:   JDWP 与 Arthas 远程调试风险的治理记录
date:       2023-07-19
author:     bbq
header-img: img/post-bg-security.jpg
catalog: true
tags:
    - Java
    - 安全
    - 运维
---

# 解决poi-fusion服务存在远程调试安全风险（JDWP）

 

## 1、背景

JDWP（Java Debug Wire Protocol）是一个为 Java 调试而设计的一个通讯交互协议

JDWP协议本身没有提供认证鉴权机制，网络可达即可建立连接，Arthas是一款线上监控诊断产品，自身提供了密码认证方式，但在美团内部使用中并未开启Arthas的认证能力

攻击者可直接用JDWP或Arthas获取远程主机控制权限，功能成本极低，对生产网络造成极大范围的安全风险

信息安全部联合Jumper团队为Jumper增加了安全使用JDWP和Arthas的相关能力，公司即将禁止在内网中以非本地环回地址监听开启JDWP&Arthas端口

## 2、工业界处理方案

|          | **线上环境**                                                 | **线下环境**                                                 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 腾讯     | Prod环境：禁止开放JDWP端口，禁止调试，检测到开放发安全工单  ST环境：与Prod相同，非特殊情况禁止开放，开放后需要使用跳板机连接 | 允许开放JDWP端口，但开启JDWP端口时必须研发手动配置iptables仅允许当前办公机IP访问，如果没有配置，会通过扫描发送安全工单 |
| 蚂蚁     | Prod环境：禁止开放JDWP端口，禁止调试，检测到开放发安全工单  ST环境：与Prod环境环境相同，非特殊情况禁止开放，开放后需要使用跳板机连接 | 禁止研发手动开启JDWP  提供平台供研发申请调试，申请通过后会自动重启目标Java进程，开启JDWP端口允许办公网连接 |
| 阿里集团 | Prod环境：禁止开放JDWP端口，禁止调试，检测到开放发安全工单  ST环境：与Prod相同，非特殊情况禁止开放，开放后需要使用跳板机连接 | 线下环境不做限制，计划治理，暂不明确治理方式                 |
| 美团     | 允许开放JDWP端口，但必须监听在127.0.0.1，通过跳板机经过二次认证访问 | 与线上相同                                                   |

本次告警出现的源头在于，两台线下环境机器开放了JDWP端口，但是没有禁止远程调试。与上表的信息不一致。

| **主机名**                          | **端口号** | **进程** | **父进程** | **是否支持自动修复**               | **Tomcat版本信息**               |
| ----------------------------------- | ---------- | -------- | ---------- | ---------------------------------- | -------------------------------- |
| set-gh-nimbus-fusion-service-test04 | 44399      | java     | supervise  | 不支持自动修复，请按照文档手动修复 | 非Tomcat导致的JDWP风险，无此字段 |
| set-hh-nimbus-fusion-service-test01 | 44399      | java     | supervise  | 不支持自动修复，请按照文档手动修复 | 非Tomcat导致的JDWP风险，无此字段 |

 

## 3、解决方案

本次问题出现在（线下环境）MDP框架且发布项类型为普通类型。

### 3.1 JDWP风险修复

在appkey对应的代码中找到**deploy/run.sh**文件，此文件为MDP框架的部署脚本，找到以下代码：

```
function remoteDebug(){

  if [ -z "$DEBUG_PORT" ]; then

    DEBUG_PORT=44399

  fi

  \#QA要求在线下环境提供覆盖率扫描功能参数

  if [ -z "$JACOCO_ENABLED" ]; then

    JACOCO_ENABLED=true

  fi

  DEBUG_CMD=""

  current_env=$(getEnv)

  supported_envs="dev test"

  for env in $supported_envs

  do

    if [ "$current_env" == "$env" ]; then

      DEBUG_CMD="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=$DEBUG_PORT"

      if [ "$JACOCO_ENABLED" == "true" ]; then

        DEBUG_CMD="$DEBUG_CMD -javaagent:/opt/meituan/qa_test/jacocoagent.jar=output=tcpserver,port=6300,address=*"

      fi

      break

    fi

  done

  echo "$DEBUG_CMD"

}
```

将DEBUG_PORT=44399修改为DEBUG_PORT=127.0.0.1:44399，重新发布，即可完成JDWP修复。

### 3.2 Arthas风险修复

如果对应环境已经开启镜像更新，重新发布服务即可自动修复Arthas风险。（这是我们服务的状态，所以后续操作可以省略）

如果对应环境未开启镜像更新，请按照以下步骤操作

在appkey对应的代码中找到**deploy/check.sh**文件，此文件为MDP框架的健康检查脚本包含Arthas的启动代码，找到以下代码：

```
function installArthas() {

  if [ -z "$ARTHAS_ENABLED" ]; then

    ARTHAS_ENABLED=true

  fi

  if [ "$ARTHAS_ENABLED" != "true" ]; then

    echo "arthas is disabled."

    return

  fi

 

  if [ ! -d arthas ]; then

    mkdir arthas

  fi

  cd arthas || return

  if [ ! -x as.sh ]; then

    echo "Need to download and install arthas."

    curl -s -O https://s3plus.sankuai.com/v1/mss_c517ad14def1420690117f60f5150b79/arthas/arthas-3.6.1-mdp.tar.gz

    tar -xzf arthas-3.6.1-mdp.tar.gz

    sh install-local.sh

    echo "Successfully installed arthas."

  else

    echo "Found arthas, no need to install."

  fi

  if [ -x as.sh ]; then

    ./as.sh --attach-only @0.0.0.0:44397:44398

  fi

  echo "Successfully enabled arthas."

  cd ..

}
```

将./as.sh --attach-only @0.0.0.0:44397:44398修改为./as.sh --attach-only @127.0.0.1:0:44398，重新发布，即可完成修复

**done！**

 

 

 

 

 

 