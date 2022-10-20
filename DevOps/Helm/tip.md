# Concept

## Chart
Helm package. k8s 내에 배포할 애플리케이션, 서비스, 툴들이 모두 포함되어 있음. Homebrew fomula, Apt의 dpkg같은 존재

## Repository
차트가 모여있고 공유되는 장소.

## Release
Chart를 install할 때마다 생김. java로 치면 클래스를 통해 만들어진 객체임.

# TIP
- chart를 다운받기 위해선 helm repo 추가가 필요하다
  - ex) `helm repo add bitnami https://charts.bitnami.com/bitnami`
- 한 클러스터에서 helm install을 여러번 할 때 마다 새로운 release가 생성됨. 각 release는 독립적임. 기존 Release를 업데이트할 목적이라면 `helm upgrade` 사용할것
- `helm -h` 를 사용하면 사용할 수 있는 커맨드를 알 수 있다. `helm {command} -h`를 할 경우 해당 커맨드의 사용법을 자세히 알 수 있음.


# values 변경

## helm install 하기전에 values 값을 보고 싶을때
`helm show values` 사용하면 됨. 

ex) `helm show values bitnami/airflow`

## values 값 override 하는 방법
아래와 같이 설정 중 일부분만 -f(--values) 로 넘겨주면 설정 일부분만 override 되고 나머지 설정은 default를 따른다. 
```bash
$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name
```

-f 뒤에는 여러개의 파일들을 명시할 수 있는데, 가장 오른쪽 파일이 우선순위가 가장 높다. 

--set 명령어로 파일없이 커맨드라인에서 바로 설정 override할 수 있는데, --set이랑 -f 같이 쓰면 --set이 더 우선순위 높음.   


# Reference
- https://helm.sh/docs/intro/using_helm/
