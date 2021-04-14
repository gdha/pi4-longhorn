# README of pi4-longhorn

## Prepare the USB block device for persistemt storage for k3s

## Getting helm executable
Getting helm from URL https://helm.sh/docs/intro/install/

```bash
gdha@n1:~/projects/pi4-longhorn$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
gdha@n1:~/projects/pi4-longhorn$ chmod 700 get_helm.sh
gdha@n1:~/projects/pi4-longhorn$ ./get_helm.sh 
Downloading https://get.helm.sh/helm-v3.5.3-linux-arm64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```

- installing longhorn via helm

 * see https://longhorn.io/docs/1.1.0/advanced-resources/default-disk-and-node-config/
   Adding Node Tags to New Nodes

## Adding helm chart repo

```bash
gdha@n1:~/projects/pi4-longhorn$ helm repo add longhorn https://charts.longhorn.io
"longhorn" has been added to your repositories

gdha@n1:~/projects/pi4-longhorn$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "longhorn" chart repository
Update Complete. ⎈Happy Helming!⎈
```

## Installing longhorn with helm

```bash
$ helm install longhorn longhorn/longhorn --namespace longhorn-system \
  --set defaultSettings.defaultDataPath="/app/longhorn/"
NAME: longhorn
LAST DEPLOYED: Wed Apr 14 09:23:52 2021
NAMESPACE: longhorn-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Longhorn is now installed on the cluster!

Please wait a few minutes for other Longhorn components such as CSI deployments, Engine Images, and Instance Managers to be initialized.

Visit our documentation at https://longhorn.io/docs/
```

## Prepare the longhorn-ingress (required for UI)

- Accessing longhorn UI
 see https://longhorn.io/docs/1.1.0/deploy/accessing-the-ui/longhorn-ingress/

```bash
gdha@n1:~/projects/pi4-longhorn$ USER=gdha; PASSWORD=*******; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth


gdha@n1:~/projects/pi4-longhorn$ cat auth 
gdha:$apr1$XXXXXXXXXXXXXXXXXXXXXXXX

gdha@n1:~/projects/pi4-longhorn$ kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
secret/basic-auth created

gdha@n1:~/projects/pi4-longhorn$ kubectl -n longhorn-system get secret basic-auth -o yaml
apiVersion: v1
data:
  auth: Z2RoYTokYXByMSRLTU1hQWpiSiROVENtRWI2Qm05dDdvSmJXV1RlWVcuCg==
kind: Secret
metadata:
  creationTimestamp: "2021-04-09T15:38:05Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:auth: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2021-04-09T15:38:05Z"
  name: basic-auth
  namespace: longhorn-system
  resourceVersion: "763254"
  uid: 0a88e170-071d-4813-934f-9dec352be01d
type: Opaque
```

```bash
gdha@n1:~/projects/pi4-longhorn$ echo "
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # prevent the controller from redirecting (308) to HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: 'false'
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required '
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: longhorn-frontend
          servicePort: 80
" | kubectl -n longhorn-system create -f -
ingress.networking.k8s.io/longhorn-ingress created

gdha@n1:~/projects/pi4-longhorn$ kubectl -n longhorn-system get ingress
NAME               CLASS    HOSTS   ADDRESS         PORTS   AGE
longhorn-ingress   <none>   *       192.168.0.201   80      11s
```

Test the UI:
```bash
gdha@n1:~/projects/pi4-longhorn$ curl -v http://192.168.0.201/
```
