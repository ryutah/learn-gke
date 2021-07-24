# learn-gke

## Enabling Services

- cloudbuild.googleapis.com
- container.googleapis.com

## Memo

- kind 等のローカルクラスタで Cloud Build を利用する場合は Credential をどうするかが課題
  - [kind-Private Registries](https://kind.sigs.k8s.io/docs/user/private-registries/)
  - [Google Cloud-認証方法](https://cloud.google.com/container-registry/docs/advanced-authentication?hl=ja#token)
- GKE autopilot で skaffold すると、Pod の更新がクソ遅くて正直辛い
  ```console
  $ k get pod -w
  NAME                             READY   STATUS    RESTARTS   AGE
  my-deployment-6d4ffcb5b4-4j5mk   0/1     Pending   0          51s
  my-deployment-6f76894bc4-bmz2l   1/1     Running   0          7m45s
  my-deployment-6d4ffcb5b4-4j5mk   0/1     Pending   0          59s
  my-deployment-6d4ffcb5b4-4j5mk   0/1     Pending   0          81s
  my-deployment-6d4ffcb5b4-4j5mk   0/1     ContainerCreating   0          81s
  my-deployment-6d4ffcb5b4-4j5mk   0/1     Running             0          92s
  my-deployment-6d4ffcb5b4-4j5mk   1/1     Running             0          98s  # 開始まで1.5min以上
  ```
- 料金もそこそこするし、実用的じゃないかも
  - [Google Cloud-料金](https://cloud.google.com/kubernetes-engine/pricing?hl=ja#autopilot_mode)

## GKE / Cloud Builds を利用した Skaffold

```yaml
apiVersion: skaffold/v2beta18
kind: Config
metadata:
  name: helloworld
build:
  artifacts:
    - image: gcr.io/[PROJECT_ID]/helloworld
      buildpacks:
        builder: gcr.io/buildpacks/builder:v1
  googleCloudBuild:
    projectId: [PROJECT_ID]
    packImage: gcr.io/k8s-skaffold/pack
deploy:
  kubectl:
    manifests:
      - k8s.yaml
portForward:
  - resourceType: service
    resourceName: my-svc
    namespace: default
    port: 80
    localPort: 8080
```
