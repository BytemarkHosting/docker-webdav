## Supported tags

* [`2.4`, `latest` (*2.4/Dockerfile*)](https://github.com/BytemarkHosting/docker-webdav/blob/master/2.4/Dockerfile)

## Quick reference

This image runs an easily configurable WebDAV server with Apache.

You can configure the authentication type, the authentication of multiple users, or to run with a self-signed SSL certificate. If you want a Let's Encrypt certificate, see an example of how to do that [here](https://github.com/BytemarkHosting/configs-webdav-docker).

* **Code repository:**
  https://github.com/BytemarkHosting/docker-webdav
* **Where to file issues:**
  https://github.com/BytemarkHosting/docker-webdav/issues
* **Maintained by:**
  [Bytemark Hosting](https://www.bytemark.co.uk)
* **Supported architectures:**
  [Any architecture that the `httpd` image supports](https://hub.docker.com/_/httpd/)

## Usage

### Basic WebDAV server

This example starts a WebDAV server on port 80. It can only be accessed with a single username and password.

When using unencrypted HTTP, use `Digest` authentication (instead of `Basic`) to avoid sending plaintext passwords in the clear.

To make sure your data doesn't get deleted, you'll probably want to create a persistent storage volume (`-v vol-webdav:/var/lib/dav`) or bind mount a directory (`-v /path/to/directory:/var/lib/dav`):

```
docker run --restart always -v /srv/dav:/var/lib/dav \
    -e AUTH_TYPE=Digest -e USERNAME=alice -e PASSWORD=secret1234 \
    --publish 80:80 -d bytemark/webdav

```

#### Via Docker Compose:

```
version: '3'
services:
  webdav:
    image: bytemark/webdav
    restart: always
    ports:
      - "80:80"
    environment:
      AUTH_TYPE: Digest
      USERNAME: alice
      PASSWORD: secret1234
    volumes:
      - /srv/dav:/var/lib/dav

```
### Secure WebDAV with SSL

We recommend you use a reverse proxy (eg, Traefik) to handle SSL certificates. You can see an example of how to do that [here](https://github.com/BytemarkHosting/configs-webdav-docker).

If you're happy with a self-signed SSL certificate, specify `-e SSL_CERT=selfsigned` and the container will generate one for you.

```
docker run --restart always -v /srv/dav:/var/lib/dav \
    -e AUTH_TYPE=Basic -e USERNAME=test -e PASSWORD=test \
    -e SSL_CERT=selfsigned --publish 443:443 -d bytemark/webdav

```

If you bind mount a certificate chain to `/cert.pem` and a private key to `/privkey.pem`, the container will use that instead!

### Authenticate multiple clients

Specifying `USERNAME` and `PASSWORD` only supports a single user. If you want to have lots of different logins for various users, bind mount your own file to `/user.passwd` and the container will use that instead.

If using `Basic` authentication, run the following commands:

```
touch user.passwd
htpasswd -B user.passwd alice
htpasswd -B user.passwd bob

```

If using `Digest` authentication, run the following commands. (NB: The default `REALM` is `WebDAV`. If you specify your own `REALM`, you'll need to run `htdigest` again with the new name.)


```
touch user.passwd
htdigest user.passwd WebDAV alice
htdigest user.passwd WebDAV bob

```

Once you've created your own `user.passwd`, bind mount it into your container with `-v /path/to/user.passwd:/user.passwd`.

### Environment variables

All environment variables are optional. You probably want to at least specify `USERNAME` and `PASSWORD` (or bind mount your own authentication file to `/user.passwd`) otherwise nobody will be able to access your WebDAV server!

* **`SERVER_NAMES`**: Comma-separated list of domains (eg, `example.com,www.example.com`). The first is set as the [ServerName](https://httpd.apache.org/docs/current/mod/core.html#servername), and the rest (if any) are set as [ServerAlias](https://httpd.apache.org/docs/current/mod/core.html#serveralias). The default is `localhost`.
* **`LOCATION`**: The URL path for WebDAV (eg, if set to `/webdav` then clients should connect to `example.com/webdav`). The default is `/`.
* **`AUTH_TYPE`**: Apache authentication type to use. This can be `Basic` (best choice for HTTPS) or `Digest` (best choice for HTTP). The default is `Basic`.
* **`REALM`**: Sets [AuthName](https://httpd.apache.org/docs/current/mod/mod_authn_core.html#authname), an identifier that is displayed to clients when they connect. The default is `WebDAV`.
* **`USERNAME`**: Authenticate with this username (and the password below). This is ignored if you bind mount your own authentication file to `/user.passwd`.
* **`PASSWORD`**: Authenticate with this password (and the username above). This is ignored if you bind mount your own authentication file to `/user.passwd`.
* **`ANONYMOUS_METHODS`**: Comma-separated list of HTTP request methods (eg, `GET,POST,OPTIONS,PROPFIND`). Clients can use any method you specify here without authentication. Set to `ALL` to disable authentication. The default is to disallow any anonymous access.
* **`SSL_CERT`**: Set to `selfsigned` to generate a self-signed certificate and enable Apache's SSL module. If you specify `SERVER_NAMES`, the first domain is set as the Common Name.

