# 2 Tier App Deployment with AWS

## Setting up EC2 instances

1. Login with excel credentials
2. Set location to **EU-West Ireland**
3. Create new instance
4. Ubuntu server 16.04 64-bit(x86)
5. Choose default free type
6. Change `subnet` to **DevOpsStudent default 1a** and `auto-assign public ip` to **enable**
7. Leave storage as default
8. Add a name tag with value **eng89_filipe_xxx**
9. Configure `Security Group` naming convention **eng89_filipe_sg_xxx**
10. Launch, choose existing `key pair` - **eng89_devops** (which is the key stored in our .ssh dir in localhost)

## Connecting to instance (SSH)

1. Select instance and click `Connect`
2. Go to ssh
3. If your `key` doesn't have permissions, copy the `chmod` command
4. Copy and paste the ssh command into terminal inside the .ssh folder

</h3>

<p align:center>
<img src=aws_diagram.PNG>
</p>


## EC2 APP instance

### Adding app folder to instance

```sh
scp -i [.pem file path] -r [app folder on localhost] [ec2 instance name]:[desired dir]
scp -i eng89_devops.pem -r ~/Desktop/Work/Sparta/VMs/engi89_nginx/app/ubuntu@ec2-18-203-110-168.eu-west-1.compute.amazonaws.com:/home/ubuntu/
```

### Install dependencies
```bash
# Update
sudo apt-get update -y

# Upgrade packages
sudo apt-get upgrade -y

# Install nginx
sudo apt-get install nginx -y

# Install git
sudo apt-get install git -y

# Install nodejs
sudo apt-get install python-software-properties
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs -y

# Install pm2
sudo npm install pm2 -g
```

### Setup reverse proxy

We do reverse proxy so that the app works on the browser without the :3000

- sudo nano /etc/nginx/sites-available/default

```bash
server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://18.203.110.168:3000;  # This is the app public IP addr
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Setting an environment variable for db

Once the database is created we must set the env with the db public IP addr:
```
sudo echo "export DB_HOST=mongodb://34.242.255.250:27017/posts" >> ~/.bashrc
source ~/.bashrc
cd /app 
npm start
```

## EC2 DB instance

### Setup and connect mongodb
```bash
# Installing mongodb
wget -qO - https://www.mongodb.org/static/pgp/server-3.2.asc | sudo apt-key add -

echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update

sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

# Change mongodb conf file so that bindIp is 0.0.0.0
cd /etc
sudo rm -rf mongod.conf
sudo echo "
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
net:
  port: 27017
  bindIp: 0.0.0.0
" >> mongod.conf

# Restart and enable mongodb
sudo systemctl enable mongod
sudo systemctl restart mongod
```

## Notes

#### **SSH timeout error when trying to ssh into instance**

- Check your IP in AWS, it needs to match what you set up in SG

#### **Make sure all IPs are correct when setting up Security Groups in EC2**

<p align:center>
<img src=Inkedapp_sg_LI.jpg>
</p>
<p align:center>
<img src=Inkeddb_sg_LI.jpg>
</p>

#### **If the database is connected but /posts is empty:**

- `node ~/app/seeds seed.js`