# syncwaves
Simple Example of Syncwave emulation using Kustomize

### Usage
1. Define Core apps in the syncwaves.env file
```shell
# HelmRelease resource name
sync-wave1=rancher-remote
sync-wave2=istio
sync-wave3=nginx
sync-wave4=rook-ceph
sync-wave5=flux

# DO NOT MODIFY!
sync-wave0=rancher-remote
```
1. Add annontations to the releases
```yaml
# ./replease2.yaml

apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: release2 
  namespace: capi-stage
  annotations:
    sync-wave: 4 # this is what we want to target

# ---
# ... Some configuration omitted for brevity
# ---
```

1. Run Kustomize on the directory
```bash
kubectl kustomize ./
```
Original Optput:
```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: release2
  namespace: capi-stage
  annotations:
    sync-wave: 4 # this is what we want to target
spec:
  dependsOn:
    - name: loot-bot3
    - name: loot-bot2
    - name: loot-bot1

# ---
# ... Some configuration omitted for brevity
# ---
```

Expected Output
```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  annotations:
    sync-wave: "4"
  name: release2
spec:
  dependsOn:
  - name: rancher-remote
  - name: nginx

# ---
# ... Some configuration omitted for brevity
# ---
```
### Known Issues
1. Currently not all resources will be reconciled in the sync wave before the next wave starts
    - It may be possible to create a kustomize overlay for each "Sync-wave" and point the kustomization at the source of the release
    - The Only concern I have is that the the resource may be applied twice, once for the original kustomizaion and once for the new overlay

2. The adding annontations will remove resources from  the syncwave
    - This will probally be solved once more time is spent on the patches

3. Still need to add depends on for the subsequent waves
    - Wave 4 depends on Wave 3
    - Wave 3 depends on Wave 2 and so on...
4. Maybe if there is a selector to select all resources with a specific annontation and add their names to a list
    - Then once all names were on the list make them a dependency for subsequent waves