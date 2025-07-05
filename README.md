# Learn With KASTRO - Kubernetes Deployment

This project contains a multi-page web application deployed on AWS EKS with Nginx Ingress Controller using Jenkins CI/CD pipeline.

## Application Paths

The application provides the following paths:

- **Home Page**: `/` - Landing page with overview
- **About Page**: `/about` - Company information and team
- **Services Page**: `/services` - Detailed service offerings
- **Contact Page**: `/contact` - Contact information and form

## Prerequisites

### Jenkins Configuration

1. **DockerHub Credentials** (`dockerhub-creds`):
   - Username: Your DockerHub username
   - Password: Your DockerHub password/token

2. **AWS Credentials** (`aws-creds`):
   - Access Key ID: Your AWS access key
   - Secret Access Key: Your AWS secret key

### AWS EKS Cluster

- **Cluster Name**: `kastro-cluster`
- **Region**: `us-east-1`
- **Node Group**: Ensure you have at least 2-3 nodes for HA

## Deployment Instructions

### Step 1: Prepare Your Environment

1. **Create EKS Cluster** (if not already created):
```bash
eksctl create cluster \
  --name kastro-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

2. **Configure Jenkins Credentials**:
   - Go to Jenkins → Manage Jenkins → Manage Credentials
   - Add DockerHub credentials with ID: `dockerhub-creds`
   - Add AWS credentials with ID: `aws-creds`

### Step 2: Create Jenkins Pipeline

1. **Create New Pipeline Job**:
   - Go to Jenkins → New Item → Pipeline
   - Name: `TechSolutions-EKS-Deployment`

2. **Configure Pipeline**:
   - Pipeline → Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: Your Git repository URL
   - Script Path: `Jenkinsfile`

### Step 3: Run the Pipeline

1. **Trigger the Pipeline**:
   - Click "Build Now" on your pipeline job
   - Monitor the build logs for progress

2. **Pipeline Stages**:
   - ✅ Checkout source code
   - ✅ Build Docker image
   - ✅ Push to DockerHub
   - ✅ Configure AWS and kubectl
   - ✅ Deploy to Kubernetes
   - ✅ Install Nginx Ingress Controller
   - ✅ Deploy Ingress
   - ✅ Get Ingress URL

### Step 4: Access Your Application

After successful deployment, you'll see output like:

```
=========================================
DEPLOYMENT SUCCESSFUL!
=========================================
Application URL: http://your-load-balancer-url.us-east-1.elb.amazonaws.com

Available Paths:
- Home Page: http://your-load-balancer-url.us-east-1.elb.amazonaws.com/
- About Page: http://your-load-balancer-url.us-east-1.elb.amazonaws.com/about
- Services Page: http://your-load-balancer-url.us-east-1.elb.amazonaws.com/services
- Contact Page: http://your-load-balancer-url.us-east-1.elb.amazonaws.com/contact
=========================================
```


## Scaling the Application

```bash
# Scale up replicas
kubectl scale deployment techsolutions-deployment --replicas=5

# Scale down replicas
kubectl scale deployment techsolutions-deployment --replicas=2
```

## Clean Up

To remove the deployment:

```bash
kubectl delete -f k8s/ingress.yaml
kubectl delete -f k8s/deployment.yaml
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/deploy.yaml
```

## Customization

### Update Application
1. Modify HTML/CSS files
2. Commit changes to your repository
3. Run the Jenkins pipeline again

## Kastro Kiran V
