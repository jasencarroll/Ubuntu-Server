# Ubuntu-Server

This will work if no matter how you plan to use Ubuntu-Server. It could be used to quickly secure a Digital Ocean instance for example. It probably can't be in WSL but I'll test that sometime eventually. It can definitely be from your desktop/laptop Ubuntu instance. For me I want to not be strapped to a desk or powercord all the time so I am making my custom built dev PC an Ubuntu Server so I can SSH in using any device I want.

Script files can easily be generated from this README to automate the provisioning process. 

## Getting Started

### Using Desktop Version

If you are not running the server version of Ubuntu the first step is:

```bash
sudo apt install ubuntu-server ssh 
```

### Using Server Version

```bash
ssh root@your_server_ip
```

We'll all need that eventually.

## Firewall

In order to follow along, moving forward add sudo to everything if you're not logged in as root, or log in as root locally `sudo -i` to avoid typing sudo each time.

```bash
ufw app list
```

One of the selections should be OpenSSH so `ufw allow OpenSSH`

At this point the Firewall is blocking everything but Port 22 - this is only half way though.

## SSH Config

At this point use your favorite text editor and open the path below:
```bash
sudo nano /etc/ssh/sshd_config
```

There may be existing lines with some of this informtaion, but everything in the file is commented out so you can feel free to copy and paste these settings.

```bash
Port 22
LoginGraceTime 2m
PermitRootLogin no
StrictModes yes
MaxAuthTries 6
MaxSessions 10
PasswordAuthentication yes
PermitEmptyPasswords no
```

Then we'll want to restart

```bash
sudo systemctl restart sshd
```

Let's test!

Get your ip address using `ip a` . Note you'll want to use the one labeled something like `IPv4 address for en5s0`. If you pick the wrong one, like docker or vbox you'll get connection denied on port 22. So that's cool, I could theoretically ssh directly into my VM if I configure the VM to allow such. For now it's a Vagrant. 

```bash
ssh {username}@{XXX.XXX.X.XX}
``` 

And you should be prompted for a password, if not go back and try again. 

## Security

### Back to Firewall

```bash
sudo ufw block OpenSSH
sudo ufw allow {port} # between 1024 and 32,767
```

### Back to SSH Config

```bash
Port {port}
LoginGraceTime 2m
PermitRootLogin no
StrictModes yes
MaxAuthTries 6
MaxSessions 10
PasswordAuthentication yes
PermitEmptyPasswords no
```

Save and restart. 

```bash
sudo systemctl restart sshd
```

Now you should be able to connect on your preferred port. Though it's pretty insecure to leave password login on, even if you do have a string password that would take today's computers trillions of years to crack.

### RSA Based auth

Do this from your client.

```bash
ssh-keygen -t rsa
ssh-copy-id -p XXXX jasen@localIP
ssh -p XXXX jasen@localIP
```

Do this to your server, in your `sshd_config`

```bash
PasswordAuthentication no
```

followed by `sudo systemctl restart sshd`

Client should continue to default to RSA. Now it'll take trillions of years to crack the string passkey but they will physically have to have my client as well to match the RSA authentication.

## Port Forwarding

This is all fine and dandy if you're at home on your local network. In order to access your server through your router you need to forward that port from your router to your server. This varies on routers.

You can easily determine your outside IP if you're not running a VPN.

With that outside IP, tell the router the port you want, and the local IP of the server, and the router will pass it through.

You can also configure your server to wake up on LAN which can do anything from booting your system from off. If you have your hard drive encrypted, I'm not sure there is a way to make that work - haven't tried.

With encryption I think the computer will sit there waiting for the password. Just never booting. That would prevent any unwanted individuals from booting your system while you're not home. With the system running and hard drive unlocked the wake on LAN could wake up the server from sleeping. 

For safety purposes though it's probably best to be running a VPN on your router and also I'm going to try SSH into my VM inside of the server. This would prevent any unwanted individuals (including myself) to nuke hardware or native environments.