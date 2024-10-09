# Feign Client

FeignClient는 HTTP API를 호출할 수 있는 선언적 웹 서비스 클라이언트이다.
RESTful 웹 서비스를 호출할 때 필요한 복잡한 코드를 최소화하고 마치 로컬 메서드를 호출하듯이 간단하게 사용이 가능하다.

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/store")
    List<Store> getStores();
}
```

위와 같이 인터페이스 형태로 클라이언트를 선언을 하게 되고 실제로 구현체는 작성을 하지 않게 된다.
RestClient와 비교 시 구현체를 직접 작성을 해줘야 하는데 FeignClient를 사용하게 되면 다음과 같이 구현체가 아닌 선언부만 작성을 해주면 구현체는 알아서 작성이 된다.

설정으로 Configuration 안에 위처럼 FooConfiguration이라는 사용자 설정을 넣어 설정할 수 있는데, 설정이 들어가 있지 않으면 Default로 설정이 지정되고, 그 외로는 위처럼 Custom한 사용자 설정이 우선적으로 넣어지게 된다.

Configuration같은 경우에는 Interceptor와 같이 헤더 값을 추가하여 넣어줄 수도 있으며, 요청에 필요한 Request Key, Value를 넣어 주입을 할 수도 있다.
또한 인코더, 디코더 등을 정의를 할 수가 있는데, 클라이언트를 호출해서 에러가 발생했을 때 에러 처리를 하는 에러 디코더도 굉장히 간편하게 정의를 할 수 있게 된다.

FeignClient를 사용하려면 Application에 `@EnableFeignClients` 어노테이션을 등록해줘야 하고, 옵션 중에 basePackage, clients와 같은 경로 및 정의된 클라이언트를 지정할 수 있다.



