create new repo on ecr 
create kubernetes cluster 


go on github Repo , create folder .github/workflows
add deployment.yml file and paste the content below 

``` yml
name: Deploy to ECR
 
on: 
  push:
    branches: [ main ]
 
env:
  ECR_REPOSITORY: ecr_container (name the ecr image)
  EKS_CLUSTER_NAME: my-cluster
  AWS_REGION: ap-southeast-1
 
jobs:
  build:
    name: Deployment
    runs-on: ubuntu-latest
 
    steps:
 
    - name: Set short commit SHA
      id: commit
      uses: prompt/actions-commit-hash@v2
 
    - name: Check out code
      uses: actions/checkout@v2
 
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with: 
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
        aws-region: ${{ env.AWS_REGION }}
 
    - name: Set up JDK 14
      uses: actions/setup-java@v1
      with:
        java-version: 14
 
    - name: Build project with Maven
      run: mvn -B package --file pom.xml
 
    - name: Login to amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
 
    - name: Build, tag and Push
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ${{ env.ECR_REPOSITORY }}
        IMAGE_TAG: "latest"
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        echo "Pushing image to ECR..."
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
 
    - name: Update kube config
      run: aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION
 
    - name: Deploy to EKS
      env: 
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ steps.commit.outputs.short }}
      run: |
        kubectl rollout restart deployment/my-app1
        kubectl apply -f deployment.yml
        kubectl apply -f service.yml
```
come on terminal 
vim depoloyment.yml

``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app1
  namespace: default
  labels:
    app: my-app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app1
  template:
    metadata:
      labels:
        app: my-app1
    spec:
      containers:
      - name: my-app1
        image: 288761750357.dkr.ecr.ap-southeast-1.amazonaws.com/mansi-30:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

vim service.yml

``` yml
apiVersion: v1
kind: Service
metadata:
  name: regapp-service1
  labels:
    app: my-app1
spec:
  selector:
    app: my-app1
 
  ports:
    - port: 8080
      targetPort: 8080
 
  type: LoadBalancer
```
run the commands 
Click on Actions on git hub
kubectl get svc (paste the external ip on the browser
delete the deployment and service file from the terminala and run.

