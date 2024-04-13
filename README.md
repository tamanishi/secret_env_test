# secret_env_test

- `env:`と`envFrom:`を混在してmanifestに書けるかテスト

``` sh
~/forKubernetes/secret_env_test
> cat secret.file
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: secret.file
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ TEST_SECRET_VALUE=hogehoge
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
~/forKubernetes/secret_env_test
> kubectl create secret generic --save-config sample-auth --from-env-file ./secret.file
secret/sample-auth created
~/forKubernetes/secret_env_test
> kubectl get secrets sample-auth -o yaml
apiVersion: v1
data:
  TEST_SECRET_VALUE: aG9nZWhvZ2U=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"kind":"Secret","apiVersion":"v1","metadata":{"name":"sample-auth","creationTimestamp":null},"data":{"TEST_SECRET_VALUE":"aG9nZWhvZ2U="}}
  creationTimestamp: "2024-04-13T00:47:54Z"
  name: sample-auth
  namespace: default
  resourceVersion: "1627"
  uid: 166fe41f-dd67-4149-8ca1-99ce724885c6
type: Opaque
~/forKubernetes/secret_env_test
> cat sample-auth2.yaml
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: sample-auth2.yaml
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ apiVersion: v1
   2   │ kind: Pod
   3   │ metadata:
   4   │   name: sample-auth2
   5   │ spec:
   6   │   containers:
   7   │     - name: ubuntu
   8   │       image: ubuntu:latest
   9   │       command:
  10   │       - sh
  11   │       - -c
  12   │       args:
  13   │       - tail -f /dev/null
  14   │       env:
  15   │       - name: TEST_PLAIN_VALUE
  16   │         value: fugafuga
  17   │       envFrom:
  18   │       - secretRef:
  19   │           name: sample-auth
  20   │
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
~/forKubernetes/secret_env_test
> kubectl apply -f sample-auth2.yaml
pod/sample-auth2 created
~/forKubernetes/secret_env_test
> kubectl exec -it sample-auth2 /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@sample-auth2:/# env | grep -e TEST_
TEST_SECRET_VALUE=hogehoge
TEST_PLAIN_VALUE=fugafuga
root@sample-auth2:/#
root@sample-auth2:/# exit
exit
~/forKubernetes/secret_env_test
>
~/forKubernetes/secret_env_test
> kubectl delete pod sample-auth2
pod "sample-auth2" deleted
~/forKubernetes/secret_env_test
> kubectl delete secrets sample-auth
secret "sample-auth" deleted
~/forKubernetes/secret_env_test
>
~/forKubernetes/secret_env_test
>
```
