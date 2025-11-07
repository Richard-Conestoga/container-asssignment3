# Assignment 3 - Gitea Lab
k8s gitea lab to take dev (sqlite based) to prod (mysql based)

TLDR;

```bash
pyenv install 3
pyenv global 3
pip install ansible kubernetes
git submodule update --init --recursive
ansible-playbook up.yml
```

Wait until `kubectl get pod` shows all pods running and check the Cloudflare Tunnel List:

```bash
cloudflared tunnel list
```

Check your Cloudflare ID on the list and:

```bash
cloudflared tunnel route dns <<CLOUDFLARE_ID>> <<HOST>>
```

Edit your config file to reflet your hosts list:

```bash
sudo nano /etc/cloudflared/config.yml
```

This is how this config file looks like:

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

If any of the Sanity check fail, restart Kubernetes:

```bash
ansible-playbook down.yml
ansible-playbook up.yml
```

Double check the sanity check to make sure everything is running fine:
Try to reach your Gitea server using another tab in your browser.
