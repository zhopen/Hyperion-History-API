The version is used For cmchain.testnet.pub

- eosio-node iamges: boscore/bos:v3.0.8  
- use fixed ip with links in docker-compose.yml insteal of DNS, because of DNS reponse is too slow.  
- add "sleep 20" in ./scripts/run-nodeos.sh. because of missing "nc" in eosio-node's image.  modified docker-compose.yml. 'true' means not to call wait for  
``` 
$ vi ./scripts/run-nodeos.sh   
sleep 20
if [ "$2" = "" ]
then
  nodeos --genesis-json /home/eosio/config/genesis.json --disable-replay-opts --data-dir /home/eosio/data --config-dir /home/eosio/config &
else
  nodeos --delete-all-blocks --snapshot /home/eosio/data/snapshots/$2 --disable-replay-opts --data-dir /home/eosio/data --config-dir /home/eosio/config &
fi
```
```
    command: bash -c "/home/eosio/scripts/run-nodeos.sh true  ${SNAPSHOT:-""}"

```



# 简介

这是一个单机版的hyperion history api系统。他可以做为基于EOS技术的区块链的历史节点。

如果需要部署为高可用的版本，需要对系统中用到的子系统进行高可用的安装。

# 必要条件

主机上需要安装docker系统

docker version： >18.0

docker-compose version: >v1.25.0

> docker-compose 版本v1.25.0
>
> curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose



# 使用方法

## 配置

- 链节点的配置

```
$ ll eosio/config/
total 8
-rw-r--r-- 1 zh_op 197609 1301 11月 24 14:06 config.ini
-rw-r--r-- 1 zh_op 197609  817 11月 24 14:06 genesis.json     
```

在eosio/config目录中放入链节点服务的配置文件config.ini和genesis.json

- 修改hyperion/config下的connections.json，ecosystem.config.js

```
$ ll hyperion/config
total 2
drwxr-xr-x 1 zh_op 197609   0 11月 24 14:06 chains/
-rw-r--r-- 1 zh_op 197609 690 11月 24 14:06 connections.json
-rw-r--r-- 1 zh_op 197609 179 11月 24 14:06 ecosystem.config.js
```

```
$ cat hyperion/config/connections.json
{
  "amqp": {
    "host": "rabbitmq:5672",
    "api": "rabbitmq:15672",
    "user": "username",
    "pass": "password",
    "vhost": "hyperion"
  },
  "elasticsearch": {
    "protocol": "http",
    "host": "elasticsearch:9200",
    "ingest_nodes": ["elasticsearch:9200"],
    "user": "elastic",
    "pass": "password"
  },
  "redis": {
    "host": "redis",
    "port": "6379"
  },
  "chains": {
    "eos": {               //提示:此处的eos为chain id
      "name": "cmchain.testnet.pub",
      "chain_id": "6cbecff836a9fa60da53bf97a0a180103b2e76041d4414693d11bf39e2341547",
      "http": "http://eosio-node:8888",
      "ship": "ws://eosio-node:8080",
      "WS_ROUTER_PORT": 7001,
      "WS_ROUTER_HOST": "hyperion-api"
    }
  }
}
```

​                      

```
$ cat hyperion/config/ecosystem.config.js
const {addApiServer, addIndexer} = require("./definitions/ecosystem_settings");

module.exports = {
    apps: [
        addIndexer('eos'),       //提示:此处的eos为chain id,要与connections.json中的保持一致。
        addApiServer('eos', 1)
    ]
};
```

​                             

- 修改hyperion/config/chains/eos.config.json  

  eos.config.json中eos位置处为chain id，可以配置为其他的名字。此文件为chain id为eos链的配置文件。

```
$ cat hyperion/config/chains/eos.config.json
{
  "api": {
    "chain_name": "cmchain.testnet.pub",
    "server_addr": "hyperion-api",
    "server_port": 7000,
    "server_name": "hyperion-api:7000",
    "provider_name": "Example Provider",
    "provider_url": "https://example.com",
    "chain_api": "",
    "push_api": "",
    "chain_logo_url": "",
    "enable_caching": true,
    "cache_life": 1,
    "limits": {
      "get_actions": 1000,
      "get_voters": 100,
      "get_links": 1000,
      "get_deltas": 1000
    },
    "access_log": false,
    "enable_explorer": false,
    "chain_api_error_log": false,
    "custom_core_token": ""
  },
  "settings": {
    "preview": false,
    "chain": "eos",
    "eosio_alias": "eosio",
    "parser": "1.8",
    "auto_stop": 300,
    "index_version": "v1",
    "debug": false,
    "bp_logs": false,
    "bp_monitoring": false,
    "ipc_debug_rate": 60000,
    "allow_custom_abi": false,
    "rate_monitoring": true,
    "max_ws_payload_kb": 256,
    "ds_profiling": false,
    "auto_mode_switch": false
  },
  "blacklists": {
    "actions": [],
    "deltas": []
  },
  "whitelists": {
    "actions": [],
    "deltas": [],
    "max_depth": 10,
    "root_only": false
  },
  "scaling": {
    "readers":            1,
    "ds_queues":          1,
    "ds_threads":         1,
    "ds_pool_size":       1,
    "indexing_queues":    1,
    "ad_idx_queues":      1,
    "max_autoscale":      4,
    "batch_size":         5000,
    "resume_trigger":     5000,
    "auto_scale_trigger": 20000,
    "block_queue_limit":  10000,
    "max_queue_limit":    100000,
    "routing_mode":       "heatmap",
    "polling_interval":   10000
  },
  "indexer": {                   //主要配置这一部分
    "start_on": 0,               //从哪个块开始
    "stop_on": 0,                //到哪个块结束
    "rewrite": false,            
    "purge_queues": false,       //清空rabbitmq中的队列
    "live_reader": true,         //实时读取新块
    "live_only_mode": false,    
    "abi_scan_mode": false,      
    "fetch_block": true,
    "fetch_traces": true,
    "disable_reading": false,
    "disable_indexing": false,
    "process_deltas": true
  },
  "features": {
    "streaming": {
      "enable": false,
      "traces": false,
      "deltas": false
    },
    "tables": {
      "proposals": true,
      "accounts": true,
      "voters": true
    },
    "index_deltas": true,
    "index_transfer_memo": false,
    "index_all_deltas": true,
    "deferred_trx": false,
    "failed_trx": false,
    "resource_limits": false,
    "resource_usage": false
  },
  "prefetch": {
    "read": 50,
    "block": 100,
    "index": 500
  }
}                                                                       
```

## 创建网络

```
./create_network.sh
```
## 启动节点

```
docker-compose up -d
```


