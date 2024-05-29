# Kotlin JvmRecord

Kotlin에서 Record를? 굳이?

원래 Java 14에서 Record를 도입하였고 Kotlin의 Data Class와 매우 유사하였다.
record는 저번에 작성하였다시피 toString, equals, hashCode 함수를 자동생성해준다.

예를 들어보자.

```java
public record StudentResponse(String name, int, age) {}
```

위 코드는 학생 데이터를 response해주는 record DTO이다.
이 학생 Record는 이름, 날짜를 인자로 받아 동작하게 해준다.

```java
public class Main {
    public static void main(String[] args) {
        // 레코드 인스턴스 생성
        StudentResponse student1 = new StudentResponse("차윤범", 29);
        StudentResponse student2 = new StudentResponse("차윤붕", 30);

        // toString에 접근
        System.out.println(student1.toString()); // StudentResponse[name=차윤범, age=29]
        
        // hashCode와 equals 메서드에 접근
        System.out.println(student2.equals(student2)); // true
    }
}
```

해당 코드가 어떻게 동작되는 지 확인하기 위해 결과를 sout 출력문 옆에 작성하였고, 
toString()을 호출해보면 Kotlin의 Data Class와 동일하게 출력되는 것을 확인할 수 있다.
또한 equals나 hashCode도 동일하게 동작되어 equals처럼 같은 값을 가진 객체가 동일한 것으로 판단하는 지도 확인이 가능하다.


그럼 이제 왜 굳이 Kotlin에서 Record, 즉 JvmRecord를 지원하는 이유는 뭘까?
Kotlin Data class의 속성은 가변이라는 점이다. 하지만 Record는 불변임을 이미 보장하는 것을 내부의 final 키워드만 봐도 알 수 있다.
즉, 속성의 get 메서드만이 자동 구현되며, set 메서드는 구현되지 않는다. 
그래서 값 접근이 가능하나 Kotlin의 클래스 메서드 copy는 제공되지 않는다.

---
## @JvmRecord 사용해 자바의 record 클래스 정의하기

data class로 선언된 값 객체를 자바의 record 클래스 처럼 선언된 것과 동일하게 만들기 위해 코틀린은 1.5 버전부터 @JvmRecord 어노테이션을 제공한다.

```kotlin
@JvmRecord
data class StudentResponse(val name: String, val age: Int)
```

@JvmRecord 어노테이션을 data class에 붙이면, 이는 자바에서는 record 처럼 인식되며 record에서 생성되는 메서드들이 제공된다. 예를 들어 name()을 통해 name 프로퍼티에 접근하는 것이 포함된다.

```kotlin
public class Main {
    public static void main(String[] args) {
        // 레코드 인스턴스 생성
        StudentResponse student = new StudentResponse("차윤범", 29);

        System.out.println(student.name()); // 속성에 접근 가능
    }
}
```

---
## @JvmRecord를 사용하지 않았을 때와 무엇이 다른가?
위의 StudentResponse 클래스에서 @JvmRecord를 제거해보면 무엇이 다른지가 명확히 드러난다.

```kotlin
data class StudentResponse(val name: String, val age: Int)
```

@JvmRecord를 제거한 StudentResponse 클래스는 name() 을 통해 자바에서 name 프로퍼티에 접근할 수 없다. 대신 일반적인 클래스와 같이 getName을 통해 프로퍼티에 접근해야 한다.

