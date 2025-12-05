# 전자정부 프레임워크 학습

전자정부 프레임워크와 Spring MVC의 핵심 원리부터 실전 계층형 게시판 구현까지 학습한 내용을 정리한 프로젝트입니다.

## 개발 환경

*   **Framework**: eGovFramework 3.10.0, Spring Framework 5.3.6
*   **Database**: PostgreSQL (v42.3.8 Driver)
*   **ORM**: MyBatis 3.5.6
*   **WAS**: Apache Tomcat
*   **Build**: Maven

## 학습 내용 요약

이 프로젝트는 다음의 단계별 학습 내용을 포함하고 있습니다.

### [1. 기본기 다지기](docs/1_기본기_다지기.md)
*   **프로젝트 구조**: `src/main/java`, `src/main/resources`, `src/main/webapp` 등 표준 디렉토리 구조 이해.
*   **Spring MVC 동작 원리**: DispatcherServlet, HandlerMapping, Controller, ViewResolver의 흐름.
*   **데이터 전달**:
    *   Controller -> JSP: `Model` 객체 사용.
    *   JSP -> Controller: Command Object (VO) 사용.

### [2. DB와 친해지기](docs/2_DB와_친해지기.md)
*   **PostgreSQL 연동**: `context-datasource.xml`, `context-mapper.xml` 설정.
*   **Layered Architecture**: Controller -> Service -> DAO -> Mapper -> DB 흐름 구현.
*   **로그인/로그아웃**: HttpSession을 이용한 로그인 상태 관리.
*   **저장 프로시저**: MyBatis에서 PostgreSQL 함수 호출 방법.

### [3. 고급 기능 및 유틸리티](docs/3_고급_기능_및_유틸리티.md)
*   **Logging (Log4j2)**: `log4j2.xml` 설정 및 `slf4j`를 이용한 로그 레벨 관리 (DEBUG, INFO, ERROR).
*   **AOP (관점 지향 프로그래밍)**:
    *   핵심 로직과 공통 기능(로그, 트랜잭션 등) 분리.
    *   `@Aspect` 어노테이션을 이용한 AOP 구현.

### [4. 계층형 게시판](docs/4_실전_계층형_게시판.md)
*   **DB 설계**: `GROUP_NUM`, `GROUP_ORDER`, `GROUP_TAB` 컬럼을 이용한 계층 구조 설계.
*   **답글 로직**:
    *   부모 글의 정보를 바탕으로 자리 확보(`UPDATE`) 후 삽입(`INSERT`).
*   **수정 및 삭제**: 작성자 본인 확인 및 논리적 삭제(`UPDATE`) 구현.
*   **페이징**: `LIMIT`, `OFFSET`을 이용한 효율적인 페이징 처리.

### [5. 확장 기능](docs/5_확장_기능.md)
*   **파일 업로드**: `MultipartResolver` 및 UUID를 이용한 파일 저장.
*   **트랜잭션**: `@Transactional` 및 AOP 설정을 통한 데이터 무결성 보장.
*   **Ajax**:
    *   **JSON**: `@ResponseBody`와 Jackson 라이브러리 활용.
    *   **XML**: JDOM을 이용한 XML 응답 생성 및 파싱.
