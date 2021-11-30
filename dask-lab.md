---
marp: true
paginate: true
---

<style>
img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
</style>

# Deploying Infrastructure for Dask on Google Cloud

## Sang-Yun Oh
Dept. of Statistics and Applied Probability
University of California Santa Barbara

---

# What is Dask?

_[Dask](https://docs.dask.org/en/latest/why.html) breaks up large computations and route parts of them efficiently onto distributed hardware. Dask is routinely run on thousand-machine clusters ..._

## About Dask

* [Description of Dask](https://youtu.be/nnndxbr_Xq4)
* [Example distributed applications](https://youtu.be/nnndxbr_Xq4?t=181)
* [Distributed infrastructure](https://youtu.be/nnndxbr_Xq4?t=339)
---

# Computing Infrastructure for Dask

* Dask run on laptops, clusters, and HPC environments
* Develop/test in small scale (laptops)
* Deploy/scale in large scale (clusters and HPC environments)

## Which Infrastructure to Use?

* **Cloud computing**: slow(er) but easy access and on-demand
* **[HPC](https://docs.nersc.gov/analytics/dask/)**: highly optimized but access is restricted and may not be interactive

---

# Cloud Computing and Reproducibility

What is reproducibility?

_An article about computational result is advertising, not scholarship. The actual scholarship is the **full software environment**, **code** and **data**, that produced the result._

---

# Cloud Platforms and Infrastructure as Code (IaC)

* Cloud platforms have APIs 
* :grinning: Setups of cloud infrastructure can be coded (IaC)
* :+1: Analysis code and cloud IaC on GitHub + Data &#8594; **Reproducibility**

#### Unfortunately, cloud infrastructure is mostly Do-It-Yourself (DIY) :sob: 

---

# Infrastructure as Code 

![center w:600px](https://cloud.google.com/recommender/docs/images/iac-architecture.svg)
([Using Recommendations for Infrastructure as Code](https://cloud.google.com/recommender/docs/tutorial-iac))

---

# Goal for Today

* Conceptual understanding of distributed computing infrastructure for Dask
* Discuss enabling technologies
* Hands-on lab using Google Cloud Platform 

---

# Dask Architecture

![center w:800px](https://user-images.githubusercontent.com/11656932/62263986-bbba2f00-b3e3-11e9-9b5c-8446ba4efcf9.png)

---

# Architecture Components

Client, scheduler, and worker processes can be distributed in different ways

* :hut: _Before 2006_: physical server run (a combination of) processes
* :house: _Before 2015_: multiple virtual machines on powerful machines
* :hotel: _After 2015_: Kubernetes cluster to handle orchestration

#### What is Kubernetes? :shrug:

---

# Kubernetes and Containers

_**[Kubernetes](https://kubernetes.io)** is an open-source system for automating deployment, scaling, and management of containerized applications._

_**Application containerization** (e.g. [Docker](https://www.docker.com/resources/what-container)) is an OS-level virtualization method used to deploy and run distributed applications without launching an entire virtual machine (VM) for each app._

---

<style scoped>
    th {
        color: white;
    }
    table, tr, th, td {
        background-color: white;
        border: 0px solid white;
    }
</style>

# Enabling Technologies

|  |  |  |  |
|:-----:|:-----:|:-----:|:-----:|
|![w:200px](https://brandslogos.com/wp-content/uploads/images/docker-logo-vector.svg)| Component contents | ![w:150px](https://helm.sh/img/helm.svg)| Components wiring diagram | 
|![w:330px](https://res.cloudinary.com/practicaldev/image/fetch/s--nBx6028r--/c_imagga_scale,f_auto,fl_progressive,h_500,q_auto,w_1000/https://www.linuxadictos.com/wp-content/uploads/kubernetes-logo.jpg)| Builder and manager |![w:200px](https://www.nginx.com/wp-content/uploads/2017/12/Google-Cloud-Logo-Main.svg)| Provides resources |


---

# Hands-on Lab

Google Cloud Command Line Tool: https://shell.cloud.google.com/

### Main tools
```bash
gcloud --version        # controls Google Cloud resources
docker --version        # container-level controls 
kubectl version         # cluster-level controls
helm version            # installation "blueprint"
```

---

# Login to Google Cloud



```bash
gcloud auth list                    # check current account
# gcloud auth login syoh@ucsb.edu   # login using another account

# set default project
gcloud config set project testing-sandbox-324502
gcloud config set compute/zone us-central1-a
```

---

# Start Kubernetes Cluster

```bash
# create Kubernetes cluster
gcloud container clusters create \
    --machine-type e2-standard-4 \
    --num-nodes 2 \
    [unique-cluster-name]
```
* [Machine type documentation](https://cloud.google.com/compute/docs/machine-types#recommendations_for_machine_types)
* [Regions and zones documentation](https://cloud.google.com/compute/docs/regions-zones#available)
* Use unique cluster names: e.g. include your initials

---

# Dask Cluster Helm Chart 

Three important concepts for Helm:

1.  _**Chart**_: a bundle of information necessary to create an instance of a Kubernetes application. (the blueprint)
2.  _**Config**_: configuration information that can be merged into a packaged chart. (user specified setting)
3.  _**Release**_: running instance of a _chart_, combined with _config_. (deployed instance)

#### Dask Helm Chart [:link:](https://github.com/dask/helm-chart)

Blueprint for setting up _Jupyter Lab_, _scheduler_, and _workers_ on Kubernetes cluster


---

# Dask Helm Chart Config

* User configuration supercedes default values in [`values.yaml`](https://github.com/dask/helm-chart/blob/main/dask/values.yaml) file

    ```yaml
    cat << EOF > config.yaml
    jupyter:
        serviceType: "LoadBalancer" # makes Jupyter notebook publicly accessible
    scheduler:
        serviceType: "LoadBalancer" # makes Dask scheduler publicly accessible
    worker:
        replicas: 4
    EOF
    ```

* Instantiate Dask Cluster  
    ```bash
    helm repo add dask https://helm.dask.org/
    helm repo update
    helm install --wait my-dask -f config.yaml dask/dask    # takes a while
    ```

---

# Kubernetes running `my-dask` release

```bash
sangoh@cloudshell:~ (testing-sandbox-324502)$ kubectl get all                                                                                                 
NAME                                     READY   STATUS    RESTARTS   AGE
pod/my-dask-jupyter-54ddbfdd9d-rbsjb     1/1     Running   0          8m45s
pod/my-dask-scheduler-7f4f94bb7d-4c4fn   1/1     Running   0          8m45s
pod/my-dask-worker-6877d8f79f-42jrb      1/1     Running   0          8m44s
pod/my-dask-worker-6877d8f79f-88wvm      1/1     Running   1          8m45s
pod/my-dask-worker-6877d8f79f-9qvc5      1/1     Running   0          8m44s
pod/my-dask-worker-6877d8f79f-g659d      1/1     Running   1          8m45s
pod/my-dask-worker-6877d8f79f-htg8p      1/1     Running   1          8m45s
pod/my-dask-worker-6877d8f79f-t44jn      1/1     Running   1          8m45s

NAME                        TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                       AGE
service/kubernetes          ClusterIP      10.12.0.1      <none>          443/TCP                       11m
service/my-dask-jupyter     LoadBalancer   10.12.8.243    34.121.167.99   80:32536/TCP                  8m45s
service/my-dask-scheduler   LoadBalancer   10.12.14.160   34.67.163.35    8786:30322/TCP,80:32005/TCP   8m45s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-dask-jupyter     1/1     1            1           8m45s
deployment.apps/my-dask-scheduler   1/1     1            1           8m45s
deployment.apps/my-dask-worker      6/6     6            6           8m45s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/my-dask-jupyter-54ddbfdd9d     1         1         1       8m46s
replicaset.apps/my-dask-scheduler-7f4f94bb7d   1         1         1       8m46s
replicaset.apps/my-dask-worker-6877d8f79f      6         6         6       8m46s
```
#### Google Cloud Kubernetes Workloads Overview [:link:](https://console.cloud.google.com/kubernetes/workload/overview?project=testing-sandbox-324502)

---

# Server addresses
```bash
export DASK_SCHEDULER=$(kubectl get svc --namespace default my-dask-scheduler -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export DASK_SCHEDULER_UI_IP=$(kubectl get svc --namespace default my-dask-scheduler -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export DASK_SCHEDULER_PORT=8786
export DASK_SCHEDULER_UI_PORT=80

export JUPYTER_NOTEBOOK_IP=$(kubectl get svc --namespace default my-dask-jupyter -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export JUPYTER_NOTEBOOK_PORT=80

echo tcp://$DASK_SCHEDULER:$DASK_SCHEDULER_PORT               -- Dask Client connection
echo http://$DASK_SCHEDULER_UI_IP:$DASK_SCHEDULER_UI_PORT     -- Dask dashboard
echo http://$JUPYTER_NOTEBOOK_IP:$JUPYTER_NOTEBOOK_PORT       -- Jupyter notebook
```

---

# Run Example Notebook

1. Open Jupyter notebook specified in output
2. Copy dashboard URL into Dask Jupyter lab extension
3. Open and run `examples/04-dask-array.ipynb`
4. Additional packages are needed for `examples/05-nyc-taxi.ipynb`

---

![bg images/dask-dashboard w:900px](dask-dashboard.png)

---

# Install Additional Packages

```yaml
cat << EOF > config.yaml
jupyter:
  serviceType: "LoadBalancer" # makes Jupyter notebook publicly accessible
  env:
    - name: EXTRA_PIP_PACKAGES
      value: "pyarrow gcsfs"

scheduler:
  serviceType: "LoadBalancer" # makes Dask scheduler publicly accessible

worker:
  replicas: 4
  env:
    - name: EXTRA_PIP_PACKAGES
      value: "pyarrow gcsfs"

EOF
```

---

# Upgrade `my-dask` Release

* `upgrade` release rather than `install`
    ```bash
    helm upgrade --wait my-dask -f config.yaml dask/dask    # takes a while
    ```
* Run `examples/05-nyc-taxi.ipynb` (will break). Check why with `kubectl`
    ```bash
    kubectl get all                                 # what do you notice?
    kubectl describe [pod/my-dask-worker-00000]     # evicted resource name
    ```

---

# Try again

```yaml
cat << EOF > config.yaml
jupyter:
  serviceType: "LoadBalancer" # makes Jupyter notebook publicly accessible
  resources:
    limits:
      cpu: 1
      memory: 3G
    requests:
      cpu: 0.5
      memory: 2G
  env:
    - name: EXTRA_PIP_PACKAGES
      value: "pyarrow gcsfs matplotlib"

scheduler:
  serviceType: "LoadBalancer" # makes Dask scheduler publicly accessible
  resources:
    limits:
      cpu: 1
      memory: 3G
    requests:
      cpu: 0.5
      memory: 2G

worker:
  replicas: 15
  resources:
    limits:
      cpu: 1
      memory: 3G
    requests:
      cpu: 0.5
      memory: 2.5G
  env:
    - name: EXTRA_PIP_PACKAGES
      value: "pyarrow gcsfs"

EOF
```

---

# Upgrade `my-dask` Release again

* Resize cluster to add nodes
    ```bash
    gcloud container clusters resize [your-cluster-name] --num-nodes 5
    ```
* `upgrade` release rather than `install`
    ```bash
    helm upgrade --wait my-dask -f config.yaml dask/dask    # takes a while
    ```
* Run `examples/05-nyc-taxi.ipynb`

<!-- johann.won@gmail.com, hyunjong526@gmail.com, ahnwj2004@gmail.com, lee.sungdong.087@gmail.com, applebox153@gmail.com  -->
