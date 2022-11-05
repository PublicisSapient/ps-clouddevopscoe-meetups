# Try things out

## Pre-requisites
1. Kubernetes cluster should be provisioned locally via **Rancher Desktop**, **Minikube**, **Docker Desktop** and kubectl installed and configured to connect with cluster.
2. Cloud provider credentials should be available, to create follow below links.
    - [Microsoft Azure](https://crossplane.io/docs/v1.10/cloud-providers/azure/azure-provider.html)
    - [Amazon Web Services](https://crossplane.io/docs/v1.10/cloud-providers/aws/aws-provider.html)
    - [Google Cloud Platform](https://crossplane.io/docs/v1.10/cloud-providers/gcp/gcp-provider.html)
3. Helm should be installed

## Crossplane setup<a>x-setup</a>
```
kubectl create namespace crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```
## Demo 1 - providers setup
- Goto, [providers](https://github.com/PublicisSapient/ps-clouddevopscoe-meetups/blob/main/webinars/Infrastructure%20Automation%20-%20Past%2C%20Present%20and%20Future/providers) folder.
- Generate base64 encoded value of aws, gcp and azure credentials and mention in credentials.yaml files available in **aws**, **gcp** & **azure** folders, for example.
```
data:
  credentials: "base64 encoded value of credentials"
```
- Specify **project id** of Google Cloud in **gcp/providerconfig.yaml** file, for example.
```
spec:
  projectID: "your google cloud project id"
```
- Install listed below manifests in your k8s cluster, these files are available in **aws**, **gcp** and **azure** folders.
    - credentials.yaml
    - provider.yaml
    - providerconfig.yaml
- **providerconfig.yaml** must be applied after **provider.yaml**, with cooldown period of 10 - 20 sec, to let provider setup complete.
- To validate, run below command.
```
kubeclt get deployment -n crossplane-system
```
- Should get below response.
```
deployment.apps/crossplane-rbac-manager     1/1     1            1           3m3s
deployment.apps/crossplane                  1/1     1            1           3m3s
deployment.apps/provider-aws-XXXXXX         1/1     1            1           3m3s
deployment.apps/provider-gcp-XXXXXX         1/1     1            1           3m3s
deployment.apps/provider-azure-XXXXXX       1/1     1            1           3m3s
```

## Demo 2 - drift detection & reconciliation
- Goto, [drift](https://github.com/PublicisSapient/ps-clouddevopscoe-meetups/blob/feature-crossplane-demo/webinars/Infrastructure%20Automation%20-%20Past%2C%20Present%20and%20Future/drift) folder.
- Apply manifest **aws-firewall.yaml**, as per below
```
kubectl apply -f aws-firewall.yaml
```
- It will create one **security group** named as **x-drift-demo-sg** in **North Virginia** region.
- To validate drift detection and reconciliation, follow below
    - Login to AWS cloud web console
    - Look for security group created
    - Try to add/update/remove ingress or egress rules, wait for sometime, check if your changes are reverted to desired state of firewall rules as per **aws-firewall.yaml** file.

## Demo 3 - chicken-egg
### Description
- Demonstrate to first create k8s cluster on your local system via Rancher Desktop, Minikube, Docker Desktop.
- Setup Crossplane on your local k8s cluster.
- Provision GKE cluster via Crossplane.
- Configure and use GKE cluster to manage its own and running application workloads.

### Instructions
- Goto, [chicken-egg](https://github.com/PublicisSapient/ps-clouddevopscoe-meetups/blob/feature-crossplane-demo/webinars/Infrastructure%20Automation%20-%20Past%2C%20Present%20and%20Future/chicken-egg) folder.
- Apply manifest **gke.yaml**, to create gke cluster in **us-east1** region.
- Configure kubectl to connect with gke cluster, run below command, to get kubeconfig settings.
```
gcloud container clusters get-credentials \
   x-chicken-egg-demo --region us-east1
```
- After kubectl connect with gke cluster, apply below manifests, resources are created in x-chicken-egg-ns namespace.
    - app.yaml - create namespace and deploy an nginx image
    - svc.yaml - create load balancer
- Access latest deployed nginx, to get load balancer public ip, run below command.
```
kubectl get svc x-chicken-egg-svc -n x-chicken-egg-ns
```
- Hit load balancer public ip endpoint, should be able to get default nginx web page.
- Now [setup Crossplane](#x-setup) on gke cluster.
- After Crossplane setup, apply manifest **gke.yaml**.
- Your gke cluster should be running in AS-IS state.
- Try to access nginx web page again, should be working.
- Now reset the locally running kubernetes cluster, to manage gke cluster from its own.
- Your gke cluster should be running in AS-IS state.
- Try to access nginx web page again, should be working.
- Change replica count in app.yaml file and apply, check if replica count is changing and able to access nginx web page.
- Change **maxNodeCount** in gke.yaml, from 10 to 5, apply manifest, check gke web console if that is changed in nodepool autoscaling configs.



