# Provision a GKE Cluster using Terraform and install OpenSearch using opensearch-operator

## Follow below given steps to setup the GKE cluster using terraform and then configure OpenSearch using opensearch-operator.

### Pre-requisites

1. Install `gcloud-cli` and required auth plugin `google-cloud-sdk-gke-gcloud-auth-plugin`.

2. Enable Compute and Container APIs on google cloud.

3. Install Terraform

### Clone the Github Repo

```bash

    git clone https://github.com/misraharshit/Terraform-GKE.git

    cd Terraform-GKE

```

### Setup Authentication with google cloud, follow mentioned commands.

```bash

    gcloud auth login --cred-file=$SecretFile.json # Will attach in the email [contact: harshitmishra1002@gmail.com], can't share on github repo due to some policy.
    gcloud config set project graceful-disk-445715-j1

```

### Modify vpc.tf with secretfile key

```bash

    provider "google" {
    project = var.project_id
    region  = var.region
    credentials = file("~/graceful-disk-445715-j1-2c2e04783380.json")
    }

```

### Initialise and apply the changes on google cloud using terraform.

```bash

    terraform init
    terraform plan
    terraform apply --auto-approve

```

### Get GKE cluster credentials, run command.

```bash

    gcloud container clusters get-credentials graceful-disk-445715-j1-gke --region us-central1 --project graceful-disk-445715-j1

```

### Add opensearch-operator helm repo and install operator:

```bash

    helm repo add opensearch-operator https://opensearch-project.github.io/opensearch-k8s-operator/
    helm repo update
    helm install os opensearch-operator/opensearch-operator --version 2.7.0

```

[!CAUTION]
> **To Resolve error**: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

### Set `vm.max_map_count` kernel parameter on each host using a daemonset.

```bash

    kubectl create -f ds.yaml

```

### Install opensearch using opensearch-operator

```bash

    kubectl create -f opensearch-cluster.yaml

```

### Once the resources are deployed, we can verify the pods and services are running and validate the opensearch cluster is working via adding some index.

```bash

    kubectl get po,svc

    kubectl port-forward svc/opensearch 9200:9200 &

    curl -XPOST -k --user 'admin:admin' "https://localhost:9200/demo/_doc?pretty" -H 'Content-Type: application/json' -d'
    {
        "Name": "Harshit Mishra",
        "Repo": "Terraform-GKE",
        "Desc": "Deploy GKE using terraform"
    }
    '

    curl -XGET -k --user 'admin:admin' "https://localhost:9200/_cat/indices?v&s=index&pretty" # Command to list all the present indexes

    curl -XGET -k --user 'admin:admin' "https://localhost:9200/demo/_search?pretty" # List the documents in the demo index

```


### To perform the cleanup destroy the cluster.

```bash

    terraform destroy --auto-approve

```