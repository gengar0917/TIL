# 컴포넌트 스캔을 이용한 빈 자동 주입

[ 컴포넌트 스캔이란? ]
 

지정한 클래스들을 자동으로 스캔하여 빈으로 등록하는 기능입니다.

 

@Component 어노테이션이 붙어있는 클래스들은 전부 컴포넌트 스캔의 대상이 됩니다.

 

 

이론은 이 정도 하고 예제 코드로 컴포넌트 스캔을 자세히 알아보겠습니다.

 

 

 


컴포넌트 스캔을 테스트하기 위해 먼저 기존 새로운 AutoAppConfig 클래스를 만들어줍니다.


@Configuration
@ComponentScan(
	excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class))
public class AutoAppConfig {
 
}
 

@ComponentScan

: 컴포넌트 스캔 어노테이션을 붙여 스프링 빈을 등록할 수 있도록 합니다.

@ComponentScan(
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configurationi.class)
)

: 괄호 안에 있는 타입은 빈으로 받지 않겠다는 뜻입니다.

 

자동으로 빈을 등록시키지만 그 안에서 제외시키는 빈의 타입을 명시하는 생략 가능한 구간입니다.

 

컴포넌트 스캔을 사용하면 @Configuration이 붙은 설정 정보도 자동으로 등록되기 때문에, 

 

기존에 만든 여러가지 Config가 함께 실행되는 것을 방지해 예제를 안전하게 보호하기 위해 제외하였습니다.

 

참고: @Configuration 안에는 @Component가 있기 때문에 따로 제외시키지 않으면 자동으로 컴포넌트 대상으로 지정된다. 현업에서는 이런 식으로 제외시킬 일이 거의 없다. @Configuration 이 컴포넌트 스캔의 대상이 된 이유도 @Configuration 소스코드를 열어보면 @Component 애노테이션이 붙어있기 때문이다.

 

 

 

이렇게 컴포넌트 스캔을 사용하려면 먼저 @ComponentScan 을 설정 정보에 붙여주면 됩니다.


기존의 AppConfig와는 다르게 @Bean으로 등록한 클래스가 하나도 없는 것을 볼 수 있습니다.

앞서 설명 한 대로 컴포넌트 스캔은 @Component 어노테이션이 붙은 클래스를 스캔해서 스프링 빈으로


등록하기 때문에 빈으로 등록할 구현체 클래스들에 @Component 를 붙여주면 되겠습니다.

 

 

 



[ 의존 관계 자동 주입 ]
 

이전에 AppConfig에서는 @Bean으로 직접 설정 정보를 작성했고, 의존관계도 직접 명시했습니다.

 

하지만 이제는 이런 설정 정보 자체가 없기 때문에, 의존관계 주입이 난감해집니다.


이 문제를 해결하기 위한 어노테이션이 바로 @Autowired입니다.

 

@Autowired는 의존관계를 자동으로 주입해줍니다.

 

예시 코드와 함께 설명하겠습니다.




public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

위의 코드는 인터페이스 MemberService의 구현체 MemberServiceImpl의 로직입니다.

 

이렇게 생성자 위에 @Autowired 어노테이션을 붙이면 매개변수에 들어가야 하는

 

데이터 타입의 스프링 빈을 자동으로 주입 해 줍니다.

 

위 상황에서는 MemberRepository 타입의 스프링 빈을 알맞게 찾아 주입시킵니다.


비유하자면 ac.getBean(MemberRepository.class)를 실행한 것과 비슷하게 실행됩니다.

 

 


이를 확인하기 위해 테스트 코드를 구현 해 보겠습니다.


public class AutoAppConfigTest {
    
    @Test
    void basicScan(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberService.class);

    }
}
 

간단하게 AutoAppConfig를 이용해 MemberService 데이터 타입의 빈을 꺼내고

 

MemberService 클래스의 인스턴스인지를 확인하는 테스트 코드를 구현했습니다.

결과는 성공입니다.

 

이로 인해 @ComponentScan과 @Autowired 이용해 의존 관계를 자동으로 주입하는 것을 알 수 있습니다.

 

 

 

 

[ 간략 정리 ]
1. 컴포넌트 스캔

 

@ComponentScan 은 @Component 가 붙은 모든 클래스를 스프링 빈으로 등록합니다.

 

이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용하게 됩니다.

 

예를 들자면 MemberServiceImpl 클래스를 빈으로 등록할 때 memberServiceImpl으로 등록됩니다.

 

만약 스프링 빈의 이름을 직접 지정하고 싶으면 @Component("memberService2") 의 형식으로 이름을

 

부여할 수 있습니다.

 

 

 

2. @Autowired 의존관계 자동 주입

 

생성자에 @Autowired 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입합니다.

 

자동으로 타입이 같은 빈을 찾아서 주입하는 것인데요.

 

getBean(MemberRepository.class) 와 동일하다고 이해하면 되겠습니다.

 

 


 

생성자에 파라미터가 많아도 다 알아서 찾아서 자동으로 주입할 수 있습니다.

 

 
