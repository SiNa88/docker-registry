# Private Docker registry

Setting up a private Docker registry on a MinIO object repository

## Running a docker registry with Minio S3 backend

### Create config.yml for Docker Registry

This file will have to be mounted to /etc/docker/registry/config.yml

```
version: 0.1
log:
  level: info
  formatter: text
  fields:
    service: docker-reg
    environment: production
loglevel: debug
storage:
  s3:
    accesskey: <minio access key>
    secretkey: <minio secret key>
    region: eu-south-1
    regionendpoint: https://<docker host ip running registry>:9000/
    secure: true
    bucket: docker-registry
    encrypt: false
    v4auth: true
    chunksize: 5242880
    rootdirectory: /
  delete:
    enabled: true
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
    readonly:
      enabled: false
http:
  addr: :5000
  secret: ?????
  tls:
      certificate: /etc/minio/certs/public.crt
      key:         /etc/minio/certs/private.key
auth:
  htpasswd:
      realm: "Private docker registry"
      path: /etc/docker/registry/auth/registry.password
```

### Run minio in as a container

```
sudo docker run -d \
   -p 9000:9000 \
   -p 9090:9090 \
   --name minio \
   -v ~/minio/data:/data \
   -e "MINIO_ROOT_USER=????" \
   -e "MINIO_ROOT_PASSWORD=????" \
   quay.io/minio/minio server /data --console-address ":9090"
```


### Use docker logs to retrieve access key and secret key from minio container
`docker logs minio`


### Run Docker registry in a container
```
docker run -d -p 5000:5000 \
  -v /etc/docker/registry/config.yml:/etc/docker/registry/config.yml \
  -v /etc/minio/certs/:/certs \
  --name=registry registry:2
```

### Tag some image to push to the registry
`docker tag alpine:3.10 <docker host ip running registry>:5000/alpine:3.10`

`docker push <docker host ip running registry>:5000/alpine:3.10`



### Create username and password for user authenticated through `docker login` based on [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-20-04)


```
  sudo apt install apache2-utils -y 
  mkdir /etc/docker/registry/auth
  cd /etc/docker/registry/auth
  htpasswd -Bc registry.password username
  htpasswd -B registry.password username
```
