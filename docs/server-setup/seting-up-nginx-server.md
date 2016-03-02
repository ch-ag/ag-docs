# Seting Up Nginx Server

When using the Nginx web server, server blocks (similar to the virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain off of a single server.


> __Prerequisites__
> We're going to be using a non-root user with sudo privileges throughout this tutorial.

## Installing Nginx

```bash
$ sudo apt-get update
$ sudo apt-get install nginx
```

## Set Up New Document Root Directories

By default, Nginx on Ubuntu 14.04 has one server block enabled by default. It is configured to serve documents out of a directory at:

```
/usr/share/nginx/html
```

We won't use the default since it is easier to work with things in the /var/www directory. Ubuntu's Nginx package does not use /var/www as its document root by default due to a Debian policy about packages utilizing /var/www.

Since we are users and not package maintainers, we can tell Nginx that this is where we want our document roots to be. Specifically, we want a directory for each of our sites within the /var/www directory and we will have a directory under these called html to hold our actual files.

First, we need to create the necessary directories. We can do this with the following command. The -p flag tells mkdir to create any necessary parent directories along the way:

```bash
$ sudo mkdir -p /var/www/example.com/html
$ sudo mkdir -p /var/www/test.com/html
```

```bash
$ sudo chown -R $USER:$USER /var/www/example.com/html
$ sudo chown -R $USER:$USER /var/www/test.com/html
```

```bash
$ sudo chmod -R 755 /var/www
```

## Create Sample Pages for Each Site

Create an index.html file in your first domain:

```bash
$ nano /var/www/example.com/html/index.html
```

```html
<html>
    <head>
        <title>Welcome to Example.com!</title>
    </head>
    <body>
        <h1>Success!  The example.com server block is working!</h1>
    </body>
</html>
```


Save and close the file when you are finished.

Since the file for our second site is basically going to be the same, we can copy it over to our second document root like this:

```bash
$ cp /var/www/example.com/html/index.html /var/www/test.com/html/
```
