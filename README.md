# Visualizations CI/CD
This repository contains the configuration used to build the CI/CD pipeline for Endcoronavirus.org's 
[Green Zone Visualizations](https://github.com/vbrunsch/rankings/).
## Deployment
* Assuming an existing Kubernetes cluster with Flannel CNI.
1. Generate the necessary secrets.
   * For Concourse, consult the Helm chart's [Github repo](https://github.com/concourse/concourse-chart)
     * If you are not using a credential manager, configure localUsers in the secrets as well as 
       the `concourse.auth.localUser` value
     * If you are using the PostgreSQL chart dependency, configure the username and password
   * For the Docker Registry, you can either use a credential manager or generate credentials 
     using htpasswd and plug it into the values.yml. Then, store the creds in the secret `default/regcred` of type 
     `kubernetes.io/dockerconfigjson`
   * For Ingress-NGINX, if you are going to use SSL termination, create a TLS secret and reference it in 
     `controller.extraArgs` like so: `default-ssl-certificate: "<namespace>/<secret_name>"`
   * The Concourse pipeline also requires access to the Kubernetes cluster in which the rankings will be deployed.
    For this, first create a service account with a role of (at least) edit. Then, create a 
     secret `concourse-main/cluster-auth` (with `main` being the team the pipeline is run on).
     The secret should contain the following data:
     1. Key: `certificate-authority-data`  
        Value: the contents of `/etc/kubernetes/pki/ca.crt`
     2. Key: `token`  
        Value: the token of the service account you just created.
2. Apply persistent-volumes.yml. If you adjusted persistent volume sizes within any chart's values.yaml, adjust 
   the sizes in this file as well.
3. Adjust any values necessary within the configuration. Notable points may be the IP pool in the metallb/config.yaml,
   the Concourse URL in concourse/values.yaml, the Git repository URLs in the pipelines.
3. Deploy the following (any values.yml should be the one corresponding to that service):
   * Ingress-NGINX: `helm install ingress-nginx ingress-nginx/ingress-nginx -f values.yml`
   * [Metallb](https://metallb.universe.tf/installation/#installation-by-manifest) (then, apply the metallb/config.yaml) 
   * Docker Registry: `helm install vis twuni/docker-registry -f values.yaml`
   * Concourse: `helm install concourse -f values.yaml concourse/concourse`
4. Set the pipelines.
4. Clean up!
