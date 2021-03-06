Chapter04 - 서블릿과 JDBC
=========================

- 서블릿이 하는 주된 일은 클라이언트가 요청한 데이터를 다루는 일이다. 데이터를 가져오거나 입력, 변경, 삭제 등을 처리하려면 데이터베이스의 도움을 받아야 한다.
---
CRUD란?
--------
- Create
- Retreival
- Update
- Delete

  가장 기본적인 데이터베이스 연산
---
 JDBC란?
----
- 자바 프로그램 안에서 SQL을 실행하기 위해 데이터베이스를 연결해주는 응용프로그램 인터페이스를 말한다.
---

JDBC 프로그래밍의 첫 번째 작업은 DriverManager를 이용하여 java.sql.Driver 인터페이스 구현체를 등록 하는 일
```
DriverManager.registerDriver(new com.mysql.jdbc.Driver());
```
- com.mysql.jdbc.Driver 클래스는 MySQL드라이버 파일에 들어있다.


DriverManager의 getConnection()을 호출하여 MySQL 서버에 연결할 수 있다.
```
conn = DriverManager.getConnection("jdbc:mysql://localhost/studydb","study","study")
                                              //JDBC URL    //DBMS 사용자 아이디, 암호
```

Connection 구현체를 이용하여 SQL문을 실행할 객체를 준비한다.
```
stmt = conn.createStatement();
```
- createStatement()가 반환하는 것은 java.sql.Statement 인터페이스의 구현체이며 이 객체를 통해 데이터 베이스에 SQL문을 보낼 수 있다.

SQL문을 서버에 보내는 명령문
```
rs = stmt.executeQuery("select MNO,MNAME,EMAIL,CRE_DATE" + "from MEMBERS" + "order by MNO ASC");
```
- executeQuery()가 반환하는 객체는 java.sql.ResultSet 인터페이스의 구현체이며 이 반환 객체를 통해 서버에서 질의 결과를 가져 온다.
```
response.setContentType("text/html; charset=UTF-8");
```
- 출력 스트림을 얻기 전에 먼저 setContentType()를 호출하여 출력하려는 데이터 형식(HTML)rhk answk wlqgkq(UTF-8)을 지정한다.

모아 보기
-------
```
DriverManager.registerDriver(new com.mysql.jdbc.Driver());
			conn = DriverManager.getConnection(
					"jdbc:mysql://localhost/studydb", //JDBC URL
					"study",	// DBMS 사용자 아이디
					"study");	// DBMS 사용자 암호
			stmt = conn.createStatement();
			rs = stmt.executeQuery(
					"SELECT MNO,MNAME,EMAIL,CRE_DATE" +
					" FROM MEMBERS" +
					" ORDER BY MNO ASC");

			response.setContentType("text/html; charset=UTF-8");
```

위 작업들은 try, catch를 이용하여 실행 하고 정상적으로 수행 되든 오류가 발생하든 간에 반드시 자원 해제를 수행하기 위해 finally 블록을 이용하여 자원 해제를 한다.
```
finally {
			try {if (rs != null) rs.close();} catch(Exception e) {}
			try {if (stmt != null) stmt.close();} catch(Exception e) {}
			try {if (conn != null) conn.close();} catch(Exception e) {}
		}
```

---
HttpServlet으로 GET, POST 요청 다루기
------
HttpServlet 클래스는 GenericServlet 클래스의 하위 클래스다. 따라서 HttpServlet을 상속받으면 GenericServlet 클래스를 상속받는 것과 마찬가지로 javax.servlet.Servlet 인터페이스를 구현한 것이 된다.

![서블릿 계층도](http://postfiles13.naver.net/20160529_172/pjm5111_1464451104241Aolog_PNG/123.PNG?type=w3)
- HttpServlet의 service() 메서드가 호출되면 클라이언트 요청 방식에 따라 doGet()이나 doPost() 등의 메서드가 호출 된다.
따라서 HttpServlet을 상속받으면 service() 메서드를 직접 구현하기보다는 클라이언트의 요청 방식에 따라 doXXX() 메서드를 오버라이딩 한다.


---
리프래시(Refresh, 새로 고침)
------------------------
응답 헤더에 리프래시 정보를 추가하여 보낸다
```
response.addHeader("refresh", "1;url=list");
1초의 대기 후 list로 이동
```
혹은 HTML 본문에 포함시켜 보낼 수 있다.(단, <head> 태그 안에 리프래시를 설정하는 <meta> 태그를 추가한다)
```
out.println("<html><head><title>title...</title>");
out.println("<meta http-equiv='Refresh' content=1; url=list'>");
out.println("</head>");
```
```
HTTP 응답 정보 예)
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Refresh: 1;url=list
Content-Type: text/html;charset=UTF-8
```
리다이렉트(Redirect)
------------------
리다이렉트를 요구하는 HttpServletResponse의 sendRedirect() 호출 코드를 추가 한다.
리다이렉트는 클라이언트로 본문을 출력하지 않기 때문에 HTML을 출력하는 코드가 필요 없다.
```
response.sendRedirect("url");
```
```
HTTP 응답 정보 예)
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location: http://localhost:9999/web04/member/list
Content-Length: 0
```
####- 작업 결과를 출력한 후 다른 페이지를 이동 할 때는 리프래시를 사용
####- 작업 결과를 출력하지 않고 다른 페이지로 이동 할 때는 리다이렉트를 사용

---
서블릿 초기화 매개변수
-------------------
####서블릿 초기화 매개변수란?
- 서블릿을 생성하고 초기화하는 init()를 호출할 때 서블릿 컨테이너가 전달하는 데이터이다.
- 데이터베이스 연결 정보와 같은 정적인 데이터를 서블릿에 전달할 때 사용한다.

서블릿 초기화 매개변수는 DD파일(web.xml)의 서블릿 배치 정보에 설정할 수 있고, 애노테이션을 사용하여 서블릿 소스 코드에 설정할 수 있다.

####DD파일에 서블릿 초기화 매개변수 설정
```
<servlet>
  <servlet-name>MemberUpdateServlet</servlet-name>
  <servlet-class>spms.servlets.MemberUpdateServlet</servlet-class>
  <init-param>
    <param-name>driver</param-name>
    <param-value>com.mysql.jdbc.Driver</param-value>
  </init-param>
  <init-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost/studydb</param-value>
  </init-param>
  <init-param>
    <param-name>username</param-name>
    <param-value>study</param-value>
  </init-param>
  <init-param>
    <param-name>password</param-name>
    <param-value>study</param-value>
  </init-param>
</servlet>
```
- <init-param>은 서블릿 초기화 매개변수를 설정하는 태그로 <servlet>의 자식 엘리먼트다.
- <init-param>의 두 자식 엘리먼트인
  - <param-name>: 매개변수의 이름을 지정
  - <param-name>: 매개변수의 값을 지정
- 이처럼 소스파일 밖에 DB정보를 두면 나중에 DB정보가 바뀌어도 web.xml만 편집하면 되면 유지보수가 쉽다는 이점이 생긴다.

```
클래스 로딩
Class.forName(/* JDBC 드라이버 클래스의 이름 */);
Class.forName(this.getInitParameter("driver"));
```
- Class.forName()은 인자값으로 클래스 이름을 넘기면 해당 클래스를 찾아 로딩한다.
- 클래스 이름은 반드시 패키지 이름을 포함해야 한다.('fully qualified name' or 'QName')
- JDBC 드라이버 클래스의 이름은 서블릿 초기화 매개변수에서 얻어 온다.

---
컨텍스트 초기화 매개변수
--------------------

서블릿 초기화 매개변수는 매개변수가 선언된 서블릿에서만 사용될 수 있다. 다른 서블릿은 참조할 수 업삳. 따라서 **JDBC 드라이버와 데이터베이스 연결 정보에 대한 초기화 매개변수를 각 서블릿마다 별도로 설정해 주어야 한다.** 그러나 여러 서블릿이 사용하는 JDBC 드라이버와 DB연결 정보가 같다면, 각각의 서블릿마다 초기화 매개변수를 선언하는 것은 낭비적인 작업이다. 이런 경우 컨텍스트 초기화 매개변수를 사용한다.

web.xml 파일의 시작 부분에 다음과 같이 컨텍스트 초기화 매개변수를 선언 한다.
```
<context-param>
  <param-name>driver</param-name>
  <param-value>com.mysql.jdbc.Driver</param-value>
</context-param>
<context-param>
  <param-name>url</param-name>
  <param-value>jdbc:mysql://localhost/studydb</param-value>
</context-param>
<context-param>
  <param-name>username</param-name>
  <param-value>study</param-value>
</context-param>
<context-param>
  <param-name>password</param-name>
  <param-value>study</param-value>
</context-param>
```
컨텍스트 초기화 매개변수의 값을 얻으려면 SevletContext 객체가 필요하다.
```
doGet() 메서드의 일부분

ServletContext sc = this.getServletContext();
			Class.forName(sc.getInitParameter("driver"));
			conn = DriverManager.getConnection(
						sc.getInitParameter("url"),
						sc.getInitParameter("username"),
						sc.getInitParameter("password"));
			stmt = conn.createStatement();
```
> ServletContext sc = this.getServletContext();
이 객체를 통해 web.xml에 선언된 컨텍스트 초기화 매개변수 값을 얻을 수 있다.

####여러 서블릿에서 공통으로 참고하는 값이 있을 경우 컨텍스트 매개변수로 정의하자

----
필터
----
####필터란?
필터는 서블릿 실행 전후에 어떤 작업을 하고자 할 때 사용하는 기술
- 데이터의 암호를 해제
- 서블릿이 실행되기 전 필요한 자원 준비
- 서블릿이 실행 되면 로그 기록
- 응답 데이터 압축
- 데이터 형식 변환

  ![필터의실행](../img/필터의실행.png)

####필터 클래스
필터 클래스는 javax.servlet.Filter 인터페이스를 구현해야 한다.

  ![필터클래스](../img/필터클래스.png)
- init(){...}

  서블릿 컨테이너가 필터 인스턴스를 초기화하기 위해서 호출하는 메소드
- doFilter(){...}

  필터가 할 일을 작성

  ```
  @Override
	public void doFilter(
			ServletRequest request, ServletResponse response,
			FilterChain nextFilter) throws IOException, ServletException {
    /* 서블릿이 실행되기 전에 해야 할 작업 */

    // 다음 필터를 호출. 더이상 필터가 없다면 서블릿의 service()가 호출됨.
		nextFilter.doFilter(request, response);
    /* 서블릿을 실행한 후, 클라이언트에게 응답하기 전에 해야할 작업 */
	}
  ```
  - nextFilter 매개변수는 다음 필터를 가리킨다.
  - nextFilter.doFilter()는 다음 필터를 호출한다.
  - 서블릿이 **실행되기 전** 처리할 작업은 nextFilter.doFilter() **호출 전**에 작성.
  - 서블릿이 **실행된 후** 처리할 작업은 nextFilter.doFilter() **호출 후**에 작성.


- destroy(){...}
  필터 인스턴스를 종료하기 전에 호출하는 메소드

####DD 파일에 필터 배치 정보 설정
web.xml 파일에 문자 집합을 설정하는 필터를 등록
```
<!-- 필터 선언 -->
<filter>
  <filter-name>CharacterEncodingFilter</filter-name>
  <filter-class>spms.filters.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>

<!-- 필터 URL 매핑 -->
<filter-mapping>
  <filter-name>CharacterEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
@WebFilter 애노테이션을 사용하여 필터의 소스 파일에 직접 배치 정보를 설정
```
@WebFilter(
	urlPatterns="/*",
	initParams={
		@WebInitParam(name="encoding",value="UTF-8")
	})
public class CharacterEncodingFilter implements Filter{
  ...
```
