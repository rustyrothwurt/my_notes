# DO NOT USE ME - VERY OLD
# Packer, Terraform, Docker and AWS
The below notes are for running apps in AWS (EC2 and ECS) using various methods. Links to useful sources below.

## Requirements
See below for install instructions.
* AWS account - the tests below use the free tier -- for the most part. check your usage before doing things
* Docker, Docker Compose
* Packer
* Terraform
* Command line tool


## Methods of getting apps going in AWS
### Local Files > Create EC2 Docker Machine > SSH into Machine > docker-compose up
0. (https://github.com/rustyrothwurt/blog)[Sample repo] 
1. Export your AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as local env variables, alternately, you can put them in the command below manually. Opt: export local session vars like $region, etc. to use in below command.
2. Test your local repo with `docker-compose up`
3. Create an EC2 Docker machine with one of the commands below, replacing aws-sandbox with whatever you want to call the machine and the secret/key with your own: 
* `docker-machine create --driver amazonec2 --amazonec2-open-port 8000 --amazonec2-region us-west-1 aws-sandbox`
* `docker-machine create --driver amazonec2 --amazonec2-access-key your_key_here --amazonec2-secret-key your_secret_here  aws-sandbox`
4. Verify in AWS EC2 Dashboard it is running
5. Now in your command line, enter `docker-machine env aws-sandbox` to see the eval command to run to get connection info.
6. Copy and paste `eval $(docker-machine env aws-sandbox)` into your command line tool. If you don't get a result, try entering `bash` to use the bash shell. Several lines of variables should be shown. Copy and paste those back into your command line tool.
7. Test SSH'ing into the machine with:

`docker-machine ssh aws-sandbox`
 
 8. Exit with `exit`. Try other commands from local.
 
 `docker-machine ssh aws-sandbox pwd`
 
  `docker-machine ssh aws-sandbox sudo mkdir my_ec2_dir`

 
 9. Copy the files. You can either [mount](https://docs.docker.com/machine/reference/mount/) the docker machine locally or scp your files. 
 To copy your directory of local files containing your app and docker-compose file, run:
 
 `docker-machine scp -r -d /path_to_my_dir/my_dir/ aws-sandbox:/home/path_somewhere/my_ec2_dir`
 
 10. Run docker-compose on your image. Either ssh and run docker-compose or...
 
  `docker-machine ssh aws-sandbox cd /home/path_somewhere/my_ec2_dir && docker-compose up`
  
  11. You might have to install various packages (e..g, docker-machine via sudo apt install docker-machine)


### Local Files > Packer > AMI > EC2 (One Instance with Alerts configurable in repo)
0. Create a key/secret pair in AWS and set these as env vars.
1. Create an app. App should have an app.rb at minimum. Then shell script to configure, and a build.json See [this repo](https://github.com/rustyrothwurt/terraform_ecs_docker/tree/master/packer_terraform/packer-docker-example) for an example
2. Update the variables to your app/folder structure
2. Then, run `packer build -only=ubuntu-ami build.json` or whatever
3. Copy the name of the AMI. Verify it was created in AWS (see links below)
4. Go to  [this repo](https://github.com/rustyrothwurt/terraform_ecs_docker/tree/master/packer_terraform/packer-docker-terraform-example). update files as necessary. 
5. run terraform commands. pass in the '-var ami_id=amiid' 

### Git <> Terraform > AWS CodePipeline > ECS (with Load Balancer, Automatic Scaling, and Alerts)
0. Get a Git developer token; set AWS key/secret
1. Update the vars for the [this repo](https://github.com/rustyrothwurt/terraform_ecs_docker/), repo owner, app name, etc.
2. Update the scaling vars.
3. Update the buildspec.yml
4. terraform commands
5. possible docker run tests (https://blog.pusher.com/full-stack-testing-docker-compose/)

## References
### Ruby-specific
* [Creating a simple ruby app with Docker](https://github.com/IcaliaLabs/guides/wiki/Creating-a-new-Rails-application-project-with-Docker)
* Running tests with rake solution 1 (https://stackoverflow.com/questions/9017158/running-ruby-unit-tests-with-rake)
* other solution: https://mallibone.wordpress.com/2012/02/14/run-your-tests-with-a-single-command/
* Dockerizing Ruby apps: https://blog.codeship.com/build-minimal-docker-container-ruby-apps/
* Docker compose test env: https://tomazy.com/blog/2017/05/testing-rails-app-in-docker-containers/

### ECS and Terraform and AWS
* ECS/Terraform/Fargate: https://thecode.pub/easy-deploy-your-docker-applications-to-aws-using-ecs-and-fargate-a988a1cc842f
* Repo for above https://github.com/duduribeiro/terraform_ecs_fargate_example and https://github.com/opensanca/opensanca_jobs

### Terraform
* https://github.com/gruntwork-io/infrastructure-as-code-training
* https://scoutapm.com/blog/deploying-to-aws-part-i-running-a-rails-app-on-fargate

### Packer
* packer docker https://www.bogotobogo.com/DevOps/Docker/Docker-Packer.php
This is terratest from gruntwork. needs go installed. can run tests locally or in aws
* https://github.com/gruntwork-io/terratest/tree/master/examples/packer-docker-example
* https://github.com/gruntwork-io/terratest/tree/master/examples/terraform-packer-example

### AWS
* Console https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1
* Code Pipeline https://console.aws.amazon.com/codesuite/codebuild/projects/web-app-codebuild/build/
* Cluster https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters/web-app/services
* EC2 https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#
* ECR https://us-east-1.console.aws.amazon.com/ecr/repositories?region=us-east-1#
* S3 bucket https://s3.console.aws.amazon.com/s3/home?region=us-east-1#
* IAM  https://console.aws.amazon.com/iam/home?region=us-east-1#/home
* AMI https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Images:sort=name

#### AWS Keys
*EC2 Key Pair*
`aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem`

`chmod 400 MyKeyPair.pem`

`aws ec2 describe-key-pairs --key-name MyKeyPair`

*Access Key*
`aws iam create-access-key --user-name MyUser`


### Git
* https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys
* Developer token 

### bitbucket
* terraform provider: https://www.terraform.io/docs/providers/bitbucket/index.html

## Installs Needed
*Docker*

*Docker-machine (Windows)*
using Git bash
`if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"`

*Unzip*
`sudo apt-get install unzip`

*pip*
`ls -al /usr/local/bin/python`
`pip3 --version`
`curl -O https://bootstrap.pypa.io/get-pip.py`
`python3 get-pip.py --user`

*python?*

### ruby/rails
`ruby -v`
`rails -v`
`gem -v`

`sudo apt-get install ruby-full`

### awscli
`pip3 install awscli --upgrade --user`
`update the path if needed`


### terraform
Check the hasicorp/terraform download pages for most recent release.
*Mac instructions*
`curl -LO https://releases.hashicorp.com/terraform/0.12.0/terraform_0.12.0_darwin_amd64.zip`
`unzip terraform_0.12.0_darwin_amd64.zip -d terraform`
`cp terraform /usr/local/bin/`
update your path
`terraform -v`

*Windows*
Download the zip from the [https://releases.hashicorp.com/terraform/0.12.0/](releases page).
Unzip it, and put it somewhere. I put mine in C:/Hashicorp/

### packer
Check the hasicorp/packer download pages for most recent release.
*Mac*
`curl -LO https://releases.hashicorp.com/packer/1.4.1/packer_1.4.1_darwin_amd64.zip`
`unzip packer_1.4.1_darwin_amd64.zip -d packer`
`cp packer /usr/local/bin/`
update your path
`packer -v`

*Windows*
Download the zip.
Unzip it, and put it somewhere. I put mine in C:/Hashicorp/

### other
check for npm, node, etc.
