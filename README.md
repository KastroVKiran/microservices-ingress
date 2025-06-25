# TechSolutions Pro - Kubernetes Deployment

This project contains a multi-page web application deployed on AWS EKS with Nginx Ingress Controller using Jenkins CI/CD pipeline.

## Project Structure

```
.
├── index.html              # Home page
├── about.html              # About page
├── services.html           # Services page
├── contact.html            # Contact page
├── styles.css              # Stylesheet for all pages
├── Dockerfile              # Docker configuration
├── nginx.conf              # Nginx configuration
├── Jenkinsfile             # Jenkins CI/CD pipeline
├── k8s/
│   ├── deployment.yaml     # Kubernetes deployment and service
│   └── ingress.yaml        # Kubernetes ingress configuration
└── README.md               # This file
```

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

### Required Tools on Jenkins Agent

```bash
# Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

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

## Manual Deployment (Alternative)

If you prefer manual deployment:

```bash
# 1. Build and push Docker image
docker build -t kastrov/techsolutions-app:latest .
docker push kastrov/techsolutions-app:latest

# 2. Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name kastro-cluster

# 3. Deploy application
kubectl apply -f k8s/deployment.yaml

# 4. Install Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/deploy.yaml

# 5. Deploy Ingress
kubectl apply -f k8s/ingress.yaml

# 6. Get Load Balancer URL
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

## Monitoring and Troubleshooting

### Check Deployment Status
```bash
kubectl get pods -l app=techsolutions
kubectl get svc techsolutions-service
kubectl get ingress techsolutions-ingress
```

### View Logs
```bash
kubectl logs -l app=techsolutions
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Debug Ingress
```bash
kubectl describe ingress techsolutions-ingress
kubectl get events --sort-by=.metadata.creationTimestamp
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

### Change Domain
1. Update the `host` field in `k8s/ingress.yaml`
2. Configure DNS to point to the Load Balancer
3. Update SSL certificates if needed

### Environment Variables
Add environment variables to the deployment:

```yaml
env:
- name: ENVIRONMENT
  value: "production"
- name: LOG_LEVEL
  value: "info"
```

## Security Considerations

1. **SSL/TLS**: Configure cert-manager for automatic SSL certificates
2. **Network Policies**: Implement Kubernetes network policies
3. **RBAC**: Configure proper Role-Based Access Control
4. **Secrets**: Use Kubernetes secrets for sensitive data
5. **Image Security**: Scan Docker images for vulnerabilities

## Support

For issues and questions:
- Check the Jenkins build logs
- Review Kubernetes events: `kubectl get events`
- Verify EKS cluster status in AWS Console
- Check ingress controller logs: `kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx`