# EUCAIM Mockup deployment

This repo contains the information required for the deployment of the mockup in EUCAIM. It will feature the deployment of an NFS volume and a web server, and a file serivce attached to it.

## Prerrequisites

The components should be deployed on top of a K8s cluster. For this purpose, we used the IM dashboard in <https://im.egi.eu/im-dashboard/>  and deploy a K8s cluster. The K8s can be accessed through the Dashboard (<http://eucaim-k8s.ramses.i3m.upv.es/dashboard/#/login>), using the admin or Service Account Token.

NOTE: If DNS redirections are used, the UPV service only supports the `10.10.*.*` network in `ramses.i3m.upv.es` cloud. IM creates the VMs in the `10.0.0.*` network. To solve this issue, the following steps are needed:
- Add a new NIC to the K8s front end using the sunstone cloud interface.
- Install the nmcli package (`apt-get update && apt-get install -y network-manager`)
- Retrieve the name of the disabled device (`nmcli device status`)
- Bring it up (`nmcli device up <device>`)
- Check that the ip is properly assigned (either `ifconfig`or `ip addr`). 

## Components and topology

The deployment comprises the following objects:
- An NFS persistent volume
- A Persistent Volume Claim (PVC)
- An nginx web server that mounts the PVC on the html data directory `/usr/share/nginx/html/`
- A NodePort service for the nginx web server at the address `http://<Frontend Public ÎP>:30007`
- A Web file server using the `tinyfilemanager/tinyfilemanager:master` Docker image, that mounts the same PVC as the nginx server in `/var/www/html/data`
- A NodePort service for the web file server at the address `http://<Frontend Public ÎP>:30011`

## Deployment

The Persistent Volumen and the Persistent Volume Claim are deployed using the `nginx-vol.yaml` file. Before deploying them, the NFS server IP should be fixed. The rest of the services is deployed through the `nginx.yaml` and `file-browser.yaml` files.

We used a separate namespace for the deployment so we could delegate the administration of the services (except for the Persistent Volume) to a Service Account user. 

The credentials for the users can be updated in the config.php file using a hashed password using <https://tinyfilemanager.github.io/docs/pwd.html>. The config file can be uploaded once the deployment has been completed (NOTE: It could be turned as a configmap file for convenience).

## DNS redirections

In order to avoid opening ports in the UPV's corporate network, we created the following redirections:
- mockup-fs.ramses.i3m.upv.es navigate_next 10.10.1.34:30011
- eucaim-mockup.ramses.i3m.upv.es navigate_next 10.10.1.34:30007

 
