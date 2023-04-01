# 탐색 위치 지정 
 

 

당연한 얘기지만 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸립니다.

 

그래서 다음과 같은 코드를 이용해 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있는데요.

 

@ComponentScan(
	basePackages = "hello.core",
)
 

basePackages: 탐색할 패키지의 시작 위치를 지정합니다.

 

이 패키지를 포함한 하위 패키지를 모두 탐색합니다.

 

basePackages = {"hello.core", "hello.service"} 이런 식으로 여러 시작 위치를 지정할 수도 있습니다.

 

basePackageClasses: 지정한 클래스의 패키지를 탐색 시작 위치로 지정합니다.

 

만약 지정하지 않으면 @ComponentScan 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 됩니다.

 

 

 

 

이를 적용해 기존의 AutoAppConfig를 변경하면 다음과 같습니다.

@Configuration
@ComponentScan(
        basePackages = "hello.core.member",
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {

}
 

 

변경을 적용하면 member 패키지 아래 있는 패키지만을 탐색하며 자동 등록합니다.

 

불필요한 클래스들이 포함된 패키지를 제외함으로써 효율적으로 빈을 등록할 수 있습니다.

 

 

 

 

 

이런 식으로 시작 위치를 지정하는 방법 중 권장되는 방법이 있습니다.

 

패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것입니다.

 

최근 스프링 부트도 이 방법을 기본으로 제공하는데요.

 

예를 들어서 프로젝트가 다음과 같이 구조가 되어 있으면

 

com.hello

com.hello.serivce

com.hello.repository

 

 

com.hello 바로 아래에 AppConfig 같은 메인 설정 정보를 두고,

 

@ComponentScan 애노테이션을 붙이고, basePackages 지정은 생략하는 방식입니다.

 

 

 

이렇게 하면 com.hello 를 포함한 하위는 모두 자동으로 컴포넌트 스캔의 대상이 됩니다.

 

그리고 프로젝트 메인 설정 정보는 프로젝트를 대표하는 정보이기 때문에

 

프로젝트 시작 루트 위치에 두는 것이 일반적으로 선호됩니다.

 

 

 

참고로 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 @SpringBootApplication을

 

이 프로젝트 시작 루트 위치에 두는 것이 관례입니다.

(그리고 이 설정안에 바로 @ComponentScan 이 들어있음!)

 

 

 

 

컴포넌트 스캔 기본 대상
 

컴포넌트 스캔은 @Component 뿐만 아니라 다음과 내용도 추가로 대상에 포함합니다.

 

@Component : 컴포넌트 스캔에서 사용

 

@Controlller : 스프링 MVC 컨트롤러에서 사용

 

@Service : 스프링 비즈니스 로직에서 사용

 

@Repository : 스프링 데이터 접근 계층에서 사용

 

@Configuration : 스프링 설정 정보에서 사용

 

 

해당 클래스의 소스 코드를 보면 @Component 를 포함하고 있는 것을 알 수 있습니다.

 

@Component 
public @interface Controller {
}

@Component 
public @interface Service {
}

@Component 
public @interface Configuration {
}
 
