# RBAC Demo

# Create a namespace named 'role'
kubectl create namespace role

# Create a directory for role-related files and navigate into it
mkdir role && cd role


# Generate an RSA private key
sudo openssl genrsa -out user3.key 2048

# Create a certificate signing request (CSR) using the private key
sudo openssl req -new -key user3.key -out user3.csr

# Sign the CSR to generate a certificate (linked with Kubernetes CA)
sudo openssl x509 -req -in user3.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out user3.crt -days 500

Role and RoleBinding Setup
# Edit or create a Role YAML file
vi role.yaml
#use the role.yaml from git repo


# Create the role from the YAML file
kubectl create -f role.yaml

# Verify the created role
kubectl get roles -n role

# Edit or create a RoleBinding YAML file
vi rolebinding.yaml

#use the rolebinding .yaml from git repo

# Create the RoleBinding from the YAML file
kubectl create -f rolebinding.yaml

# Verify the created RoleBinding
kubectl get rolebinding -n role


Set Up User Credentials

# Assign credentials to user3 using the certificate and key
kubectl config set-credentials user3 --client-certificate=/home/labsuser/role/user3.crt --client-key=/home/labsuser/role/user3.key

# Set up a context for user3 in the 'role' namespace
kubectl config set-context user3-context --cluster=kubernetes --namespace=role --user=user3

# Display all contexts
kubectl config get-contexts

# View the current kubeconfig file
	
cd ..
cat .kube/config

Role Verification and Testing

# List pods in the 'role' namespace with the user-specific kubeconfig
kubectl get pods --kubeconfig=myconf

# Deploy a test application in the 'role' namespace
kubectl create deployment test --image=docker.io/httpd -n role --kubeconfig=myconf

# Verify the created deployment and its pods
kubectl get deployment --kubeconfig=myconf
kubectl get pods --kubeconfig=myconf

# Create a ConfigMap in the 'role' namespace
kubectl create configmap my-config --from-literal=key1=config1 --kubeconfig=myconf

# View the created ConfigMap
kubectl get configmaps --kubeconfig=myconf
kubectl get configmap my-config --kubeconfig=myconf -o yaml


File Management and Key Distribution

# Navigate back to the home directory
cd ..

# View the certificate and key files
cat user3.crt
cat user3.key

# Copy files to the worker node (example commands for manual steps)
scp user3.crt worker-node-1:/role/
scp user3.key worker-node-1:/role/

# On the worker node, create a directory and add files
mkdir -p /role && cd /role
vi user3.crt  # Paste the certificate content
vi user3.key  # Paste the key content

---