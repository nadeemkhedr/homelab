# Homelab

## Generate age key to use in our cluster

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

Alternative: if you don't want to using encryption for keys, you can run the create secret command manually

### Cloudflare linkding tunnel

```bash
cloudflared tunnel create ld # creates the tunnel in cloudflare
cd ~/.cloudflared
k create secret generic tunnel-credentials --from-file=credentials.json={GUID}.json
```
