# Docker Registry using StorageGRID S3 backend storage

Setup a private Docker Registry using S3 backend as storage with SSL and username/password protected.

## Pre-requisites

* git
* docker
* docker-compose

## Installation

1. Clone this:

    ```sh
    git clone https://github.com/adlytaibi/sg-registry
    ```

    ```sh
    cd sg-registry
    ```

2. Generate self-signed SSL certificate

    ```sh
    mkdir sslkeys
    ```

    1. Create private key, generate a certificate signing request

        ```sh
        openssl genrsa -out sslkeys/registry.key 2048
        ```

    2. Self-sign your own certificates: (modify `web` to match your server)

        ```sh
        openssl req -x509 -nodes -newkey rsa:4096 -keyout sslkeys/registry.key -out sslkeys/registry.pem -days 365 -subj "/C=CA/ST=Ontario/L=Toronto/O=Storage/OU=Team/CN=registry"
        ```

3. Create `htpasswd` user and password

   ```sh
   htpasswd -Bbc sslkeys/htpasswd registry registry
   ```

   Or input user's password:

   ```sh
   htpasswd -Bc sslkeys/htpasswd registry
   New password: 
   Re-type new password: 
   ```

4. Edit `docker-compose.yml` and fill-in the following parameters

   ```sh
   REGISTRY_STORAGE_S3_ACCESSKEY: access key
   REGISTRY_STORAGE_S3_SECRETKEY: secret key
   REGISTRY_STORAGE_S3_BUCKET: bucket name
   REGISTRY_STORAGE_S3_REGION: us-east-1
   REGISTRY_STORAGE_S3_REGIONENDPOINT: StorageGRID gateway or storage node
   ```

5. Start the Docker Registry container

   ```sh
   docker-compose up -d
   ```

6. Pull a new image or tag existing image to push to the new Docker Registry

   1. (Optional) Pull an image from an exiting registry

      ```sh
      docker pull busybox
      Using default tag: latest
      latest: Pulling from library/busybox
      Digest: sha256:6915be4043561d64e0ab0f8f098dc2ac48e077fe23f488ac24b665166898115a
      ```

   2. Tag the image you want to push to the new registry

      ```sh
      docker tag busybox localhost:5555/busybox
      ```
7. Login to the new Docker Registry

   ```sh
   echo registry|docker login -u registry --password-stdin localhost:5555
   Login Succeeded
   ```

   Or input user's password:

   ```sh
   docker login -u registry localhost:5555
   Password: 
   Login Succeeded
   ```

8. Push image to the new Docker Registry

   ```sh
   docker push localhost:5555/busybox
   The push refers to repository [localhost:5555/busybox]
   195be5f8be1d: Pushed 
   latest: digest: sha256:edafc0a0fb057813850d1ba44014914ca02d671ae247107ca70c94db686e7de6 size: 527
   ```

## Tear-down


* Stop and delete the Docker Registry

  ```sh
  docker-compose down
  ```

* You can then delete Registry image(s).

## Troubleshooting:

* Testing with no credentials provided yields an `Unauthorized 401` status code.

  ```sh
  curl -kI -XGET https://localhost:5555/v2/_catalog
  HTTP/2 401 
  content-type: application/json; charset=utf-8
  docker-distribution-api-version: registry/2.0
  www-authenticate: Basic realm="basic"
  x-content-type-options: nosniff
  content-length: 145
  date: Thu, 06 Feb 2020 13:13:35 GMT
  ```

* Testing credentials for registry user yields an `OK 200` status code.

  ```sh
  curl -kI -u registry -XGET https://localhost:5555/v2/_catalog
  Enter host password for user 'registry':
  HTTP/2 200 
  content-type: application/json; charset=utf-8
  docker-distribution-api-version: registry/2.0
  x-content-type-options: nosniff
  content-length: 29
  date: Thu, 06 Feb 2020 13:13:51 GMT
  ```

* When not logged in, a login to the registry server is requested.

  ```sh
  docker push localhost:5555/busybox
  The push refers to repository [localhost:5555/busybox]
  195be5f8be1d: Preparing 
  no basic auth credentials
  ```

## Further reading
* [Docker Compose](https://docs.docker.com/compose/)
* [Deploy a registry server](https://docs.docker.com/registry/deploying/)
* [Configuring a registry](https://docs.docker.com/registry/configuration/)

## Notes
This is not an official NetApp repository. NetApp Inc. is not affiliated with the posted examples in any way.
