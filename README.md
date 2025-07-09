# Welcome to My Project!  
By following the steps below, you can run the project on your local machine. Good luck!

# You need to fork both "https://github.com/Furkan-Alay/GitopsProjem" and "https://github.com/Furkan-Alay/GitappProjem" repositories from the resource section via SSH link.

# Then, open your terminal and create an SSH key. Run the following commands one by one:
* cd
* cd ~/.ssh
* ssh-keygen =>>> This will generate a private and public SSH key.
* cat [your-ssh-key-name] =>>> Copy the content from here.

# Now, add your generated public SSH key to your GitHub SSH keys section.  
(GitHub Account >>> Settings >>> SSH and GPG keys >>> New SSH key >>> Enter necessary information and click Add)

# Install Git if it's not already installed on your terminal.

# Open your terminal:
* mkdir ~/Desktop/[YourFolderName]
* cd ~/Desktop/[YourFolderName]
* Go to your forked source codes and copy the SSH clone links.
* git clone [SSH clone link for Terraform source code]
* git clone [SSH clone link for Application source code]

## In this project, we assume the Terraform source code is named "iac-vprofile" and the Application source code is "vprofile-action".  
If you used different names, make sure to replace them accordingly.

* ls
* cd iac-vprofile
* git config core.sshCommand "ssh -i ~/.ssh/gitaction -F /dev/null"
* cd ../vprofile-action/
* git config core.sshCommand "ssh -i ~/.ssh/gitaction -F /dev/null"

## Enter your own GitHub username and email:
* git config --global user.name devops541
* git config --global user.email furkanalay428@gmail.com
* cd ..

### Create a new folder (you can give it any name you like):
* cp -r iac-vprofile/ main-iac
* cd iac-vprofile
* git checkout stage

### Now let’s set up our GitHub Secrets.  
We’ll add AWS Access Key, Secret Key, S3 Bucket, and ECR Repository information to the GitHub Secret section.

* Go to both your forked Terraform and Application repositories, then click on "Secrets and variables" under the "Actions" tab.
* Log in to your AWS account and select the region you'll be using throughout the project.
* Open the "IAM" service, create a new user with "Administrator Access" permissions.
* Generate "Access Key" and "Secret Key" for this IAM user.
* Copy the Access Key and Secret Key values.
* Add these keys to the Secrets section in both your Terraform and Application repositories on GitHub.

* Create an "S3 Bucket" in your AWS account and copy its name.
* Add this S3 Bucket name to the Secrets section in your Terraform repository since we’ll use it for Terraform state management.

* Create a private repository in AWS ECR (Elastic Container Registry) and copy its URL.
* Add this ECR URL to the Secrets section in your Application repository.

### Now let’s configure the Terraform code to create the VPC infrastructure and EKS Cluster:
* Open your Terraform source code folder "iac-vprofile" with VSCode.
* Open the "variables.tf" file inside the "terraform" folder.  
Set your AWS region and EKS cluster name under the "default" section.

* Open the "terraform.tf" file in the same folder.  
In the "backend s3" block:
  * Replace the "bucket" value with your AWS S3 Bucket name.
  * Set the "region" value to your AWS region.
  * Leave the "key" value unchanged.

* Open the "vpc.tf" file and set the "name" value to your chosen EKS Cluster name.

* Go to VSCode Source Control panel and select "Commit & Push" to push your project to GitHub.
* Enter a commit message and click "Save".

### Now, let’s run our Terraform code in GitHub Actions:
* Open your "iac-vprofile" folder with VSCode.
* Create a folder named ".github/workflows".
* Inside this folder, create a file named "terraform.yml".
* Paste the following code into your "terraform.yml" file:

```bash
name: "Vprofile IAC"

on:
  push:
    branches:
      - main
      - stage
    paths:
      - terraform/**
  pull_request:
    branches:
      - main
    paths:
      - terraform/**

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  BUCKET_TF_STATE: ${{ secrets.BUCKET_TF_STATE }}
  AWS_REGION: us-east-2
  EKS_CLUSTER: vprofile-eks

jobs:
  terraform:
    name: "Apply terraform code changes"
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./terraform

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Terraform with specified version on the runner
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.3

      - name: Terraform init
        id: init
        run: terraform init -backend-config="bucket=$BUCKET_TF_STATE"

      - name: Terraform format
        id: fmt
        run: terraform fmt -check

      - name: Terraform validate
        id: validate
        run: terraform validate

      - name: Terraform plan
        id: plan
        run: terraform plan -no-color -input=false -out planfile
        continue-on-error: true

      - name: Terraform plan status
        if: steps.plan.outcome == 'failure'
        run: exit 1
```

* Yukarıda yapıştırdığım kod içerisindeki "AWS_REGION" kısmına S3 Bucket ve ECR bölgemizi ekliyoruz.
* VSCode Source Control kısmından Commit&Push seçeneğine tıklıyoruz ve Commit mesajını yazdıktan sonra Save basıyoruz.
* Github Actions kısmında Pipeline adımlarının gerçekleştiğini görmüş olacağız. 
### Şimdi ise Terraform kodlarımızla VPC Altyapısını ve EKS Cluster yapısını oluşturalım
* terraform.yml dosyanızı açıp bu içeriği ekleyin:
``` bash
       - name: Terraform Apply
         id: apple
         if: github.ref == 'refs/heads/main' && github.event_name == 'push'
         run: terraform apply -auto-approve -input=false -parallelism=1 planfile

       - name: Configure AWS credentials
         uses: aws-actions/configure-aws-credentials@v1
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: ${{ env.AWS_REGION }}
     
       - name: Get Kube config file
         id: getconfig
         if: steps.apple.outcome == 'success'
         run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER }} 

       - name: Install Ingress controller
         if: steps.apple.outcome == 'success' && steps.getconfig.outcome == 'success'
         run: kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```
* The following commands are the final ones to create the VPC and EKS Cluster. You should combine them with the previous commands.
* We added content to our terraform/ folder and performed Commit & Push.
* In the GitHub Actions section, the workflow will start, but the latest commands won’t run yet because we haven’t pushed them to the "main" stage.  
We will perform this push operation at the end of the project.

* Open the folder containing both Terraform and Application source code in your terminal.  
Run the following commands to perform the merge operation:
* cd iac-vprofile/
* git checkout stage
* git pull
* git checkout main
* git add .
* git commit -m "This is a Fix message"
* git merge stage

### Now let’s enter the Application’s Workflow section.  
We’ll set up SonarCloud, create a Sonar token, and configure GitHub Secrets.

* Go to https://sonarcloud.io/ and sign up with your GitHub account to create a SonarCloud account.
* Click "Create new organization" and manually create an organization.  
Select the "Free Plan" — it will be sufficient.  
Click "Analyze a new project". Enter the same value for both "Display name" and "Project Key". Select "Public" visibility.

* On the top right, click "My account", then "Security".  
Generate a new token and copy the generated password.  
In your GitHub repository’s Secrets section, create a new Secret:
  * Name: `SONAR_TOKEN`
  * Value: Paste your token password.

* Add the name of your created organization as a Secret:
  * Name: `SONAR_ORGANIZATION`
  * Value: Enter your organization name.

* Create another new Secret:
  * Name: `SONAR_PROJECT_KEY`
  * Value: Enter your Project Key.

* Lastly, create one more GitHub Secret:
  * Name: `SONAR_URL`
  * Value: `https://sonarcloud.io`

* Open the forked application source code in VSCode.
* Create a folder named `.github/workflows`.
* Inside this folder, create a file named `main.yml`.
* Copy and paste the following code into your `main.yml` file:

```bash
name: vprofile actions
on: workflow_dispatch
env:
  AWS_REGION: us-east-2
  ECR_REPOSITORY: vprofileapp
  EKS_CLUSTER: vprofile-eks

jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test

      - name: Checkstyle
        run: mvn checkstyle:checkstyle

      # Setup Java 11 to be default (sonar-scanner requirement as of 5.x)
      - name: Set Java 11
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin' # See 'Supported distributions' for available options
          java-version: '11'

      # Setup sonar-scanner
      - name: Setup SonarQube
        uses: warchant/setup-sonar-scanner@v7

      # Run sonar-scanner
      - name: SonarQube Scan
        run: sonar-scanner
          -Dsonar.host.url=${{ secrets.SONAR_URL }}
          -Dsonar.login=${{ secrets.SONAR_TOKEN }}
          -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
          -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
          -Dsonar.sources=src/
          -Dsonar.junit.reportsPath=target/surefire-reports/ 
          -Dsonar.jacoco.reportsPath=target/jacoco.exec 
          -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
          -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/  

      # Check the Quality Gate status.
      - name: SonarQube Quality Gate check
        id: sonarqube-quality-gate-check
        uses: sonarsource/sonarqube-quality-gate-action@master

      # Force to fail step after specific time.
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_URL }} #OPTIONAL
```
* From the VSCode Source Control panel, click "Commit & Push".
* Go to your application source code repository, then to the "Actions" tab.  
Click on "vprofile-actions", then "Run workflow".  
It will fail the first time — run it again and the test process will complete.

### Now, let's build and publish the Docker container image.
* In the `main.yml` file, paste the following code exactly without changing its location in the file:

```bash
  BUILD_AND_PUBLISH:   
    needs: Testing
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Build & Upload image to ECR
        uses: appleboy/docker-ecr-action@master
        with:
          access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          registry: ${{ secrets.REGISTRY }}
          repo: ${{ env.ECR_REPOSITORY }}
          region: ${{ env.AWS_REGION }}
          tags: latest,${{ github.run_number }}
          daemon_off: false
          dockerfile: ./Dockerfile
          context: ./

```
Paste this code into your main.yml file without modifying its position.

From the VSCode Source Control panel, perform a "Commit & Push".

Go to the "Actions" tab of your application source code repository.
Click "vprofile-actions", then "Run workflow" and your project will run successfully.

Go to the AWS ECR Registry service. Click on your application repository and you'll see the Docker image has been built and published.

Now, let's configure the final EKS Deploy step.
Install the Kubernetes-Helm package on your Windows Powershell and Gitbash terminal.

Enter your application source code directory and run these commands:
```bash
helm create vprofilecharts

rm -rf vprofilecharts/templates/*

cp kubernetes/vpro-app/* vprofilecharts/templates/

cd vprofilecharts/templates/
```
Open your application source code with VSCode.
Go to helm/vprofilecharts/templates/vproappdep.yml and replace this content:

image: vprofile/vprofileapp
with

image: {{ .Values.appimage}}:{{ .Values.apptag }}

Now, add the following code to your main.yml file:
```bash
  DeployToEKS:
    needs: BUILD_AND_PUBLISH
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Get Kube config file
        run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER }}

      - name: Print config file
        run: cat ~/.kube/config

      - name: Login to ECR
        run: kubectl create secret docker-registry regcred --docker-server=${{ secrets.REGISTRY }} --docker-username=AWS  --docker-password=$(aws ecr get-login-password)

      - name: Deploy Helm
        uses: bitovi/github-actions-deploy-eks-helm@v1.2.8
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
          cluster-name: ${{ env.EKS_CLUSTER }}
          #config-files: .github/values/dev.yaml
          chart-path: helm/vprofilecharts
          namespace: default
          values: appimage=${{ secrets.REGISTRY }}/${{ env.ECR_REPOSITORY }},apptag=${{ github.run_number }}
          name: vprofile-stack

```
From the VSCode Source Control panel, perform a "Commit & Push".

Go to the GitHub Actions section, click "vprofile-actions", then "Run workflow" and the workflow will start.

After all steps are completed, all resources will be created and your application will be deployed inside a Docker container.
Your application will now be running.

To access your application:

Go to AWS Load Balancer service.

Copy the DNS address.

Paste it into your browser — your application will appear.

Optionally, you can purchase a domain service and register the Load Balancer DNS address as a CNAME record.
This way, users can access your application using your custom domain name.

Since our application is complete, let's explain how to clean up all resources.
The cleanup will consist of two steps: deleting the Ingress Controller and using the terraform destroy command.

Log into your AWS account, go to the IAM User section, and select your created user.

Generate a new Access and Secret Key.

Open Gitbash terminal and run:
```bash
aws configure
```
Remove the Kubernetes config file:
```bash
rm -rf ~/.kube/config
```
Get the new Kube config file:
```bash
aws eks update-kubeconfig --region us-east-2 --name vprofile-eks
```
Open the Terraform source code directory.

Run:
```bash
cat .github/workflows/terraform.yml
```
Copy the last line containing the HTTPS link.
Delete the Kubernetes resources:
```bash
kubectl delete -f [your_copied_link]
helm uninstall vprofile-stack/
```
Run the following command — it might give a warning about the Terraform version.
If it does, update the version in terraform.yml according to the warning.
terraform init -backend-config="bucket/vprofileaction5411"
Replace vprofileaction5411 with your own S3 Bucket name.
If you get a version error, fix it as mentioned above and run the command again.

After this, run:
```bash
terraform destroy
```
After typing terraform destroy, type yes when prompted.
Wait about 15-20 minutes for all resources to be fully removed.
