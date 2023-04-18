# NextJS Basic

## keyword
- client-side navigation
  - 같은 페이지에서 특정 부분만 re-rendering 하는 것
- Parallel Routes
  - 한 개 이상의 페이지를 동시에 렌더링
  - 이 페이지들은 같은 view에서 독립적으로 이동 가능
  - dynamic한 섹션, 가령 대시보드, 소셜 사이트의 피드 페이지
  - @folder convention 사용
- Intercepting Routes
  - 현재 레이아웃에서 브라우저 URL을 마스킹하는 동안, 새로운 route를 로드하는 것
  - 현재 페이지의 컨텍스트가 중요할 때 사용
  - 가령 하나의 태스크를 수정하는 동안 모든 태스크 보기, 사이드 모달에서 카트 열기, 피드에서 사진 확대
  - 폴더명에 (..)를 prefix로 하는 컨벤션
- Rendering Fundamentals
  - Rendering Environments
    - 흔히 아는 client, server에 대한 설명
  - Component-level Client and Server Rendering
    - React 18 이전엔 React에서 렌더링 방법으로 클라이언트만 지원
    - Next.js는 애플리케이션을 pages로 나누고, 서버에서 prerender 하는 방법을 제공
      - 대신 이건 추가적인 자바스크립트가 클라이언트에게 필요하게 됨.(초기 HTML interactive를 위해)
    - 이제 React는 컴포넌트 레벨에서 렌더링 환경의 선택을 지원함(클라이언트 Or 서버)
    - 컴포넌트들은 트리 형태로 관계가 구성되는데, 서버 컴포넌트 자식으로 클라 컴포넌트가 들어간다던지, 반대도 가능
  - Static and Dynamic Rendering on the Server
    - Next.js는 서버에서 렌더링을 최적화 하는 옵션을 제공. static과 dynamic
    - static
      - 서버 빌드 타임에 컴포넌트들(클라, 서버 컴포넌트)이 프리 렌더 됨. 
    - dynamic
      - request 타임에 컴포넌트들(클라, 서버 컴포넌트)이 렌더링됨
  - Edge and Node.js Runtimes
    - 서버에서 페이지가 렌더될 수 있는 런타임 2가지가 존재
    - Node.js
      - 디폴트 설정
      - 모든 Node.js API, 에코시스템의 호환되는 패키지들에 접근 가능
    - Edge
      - 웹 API 기반
  - 
- 