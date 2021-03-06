---
title: "Configuration Reference"
linkTitle: "Configuration Reference"
weight: 5
description: >
  Anka Build Cloud Configuration Reference
---

## Controller Configuration Reference

Configuring your Anka Build Cloud Controller & Registry to enable features or customize URLs has several methods available.

> If you're using the standalone Registry package, you'll need to use Flags/Options and edit your `/Library/LaunchDaemons/com.veertu.anka.registry.plist`, then restart it with `launchctl unload /Library/. . . && launchctl load /Library/. . .`

{{< tabs tabTotal="2" tabID="1" tabName1="Environment Variables" tabName2="Flags / Options" >}}
{{< tab tabNum="1" >}}

---

Depending on the package you're using (native or docker), you can set ENV variables to modify the configuration of your controller and registry.

#### docker-compose.yml (docker)

```yml
  anka-controller:
    container_name: anka-controller
    build:
       context: .
       dockerfile: anka-controller.docker
    ports:
       - "8090:80"
       #- "8100:8100"
    volumes:
       - /Users/myUserName:/mnt/cert
    depends_on:
       - etcd
       - anka-registry
    restart: always
    environment:
      ANKA_REGISTRY_ADDR: "http://anka.registry:8089"
      ANKA_USE_HTTPS: "false"
      ANKA_SKIP_TLS_VERIFICATION: "false"
      ANKA_SERVER_CERT: "/mnt/cert/anka-controller-crt.pem"
      ANKA_SERVER_KEY: "/mnt/cert/anka-controller-key.pem"
      ANKA_CA_CERT: "/mnt/cert/anka-ca-crt.pem"
      ANKA_ENABLE_AUTH: "false"
      ANKA_ROOT_TOKEN: "1111111111"
```

#### /usr/local/bin/anka-controllerd (native)

> You must comment out the export to disable

```bash
#!/bin/bash

export ANKA_STANDALONE="true"
export ANKA_LISTEN_ADDR=":8090"
export ANKA_DATA_DIR="/Library/Application Support/Veertu/Anka/anka-controller"
export ANKA_ENABLE_CENTRAL_LOGGING="true"
export ANKA_LOG_DIR="/Library/Logs/Veertu/AnkaController"

export ANKA_RUN_REGISTRY="true"
export ANKA_REGISTRY_BASE_PATH="/Library/Application Support/Veertu/Anka/registry"
export ANKA_REGISTRY_LISTEN_ADDRESS="anka.registry:8089"
export ANKA_ANKA_REGISTRY="http://anka.registry:8089"

#export ANKA_USE_HTTPS="true"
#export ANKA_SKIP_TLS_VERIFICATION="true"
#export ANKA_SERVER_CERT="/Users/MyUser/anka-controller-crt.pem"
#export ANKA_SERVER_KEY="/Users/MyUser/anka-controller-key.pem"
#export ANKA_CA_CERT="/Users/MyUser/anka-ca-crt.pem"
#export ANKA_CLIENT_CERT="/Users/MyUser/anka-controller-crt.pem"
#export ANKA_CLIENT_CERT_KEY="/Users/MyUser/anka-controller-key.pem"

#export ANKA_ENABLE_AUTH="true"
#export ANKA_ENABLE_REGISTRY_AUTHORIZATION="true"
#export ANKA_ROOT_TOKEN="1111111111"

/Library/Application\ Support/Veertu/Anka/bin/anka-controller
```

### General and Common

> When editing the `/usr/local/bin/anka-controllerd`, be sure to use export when setting the ENV

| Name | Type | Description | Default Value | ENV |
| --- | :---: | --- | :---: | :---: |
| Version | bool | Prints controller version and exits | - | ANKA_VERSION | 
| Registry address | string | Anka Registry external URL (http[s]://hostname:[port]). This is passed to the Nodes, so they can download (and start) VMs | (required) | ANKA_ANKA_REGISTRY |
| Configuration file | string | Path to a configuration file in INI format. You can use the file with/without the command line parameters and env variables | - | ANKA_CONFIG |
| Listen address | string | Listen on this address (:80 is equivalent to 0.0.0.0:80). Use the format `[address]:port` | :80 | ANKA_LISTEN_ADDR |
| Local Registry Address | string | Anka Registry local address in format `http[s]://hostname:[port]`. This parameter is for situations where the Controller and Registry are on the same network. For example `http://locahost:8089` | - | ANKA_LOCAL_ANKA_REGISTRY |
| Number of concurrent workers | int | The number of concurrent workers processing node tasks | 2 | ANKA_NUM_WORKERS |
| Standalone mode | bool | Run an embedded ETCD server alongside the controller | false | ANKA_STANDALONE |
| ETCD endpoints | string | Comma separated list of etcd hosts | 127.0.0.1:2379 | ANKA_ETCD_ENDPOINTS |
| Allow empty registry | bool | Allow controller to start without a 'Registry address' | false | ANKA_ALLOW_EMPTY_REGISTRY |
| Enable event logging | bool | Enables event logging. **`Requires a Enterprise Plus license and will show under the Controller's Logs section after the first instance is created.`** | false | ANKA_ENABLE_EVENT_LOGGING |
| Event log url | string | The URL to post events (in json format) | - | ANKA_EVENT_LOG_URL |
| Enable central logging | bool | Enables central logging | false | ANKA_ENABLE_CENTRAL_LOGGING |
| Push registry | string | Comma separated list of Registry addresses to use for push operations (saveImage/Jenkins cache building) | - | ANKA_PUSH_REGISTRY |
| ETCD defrag interval | duration | Defrag ETCD (all servers) in this interval. Pass 0 to disable | 3h | ANKA_DEFRAG_DB_INTERVAL |
| Instance time out | duration | The time that instances stay in 'Terminated' state | 1m | ANKA_INSTANCE_TIME_OUT |
| Manage MAC addresses | bool | Let the controller manage VM MAC addresses to ensure uniqueness/prevent collision. **`Requires VM Templates/Tags be stored in your Registry in a stopped state (vs suspended).`** | false | ANKA_MANAGE_MAC_ADDRESSES |
| Clean MAC addresses interval | duration | Interval between cleanings of unused MAC addresses | 1h | ANKA_CLEAN_MAC_ADDRESS_INTERVAL |

### Logging 

| Name | Type | Description | Default Value | ENV |
| --- | :---: | --- | :---: | :---: |
| Log level | int | Log level verbosity. Higher number means more verbose | 0 | ANKA_LOG_LEVEL |
| Log to stderr | bool | log to standard error instead of files | false | ANKA_LOGTOSTDERR |
| Log directory | string | Write log files in this directory | | ANKA_LOG_DIR |
| Also log to stderr| bool | Log to standard error as well as files | true | ANKA_ALSOLOGTOSTDERR |

### TLS

| Name | Type | Description | Default Value | ENV |  
| --- | :---: | --- | :---: | :---: |
Enable https | bool | Use https protocol for the controller portal/APIs. **Must pass this to enable TLS** | false | ANKA_USE_HTTPS 
CA certificate | string | Path to a CA cert to use for authenticating clients | - | ANKA_CA_CERT   
Root certificate | string | Similar to CA certificate | - | ANKA_ROOT_CERT 
Server certificate | string | Path to TLS server certificate | - | ANKA_SERVER_CERT
Server certificate key | string | Path to the server certificate's private key | - | ANKA_SERVER_KEY
Skip TLS verification | bool | Don't verify TLS certificates | false | ANKA_SKIP_TLS_VERIFICATION
Client certificate | string | Path to client certificate. The Controller will use this certificate when making http requests (mainly to the Registry). | - | ANKA_CLIENT_CERT
Client certificate key| string | Path to the client certificate's private key | - | ANKA_CLIENT_CERT_KEY
Client keystore| string | Path to a client keystore file in pkcs12 format. The Controller will use the certificate from this key store when making http requests (mainly to the Registry). | - | ANKA_CLIENT_KEYSTORE
Client keystore password | string | Password for the client keystore (optional). | - | ANKA_CLIENT_KEYPASS

### Built in Registry

| Name | Type | Description | Default Value | ENV | 
| --- | :---: | --- | :---: | :---: |
Run registry | bool | Run the embedded Registry server | false | ANKA_RUN_REGISTRY
Registry listen address| string | Address for Registry to listen on (:8089 is equivalent to 0.0.0.0:8089). Use the format `[address]:port` | :8089 | ANKA_REGISTRY_LISTEN_ADDRESS
Registry base path | string | Path for registry's data | - | ANKA_REGISTRY_BASE_PATH
Registry access logs| bool | Enables registry access logs | false | ANKA_REGISTRY_ACCESS_LOGS
Enable registry authorization| | Enables authorization for the Registry | false | ANKA_ENABLE_REGISTRY_AUTHORIZATION

### Built in ETCD

| Name | Type | Description | Default Value | ENV | 
| --- | :---: | --- | :---: | :---: |
Server name | string | Human readable name for ETCD server | anka-etcd | ANKA_NAME      
Data directory | string | Path to use for saving ETCD data | /tmp/etcd-data | ANKA_DATA_DIR  
Initial cluster| string | Initial cluster configuration for bootstrapping etcd server | anka-etcd=http://0.0.0.0:2380 | ANKA_INITIAL_CLUSTER
Listen peer urls| string | Comma separated URLs for ETCD server to server communication (when clustering ETCD) | http://0.0.0.0:2380 | ANKA_LISTEN_PEER_URLS
Initial advertise peer urls| string | Comma separated URLs for ETCD server to server communication to advertise | http://0.0.0.0:2380 | ANKA_INITIAL_ADVERTISE_PEER_URLS
Initial ETCD state| string | Initial ETCD cluster state ('new' or 'existing') | new | ANKA_INITIAL_CLUSTER_STATE
Initial ETCD token| string | Initial token for the ETCD cluster during bootstrap | etcd-server | ANKA_INITIAL_CLUSTER_TOKEN
Listen client urls| string | Comma separated URLs for ETCD to serve clients (Controller) | http://127.0.0.1:2379 | ANKA_LISTEN_CLIENT_URLS
Auto compaction mode| string | Auto compaction mode, either 'periodic' or 'revision'. | periodic | ANKA_AUTO_COMPACTION_MODE
Advertise client urls| string | Client urls for etcd server to advertise | http://127.0.0.1:2379 | ANKA_ADVERTISE_CLIENT_URLS
Compaction retention interval | string | Auto compaction retention length. 0 means disable auto compaction. | 30m | ANKA_AUTO_COMPACTION_RETENTION

### Authentication and Authorization 

| Name | Type | Description | Default Value | ENV | 
| --- | :---: | --- | :---: | :---: | 
Anable authentication | bool | Enable authentication module. **Must pass this for authentication to work** | false | ANKA_ENABLE_AUTH
Root static token | string | A token to authenticate as super user | - | ANKA_ROOT_TOKEN
OpenId connect display name| string | Name of open id server to display in login page. The text will say "Login with X" | - | ANKA_OIDC_DISPLAY_NAME
OpenId connect provider url| string | Open ID connect provider url | - | ANKA_OIDC_PROVIDER_URL
OpenId connect  client id | string | Open ID connect client id | - | ANKA_OIDC_CLIENT_ID
OpenId connect  username claim| string | Open ID connect claim key to use for user name | name | ANKA_OIDC_USERNAME_CLAIM
OpenId connect groups claim| string | Open ID connect claim key to use for groups, | groups | ANKA_OIDC_GROUPS_CLAIM

### Separate queue interface

> This is an advanced feature, it allows you to have a second http interface that will be used only by the cluster's Nodes

| Name | Type | Description | Default Value | ENV | 
| --- | :---: | --- | :---: | :---: | 
Queue address | string | Setting this address will activate a separate http server that will only serve queue requests (only for Node communication). | - | ANKA_QUEUE_ADDR
Queue CA certificate | string | Path to a CA certificate to use for authenticating clients | - | ANKA_QUEUE_CA_CERT
Queue server certificate| string | Path to TLS server certificate | - | ANKA_QUEUE_SERVER_CERT
Queue server certificate key| string | Path to the server certificate's private key | - | ANKA_QUEUE_SERVER_KEY
Use queue TLS | | Enables queue tls | false | ANKA_USE_QUEUE_TLS
Enable queue auth| | Enables queue authentication/authorization | false | ANKA_ENABLE_QUEUE_AUTH

### Internal

> Parameters used internally. It's recommended to use the Default Values.

| Name | Type | Description | Default Value | ENV | 
| --- | :---: | --- | :---: | :---: |
Clean process interval| duration | The interval to clean the queues (delete any tasks older than 24 hours), 0 to disable | 1h | ANKA_CLEAN_QUEUES_INTERVAL
allow cors | bool | If true adds Acces-Control-Allow-Origin to all routes | default | ANKA_ALLOW_CORS
Scheduler interval| duration | Interval for checking scheduled tasks | 30m | ANKA_SCHEDULER_INTERVAL
allowUnknownFlags| | Don't terminate the app if ini file contains unknown flags. | default | ANKA_ALLOWUNKNOWNFLAGS
Dump flags | bool | Dumps values for all flags defined in the app into stdout in ini-compatible syntax and terminates the app. | false | ANKA_DUMPFLAGS 

{{< /tab >}}
{{< tab tabNum="2" >}}

---

Depending on the package you're using (native or docker), you can include flags to modify the configuration of your controller and registry. However, you need to override the default entrypoint (you can find the default in the .docker file).

#### docker-compose.yml (docker)

```yml
  anka-controller:
    container_name: anka-controller
    build:
       context: .
       dockerfile: anka-controller.docker
    ports:
       - "8090:80"
    volumes:
       - /Users/myUser/:/mnt/cert
    depends_on:
       - etcd
       - anka-registry
    restart: always
    entrypoint: ["/bin/bash", "-c", "anka-controller --standalone --enable-central-logging --anka-registry http://anka.registry:8089 --etcd-endpoints etcd:2379 --log_dir /var/log/anka-controller --local-anka-registry http://anka-registry:8085"]

  anka-registry:
    container_name: anka-registry
    build:
        context: .
        dockerfile: anka-registry.docker
    ports:
        - "8091:8089"
    restart: always
    volumes:
      - "/Library/Application Support/Veertu/Anka/registry:/mnt/vol"
      - /Users/myUser/:/mnt/cert
    #environment:
      #ANKA_USE_HTTPS: "true"
      #ANKA_SKIP_TLS_VERIFICATION: "true"
      #ANKA_SERVER_CERT: "/mnt/cert/anka-controller-crt.pem"
      #ANKA_SERVER_KEY: "/mnt/cert/anka-controller-key.pem"
      #ANKA_CA_CERT: "/mnt/cert/anka-ca-crt.pem"
      #ANKA_ENABLE_AUTH: "false"
```

#### /usr/local/bin/anka-controllerd (native)

```bash
#!/bin/bash
LOG_DIR="/Library/Logs/Veertu/AnkaController"
LISTEN_ADDRESS=":8090"
DATA_DIR="/Library/Application Support/Veertu/Anka/anka-controller"
REGISTRY_BASE_PATH="/Library/Application Support/Veertu/Anka/registry"
/Library/Application\ Support/Veertu/Anka/bin/anka-controller \
--standalone \
--listen_addr "$LISTEN_ADDRESS" \
--enable-central-logging \
--log_dir "$LOG_DIR" \
--data-dir "$DATA_DIR" \
--run-registry \
--registry-base-path  "$REGISTRY_BASE_PATH" \
--registry-listen-address "anka.registry:8089" \
--anka-registry "http://anka.registry:8089"
```


### General and Common

| Name | Type | Description | Default Value | flag / opt |
| --- | :---: | --- | :---: | :---: |
| Version | bool | Prints controller version and exits | - | `--version` | 
| Registry address | string | Anka Registry external URL (http[s]://hostname:[port]). This is passed to the Nodes, so they can download (and start) VMs | (required) | `--anka-registry` |
| Configuration file | string | Path to a configuration file in INI format. You can use the file with/without the command line parameters and env variables | - | `--config` |
| Listen address | string | Listen on this address (:80 is equivalent to 0.0.0.0:80). Use the format `[address]:port` | :80 | `--listen_addr` |
| Local Registry Address | string | Anka Registry local address in format `http[s]://hostname:[port]`. This parameter is for situations where the Controller and Registry are on the same network. For example `http://locahost:8089` | - | `--local-anka-registry` |
| Number of concurrent workers | int | The number of concurrent workers processing node tasks | 2 | `--num-workers` |
| Standalone mode | bool | Run an embedded ETCD server alongside the controller | false | `--standalone` |
| ETCD endpoints | string | Comma separated list of etcd hosts | 127.0.0.1:2379 | `--etcd-endpoints` |
| Allow empty registry | bool | Allow controller to start without a 'Registry address' | false | `--allow-empty-registry` |
| Enable event logging | bool | Enables event logging. **`Requires a Enterprise Plus license and will show under the Controller's Logs section after the first instance is created.`** | false | `--enable-event-logging` |
| Event log url | string | The URL to post events (in json format) | - | `--event-log-url` |
| Enable central logging | bool | Enables central logging | false | `--enable-central-logging` |
| Push registry | string | Comma separated list of Registry addresses to use for push operations | - | `--push-registry` |
| ETCD defrag interval | duration | Defrag ETCD (all servers) in this interval. Pass 0 to disable | 3h | `--defrag-db-interval` |
| Instance time out | duration | The time that instances stay in 'Terminated' state | 1m | `--instance-time-out` |
| Manage MAC addresses | bool | Let the controller manage VM MAC addresses to ensure uniqueness/prevent collision. **`Requires VM Templates/Tags be stored in your Registry in a stopped state (vs suspended).`** | false | `--manage-mac-addresses` |
| Clean MAC addresses interval | duration | Interval between cleanings of unused MAC addresses | 1h | `--clean-mac-address-interval` |

### Logging 

| Name | Type | Description | Default Value | flag / opt |
| --- | :---: | --- | :---: | :---: |
| Log level | int | Log level verbosity. Higher number means more verbose | 0 | `--log-level` |
| Log to stderr | bool | log to standard error instead of files | false | `--logtostderr` |
| Log directory | string | Write log files in this directory | | `--log_dir` |
| Also log to stderr| bool | Log to standard error as well as files | true | `--alsologtostderr` |

### TLS

| Name | Type | Description | Default Value | flag / opt |  
| --- | :---: | --- | :---: | :---: |
Enable https | bool | Use https protocol for the controller portal/APIs. **Must pass this to enable TLS** | false | `--use-https` 
CA certificate | string | Path to a CA cert to use for authenticating clients | - | `--ca-cert`   
Root certificate | string | Alias of CA certificate | - | `--root-cert`
Server certificate | string | Path to TLS server certificate | - | `--server-cert`
Server certificate key | string | Path to the server certificate's private key | - | `--server-key`
Skip TLS verification | bool | Don't verify TLS certificates | false | `--skip-tls-verification`
Client certificate | string | Path to client certificate. The Controller will use this certificate when making http requests (mainly to the Registry). | - | `--client-cert`
Client certificate key| string | Path to the client certificate's private key | - | `--client-cert-key`
Client keystore| string | Path to a client keystore file in pkcs12 format. The Controller will use the certificate from this key store when making http requests (mainly to the Registry). | - | `--client-keystore`
Client keystore password | string | Password for the client keystore (optional). | - | `--client-keypass`

### Built in Registry

| Name | Type | Description | Default Value | flag / opt | 
| --- | :---: | --- | :---: | :---: |
Run registry | bool | Run the embedded Registry server | false | `--run-registry`
Registry listen address| string | Address for Registry to listen on (:8089 is equivalent to 0.0.0.0:8089). Use the format `[address]:port` | :8089 | `--registry-listen-address`
Registry base path | string | Path for registry's data | - | `--registry-base-path`
Registry access logs| bool | Enables registry access logs | false | `--registry-access-logs`
Enable registry authorization| | Enables authorization for the Registry | false | `--enable-registry-authorization`

### Built in ETCD

| Name | Type | Description | Default Value | flag / opt | 
| --- | :---: | --- | :---: | :---: |
Server name | string | Human readable name for ETCD server | anka-etcd | `--name`
Data directory | string | Path to use for saving ETCD data | /tmp/etcd-data | `--data-dir` 
Initial cluster| string | Initial cluster configuration for bootstrapping etcd server | anka-etcd=http://0.0.0.0:2380 | `--initial-cluster`
Listen peer urls| string | Comma separated URLs for ETCD server to server communication (when clustering ETCD) | http://0.0.0.0:2380 | `--listen-peer-urls`
Initial advertise peer urls| string | Comma separated URLs for ETCD server to server communication to advertise | http://0.0.0.0:2380 | `--initial-advertise-peer-urls`
Initial ETCD state| string | Initial ETCD cluster state ('new' or 'existing') | new | `--initial-cluster-state`
Initial ETCD token| string | Initial token for the ETCD cluster during bootstrap | etcd-server | `--initial-cluster-token`
Listen client urls| string | Comma separated URLs for ETCD to serve clients (Controller) | http://127.0.0.1:2379 | `--listen-client-urls`
Auto compaction mode| string | Auto compaction mode, either 'periodic' or 'revision'. | periodic | `--auto-compaction-mode`
Advertise client urls| string | Client urls for etcd server to advertise | http://127.0.0.1:2379 | `--advertise-client-urls`
Compaction retention interval | string | Auto compaction retention length. 0 means disable auto compaction. | 30m | `--auto-compaction-retention`

### Authentication and Authorization 

| Name | Type | Description | Default Value | flag / opt | 
| --- | :---: | --- | :---: | :---: |
Anable authentication | bool | Enable authentication module. **Must pass this for authentication to work** | false | `--enable-auth`
Root static token | string | A token to authenticate as super user | - | `--root-token`
OpenId connect display name| string | Name of open id server to display in login page. The text will say "Login with X" | - | `--oidc-display-name`
OpenId connect provider url| string | Open ID connect provider url | - | `--oidc-provider-url`
OpenId connect  client id | string | Open ID connect client id | - | `--oidc-client-id`
OpenId connect  username claim| string | Open ID connect claim key to use for user name | name | `--oidc-username-claim`
OpenId connect groups claim| string | Open ID connect claim key to use for groups, | groups | `--oidc-groups-claim`

### Separate queue interface

> This is an advanced feature, it allows you to have a second http interface that will be used only by the cluster's Nodes

| Name | Type | Description | Default Value | flag / opt | 
| --- | :---: | --- | :---: | :---: |
Queue address | string | Setting this address will activate a separate http server that will only serve queue requests (only for Node communication). | - | `--queue-addr`
Queue CA certificate | string | Path to a CA certificate to use for authenticating clients | - | `--queue-ca-cert`
Queue server certificate| string | Path to TLS server certificate | - | `--queue-server-cert`
Queue server certificate key| string | Path to the server certificate's private key | - | `--queue-server-key`
Use queue TLS | | Enables queue tls | false | `--use-queue-tls`
Enable queue auth| | Enables queue authentication/authorization | false | `--enable-queue-auth`

### Internal

> Parameters used internally. It's recommended to use the Default Values.

| Name | Type | Description | Default Value | flag / opt | 
| --- | :---: | --- | :---: | :---: |
Clean process interval| duration | The interval to clean the queues (delete any tasks older than 24 hours), 0 to disable | 1h | `--clean-queues-interval`
allow cors | bool | If true adds Acces-Control-Allow-Origin to all routes | default | `--allow-cors`
Scheduler interval| duration | Interval for checking scheduled tasks | 30m | `--scheduler-interval`
allowUnknownFlags| | Don't terminate the app if ini file contains unknown flags. | default | `--allowUnknownFlags`
Dump flags | bool | Dumps values for all flags defined in the app into stdout in ini-compatible syntax and terminates the app. | false | `--dumpflags`

{{< /tab >}}
{{< /tabs >}}