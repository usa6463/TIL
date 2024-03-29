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

## Named Template
- 파일안에 명시된 탬플릿이나 이름이 주어짐. 
- 템플릿 관리를 위한 3가지 Action (함수가 아니다)
  - `define`: named template을 선언하는데 사용
    - ```
      {{/* Generate basic labels */}}
      {{- define "mychart.labels" }}
      labels:
      generator: helm
      date: {{ now | htmlDate }}
      {{- end }}
      ```
  - `template`: 선언된 named template를 불러오는데 사용
    - ```
      metadata:
      name: {{ .Release.Name }}-configmap
      {{- template "mychart.labels" . }}
      ```
    - 두번째 argument에는 scope를 전달해준다.
    - 사용중 indent가 생각지 못하게 들어갈 수 있다. 이건 대체되는 텍스트가 오른쪽 정렬이기 때문(?). 잘 이해는 안감
    - 대신 include 함수로 indent 제어하면서 named template 가져올 수 있음.
  - `block`
- `include` 함수
  - `template` action과 유사한 동작을 한다.
  - 단 가지고온 named template 내용에 대해 indent를 적용 가능함. 
    - ex. `{{ include "mychart.app" . | indent 2 }}`

- 주의사항
  - 템플릿 이름은 global
  - 동일한 이름의 template이 있다면 나중에 Load된 템플릿이 사용됨
    - 이를 막기위한 네이밍 컨벤션으로 chart 이름을 prefix로 붙이는게 유명함
- 템플릿 파일 네이밍 컨벤션
  - 대부분의 template 파일은 K8s manifest를 포함한다
  - NOTES.txt를 예외
  - 그리고 이름이 _로 시작하는 파일은 K8s manifest를 포함하지 않는다. 대신 partial, helper로써 다른 template에서 사용될 수 있다.

## NOTES.txt
- `helm install`이나 `helm upgrade`가 끝나면 사용자에게 출력되는 내용
- `/templates` 디렉토리 내에 포함된다
- 템플릿처럼 처리되며 모든 일반 템플릿 객체 접근 가능

## 파일 접근하기
- 탬플릿 렌더링 하는 것 외에, 파일 내용을 그대로 읽어올(inject) 수도 있다.
- `.Files` 객체를 통해 가능하나 일부 파일은 불가능하다
  - `/templates` 디렉토리 내 파일은 불가능
  - `.helmignore`로 인해 제외된 파일은 액세스 불가능 
- chart에 file을 추가해도 되지만 쿠버네티스 객체 저장소 크기에는 한계가 있어, 차트의 크기는 1M보단 작아야 한다
- 접근 방법
  - `.Files.Get 파일명`
- 경로 헬퍼
  - base, dir, ext, isAbs, clean
  - Go lang의 path 패키지라고 한다. helm 쪽 함수명 첫글자가 소문자인것만 다름.
  - file path에 대한 standard한 연산 제공
- Glob 패턴
  - `Files.Glob(pattern string)` 으로 글롭 패턴이라는 것을 통해 특정 파일만 추출할 수 있음.
- Lines
  - Lines 를 통해 파일의 각 라인을 리스트 같이 받아올 수 있음. => range와 결합해서 사용 가능 
    - ```
      data:
        some-file.txt: {{ range .Files.Lines "foo/bar.txt" }}
          {{ . }}{{ end }}
      ```

## 서브차트와 글로벌 값
- 서브차트에 대한 디테일
  - 서브차트는 부모 차트에 의존성을 가지지 않는다
  - 부모차트는 그래서 부모 차트의 values에 접근할 수 없다
  - 부모 차트는 서브 차트의 values를 Override할 수 있다.
  - 헬름은 모든 차트들이 접근 가능한 글로벌 values를 가질 수 있다. 
- 서브차트 values 오버라이딩 방법
  - 부모 차트의 values.yaml에서 서브차트의 이름으로 키를 만든다. 그리고 그 키 안의 values들은 서브차트에 오버라이딩 된다.
  - ex
    - ```yaml
      mysubchart:
        dessert: icecream
      ```
    - 위와 같이 부모차트의 values에 명시해두면, 서브차트에선 .Values.dessert 가 위 값으로 오버라이딩됨. 
  - 단 helm install 할 때 부모차트 기준으로 install 할때만 적용. 
- 글로벌 value
  - parent chart의 values.yaml에 global이라는 키를 만들고 그 안에 values들을 넣으면 부모, 서브 차트 관계 없이 모두 접근가능. {{ .Values.global.... }}
- named template의 경우 부모, 서브 차트 관계 없이 전역적으로 공유된다. 

## .helmignore 파일
- `helm package` 명령시 추가하지 않을 파일을 명시할 수 있다. 

## 디버깅
- `helm lint`: 코드가 모범 사례에 맞는지 검증 가능
- `helm install --debug --dry-run`, `helm template --debug` : 렌더링된 탬플릿 확인 가능
- `helm get manifest` : 배포된 helm chart 대상으로 어떤 Template들이 배포되었는지 확인 가능

## TIP
- 중괄호 사용시 {{-,  -}} 같이 하이픈을 붙이면 왼쪽 혹은 오른쪽의 공백을 제거 가능
- |- 는 multi line string 선언에 사용 가능