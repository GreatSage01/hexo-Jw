---
title: Docker Remote API
date: 2020/3/31 10:52:54 
categories:
    - docker
    - swarm
tags:
    - docker
    - Docker API
math: true
---

Docker Remote API 笔记记录

<!--more-->

1、安装curl和jq 来发送API

	wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
	mv jq-linux64 /usr/local/bin/jq
	chmod +x /usr/local/bin/jq

2、API官方文档

	
    https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/

3、API范例

1. 查看集群节点

    	1、  curl \
    	      --unix-socket /var/run/docker.sock  \
    	      http:/nodes | jq '.' 
	    2、 curl \
     	      --unix-socket /var/run/docker.sock \
     		  http:/nodes/swarm-1 | jq '.'

1. 创建新服务

        curl -XPOST \ 
          -d '{
          "Name": "go-demo-db",
          "TaskTemplate":{"ContainerSpec":{"Image":"mongo:3.2.10"}}
          } '\
          --unix-socket /var/run/docker.sock \
          http:/services/create | jq '.'

1. 查看服务状态

	    curl --unix-socket /var/run/docker.sock   http:/services/go-demo-db | jq '.'

1. 查看服务的特定参数

    服务版本：

        VERSION=$(curl --unix-socket /var/run/docker.sock   http:/services/go-demo-db | jq '.Version.Index')
    服务ID：

		ID=$(curl --unix-socket /var/run/docker.sock   http:/services/go-demo-db | jq --raw-output '.ID')

		#--raw-output去掉冒号""


1. 更新服务参数

		curl -XPOST\
		  -d '{
		  "Name": "go-demo-db-1",
		  "TaskTemplate":{"ContainerSpec":{"Image":"mongo:3.2.10"}},
		  "Mode":{"Replicated":{"Replicas":3}}
		  } '\
		  --unix-socket /var/run/docker.sock \
		  http:/services/$ID/update?version=$VERSION

1. 获取容器的统计数据

    	curl --unix-socket /var/run/docker.sock   http:/containers/a586e70a2f22/stats
	    这个统计是流式的，禁用流式用stram参数
		curl --unix-socket /var/run/docker.sock   http:/containers/a586e70a2f22/stats?stream=false

1. 删除服务

        curl -XDELETE --unix-socket /var/run/docker.sockhttp:/services/go-demo-db

