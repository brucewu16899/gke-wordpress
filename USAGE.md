# Using GKE WordPress
## Prerequisites
* **GKE Cluster**. Follow the [official Kubernetes guide](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-container-cluster "Creating a Container Cluster").
* **Cloud SQL Instance**. Follow the [Creating a Google Cloud SQL guide](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance") 
* **Cloud SQL credentials** saved to your locally as `credentials.json`. You'll need them later. See [Connecting Cloud SQL to Kubernetes Engine](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine).
* **Domain and access to its DNS settings**. These instructions use the generic domain name `mysite.com` as an example site domain. You should replace it with your own domain name.
* Upon deploying WordPress you should install:
  * [**Redis Object Cache**](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin to connect your site to the Redis `Deployment`
  * [**NGINX Cache**](https://wordpress.org/plugins/nginx-cache/) if you want to make sure changes appear on your website promptly.

## Installation
1. Install **GKE WordPress** project locally
```bash
$ mkdir -p gke-wp{wp-sites} && cd gke-wp
$ git clone https://github.com/stcox/gke-wordpress.git && cd gke-wordpress
```

2. Install **Helm & Tiller**
```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
$ kubectl create -f tiller-rbac-config.yaml
$ helm init --service-account tiller
```

3. Install core cluster services: **Nginx-Ingress, Kube-Lego and Redis**. Replace the kube-lego `legoEmail=` value with your own email address.
```bash
$ helm install nginx-ingress
$ helm install kube-lego --set legoEmail=myemail@mysite.com
$ helm install redis && cd ..
```

## Usage
### Adding websites
The example uses `mysite-com`, for the site's namespace, and `mysite.com` for the domain. **All WordPress namespaces are automatically prefixed with **`wp-`** to create the site's namespace, so they are easier to find; consequently, the example namespace will appear as **`wp-mysite-com`** in kubernetes.

The example domain, `mysite.com` only works with HTTP via a local hosts file. You should substitute your own domain.

Free LetsEncrypt TLS/SSL/HTTPS/HTTP2 certificates are available for any domains you control. LetsEncrypt is enabled by setting `tls: true` in the site's configuration file.

1. **Create an A record** for your domain, `mysite.com` at your domain name provider (Godaddy, Bluehost, etc.), and point it to your Ingress IP address. [Get your cluster's Ingress IP Address](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/service?namespace=nginx-ingress)

2. **Configure site values** for `mysite.com`.
```bash
$ cp ./gke-wordpress/wordpress/values.yaml ./wp-sites/mysite-com.yaml
$ nano ./wp-sites/mysite-com.yaml
```

3. **Install WordPress** with `mysite.com` values.
```bash
$ helm install -f ./wp-sites/mysite-com.yaml ./gke-wordpress/wordpress
```

## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.

## Donate
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=FNLE7XYVKHSS2)
