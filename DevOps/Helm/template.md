# Template

## basic
- chart 폴더 내에 `templates` 라는 이름의 디렉토리에 template 파일들을 구성
- helm은 `templates` 디렉토리 내 모든 파일을 렌더링 엔진으로 전송, 이후 처리 결과를 k8s로 보낸다
- 파일 내 `{{ ... }}`로 둘러싼 코드에 대해 렌더링이 수행된다. 

## directory structure
- `templates` 디렉토리 내에 여러 파일이 존재
- `NOTES.txt`: `helm install` 시 사용자에게 표시
- `*.yaml` 쿠버네티스 오브젝트를 생성하기 위한 각종 매니페스트
- `_helpers.tpl`: 차트 전체에 다시 사용할 수 있는 템플릿 헬퍼를 지정하는 공간

## 빌트인 객체
- 탬플릿에서 접근 가능한 객체. 예를들어 `{{ .Release.Name }}` 이런식으로 접근 가능
- `Release`: 릴리즈 이름, 네임스페이스, revision 번호 등
- `Values`: values.yaml 파일에서 전달되는 값들에 접근 가능
- `Files`: 차트 내 특수하지 않은 파일들에 대한 접근
- `Capabilities`: 클러스터가 지원하는 기능에 대한 정보 제공 (API 버전, k8s 버전 등)
- `Templates`: 현재 템플릿에 대한 정보 제공 (현재 템플릿에 대한 파일 경로 등)

## Values 파일
- 렌더링시 적용되는 value 우선 순위. 위에 있을수록 우선순위가 높다(우선순위가 높으면 낮은 경우를 덮어 쓴다.)
  - `--set`에 명시된 내용
  - `-f values.yaml` 와 같이 지정한 values.yaml에 명시된 내용
  - 대상이 sub chart일 경우 parent chart의 `values.yaml`
  - 차트 내의 `values.yaml`
- key를 삭제하는 방법 중 하나로 value를 null로 주는 방법이 있다. ex. `--set livenessProbe.httpGet=null`

## 템플릿 함수
- 탬플릿 내에서 쓸 수 있는 함수
- 항목이 많으므로 [문서](https://helm.sh/ko/docs/chart_template_guide/function_list/)를 참고하면서 사용하라
- ex. `{{ lower "HELLO" }}`

## 파이프라인
- linux shell에서의 파이프라인과 동일한 기능
- argument가 많은 함수의 경우 파이프라인으로 넘어온 값은 마지막 argument로 들어간다.
- ex. `{{ .Values.favorite.drink | repeat 5 | quote }}`

## 흐름 제어
- `if/else`: 프로그래밍 언어의 if-else와 동일
- `with`: 매번 .Values... 로 타고 들어가 values 값을 참조한다던지 하는건 코드상 반복이 많을 수 있음. 미리 scope를 지정할 때 사용
- `range`: 프로그래밍 언어의 for each와 동일. 리스트, tuple, key-value 다 가능
- 사용 뒤엔 {{ end }} 를 붙일 것.

## 변수
- `$변수명` 같은 형태로 값 할당 및 접근 가능
- 값 할당시엔 := 를 사용 
  - ex. `{{- $relname := .Release.Name -}}`

## TIP
- 중괄호 사용시 {{-,  -}} 같이 하이픈을 붙이면 왼쪽 혹은 오른쪽의 공백을 제거 가능
- |- 는 multi line string 선언에 사용 가능