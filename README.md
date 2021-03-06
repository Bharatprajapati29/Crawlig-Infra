# Crawlig-Infra

# What We Need

  AWS CLOUD-SERVER
  
  AWS Lambda as a crawling backend
  
  AWS S3 to store crawled Html data
  
  An AWS EC2 instance used as a master server that schedules the crawl task and hosts the mongodb that we use a queue
  
  MongoDB 
	

Install the distributed crawling infrastructure within the AWS cloud infrastructure.

     Repo : https://github.com/NikolaiT/Crawling-Infrastructure

    // Start a crawl task that will crawl the Html of the top 10.000 websites and store the cleaned Html documents on s3.For the top 10k websites, we use the scientific tranco list

# Setting up the infrastructure :

Launch Suitable AMI

configure AWS-Profile

setup Docker and installed dependacies for project

Deploying Master server

Deploying the crawler to AWS Lambda


# Steps:

Create Cluster In MongoDB

Create Database admin user and noteDown MongoDB-SRV

Configure Security Group in AWS

Create IAM-user With cli-access and attach policies s3,IAM,Cloudformation,Lambda,API-Gateway

Connect EC2

Configure AWS-CLI

// REF : Install Docker == https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

// REF : Installing nodejs, yarn and typescript on the server == https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/

     Excute Commands : 
     sudo apt install apt-transport-https ca-certificates curl software-properties-common
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu 
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
     sudo apt update
     sudo apt install docker-ce -y
     sudo systemctl status docker
     sudo usermod -aG docker ${USER}   
     su - ${USER}
     id -nG
     id
     docker help
     systemctl status docker.service 
     sudo usermod -aG docker ubuntu  // Add Manually userName
     id
     curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
     sudo apt-get install -y nodejs
     node --version
     npm --version
     sudo npm install -g typescript
     tsc --version
     sudo curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
     sudo chown -R $USER:$(id -gn $USER) /home/ubuntu/.config
     curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
     echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt update && sudo apt install yarn
     yarn --version
     

# Clone the project and install & compile


     git clone https://github.com/NikolaiT/Crawling-Infrastructure.git
     cd Crawling-Infrastructure/
     sudo npm install -g typescript
Now we are ready to install & compile the project locally:

     cd master/
     npm install
     
     //  switch to the library and install and compile it first!
     cd ../lib/
     tsc
     
     // go back to the master and compile it
     cd ../master/
     tsc

After that step, the crawling infrastructure should be compiled successfully.

Deploy the Master server

Configure and Update this File :	master/deploy/env/deploy.env  [[ change IP / path to .PEM file / and do what other changes required. ]]


fillup the Creads in : master/env/skeleton_production.env and update & modify the missing parameters.

rename the file from master/env/skeleton_production.env to master/env/production.env

create an empty master/env/development.env  // . It is required by docker swarm.
  
     Excute Commands:
     cd master
    ./deploy/deploy.sh deploy

The above command will compile the source code locally and deploy to the remote server and initialize a docker swarm there.

Now we have to create a default configuration on master server with the following commands:  
    
    node ctrl.js --action cfg --what create
  
You can test if deployment was successful by executing the following command:

    $ ./deploy/test.sh
    https://34.193.81.78:9001/
    {
	"scheduler_uptime": "",
	"scheduler_version": "v1.3",
	"api_version": "1.0.2",
	"machines_allocated": 0,
	"http_machines_allocated": 0,
	"num_total_items": 0,
	"num_tasks": 0

You will Get the OutPut like this.

Note : Make Sure Successfully Run this commands.


# Deploying the crawler to AWS Lambda


Switch to crawler Dir:

     sudo npm install -g serverless
     sudo npm install -g typescript 
     crawler/deploy_all.js  [[ change your region where you want to deploy your Lambda Function

After that we have to actually create S3 buckets on those regions, otherwise our crawled data could not be correctly stored. You can create AWS buckets programmatically with the script scripts/create_buckets.sh with the command:

     ./scripts/create_buckets.sh

Now that we have created those buckets, it's time to update the available regions on our master server. We change the directory to master/ and issue the following commands:

    cd master;

    export $(grep -v '^#' env/production.env | xargs -0);

    node ctrl.js --action cfg --what update_functions

Now the master server has the correct functions configured.

After those steps, it's finally time to upload our crawling code to AWS Lambda.

We can do this with the following commands:

change back to worker directory:

      cd ../crawler;
      export $(grep -v '^#' env/crawler.env | xargs -0);
     // copy file from Crawling-Infrastructure/crawler/env/[.env] to Crawling-Infrastructure/crawler/crawler.env
      node deploy_all.js worker

Configure the Master server : 

     cd master/
    ./launch_frontend_interface.sh






