# Mutual Authentication Protected Server
Proof of concept of implementing 2-way SSL Verification in NGINX (thus in any tech stack) demonstrated by using docker.
The dockerfile installs php and sets up a simple php server.

### Setup Instructions

After cloning, create a .env file based on the ``.env.example`` and fill in ``volume_path`` (which will likely contain the current folder where you placed this repository locally), and the ``ip`` you want it accessible (let's say for ex. 192.168.13.1)

Create image, container, and run it with:
```bash
docker-compose build
docker-compose up -d
```

### Testing / Proof of Concept

#### Test without using a certificate (Should Fail)

Try accessing via cUrl
```bash
curl --insecure https://192.168.13.1
```

This should result in an error message:
__400 Bad Request__
No required SSL certificate was sent

Note that we use --insecure since we're only using a self signed certificate in our server's SSL. This is fine as long as it's only accessed internally by the organization. Just make sure to secure the generated ``ca.key`` file.

#### Test without using a certificate that's not signed by the server's CA (Should Fail)

Generate a certificate
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
	-subj "/C=PH/ST=NCR/L=Pasig/O=NuWorks Interactive Labs Inc./OU=Tech Dept./CN=Unsigned Cert" \
	-keyout unsigned-cert.key -out unsigned-cert.crt
```

Try accessing via cUrl

```bash
curl --insecure --cert ./unsigned-cert.crt --key ./unsigned-cert.key https://192.168.13.1
```

This should result in an error message:
__400 Bad Request__
No required SSL certificate was sent

#### Test without using a certificate that's signed by the CA (Should be successful)

For demonstration purposes, this dockerfile is set to generate a signed certificate using the server's CA (Certificate Authority). It's located at ``/root/certs/client/``

Copy the file via scp if this is in a real server with scp support
```bash
scp root@192.168.13.1:/root/certs/client/client.crt .
scp root@192.168.13.1:/root/certs/client/client.key .
```

Or if in docker, just manually copy the files to outside the container by copying it the directory you set using ``volume_path`` to expose the files.

Test the connection using cUrl

```bash
curl --insecure --cert ./client.crt --key ./client.key https://192.168.13.1
```

Or create a pem file and use the combined certificate and key using

```bash
cat client.crt client.key > client.pem
curl --insecure --cert ./client.pem https://192.168.13.1
```

Both method should now work and output the response of this repository's ``index.php`` file.
