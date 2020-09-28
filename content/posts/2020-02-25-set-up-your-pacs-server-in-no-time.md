---
title:  "Orthanc: setup your PACS server in no time"
date:   2020-02-25T13:32:56+0800
tags: [orthanc, pacs]
categories: technique
---
This article demonstrate how you can setup a PACS server using docker. 

### Using Docker
```shell
docker run \
    --rm \
    -p 4242:4242 \
    -p 8042:8042 \
    -v {PATH_IN_YOUR_DISK}:/var/lib/orthanc/db/ \
    osimis/orthanc:20.2.0
```
- `8042` is the default port that orthanc serve. Use `-p` to export it.
- `{PATH_IN_YOUR_DISK}` is an arbitrary path that in host disk, which `Orthanc` will save the dicom information inside it, make sure you create it before you run. `/var/lib/orthanc/db/` is the default path live inside orthanc image.

### Using Docker Compose

It is pretty similar to the `docker run` command.
```yaml
version: "3.7"
services:
  orthanc:
    image: osimis/orthanc:20.2.0
    restart: always
    ports:
      - 4242:4242
      - 8042:8042
    volumes:
      - {PATH_IN_YOUR_DISK}:/var/lib/orthanc/db/
```


Once you can successfully run the Orthanc docker image, you might wonder how to configurate it.

### Configuration

#### Method 1 -- via ENV
You can configure Orthanc for docker by setting the environment variable just like other docker image such as `mysql`, it's a normal way to setup your docker image.

Below are the frequently used variable name:
- `AC_AUTHENTICATION_ENABLED`
- `AC_REGISTERED_USERS`



#### Method 2 -- via hardcode

Though setting ENV is pretty straight forward, but sometimes:
1. You have a lot to configure.
2. You just cannot find the correct environment variable you want to set.
3. The variables are multiline, like `AC_REGISTERED_USERS`, which is hard to set:
```
  AC_REGISTERED_USERS=
  {
  "foo": "baz"
  }
```
For these scenario, customize the configuration file is another option. It is the old way to setup hence it's well documented, see [here](https://book.orthanc-server.com/users/configuration.html#configuration) for documentations and [here](https://bitbucket.org/sjodogne/orthanc/raw/Orthanc-1.5.8/Resources/Configuration.json) for an example.

There are four existed configure files:
- `/etc/orthanc/http.json`
```json
{
    "HttpTimeout": 0,
    "HttpsVerifyPeers": true,
    "HttpsCACertificates" : "/etc/ssl/certs/ca-certificates.crt",
    "HttpProxy": "",
    "HttpPort": 8042,
    "HttpVerbose": false,
    "KeepAlive": true,
    "TcpNoDelay": true,
    "HttpRequestTimeout": 30
}
```
- `/etc/orthanc/remote-access.json`
```json
{
    "RemoteAccessAllowed": true,
    "AuthenticationEnabled": true,
    "RegisteredUsers": {}
}
```
- `/etc/orthanc/storage.json`
```json
{
    "StorageDirectory": "/var/lib/orthanc/db",
    "MaximumStorageSize": 0,
    "MaximumPatientCount": 0,
    "StoreDicom": true
}
```
- `/etc/orthanc/plugins.json`
```json
{
    "Plugins": ["/usr/share/orthanc/plugins"]
}
```
Note that you **cannot** set the same key in two (or more) different config files. So if you add `RegisteredUsers` in your customized config `orthanc.json`, it will conflict with one in `remote-access.json`. Set it in `remote-access.json` instead.

You can edit those files or create new one inside `/etc/orthanc`. There are some ways to do that:

#### Add in Dockerfile
```docker
FROM osimis/orthanc:20.2.0
COPY {YOUR_CONFIG_FILE_NAME}.json /etc/orthanc/
```
and 
```shell
docker build -f {DOCKERFILE} -t local/orthanc
docker run \
    --rm \
    -p 4242:4242 \
    -p 8042:8042 \
    -v {PATH_IN_YOUR_DISK}:/var/lib/orthanc/db/ \
    local/orthanc
```
But it might be tedious to build it, so you can consider mounting the complete configuration:

#### Mount when `docker run`

Make sure you have all the necessary settings in `{CUSTOMIZE_CONFIG}/`, which includes:

- `/etc/orthanc/http.json`
- `/etc/orthanc/remote-access.json`
- `/etc/orthanc/storage.json`
- `/etc/orthanc/plugins.json`

and
```shell
docker run \
    --rm \
    -p 4242:4242 \
    -p 8042:8042 \
    -v {PATH_IN_YOUR_DISK}:/var/lib/orthanc/db/ \
    -v {CUSTOMIZE_CONFIG}:/etc/orthanc \
    osimis/orthanc:20.2.0
```


This way you will not have to build the docker image. And you are good to go!