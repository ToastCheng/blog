---
title: "Firestore: Emulator"
date: 2021-01-11T15:22:00+0800
tags: [note, firestore, emulator]
categories: technique
---


There is an existing docker image [firestore-emulator](https://hub.docker.com/r/mtlynch/firestore-emulator/dockerfile), that can spinup the firestore emulator, just run:
```
docker run \
    -d \
    --rm \
    --name firestore-emulator \
    --env "FIRESTORE_PROJECT_ID=test-project" \
    --env "PORT=8080" \
    mtlynch/firestore-emulator
```

In case you want a newer version of firestore emulator, you can clone the git repository of this docker image and build with the desired gcloud sdk version.
The first line of dockerfile is:

```dockerfile
ARG GCLOUD_SDK_VERSION=271.0.0-alpine
```

Notice that the `ARG` let you decide your `GCLOUD_SDK_VERSION` at build time.

To build a desired version is simple as follow:
```bash
git clone git@github.com:mtlynch/firestore-emulator-docker.git
cd firestore-emulator-docker

# build with specific gcloud sdk version, e.g. 322.0.0.-alpine
docker build -f Dockerfile . \
    --build-arg GCLOUD_SDK_VERSION=322.0.0-alpine \
    --tag firestore-emulator
```

In the client side code, you need to initialize Firestore client with different configuration. 
- projectId: should match the environment variable `FIRESTORE_PROJECT_ID` you passed into docker run.
- host: should be the environment variable `PORT` you passed into docker run.
- ssl: should be `false` since we are using plain text to communicate with emulator.

The code looks like this:
```javascript
const client = new Firestore({ projectId: 'test-project', host: 'localhost:8080', ssl: false });
// use it as usual
const snapshot = await client.collection('col').doc('doc').get();
await client.collection('col').doc('doc').set({ field: 'value' });
```

When you are running multiple transactions on emulator and causing dead lock, firestore emulator needs more time to abort transaction and retry, it could take 30 second or more. For more details, see [#2452](https://github.com/firebase/firebase-tools/issues/2452), [#2604](https://github.com/googleapis/google-cloud-go/issues/2604).

So if you are running emulator for unittesting, make sure you set the timeout high enough to let the test pass. If you are using `mocha`, it should look something like this:
```bash
mocha \
    --timeout=60000 \
    --exit \
    --require ts-node/register \
    ./test/firestore/*.test.ts
```
