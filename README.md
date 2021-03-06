Note: Extended from original AWS ECS Reference architecture to support production requirements

# ECS Reference Architecture: Continuous Deployment

The ECS Continuous Deployment reference architecture demonstrates how to achieve
[continuous deployment][continuous-deployment] of an application to Amazon
Elastic Container Service (Amazon ECS) using AWS CodePipeline and AWS
CodeBuild. With continuous deployment, software revisions are deployed to a
production environment automatically without explicit approval from a developer,
making the entire software release process automated.

Launching this AWS CloudFormation stack provisions a continuous deployment
process that uses AWS CodePipeline to monitor a GitHub repository for new
commits and AWS CodeBuild to create a new Docker container image and to push it
into Amazon Elastic Container Registry (Amazon ECR).

When creating this stack, you can opt to deploy the service onto [AWS
Fargate][fargate] or [Amazon EC2][ec2]. AWS Fargate allows you to run containers
without managing clusters or services. If you choose Amazon EC2, an Auto Scaling
group of t2.micro instances will be created to host your service.

[![](images/architecture.png)][architecture]

## Running a project

#### 1. Reference a working Docker Enabled Project

This reference solution stack can deploy any single Docker image project to AWS Fargate.

#### 2. Create the CloudFormation stack

Deploy | Region Name | Region | Launch Types
:---: | ------------ | ------------- | -------------
[🚀][us-east-1] | US East (N. Virginia) | us-east-1 | Fargate, EC2
[🚀][us-east-2] | US East (Ohio) | us-east-2 | EC2
[🚀][us-west-1] | US West (N. California) | us-west-1 | EC2
[🚀][us-west-2] | US West (Oregon) | us-west-2 | EC2
[🚀][eu-west-1] | EU (Ireland) | eu-west-1 | EC2
[🚀][eu-west-2] | EU (London) | eu-west-2 | EC2
[🚀][eu-central-1] | EU (Frankfurt) | eu-central-1 | EC2
[🚀][ap-southeast-1] | Asia Pacific (Singapore) | ap-southeast-1 | EC2
[🚀][ap-southeast-2] | Asia Pacific (Sydney) | ap-southeast-2 | EC2
[🚀][ap-northeast-1] | Asia Pacific (Tokyo) | ap-northeast-1 | EC2
[🚀][ap-northeast-2] | Asia Pacific (Seoul) | ap-northeast-2 | EC2
[🚀][ca-central-1] | Canada (Central) | ca-central-1 | EC2

This reference architecture can only be deployed to Regions which have all
necessary services available. See the [Region
Table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)
for information about service availability.

The CloudFormation template requires the following parameters:

- Cluster Configuration
  - **Launch Type**: Deploy the service using either AWS Fargate or Amazon EC2.
    Selecting EC2 will create an Auto Scaling group of t2.micro instances for
    your cluster. See the [documentation][launch-types] to learn more about
    launch types.
  - **Domain**: The domain name used to map to the LoadBalancer (a certificate will be generated)

- GitHub Configuration
  - **Repo**: The repo name of the sample service.
  - **Branch**: The branch of the repo to deploy continuously.
  - **User**: Your username on GitHub.
  - **Personal Access Token**: Token for the user specified above.
    ([https://github.com/settings/tokens](https://github.com/settings/tokens))
    
- Stack Configuration (configure in SSM ParameterStore)
  - ***GitHubToken***: The Github Personal Accesstoken 
  - ***TemplateBucket***: The bucket the template is stored at
  - ***AccessKey***: An AWS AccessKey with sufficient permissions to access Docker build dependencies
  - ***SecretKey***: Matching secretkey from AWS credential key pair 
  - ***Certificate***: The ARN for the SSL certificate  

The CloudFormation stack provides the following output:

- **ServiceUrl**: The sample service that is being continuously deployed.
- **PipelineUrl**: The continuous deployment pipeline in the AWS Management
  Console.

### Testing the project

After the CloudFormation stack is created, the latest commit to the GitHub
repository is run through the pipeline and deployed to ECS. Open the
**PipelineUrl** to watch the first revision run through the CodePipeline
pipeline. After the deploy step turns green, open the URL from **ServiceUrl**
which loads one or two outcomes ***after build***:
- ***On Failure*** : The default index page for Tomcat 8.0 will be displayed
- ***On Success*** : Contents returned at '/' of your service will be displayed 

To test continuous deployment, make a change to your source repository and push it to GitHub. CodePipeline detects the change, builds the new application, and deploys it to your cluster
automatically. After the pipeline finishes deploying the revision, reload the
page to see the changes made.

### Cleaning up the example resources

To remove all resources created by this project, do the following:

1. Delete the main CloudFormation stack which deletes the substacks and resources.
1. Manually delete resources which may contain content:

    - S3 Bucket: ArtifactBucket
    - ECR Repository: Repository

## CloudFormation template resources

The following sections explains all of the resources created by the
CloudFormation template provided with this project.

#### [DeploymentPipeline](templates/deployment-pipeline.yaml)

  Resources that compose the deployment pipeline include the CodeBuild project,
  the CodePipeline pipeline, an S3 bucket for deployment artifacts, and all
  necessary IAM roles used by those services.

#### [Service](templates/service.yaml)

  An ECS task definition, service, IAM role, and ECR repository for the sample
  application. This template is used by the CodePipeline pipeline to deploy the
  sample service continuously.

#### [Cluster](templates/ecs-cluster.yaml)

  An ECS cluster optionally backed by an Auto Scaling group of EC2 instances
  running the Amazon ECS-optimized AMI for the EC2 launch type.

#### [Load Balancer](templates/load-balancer.yaml)

  An Application Load Balancer to be used for traffic to the sample application.

#### [VPC](templates/vpc.yaml)

  A VPC with two public subnets on two separate Availability Zones, an internet
  gateway, and a route table with a default route to the public internet.

## License

This reference architecture sample is [licensed][license] under Apache 2.0.

[continuous-deployment]: https://aws.amazon.com/devops/continuous-delivery/
[architecture]: images/architecture.pdf
[license]: LICENSE
[fargate]: https://aws.amazon.com/fargate/
[ec2]: https://aws.amazon.com/ec2/
[launch-types]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_types.html
