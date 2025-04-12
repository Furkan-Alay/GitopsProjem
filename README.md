# Projeme Hoşgeldiniz Aşağıdaki adımları takip ederek projeyi localinizde çalıştırabilirsiniz İyi çalışmalar.
# "https://github.com/Furkan-Alay/GitopsProjem" ve "https://github.com/Furkan-Alay/GitappProjem" buradaki resource kısmına gidip ayrı ayrı her ikisi için SSH linkinden fork işlemini yapmanız gerekmektedir.
# Daha sonra terminalinizi açıp SSH key oluşturmanız gerekmektedir. buradaki kodları sırayla terminale yazınız. 
* cd
* cd ~/.ssh
* ssh-keygen =>>> Burada private ve public ssh key oluşacaktır.
* cat [sshkeyadınız] =>>> Burada içeriği kopyalayın.
# Daha sonra oluşturduğunuz public Key bilgilerinizi Kendi Github SSH Key kısmına kaydedin. (Github hesabınıza girin >>>> Settings >>>>> SSH and GPG keys >>> New SSH key >>> Gerekli bilgileri girin ve Add basın) 
# Terminalinizde Git yükle değilse git paketini yükleyiniz.
# Daha sonra Terminalinizi açın
* mkdir ~/Desktop/[Klasör adınız]
* cd ~/Desktop/[Klasör adınız]
* Oluşturduğunuz fork kaynak kodlarına girin ve SSH linklerini kopyalayın.
* git clone [SSH Klone linki(Terraform kaynak kodu)] 
* git clone [SSH Klone linki(Uygulama kaynak kodu)]
## Bu projede Terraform kaynak kodu için "iac-vprofile" fork ve Uygulama kodu için "vprofile-action" isimleri kullandığımızı varsayacağım başka bir isim kullandıysanız değiştirin.
* ls
* cd iac-vprofile
* git config core.sshCommand "ssh -i ~/.ssh/gitaction -F /dev/null"
* cd ../vprofile-action/
* git config core.sshCommand "ssh -i ~/.ssh/gitaction -F /dev/null"
## Burada kendi Github hesap adınızı ve mailinizi giriniz.
* git config --global user.name devops541
* git config --global user.email furkanalay428@gmail.com
* cd ..
### Yeni bir klasör kopyaladık istediğiniz ismi burada verebilirsiniz.
* cp -r iac-vprofile/ main-iac
* cd iac-vprofile
* git checkout stage
### Şimdi ise Github Secrets adımlarımızı oluşturalım. Burada AWS ortamında oluşturduğumuz AWS Access Key,Secret Key,S3 Bucket ve ECR Repository bilgilerini Github Secret kısmına kaydedeceğiz.
* Terraform ve Uygulama kodlarını fork etmiştik. Bu kaynak kodlarına giriyoruz ve "Secret and variables" kısmındaki "Actions" kısımlarına basıyoruz.
* AWS hesabımıza giriyoruz ve projenin sonuna kadar kullanacağımız "Region" seçiyoruz.
* "IAM" servisini açıp Create User kısmına basıyoruz. Bu kullanıcının "Administrator Access" yetkisine sahip olması gerekiyor çünkü AWS hesabında kaynak kullanacaktır.
* IAM User kullanıcısı oluşturuyoruz. Bu kullanıcı için "Create Access Key" kısmına basıp "Access Key" ve "Secret Key" oluşturuyoruz.
* Access Key ve Secret Key bilgilerini kopyalıyoruz.
* Terraform kaynak kodumuzun ve Uygulama kısmına Github Secret kısmına bu bilgileri kaydediyoruz.
* AWS Hesabımızda "S3 Bucket" oluşturuyoruz ve "S3 Bucket Name" kopyalıyoruz.
* S3 Bucket Name kısmını Terraform kaynak kodumuzun içerisinde olduğu Github Secret kısmına kaydediyoruz.Buraya kaydetmemizin sebebi S3 Bucket servisini Terraform içerisinde kullanacağız  
* AWS Hesabımızda ECR(Elastic Container Registry) içerisinde private bir repository oluşturuyoruz ve ECR URL'mizi kopyalıyoruz.
* ECR URL'mizi Uygulama kaynak kodumuzun içerisinde olduğu Github Secret kısmına kaydediyoruz.
### Şimdi ise Terraform kodlarımızın içerisine Girip VPC infrastructure ve EKS Cluster oluşturalım.
* Terraform kaynak kodumuzun "iac-vprofile" klasörünü açıyoruz. VSCode uygulamasıyla açmamız gerekiyor.
* terraform klasörü içerisindeki "variables.tf" dosyamızı açıyoruz. AWS hesabımızdaki region bilgisini ve terraform ile oluşturacağımız EKS Cluster adını "default" kısmında yazıyoruz.
* Aynı şekilde terraform klasörü içerisindeki "terraform.tf" dosyamızı açıyoruz. "backend "s3"" içerisindeki "bucket" kısmına AWS Hesabımızda oluşturduğumuz S3 bucket name kısmını giriyoruz. AWS region bilgilerimizi de "region" kısmına giriyoruz. "key" kısmına dokunmuyoruz.
* "vpc.tf" dosyamızı açıyoruz. "name" değerine EKS Cluster için belirlediğimiz ismi giriyoruz.
* VSCode Source Control paneline gelip Projemizi "Commit & Push" seçeneğiyle Github'a push etmeliyiz.
* Çıkan ekranda Commit mesajını yazıp "Save" seçeneğine basacağız.
### Şimdi ise Terraform kodlarımızı Github Actions kısmında çalıştıralım.
* Terraform kaynak kodumuzun "iac-vprofie" klasörünü VScode ile açtık.
* ".github/workflows" adında bir klasör oluşturduk.Bu klasör içinde "terraform.yml" dosyası oluşturuyoruz.Bu dosya ile github actions içerisinde terraform kodlarımızı çalıştırmış olacağız.
* Aşağıdaki kodları "terraform.yml" dosyamıza yapıştıralım
* name: "Vprofile IAC"  
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
 ##### Credentials for deployment to AWS
 AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
 AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
 ##### S3 bucket for the Terraform state
 BUCKET_TF_STATE: ${{ secrets.BUCKET_TF_STATE}}
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
* Yukarıda yapıştırdığım kod içerisindeki "AWS_REGION" kısmına S3 Bucket ve ECR bölgemizi ekliyoruz.
* VSCode Source Control kısmından Commit&Push seçeneğine tıklıyoruz ve Commit mesajını yazdıktan sonra Save basıyoruz.
* Github Actions kısmında Pipeline adımlarının gerçekleştiğini görmüş olacağız. 
### Şimdi ise Terraform kodlarımızla VPC Altyapısını ve EKS Cluster yapısını oluşturalım
* terraform.yml dosyanızı açıp bu içeriği ekleyin:
* - name: Terraform Apply
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

* Buradaki komutları VPC ve EKS Cluster oluşturacak olan son komutlarımızdır. Bu komutları bir önceki komutlarla birleştirmelisiniz.
* terraform/ klasörümüze bir içerik ekledik ve Commit&Push işlemi yaptık.
* Github Actions kısmında Workflow başlayacaktır ancak son eklediğimiz komutlar çalışmayacaktır.Çünkü "main" stage içerisinde push işlemi yapmadık.Projenin sonunda push işlemini gerçekleştireceğiz.
* Terraform ve Uygulama kodumuzun olduğu klasörü terminalle açıyoruz.Aşağıdaki komutlarla merge işlemini gerçekleştirmiş olacağız:
* cd iac-vprofile/
* git checkout stage
* git pull
* git checkout main
* git add .
* git commit -m "Bu bir Duzeltme mesajıdır"
* git merge stage
### Şimdi ise Uygulamamızın Workflow kısmına giriş yapalım. Burada Sonar Cloud oluşturacağız,Sonar token oluşturacağız ve Github Secret ayarlarını yapacağız.
* https://sonarcloud.io/ sitesinden Github hesabımızdan bir sonarcloud hesabı oluşturuyoruz.
* "Create new organization" basarak manuel olarak organizasyon oluşturmamız gerekiyor. "Free Plan" seçmek bizim için yeterli olacaktır. "Analyze a new project" basıyoruz."Display name" ve "Project Key" bilgilerini aynı giriyoruz. "Public" visibility seçiyoruz.
* Sağ yukarıdan "My account" basıyoruz."Security" basıyoruz. Burada bir tane token oluşturuyoruz.Token oluşturduktan sonra çıkan şifreyi kopyalıyoruz.Github Secret kısmında yeni bir Secret oluşturuyoruz."Name" kısmına "SONAR_TOKEN" yazdıktan sonra Token şifremizi secret kısmına giriyoruz ve add basıyoruz.
* Oluşturduğumuz Organizasyon ismini Secret kısmına giriyoruz ve name olarak "SONAR_ORGANIZATION" giriyoruz.
* Yeni bir Secret açıyoruz ve name kısmına "SONAR_PROJECT_KEY" giriyoruz secret kısmına ise "Project_Key" bilgisimizi giriyoruz.
* Son olarak yeni bir Github Secret oluşturuyoruz.Name kısmına "SONAR_URL" giriyoruz Secret kısmına ise "https://sonarcloud.io" giriyoruz. 
* Fork ettiğimizi uygulama kaynak kodumuzu VScode ile açıyoruz.
* ".github/workflows" adında bir klasör oluşturuyoruz ve bu klasörün içerisine "main.yml" dosyası oluşturuyoruz.Bu dosyamızın içerisin aşağıdaki kodları yazıyoruz:
* name: vprofile actions
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

      # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
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
* VScode Source Control kısmından "Commit&Push" basıyoruz.
* Uygulamamızın kaynak koduna geliyoruz.Actions kısmına geliyoruz."vprofile-actions" seçeneğine basıyoruz. "Run workflow" basıyoruz.İlk başta workflow hata verecektir.Tekrardan çalıştırıyoruz ve Test işlemi bitti.
