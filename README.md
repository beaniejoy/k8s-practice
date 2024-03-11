# kubernetes practice

## Pod 

```shell
$ kubectl apply -f my-simple-pod.yaml
```

```shell
$ kubectl get pod
$ kubectl get pods
$ kubectl get po

$ kubectl describe pod my-simple-pod
```
pods, pod, po 모두 가능


```shell
$ kubectl delete pod my-simple-pod
```

```shell
$ kubectl explain pod
$ kubectl explain pod.spec
```
pod에 대한 스펙을 볼 수 있음

<br>

## ReplicaSet

- `my-reoplicaset.yaml`

```shell
$ brew install watch
```
주기를 가지고 설정한 명령어를 실행

```shell
$ kubectl apply -f my-replicaset.yaml
$ kubectl get replicaset nginx-replicaset
```

```shell
$ kubectl delete pod nginx-replicaset-wl6r8
```
특정 파드를 지우면 해당 파드는 terminated 되는 동시에 새로운 파드가 뜨게 된다. (replicas 3으로 설정)  
그리고 replicaset을 통해 pod를 띄우면 `[replicaset.name]-[hashcode]` 형식으로 파드별로 고유의 값을 자동으로 부여받는다. 


파드를 늘려줄 때는 yaml에서 replicas를 늘려주면 늘어난 개수만큼만 신규 파드가 생성된다. (기존 것은 그대로)

```shell
$ kubectl scale replicaset/nginx-replicaset --replicas=2
```
위와 같이 명령형으로 스케일 조정 가능

```shell
$ kubectl label pod my-simple-pod app=nginx
```
위와 같이 실행중인 파드에도 label 부여 가능하다. 이 상태로 replica 개수를 줄이면 의도치 않은 파드가 죽어버릴 수 있음. 그렇기에 replicaset 설정시 label은 고유의 값으로 지정하는 것이 좋다.

그리고 replicaset은 파드의 spec을 신경쓰지 않는다. 변경 인지 못함 (=> deployment 존재이유)  
만약 버전을 업그레이드 하고 싶으면 기존 pod를 delete해야 한다. (자동 조정에 의해 새로운 버전의 파드가 뜨게 될 것이다.)

<br>

## Deployment

replicaset과 마찬가지로 자동으로 각 파드의 이름을 정해준다.  
`[deployment-name]-[replica:hashcode]-[pod:hashcode]` 형태로 이름 자동 부여

```shell
$ kubectl get deployment
$ kubectl rollout status deployment/my-deploy
$ kubectl rollout history deployment/my-deploy
```
rollout history 명령을 통해 rollout 이력 확인 가능 (revision number 확인)

```shell
$ kubectl rollout undo deployment/my-deploy --to-revision=[revision_num]
```
지정한 revision number로 롤백 가능

<br>

## StatefulSet

redis-set-0, redis-set-1 이런식으로 statefulSet pod가 생성된다.

```shell
$ kubectl api-resources
```
참고로 각 객체에 대해 쿠베에서 약어를 지원해주는데 어떤 약어가 있는지 위의 명령어로 확인가능

```shell
$ kubectl scale sts/redis-set --replicas=5
$ kubectl scale sts/redis-set --replicas=2
```
스케일을 5개로 늘렸다가 2개로 다시 줄이면 sts는 마지막 기준으로 2부터 생성되고 맨마지막부터 제거
