# Set Up a Node.js Application for Production

Node.js is an open source Javascript runtime environment for easily building server-side and networking applications. The platform runs on Linux, OS X, FreeBSD, and Windows, and its applications are written in JavaScript. Node.js applications can be run at the command line but we will teach you how to run them as a service, so they will automatically restart on reboot or failure, so you can use them in a production environment.

In this tutorial, we will cover setting up a production-ready Node.js environment that is composed of two Ubuntu 14.04 servers; one server will run Node.js applications managed by PM2, while the other will provide users with access to the application through an Nginx reverse proxy to the application server.

## Install Node.js

We will install the latest LTS release of Node.js, on the app server.

let's update the apt-get package lists with this command:

```bash
$ sudo apt-get update
```

Go to the Node.js Downloads page and find the Linux Binaries (.tar.gz) download link. Right-click it, and copy its link address to your clipboard. At the time of this writing, the latest LTS release is 4.2.3. If you prefer to install the latest stable release of Node.js, go to the appropriate page and copy that link.

Change to your home directory and download the Node.js source with wget. Paste the download link in place of the highlighted part:

```bash
$ cd ~
$ wget https://nodejs.org/dist/v4.2.3/node-v4.2.3-linux-x64.tar.gz
```

Now extract the tar archive you just downloaded into the node directory with these command:

```bash
mkdir node
tar xvf node-v*.tar.gz --strip-components=1 -C ./node
```

If you want to delete the Node.js archive that you downloaded, since we no longer need it, change to your home directory and use this rm command:

```bash
$ cd ~
$ rm -rf node-v*
```

Next, we'll configure the global prefix of npm, where npm will create symbolic links to installed Node packages, to somewhere that it's in your default path. We'll set it to /usr/local with this command:

```bash
$ mkdir node/etc
$ echo 'prefix=/usr/local' > node/etc/npmrc
```

Now we're ready to move the node and npm binaries our installation location. We'll move it into /opt/node with this command:

```bash
sudo mv node /opt/
```

At this point, you may want to make root the owner of the files:

```bash
$ sudo chown -R root: /opt/node
```

Lastly, let's create symbolic links of the node and npm binaries in your default path. We'll put the links in /usr/local/bin with these commands:

```bash
$ sudo ln -s /opt/node/bin/node /usr/local/bin/node
$ sudo ln -s /opt/node/bin/npm /usr/local/bin/npm
```

Verify that Node is installed by checking its version with this command:

```bash
$ node -v
```

The Node.js runtime is now installed, and ready to run an application! Let's write a Node.js application.


## Install PM2

ow we will install PM2, which is a process manager for Node.js applications. PM2 provides an easy way to manage and daemonize applications (run them as a service).

We will use Node Packaged Modules (NPM), which is basically a package manager for Node modules that installs with Node.js, to install PM2 on our app server. Use this command to install PM2:

```bash
$ sudo npm install pm2 -g
```

## Manage Application with PM2

PM2 is simple and easy to use. We will cover a few basic uses of PM2.

### Start Application

The first thing you will want to do is use the pm2 start command to run your application, in the background:

```bash
$ pm2 start app.js
```

This also adds your application to PM2's process list, which is outputted every time you start an application:

As you can see, PM2 automatically assigns an App name (based on the filename, without the .js extension) and a PM2 id. PM2 also maintains other information, such as the PID of the process, its current status, and memory usage.

Applications that are running under PM2 will be restarted automatically if the application crashes or is killed, but an additional step needs to be taken to get the application to launch on system startup (boot or reboot). Luckily, PM2 provides an easy way to do this, the startup subcommand.

The startup subcommand generates and configures a startup script to launch PM2 and its managed processes on server boots. You must also specify the platform you are running on, which is ubuntu, in our case:

```bash
$ pm2 startup ubuntu
```

The last line of the resulting output will include a command (that must be run with superuser privileges) that you must run:


> Output:
[PM2] You have to run this command as root
[PM2] Execute the following command :
[PM2] <mark>sudo su -c "env PATH=$PATH:/opt/node/bin pm2 startup ubuntu -u agadmin --hp /home/agadmin"</mark>


Run the command that was generated (similar to the highlighted output above) to set PM2 up to start on boot (use the command from your own output):

```
$ sudo su -c "env PATH=$PATH:/opt/node/bin pm2 startup ubuntu -u agadmin --hp /home/agadmin"
```


### Other PM2 Usage (Optional)

PM2 provides many subcommands that allow you to manage or look up information about your applications. Note that running pm2 without any arguments will display a help page, including example usage, that covers PM2 usage in more detail than this section of the tutorial.

Stop an application with this command (specify the PM2 App name or id):

```bash
$ pm2 stop example
```

Restart an application with this command (specify the PM2 App name or id):

```bash
$ pm2 restart example
```

The list of applications currently managed by PM2 can also be looked up with the list subcommand:

```bash
$ pm2 list
```

More information about a specific application can be found by using the info subcommand (specify the PM2 App name or id)::

```bash
$ pm2 info example
```
The PM2 process monitor can be pulled up with the monit subcommand. This displays the application status, CPU, and memory usage:

```bash
$ pm2 monit
```

Now that your Node.js application is running, and managed by PM2, let's set up the reverse proxy.

## Set Up Reverse Proxy Server

Now that your application is running, and listening on a private IP address, you need to set up a way for your users to access it. We will set up an Nginx web server as a reverse proxy for this purpose. This tutorial will set up an Nginx server from scratch. If you already have an Nginx server setup, you can just copy the location block into the server block of your choice (make sure the location does not conflict with any of your web server's existing content).

```bash
# /etc/nginx/sites-available/<node-app-site>
server {
    listen 80;

    server_name example.com;

    location / {
        proxy_pass http://<APP_PRIVATE_IP_ADDRESS>:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```


his configures the web server to respond to requests at its root. Assuming our server is available at example.com, accessing http://example.com/ via a web browser would send the request to the application server's private IP address on port 8080, which would be received and replied to by the Node.js application.

You can add additional location blocks to the same server block to provide access to other applications on the same web server. For example, if you were also running another Node.js application on the app server on port 8081, you could add this location block to allow access to it via http://example.com/app2:

```bash
# Nginx Configuration — Additional Locations

    location /app2 {
        proxy_pass http://APP_PRIVATE_IP_ADDRESS:8081;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```

Once you are done adding the location blocks for your applications, save and exit.

On the web server, restart Nginx:

```bash
$ sudo service nginx restart
```

Assuming that your Node.js application is running, and your application and Nginx configurations are correct, you should be able to access your application via the reverse proxy of the web server. Try it out by accessing your web server's URL (its public IP address or domain name).
