# 필터
 

includeFilters : 컴포넌트 스캔 대상을 추가로 지정한다.

 

excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다

 

일단 예제로 필터를 사용 해 보겠습니다.

 

package hello.core.scan.filter;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {
}
MyIncludeComponent라는 이름의 애노테이션을 만들어줍니다.

 

package hello.core.scan.filter;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}
MyExcludeComponent라는 이름의 애노테이션을 만들어줍니다.

 

자동 스캔에서 include는 포함, exclude는 제외할 수 있는 애노테이션을 만든 것입니다.

 

이 애노테이션들을 붙혀줄 클래스들 또한 생성 해 줍니다.

 

 

package hello.core.scan.filter;

import org.springframework.stereotype.Component;

@MyIncludeComponent
public class BeanA {
}
MyIncludeComponent 애노테이션이 붙은 BeanA 클래스

 

 

package hello.core.scan.filter;

@MyExcludeComponent
public class BeanB {
}
MyExcludeComponent 애노테이션이 붙은 BeanB 클래스

 

 

ComponentFilterAppConfigTest 클래스를 만들어서 appconfig를 다음과 같은 코드로 생성하였습니다.

 

@Configuration
@ComponentScan(
        includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
        )
static class ComoponentFilterAppconfig{
}

@Configuration 애노테이션은 이 클래스가 스프링의 설정 클래스임을 나타냅니다.

 

이 클래스 안에서는 스프링 빈을 정의하고, 다른 설정들을 구성할 수 있습니다.

@ComponentScan 애노테이션은 스프링이 빈을 검색할 패키지를 설정합니다.

 

이 어노테이션은 includeFilters와 excludeFilters 속성을 갖고 있습니다.

includeFilters 속성은 특정 조건에 맞는 빈만 스캔 대상으로 설정하는데, 

 

해당 예제에서는 MyIncludeComponent 어노테이션이 붙은 클래스만 스캔 대상으로 설정하였습니다.

excludeFilters 속성은 스캔 대상에서 제외할 빈을 설정하는데, 

 

해당 예제에서는 MyExcludeComponent 어노테이션이 붙은 클래스는 스캔 대상에서 제외하였습니다.

따라서, 이 코드에서는 MyIncludeComponent 어노테이션이 붙은 클래스들만 빈으로 등록되고,

 

MyExcludeComponent 어노테이션이 붙은 클래스들은 빈으로 등록되지 않습니다.

 

 

 

그리고 이 AppConfig를 포함한 ComponentFilterAppConfigTest 클래스의 전체 코드입니다.

 

package hello.core.scan.filter;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

import static org.assertj.core.api.Assertions.*;
import static org.springframework.context.annotation.ComponentScan.*;

public class ComponentFilterAppConfigTest {

    @Test
    void filterScan(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);
        assertThat(beanA).isNotNull();

//        BeanB beanB = ac.getBean("beanB", BeanB.class);
    }

    @Configuration
    @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
            )
    static class ComponentFilterAppConfig{
    }
}
 

해당 테스트 코드는 당연히 문제가 없습니다.

 

하지만 예외가 발생할 수 있는데요. 그럴 때에는 아래 글을 참고하시길 바랍니다.

 https://kes0917.tistory.com/39
 

아무튼 해당 테스트 코드를 실행하면 테스트에 통과합니다.

 

이번에는 beanB를 조회 해 보겠습니다.

 

beanB 조회하는 코드는 아래와 같습니다.

BeanB beanB = ac.getBean("beanB", BeanB.class);
 

 

결과:


org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'beanB' available

 

BeanB라는 이름의 빈을 컨테이너에서 못 찾는다는 예외입니다.

 

MyExcludeComponent 애노테이션은 excludeFilters를 이용해 빈에서 제외시켰기 때문에 당연한 결과입니다.

 

예외를 처리하기 위해 예외 처리 코드 또한 입력 해 줍니다.

 

org.junit.jupiter.api.Assertions.assertThrows(
        NoSuchBeanDefinitionException.class,
        () -> ac.getBean("beanB", BeanB.class));
 

람다 문법 기반의 예외 처리 코드입니다.

 

NoSuchBeanDefinitionException 예외가 발생하지 않으면 테스트가 실패됩니다.

 

테스트는 당연하게도 성공합니다.

 

간단히 예제로 필터에 대해 알아봤으면 좀 더 자세한 내용들을 공부 해 보겠습니다.

 

 

 

# FilterType 옵션
 

FilterType은 5가지 옵션이 있습니다.

 

ANNOTATION: 기본값으로 애노테이션을 인식해서 동작합니다.
 

ex) org.example.SomeAnnotation

 

이 옵션은 생략이 가능하며, 예제 코드

@Configuration
@ComponentScan(
        includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
        )
static class ComponentFilterAppConfig{
}
 

에서도 type = FilterType.ANNOTATION을 생략할 수 있습니다.

 

 

ASSIGNABLE_TYPE: 지정한 타입과 자식 타입을 인식해서 동작합니다.
 

ex) org.example.SomeClass

 

@Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MyExcludeComponent.class)
 

위와 같이 작성할 수 있습니다.

 

 

ASPECTJ: AspectJ 패턴 사용
 

ex) org.example..*Service+

 

 

REGEX: 정규 표현식
 

ex) org\.example\.Default.*

 

 

CUSTOM: TypeFilter 이라는 인터페이스를 구현해서 처리
 

ex) org.example.MyTypeFilter

 

 

참고로 @Component면 충분하기 때문에, includeFilters 를 사용할 일은 거의 없습니다.

 

excludeFilters 는 여러가지 이유로 간혹 사용할 때가 있지만 많지는 않다고 합니다.

 

(특히 최근 스프링 부트는 컴포넌트 스캔을 기본으로 제공하는데, 개인적으로는 옵션을 변경하면서 사용하기 보다는 스프링의 기본 설정에 최대한 맞추어 사용하는 것을 권장하고, 김영한 튜터님도 선호하는 편이다.)

 

추가로, 김영한님은 가끔 ASSIGNABLE_TYPE 정도 사용하고,

 

ANNOTATION의 경우 이미 구비되어 있어서 (생략 가능하니까) 안 쓰고

 

요즘에는 includeFilters, excludeFilters 해서 사용할 일이 크게 없다고 하시네요.

 

 

 

여담이지만 이렇게 includeFilters나 excludeFilters를 중괄호로 엮어 가독성 좋게 사용할 수도 있습니다.

 

@ComponentScan(
        includeFilters = {
                @Filter(type = FilterType.ANNOTATION, classes =
                        MyIncludeComponent.class),
        },
        excludeFilters = {
                @Filter(type = FilterType.ANNOTATION, classes =
                        MyExcludeComponent.class),
                @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BeanA.class)
        }
)
 
