# tekton-example
This repo is use in [eazytraining's kubernetes course](https://eazytraining.fr/cours/kubernetes-les-bases-pour-devops) to demonstrate how to use [tekton](https://github.com/tektoncd/pipeline) to create kubernetes pipeline.
This code is based on the great [tekton tutorial](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md). So you can read it also for some usefull information.
Here we will only give you the command you need to run to setup a pipeline example.
## Install tekton

    kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

## Prerequisites
### Create secret to storage dockerhub secret

    kubectl create secret docker-registry regcred \
                    --docker-server="https://index.docker.io/v1/" \
                    --docker-username=dirane \
                    --docker-password="mypassword" \
                    --docker-email=diranetafen@yahoo.com
    
### Create service account that will used to pass secret to tekton

    kubectl apply -f https://raw.githubusercontent.com/eazytrainingfr/tekton-example/master/simple-webapp-docker-service.yml
    
### Create ClusterRole and ClusterRolebinding for the service account

    kubectl create clusterrole simple-webapp-docker-role \
               --verb=* \
               --resource=deployments,deployments.apps
    kubectl create clusterrolebinding simple-webapp-docker-binding \
             --clusterrole=simple-webapp-docker-role \
             --serviceaccount=default:simple-webapp-docker-service

## Pipeline Setup
    
### Create git and image resource

    kubectl apply -f code-resource.yml 
    kubectl apply -f image-resource.yml

### Create task and taskrun to automate image build

    kubectl apply -f task.yml
    kubectl apply -f taskrun.yml
    kubectl get tekton-pipelines
    kubectl describe taskrun build-docker-image-from-git-source-task-run

Verify that the taskrun is marked successed before going ahead.
You can check that the image is pushed on you dockerhub repo
  
### Create the complste pipeline that will build and deploy application
    kubectl apply -f deploy-using-kubectl.yml
    kubectl apply -f pipeline.yml
    kubectl apply -f pipelinerun.yml
    kubectl get tekton-pipelines
    kubectl describe pipelinerun.tekton.dev/simple-webapp-docker-pipeline-run-1

Verify that the application is deployed based on this [deployment file](https://github.com/eazytrainingfr/simple-webapp-docker/blob/master/kubernetes/deployment.yml) 
You can list pod : `kubectl get pod`and verify that a pod is newly created
You can also use `kubectl port-forward $(kubectl get po  -l app=simple-webapp-docker -o=name) 8080:8080 --address 0.0.0.0` to verify that application is well responding
If you face any issue, please let us know at eazytrainingfr@gmail.com
