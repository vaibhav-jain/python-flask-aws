# python-flask-aws [![DEV3L](https://circleci.com/bb/dev3l/python-flask-aws.svg?style=svg)](https://bitbucket.org/dev3l/python-flask-aws/)

![Cover Image](https://dev-to-uploads.s3.amazonaws.com/i/2bciy3vdg0et5erjh7k3.jpg)

## [Dockerized Python Flask app to AWS ECR/ECS using CircleCI](https://dev.to/dev3l/dockerized-python-flask-app-to-aws-ecr-ecs-using-circleci-6ko)

I spent some time yesterday getting a Python [Flask](https://www.palletsprojects.com/p/flask/) application into Amazon Web Services (AWS). Hopefully, this tutorial can help you save some time in doing the same for your own Python (or Docker) web applications!

![Flask hello-world AWS](https://dev-to-uploads.s3.amazonaws.com/i/bby0p0obq3ms1xgo16mq.png)

Continuous Integration (CI) and Continuous Deployment (CD) are powerful techniques that enable teams to rapidly collaborate to produce consistent value. Each code push to a designated branch kicks off the pipeline to build, test, and deploy code to the target environment. CircleCI is an integration pipeline as a service that enables developers to create pipelines without having to worry about managing the infrastructure.

Docker is a wonderful technology that standardizes the management and delivery of applications. It's like the USPS motto, "If it fits, it ships". All of the major cloud providers have a solution to deploy and manage Docker containers, in this tutorial we will be utilizing AWS's ECR/ECS to do so.

# Prerequisites

- [Python 3+](https://www.python.org/downloads/)
- [Python Virtual Environments](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/)
- [CircleCI Account](https://circleci.com/)
- [Docker](https://www.docker.com/products/docker-desktop)
- [Terraform](https://www.terraform.io/)
- [GitHub](https://github.com/) / [BitBucket](https://bitbucket.org/)
- [AWS Account](https://aws.amazon.com/)

# References

- [Deploying to AWS ECR/ECS](https://circleci.com/docs/2.0/ecs-ecr/)
- [Dockerize your Flask Application](https://runnable.com/docker/python/dockerize-your-flask-application)
- [DEV3L: python-flask-aws](https://bitbucket.org/dev3l/python-flask-aws/src/master/)

# Walkthrough

## 1. Git clone [DEV3L: python-flask-aws](https://bitbucket.org/dev3l/python-flask-aws/src/master/)

This is a barebones repository. It is intentionally slim, so you can focus on getting something up and running sooner than later.

```
: git clone https://dev3l@bitbucket.org/dev3l/python-flask-aws.git
: cd python-flask-aws
plython-flask-aws: 
```

## 2. Run Python Flask hello-world application

The following commands executed from the command line will create a new virtual environment called `python-flask-aws`, activate it, install the dependencies inside of the `requirements.txt` file, and finally launch the hello-world Flask application.

```
: python3 -m venv python-flask-aws
: source python-flask-aws/bin/activate
: pip install -r requirements.txt
: python app.py
---
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 262-674-088
```

We should be able to load the website by going to `http://localhost` or `http://0.0.0.0`. Once verified, go back to the terminal and press `ctrl-c` to stop the application.

## 3. Build the Docker image

Now that we know the application can run outside of Docker, let's verify that we can get it to run inside of a container.

```
: docker build -t python-flask:latest .
: docker run -d -p 80:80 python-flask
```

Once again, at this point verify that the application is running available through the browser at `http://localhost`

## 4. Hookup CircleCI

Let's disconnect your local copy of the repository and push it up to your own. This could be done through a fork or manually. If not done through a fork, create a new empty git repository on GitHub or BitBucket.

```
: rm -rf .git
: git remote add origin <clone url>
: git add .
: git commit "Add flask-aws project scaffolding"
: git push
```

Once you have a separate or forked copy of the repository, in the browser navigate to [CircleCI](https://circleci.com/). From the left menu, choose Add Projects. If not already done, allow CircleCI to connect to your repositories. Click the `Set Up Project` on your repository from the list.

CircleCI uses `./circleci/config.yml` as a template to build your application. Now we need to set build environment variables that are referenced within this file.

In order to do this we need four pieces of information:

- AWS Access Key
- AWS Secret Key
- AWS Account Id
- AWS Region

References on how to do get this information can be found [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) for the access and secret key. Your account id can be found under the `My Account` menu from the AWS console.

In CircleCI for the pipeline, go to Setting and enter the following environment variables:

| Variable                       | Description                                               |
| ------------------------------ | --------------------------------------------------------- |
| `AWS_ACCESS_KEY_ID`            | Used by the AWS CLI                                       |
| `AWS_SECRET_ACCESS_KEY `       | Used by the AWS CLI                                       |
| `AWS_DEFAULT_REGION`           | Used by the AWS CLI. Example value: "us-east-1" (Please make sure the specified region is supported by the Fargate launch type)                          |
| `AWS_ACCOUNT_ID`               | AWS account id. This information is required for deployment.                                   |
| `AWS_RESOURCE_NAME_PREFIX`     | Prefix that some of the required AWS resources are assumed to have in their names. The value should correspond to the `aws_resource_prefix` variable value in `terraform_setup/terraform.tfvars`.        

## 5. Use Terraform to create AWS infrastructure

We need to create the AWS infrastructure necessary to host our application. This can be done manually [following AWS tutorials](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-ecs-ecr-codedeploy.html) to do so, but Terraform is a cloud-agnostic way to scaffold the pieces necessary. This approach allows us to easily create environments that are identical - for example, if we wanted to create a stack for our test environment that mirrors a production environment.

From the example repository, open up and edit the `terraform.tfvars` file located in the `cloudformation-templates` directory. I added this file into the .gitignore as to not accidentally push these secrets to the public repository, but please be careful on each push. Set the following to your specific values.

```
aws_access_key = ""
aws_secret_key = ""
aws_account_id = ""
aws_region     = ""
```

Once this is done, open up a terminal session from inside the `cloudformation-templates` directory.

```
: terraform init
: terraform plan
: terraform apply
```

It will take a few minutes. Afterward, you will now have the necessary infrastructure to deploy the application to AWS.

## 6. Create a successful build

After all this preparation, we are finally ready to see the magic happen! With our repository setup, the pipeline in place, and infrastructure scaffolded through Terraform, all we have to do now is push a change to the master branch. 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/5tlhymgu9md2m151y7vx.png)

Once the build is green, the last step is to find the URL our application is hosted on!

## 7. View application on AWS

I am sure there is a better way to do this, but I am going to walk through the steps that I followed to find the DNS A Record entry created by AWS.

- Sign in to the AWS Console
- Go to the EC2 service
- From the left menu, select Load Balancers
- The DNS name field for your load balancer

# Conclusion

There is still much to be done here. For example, how do we enable SSL? I wanted this to be a starting point to create a CI/CD pipeline into AWS. Granted we only deployed a 'Hello, World!' application, the toolchain utilized is complex. Having an example lets us layer upon the complexity.
