다양한 의존관계 주입 방법
의존관계 주입은 크게 4가지 방법이 있다.
생성자 주입
수정자 주입(setter 주입)
필드 주입
일반 메서드 주입
생성자 주입
이름 그대로 생성자를 통해서 의존 관계를 주입 받는 방법이다.
지금까지 우리가 진행했던 방법이 바로 생성자 주입이다.
특징 (중요~!!!!!!)
생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.

-> 코드를 잘 짜면 값을 처음 한 번 세팅하고 다음부터는 세팅 못 하도록 막을 수 잇음
불변, 필수 의존관계에 사용

-> 생성자는 두 번 호출 안 되니께 강제로 수정하는 코드 만들지 않는 이상 불변임

애플리케이션 configuration으로 빌딩돼서 올라갈 떄 이미 연관관계 그림을 다 만들고 빌딩하고 싶은 거임

연극 시작 전에 배우 다 정하고 연극 중간에 배우 바꿀 일 없는 것과 동일

그럼 내가 이거 임의로 수정하는 메서드 추가하면 됨? 안됨



개발에서 불변은 중요하다. 좋은 개발 습관은 제약이 있는 것. 한계점과 제약이 있어야 함. 다 열어두면 어디서 수정하는지 모름. 



내가 딱 값을 세팅하고 이거는 이제 더 이상 값을 바꾸면 안됨 ㅇㅇ 하잔어?

=> 가급적이면 생성자에다 값 넣고 수정자 메서드나 세터 메서드 그런 걸 안 만들면 됨 ㅇㅇ

(특징은 항상 ㄴㄴ, 주로 ㅇㅇ)



@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}


이게 생성자 주입 예시 있음 여기에

https://kes0917.tistory.com/32



생성자의 파라미터가 있으면 웬만하면 null 넣으면 안됨 (문서에 명시되어 있는 게 아니고서야) 값 넣어줘야 함.





중요! 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 된다. 물론 스프링 빈에만 해당한다.



객체가 생성되어 빈에 등록될 때 생성자가 반드시 호출되기 때문에 빈의 등록과 의존관계 주입이 같이 일ㅇ어난다.











수정자 주입(setter 주입)
setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
특징
선택, 변경 가능성이 있는 의존관계에 사용
자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.



(필드 값을 직접 수정하기 좀 그러니까 메서드를 이용해 수정하는 것임 이걸 세터 (setter), 수정자라 함)



@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        System.out.println("discountPolicy = " + discountPolicy);
        this.discountPolicy = discountPolicy;
    }

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        System.out.println("memberRepository = " + memberRepository);
        this.memberRepository = memberRepository;
    }

}


이런 식으로 자동 주입을 시킬 수 있음 (soutv는 테스트 용)



스프링 컨테이너는 두 가지 역할

1. 빈을 등록한다.

2. 의존관곌 자동주입한다.



그래서 생성자는 어쩔 수 없이 빈 등록할 때 자동주입이 일어나고 수정자는 두 번째 단계에서 일어남

(순서로 따지면 생성자 1, 수정자가 2)



선택적으로 하려면 @Autowired(required = false)
