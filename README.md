# AUTOMATING LOADBALANCER WITH SHELL SCRIPTING<br />
<br />

### Automate the deployment of webservers 

### Deploying and configuring webservers

### Use of Nginx as a load balancer
<br />

## Step 1
<br />
I created and started three Ec2 instance

<br />

![created 3 Ec2 instances](<img/1 aws instance.png>)

i opened port `8000` in inbound rules to allow incoming traffic 

![opened port 8000](<img/2 set port 8000.png>)

next i connected to my ec2 instance using SSH

![ssh connected](<img/3 connected to my ec2 instance.png>)

next i created a script file using the command below 

```
sudo vi install.sh
```

![created the script file](<img/4 create a script file.png>)


next i typed the script code below
<br />

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```

![edit file](<img/5 edit script of first ec2 instance.png>)

next i changed permission on the file to make it executable using the command below

```
sudo chmod +x install.sh
```

![change of ownership](<img/7 change ownership of script file.png>)

Next i run the script file using the code below

```
./install.sh PUBLIC_IP
```


`./install.sh 51.20.87.229`

> "51.20.87.229" is the `PUBLIC_IP` of the Ec2 instance

![run the script file](<img/8 run script file.png>)

##### NOTE I REPEATED THE ABOVE STEPS FOR THE SECOND EC2 INSTANCE
 <br />

## STEP 2
 <br />

### DEPLOYING NGINX AS A LOAD BALANCER USING SHELL SCRIPT

First i ssh into my ec2 instance i will use a load balancer

next i created a script file using the command below

```
sudo vi nginx.sh
```

![nginx script](<img/9 create and open nginx script.png>)

next i typed the script comamnd below into the script file `nginx.sh`

```

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx
```
 <br />
  <br />

![nginx script code](<img/10 configure and edit nginx script.png>)

 next i save and closed the 'vi' file

next i changed the file permission to make it executable using the command below
```
sudo chmod +x nginx.sh
 ```

![change nginx file permission](<img/11 change file permission on the nginx script file.png>)

next i run the command below to execute the script file

```
./nginx.sh 13.60.6.232 51.20.87.229:8000 13.60.16.86:8000
```

`13.60.6.232` is the public address of the nginx loadbalancer ec2 instance

`51.20.87.229` is the public address of the first ec2 instance

`13.60.16.86` is the public address of the second ec2 instance

![load 1 output](<img/12 load 1 output.png>)

![load 2 output](<img/13 load 2 output.png>)

![nginx load balancer](<img/14 loadbalancing output.png>)


 <br />
  <br />

### CHALLENGES

The major challenge i encountered with this project is with getting the scripting correctly eventually i was able to identify where i was wrong and make the necessary correction.