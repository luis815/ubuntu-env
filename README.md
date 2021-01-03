# ubuntu-env
DigitalOcean NodeJS + VueJS remote development environment

## Prep
* Follow DigitalOcean instructions for creating ssh keys
* Create DigitalOcean droplet
* Setup domain with droplet

## Guide
Make sure droplet is up-to-date
```
sudo apt-get update && sudo apt-get upgrade
```

### NGINX [original instructions](https://ubuntu.com/tutorials/install-and-configure-nginx#1-overview)
Install NGINX
```
sudo apt install nginx
```

Edit NGINX default file and remove all comments
```
nano /etc/nginx/sites-available/default
```

### Reverse proxy for vue dev server [original instructions](https://gist.github.com/bradtraversy/cd90d1ed3c462fe3bddd11bf8953a896)
Replace default location block with
```
	location ~ ^/(app|sockjs-node) {
		proxy_pass http://localhost:8080;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}

	location / {
		proxy_pass http://localhost:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
```

Restart NGINX
```
sudo service nginx restart
```

### NodeJS [original instructions](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)
Install NodeJS
```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Create sudo user
Create new user
```
adduser whateveryouwant
```

Add user to sudo
```
usermod -aG sudo userfrombefore
```

Switch user
DO NOT SKIP THIS
```
su userfrombefore
```

### Set up ssh key with new user [original instructions](https://stackoverflow.com/questions/6377009/adding-a-public-key-to-ssh-authorized-keys-does-not-log-me-in-automatically)
From new user home directory directory
```
cd ~/

mkdir .ssh
chmod 700 .ssh/

touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

Edit authorized_keys and paste the content
of your LOCAL MACHINE's id_rsa.pub file
similar to DigitalOcean ssh key setup instructions
```
nano .ssh/authorized_keys
```

### Certbot [original instructions](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx)
Prep for certbot
```
sudo snap install core; sudo snap refresh core
```

Install certbot
```
sudo snap install --classic certbot
```

Prep command
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Run certbot
```
sudo certbot --nginx

sudo certbot renew --dry-run
```

### Firewall [original instructions](https://gist.github.com/bradtraversy/cd90d1ed3c462fe3bddd11bf8953a896)
Set up firewall for ssh, http and https
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

## Vue config
In your vue.config.js
This config does the following:
* publicPath sets where in the port :8080 the app will be served (matches NGINX config)
* The headers prevent caching in Safari browser
* disableHostCheck allows the app to be accessed from outside localhost
* public adjusts HRM server to be accessed from reverse proxy (not very knowledge about this, but it works)
* watchOptions I'm unsure if webpack ignores node modules by default
* chainWebpack These settings are for Safari browser too

```javascript
module.exports = {
	publicPath: "/app",
	devServer: {
		headers: {
			"Cache-Control": "no-store",
			Vary: "*",
		},
		disableHostCheck: true,
		public: "0.0.0.0",
	},
	configureWebpack: {
		watchOptions: {
			ignored: /node_modules/,
		},
	},
	chainWebpack(config) {
		config.plugins.delete("preload");
		config.plugins.delete("prefetch");
	},
};

```

## Now what?
* Your vue app can be accessed from
```
https://<yourdomain>/app
```
* Your api should listen on port 3000 to be accessed remotely
* You can work on your project using VSCode Remote - SSH extension (from microsoft)
