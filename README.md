## Pipeline-dso

The goal of this repository is to integrate security within a DevOps pipeline (here, Tekton).

A conventional CI/CD pipeline mostly has tasks like fetch, build, test, deploy. We need to integrate security into the pipeline. This could be further extended to the pipeline being failed if the security check is not fulfilled and right set of people could be notified.



## DevSecOps Pipeline
The DevSecOps Pipeline consists of the following task which are given below:


* [clone-git-repo](https://github.com/tektoncd/catalog/blob/main/task/git-clone/0.3/git-clone.yaml)
Git clone repo task clones given repository url into the workspace.


* [secret-detection](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/GITLEAKS/gitleaks.yaml)
Secret detection task is responsible for detecting secrets like api keys, passwords present in the repository. Read more about the task [here](https://github.com/tektoncd/catalog/tree/main/task/gitleaks/0.1)

* [scan-repo](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/SCAN_REPO/scan-repo.yaml)
Scan-repo task is responsible for scanning the given repository for CVEs (Common Vulnerabilities and Exposures) and creates a vulnerability report. Working of Trivy can be better understood [here](https://rastogee-ayushi.medium.com/trivy-keep-your-artifacts-vulnerability-free-6dce292134e5).


### CRDA
* [setup-env](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/CRDA/setup-env.yaml)
Setup-env task downloads all the dependencies used by the project depending on the ecosystem so Redhat CodeReady Dependency Analytics can perform the analysis and check for vulnerabilities.

* [crda-task](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/CRDA/crda-task.yaml)
CRDA task uses RedHat CodeReady Dependency Tool to scan and check if any vulnerabilities lie in the dependencies used by the project and generates a whole report.

* [check-vulnerabilities](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/CRDA/setup-env.yaml)
Check vulnerabilities task reads the report generated in CRDA task and checks if all the vulnerabilities are within limits or not. It breaks the build if the vulnerabilities are not within the limits.

You can read more about the need, prerequistes and working of these three tasks please go through [CRDA](https://github.com/tektoncd/catalog/blob/edb15d9f4334822f319393ad271bf11106eca926/task/redhat-codeready-dependency-analysis/0.3/README.md).


* [build-push](https://github.com/urvashigupta7/Go-Server/blob/master/ci/task/build-push.yaml)
Build push task is responsible for building the image from docker file and pushing it to the docker registry.



* [check-dep-version](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/KUBERNETES_MANIFEST_CHECKS/check-dep-version.yaml)
Check dep version task is responsible for finding out the deprecated or removed apiVersions in kubernetes manifest files. Read more about the task [here](https://github.com/urvashigupta7/kubernetes_manifest_checks/tree/master/tasks/pluto).


* [lint-yaml](https://github.com/ayushi-24git/pipeline-dso/blob/main/Tasks/KUBERNETES_MANIFEST_CHECKS/lint-yaml.yaml)
Lint yaml task checks for misconfigurations in kubernetes objects by scanning the kubernetes manifest files. Read more about the task [here](https://github.com/urvashigupta7/kubernetes_manifest_checks/tree/master/tasks/kube_linter).



## Secrets
* For build and push task docker credentials are required (to push the image). build-push task uses the `docker-creds` secret to get `USERNAME` and `PASSWORD`. Use the below command to create `docker-creds` secret; replace `yourusername` and `yourpassword` to your own.

```
kubectl create secret generic docker-creds \
      --from-literal=USERNAME=yourusername\
      --from-literal=PASSWORD=yourpassword
```

* Create secrets for crda key and crda token

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: crda
type: Opaque
stringData:
  crda-key: your_crda_key
---
apiVersion: v1
kind: Secret
metadata:
  name: token
type: Opaque
stringData:
  auth-token: your_crda_token

```


## Workspaces
Workspace is used as a common filesystem between tasks and used for inputs and outputs of task, in workspace dependencies are installed for further usage while running the task.
* **source** : A Workspace containing your source directory.


## Setting up cluster
Set up a cluster using minikube by doing a `minikube start`.

## Setting up Tekton
Install tekton with the following command after setting up the cluster

`kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml`

This will install all the necessary Tekton components to get started.

## Applying the Tasks and Pipeline yamls
Apply all the mentioned tasks.

`kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.3/git-clone.yaml`

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/main/Tasks/GITLEAKS`

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/main/Tasks/SCAN_REPO`

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/main/Tasks/CRDA`

`kubectl apply -f https://raw.githubusercontent.com/urvashigupta7/Go-Server/master/ci/task/build-push.yaml`

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/main/Tasks/KUBERNETES_MANIFEST_CHECKS`

Apply the pipeline  and pipelinerun yamls as:

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/pipeline.yaml`

`kubectl apply -f https://raw.githubusercontent.com/ayushi-24git/pipeline-dso/main/pipelinerun.yaml`


## Pipeline parameters
The following parameters will be asked henceforth.
* **git-repository-url** : git repository to be scanned.
* **k8-manifest-dir** : path to kubernetes mainifest directory or file.
* **image-name** : Name of the image which will be built.
* **language** : String value for each language; golang, npm, python, java
* **manifest-file-path** : Path of the manifest file to on which crda analysis will be performed.
* **pkg-installation-directory-path** : Path of a directory in workspace, where dependencies will be installed.
* **report-file-path** : Path of the file to crda save analysis report.
* **crda-image** : Image of the specific language which is used by CRDA to perform analysis.
* **total-vulnerabilities** : Maximum number of vulnerabilities allowed for crda.
* **critical-vulnerabilities** : Maximum number of critical vulnerabilities allowed for crda.
* **high-vulnerabilities** : Maximum number of high vulnerabilities allowed for crda.
* **medium-vulnerabilities** : Maximum number of medium vulnerabilities allowed for crda.
* **low-vulnerabilities** : Maximum number of low vulnerabilities allowed for crda.
* **output_format** : Report output format
* **repo-scan** : The repository on which you want to perform security checks.

## Check logs
Now, the pipeline has successfully started. You can check the logs using the following command:

`tkn pipelinerun logs <name-of-the-pipelinerun>`.

You can see the table of all vulnerabilities (if any) detected by Trivy.
