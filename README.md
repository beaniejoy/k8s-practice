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

<br>

## Job & CronJob

- my-job.yaml
- my-cronjob.yaml

`activeDeadlineSeconds`은 해당 컨테이너의 실행 시간이 설정값 초과인 경우 timeout 종료 시킴  
실제로 해당 설정값도 더 오래 Job 객체의 파드가 존재하게 되는 이유는 graceful shutdown 및 초기 생성 시간 등 때문에 더 오래 걸

<br>

## Configuration

```shell
$ kubectl exec env-test-pod -- printenv
```
exec를 통해 파드의 환경변수 print

kube documentation에서 `downward` 검색 > Downward API에서 fieldRef로 가져올 수 있는 값들 확인 가능

```shell
$ kubectl get pods -o wide
$ kubectl get deploy my-deploy -o yaml
```
pod의 내용을 확장해서 더 많은 정보들과 같이 확인 가능
yaml 파일 확인하고 싶을 때에도 사용 가능
      
```shell
$ kubectl create secret generic db-info --from-literal=DB_PW=beanie
```

```shell
$ kubectl get secret my-secret -o yaml
$ echo [encoded_string] | base64 -d
```
yaml 파일 확인해서 base64 인코딩된 내용을 디코딩해서 볼 수 있다.

`my-config-pod.yaml` 내용

```shell
$ kubectl exec my-config-pod -- cat /config-files/welcome.sh
```
volumeMounts path에 지정한 경로에 volume에 들어갔던 파일이 컨테이너에 들어가 있는 것 확인 가능

<br>

## PV

`my-storageclass.yaml` 참고  
StorageClass의 `volumeBindingMode`가 `WaitForFirstConsumer`인 경우 pod에 해당 sc를 적용한 것이 생성될 때 pvc가 Bound 된다.  
(dynamic pvc 생성만 하는 경우 pod 적용전이기에 pending 상태로 떠있게 된다.)

StatefulSet에도 dynamic pvc를 적용할 수 있는데, 해당 spec에 `volumeClaimTemplates`를 가지고 설정할 수 있다.  
그렇게 되면 해당 sts를 실제 적용할 때 pvc도 같이 새로 생성, replica 개수를 늘리면 그에 따라 pvc가 같이 생성  
replica 개수를 줄여도 기존 생성되었던 pvc는 삭제안된다. 그래서 replica 개수를 줄였다 다시 늘리면 기존 pvc를 재활용하게 된다.

<br>

## Network

Pod안에 여러 개의 container가 있는 경우 특정 container에 대해 exec 명령 수행하는 경우

```shell
kubectl exec my-localhost-pod -c another-container -- curl localhost:80
```

Service 객체를 쿠베에 적용하면 endpoint slice가 자동으로 생성되는데 서비스는 endpoint slice 기준으로 라우팅을 해준다.  

```shell
kubectl get endpointslice
NAME               ADDRESSTYPE   PORTS   ENDPOINTS                            AGE
kubernetes         IPv4          6443    172.18.0.2                           23d
my-service-hj8zr   IPv4          80      10.244.0.9,10.244.0.10,10.244.0.11   3m21s
```
위와 같이 나오는데 `ENDPOINTS`에 나오는 애들은 pod의 ip 주소라고 보면 된다. 

`my-localhost-pod`의 nginx container에서 serivce를 호출하면 service에 연결된 deployment의 pod 중 하나가 응답하게 된다.

ExternalName

```shell
kubectl exec my-localhost-pod -c another-container -- curl --header "Host: www.google.com" google-service
```

<br>

## namespace, NetworkPolicy 관련 내용

```shell
$ kubectl create namespace my-namespace
$ kubectl create namespace your-namespace
$ kubectl create namespace another-namespace
```

my-namespace의 my-nginx-pod에 들어가서 another, your namespace의 pod내 nginx 호출

```shell
$ kubectl -n my-namespace exec -it my-nginx-pod -- bash
root@my-nginx-pod:/# curl another-nginx-service.another-namespace.svc.cluster.local:9080 
root@my-nginx-pod:/# curl your-nginx-service.your-namespace.svc.cluster.local:9080
```

