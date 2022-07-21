# Installation

The latest version can be downloaded from the Releases page:  [https://github.com/sasjs/server/releases](https://github.com/sasjs/server/releases)

It can also be installed in just two lines of code (on linux):

```bash
curl -L https://github.com/sasjs/server/releases/latest/download/linux.zip > linux.zip
unzip linux.zip
```

For windows, just download the zip file and launch.  You can ignore / cancel the popup, the app does NOT require administrative privileges.

## Configuration

After installation, there is only one mandatory input:

* `SAS_PATH` -> the full path to the SAS executable in your environment

However there are many additional [settings](/settings) you can make - these can go in a file called `.env` in the same folder as the executable.

A full guide to deploying SASjs Server on a VPS is also available [here](https://sasapps.io/sasjs-server-on-vps).


## SSL Certificates

To run over https, SASjs server needs a copy of your certificates.  The path to these certificates should be provided in the [`PRIVATE_KEY`](/settings/#private_key) and [`CERT_CHAIN`](/settings/#cert_chain) settings.  If you are using self-signed certificates, then the [`CA_ROOT`](/settings/#ca_root) file should also be provided.


Suggested instructions for obtaining certificates on a linux environment are provided below.

```
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
# provide domain to prompt below WITHOUT https prefix, eg your.domain.com
sudo certbot certonly --standalone
# check for certificates
sudo ls /etc/letsencrypt/live/your.domain.com
```

For debugging certificate issues, the following link is useful:  [https://certbot.eff.org/instructions?ws=other&os=ubuntufocal](https://certbot.eff.org/instructions?ws=other&os=ubuntufocal)
