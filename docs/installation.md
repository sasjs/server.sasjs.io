# Installation

The latest version can be downloaded from the Releases page:  [https://github.com/sasjs/server/releases](https://github.com/sasjs/server/releases)

It can also be installed in just two lines of code (on linux):

```bash
curl -L https://github.com/sasjs/server/releases/latest/download/linux.zip > linux.zip
unzip linux.zip
```


## Configuration

After installation, there is only one mandatory input:

* `SAS_PATH` -> the full path to the SAS executable in your environment

Additional [settings](/settings) can be configured as environment variables.


## SSL Certificates

If you would like to run your server on https, then you will need to provide certificates.  Sample instructions for obtaining certificates on a linux environment are provided below.

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
