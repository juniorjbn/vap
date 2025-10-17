## Validating Admission Policy Examples LAB

This lab is crafted to inspire the integration of validating admission policies in your environments, promoting the adoption of best practices. Embrace these strategies to fortify the security and reliability of your systems, and stay vigilant for upcoming vulnerabilities and misconfigurations with the assistance of [Zora](https://undistro.io/zora-dashboard/) & [Marvin](https://undistro.io/marvin)! Happy Kuberneting!

# Step by step *([oh baby!](https://www.youtube.com/watch?v=GDEDQ8TCzW8))*

## 1. Create KinD Cluster first.
For this lab we'll use an *KinD 1.33.4v Cluster * .
```kind create cluster --config  assets/kind_config.yaml```

[Matheus Faria](https://github.com/matheusfm) wrote an awesome post about this [here](https://getup.io/en/blog/standardizing-enforcement-of-security-policies#getting-started-with-validating-admission-policies). 

## 2. Deploy some apps for demo *(optional)*

#Google - Demo Store

`git clone https://github.com/GoogleCloudPlatform/microservices-demo`   
`cd microservices-demo/`
`kubectl apply -f ./release/kubernetes-manifests.yaml`

#Buoyant - Emojivoto

`kubectl apply -k github.com/BuoyantIO/emojivoto/kustomize/deployment`

#PodInfo

`helm repo add podinfo https://stefanprodan.github.io/podinfo`
`kubectl create ns test`
`helm upgrade --install --wait frontend --namespace test --set replicaCount=2 --set backend=http://backend-podinfo:9898/echo podinfo/podinfo`
`helm test frontend --namespace test`
`helm upgrade --install --wait backend --namespace test --set redis.enabled=true podinfo/podinfo`


## 3. [Deploy Zora with Marvin](https://zora-docs.undistro.io/latest/dashboard/#getting-started) to have visibility of your cluster

    helm plugin install https://github.com/undistro/helm-zoraauth
    helm zoraauth --audience="zora_prod" \
      --client-id="e0eNfHFrvyHYMm6OfzMDDQuLNhTeYbjA" \
      --domain="login.undistro.io"
    helm repo add undistro https://charts.undistro.io --force-update
    helm repo update undistro
    helm upgrade --install zora undistro/zora \
      -n zora-system --create-namespace --wait \
      --set clusterName="$(kubectl config current-context)" \
      --set saas.workspaceID='YourWorkspaceID' \
      --values tokens.yaml

  
 

## Validating Admission Policy - Examples
#***What is Validating Admission Policy?*** >>> https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/#getting-started-with-validating-admission-policy


#How Validating Admission Policy works inside Kubernetes API.

![VAP in K8s](assets/vap.avif)

*#Example 01 - Maximum replicas*

 1. Create Namespace (`kubectl apply -f namespace_replicas.yml`)
 2. Create Policy (`kubectl apply -f policy_replicas.yml`)
 3. Binding Policy (`kubectl apply -f policy_replicas_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_replicas.yml`)

Comments: This is the documentation example, and the deploy will fail because it's configured to have 6 replicas. Update line 9 in the `nginx_deploy_replicas.yml` and apply it again to see it working.

___

*#Example 02 - Prevent containers with images outside defined registry*

 1. Create Namespace (`kubectl apply -f namespace_registry.yml`)
 2. Create Policy (`kubectl apply -f policy_registry.yml`)
 3. Binding Policy (`kubectl apply -f policy_registry_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_registry.yml`)

Comments: The registry is defined in the line 14 of the `policy_registry.yml`, here we are using the aws ecr to get the nginx image, if you want to see it failing just update the line 18 in the `nginx_deploy_registry.yml` to another registry, or none to use dockerhub.

___

*#Example 03 - Warning containers with `latest` image tag*

 1. Create Namespace (`kubectl apply -f namespace_latest.yml`)
 2. Create Policy (`kubectl apply -f policy_latest.yml`)
 3. Binding Policy (`kubectl apply -f policy_latest_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_latest.yml`)

Comments: This example allows the deployment to proceed, but issues a warning message to inform the user that this practice will be prevented in the near future. Engage in discussions with your team, evangelize them, and highlight the benefits of adhering to best practices :D.

___  
    
*#Example 04 - Prevent containers with privileged security context*

 1. Create Namespace (`kubectl apply -f namespace_root.yml`)
 2. Create Policy (`kubectl apply -f policy_root.yml`)
 3. Binding Policy (`kubectl apply -f policy_root_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_root.yml`)

Comments: In the line 18 of the `policy_root.yml` file, we check for the `privileged` attribute in the securityContext to be set to false. This is considered a secure configuration. If it's set to `true`, the operation will fail.

___  

*#Example 05 - Prevent deployments without required labels*

 1. Create Namespace (`kubectl apply -f namespace_labels.yml`)
 2. Create Policy (`kubectl apply -f policy_labels.yml`)
 3. Binding Policy (`kubectl apply -f policy_labels_binding.yml`)
 4. Create ConfigMap (`kubectl apply -f configmap_labels.yml`)
 5. Test Deployment (`kubectl apply -f nginx_deploy_labels.yml`)

Comments: This policy utilizes a configMap with a list of required labels. The initial deployment will fail; therefore, uncomment lines 6-8 in the `nginx_deploy_labels.yml` file and reapply it for a successful deployment.

___  
  
*#Example 06 - Prevent deployments without probes defined*

 1. Create Namespace (`kubectl apply -f namespace_probes.yml`)
 2. Create Policy (`kubectl apply -f policy_probes.yml`)
 3. Binding Policy (`kubectl apply -f policy_probes_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_probes.yml`)

Comments: Probes are a best practice to inform Kubernetes about when your app is ready to receive connections or needs to be restarted. While it's not a required field in deployment, creating this VAP allows you to enforce liveness, readiness, and startup probes. Uncomment lines 28-51 in the `nginx_deploy_probes.yml` to successfully deploy it.

___  

*#Example 07 - Prevent deployments without resources defined*

 1. Create Namespace (`kubectl apply -f namespace_resources.yml`)
 2. Create Policy (`kubectl apply -f policy_resources.yml`)
 3. Binding Policy (`kubectl apply -f policy_resources_binding.yml`)
 4. Test Deployment (`kubectl apply -f nginx_deploy_resources.yml`)

Comments: It's considered a best practice to define resources in your deployment, as described [here](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits). There's an ongoing discussion about whether to set CPU limits or not, so choose wisely :D ([article 1](https://home.robusta.dev/blog/stop-using-cpu-limits) & [article 2](https://dnastacio.medium.com/why-you-should-keep-using-cpu-limits-on-kubernetes-60c4e50dfc61)). Uncomment lines 21-27 in the file `nginx_deploy_resources.yml` to make it work.

___  
  

This was presented during a workshop in Portuguese (Brazil) and is available online on [YouTube](https://www.youtube.com/getupcloud).

Other useful links can be found [here](http://gtup.me/useful_links). 
