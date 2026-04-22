# Step-by-step Guide to Build an End-to-End CI/CD Pipeline with AWS, Terraform, Docker, Kubernetes & Helm
Kubernetes is an open-source platform for automating deployment, scaling, and management of containerized applications. It introduces the Pod as the smallest deployable unit. The Pod encapsulates one or more containers that share the same network, namespace and can access shared storage volumes. This abstraction allows Kubernetes to manage groups of tightly coupled containers as a single logical unit, simplifying communication and coordination. Kubernetes as an orchestration tool for containers, allows them to run reliably, scale up and down, and communicate with each other across many servers, making sure the application is always running and has enough resources.

Previously, we demonstrated how to deploy applications using Kubernetes, Docker, Terraform and Helm. We have mastered each component : Docker for packaging, Terraform for infrastructure, Kubernetes for orchestration, and Helm for application packaging. In the real world, those components do not function individually and in isolation; they are integrated into an automated Continuous Integration and Continuous Delivery (CI/CD) pipeline.
A CI/CD pipeline automates the entire software release process, from developers committing code to the application running in production. This means:
- Continuous Integration (CI): Every code change is automatically built, tested, and integrated into a shared repository.
- Continuous Delivery (CD): Changes are automatically prepared for release to production, allowing for rapid and reliable deployments.

This comprehensive step-by-step guide walks you through the process of Building an End-to-End CI/CD Pipeline with AWS, Terraform, Docker, Kubernetes and Helm. In this projectb, we will build a complete end-to-end CI/CD pipeline using AWS CodePipeline to orchestrate the workflow, AWS CodeBuild to build our Docker images and prepare our Helm Chart, AWS ECR to store the images, and finally deploy our application to AWS EKS using our Helm Chart. All of this infrastructure, including the pipeline itself, will be defined and managed by Terraform. The aim of this project is to learn :  
- The components of an end-to-end CI/CD pipeline.
- How to use AWS CodePipeline to orchestrate build and deploy stages.
- How to configure AWS CodeBuild for multi-stage builds (Docker build, Helm package).
- How to integrate GitHub as a source for CodePipeline.
- How to use Terraform to provision the entire CI/CD pipeline infrastructure.
- How to deploy a Helm Chart to EKS automatically from a pipeline.
- The interaction between Git, CodeBuild, ECR, EKS, and Helm in an automated workflow.

## Prerequisites
- You must have successfully deployed the Full-Stack App on EKS with RDS and with Helm.
- Your EKS cluster should be Running and your kubeconfig should be configured to connect to it. (We will manage its lifecycle with this new Terraform setup).
- Your my-flask-app Docker image should be in your AWS ECR repository.
- Your docker-compose-app folder should be ready.
- Your my-flask-chart Helm Chart should be ready.
- GitHub Repository: You will need a single GitHub repository containing both your docker-compose-app folder (which includes app.py, Dockerfile, requirements.txt, templates/, and buildspec.yml for the Docker build) AND your my-flask-chart folder.

**Recommended Structure in GitHub**:
<img width="546" height="327" alt="0" src="https://github.com/user-attachments/assets/f633b2e9-dfe4-4c7b-9a34-d8e6e086c8fa" />

**Important**: The buildspec.yml in docker-compose-app will be updated for CodeBuild. The my-flask-chart/values.yaml will be updated to use the ECR image.

## Step-by-step instructions
### Step 1: Prepare Your Application Code and Helm Chart for the Pipeline
For the CI/CD pipeline, CodeBuild will need a buildspec.yml to build the Docker image and then package the Helm Chart. The Helm Chart's values.yaml needs to be updated to use the ECR image that CodeBuild will push. To complete Step 1 follow the instructions below : 
- Update buildspec.yml: Open docker-compose-app/buildspec.yml  and replace its content with   this new version.  This buildspec.yml will be used by the Build Stage of CodePipeline.
<img width="937" height="768" alt="1" src="https://github.com/user-attachments/assets/35b6f818-1df7-48fc-8b43-03a3f8c3c4b7" />


- Update my-flask-chart/values.yaml: Open my-flask-chart/values.yaml and ensure its image section points to your ECR repository. The tag will be dynamically replaced by buildspec.yml.
<img width="694" height="577" alt="2" src="https://github.com/user-attachments/assets/59c03f4d-2186-4461-b2ee-997192c93cd1" />

- Push all changes to your GitHub repository using the command :
<img width="1148" height="520" alt="3" src="https://github.com/user-attachments/assets/96ec2726-beba-4188-96de-5fc50c4ad3eb" />
<img width="797" height="228" alt="4" src="https://github.com/user-attachments/assets/0eb50dd0-6c9c-4343-a881-83714260c7f2" />
<img width="691" height="397" alt="5" src="https://github.com/user-attachments/assets/a7ce4d8f-9aaf-46dc-b233-996c7df02bda" />
<img width="686" height="101" alt="6" src="https://github.com/user-attachments/assets/6ee3fea7-253e-4fb0-a19a-a1bde3bd9783" />
<img width="887" height="236" alt="7" src="https://github.com/user-attachments/assets/07138cd6-c0d8-4e89-bbcc-0c5976c535b7" />
<img width="934" height="425" alt="8" src="https://github.com/user-attachments/assets/e7744187-2acc-49b8-9f9e-d15f89fb00b6" />

**Note** : Make sure your docker-compose-app/ folder (with the updated buildspec.yml) and your my-flask-chart/ folder (with the updated values.yaml) are pushed to the same GitHub repository. This is the repository CodePipeline will monitor.


### Step 2: Update Terraform Configuration for CI/CD Pipeline
We need to add Terraform resources for AWS CodePipeline, CodeBuild, and an S3 bucket to store pipeline artifacts. This will involve defining new IAM roles and policies to grant these services the necessary permissions to interact with each other and with EKS. To complete Step 2 follow the instructions below : 
- Open your eks-cluster-terraform folder.
- Create a new file pipeline.tf and add the following content :
<img width="883" height="860" alt="9" src="https://github.com/user-attachments/assets/b4401a50-e8cd-4df6-95d8-1fd01370404a" />
<img width="902" height="830" alt="10" src="https://github.com/user-attachments/assets/81637893-15f3-4f79-9e0c-c717d16e107f" />
<img width="815" height="843" alt="11" src="https://github.com/user-attachments/assets/d2560908-8cf8-4e14-b161-225cbe91e174" />
<img width="1388" height="854" alt="12" src="https://github.com/user-attachments/assets/25816081-a19a-4d76-9a36-84fe71731857" />
<img width="1366" height="858" alt="13" src="https://github.com/user-attachments/assets/2710850f-24b9-499d-91ab-b44939976a6d" />
<img width="855" height="540" alt="14" src="https://github.com/user-attachments/assets/b6f03057-2b43-47a4-b9b2-6f1fc1f56f73" />
<img width="1398" height="606" alt="15" src="https://github.com/user-attachments/assets/9da6895a-75ac-4b8a-a4f6-8cb0d557d511" />
<img width="1356" height="568" alt="16" src="https://github.com/user-attachments/assets/4ae2d2ed-8bc6-40b7-875e-b53b1ff94097" />
<img width="1368" height="585" alt="17" src="https://github.com/user-attachments/assets/d6c7ac76-04e1-4117-a745-bc4597c0f0bb" />
<img width="1399" height="475" alt="18" src="https://github.com/user-attachments/assets/808f6836-93d1-4be6-b4e9-974112678a83" />

- Update eks-cluster-terraform/variables.tf to include the new variables for the pipeline.
- Save all updated Terraform files.
<img width="873" height="827" alt="19" src="https://github.com/user-attachments/assets/c5acc278-e65f-4afd-b178-e3365eade4d2" />
<img width="945" height="820" alt="20" src="https://github.com/user-attachments/assets/27f7dc09-20df-41e1-a0b3-b51f8ce110c0" />


### Step 3: Apply Terraform Changes (Provision CI/CD Pipeline)
This step will provision all the new AWS resources for your CI/CD pipeline, including the S3 bucket, IAM roles, CodePipeline, and the new CodeBuild project for deployment. To complete Step 3 follow the instructions below : 
Open your command line or terminal and Navigate to your eks-cluster-terraform folder.
- Initialize Terraform using the command : terraform init. Ensure all new providers and modules are downloaded.
<img width="660" height="297" alt="21" src="https://github.com/user-attachments/assets/58d11270-17b5-46dd-8d54-01444649286c" />

- Plan your infrastructure changes using the command : terraform plan
**Expected output** : Review the plan carefully. You will see many new resources related to CodePipeline, CodeBuild, and S3.
<img width="988" height="800" alt="22" src="https://github.com/user-attachments/assets/f1b9c5b7-3993-4cd0-8f98-624632d4c79b" />
<img width="595" height="828" alt="23" src="https://github.com/user-attachments/assets/32648a7d-c032-4f0b-a386-7c0bd4a4b1b3" />
<img width="609" height="827" alt="24" src="https://github.com/user-attachments/assets/63564b06-2445-41e2-82c7-f9d1b0976b1a" />
<img width="863" height="831" alt="25" src="https://github.com/user-attachments/assets/329c9d59-29af-47df-88bc-248a3a82e0ec" />
<img width="1381" height="833" alt="26" src="https://github.com/user-attachments/assets/fcdf2132-860c-4ae5-bb61-97dbbd29944b" />
<img width="979" height="813" alt="27" src="https://github.com/user-attachments/assets/cc2bb046-220e-4e1d-b0c0-34ccf4b5d1bc" />
<img width="586" height="826" alt="28" src="https://github.com/user-attachments/assets/71e94203-0214-42a7-b0ff-096aa5ae0e31" />
<img width="643" height="834" alt="29" src="https://github.com/user-attachments/assets/ed82eeb7-4051-417d-b4d5-8c0c51c137e7" />
<img width="484" height="808" alt="30" src="https://github.com/user-attachments/assets/64a153d5-8f90-4ecc-8e03-d58613c8ce7b" />
<img width="544" height="820" alt="31" src="https://github.com/user-attachments/assets/e34e641e-7ecb-4600-abc8-f9a9a05c36af" />
<img width="637" height="832" alt="32" src="https://github.com/user-attachments/assets/e78a8e5d-2968-4efb-bf6b-322c58403516" />
<img width="448" height="822" alt="33" src="https://github.com/user-attachments/assets/cbdd77a3-eb37-42de-b62e-3aaed8afd54f" />
<img width="711" height="813" alt="34" src="https://github.com/user-attachments/assets/bb4c5b0e-4346-4ef1-bf10-734389576841" />
<img width="1424" height="814" alt="35" src="https://github.com/user-attachments/assets/e003d744-53b0-4999-a337-2c039cd38e86" />
<img width="642" height="819" alt="37" src="https://github.com/user-attachments/assets/272a6383-4437-4bef-b74f-7cb781a4f803" />
<img width="605" height="806" alt="38" src="https://github.com/user-attachments/assets/85cb7574-8ee6-4090-b5d7-be00e8cd039f" />
<img width="1120" height="444" alt="43" src="https://github.com/user-attachments/assets/51755879-ee9c-4bb3-b9e9-5f5f224d4b58" />

- Apply your infrastructure changes using the command : terraform apply. Type yes to confirm when prompted.
**Expected output** :  This process will take several minutes to provision the pipeline resources.
<img width="1442" height="868" alt="64" src="https://github.com/user-attachments/assets/a68ba442-a5f1-47a1-9e85-2860b7830fda" />


Error
Explanation
Solution
Troubleshooting Steps
CHECK ERROR AND TROUBLESHOOTING

### Step 4: Trigger the Pipeline and Observe Deployment
Your entire CI/CD pipeline is now set up! Any new code push to your configured GitHub branch will automatically trigger the pipeline. For the first run, you might need to manually trigger it or make a small dummy commit. To complete Step 4 follow the instructions below : 
Manually trigger the pipeline (optional, or make a dummy commit):
Go to the AWS Management Console and search for "CodePipeline."
Click on "Pipelines" and find your pipeline (e.g., my-k8s-cluster-app-pipeline).
Click on the pipeline name.
In the "Source" stage, click "Release change" to manually start the pipeline.
Alternatively, make a small, harmless change to a file in your GitHub repository (e.g., add a comment to app.py) and push it. This should automatically trigger the pipeline.

error : 
the source stage : Unable to use Connection: arn:aws:codeconnections:us-east-1:694862618269:connection/d9ae2bd9-96a7-4227-bc28-324643e1997c. The provided role does not have sufficient permissions.
error : The provided role does not have sufficient permissions
It means the AWS CodePipeline service role (the IAM role you assigned to the pipeline) does not have permission to use the GitHub CodeStar connection.
add : 
{
  Effect = "Allow"
  Action = [
    "codestar-connections:UseConnection"
  ]
  Resource = var.github_connection_arn
}






error : Latest action execution message
Build terminated with state: FAILED. Phase: QUEUED, Code: ACCESS_DENIED, Message: Service role arn:aws:iam::694862618269:role/codebuild-my-flask-app-role does not allow AWS CodeBuild to create Amazon CloudWatch Logs log streams for build arn:aws:codebuild:us-east-1:694862618269:build/my-flask-app-build:9529455f-6ebc-4020-bf52-45c49d6b9e18. Error message: User: arn:aws:sts::694862618269:assumed-role/codebuild-my-flask-app-role/AWSCodeBuild-9529455f-6ebc-4020-bf52-45c49d6b9e18 is not authorized to perform: logs:CreateLogStream on resource: arn:aws:logs:us-east-1:694862618269:log-group:/aws/codebuild/my-flask-app-build:log-stream:build-log-stream/9529455f-6ebc-4020-bf52-45c49d6b9e18 because no identity-based policy allows the logs:CreateLogStream action


Ah! That error is very common with CodeBuild. It’s not about your S3 or pipeline—it’s IAM permissions for the CodeBuild service role.
Here’s what’s happening:
Your CodeBuild project uses a service role (codebuild-my-flask-app-role).
CodeBuild needs to write logs to CloudWatch Logs.
Currently, the IAM role does not have permissions for logs:CreateLogStream and logs:PutLogEvents.
Without these, the build fails immediately in the QUEUED phase.

How to fix it
Add a policy to the CodeBuild IAM role allowing CloudWatch Logs actions. Here’s an example inline policy:
add : 
resource "aws_iam_role_policy" "codebuild_logs_policy" {
  name = "${var.cluster_name}-codebuild-logs-policy"
  role = aws_iam_role.codebuild_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:${var.aws_region}:${data.aws_caller_identity.current.account_id}:log-group:/aws/codebuild/*"
      }
    ]
  })
}


or

#######################################################
# --- IAM Role Policy for CodeBuild Logs (CloudWatch) ---
#######################################################
resource "aws_iam_role_policy" "codebuild_logs_policy" {
  name = "${var.cluster_name}-codebuild-logs-policy"
  role = aws_iam_role.codebuild_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:${var.aws_region}:${data.aws_caller_identity.current.account_id}:log-group:/aws/codebuild/*"
      }
    ]
  })
}


The error can comes from an IAM permissions, missing environment variables or networking issues . 









Monitor the pipeline execution:
Watch the pipeline stages in the CodePipeline console. You'll see "Source," "Build," and "Deploy" stages.
Click on the "Details" link for each stage to see the logs in CodeBuild.
In the "Build" stage logs, you should see Docker image building, pushing to ECR, and Helm chart packaging.
In the "Deploy" stage logs, you should see kubectl and helm commands being executed against your EKS cluster.
Verify Docker image in ECR :
Once the "Build" stage succeeds, go to AWS ECR and check my-flask-app-repo. You should see new image tags corresponding to your latest commit.
Verify application deployment in EKS: Once the "Deploy" stage succeeds, open your local terminal.
Ensure your kubectl is still configured to your EKS cluster using the command : aws eks update-kubeconfig --name my-k8s-cluster --region us-east-1
Check your Kubernetes resources using the commands :
kubectl get deployments -l app=flask-message-board kubectl get pods -l app=flask-message-board kubectl get service flask-message-board-service
Get the EXTERNAL-IP (DNS name) of your flask-message-board-service (it might take a few minutes for the ALB to update). Open your web browser and access http://<ALB_DNS_NAME>.
Expected output : You should see your "Simple Message Board" application, now deployed completely automatically by your CI/CD pipeline! Try adding messages to confirm it's working.
Congratulations! You've successfully built and deployed a full-stack application to EKS using an automated CI/CD pipeline managed by Terraform and leveraging Docker, Kubernetes, and Helm. This is a truly production-ready workflow!
### Step 5 : Clean Up (Extremely Crucial!)
Running an EKS cluster, RDS database, and a CI/CD pipeline incurs significant costs. It is absolutely critical to destroy all resources created in this lab and previous labs (Lab 14, 15) when you are finished to avoid ongoing charges. To complete Step 5 follow the instructions below : 
Delete the application from EKS (optional, terraform destroy will do this anyway, but good practice): kubectl delete -f flask-app-eks-deployment-service.yaml
Destroy all AWS infrastructure created by Terraform : cd eks-cluster-terraform to Ensure you are in the correct directory and run the command : terraform destroy. Type yes and press Enter to confirm.


Expected output : Be patient! Destroying all these resources (EKS, RDS, CodePipeline, CodeBuild, S3, IAM, VPC) takes a considerable amount of time (20-40+ minutes). Terraform will provide progress updates. Wait for "Destroy complete!" message. This ensures you don't leave resources running and incur costs.
### Summary
This breakdown provides a step-by-step guide to Build a complete End-to-End CI/CD Pipeline with AWS, Terraform, Docker, Kubernetes and Helm. By completing this lab, we had an overview on : 
How to use AWS CodePipeline to orchestrate build and deploy stages.
How to configure AWS CodeBuild for multi-stage builds (Docker build, Helm package).
How to integrate GitHub as a source for CodePipeline.
How to use Terraform to provision the entire CI/CD pipeline infrastructure.
How to deploy a Helm Chart to EKS automatically from a pipeline.
By integrating Terraform, Docker, Kubernetes, and Helm within an AWS environment, we automated the entire application lifecycle, from infrastructure provisioning to container deployment and continuous updates. This approach ensured reproducibility, scalability, and consistent builds across different environments. Additionally, by incorporating CI/CD tools, we automated the build, test, and deployment processes, enabling rapid and reliable software delivery with minimal manual intervention. Overall, this workflow reinforced best practices in DevOps and infrastructure automation, demonstrating the power of combining these tools to build a robust, scalable, and maintainable CI/CD pipeline.
