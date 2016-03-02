
## Configure Users

When you first create a new Ubuntu 14.04 server, there are a few configuration steps that you should take early on as part of the basic setup. This will increase the security and usability of your server and will give you a solid foundation for subsequent actions.

### Root Login

log into your server, using server's public IP address and the SSH private key(s) or the password for the "root" user's account.

```bash
local$ ssh root@<SERVER_IP_ADDRESS>
```

Complete the login process by accepting the warning about host authenticity, if it appears, then providing your root authentication (password or private key). If it is your first time logging into the server, with a password, you will also be prompted to change the root password.

### Create a New User

Once you are logged in as root, you can add the new user account that we will use to log in from now on.

This example creates a new user called "agadmin":

```bash
root$ adduser agadmin
```

You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit "ENTER" in any field you wish to skip.

### Root Privileges

Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as "super user" or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word sudo before each command.

To add these privileges to our new user, we need to add the new user to the "sudo" group. By default, on Ubuntu 14.04, users who belong to the "sudo" group are allowed to use the sudo command.

As root, run this command to add your new user to the sudo group (substitute `agadmin` word with your new user):

```bash
root$ gpasswd -a agadmin sudo
```

Now your user can run commands with super user privileges! For more information about how this works, check out this [sudoers tutorial](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos).

### Add Public Key Authentication (Recommended)

The next step in securing your server is to set up public key authentication for your new user. Setting this up will increase the security of your server by requiring a private SSH key to log in.

#### Generate a Key Pair

If you do not already have an SSH key pair, which consists of a public and private key, you need to generate one. If you already have a key that you want to use, skip to the Copy the Public Key step.

To generate a new key pair, enter the following command at the terminal of your local machine (ie. your computer):

```bash
local$ ssh-keygen
```

Assuming your local user is called "localuser", you will see output that looks like the following:

```bash
# ssh-keygen output
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/localuser/.ssh/id_rsa):
Hit return to accept this file name and path (or enter a new name).
```

Next, you will be prompted for a passphrase to secure the key with. You may either enter a passphrase or leave the passphrase blank.

!!! note
    If you leave the passphrase blank, you will be able to use the private key for authentication without entering a passphrase. If you enter a passphrase, you will need both the private key and the passphrase to log in. Securing your keys with passphrases is more secure, but both methods have their uses and are more secure than basic password authentication.

This generates a private key, `id_rsa`, and a public key, `id_rsa.pub`, in the `$HOME/.ssh/` directory of the localuser's home directory. Remember that the private key should not be shared with anyone who should not have access to your servers!

#### Copy the Public Key

After generating an SSH key pair, you will want to copy your public key to your new server. We will cover two easy ways to do this.

###### Option 1: Use ssh-copy-id

If your local machine has the ssh-copy-id script installed, you can use it to install your public key to any user that you have login credentials for.

Run the ssh-copy-id script by specifying the user and IP address of the server that you want to install the key on, like this:

```bash
local$ ssh-copy-id -i /path/to/id_rsa agadmin@<SERVER_IP_ADDRESS>
```
After providing your password at the prompt, your public key will be added to the remote user's .ssh/authorized_keys file. The corresponding private key can now be used to log into the server.

###### Option 2: Manually Install the Key

Assuming you generated an SSH key pair using the previous step, use the following command at the terminal of your local machine to print your public key (id_rsa.pub):

```bash
local$ pbcopy ~/.ssh/id_rsa.pub
```
This should copy your public SSH key to your systems clipboard.

or

```bash
local$ cat ~/.ssh/id_rsa.pub
```
This should print your public SSH key, which should look something like the following:

```bash
# id_rsa.pub contents
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf localuser@machine.local
```
Select the public key, and copy it to your clipboard.

!!! danger-inline "don't copy any extra character or white space"


####### Add Public Key to New Remote User

To enable the use of SSH key to authenticate as the new remote user, you must add the public key to a special file in the user's home directory.

On the server, as the root user, enter the following command to switch to the new user (substitute your own user name):

```bash
root$ su - agadmin
```

Now you will be in your new user's home directory.

Create a new directory called `.ssh` and restrict its permissions with the following commands:

```bash
agdmin$ mkdir .ssh
agdmin$ chmod 700 .ssh
```

Now open a file in `.ssh/` called `authorized_keys` with a text editor. We will use nano to edit the file:

```bash
agdmin$  nano .ssh/authorized_keys
```

Now insert your public key (which should be in your clipboard) by pasting it into the editor.

Hit <kbd>CTR</kbd>+<kbd>X</kbd> to exit the file, then <kbd>Y</kbd> to save the changes that you made, then <kbd>ENTER</kbd> to confirm the file name.

Now restrict the permissions of the authorized_keys file with this command:

```bash
agdmin$ chmod 600 .ssh/authorized_keys
```

Type this command once to return to the root user:

```bash
agdmin$ exit
```

Now you may SSH login as your new user, using the private key as authentication.


### Configure SSH Daemon

Now that we have our new account, we can secure our server a little bit by modifying its SSH daemon configuration (the program that allows us to log in remotely) to disallow remote SSH access to the root account.

Begin by opening the configuration file with your text editor as root:

```bash
nano /etc/ssh/sshd_config
```

Next, we need to find the line that looks like this:

```bash
#/etc/ssh/sshd_config (before)
PermitRootLogin yes
```
Here, we have the option to disable root login through SSH. This is generally a more secure setting since we can now access our server through our normal user account and escalate privileges when necessary.

Modify this line to "no" like this to disable root login:

```bash
#/etc/ssh/sshd_config (after)
PermitRootLogin no
```

Disabling remote root login is highly recommended on every server!

When you are finished making your changes, save and close the file using the method we went over earlier  (<kbd>CTRL</kbd>+<kbd>X</kbd>, then <kbd>Y</kbd>, then <kbd>ENTER</kbd>).

### Reload SSH
Now that we have made our change, we need to restart the SSH service so that it will use our new configuration.

Type this to restart SSH:

```bash
agadmin$ service ssh restart
```

Now, before we log out of the server, we should test our new configuration. We do not want to disconnect until we can confirm that new connections can be established successfully.

Open a new terminal window on your local machine. In the new window, we need to begin a new connection to our server. This time, instead of using the root account, we want to use the new account that we created.

For the server that we showed you how to configure above, you would connect using this command. Substitute your own user name and server IP address where appropriate:

```
local$ ssh agadmin@<SERVER_IP_ADDRESS>
```

!!! note
    If you are using PuTTY to connect to your servers, be sure to update the session's port number to match your server's current configuration.

You will be prompted for the new user's password that you configured. After that, you will be logged in as your new user.

Remember, if you need to run a command with root privileges, type "sudo" before it like this:

```bash
agadmin$ sudo <command_to_run>
```

If all is well, you can exit your sessions by typing:

```bash
agadmin$ exit  
```

## Initial System Configuration

### Configuring a Basic Firewall

The SSH daemon runs on port 22 by default and ufw can implement a rule by name if the default has not been changed. So if you have not modified SSH port, you can enable the exception by typing:

```bash
$ sudo ufw allow ssh
```

If you have modified the port that the SSH daemon is listening on, you will have to allow it by specifying the actual port number, along with the TCP protocol:

```bash
$ sudo ufw allow 4444/tcp
```

This is the bare minimum firewall configuration. It will only allow traffic on your SSH port and all other services will be inaccessible. If you plan on running additional services, you will need to open the firewall at each port required.

If you plan on running a conventional HTTP web server, you will need to allow access to port 80:

```bash
$ sudo ufw allow 80/tcp
```

If you plan to run a web server with SSL/TLS enabled, you should allow traffic to that port as well:

```bash
$ sudo ufw allow 443/tcp
```

If you need SMTP email enabled, port 25 will need to be opened:

```bash
$ sudo ufw allow 25/tcp
```

After you've finished adding the exceptions, you can review your selections by typing:

```bash
$ sudo ufw show added
```

If everything looks good, you can enable the firewall by typing:

```bash
$ sudo ufw enable
```

You will be asked to confirm your selection, so type "y" if you wish to continue. This will apply the exceptions you made, block all other traffic, and configure your firewall to start automatically at boot.

Remember that you will have to explicitly open the ports for any additional services that you may configure later. For more in-depth information, check out our article on configuring the ufw firewall.

### Configure Timezones and Network Time Protocol Synchronization

The next step is to set the localization settings for your server and configure the Network Time Protocol (NTP) synchronization.

The first step will ensure that your server is operating under the correct time zone. The second step will configure your system to synchronize its system clock to the standard time maintained by a global network of NTP servers. This will help prevent some inconsistent behavior that can arise from out-of-sync clocks.

#### Configure Timezones

Our first step is to set our server's timezone. This is a very simple procedure that can be accomplished by reconfiguring the tzdata package:

```bash
$ sudo dpkg-reconfigure tzdata
```

You will be presented with a menu system that allows you to select the geographic region of your server

#### Configure NTP Synchronization

Now that you have your timezone set, we should configure NTP. This will allow your computer to stay in sync with other servers, leading to more predictability in operations that rely on having the correct time.

For NTP synchronization, we will use a service called ntp, which we can install from Ubuntu's default repositories:

```bash
$ sudo apt-get update
$ sudo apt-get install ntp
```

This is all that you have to do to set up NTP synchronization on Ubuntu. The daemon will start automatically each boot and will continuously adjust the system time to be in-line with the global NTP servers throughout the day.

Click here if you wish to learn more about [NTP servers](https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-12-04).

### Create a Swap File

!!! danger "Don't Create Swap on SSD"
    Swap file sould not be created on SSD drives, note that digitalocean uese SSD drives.


### Take a Snapshot of your Current Configuration

If you are happy with your configuration and wish to use this as a base for future installations, you can take a snapshot of your server through the DigitalOcean control panel.

To do so, shutdown your server from the command line by typing:

```bash
sudo poweroff
```
Now, in the DigitalOcean control panel, you can take a snapshot by visiting the "Snapshots" tab of your server:

![](https://assets.digitalocean.com/articles/1404_optional_recommended/snapshots.png)

After taking your snapshot, you will be able to use that image as a base for future installations by selecting the snapshot from the "My Snapshots" tab for images during the creation process:

![](https://assets.digitalocean.com/articles/1404_optional_recommended/use_snapshot.png)


## Recommended Packages and Software

### GIT

```bash
$ sudo apt-get install -y git
```

### Build Tools (gcc, make)

```bash
$ sudo apt-get install -y build-essential
```
