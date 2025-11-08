# Assignment 3 - Gitea Lab


This repository was created by using the template available at https://github.com/conestogac-acsit/cdevops-gitea.git

Firstly, go to the repository above, click on "Use this template" and then "Create a new repository"

Clone the repository you just created using the command:

```bash
git clone <<url>>
```

After cloning it, start the steps to install the system:

```bash
pyenv install 3
pyenv global 3
pip install ansible kubernetes
git submodule update --init --recursive
ansible-playbook up.yml
```

Wait until `kubectl get pod` shows all pods running, and check the Cloudflare Tunnel List:

```bash
cloudflared tunnel list
```

Check your Cloudflare ID on the list and:

```bash
cloudflared tunnel route dns <<CLOUDFLARE_ID>> <<HOST>>
```

Edit your config file to reflect your hosts list:

```bash
sudo nano /etc/cloudflared/config.yml
```

This is how this config file looks:

```bash
tunnel: <<CLOUDFLARE_ID>>
credentials-file: /home/ubuntu/.cloudflared/<<CLOUDFLARE_ID>>.json 

ingress:
  - hostname: devsecmindset.dev
    service: http://localhost:4180 # port oauth2-proxy is listening on
  - hostname: dev.devsecmindset.dev
    service: http://localhost:8080 # anything you run on this port will be accessible from the hostname ... use to smoke test
  - hostname: gitea.devsecmindset.dev
    service: http://localhost  

  - service: http_status:404
```

Restart the Cloudflared service:

```bash
sudo systemctl restart cloudflared
```

Apply the ingress.yml file for Gitea:

```bash
kubectl apply -f gitea/ingress.yml
```

Sanity check -> check if Kubernetes is reading the IP properly:

```bash
kubectl get ingress
```

Should have something like this:

```bash
NAME            CLASS   HOSTS                     ADDRESS      PORTS   AGE
gitea-ingress   nginx   gitea.devsecmindset.dev   10.172.27.3  80      15m
```

Sanity check 2 -> check if Kubernetes is reading the sources:

```bash
kubectl get svc
```

Should have something like this:

```bash
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.43.124.80    10.172.27.3   80:30911/TCP,443:32591/TCP   38s

```

If any of the Sanity checks fail, restart Kubernetes:

```bash
ansible-playbook down.yml
ansible-playbook up.yml
```

Double-check the sanity check to make sure everything is running fine:
Try to reach your Gitea server using another tab in your browser using your URL.

## Gitea Data Persistence


Install MySQL for Gitea

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install gitea-mysql bitnami/mysql \
  --set auth.rootPassword=root123 \
  --set auth.database=gitea \
  --set auth.username=gitea \
  --set auth.password=gitea_pass \
  --set primary.persistence.enabled=true \
  --set primary.persistence.size=5Gi
```

Deploy Gitea in Production Mode

```bash
helm install gitea-prod gitea-charts/gitea -f gitea/values.yaml
```

Now you are good to go. Start enjoying your selfhosted Gitea.

