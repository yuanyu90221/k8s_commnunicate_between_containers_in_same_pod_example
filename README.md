# k8s_commnunicate_between_containers_in_same_pod_example

## 前言

今天要來實作 [Communicate Between Containers in the Same Pod Using a Shared Volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/) 這個任務

k8s 在一個 Pod 內可以跑多個 container

而一般來說 Pod 會有一個主要應用的 container , 其他則是輔助應用的 container

這些 container 可以透過掛載 Pod 內的同一個 Volume 來做溝通

這次這個任務就是要來實踐上面所說

建立兩個 Container , nginx 與 debian , 共同掛載同一個 Volume

debian 把資料寫入 shared-data 內

nginx 再從 shared-data 把檔案牘出來


## 佈署目標

1 建立一個包含兩個 container 的 Pod 

2 進入 nginx 的 container 驗證可以從 shared-data 讀出 debian 寫入的檔

3 清除佈署

## 建立一個包含兩個 container 的 Pod 


建立 two-container-pod.yaml 如下：

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```

建立一個 Pod

名稱設定為 two-containers

設定 volumes 名稱為 shared-data

設定 container nginx-container 的 image 為 nginx

設定 container nginx-container 掛載 volume shared-data 到 /usr/share/nginx/html

設定 container debian 的 image 為 debian

設定 container debian 掛載 volume shared-data 到 /pod-data

設定 container debian 執行指令如下 

```shell=
/bin/sh -c "echo Hello from the debian container" > /pod-data/index.html
```

建立佈署指令如下：

```shell=
kubectl apply -f two-container-pod.yaml
```
![](https://i.imgur.com/nurvvG5.png)


檢查佈署指令如下：

```shell=
kubectl get pod two-containers --output=yaml
```
![](https://i.imgur.com/kpPvS24.png)

## 進入 nginx 的 container 驗證可以從 shared-data 讀出 debian 寫入的檔

進入 nginx-container 指令如下：

```shell=
kubectl exec -it two-containers -c nginx-container -- /bin/bash
```
![](https://i.imgur.com/jF4py6d.png)

在 nginx-container terminal 執行指令如下:

```shell=
apt-get update
apt-get install curl procps
ps aux
```
![](https://i.imgur.com/3E3nOSJ.png)


執行驗證指令如下:

```shell=
curl localhost
```
![](https://i.imgur.com/RwXvOZO.png)

## 清除佈署
```shell=
kubectl delete -f two-container-pod.yaml
```

## 後記

這個範例任務中, 兩個在同一個 Pod 的兩個 container 透過 Pod 的 Volumes 來做溝通
 
然而, 假設 Pod 被刪除並且重新建立, 任何建立在 Volume 內的資料都會消失

所以並非是個好方法, 溝通方式還是應該用透過 api 或是 message queue 的方式才比較即時