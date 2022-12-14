# TypeConverter
Converter(형변환),Fomatter(포맷변환) 인터페이스를 사용하여 변환기 구현해 WebMvcConfigurer 인터페이스를 이용한Config를 구현하여 사용해본다. Thymeleaf에서 제공하는 변환표기법도 사용해본다
<br/>
스프링 타입 컨버터 소개<br/>
문자를 숫자로 변환하거나, 반대로 숫자를 문자로 변환해야 하는 것 처럼 애플리케이션을 개발하다 보면
타입을 변환해야 하는 경우가 상당히 많다.<br/>
<br/>
String data = request.getParameter("data")<br/>
HTTP 요청 파라미터는 모두 문자로 처리된다. 따라서 요청 파라미터를 자바에서 다른 타입으로 변환해서
사용하고 싶으면 다음과 같이 숫자 타입으로 변환하는 과정을 거쳐야 한다.<br/>
Integer intValue = Integer.valueOf(data)<br/>
<br/>
이번에는 스프링 MVC가 제공하는 @RequestParam 을 사용해보자.<br/>
![image](https://user-images.githubusercontent.com/69129562/206471999-0f566ea8-8ff2-44ea-94a2-af56107c2e69.png)
앞서 보았듯이 HTTP 쿼리 스트링으로 전달하는 data=10 부분에서 10은 숫자 10이 아니라 문자 10이다.<br/>
스프링이 제공하는 @RequestParam 을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 편리하게
받을 수 있다.<br/>
이것은 스프링이 중간에서 타입을 변환해주었기 때문이다.<br/>
이러한 예는 @ModelAttribute , @PathVariable 에서도 확인할 수 있다<br/>
@ModelAttribute UserData data<br/>
class UserData {<br/>
 Integer data;<br/>
}<br/>
<br/>
/users/{userId}<br/>
@PathVariable("userId") Integer data<br/>
스프링의 타입 변환 적용 예<br/>
-스프링 MVC 요청 파라미터<br/>
-@RequestParam , @ModelAttribute , @PathVariable<br/>
-@Value 등으로 YML 정보 읽기<br/>
-XML에 넣은 스프링 빈 정보를 변환<br/>
-뷰를 렌더링 할 때<br/>
<br/>
스프링과 타입 변환<br/>
이렇게 타입을 변환해야 하는 경우는 상당히 많다. 개발자가 직접 하나하나 타입 변환을 해야 한다면, 생각만
해도 괴로울 것이다.<br/>
스프링이 중간에 타입 변환기를 사용해서 타입을 String Integer 로 변환해주었기 때문에 개발자는
편리하게 해당 타입을 바로 받을 수 있다.<br/> 앞에서는 문자를 숫자로 변경하는 예시를 들었지만, 반대로 숫자를
문자로 변경하는 것도 가능하고, Boolean 타입을 숫자로 변경하는 것도 가능하다.<br/> 만약 개발자가 새로운
타입을 만들어서 변환하고 싶으면 어떻게 하면 될까?<br/>
<br/>
![image](https://user-images.githubusercontent.com/69129562/206473137-ca0e629c-3725-4228-bdfb-c89a1853d87a.png)
스프링은 확장 가능한 컨버터 인터페이스를 제공한다.<br/>
개발자는 스프링에 추가적인 타입 변환이 필요하면 이 컨버터 인터페이스를 구현해서 등록하면 된다.<br/>
이 컨버터 인터페이스는 모든 타입에 적용할 수 있다. 필요하면 X Y 타입으로 변환하는 컨버터
인터페이스를 만들고, 또 Y X 타입으로 변환하는 컨버터 인터페이스를 만들어서 등록하면 된다.<br/>
예를 들어서 문자로 "true" 가 오면 Boolean 타입으로 받고 싶으면 String Boolean 타입으로
변환되도록 컨버터 인터페이스를 만들어서 등록하고, 반대로 적용하고 싶으면 Boolean String
타입으로 변환되도록 컨버터를 추가로 만들어서 등록하면 된다.<br/>
<br/>
먼저 가장 단순한 형태인 문자를 숫자로 바꾸는 타입 컨버터를 만들어보자.<br/>
public class StringToIntegerConverter implements Converter<String, Integer> {<br/>
 @Override<br/>
 public Integer convert(String source) {<br/>
 log.info("convert source={}", source);<br/>
 return Integer.valueOf(source);<br/>
 }<br/>
}<br/>
String Integer 로 변환하기 때문에 소스가 String 이 된다. 이 문자를
Integer.valueOf(source) 를 사용해서 숫자로 변경한 다음에 변경된 숫자를 반환하면 된다.<br/>
![image](https://user-images.githubusercontent.com/69129562/206473137-ca0e629c-3725-4228-bdfb-c89a1853d87a.png)
<br/>
![image](https://user-images.githubusercontent.com/69129562/206475235-5a1c9550-8a46-4541-baf1-c3b72e14349b.png)
127.0.0.1:8080 같은 문자를 입력하면 IpPort 객체를 만들어 반환한다.<br/>
<br/>
컨버전 서비스 - ConversionService<br/>
이렇게 타입 컨버터를 하나하나 직접 찾아서 타입 변환에 사용하는 것은 매우 불편하다. 그래서 스프링은
개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공하는데, 이것이 바로 컨버전
서비스( ConversionService )이다.<br/>
<br/>
등록과 사용 분리<br/>
컨버터를 등록할 때는 StringToIntegerConverter 같은 타입 컨버터를 명확하게 알아야 한다.<br/> 반면에
컨버터를 사용하는 입장에서는 타입 컨버터를 전혀 몰라도 된다.<br/> 타입 컨버터들은 모두 컨버전 서비스
내부에 숨어서 제공된다. 따라서 타입을 변환을 원하는 사용자는 컨버전 서비스 인터페이스에만 의존하면
된다. 물론 컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존관계 주입을 사용해야 한다.<br/>
<br/>
컨버전 서비스 사용<br/>
Integer value = conversionService.convert("10", Integer.class)<br/>
<br/>
인터페이스 분리 원칙 - ISP(Interface Segregation Principle)<br/>
인터페이스 분리 원칙은 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다.<br/>
DefaultConversionService 는 다음 두 인터페이스를 구현했다.<br/>
  -ConversionService : 컨버터 사용에 초점<br/>
  -ConverterRegistry : 컨버터 등록에 초점<br/>
이렇게 인터페이스를 분리하면 컨버터를 사용하는 클라이언트와 컨버터를 등록하고 관리하는 클라이언트의
관심사를 명확하게 분리할 수 있다.<br/> 특히 컨버터를 사용하는 클라이언트는 ConversionService 만
의존하면 되므로, 컨버터를 어떻게 등록하고 관리하는지는 전혀 몰라도 된다. 결과적으로 컨버터를
사용하는 클라이언트는 꼭 필요한 메서드만 알게된다. 이렇게 인터페이스를 분리하는 것을 ISP 라 한다<br/>
<br/>
포맷터 - Formatter<br/>
Converter 는 입력과 출력 타입에 제한이 없는, 범용 타입 변환 기능을 제공한다.<br/>
이번에는 일반적인 웹 애플리케이션 환경을 생각해보자. 불린 타입을 숫자로 바꾸는 것 같은 범용 기능
보다는 개발자 입장에서는
문자를 다른 타입으로 변환하거나, 다른 타입을 문자로 변환하는 상황이 대부분이다.<br/>
앞서 살펴본 예제들을 떠올려 보면 문자를 다른 객체로 변환하거나 객체를 문자로 변환하는 일이
대부분이다.<br/>
웹 애플리케이션에서 객체를 문자로, 문자를 객체로 변환하는 예<br/>
화면에 숫자를 출력해야 하는데, Integer String 출력 시점에 숫자 1000 문자 "1,000" 이렇게
1000 단위에 쉼표를 넣어서 출력하거나, 또는 "1,000" 라는 문자를 1000 이라는 숫자로 변경해야 한다.<br/>
날짜 객체를 문자인 "2021-01-01 10:50:11" 와 같이 출력하거나 또는 그 반대의 상황<br/>
Locale<br/>
여기에 추가로 날짜 숫자의 표현 방법은 Locale 현지화 정보가 사용될 수 있다.<br/>
이렇게 객체를 특정한 포멧에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 특화된 기능이
바로 포맷터( Formatter )이다. 포맷터는 컨버터의 특별한 버전으로 이해하면 된다.<br/>
Converter vs Formatter<br/>
Converter 는 범용(객체 객체)<br/>
Formatter 는 문자에 특화(객체 문자, 문자 객체) + 현지화(Locale)<br/>
Converter 의 특별한 버전<br/>
<br/>
포맷터 - Formatter 만들기<br/>
포맷터( Formatter )는 객체를 문자로 변경하고, 문자를 객체로 변경하는 두 가지 기능을 모두 수행한다.<br/>
String print(T object, Locale locale) : 객체를 문자로 변경한다.<br/>
T parse(String text, Locale locale) : 문자를 객체로 변경한다.<br/>
<br/>
