---
layout: post
title: "Argo Workflows в EKS: коли CronJobs вже не вивозять"
date: 2026-05-26 20:10:00 +0200
categories: [devops]
tags: [kubernetes, eks, argo-workflows, argocd, dex, ldap]
featured: false
preview_image: /assets/stickers/tools/argo.svg
case_type: change-case
system_stage: existing
---

Argo Workflows я, чесно кажучи, не планував чіпати. Але розробники дуже хочуть, а я і не проти. Тож наливаємо ~~пиво~~чай і йдемо робити дивні дива.

Суть проста, у нас є CronJobs у Kubernetes, і команді розробників хочеться нормальну веб-морду, де можна руками запускати задачі, дивитися історію запусків і не клікати по 15 вкладках. Звісно частину можна закрити через Grafana(шо вже зроблено) + ручні тригери з того ж самого ArgoCD, але це той випадок, коли "можна", але нам не "зручно". Тому після недовгого сумісного R&D зупинились на Argo Workflows.

## Встановлення чарта {#install-chart}

Є кластер EKS (Kubernetes 1.35), є ArgoCD, тож ставимо Argo Workflows через Helm.

І так додаємо репозиторій чарту.

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

Далі глянемо що у нас по версіях чарту.
```bash
helm search repo argo/argo-workflows --versions | head -20

NAME               	CHART VERSION	APP VERSION	DESCRIPTION                    
argo/argo-workflows	1.0.14       	v4.0.5     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.13       	v4.0.5     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.12       	v4.0.5     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.11       	v4.0.4     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.10       	v4.0.4     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.9        	v4.0.4     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.8        	v4.0.4     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.7        	v4.0.4     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.6        	v4.0.3     	A Helm chart for Argo Workflows
argo/argo-workflows	1.0.5        	v4.0.3     	A Helm chart for Argo Workflows
...
```

Ну і далі, як ви здогадались зберігаємо дефолтніе values в файл, або якщо дуже хочется можна скачати в репо чарту [ArgoWorkflow](https://github.com/argoproj/argo-helm/tree/main/charts/argo-workflows)

```bash
helm show values argo/argo-workflows \
  --version "1.0.14" \
  > values.yaml
```
Версія `1.0.14` тут як приклад з мого кейсу. Якщо читаєте це пізніше, просто беріть актуальну з `helm search`.

## Налаштування контролера {#controller-config}

Далі проводимо базові налаштування під наші потреби. Я не хотів обмежувати контролер лише namespace релізу, тому ставимо так.

```yaml
singleNamespace: false

controller:
  workflowNamespaces: []
```

Так, `workflowNamespaces: []`  це свідомо "широко" на старті. Для production краще одразу задати явний список namespace.

### Ingress для UI {#ingress-ui}

Щоб у UI взагалі можна було зайти, піднімаємо ingress. В нашому випадку `ingressClassName: "haproxy"`

```yaml
server:
  ingress:
    enabled: true
    annotations:
      blackbox.io/enabled: "true"
      haproxy.org/allow-list: "<CIDR1>,<CIDR2>,..."
    ingressClassName: "haproxy"
    hosts:
      - argo-workflows.paparoot.net
    paths:
      - /
    pathType: Prefix
    tls:
      - secretName: argo-workflows-server-tls
        hosts:
          - argo-workflows.paparoot.net
```
Тут все по простому, накинули в анотації ACL, моніторинг, TLS.

### SSO через Dex + LDAP {#sso-dex-ldap}

Тут починається найцікавіша частина цього квесту, А - Авторизація. Так як у нас є LDAP FreeIPA то за правилами доброго тону хотілося б щоб використовували саме його, але у Argo Workflows з коробки немає прямого LDAP login, тому робимо обхідний, але нормальний шлях: `Argo Workflows -> Dex (в ArgoCD) -> LDAP -> назад OIDC token`.

Фрагмент Dex-конфіга в ArgoCD.

```yaml
configs:
  cm:
    dex.config: |
      staticClients:
      - id: argo-workflows-sso
        name: Argo Workflows
        secret: "ARGO_WORKFLOWS_SSO_SECRET"
        redirectURIs:
        - https://argo-workflows.paparoot.net/oauth2/callback
      connectors:
      - type: ldap
        id: ldap
        name: LDAP
        config:
          host: ipa.paparoot.net:636
```

І конфіг самого Argo Workflows:

```yaml
server:
  authModes:
    - sso
  sso:
    enabled: true
    issuer: https://argocd.paparoot.net/api/dex
    redirectUrl: "https://argo-workflows.paparoot.net/oauth2/callback"
    clientId:
      name: argo-server-sso
      key: client-id
    clientSecret:
      name: argo-server-sso
      key: client-secret
    scopes:
      - openid
      - profile
      - email
      - groups
```

І ще момент. `clientId`/`clientSecret` у цьому конфігу посилаються на Kubernetes Secret `argo-server-sso`. Тобто цей секрет потрібно створити окремо, інакше SSO просто не злетить. В мене цей сікрет лежить окремим маніфестом у репозиторії.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: argo-server-sso
  namespace: argocd
type: Opaque
stringData:
  client-id: argo-workflows-sso
  client-secret: "ARGO_WORKFLOWS_SSO_SECRET"
```

### RBAC мапінг груп {#rbac-group-mapping}

Далі мапимо LDAP-групи через `extraObjects`. Беремо групи, які вже створені під ArgoCD, і робимо під них `ServiceAccount`, `Secret`, `ClusterRole` та `ClusterRoleBinding`.

Логіка цього \"велосипеда\" така. Користувач логіниться через LDAP/Dex, Argo Workflows читає `groups` claim з OIDC token, підбирає відповідний `ServiceAccount` за правилом `workflows.argoproj.io/rbac-rule` і застосовує його права. Якщо матчів кілька, спрацьовує `workflows.argoproj.io/rbac-rule-precedence` (пріоритет вибору). 
Окремо важливий момент. `*.service-account-token` secret тут створюємо свідомо, бо без нього легко зловити `failed to get service account secret`. Argo очікує token-secret для обраного SA з іменем `<sa>.service-account-token`.

Ось робочий приклад `extraObjects`.

```yaml
extraObjects:
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: argo-workflows-sso-admin
      namespace: argocd
      annotations:
        workflows.argoproj.io/rbac-rule: "'argocd' in groups || 'cn=argocd,cn=groups,cn=accounts,dc=paparoot,dc=net' in groups || 'argocd-analytics' in groups || 'cn=argocd-analytics,cn=groups,cn=accounts,dc=paparoot,dc=net' in groups"
        workflows.argoproj.io/rbac-rule-precedence: "100"
  - apiVersion: v1
    kind: Secret
    metadata:
      name: argo-workflows-sso-admin.service-account-token
      namespace: argocd
      annotations:
        kubernetes.io/service-account.name: argo-workflows-sso-admin
    type: kubernetes.io/service-account-token
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: argo-workflows-sso-readonly
      namespace: argocd
      annotations:
        workflows.argoproj.io/rbac-rule: "'argocd-analytics-ro' in groups || 'cn=argocd-analytics-ro,cn=groups,cn=accounts,dc=paparoot,dc=net' in groups"
        workflows.argoproj.io/rbac-rule-precedence: "10"
  - apiVersion: v1
    kind: Secret
    metadata:
      name: argo-workflows-sso-readonly.service-account-token
      namespace: argocd
      annotations:
        kubernetes.io/service-account.name: argo-workflows-sso-readonly
    type: kubernetes.io/service-account-token
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argo-workflows-sso-admin
    rules:
      - apiGroups: [""]
        resources: ["pods", "pods/log", "events", "configmaps", "secrets", "serviceaccounts"]
        verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
      - apiGroups: ["batch"]
        resources: ["jobs", "cronjobs"]
        verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
      - apiGroups: ["argoproj.io"]
        resources: ["workflows", "workflowtaskresults", "workflowtemplates", "cronworkflows", "clusterworkflowtemplates"]
        verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
      - apiGroups: ["argoproj.io"]
        resources: ["eventsources", "sensors", "eventbuses"]
        verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
      - apiGroups: ["argoproj.io"]
        resources: ["workflows/finalizers", "workflowtemplates/finalizers", "cronworkflows/finalizers", "clusterworkflowtemplates/finalizers"]
        verbs: ["update"]
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: argo-workflows-sso-admin
    subjects:
      - kind: ServiceAccount
        name: argo-workflows-sso-admin
        namespace: argocd
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: argo-workflows-sso-admin
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argo-workflows-sso-readonly
    rules:
      - apiGroups: [""]
        resources: ["pods", "pods/log", "events"]
        verbs: ["get", "list", "watch"]
      - apiGroups: ["argoproj.io"]
        resources: ["workflows", "workflowtemplates", "cronworkflows", "clusterworkflowtemplates"]
        verbs: ["get", "list", "watch"]
      - apiGroups: ["argoproj.io"]
        resources: ["eventsources", "sensors", "eventbuses"]
        verbs: ["get", "list", "watch"]
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: argo-workflows-sso-readonly
    subjects:
      - kind: ServiceAccount
        name: argo-workflows-sso-readonly
        namespace: argocd
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: argo-workflows-sso-readonly
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argo-workflows-server-sso-rbac-reader
    rules:
      - apiGroups: [""]
        resources: ["serviceaccounts", "secrets"]
        verbs: ["get", "list", "watch"]
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: argo-workflows-server-sso-rbac-reader
    subjects:
      - kind: ServiceAccount
        name: argo-workflows-server
        namespace: argocd
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: argo-workflows-server-sso-rbac-reader
```

Ключові моменти.
- `workflows.argoproj.io/rbac-rule` — правило відповідності груп;
- `workflows.argoproj.io/rbac-rule-precedence` — пріоритет вибору;
- `*.service-account-token` secret — потрібен, інакше можна словити `failed to get service account secret`.

Для прикладу робимо окремі ролі під admin і readonly, плюс binding для `argo-workflows-server`, щоб він міг читати потрібні `serviceaccounts/secrets`.

### Артефакти в S3 через IRSA {#s3-irsa-artifacts}

Для зберігання output-файлів workflow та archived logs підключаємо S3 як сховище артефактів. IRSA role і policy до неї роблю через Terraform, але якщо робите це руками, можна скористатись офіційною документацією: [AWS EKS IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). Використовуємо саме IRSA, а не статичні ключі. `useStaticCredentials: false` і `useSDKCreds: true` означають, що беремо AWS credentials із IAM role pod'ів.

IAM роль прив'язуємо через `eks.amazonaws.com/role-arn` на `workflow/controller/server` service accounts. У результаті pod'и отримують тимчасові AWS credentials через OIDC/STS. `archiveLogs: true` вмикає зберігання логів як артефактів у S3, а `keyFormat` задає зрозумілу ієрархію `<namespace>/<workflow>/<pod>`.

```yaml
workflow:
  serviceAccount:
    create: true
    name: "argo-workflow"
    annotations:
      eks.amazonaws.com/role-arn: "ARGO_WORKFLOWS_ARTIFACTS_IAM_ROLE_ARN"

controller:
  serviceAccount:
    create: true
    annotations:
      eks.amazonaws.com/role-arn: "ARGO_WORKFLOWS_ARTIFACTS_IAM_ROLE_ARN"

server:
  serviceAccount:
    create: true
    annotations:
      eks.amazonaws.com/role-arn: "ARGO_WORKFLOWS_ARTIFACTS_IAM_ROLE_ARN"

useStaticCredentials: false

artifactRepository:
  archiveLogs: true
  s3:
    bucket: eks-s3-argo-workflows-bucket
    endpoint: s3.eu-central-1.amazonaws.com
    region: eu-central-1
    useSDKCreds: true
    keyFormat: "{{workflow.namespace}}/{{workflow.name}}/{{pod.name}}"
```

Після цього ставимо Argo Workflows з нашим `values.yaml`.

```bash
helm upgrade --install argo-workflows argo/argo-workflows \
  --namespace argocd \
  --version "1.0.14" \
  -f values.yaml
```

## Перевірка в UI {#ui-check}

Далі заходимо по домену з Ingress (DNS сподіваюсь уже прописаний), логінимось через LDAP і отримуємо той самий довгоочікуванний веб-інтерфейс.

![Argo Workflows UI]({{ '/assets/img/posts/argo-workflows-v-eks/argo-workflows-ui-login.png' | relative_url }})

## Тестовий CronWorkflow {#test-cronworkflow}

І так, тепер тестуємо `CronWorkflow`, щоб перевірити не тільки UI, а й весь ланцюг. CRD, контролер, `ServiceAccount`, IRSA, S3-артефакти і базову поведінку history/TTL.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: test-cronworkflow
  namespace: argocd
spec:
  schedules:
    - "*/2 * * * *"
  timezone: "UTC"
  suspend: true
  concurrencyPolicy: "Forbid"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  workflowSpec:
    serviceAccountName: argo-workflow
    entrypoint: main
    ttlStrategy:
      secondsAfterSuccess: 3600
      secondsAfterFailure: 3600
    templates:
      - name: main
        container:
          image: alpine:3.20
          command: ["/bin/sh", "-c"]
          args:
            - |
              set -eu
              mkdir -p /tmp/artifacts
              echo "hello from test-cronworkflow" > /tmp/artifacts/result.txt
              date -u +"%Y-%m-%dT%H:%M:%SZ" >> /tmp/artifacts/result.txt
              cat /tmp/artifacts/result.txt
        outputs:
          artifacts:
            - name: result
              path: /tmp/artifacts/result.txt
```

Приміняємо і перевіряємо.

```bash
kubectl apply -f test-cronworkflow.yaml
```

```bash
kubectl get cronworkflow -n argocd

NAME                AGE
test-cronworkflow   85s
```

```bash
kubectl describe cronworkflow test-cronworkflow -n argocd

Name:         test-cronworkflow
Namespace:    argocd
Labels:       <none>
Annotations:  cronworkflows.argoproj.io/last-used-schedule: CRON_TZ=UTC */2 * * * *
API Version:  argoproj.io/v1alpha1
Kind:         CronWorkflow
Metadata:
  Creation Timestamp:  2026-05-27T15:31:30Z
  Generation:          2
  Resource Version:    85605963
  UID:                 7d4a3606-00cc-44e9-a5b6-f6fd83d47696
Spec:
  Concurrency Policy:         Forbid
  Failed Jobs History Limit:  3
  Schedules:
    */2 * * * *
  Successful Jobs History Limit:  3
  Suspend:                        true
  Timezone:                       UTC
  Workflow Spec:
    Entrypoint:            main
    Service Account Name:  argo-workflow
    Templates:
      Container:
        Args:
          set -eu
mkdir -p /tmp/artifacts
echo "hello from test-cronworkflow" > /tmp/artifacts/result.txt
date -u +"%Y-%m-%dT%H:%M:%SZ" >> /tmp/artifacts/result.txt
cat /tmp/artifacts/result.txt

        Command:
          /bin/sh
          -c
        Image:  alpine:3.20
      Name:     main
      Outputs:
        Artifacts:
          Name:  result
          Path:  /tmp/artifacts/result.txt
    Ttl Strategy:
      Seconds After Failure:  3600
      Seconds After Success:  3600
Status:
  Failed:     0
  Phase:      
  Succeeded:  0
Events:       <none>
```

У UI бачимо `CronWorkflow` з іконкою паузи (бо `suspend: true`), відкриваємо його, тиснемо `SUBMIT` і чекаємо run.

![CronWorkflow in UI]({{ '/assets/img/posts/argo-workflows-v-eks/argo-workflows-ui-cronworkflow.png' | relative_url }})

Після успішного run дивимось, чи артефакти реально потрапили в S3:

![S3 artifacts]({{ '/assets/img/posts/argo-workflows-v-eks/argo-workflows-s3-artifacts.png' | relative_url }})

Ну ще через aws cli перевіримо бо ми не довірливі.
```bash
aws s3 ls s3://eks-s3-argo-workflows-bucket/argocd/test-cronworkflow-<run-id>/test-cronworkflow-<run-id>/

2026-05-27 17:35:42         50 main.log
2026-05-27 17:35:41        156 result.tgz
```

На цьому етапі вже видно, що все працює як планувалось: workflow успішно відпрацьовує, логи архівуються, а артефакти реально потрапляють у S3.

Можна ще підкрутити `keyFormat`, щоб не дублювались директорії. Але то відполіруєм потім.

## Примітки {#notes}

Тут зібрав короткі практичні моменти, які найчастіше вилазять уже після першого запуску.

З важливого. Якщо workflow запускається не в тому namespace, де правильно налаштований `ServiceAccount` для IRSA, він або не стартує нормально, або падає на роботі з артефактами.

Що має бути обов'язково.
- у namespace запуску є `ServiceAccount`, який вказаний у `workflowSpec.serviceAccountName`;
- у цього SA є анотація `eks.amazonaws.com/role-arn`;
- IAM роль за цим ARN має права на потрібний S3 bucket/prefix.

Якщо цього нема, у логах workflow/controller зазвичай бачимо `AccessDenied` або `NoCredentialProviders`. У UI це часто виглядає як failed run або відсутність очікуваних артефактів після запуску.

Також всі наші Workflows можна деплоїти як звизчайні маніфести через ArgoCD.

## Висновок {#summary}

Argo Workflows у цьому кейсі зайшов як треба. Ручні запуски перестають бути шаманським ритуалом, історія запусків лежить на місці, доступи не розповзаються в хаос, а команда менше страждає в стилі "хто, де і чому це запускав о 04:20, воно запускалось за розкладом чи ні, та де DevOps який знеає де подивтись логи?".
