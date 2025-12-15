# Home-lab

## Commands

```bash
# force flux reconcile instead of waiting for flux to pick up the github changes
flux reconcile kustomization apps

```

## Secrets

We are using SOPS to push encrypted secrets as part of our code.
A better approach is to use a vault like AWS secret manager or
Github secrets

### SOPS (Recommended)

To encrypt a k8s secret you can do the following:

```bash
export AGE_PUBLIC={public_age_key}
sops --age=$AGE_PUBLIC --encrypt --encrypted-regex '^(data|stringData)$' --in-place test-secret.yaml
```

The result `yaml` can be pushed to Github

#### Install and enable SOPS (first time)

The first step is manual and have to be done when creating a new cluster

```
# Step 1
age-keygen -o age.agekey # save this to a pass manager

# Add it to our cluster
cat age.agekey |
             kubectl create secret generic sops-age \
                 --namespace=flux-system \
                 --from-file=age.agekey=/dev/stdin


# Step 2
# In the clusters/staging directory, add:

#.sops.yaml

creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    age: age1zd5n9zx0dsdwdggjuvz32ppngn45gk0tcnwlwfs53ev7szefmdfqarudsl


# in the apps.yaml Kustomization file, add under spec:
# (same level as prune: true)

decryption:
    provider: sops
    secretRef:
      name: sops-age

```

### Cloudflare linkding tunnel (Not recommended/manual)

Alternative: if you don't want to using encryption for keys
you can run the create secret command manually

```bash
cloudflared tunnel create ld # creates the tunnel in Cloudflare
cd ~/.cloudflared
k create secret generic tunnel-credentials --from-file=credentials.json={GUID}.json
```
