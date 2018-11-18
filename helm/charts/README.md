# Helm Chart for Twistlock Console 


## Downloading the charts

Just clone this repository and install from the charts, we don't make Twistlock charts available from the Helm package repository as our product is commercial.

## Prerequisites

### Helm 

You will need [Helm](https://helm.sh/) installed (both the Helm client locally and the Tiller server component) on your Kubernetes cluster.

### Firewall
You will need to configure your kubenetes cluster firewall rules to allow ingress traffic on port 8081 for http and 8083 for https browser access.  Port 8084 is utilized for defender to console communications within your cluster (via TLS).

### Secrets

You will need the access token that comes with your Twistlock subscription; look for this in an e-mail from Twistlock support.

## Installing the Helm chart

* Copy **charts/twistlock-console/valuesTemplate.yaml** to **charts/twistlock-console/values.yaml** and fill in version, image tag, imageName, and  access token in values.yaml. Use _ (underscores) for tag and image name and . (periods) for version - i.e. **version: 2.5.127**, **tag: ```2_5_127```**.


* Review and optionally edit **charts/twistlock-console/charts/console/values.yaml** if you want to change the service type or default ports for the Twistlock console. **serviceType** can be **LoadBalancer** or **NodePort**.  LoadBalancer is the approved way to deploy in kubernetes and is the only choice we tested with this helm chart. Persistent volume size of **50GB** is recommended for larger production environments, **10GB** is sufficient for a trial or smaller deployment (less than 10 defenders).


* Now run:

	$ helm install twistlock-console -n twistlock-console --namespace=twistlock

	
## Configure and setup your Twistlock Console

NOTE: console https port defaults to 8083, if you changed it, use the https port you chose in twistlock-console/charts/console/values.yaml

You can see your console external IP address with:

	kubectl get service -n twistlock
	
Log into your console via a browser - https://<CONSOLE_EXTERNAL_IP>:8083, create an admin account, and install your license.  

## Installing the Twistlock Defender DS

The Helm chart installs the Twistlock console only.  

Provided script installs Defender daemonset after console is up and runing and license has been installed.  The httpsPort defaults to 8083 but it must match the httpsPort in **charts/twistlock-console/charts/console/values.yaml**.

	$ ./install_defender_ds.sh <8083>

For more information see support documentation [here.](https://docs.twistlock.com/docs/latest/install/install_kubernetes.html#_install_defender)

## Uninstalling Twistlock

First remove defender daemonset by running

	$ kubectl delete -f defender_ds.yaml

Then remove Twistlock console and namespace:

	$ helm delete twistlock-console --purge
	$ kubectl delete ns twistlock 