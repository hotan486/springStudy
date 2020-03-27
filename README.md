# 3.스프링 loC와 DI 소개

## 3.5.4 메서드 주입 사용하기

### 3.5.4.1 룩업 메서드 주입

- 룩업 메서드 주입 : 
  
    1. 싱글턴 빈이 다른 비싱글턴 빈에 의존시 문제발생
   
    2. 수정자나 생성자 -> 비성글턴을 싱글턴으로 만듬 
   
    3. 싱글턴 빈으로 비싱글틴을 매번 새로 하고 싶다...
- 이해를 위한 예시 
  - lockOpener 사물함(locker)을 여는 서비스 
  - lockOpener 는 주입 된 KeyHelper 를 의존
  - keyHepler 는 재사용에 미적합 셜계
  - openLock() 메서드 호출시 새로운 KeyHelper 가 필요
  - lockOpener 는 싱글턴 KeyHelper 비싱글턴
  - 일반적 생성자나 수정자 주입시 keyHepler 는 재사용 됨
  - 새로이 스턴스 전달 하려면 룩업 메서드 주입이 필요

- ApplicationContextAware 인터페이스로  싱글턴은 ApplicationContext 사용 비싱글턴 의존을 새로이 인스턴스 가능 
- 동일한 방식이 룩업 메서드 주입

- 사용 방법
  - 비싱글턴 빈의 인스턴스를 반환하는 룩업 메서드를 싱글턴 빈에 선언

  - 싱글턴에 참조 -> 룩업메서드를 사용해서 동적생성 서브클래스에 참조
  - abstract 사용하여 선언

  - 하나의 비싱글턴 빈과 두개의 싱글턴 빈을 생성
  - 하나는 수정자 하나는 메서드 주입 

> com.apress.prospring5.ch3.methodInjection.Sigger.java
``` java
// 하나의 비싱글턴 빈
public class Singer {
	
    //객체의 속성 선언
	private String lyric = "I played a quick game of chess with the salt and pepper shaker";

    //객체의 메서드 
    public void sing() {
        //System.out.println("lyric"); 	
    }
}
```

> com.apress.prospring5.ch3.methodInjection.DomoBean.java

``` java
public interface DemoBean {
    //Singer 의 참조
	Singer getMySinger();
    //메서드 룩업
	void doSomething();
}

```
> com.apress.prospring5.ch3.methodInjection.StandardLookupBean.java

``` java
// DemoBean 상속 StandardLookupBean
public class StandardLookupBean implements DemoBean {

    // 객체 선언
	private Singer mySinger;

    // 객체 주입 
	public void setMySinger(Singer mySinger) {
		this.mySinger = mySinger;
	}

    // 주입된 객체 참조
	@Override
	public Singer getMySinger() {
		return this.mySinger;
	}

    // 객체에 선언된 메서드 접근
	@Override
	public void doSomething() {
		mySinger.sing();
	}
}
```

> com.apress.prospring5.ch3.methodInjection.AbstractLookupDemoBean.java

``` java
// 메서드 주입을 이용한 AbstractLookupDemoBean 클래스
public abstract class AbstractLookupDemoBean implements DemoBean {
    // 실 호출은 doSomething
    public abstract Singer getMySinger();

    // getMySinger를 통해 xml에서 객체를 인스턴스함
    @Override
    public void doSomething() {
        getMySinger().sing();
    }
}
```

> rsc.main.resources.spring.methodInjection.app-context-xml.xml

``` xml
<!-- singer 빈 구성 -->
<bean id="singer" class="com.apress.prospring5.ch3.methodInjection.Singer" scope="prototype"/>

<!-- AbstractLookupDemoBean 빈 구성 -->
<!-- lookup-method -> 메서드 룩업 구성 -->
<!-- name -> 빈이 Override하는 메서드 이름 -->
<!-- 인수가 없어야 함, 반환이 해당 객체나 그서브 클래스 타입 여야 함 -->
<!-- 반환 된 빈 -->
<bean id="abstractLookupBean" class="com.apress.prospring5.ch3.methodInjection.AbstractLookupDemoBean">
    <lookup-method name="getMySinger" bean="singer"/>
</bean>

<!-- StandardLookupBean 빈 구성, 속성으로 singer 선언 -->
<bean id="standardLookupBean" class="com.apress.prospring5.ch3.methodInjection.StandardLookupBean">
    <property name="mySinger" ref="singer"/>
</bean>
```
> com.apress.prospring5.ch3.methodInjection.LookupDemo.java

``` java
// 실행
public class LookupDemo {

	public static void main(String... args) {
		// 컨테이너 생성
        GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
		// 해당 정보를 읽음
        ctx.load("spring/methodInjection/app-context-xml.xml");
		// 빈 객체 생성
        ctx.refresh();

        // xml에서 룩업된 메서드를 통해 주입 받음 
		DemoBean abstractBean = ctx.getBean("abstractLookupBean", DemoBean.class);
        // standardLookupBean 선언된 객체를 통해 주입 받음
		DemoBean standardBean = ctx.getBean("standardLookupBean", DemoBean.class);

		displayInfo("abstractLookupBean", abstractBean); // 추상 클래스로 통해 비 싱글턴으로 생
		displayInfo("standardLookupBean", standardBean); // 싱글턴으로 생성 

		ctx.close();
	}

    // 정보출력 
	public static void displayInfo(String beanName, DemoBean bean) {
		Singer singer1 = bean.getMySinger();
		Singer singer2 = bean.getMySinger();

        // 싱글턴인지 비싱글턴인지 확인하고
		System.out.println("[" + beanName + "]: Singer 인스턴스는 같은가? "
				+ (singer1 == singer2));

        //성능 측정을 위해
		StopWatch stopWatch = new StopWatch();
		//시작
        stopWatch.start("lookupDemo");
        
		for (int x = 0; x < 100000; x++) {
			// 객체를 생성
            Singer singer = bean.getMySinger();
			// 객체 메서드 출력
            singer.sing();
		}

        //종료
		stopWatch.stop();
		
        // 측정 시간 출력
        System.out.println("100000번을 얻어오는 데 걸린 시간: " + stopWatch.getTotalTimeMillis() + " ms");

        // 싱글턴의 경우 시간이 덜 걸리고 
        // 비싱글턴인 경우 시간이 더걸림
	}
}
```

### 3.5.4.2 메서드 대체
