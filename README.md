# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app
  labels:
    app: deployment-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-app
  template:
    metadata:
      labels:
        app: deployment-app
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo New-string >> /output/New-file.txt; sleep 5; done']
        volumeMounts:
          - name: vol
            mountPath: /output
        imagePullPolicy: IfNotPresent
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
          name: multitool
        volumeMounts:
        - name: vol
          mountPath: /input
        imagePullPolicy: IfNotPresent
      volumes:
      - name: vol
        emptyDir: {}
```

4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

<p align="center">
    <img width="1200 height="600" src="/img/deployment-app.png">
</p>

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-app
  namespace: default
  labels:
    k8s-app: daemonset-app
spec:
  selector:
    matchLabels:
      name: daemonset-app
  template:
    metadata:
      labels:
        name: daemonset-app
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /multitool
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

| DaemonSet ls /multitool |
| :---: |
<p align="center">
    <img width="1200 height="600" src="/img/daemonset-app-multitool-ls.png">
</p>

| Localhost ls /var/log |
| :---: |
<p align="center">
    <img width="1200 height="600" src="/img/host-var-log-ls.png">
</p>

| DaemonSet journalctl |
| :---: |
<p align="center">
    <img width="1200 height="600" src="/img/daemonset-app-multitool-journal.png">
</p>

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
