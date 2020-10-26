RTMPAuthd
=========

RTMPAuthd is an authentication & notification system to be used along side the nginx rtmp module.

## Background
RTMPAuthd project was built to serve the needs of a small Discord community to allow high quality video streaming to a private rtmp server to remain social during the COVID-19 pandemic. When a member starts streaming, a notification is posted in discord as well as when the stream gains or loses a viewer.  
Each member may also have a twitch channel configured in RTMPAuthd which differs from their discord/publisher user name. When a twitch channel is defined for a member, notifications will be posted when the twitch stream is live/off-line. This functionality can also serve as general twitch notifications for favorite streamers of the discord community.

## Features
- Simple authentication for NGiNX RTMP module
- Discord channel notifications via webhook
- Twitch stream notifications
- HTTP REST user management
- Embedded database
- Single binary deployment

## Configuration
The project is configured with environment variables. All required exported variables are provided with defaults in `init/rtmpauthd.env`.

1. Create a local copy of the file
    ```
    mkdir /etc/rtmpauthd
    rtmpauthd -environment > /etc/rtmpauthd/rtmpauthd.env
    ```
2. Update the variables to suit your needs

## Install Service
Installation documentation WIP

TL;DR - compile project to a binary and either setup as a service with systemd or deploy the project in a container. Ensure all environment variables are configured from the previous section of this document.


A basic systemd unit-file can be generated with the following command
```
rtmpauthd -unitfile > /etc/systemd/system/rtmpauthd.service
systemctl daemon-reload
```

## Managing RTMP Publishers
User management can be performed with some basic REST calls. You can build a custom application around the API or you can simply interact with via your favorite REST client. For the sake of simplicity, the following examples will be demonstrated using the `curl` command.
NOTE: The primary key for all records in database is the publisher _name_ (discord user name)

### Adding/Updating a publisher
```
curl -X POST -d '{"name": "discord_username", "key": "private_rtmp_stream_key"}' http://127.0.0.1:9090/api/publisher
```
expected response status code: `204`

Optionally, If a user would also like to provide notifications for their public twitch stream:
```
curl -X POST -d '{"name": "discord_username", "key": "private_rtmp_stream_key", "twitch_stream": "twitch_username"}' http://127.0.0.1:9090/api/publisher
```
expected response status code: `204`

### Retrieve all publishers
```
curl http://127.0.0.1:9090/api/publisher
```

expected response status code: `200`
```
[
  {
    "name": "discord_username",
    "key": "abcdefghijklmnopqrstuvwxyz0123456789",
    "twitch_stream": "twitch_username"
  }
]
```

### Retrieve a single publisher
```
curl http://127.0.0.1:9090/api/publisher?name=discord_username
```

expected response status code: `200`
```
{
    "name": "discord_username",
    "key": "abcdefghijklmnopqrstuvwxyz0123456789",
    "twitch_stream": "twitch_username"
}
```

### Deleting a publisher
```
curl -X DELETE -d '{"name": "discord_username"}' http://127.0.0.1:9090/api/publisher
```

expected response status code: `204`

## Build From Source
If you would rather compile the project from source, please install the latest version of the Go programming language  [here](https://golang.org/dl/).
```
GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o rtmpauthd main.go
```

## Security considerations
While it is possible to run this service on a different host, it is intended to run on the same host/container pod as nginx and communicate via localhost. Due to this assumption, the rtmpauthd service should NOT be publicly accessible or firewall rules should be configured to only allow connection from the nginx host/container.
