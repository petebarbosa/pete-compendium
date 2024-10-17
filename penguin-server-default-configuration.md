<a id="readme-top"></a>

# My server default configuration

The intention of this guide is to set up a brand new installation of a Debian based servers. With this guide we'll be able:

- Connect locally via SSH
- Connect remotely via SSH
- Connect using a custom No-IP address
- Set security measures for all the steps above

P.S.: In this guide I'm considering you're using two machines, an `ubuntu_server` and an `ubuntu_client`.

## Step 1: Install SSH on the `ubuntu_server`

1. Open a terminal.

2. Update the package list and install the OpenSSH server:

    ```bash
    sudo apt update
    sudo apt install openssh-server
    ```

3. Start the SSH service and enable it to start on boot:

    ```bash
    sudo systemctl start ssh
    sudo systemctl enable ssh
    ```

4. Check the status of the SSH service to ensure it's running:

    ```bash
    sudo systemctl status ssh
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Step 2: Add the Public Key to the `ubuntu_server`

To secure your SSH server and ensure that only clients you explicitly approve can connect, you can implement **key-based authentication** and disable password logins. Here's how to do it:

1. On your `ubuntu_client`, copy the public key (`id_rsa.pub`) to your `ubuntu_server` using the following command:

    ```bash
    ssh-copy-id username@ubuntu_server_ip
    ```

   - Replace `username` with your `ubuntu_server` username and `ubuntu_server_ip` with the actual IP address of the server.
   - This will append your public key to the `~/.ssh/authorized_keys` file on the server.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Step 3: Disable Password Authentication on the `ubuntu_server`

1. On the `ubuntu_server`, open the SSH configuration file for editing:

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

2. Find and update the following lines to disable password authentication and root login:

    ```bash
    PermitRootLogin no
    PasswordAuthentication no
    ```

3. Restart the SSH service for the changes to take effect:

    ```bash
    sudo systemctl restart ssh
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Step 3: Install fail2ban on `ubuntu_server`

Install and configure Fail2Ban to protect against brute force attacks.

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Step 4: (Optional) Install No-IP DUC on `ubuntu_server`

This is an optional step. You do not need it, but since I'm going to use my local server to host some personal projects and may need to access my pc remotely, I like to do.

1. Download and install:

```bash
wget --content-disposition https://www.noip.com/download/linux/latest
tar xf noip-duc_3.3.0.tar.gz
cd /home/$USER/noip-duc_3.3.0/binaries && sudo apt install ./noip-duc_3.3.0_amd64.deb
```

2. To login and send updates using DDNS key, enter the following:

```bash
noip-duc -g all.ddnskey.com --username <DDNS Key Username> --password <DDNS Key Password>
```
### Step 4.1: (Optional) Set No-IP to auto-start

1. Get `noip-duc` install location:

```bash
which noip-duc
```

2. Create a `noip-duc.conf` file:

```bash
sudo nano /etc/noip-duc.conf
```

3. Inside of it add your No-IP username and password:

```bash
USERNAME="YOUR_USERNAME"
PASSWORD="YOUR_PASSWORD"
```

4. Secure the file (I have NO IDEA if it's enough security):

```bash
sudo chmod 600 /etc/noip-duc.conf
```

5. Create a systemd service file:

```bash
sudo vim /etc/systemd/system/noip-duc.service
```

6. Add the following content:

```ini
[Unit]
Description=No-IP DUC (Dynamic DNS Update Client)
After=network.target

[Service]
Type=simple
EnvironmentFile=/etc/noip-duc.conf
ExecStart=<noip-duc-address-from-one> --username $USERNAME --password $PASSWORD
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

7. Reload systemd daemon and restar the service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart noip-duc
sudo systemctl status noip-duc
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

