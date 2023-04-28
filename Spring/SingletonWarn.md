# <싱글톤 방식의 주의점>

객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안됩니다.
즉, 무상태(stateless)로 설계해야 합니다. 또한 특정 클라이언트에 의존적인 필드가 있으면 안되고 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안됩니다.
값을 가급적이면 수정하지 않고 가급적 읽기만 가능해야 하고 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal등을 사용해야 합니다.
스프링 빈의 필드에 공유 값을 설정하면 정말 정말 큰 장애가 발생할 수 있기 때문에 아주 조심해야 합니다.!!!
이런 주의사항들은 말로만 설명을 들으면 잘 와닿지 않기 때문에 로직을 구현하며 설명하겠습니다.

public class StatefulService {

    private int price;

    public void order(String name, int price){
        System.out.println("name = " + name + "price = " + price);
        this.price = price; //여기가 문제 !
    }

    public int getPrice(){
        return price;
    }
}
 
price라는 값의 상태를 계속해서 유지하는 클래스입니다.

class StatefulServiceTest {

    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);
        //ThreadA: B사용자 10000원 주문
        statefulService2.order("userB", 20000);

        //ThreadA: 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);
    }

    static class TestConfig{

        @Bean
        public StatefulService statefulService(){
            return new StatefulService();
        }
    }
}
 
해당 객체를 빈으로 등록 후 스프링 컨테이너를 테스트합니다. 해당 상황은 사용자 A가 주문하고 주문금액을 조회하는 도중  사용자 B가 20000원을 주문한 상황입니다.
쉽게 말 하면 사용자 A가 행동을 하던 중 사용자 B가 끼어든 상황입니다. 실행결과는 price 값으로 기대했던 10000이 아닌 20000이 결과값으로 나왔습니다.
앞서 말 한 주의점 중 객체 인스턴스를 공유하기 때문에 상태를 유지하도록 설계하면 안된다는 이유가 바로 이것 때문입니다.
price 필드는 상태를 유지하기 때문에 가장 마지막 사용자에 의해 저장된 값만을 가지고 있게 됩니다.
그렇기 때문에 price는 기대했던 10000이 아닌 20000을 반환하게 되는 것입니다. 해당 사항을 조심하며 코드를 수정 해 보겠습니다.

 public class StatefulService {

    private int price;

    public int order(String name, int price){
        System.out.println("name = " + name + "price = " + price);
//        this.price = price; //여기가 문제 !
        return price;
    }

//    public int getPrice(){
//        return price;
//    }
}
 
상태를 유지하는 필드를 제거하고 대신 공유되지 않는 지역 변수를 사용했습니다.

class StatefulServiceTest {

    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
        int userAPrice = statefulService1.order("userA", 10000);
        //ThreadA: B사용자 10000원 주문
        int userBPrice = statefulService2.order("userB", 20000);

        //ThreadA: 사용자A 주문 금액 조회
//        int price = statefulService1.getPrice();
        System.out.println("price = " + userAPrice);

//        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig{

        @Bean
        public StatefulService statefulService(){
            return new StatefulService();
        }
    }
}
 
이렇게 수정하면 price가 기대했던 10000을 반환하게 됩니다. 이와 같이 스프링 빈은 무상태로 설계해야 합니다.
