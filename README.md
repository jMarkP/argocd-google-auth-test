# ArgoCD Google OIDC auth example

Instructions:
- Have a running local Kubernetes cluster
- Install ArgoCD using default instructions from https://argo-cd.readthedocs.io/en/stable/getting_started/:
```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
- Add an entry to your machine's hosts file to point argocd.example.com to localhost. On a Mac:
```
$ sudo nano /private/etc/hosts
```
```
127.0.01 argocd.example.com
```
- Setup an OIDC client in GCP: https://console.cloud.google.com/apis/credentials:
  - Name: anything
  - Authorized redirect URI: `https://argocd.example.com:8080/auth/callback`
  - Remember the Client ID and secret
- Edit the argocd configmap to contain the OIDC config
```
$ kubectl -n argocd edit cm argocd-cm
```
```
apiVersion: v1
data:
  oidc.config: |
    name: Google
    issuer: https://accounts.google.com
    clientID:  YOUR_CLIENT_ID
    clientSecret: YOUR_CLIENT_SECRET
    requestedScopes: ["openid", "email"]
    requestedIDTokenClaims: {}
  url: https://argocd.example.com:8080
  admin.enabled: "false"
kind: ConfigMap
metadata:
  LEAVE THIS UNCHANGED
```
- In a new window setup port forwarder:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
- Navigate to https://argocd.example.com:8080
- Accept the SSL warning and continue
- Login with Google

# To test CURLing using the cookie
- In the browser, look for the cookie value in the network inspector for a request to ArgoCD. e.g.
```
argocd.token=eyJhb...kFRxA
```
- Copy that to a file so you can refer back to it
- Take the bit after `argocd.token=` and add it to an env var in your terminal
```
ARGO_TOKEN=<pasted token>
```
- Make a CURL like the following (`-k` to ignore cert issues)
```
curl -v -k --cookie "argocd.token=$ARGO_TOKEN" -X PUT https://argocd.example.com:8080/api/v1/projects/seed
```