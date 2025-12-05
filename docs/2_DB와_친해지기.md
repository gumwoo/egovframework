# 2. DB와 친해지기 (5강 ~ 7강)

이 문서는 **데이터베이스(PostgreSQL)** 연동, **로그인** 처리, 그리고 **저장 프로시저** 호출에 대한 상세한 내용을 다룹니다.

---

## 1. 데이터베이스 연동 (5강)
Oracle 대신 **PostgreSQL**을 사용하는 환경을 기준으로 설명합니다.

### 1.1 라이브러리 추가 (`pom.xml`)
DB 연결(`jdbc`)과 SQL 매핑(`mybatis`), 그리고 커넥션 풀(`commons-dbcp2`) 라이브러리가 필요합니다.

```xml
<!-- PostgreSQL JDBC Driver -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.3.8</version>
</dependency>

<!-- MyBatis (SQL Mapper) -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>

<!-- Connection Pool (DBCP2) -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.9.0</version>
</dependency>
```

### 1.2 접속 정보 설정 (`context-datasource.xml`)
Spring에게 "이 DB에 접속해라"라고 알려주는 설정입니다.

```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="org.postgresql.Driver"/>
    <property name="url" value="jdbc:postgresql://localhost:5432/postgres"/>
    <property name="username" value="postgres"/>
    <property name="password" value="password"/>
</bean>
```

### 1.3 MyBatis 설정 (`context-mapper.xml`)
MyBatis가 SQL 파일(`*.xml`)을 어디서 찾을지, 어떤 규칙을 따를지 설정합니다.

```xml
<!-- SqlSessionFactory: MyBatis의 핵심 공장 -->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!-- Mapper XML 파일 위치 지정 -->
    <property name="mapperLocations" value="classpath:/egovframework/sqlmap/example/mappers/*.xml" />
    <!-- 카멜케이스 자동 변환 (user_id -> userId) 등 설정 파일 -->
    <property name="configLocation" value="classpath:/egovframework/sqlmap/example/sql-map-config.xml" />
</bean>

<!-- MapperScanner: DAO 인터페이스를 자동으로 찾아서 구현체 생성 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="egovframework.example.sample.service.impl" />
</bean>
```

---

## 2. Layered Architecture (계층형 아키텍처)
전자정부 프레임워크의 표준 개발 패턴입니다. 데이터가 흘러가는 순서를 반드시 기억해야 합니다.

**흐름: Controller -> Service -> DAO -> Mapper(XML) -> DB**

### 2.1 Controller (`EgovSampleController.java`)
요청을 받고 Service에게 일을 시킵니다.
```java
@Controller
public class EgovSampleController {
    
    @Resource(name = "sampleService")
    private EgovSampleService sampleService;

    @RequestMapping("/sampleList.do")
    public String selectSampleList(Model model) throws Exception {
        // Service 호출
        List<?> list = sampleService.selectSampleList();
        model.addAttribute("resultList", list);
        return "sample/egovSampleList";
    }
}
```

### 2.2 Service (`EgovSampleServiceImpl.java`)
비즈니스 로직을 처리하고 DAO를 호출합니다. (인터페이스와 구현체로 나뉩니다)
```java
@Service("sampleService")
public class EgovSampleServiceImpl implements EgovSampleService {

    @Resource(name = "sampleDAO")
    private SampleDAO sampleDAO;

    @Override
    public List<?> selectSampleList() throws Exception {
        return sampleDAO.selectSampleList();
    }
}
```

### 2.3 DAO (`SampleDAO.java`)
MyBatis Mapper 인터페이스입니다. 메소드 이름이 XML의 ID와 일치해야 합니다.
```java
@Mapper("sampleDAO")
public interface SampleDAO {
    // XML의 <select id="selectSampleList"> 와 매핑됨
    List<?> selectSampleList() throws Exception;
}
```

### 2.4 Mapper XML (`Sample_SQL.xml`)
실제 SQL을 작성하는 곳입니다.
```xml
<mapper namespace="egovframework.example.sample.service.impl.SampleDAO">
    
    <select id="selectSampleList" resultType="egovMap">
        SELECT
            ID,
            NAME,
            DESCRIPTION,
            USE_YN,
            REG_USER
        FROM SAMPLE
        ORDER BY ID ASC
    </select>

</mapper>
```

---

## 3. 로그인 (6강)
웹은 기본적으로 **Stateless(무상태)**입니다. 페이지를 이동하면 이전 정보를 까먹습니다.
그래서 **Session(세션)**이라는 서버 쪽 메모리에 로그인 정보를 저장해야 합니다.

### 3.1 로그인 로직 (Controller)
```java
@RequestMapping("/loginAction.do")
public String loginAction(
        @RequestParam("id") String id,
        @RequestParam("password") String password,
        HttpServletRequest request) throws Exception {

    // 1. DB에서 사용자 확인 (Service -> DAO -> DB)
    LoginVO user = loginService.selectUser(id, password);

    if (user != null) {
        // 2. 로그인 성공: 세션에 사용자 정보 저장
        // request.getSession()은 세션이 없으면 새로 만들고, 있으면 가져옴
        request.getSession().setAttribute("LoginVO", user);
        return "redirect:/main.do";
    } else {
        // 3. 로그인 실패
        return "redirect:/login.do?error=true";
    }
}
```

### 3.2 로그아웃 로직
```java
@RequestMapping("/logout.do")
public String logout(HttpServletRequest request) {
    // 세션 무효화 (저장된 정보 싹 날림)
    request.getSession().invalidate();
    return "redirect:/login.do";
}
```

### 3.3 로그인 체크 (JSP)
```jsp
<!-- 세션에 LoginVO가 없으면 로그인 페이지로 튕겨내기 -->
<c:if test="${sessionScope.LoginVO == null}">
    <script>
        alert("로그인이 필요합니다.");
        location.href = "/login.do";
    </script>
</c:if>

<!-- 로그인한 사람 이름 보여주기 -->
<p>환영합니다, ${sessionScope.LoginVO.name}님!</p>
```

---

## 4. 저장 프로시저 호출 (7강)
복잡한 로직을 DB 함수(Function)로 만들어두고, 자바에서 호출하는 방법입니다.
PostgreSQL은 Oracle의 프로시저와 달리 `FUNCTION`을 사용하며, 호출 방식이 조금 다릅니다.

### 4.1 PostgreSQL 함수 생성
```sql
CREATE OR REPLACE FUNCTION fn_get_user_name(p_user_id VARCHAR)
RETURNS VARCHAR AS $$
DECLARE
    v_user_name VARCHAR;
BEGIN
    SELECT name INTO v_user_name
    FROM users
    WHERE id = p_user_id;
    
    RETURN v_user_name;
END;
$$ LANGUAGE plpgsql;
```

### 4.2 MyBatis 호출 (`Mapper XML`)
`statementType="CALLABLE"`을 사용하지 않고, 일반 `SELECT` 문처럼 호출하는 것이 PostgreSQL에서는 더 일반적이고 편합니다.

```xml
<!-- 방법 1: 일반 SELECT 처럼 호출 (추천) -->
<select id="selectUserNameFunc" resultType="string" parameterType="string">
    SELECT fn_get_user_name(#{userId})
</select>
```

만약 `OUT` 파라미터가 있는 복잡한 프로시저라면:
```xml
<!-- 방법 2: Callable Statement 사용 -->
<select id="callProcedure" statementType="CALLABLE" parameterType="hashmap">
    { CALL my_complex_procedure(
        #{param1, mode=IN},
        #{result, mode=OUT, jdbcType=VARCHAR}
    ) }
</select>
```

---

### 요약
1.  **DB 연동**: `pom.xml`(라이브러리) -> `context-datasource.xml`(접속정보) -> `context-mapper.xml`(MyBatis설정).
2.  **개발 패턴**: Controller -> Service -> DAO -> Mapper XML 순서로 개발.
3.  **로그인**: `session.setAttribute("key", value)`로 저장하고, `session.getAttribute("key")`로 꺼내 쓴다.
4.  **프로시저**: DB에 로직을 숨겨두고 자바에서는 함수처럼 호출한다.
