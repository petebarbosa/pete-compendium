<a id="readme-top"></a>

<div align="center">

# My server default configuration

This guide's intention is to set up a brand new installation of a Debian based servers. With this guide we'll be able:

</div>

- Connect locally via SSH
- Connect remotely via SSH
- Connect using a custom No-IP address
- Set security measures for all the steps above

P.S.: In this guide I'm considering you're using two machines, an `ubuntu_server` and an `ubuntu_client`.

P.P.S: If you're using a GUI installation of Ubuntu (the non-server installation), consider using [omakub](https://omakub.org/). It's going to install everything your environment. You'll just need to make a few tweaks.

## Step 1: Install SSH on the `ubuntu_server`

1. Open a terminal.

2. Update the package list and install the OpenSSH server:

    ```bash
    sudo apt update
    sudo apt upgrade
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

1. First lets get the ip address from our `ubuntu_server` with:

    ```sh
    ifconfig
    ```

2. On your `ubuntu_client`, copy the public key (`id_rsa.pub`) to your `ubuntu_server` using the following command:

    ```bash
    ssh-copy-id username@ubuntu_server_ip
    ```

   - Replace `username` with your `ubuntu_server` username and `ubuntu_server_ip` with the actual IP address of the server.
   - This will append your public key to the `~/.ssh/authorized_keys` file on the server.

3. Now log into the server with:

    ```sh
    ssh username@ubuntu_server_ip
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>


## Step 3: Disable Password Authentication on the `ubuntu_server`

1. (Optional) I like to use vim, so I'll install it. You can skip this and use `nano`

    ```sh
    sudo apt install vim
    ```

2. On the `ubuntu_server`, open the SSH configuration file for editing:

    ```bash
    sudo vim /etc/ssh/sshd_config
    ```

3. Find and update the following lines to disable password authentication and root login:

    ```bash
    PermitRootLogin no
    PasswordAuthentication no
    ```

4. Restart the SSH service for the changes to take effect:

    ```bash
    sudo systemctl restart ssh
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>


## Step 4: Install fail2ban on `ubuntu_server`

Install and configure Fail2Ban to protect against brute force attacks.

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## (Optional) Step 5: Install docker

Since I like using docker, it's one of those things that when you get it, you just can't live without it, we'll instal it. Here is the [documentation](https://docs.docker.com/desktop/install/linux/ubuntu/).

Docker engine comes bundled with Docker Desktop (and let's be sincere here, it's easier to watch with it).

1. Set up Docker's `apt` repository:

    ```sh
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    
    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```

2. Install Docker packages:

    ```sh
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

3. Verify installation:

    ```sh
    sudo docker run hello-world
    ```

