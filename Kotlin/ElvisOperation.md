## 엘비스 연산자 (Elvis Operation)

엘비스 연산자는 ?:로 표현하며, ?:의 왼쪽 객체가 non-null이면 그 객체의 값이 리턴되고, null이라면 ?:의 오른쪽 값을 리턴한다.

먼저 if-else로 구현한 예제 코드를 보고, 이 코드를 엘비스 연산자를 이용하여 다시 구현하면서 어떻게 동작하는지 확인할 것이다.


### if-else

아래 예제는 if-else를 사용하여 문자열의 길이를 구하는 코드이다.

String이 null이면 -1을 len에 할당하고, non-null이면 length를 len에 할당하는 코드입니다.

```kotlin
fun main(args: Array<String>){

    val str: String? = "1234"
    val nullStr: String? = null

    var len: Int = if (str != null) str.length else -1
    println("str.length: $len")

    len = if (nullStr != null) nullStr.length else -1
    println("nullStr.length: $len")

}
```


```
// 출력값
str.length: 4
nullStr.length: -1
```

이 코드를 엘비스 연산자를 이용하여 다시 구현해볼 것이다.

---
### Elvis Operation
str?.length ?: -1는 엘비스 연산자 왼쪽의 str?.length 객체가 null이 아니면 그 값을 리턴하고, null이면 -1을 리턴한다.

?.는 Safe call이라고 부르는 것인데, str?.length는 str 객체가 null일 때 null을 리턴하고, null이 아닐 때는 str.length를 리턴한다.

```kotlin
fun main(args: Array<String>){

    val str: String? = "1234"
    val nullStr: String? = null

    var len: Int = str?.length ?: -1
    println("str.length: $len")

    len = nullStr?.length ?: -1
    println("nullStr.length: $len")

}
```

```
// 출력값
str.length: 4
nullStr.length: -1
```

---
### null 리턴 예제
메소드에서 null을 리턴하도록 만들 수도 있다.

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
}
```

---
### Throw Exception 예제
Exception이 발생되도록 구현할 수도 있다.

```kotlin
fun foo(node: Node): String? {
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
}
```

---