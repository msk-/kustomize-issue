Specifying `nameSuffix` or `namePrefix` in `handlers/kustomization.yaml` seems to result in the
patch specified in `./kustomization.yaml` not being applied with the correct hash suffix.

Moving `./handlers/deployment.yaml` into this directory, and into this `kustomization.yaml` file
appears to cause the replacement to function correctly both in the presence and absence of
`nameSuffix` or `namePrefix`.

Removing the dependency on `./mysql` in `./kustomize.yaml` and generating the config map (instead
of merging it) also works as expected.

```
$ kustomize version
Version: {Version:3.3.1 GitCommit:f2ac5a2d0df13c047fb20cbc12ef1a3b41ce2dad BuildDate:unknown GoOs:linux GoArch:amd64}
```

Reproduce:
```sh
kustomize build
```

Produces:
```yaml
apiVersion: v1
data:
  MYSQL_DATABASE: db
  MYSQL_USER: user
kind: ConfigMap
metadata:
  annotations: {}
  labels: {}
  name: mysql-h7d4485k9d
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: short-suffix
spec:
  template:
    spec:
      containers:
      - env:
        - valueFrom:
            configMapKeyRef:
              key: MYSQL_DATABASE
              name: mysql
        envFrom:
        - configMapRef:
            name: mysql
        name: handler
```

Removing the `nameSuffix` field from `./handlers/kustomization.yaml` such that it contains:
```yaml
resources:
  - ./deployment.yaml
```
yields from `kustomize build`:
```yaml
apiVersion: v1
data:
  MYSQL_DATABASE: db
  MYSQL_USER: user
kind: ConfigMap
metadata:
  annotations: {}
  labels: {}
  name: mysql-h7d4485k9d
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: short
spec:
  template:
    spec:
      containers:
      - env:
        - valueFrom:
            configMapKeyRef:
              key: MYSQL_DATABASE
              name: mysql-h7d4485k9d
        envFrom:
        - configMapRef:
            name: mysql-h7d4485k9d
        name: handler
```
