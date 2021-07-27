# 2 Tier App Deployment

<p align:center>
<img src=aws_diagram.PNG>
</p>
####
## EC2 APP instance

### Adding app folder to instance

```bash
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
```

#### 

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

### Make sure all IPs are correct when setting up Security Groups in EC2

<p align:center>
<img src=app_sg.png>
</p>
<p align:center>
<img src=db_sg.png>
</p>