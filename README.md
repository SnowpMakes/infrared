<p align="center">
   <img width="300" height="auto" src="https://i.imgur.com/sD8cjJc.png">
 </p>

# Infrared - a Minecraft Proxy

fork from [haveachin/infrared](https://github.com/haveachin/infrared)

## Added/changed Features

- Default placeholder for invalid domain and kick message
- Antibot based on geoip lookups and username lookups
- Added handshakes and blocked connections to prometheus exporter
- Allow multiple domains in 1 configfile
- Global .json config

## Command-Line Flags

`-config-path` specifies the path to all your server configs [default: `"./configs/"`]

`-receive-proxy-protocol` if Infrared should be able to receive proxy protocol [default: `false`]

`-enable-prometheus` enables the Prometheus stats exporter [default: `false`]

`-prometheus-bind` specifies what the Prometheus HTTP server should bind to [default: `:9100`]

### Example Usage

`./infrared -config-path="." -receive-proxy-protocol=true -enable-prometheus -prometheus-bind="localhost:9123"`

## Proxy Config

| Field Name        | Type    | Required | Default                                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|-------------------|---------|----------|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| domainNames       | String[]  | true     | localhost                                      | Should be [fully qualified domain name](https://en.wikipedia.org/wiki/Domain_name). <br>Note: Every string is accepted. So `localhost` is also valid.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| listenTo          | String  | true     | :25565                                         | The address (usually just the port; so short term `:port`) that the proxy should listen to for incoming connections.<br>Accepts basically every address format you throw at it. Valid examples: `:25565`, `localhost:25565`, `0.0.0.0:25565`, `127.0.0.1:25565`, `example.de:25565`                                                                                                                                                                                                                                                                                                        |
| proxyTo           | String  | true     |                                                | The address that the proxy should send incoming connections to. Accepts Same formats as the `listenTo` field.                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| proxyBind         | String  | false    |                                                | The local IP that is being used to dail to the server on `proxyTo`. (Same as Nginx `proxy-bind`)                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| disconnectMessage | String  | false    | Sorry {{username}}, but the server is offline. | The message a client sees when he gets disconnected from Infrared due to the server on `proxyTo` won't respond. Currently available placeholders:<br>- `username` the username of player that tries to connect<br>- `now` the current server time<br>- `remoteAddress` the address of the client that tries to connect<br>- `localAddress` the local address of the server<br>- `domain` the domain of the proxy (same as `domainName`)<br>- `proxyTo` the address that the proxy proxies to (same as `proxyTo`)<br>- `listenTo` the address that Infrared listens on (same as `listenTo`) |
| timeout           | Integer | true     | 1000                                           | The time in milliseconds for the proxy to wait for a ping response before the host (the address you proxyTo) will be declared as offline. This "online check" will be resend for every new connection.                                                                                                                                                                                                                                                                                                                                                                                     |
| proxyProtocol     | Boolean | false    | false                                          | If Infrared should use HAProxy's Proxy Protocol for IP **forwarding**.<br>Warning: You should only ever set this to true if you now that the server you `proxyTo` is compatible.                                                                                                                                                                                                                                                                                                                                                                                                           |
| realIp            | Boolean | false    | false                                          | If Infrared should use TCPShield/RealIP Protocol for IP **forwarding**.<br>Warning: You should only ever set this to true if you now that the server you `proxyTo` is compatible.                                                                                                                                                                                                                                                                                                                                                                                                          |
| docker            | Object  | false    | See [Docker](#Docker)                          | Optional Docker configuration to automatically start a container and stop it again if unused.  <br>Note: Infrared will not take direct connections into account. Be sure to route all traffic that connects to the container through Infrared.                                                                                                                                                                                                                                                                                                                                             |
| onlineStatus      | Object  | false    |                                                | This is the response that Infrared will give when a client asks for the server status and the server is online.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| offlineStatus     | Object  | false    | See [Response Status](#response-status)        | This is the response that Infrared will give when a client asks for the server status and the server is offline.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| callbackServer    | Object  | false    | See [Callback Server](#callback-server)        | Optional callback server configuration to send events as a POST request to a specified URL.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

### Response Status

| Field Name     | Type    | Required | Default         | Description                                                                                                                                          |
|----------------|---------|----------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| versionName    | String  | false    | Infrared 1.18 | The version name of the Minecraft Server.                                                                                                            |
| protocolNumber | Integer | true     | 757             | The protocol version number.                                                                                                                         |
| maxPlayers     | Integer | false    | 20              | The maximum number of players that can join the server.<br>Note: Infrared will not limit more players from joining. This number is just for display. |
| playersOnline  | Integer | false    | 0               | The number of online players.<br>Note: Infrared will not that this number is also just for display.                                                  |
| playerSamples  | Array   | false    |                 | An array of player samples. See [Player Sample](#Player Sample).                                                                                     |
| iconPath       | String  | false    |                 | The path to the server icon.                                                                                                                         |
| motd           | String  | false    |                 | The motto of the day, short MOTD.                                                                                                                    |

### Examples

#### Minimal Config

<details>
<summary>min.example.com</summary>

```json
{
  "domainNames": ["mc.example.com", "example.com"],
  "proxyTo": ":8080"
}
```

</details>

#### Full Config

<details>
<summary>full.example.com</summary>

```json
{
  "domainNames": ["mc.example.com", "example.com"],
  "listenTo": ":25565",
  "proxyTo": ":8080",
  "proxyBind": "0.0.0.0",
  "proxyProtocol": false,
  "realIp": false,
  "timeout": 1000,
  "disconnectMessage": "Username: {{username}}\nNow: {{now}}\nRemoteAddress: {{remoteAddress}}\nLocalAddress: {{localAddress}}\nDomain: {{domain}}\nProxyTo: {{proxyTo}}\nListenTo: {{listenTo}}",
  "docker": {
    "dnsServer": "127.0.0.11",
    "containerName": "mc",
    "timeout": 30000,
    "portainer": {
      "address": "localhost:9000",
      "endpointId": "1",
      "username": "admin",
      "password": "foobar"
    }
  },
  "onlineStatus": {
    "versionName": "1.18",
    "protocolNumber": 757,
    "maxPlayers": 20,
    "playersOnline": 2,
    "playerSamples": [
      {
        "name": "Steve",
        "uuid": "8667ba71-b85a-4004-af54-457a9734eed7"
      },
      {
        "name": "Alex",
        "uuid": "ec561538-f3fd-461d-aff5-086b22154bce"
      }
    ],
    "motd": "Join us!"
  },
  "offlineStatus": {
    "versionName": "1.18",
    "protocolNumber": 757,
    "maxPlayers": 20,
    "playersOnline": 0,
    "motd": "Server is currently offline"
  },
  "callbackServer": {
    "url": "https://mc.example.com/callback",
    "events": [
      "Error",
      "PlayerJoin",
      "PlayerLeave",
      "ContainerStart",
      "ContainerStop"
    ]
  }
}
```

</details>

## Prometheus exporter
The built-in prometheus exporter can be used to view metrics about infrareds operation.  
When the command line flag `-enable-prometheus` is enabled it will bind to `:9100` by default, if you would like to use another port or use an application like [node_exporter](https://github.com/prometheus/node_exporter) that also uses port 9100 on the same machine you can change the port with the `-prometheus-bind` command line flag, example: `-prometheus-bind=":9070"`.  
It is recommended to firewall the prometheus exporter with an application like *ufw* or *iptables* to make it only accessible by your own Prometheus instance.
### Prometheus configuration:
Example prometheus.yml configuration:
```yaml
scrape_configs:
  - job_name: infrared
    static_configs:
    - targets: ['infrared-exporter-hostname:port']
```

### Metrics:
* infrared_connected: show the amount of connected players per instance and proxy:
  * **Example response:** `infrared_connected{host="proxy.example.com",instance="vps1.example.com:9070",job="infrared"} 10`
  * **host:** listenTo domain as specified in the infrared configuration.
  * **instance:** what infrared instance the amount of players are connected to.
  * **job:** what job was specified in the prometheus configuration.
* infrared_handshakes: counter of the number of handshake packets received per instande, type and target:
  * **Example response:** `infrared_handshakes{instance="vps1.example.com:9070",type="status",host="proxy.example.com"} 5`
  * **instance:** what infrared instance handshakes were received on.
  * **type:** the type of handshake received; "status", "login", "cancelled_host" and "cancelled_name".
  * **country:** country where the player ip is from.
  * **host:** the target host specified by the "Server Address" field in the handshake packet. [[1]](https://wiki.vg/Protocol#Handshaking)
