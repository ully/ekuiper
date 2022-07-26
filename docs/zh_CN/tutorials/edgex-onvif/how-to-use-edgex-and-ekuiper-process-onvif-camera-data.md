<a name="zeMUl"></a>
# 
<a name="gkXTy"></a>
# 背景
现在边缘场景里有一类重要的场景就是视频，图片场景，通过终端摄像头采集图像，然后边缘进行分析处理，最终获得指令决策。而如何对接摄像头等设备并发送给视频图像分析程序处理是一个高频问题。在这里我将介绍结合edgex与ekuiper是如何解决这一问题。
<a name="vgJ2j"></a>
# 对接流程
<a name="SjD18"></a>
## 1.onvif协议及测试工具介绍
> Onvif，即Open Network Video Interface Forum ，可以译为开放型网络视频接口论坛，是安迅士、博世、索尼在2008年共同成立的一个国际性、开发型网络视频产品标准网络接口的开发论坛，后来由于这个技术开发论坛共同制定的开发型行业标准，就用该论坛的大写字母命名，即ONVIF 网络视频标准规范，习惯简称为：ONVIF协议。

onvif协议本质上是为了统一不同厂商的网络视频设备的一种规范，它包含了网络视频设备的基本功能的定义，操作接口，以及最重要的流媒体接口，这里需要说明的是onvif与rtsp的关系，onvif是一个网络视频设备的协议，它的流媒体视频部分采用了成熟的rtsp协议。支持rtsp协议设备的摄像头，不一定支持onvif，而支持onvif协议的设备一定是支持rtsp协议的。这是onvif组织的的官方网站（[https://www.onvif.org/ch/](https://www.onvif.org/ch/)），在上面可以看到官方认证的满足onvif的产品。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658810905227-0ec72b68-fd7b-4292-a7fd-1b1bc7b90e99.png#clientId=u805e8d11-69e2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=421&id=u9672a5b0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=841&originWidth=1707&originalType=binary&ratio=1&rotation=0&showTitle=false&size=477101&status=done&style=none&taskId=u9107cb40-55d5-4ded-af6c-5dd1ef447a4&title=&width=853.5)
<a name="XnWSt"></a>
## 2.模拟onvif摄像头设备
为了下面便于对接演示，如果没有现成的onvif摄像头，笔者介绍两款桌面摄像头软件供大家测试使用（试用版本均可免费使用15天）。<br />两款可选，均可试用15天

- DESKCAMERA [https://www.deskcamera.com/download/](https://www.deskcamera.com/download/) 
- ITVDESK [https://itvdesk.eu/en/products/product-itvdesk-onvif-ipcamera/download](https://itvdesk.eu/en/products/product-itvdesk-onvif-ipcamera/download)

下图为itvdesk产品的截图，简单通过以下几步，便能够很快配置出一个满足onvif协议的摄像头。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658633154997-8f37b234-ec56-4dd6-a936-fb0a202970e9.png#clientId=u1c453ae3-9a8a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=279&id=uc201543c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=755&originWidth=787&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71693&status=done&style=none&taskId=u2ec9d76e-5b03-4b86-9c62-41c106a5a20&title=&width=290.5)![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658638969141-987f18a9-1905-4445-a807-b58b430ac03c.png#clientId=u1c453ae3-9a8a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=280&id=u1ca60b13&margin=%5Bobject%20Object%5D&name=image.png&originHeight=660&originWidth=604&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56234&status=done&style=none&taskId=u3cccd76f-9e73-406a-9f05-e32e827a224&title=&width=256)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658638877862-e9da7232-c8f6-4763-9ee2-ed8e78b92018.png#clientId=u1c453ae3-9a8a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=215&id=u1bc4ad6b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=612&originWidth=802&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81786&status=done&style=none&taskId=ufa2babfd-ec41-42c1-adc3-2fa4769fc8f&title=&width=282)<br />注意，为了便于验证如图关掉onvif和rtsp验证。<br />为了方便测试，onvif官方提供了一个测试工具（ONVIF Device Test Tool），可以在此[下载](https://www.onlinedown.net/soft/971597.htm)，下载安装完成，打开该软件，选择合适的网段，点击“discover device”，如图便能够看到当前网络满足onvif协议的设备列表。：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658810089693-a2d3f50d-b442-4240-b363-a3ecdf0df4ac.png#clientId=uaff7801f-c82d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=435&id=fkbmJ&margin=%5Bobject%20Object%5D&name=image.png&originHeight=869&originWidth=975&originalType=binary&ratio=1&rotation=0&showTitle=false&size=152339&status=done&style=none&taskId=u5723e259-8d4d-4db0-8eee-548ed346f76&title=&width=487.5)<br />通过该软件可以对该设备onvif协议满足情况进行测试。<br />点击“debug”选项卡，点击“Get URLs”，获得视频流地址，点击Play Video，便能实时看到摄像头的视频。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658810147455-d32b9453-09e6-45e6-aa59-21d26712e24f.png#clientId=uaff7801f-c82d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=447&id=MMztV&margin=%5Bobject%20Object%5D&name=image.png&originHeight=894&originWidth=992&originalType=binary&ratio=1&rotation=0&showTitle=false&size=251570&status=done&style=none&taskId=ubb589c03-31c9-409a-9d22-3fa66f97c4f&title=&width=496)<br />根据测试工具可以得到其onvif地址和配置，即：http://10.100.116.172:7000/onvif/device_service，而其视频流地址为：rtsp://10.100.116.172:5554/ipc1-stream1/videodevice
<a name="bZZYF"></a>
## 3.将设备接入到edgex
为了能够让edgex接入设备，首先需要安装支持onvif协议的device service（设备服务）。以下内容均以edgex  2.1-jakarta版本（目前最新的LTS版本）为例演示，在这里能够看到edgex支持的所有设备服务列表。<br />[https://docs.edgexfoundry.org/2.1/microservices/device/Ch-DeviceServiceList/](https://docs.edgexfoundry.org/2.1/microservices/device/Ch-DeviceServiceList/)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658811142191-6d65bef0-cec7-4a21-b791-5d98d62e5a92.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=380&id=u70b8edc5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=759&originWidth=641&originalType=binary&ratio=1&rotation=0&showTitle=false&size=90724&status=done&style=none&taskId=u862a055e-52aa-4310-a7f7-24bff079650&title=&width=320.5)

可以看到，[device-camera-go](https://github.com/edgexfoundry/device-camera-go)就是onvif支持的device-service。其对应的github地址为：<br />[https://github.com/edgexfoundry/device-camera-go](https://github.com/edgexfoundry/device-camera-go)。（当然也可以安装最新的设备服务[device-onvif-camera](https://github.com/edgexfoundry/device-onvif-camera)）。
<a name="T4R0B"></a>
### 安装edgex及device-camera
<a name="RtN2A"></a>
#### 1）docker compose
安装设备服务，有一个比较推荐的做法是基于它们的edgex-compose 来生成安装文件。<br />[https://github.com/edgexfoundry/edgex-compose](https://github.com/edgexfoundry/edgex-compose)<br />具体步骤：
```yaml
git clone https://github.com/edgexfoundry/edgex-compose.git
cd edgex-compose/compose-builder
git checkout jakarta
 make gen ds-mqtt mqtt-broker ds-camera  ds-modbus  modbus-sim  no-secty ui //按需选择模块
```

通过上面的命令便可以生成一个包含ds-camera，即device-camera-go设备协议的docker compose文件。通过docker compose up，<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658811739558-fba3e01e-d2ac-48d5-a2f8-77fd678fb715.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=ud0ed5255&margin=%5Bobject%20Object%5D&name=image.png&originHeight=308&originWidth=1452&originalType=binary&ratio=1&rotation=0&showTitle=false&size=375606&status=done&style=none&taskId=ufba1edca-61ad-456d-b2f9-18daa1ab97d&title=&width=726)<br />这样设备驱动就成功安装了。
<a name="psQFH"></a>
#### 2）k3s + helm chart
在这里再介绍笔者实际使用的方式--基于k3s部署的edgex中部署device-camera-go。<br />首先，通过helmchart安装edgex
```yaml
$ git clone https://github.com/edgexfoundry/edgex-examples.git
$ cd edgex-examples
$ git checkout 82ce72ad71  #jakarta版本
$ cd deployment/helm
$ kubectl create namespace edgex
$ helm install edgex-jakarta -n edgex .
```
安装完成后，默认安装的协议不含device-camera，可参考其它device，增加camera支持。<br />进入deployment/helm/template目录，创建一个新目录edgex-device-camera，在该目录中，新建文件： **edgex-device-camera.yaml，内容为：**
```yaml
# Copyright (C) 2022 Intel Corporation
#
# SPDX-License-Identifier: Apache-2.0
#
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    org.edgexfoundry.service: {{.Values.edgex.app.device.camera}}
  name: {{.Values.edgex.app.device.camera}}
spec:
  replicas: {{.Values.edgex.replicas.device.camera}}
  selector:
    matchLabels:
      org.edgexfoundry.service: {{.Values.edgex.app.device.camera}}
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        org.edgexfoundry.service: {{.Values.edgex.app.device.camera}}
    spec:
      automountServiceAccountToken: false
      containers:
      - name: {{.Values.edgex.app.device.camera}}
        image: {{.Values.edgex.image.device.camera.repository}}:{{.Values.edgex.image.device.camera.tag}}
        imagePullPolicy: {{.Values.edgex.image.device.camera.pullPolicy}}
      {{- if .Values.edgex.security.enabled }}
        command: ["/edgex-init/ready_to_run_wait_install.sh"]
        args: ["/device-camera", "-cp=consul.http://edgex-core-consul:8500", "--registry", "--confdir=/res"]
      {{- end}}
        ports:
        - containerPort: {{.Values.edgex.port.device.camera}}
      {{- if not .Values.edgex.security.enabled }}
          hostPort: {{.Values.edgex.port.device.camera}}
          hostIP: {{.Values.edgex.hostPortInternalBind}}
      {{- end}}
        env:
        - name: SERVICE_HOST
          value: {{.Values.edgex.app.device.camera}}
        envFrom:
        - configMapRef:
            name: edgex-common-variables
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
        startupProbe:
          tcpSocket:
            port: {{.Values.edgex.port.device.camera}}
          periodSeconds: 1
          failureThreshold: 120
        livenessProbe:
          tcpSocket:
            port: {{.Values.edgex.port.device.camera}}
      {{- if .Values.edgex.security.enabled }}
        volumeMounts:
        - mountPath: /edgex-init
          name: edgex-init
        - mountPath: /tmp/edgex/secrets
          name: edgex-secrets
      {{- end }}
      {{- if .Values.edgex.resources.device.camera.enforceLimits }}
        resources:
          limits:
            memory: {{ .Values.edgex.resources.device.camera.limits.memory }}
            cpu: {{ .Values.edgex.resources.device.camera.limits.cpu }}
          requests:
            memory: {{ .Values.edgex.resources.device.camera.requests.memory }}
            cpu: {{ .Values.edgex.resources.device.camera.requests.cpu }}
      {{- end}}
      hostname: {{.Values.edgex.app.device.camera}}
      restartPolicy: Always
      securityContext:
        runAsNonRoot: true
        runAsUser: {{ .Values.edgex.security.runAsUser }}
        runAsGroup: {{ .Values.edgex.security.runAsGroup }}
    {{- if .Values.edgex.security.enabled }}
      volumes:
      - name: edgex-init
        persistentVolumeClaim:
          claimName: edgex-init
      - name: edgex-secrets
        persistentVolumeClaim:
          claimName: edgex-secrets
    {{- end}}

```
新建文件[edgex-device-camera-service.yaml](https://gitlab.4pd.io/meteor/aiot/product-deployment/-/blob/master/edgex-helm/templates/edgex-device-camera/edgex-device-camera-service.yaml)，内容为：
```yaml
# Copyright (C) 2022 Intel Corporation
#
# SPDX-License-Identifier: Apache-2.0
#
apiVersion: v1
kind: Service
metadata:
  labels:
    org.edgexfoundry.service: {{.Values.edgex.app.device.camera}}
  name: {{.Values.edgex.app.device.camera}}
spec:
  ports:
  - name: "http"
    port: {{.Values.edgex.port.device.camera}}
  selector:
    org.edgexfoundry.service: {{.Values.edgex.app.device.camera}}
  type: {{.Values.expose.type}}
```
然后增加device-camera相关内容，修改values.yml文件为：
```yaml
# Copyright (C) 2022 Intel Corporation
#
# SPDX-License-Identifier: Apache-2.0
#
# Default values for Edgex.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
expose:
  # Option value: ClusterIP/NodePort/LoadBalancer
  type: NodePort
# edgex defines a set of configuration properties for application-level concerns
edgex:
  # app defines a single point in naming/referring to an application. For each application, the value
  # define its label name, resource name or base of the resource name, and service name.
  app:
    core:
      command: edgex-core-command
      data: edgex-core-data
      metadata: edgex-core-metadata
    support:
      notifications: edgex-support-notifications
      scheduler: edgex-support-scheduler
    appservice:
      rules: edgex-app-rules-engine
    device:
      virtual: edgex-device-virtual
      rest: edgex-device-rest
      camera: edgex-device-camera
    ui: edgex-ui
    system: edgex-sys-mgmt-agent
    consul: edgex-core-consul
    redis: edgex-redis
    ekuiper: edgex-kuiper
    vault: edgex-vault
    bootstrapper: edgex-security-bootstrapper
    secretstoresetup: edgex-security-secretstore-setup
  # image defines configuration properties for docker-image-level concerns
  image:
    core:
      command:
        repository: edgexfoundry/core-command
        tag: "2.1.1"
        pullPolicy: IfNotPresent
      data:
        repository: edgexfoundry/core-data
        tag: "2.1.1"
        pullPolicy: IfNotPresent
      metadata:
        repository: edgexfoundry/core-metadata
        tag: "2.1.1"
        pullPolicy: IfNotPresent
    support:
      notifications:
        repository: edgexfoundry/support-notifications
        tag: "2.1.1"
        pullPolicy: IfNotPresent
      scheduler:
        repository: edgexfoundry/support-scheduler
        tag: "2.1.1"
        pullPolicy: IfNotPresent
    appservice:
      rules:
        repository: edgexfoundry/app-service-configurable
        tag: "2.1.1"
        pullPolicy: IfNotPresent
    device:
      virtual:
        repository: edgexfoundry/device-virtual
        tag: "2.1.1"
        pullPolicy: IfNotPresent
      rest:
        repository: edgexfoundry/device-rest
        tag: "2.1.1"
        pullPolicy: IfNotPresent
      camera:
        repository: edgexfoundry/device-camera
        tag: "2.1.0"
        pullPolicy: IfNotPresent
    ui:
      repository: edgexfoundry/edgex-ui
      tag: "2.1.0"
      pullPolicy: IfNotPresent
    system:
      repository: edgexfoundry/sys-mgmt-agent
      tag: "2.1.1"
      pullPolicy: IfNotPresent
    consul:
      repository: consul
      tag: "1.10.3"
      pullPolicy: IfNotPresent
    redis:
      repository: redis
      tag: "6.2.6-alpine"
      pullPolicy: IfNotPresent
    ekuiper:
      repository: lfedge/ekuiper
      tag: "1.4.4-alpine"
      pullPolicy: IfNotPresent
    vault:
      repository: vault
      tag: "1.8.5"
      pullPolicy: IfNotPresent
    bootstrapper:
      repository: edgexfoundry/security-bootstrapper
      tag: "2.1.0"
      pullPolicy: IfNotPresent
    secretstoresetup:
      repository: edgexfoundry/security-secretstore-setup
      tag: "2.1.0"
      pullPolicy: IfNotPresent
  # port defines configuration properties for container, target and host ports
  port:
    core:
      data: 59880
      metadata: 59881
      command: 59882
    support:
      notifications: 59860
      scheduler: 59861
    appservice:
      rules: 59701
    device:
      virtual: 59900
      rest: 59986
      camera: 59985
    system: 58890
    ui: 4000
    consul: 8500
    redis: 6379
    ekuiper: 59720
  # ports used by security bootstrapping for stage gating edgex init
  bootstrap:
    port:
      start: 54321
      readytorun: 54329
      secretstoretokensready: 54322
      databaseready: 54323
      registryready: 54324
      kongdbready: 54325
  # Duplicate default IP binding choice of docker-compose
  hostPortInternalBind: 127.0.0.1
  hostPortExternalBind: 0.0.0.0
  # replicas defines the number of replicas in a Deployment for the respective application
  replicas:
    core:
      command: 1
      data: 1
      metadata: 1
    support:
      notifications: 1
      scheduler: 1
    appservice:
      rules: 1
    device:
      virtual: 1
      rest: 1
      camera: 1
    ui: 1
    system: 1
    consul: 1
    redis: 1
    ekuiper: 1
  # UID/GID for container user
  security:
    enabled: false
    runAsUser: 2002
    runAsGroup: 2001
    tlsHost: edgex
  # resources defines the cpu and memory limits and requests for the respective application
  resources:
    core:
      command:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
      data:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
      metadata:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
    support:
      notifications:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
      scheduler:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
    appservice:
      rules:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
    device:
      virtual:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
      rest:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
      camera:
        enforceLimits: false
        limits:
          cpu: 1
          memory: 512Mi
        requests:
          cpu: 0.5
          memory: 256Mi
    ui:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    system:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    consul:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    redis:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    ekuiper:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    vault:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    bootstrapper:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
    secretstoresetup:
      enforceLimits: false
      limits:
        cpu: 1
        memory: 512Mi
      requests:
        cpu: 0.5
        memory: 256Mi
  storage:
    className: ""

```
再次helm upgrade即可。<br />安装完成后，可进入edgex-ui管理页面查看，如图：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658812585909-7215e916-a25b-44d6-9301-9f210a1ef0ef.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=240&id=uf707bc8d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=479&originWidth=1136&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68089&status=done&style=none&taskId=u1da5ad1e-7574-435b-adbe-68fa46a5cdc&title=&width=568)
<a name="dSSHU"></a>
### 添加设备到edgex
1）首先定义device-profile文件<br />实际上，安装完成后，该服务会自带三个profile文件（[https://github.com/edgexfoundry/device-camera-go/tree/main/cmd/res/profiles](https://github.com/edgexfoundry/device-camera-go/tree/main/cmd/res/profiles)），即：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658812957249-bd9abd8f-1e45-4fcc-8426-676c8d131326.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=62&id=ud73383f1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=123&originWidth=1216&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28856&status=done&style=none&taskId=u6fd579ae-fe03-4c25-98ed-25890f8d969&title=&width=608)<br />我们为了演示完整，自己做一个profile，名为demo-onvif.yaml,内容和camera.yaml一致。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813117057-4bb46ee2-c4c6-4174-8d96-2c74984e7a30.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=227&id=u6cfc02b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=454&originWidth=1420&originalType=binary&ratio=1&rotation=0&showTitle=false&size=95646&status=done&style=none&taskId=u852e49fa-480e-43c5-becc-87b8941fc15&title=&width=710)<br />点击add，将camera.yaml的内容粘贴到输入框，并修改其name为demo-onvif，并提交。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813236349-82083e69-5dfd-47b0-b6dc-3862d3a64e8a.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=361&id=u2a60ef93&margin=%5Bobject%20Object%5D&name=image.png&originHeight=721&originWidth=1427&originalType=binary&ratio=1&rotation=0&showTitle=false&size=140984&status=done&style=none&taskId=ua75a0ce0-1f0b-49b0-b4a6-0e7fd108fbc&title=&width=713.5)<br />此时，在device profile中就增加了刚刚添加的设备描述文件“demo-onvif”。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813298024-a6864086-dd00-428e-975b-b2232c1e0547.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=203&id=u767610e6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=406&originWidth=1432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93761&status=done&style=none&taskId=ufaae451e-1828-4676-9418-6ebf19ea62b&title=&width=716)<br />第二步，添加设备。进入device选项卡，点击“Add”<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813345783-b17a10b2-f23c-4cc6-b62b-9fa2bbc354fa.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=188&id=u61498d51&margin=%5Bobject%20Object%5D&name=image.png&originHeight=376&originWidth=1445&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91906&status=done&style=none&taskId=u7b1c139c-f64a-48b8-a40b-235e8d3c205&title=&width=722.5)<br />选择设备服务“device-camera”，下一步<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813381984-f1bc1fab-f895-4477-9d33-303f801580de.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=248&id=ub87122d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=496&originWidth=1429&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91592&status=done&style=none&taskId=u711a00cd-2c01-4d22-bf43-ce580770060&title=&width=714.5)选择描述文件“demo-onvif”，下一步<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813426261-39a6c173-88a9-4da3-b1fb-67026c9065f4.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=249&id=u5dc8f4ab&margin=%5Bobject%20Object%5D&name=image.png&originHeight=498&originWidth=1423&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106962&status=done&style=none&taskId=u94420664-735a-4032-b490-44b2252829e&title=&width=711.5)<br />填写设备名称“demo-onvif-1”,下一步，<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813461322-0302d3ec-3dbb-401a-be93-79a90454761b.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=213&id=ud4de20ce&margin=%5Bobject%20Object%5D&name=image.png&originHeight=426&originWidth=1405&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51587&status=done&style=none&taskId=uf7a5148b-9e15-4045-9cd1-ff4a2128fdf&title=&width=702.5)<br />填写自动采集事件的配置：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658813539702-9d669240-eef9-4b3b-996d-f938e1140eb6.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=277&id=u67fefc25&margin=%5Bobject%20Object%5D&name=image.png&originHeight=554&originWidth=1432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64235&status=done&style=none&taskId=u8e0f8348-162b-4b7b-8ffe-11e15dde0cf&title=&width=716)<br />这里说明一下，对于onvif协议有一个“onvifSnapshot”的方法，可以获得摄像头当前状态截图，但因为是我们使用的是试用版本，该截图为实际产品logo，因此我们选择了定期调用onvifStreamUri方法，即定期上报rtsp的地址。<br />最后一步，配置协议。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658814423774-ac70a25e-96e2-47de-af99-7d9d576e047f.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=183&id=uaee94a98&margin=%5Bobject%20Object%5D&name=image.png&originHeight=365&originWidth=1410&originalType=binary&ratio=1&rotation=0&showTitle=false&size=66689&status=done&style=none&taskId=u765b9f7d-17ba-4deb-83d4-ab1c296e530&title=&width=705)

首先，选择自定义协议模版，协议名称为“HTTP”（必须大写），增加三个属性，分别是：<br />Address：设备的IP:Port<br />AuthMethod,即认证方式，由于验证我们关闭了验证方式，故填none，如果需要认证，支持digest和usernamepassword，可参考：[https://github.com/edgexfoundry/device-camera-go/blob/main/cmd/res/devices/camera.toml](https://github.com/edgexfoundry/device-camera-go/blob/main/cmd/res/devices/camera.toml)<br />CredentialsPath，因为没有认证故保持了默认。<br />提交后，设备便添加完成。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658814088282-83c04a56-246a-4ae4-bb98-f12027aefc13.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=213&id=u690fc653&margin=%5Bobject%20Object%5D&name=image.png&originHeight=425&originWidth=1429&originalType=binary&ratio=1&rotation=0&showTitle=false&size=97440&status=done&style=none&taskId=ufe146d73-683a-4bbe-bf7d-dc92d58debd&title=&width=714.5)<br />点击图上command标识，验证是否连接成功。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658814456556-9be57c01-dbe8-4b1c-8191-61e1c031f500.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=uda620db3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=699&originWidth=982&originalType=binary&ratio=1&rotation=0&showTitle=false&size=111263&status=done&style=none&taskId=uff5fdf83-1cda-4adc-bf34-6cb73f0f845&title=&width=491)<br />如图，成功连接，并获得到了rtsp地址。如果你无法连接成功，点击报错，可以查看后台“edgex-device-camera”日志排除配置问题和设备问题。
<a name="hSxGl"></a>
### ekuper解析报文，对接下游解析程序
如刚才介绍，我们设置了一个每个15s采集设备视频流地址的event，可以在edgex-ui，datacenter模块查看，是否正常上报，如图：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658814724236-e5881ba8-4ee0-4279-8a18-3a6fd71d1c22.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=396&id=ud84973c5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=791&originWidth=1695&originalType=binary&ratio=1&rotation=0&showTitle=false&size=257862&status=done&style=none&taskId=ube90f3d8-50ff-49c3-b467-6789d6f0cd9&title=&width=847.5)<br />如图，可见采集工作正常。<br />下面，我们创建一个edgex流，并增加一个解析规则到ekuiper，利用ekuiper将解析后的地址发送到解析程序（也可以使用HTTP sink），可以使用控制台，也可以使用edgex客户端。由于edgex客户端有bug，建议直接通过ekuper manager 来配置。：<br />流：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658817518392-104131ae-6394-4ed9-8b8e-b54e6bae01c7.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=503&id=ub6d0dbcd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1006&originWidth=1728&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86470&status=done&style=none&taskId=uaa982cf8-3def-428f-aa71-19b0660007a&title=&width=864)<br />规则：
```yaml
SELECT "true" as isRetImg, (json_path_query(OnvifStreamURI, "$")->MediaUri)->Uri AS rtspUrl, meta(deviceName) as deviceName FROM EdgexStream where meta(profileName) = "demo-onvif"
```
在ekuiper命令行检查是否正确：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658816540785-10b20751-3f79-46dd-b839-7c8e6792b4a1.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=106&id=sOYkm&margin=%5Bobject%20Object%5D&name=image.png&originHeight=212&originWidth=1856&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60412&status=done&style=none&taskId=u087a5ae1-45ef-4e31-8e47-b24b6006c2b&title=&width=928)<br />在控制台配置：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658817273745-5e3e83d5-062b-440c-891f-11f8053442e3.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=587&id=u801b8662&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1174&originWidth=1774&originalType=binary&ratio=1&rotation=0&showTitle=false&size=85206&status=done&style=none&taskId=u444d62dc-cfd7-446c-a5eb-1205d229def&title=&width=887)<br />选择合适的sink，如rest:<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/28211224/1658817333355-3749bbea-2d26-4a6e-9ec5-02480b7f2c2d.png#clientId=ua6b29b05-b260-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=676&id=u6b1edb0f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1352&originWidth=926&originalType=binary&ratio=1&rotation=0&showTitle=false&size=109636&status=done&style=none&taskId=uc262464d-b4ef-490c-ae82-128c8cba8f3&title=&width=463)<br />配置的地址便是下游需要获得rstp流媒体的服务。
<a name="QjNbt"></a>
### 解析服务接收示例
onvif本质上来讲它是一个控制协议，因此，真的视频数据是由rtsp协议承担的，因此，解析服务要工作，实际上就是需要一个程序能够解析rtsp视频数据，一般来讲，对于cv领域opencv是很好的选择。笔者项目就是通过一个python flask 的web服务，实时接收流媒体地址，并通过opencv截图做内容识别。
```yaml
@app.route('/check_mask_rtsp', methods=['POST'])
def checkMaskRtsp():
    req = request.get_json()
    ret = check_mask_internal_rtsp(req)
    return jsonify(ret)

def check_mask_internal_rtsp(req):
    if isinstance(req, str):
        req = json.loads(req)
    isRetImg = req["isRetImg"]
    rtspUrl = req["rtspUrl"]
    deviceName = req["deviceName"]
    cap = cv2.VideoCapture(rtspUrl)
    if not cap.isOpened():
        print("Video open failed.")
        return
    status, img_raw = cap.read()
    ...
    #做推理识别
    ...
    ret = {}
    results = []
    for i in range(len(output[0])):
        result = {"mask": True if output[0][i][0] == 0 else False,
                  "confidence": output[0][i][1],
                  "position": {"left": output[0][i][2], "bottom": output[0][i][3], "right": output[0][i][2],
                               "top": output[0][i][4]}}
        results.append(result)
    ret["results"] = results
    ret["deviceName"] = deviceName
    if isRetImg:
        ret["renderImg"] = str(output[1])
    return ret
```
对于直接视频处理也可以：
```yaml
import cv2
import time
url = '${rtspUrl}'
fpsTime = time.time()
cap = cv2.VideoCapture(url)
while(cap.isOpened()):
    # Capture frame-by-frame
    ret, frame = cap.read()
    #做视频处理
    cv2.imshow('frame',frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```
至此，所有的介绍均介绍完毕，如有不清楚，可以留言讨论。
