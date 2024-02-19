### Generate a private key and CSR file for the user 'Viola' using OpenSSL

```bash
# the command to generate the CSR file using OpenSSL
openssl genrsa -out viola.key 2048

# the command to create the csr file
openssl req -new -key viola.key -subj "/CN=viola/O=developers" -out viola.csr
```

### Submit a CSR (certificate signing request) object in Kubernetes and approve the request using kubectl.

```bash
# the commands to create a CSR via YAML file
export REQUEST=$(cat viola.csr | base64 -w 0)

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
 name: viola
spec:
 groups:
- developers
request: $REQUEST
signerName: kubernetes.io/kube-apiserver-client
usages:
- client auth
EOF

# the command to approve the certificate signing request
kubectl certificate approve viola
```

### Add the credentials from the CSR to a new kubeconfig for Viola

```bash
# the command to extract the certificate from the CSR
kubectl get csr viola -o jsonpath='{.status.certificate}' | base64 -d > viola.crt

# the command to add the credentials from the CSR to a new kubeconfig file
kubectl config set-credentials viola --client-key=viola.key --client-certificate=viola.crt --embed-certs
```

### Configure the context for the kubeconfig

```bash
# complete the command to set the context for the 'Voila' user
kubectl config set-context viola --user=viola --cluster=kubernetes
```

### Test access by using the auth can-i option with the kubectl command

```bash
# complete the command to test access using 'auth can-i' with kubectl
kubectl auth can-i list pods --as viola
```

### Create a role and role binding that will allow Viola to create, list and delete pods and deployments

```bash
# create the role that allows Viola to create, list and delete pods and deployments
kubectl create role dev-pod-creator –verb=get,list,watch,delete –resource=pods,deploy

# create a role binding to bind the role to user Viola
kubectl create rolebinding pod-developer-binding --role=dev-pod-creator --user=viola
```

# alternative solution
```bash
kubectl create rolebinding pod-reader-bind --role=pod-reader --group=developers
```
