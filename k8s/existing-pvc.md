---
title: existing-pvc
description: подключение существующего pvc к деплойменту
published: true
date: 2020-03-23T06:44:27.099Z
tags: k8s, pvc, existing-pvc
---

# Как подключить существующий pvc в деплоймент
допустим, был у вас деплой, в котором поды монтирование pvc (тут и далее речь о OpenEBS Jiva)
деплой как-то умер, pvc осталось в состоянии `release`.

### Задача: 
подключить к новому деплою старый pvc

### Решение:

> 1.создаем pvc.yaml с содержанием:
{.is-success}


```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    openebs.io/target-affinity: your-app
  name: app-pvc
  namespace: you-namespace
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: oebs-retain
  volumeMode: Filesystem
  volumeName: pvc-bla
```

где:
- `pvc-bla` = имени старого существующего pvc
- `app-pvc `- имя деплоя, куда монтируем (в деплое конечно же обратная связь с pvc)
- `you-namespace` - неймспейс куда монтируем
- `storage` - размер, который сейчас на старом хранилище

> 2.патчим волюм
{.is-success}

`kubectl patch pv pvc-bla -p '{"spec":{"claimRef": null}}'` -  что бы убрать признак `release` и высвободить волюм, он должен перейти в статус `available`

> 3.проверяем тип класса
{.is-success}

`kube describe pv pvc-bla`   - смотрим, какой storageClassName был у волюма до этого (если они у вас меняются после каждого обновления openEBS, как у меня)

> 4.запускаем
{.is-success}

`kube apply -f pvc.yaml`  - ну и собственно монтируем