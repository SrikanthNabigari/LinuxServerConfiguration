## Linux Server Configuration Udacity FSND Project 7

## AWS Server Deployed on Below Addresses

- [35.160.72.118](http://35.160.72.118/)
- [ec2-35-160-72-118.us-west-2.compute.amazonaws.com](http://ec2-35-160-72-118.us-west-2.compute.amazonaws.com/)

## Configuration

### 1 - SSH access to the instance

1. Download Private Key [here](blob:https://www.udacity.com/95349ac3-5488-4aa4-aedf-21febdaac080)
2. Move the private key file into the folder ~/.ssh

    ```
    mv ~/Downloads/udacity_key.rsa ~/.ssh/
    ```
3. Secure the key with

    ```
    chmod 600 ~/.ssh/udacity_key.rsa
    ```

4. Log into root

    ```
    ssh -i ~/.ssh/udacity_key.rsa root@35.160.72.118
    ```

### 2 - Add User Grader and Provide Permissions and SSH login

1. Create User Grader

    ```
    sudo adduser Grader
    ```

    provide the neccessities of the new User

2. Copy the root authorized key into grader

    ```
    sudo cp ~/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
    ```
3. Change the Permissions to the .ssh folder and authorized_keys

```
sudo chown grader /home/grader/.ssh
```

```
sudo chgrp grader /home/grader/.ssh
```

```
sudo chown grader /home/grader/.ssh/authorized_keys
```

```
sudo chgrp grader /home/grader/.ssh/authorized_keys
```

```
sudo chmod 700 /home/grader/.ssh
```

```
sudo chmod 644 /home/grader/.ssh/authorized_keys
```

### 3 - Change the SSH port from 22 to 2200 and AllowsUsers to login
    
```
sudo nano /etc/ssh/sshd_config
```

Change the port `22 to 2200` and add `AllowsUsers grader` in the sshd config file

Now restart the ssh

```
sudo service ssh restart
```

### 4 - Configure Uncomplicated Firewall to only allow incoming  connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
sudo ufw default deny incoming
```

```
sudo ufw default allow outgoing
```

```
sudo ufw allow 2200
```

```
sudo ufw allow 80
```

```
sudo ufw allow 123
```

```
sudo ufw enable
```

### 5 - Login as Grader User

exit from root login and

```
sudo ssh -i ~/.ssh/udacity_key.rsa grader@35.160.72.118
```

### 6 -     