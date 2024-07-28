# Ku-ber-n-e-t-s-k8s

Kubernetes Installation Using KOPS on EC2

Certainly! Here’s a step-by-step guide starting from the creation of an AWS EC2 instance to setting up and exposing Argo CD:

1. Create an EC2 Instance
Log in to AWS Console:

Go to the AWS Management Console.
Launch an EC2 Instance:

Navigate to the EC2 Dashboard and click “Launch Instance.”
.
Choose an instance type, e.g., t2.medium.


Configure security group:
Allow SSH (port 22) from your IP.
Allow HTTP (port 80) and HTTPS (port 443) from anywhere.
Allow custom TCP (port 8080) for Argo CD access.



2. Install Kubernetes and kops
Install Kubernetes CLI (kubectl):

///cmd///



curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl


Install kops:
///cmd///



curl -Lo kops https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/

configure the AWS-cli 
and map ur credentials access key and secret key. also attach the required roles to your EC2 instance.

3. Set Up Kubernetes Cluster with kops
Create an S3 bucket for kops state:

////cmd///


aws s3api create-bucket --bucket <your-kops-state-store> --region <your-region> --create-bucket-configuration LocationConstraint=<your-region>

Export environment variables:
////cmd///


export KOPS_STATE_STORE=s3://<your-kops-state-store>
export NAME=<your-cluster-name>.k8s.local



Create a Kubernetes cluster:
Create the cluster

////cmd///



kops create cluster --zones=<your-aws-zone> --name=${NAME}
kops update cluster ${NAME} --yes
kops validate cluster --wait 10m


4. Install Argo CD
Create Argo CD namespace:

////cmd///

kubectl create namespace argocd

Install Argo CD:

////cmd///

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


5. Expose Argo CD Server
Create a LoadBalancer service for Argo CD server:

Create a file argocd-server-service.yaml with the following content:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: argocd-server
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-server
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  - port: 443
    targetPort: 8080
    protocol: TCP
    name: https
  selector:
    app.kubernetes.io/name: argocd-server
	
	
Apply the service definition:

////cmd///
kubectl apply -f argocd-server-service.yaml

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'


Retrieve the External IP:

////cmd///
kubectl get svc -n argocd

Look for the EXTERNAL-IP under the argocd-server service.

6. Access Argo CD
Get Initial Admin Password:

////cmd///
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

Log in to Argo CD:

Open your browser and go to http://<external-ip>.
Log in with the username admin and the password you retrieved.

7. Secure Argo CD
Change the Admin Password:

Navigate to Settings in the Argo CD UI and update the admin password.
Enable HTTPS:

Follow the official documentation to set up HTTPS for secure access.
By following these steps, you will have Argo CD set up and exposed through AWS EC2. Let me know if you need any more help!




kops delete cluster --name=myfirstlocalcluster.k8s.local --yes
aws s3 rb s3://my-store-unique1234 --force

export KOPS_STATE_STORE=s3://my-store-unique1234




My Practice..///

aws s3api create-bucket --bucket my-store-unique1234 --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1


export KOPS_STATE_STORE=s3://my-store-unique1234
export NAME=myfirstlocalcluster.k8s.local



kops create cluster --zones=ap-south-1b --name=${NAME}
kops update cluster ${NAME} --yes
kops validate cluster --wait 10m
