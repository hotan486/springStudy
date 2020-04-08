# 4. 스프링 구성 상세와 스프링 부트

## 4.11 프로 파일 

- 프로파일(profile) 활성화시 해당 프로파일에 정의된 ApplicationContext 인스턴스만 구성
  
### 4.11.1 스프링 프로파일 기능 사용 예제

- FoodProviderService 
    
    - 유치원에서 고등학교까지 학교에 식품을 공급하는 일 담당 서비스

- proideLunchSet()
  
    - FoodProviderService 인터페이스에 서비스 요청하여 학교의 모든 학생에게 제공할 점심 세트를 생산하는 메서드

- Food

    - 점심세트 클래스 
    - name 속성만 존재

- FoodProviderService 두 공급자 
    
    - 공급자의 점심 세트 제공 서비스는 동일하지만 음식 구성을 다름 

    - 이 경우 프로파일을 이용하여 구성시 해당 설정 정보에 profile="..." 해당 내용을 명시 
    - 이 경우 프로파일 미 활성화시 에러 해당 프로파일 생성시 해당 빈 인스턴스 설정 정보만 생성

- 독립형 Application 에서 ApplicationContext 사용시 프로파일 활성화

    - *-config.xml 와일드카드를 붙힘으로 둘다 읽음
    - 실제 jvm 인수로 프로파일 활성화
    - -Dspring.profiles.active="kindergarten"

### 4.11.2 자바 구성을 사용하는 스프링 프로파일

- @Configuration 설정 파일
- @profile("...")
- @Bean 빈 메서드 명시후 등록
- 자바 파일경우 와일드 카드 인자 없이 두클래스모두 ApplicationContext 등록

### 4.11.3 프로파일 사용시 고려 사항

- 프로 파일기능은 개발자가 빌드도구(메이븐에 프로파일)로 수행하던 애플리케이션의 실행 구성을 관리할수 있는 다른 방법을 제공
- 빌드 도구들은 도구 실행시 지정한 프로파일 인수에 의존에 프로파일에 구성과 프로퍼티 파일을 자바 아카이브에 넣고 배포
- 스프링의 경우 개발자가 스스로 프로 파일을 정의하고 프로그래밍 또는 jvm 인수를 전달해 활성화 -> 프로파일 관리도 프로그래밍화
- 단점은 다른 개발환경에 함꼐 번들링 할 경우 오류에 취약함 
  
## 4.12 Environment와 PropertySource 추상화

- environment
    
    - 프로 파일 활성화 인터페이스
    - 실행환경을 추상화하는 추상레이어 

- [Java] System Properties
  - vi ~/.bash_profile 환경변수
  - echo $PATH 
  - https://unabated.tistory.com/entry/Java%EC%97%90%EC%84%9C-SystemgetProperty    

  - http://dorbae.blogspot.com/2015/04/java-system-properties.html

- GenericXmlApplicationContext 초기화 후에 ConfigurableEnvironment 인터페이스 참조
- ConfigurableEnvironment 인터페이스 통해 MutablePropertySources 가져옴
- Map에 넣고 해당 키 생성후 MapPropertySource 인스턴스 생성
- addFirst 메서드에 넣고 MutablePropertySources에 추가 

- environment를 통해 -> jvm 시스템 프로퍼티 -> 환경변수 -> 애플리케이션 정의 프로퍼티 순서로 접근 
  
- 순서 제어도 가능 애플리케이션 정의 프로퍼티를 map에 담아 명시하면 처음 값으로 출력 가능 
- environment에 애플리케이션 정의 프로퍼티 사용하는 일은 없음

- 프로퍼티 위치 지정자를 통해 해석값을 주입 하는데 사용 

- 프러퍼티 파일에 해당 속성을 명시하고 xml에서 

- environment가 읽기 위해서 context:property-placeholder 해당 프로퍼티 파일 명시 -> jvm 보다 먼저 읽으려면 local-override="true"

## 4.13 jsr-330 애너테이션을 사용한 구성

- 애너테이션 형식의 자바 의존성 주입 라이브러리 
- 사용시 javax.inject -> 메이븐에서 추가함
- @named 통해 빈 정의
- @Inject 생성자 주입시 사용 
- @Singleton 스프링에 프로토타입과 동일, 자바는 비싱글턴 하지만 스프링은 싱글턴 
- xml 설정은 스프링과 동일하게 구성 
- 스프링과 330의 차이점 

    - @Autowired 사용시 requird 속성으로 DI가 반드시 제공 되야 하는지 나타낼수 있음 하지만 @Inject 없음
    - 330은 싱글턴 비싱글턴만 지원 
    - @Lazy 요청 시점에 빈 인스턴스 생성 330은 없음

- 한가지 사용을 권장하고 스프링 사용을 적극 권장 

## 4.14 그루비를 사용한 구성

- GenericGroovyApplicationContext 통해 컨테이너 생성 
- groovy-all, groovy-eclipse-compiler, groovy-eclipse-batch 설치
- plugin에서 실제 proovy가 들어왔는지 필수 확인 
- https://cepojyze.tistory.com/22 이클립스 상에서도 설치 -> 해당 버전 필수 확인 

## 4.15 스프링 부트