# <AppConfig와 싱글톤>
 
다음 로직은 AppConfig 클래스입니다.

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
    @Bean
    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
 
그런데 이 로직 중 memberService와 orderService 메서드를 보면 각각 memberRepository()를 호출합니다.
한 번씩만 호출한다 해도 적어도 두 번 이상 memberRepository를 생성하게 되는 것처럼 보입니다.
이렇게 되면 당연하게도 싱글톤이 깨지게 되지만 스프링 컨테이너는 싱글톤을 보장하고 있습니다.
이를 확인하기 위해 테스트 코드를 작성 해 보겠습니다.

 public MemberRepository getMemberRepository(){
	return memberRepository;
}
 
먼저 OrderServiceImpl와 MemberService 클래스에 위와 같이 memberRepository를 반환하는 메서드를 추가 해 주고

 public class ConfigurationSingletonTest {

    @Test
    void configurationTest(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("MemberService => memberRepository = " + memberRepository1);
        System.out.println("OrderService => memberRepository = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}
 
이와 같이 memberRepository들을 비교 해 줍니다.
getBean으로 꺼낸 MemberServiceImpl의 getMemberRepository 메서드를 이용해 꺼낸 memberRepository1,
getBean으로 꺼낸 OrderServiceImpl의 getMemberRepository 메서드를 이용해 꺼낸 memberRepository2,
getBean으로 바로 꺼낸 memberRepository까지
이 세 MemberRepository들을 비교하겠습니다. 
결과는 세 MemberRepository가 다 같은 것을 알 수 있습니다. memberRepository 인스턴스는 모두 같은 인스턴스가 공유되어 사용된다는 것인데요.
그러나 AppConfig의 자바 코드를 보면 분명히 각각 2번 new MemoryRepository 호출해서 다른 인스턴스가 생성되어야 합니다. 왜 이렇게 될까요?
이번엔 아예 AppConfig에서 System.out을 통해 새 인스턴스를 생성하는 횟수를 확인하겠습니다.

 @Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }
    @Bean
    public OrderService orderService(){
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
 
이를 실행하면 결과는 예상했던 것과 달리 메서드가 한 번만 호출됩니다. 자세히까진 몰라도 스프링이 싱글톤을 보장 해 주는 것을 알 수 있습니다.
그럼 이제 왜 단순한 자바코드로 작성된 AppConfig가 싱글톤으로 보장되는지를 알아보겠습니다.

 
# @configuration은 싱글톤을 위해 존재한다
 
스프링 컨테이너는 싱글톤 레지스트리입니다. 싱글톤 패턴의 단점을 보완하기 위해 빈을 싱글톤으로 관리한다는 뜻입니다.
한 마디로 스프링 빈이 싱글톤이 되도록 보장 해 주어야 하는데요. 하지만 스프링이 자바 코드까지 영향을 주어 바꾸기는 너무 어렵습니다.
위의 자바 코드만 본다면 분명 3번 호출되어야 하는 게 맞습니다. 그래서 스프링 컨테이너는 싱글톤을 유지하기 위해 클래스의 바이트코드를 조작하는 라이브러리를 사용합니다.
모든 비밀은  @Configuration을 적용한 AppConfig에 있습니다.
사실 AnnotationConfigApplicationContext에 파라미터로 넘긴 값은 스프링 빈으로 등록됩니다. 간단히 말하면 AppConfig도 스프링 빈이 된다는 뜻입니다. 
AppConfig스프링 빈을 조회해서 클래스 정보를 출력해 보겠습니다..

 @Test
void configurationDeep(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
}
 
순수한 클래스라면 결과는 다음과 같아야 한다.
bean = class hello.core.AppConfig

그런데 예상과 달리 결과는 다음과 같습니다.
bean = class hello.core.AppConfig$$SpringCGLIB$$0

 클래스 명이 상당히 복잡합니다.
이유는 간단합니다. 이것은 제가 만든 클래스가 아니기 때문입니다. 강의에서 나온 설명을 보여드리겠습니다.

 설명:
이것은 내가 만든 클래스가 아니다. 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 이용해 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 
그 다른 클래스를 스프링 빈으로 등록한 것이다.

무슨 말인지 전혀 모르겠습니다..
다음과 같은 그림을 보며 이해를 위해 설명 해 보겠습니다.
CGLIB라는 바이트코드 조작 라이브러리가 있습니다. 이 라이브러리를 가지고 AppConfig를 상속 받는 다른 클래스를 만듭니다.
그리고 스프링 컨테이너는 AppConfig가 아닌 조작한 AppConfig@CGLIB를 스프링 빈으로 등록 해 버립니다.
이름은 appConfig지만 인스턴스 객체는 AppConfig@CGLIB으로 들어가게 되는 것입니다. AppConfig@CGLIB는 AppConfig의 자식 클래스이므로 AppConfig 타입으로 조회가 가능합니다.
CGLIB가 붙는 그 임의의 다른 클래스가 바로 싱글톤이 되도록 도와줍니다. 실제로는 훨씬 복잡하겠지만 간단히 작성하자면, 아마 다음과 같이 바이트코드를 조작 해 작성되어 있을 것입니다.

 @Bean
public MemberRepository memberRepository(){

	if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
    	return 스프링 컨테이너에서 찾아서 반환;
    } else{ //스프링 컨테이너에 없으면
    	기존 로직을 호출해서 MEmoryMemberRepository를 생성하고 스프링 컨테이너에 등록
        return 반환
    }
}
 
@Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어집니다.
이런 식으로 싱글톤 보장하게 되는 것이죠.
그렇다면 @Configuration를 붙이면 바이트코드를 조작하는 CGLIB 기술을 이용해 싱글톤을 보장한다는 것인데 만약 @Bean만 적용하고 @Configuration을 제외하면 어떻게 될까요?
위의 테스트 코드에서 @Configuration만 제외하고 실행 해 봤습니다.지금까지 실험한 테스트 코드들을 되짚어 실행 해 보겠습니다.

@Test
void configurationDeep(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
}
 
결과는 다음과 같습니다.
bean = class hello.core.AppConfig

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }
    @Bean
    public OrderService orderService(){
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
 
결과는
call AppConfig.memberRepository가 3번 호출됩니다.

 public class ConfigurationSingletonTest {

    @Test
    void configurationTest(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("MemberService => memberRepository = " + memberRepository1);
        System.out.println("OrderService => memberRepository = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}
 
이 테스트의 결과 역시
memberRepository 비교 결과는 실패하고 각 세 인스턴스가 모두 다른 걸 알 수 있습니다.

 AppConfig 클래스가 원본 클래스 그대로 이용되고 인스턴스가 호출될 때마다 생성되고 세 인스턴스가 모두 다른 객체가 되었습니다.
싱글톤이 깨진 것입니다.
@Bean만 사용해도 스프링 빈으로 등록되긴 하지만, 싱글톤은 보장하지 않는 것을 알 수 있습니다.
memberRepository()처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않습니다.
하지만 크게 고민할 것 없이 스프링 설정 정보는 항상 @Configuration를 사용하면 되겠습니다.
