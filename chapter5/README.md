# 5장 제네릭

*** 

# item26. 로타입(raw type)은 사용하지 말라
* raw type : 제네릭 타입(제네릭 클래스 or 제네릭 인터페이스)에서 타입 매개변수를 전혀 사용하지 않을 때 
ex) List<E> 의 raw type = List 
  <br />
  raw type은 제네릭 타입 정보가 전부 지워진 것처럼 동작 (제네릭이 도래하기 전의 코드와 호환되기 위해)
  <br />
  
* 제네릭 타입은 오류를 컴파일 단계에서 발견할 수 있도록 도와줌, 해당 타입 매개 변수에 엉뚱한 인스턴스를 넣을 때 컴파일 에러 발생 
<br />
  
* raw type을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다. 
<br />
  
* 그렇다면 raw type은 왜 있는 것인지 ? "호환성"
<br />


List (X) - 타입 안정성을 잃음 <br /> 
List<Object> (O) - 모든 타입을 허용한다는 의사를 컴파일러에 전달하기 때문


* 제네릭 타입을 쓰고 싶지만, 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 -> ?(비한정적 와일드 카드 타입?)

* 예외
  1) class 리터럴에는 raw type 사용해야함 
    <br />
ex) List.class, String[] class, int.class (O)
  List<String>.class, List<?>.class (X)
     <br />
     
    2) instanceof 연산자
    <br />
    ex)
       <code>
       if (o instanceof Set) { Set<?> s = (Set<?>) o; }
       </code>
       
***
# item27. 비검사 경고를 제거하라 

* 비검사 경고(warning: [unchecked])는 최선을 다해서 제거해야한다. <br /> 
만약, 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 
  > @SuppreddWarnings("unchecked")
  
  annotation으로 경고를 숨기고, 근거를 주석으로 남길 것 

***
# item 28. 배열보다는 리스트를 사용하라

### 배열과 제네릭 타입의 차이점 
1. 배열은 공변(다형성 보장), 제네릭은 불공변(타입 안정성)

- 배열 <br />
<pre>
<code>
Object objectArray = new Long[1]; // 컴파일이 된다, Long은 Object의 하위라 배열의 타입도 하위 호환됨
objectArray = 'hello';            // 런타임에서 ArrayStoreException을 던짐 
</code>
</pre>

- 리스트 <br />
<pre>
<code>
List(Object) ol = ArrayList(Long)(); // 호환되지 않아 컴파일에서부터 에러 
</code>
</pre>

2. 배열은 실체화, 제네릭은 소거 

=> 배열은 런타임에도 자신의 원소의 타입을 인지, 확인 <br />
제네릭은 런타입에 타입정보 소거, 컴파일시에만 검사 <br />
선언된 타입에 다른 타입을 넣으려 할 때, 배열은 런타임 시에, 제네릭은 컴파일 시에 에러가 나기 때문에 조금 더 안전

### 실체화 불가 타입에 대한 배열 생성은 불가 
new List<E>[], new List<String>[], new E[] (런타임에 정보가 없어지는 타입들)
배열이라 런타임에 에러를 뱉어야 하는데, 타입에 대한 정보 실체가 없는 실체화 불가 타입은 배열 생성이 불가해야함 
=> 제네릭 배열 생성 오류 

제네릭 배열이 가능하다고 가정하면 런타임에 ClassCastException 발생할 수 있는데, 이는 컴파일 타임에
타입 에러를 체크하겠다는 제네릭의 취지에 어긋남 

둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 배열을 리스트로 대체하자 

***
# item 29. 이왕이면 제네릭 타입으로 만들라

- Stack등 유틸리티 클래스를 만들 때는 제네릭 타입으로 만드는 것이 좋다
- 아이템 28에서 배열보다는 리스트를 사용하라고 했지만,
  경우에 따라 리스트를 사용할 수 없는 경우도 있다.
  성능은 배열이 우세, 실제로 HashMap같은건 성능 때문에 내부적으로 배열을 사용
  ArrayList같은 제네릭 타입을 만드는 경우에도 결국은 기본 타입인
  배열을 사용해야 한다

***
# item 30. 이왕이면 제네릭 메소로 만들라
- 제네릭 싱글턴 팩토리 패턴
  제네릭 싱글턴 팩터리를 안쓴다면,
  쓸 때 마다 바깥에서 캐스팅 해주거나,
  며<String> IDENTITY_FN_STR 이런 식으로
  타입을 지정해서 변수를 다 만들어놔야함

- 재귀적 타입 한정
  ex) <E extends Comparable<E>>
  Comparable<E>를 구현한 클래스만 E가 될 수 있다
  위보다는, <E extends Comparable<? super E>>를 쓰는 편이 좋다
  
***
# item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라 
- PECS <br />
T의 Producer라면 Extends, Consumer라면 Super ... <br />

- public static double sumOfList(List<? extends Number> list) { ... } <br />
=> 지정된(Number)의 하위 타입(Integer)만 받을 수 있다
- public static void printList(List<?> list) { ... } <br />
=> 제한 없음
- public static void addNumbers(List<? super Integer> list) { ... } <br />
=> 지정된(Integer) 타입의 상위 타입만 허용(Number, Object)


  출처 - https://bravenamme.github.io/2019/12/12/java-generic/
  
*** 
# 질문 정리 

### 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 기본 타입인 배열을 사용해 구현해야 한다.
=>
자바가 리스트를 기본 타입으로(기본 자료형으로) 제공하지 않으므로 내부에서 배열을 사용하여 구현해서 사용하고 있음을 의미하는 것으로 이해하였습니다.
- ArrayList.java
<pre>
<code>
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}


/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) { // 한정적 와일드 카드 타입 
    Object[] a = c.toArray(); // 매개변수로 받은 c를 toArray메소드를 통해 배열로 변환
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
</code>
</pre>

### HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.
=> 배열은 인덱스를 사용하여 자료의 검색이 한번에 이루어지기 때문에
검색 속도가 빠르며, 이 때문에 성능을 높일 목적으로 해쉬에서 내부적으로 배열을 사용하,
HashMap은 내부적으로 Hashing을 이용하여 구현한 컬렉션이기 때문에 많은 양의 데이터를 검색하는데 있어 검색 속도가 빠르기 위해서
(성능을 높일 목적으로) hashing에서 배열을 사용한 것을 이야기한 것 같습니다. 


### 공통. 제네릭 타입 소거 에 대해 복습하는 시간을 가졌으면 좋겠습니다

제네릭을 도입하기 전의 기존의 코드를 수용하고 호환성 위해 : 제네릭 타입 소거 + raw 타입 지원 <br />
컴파일 타임에만 타입 제약 조건을 정의하고, 런타임에는 타입을 제거한다. <br />
Java 컴파일러의 타입 소거 적용
1) 제네릭 타입 소거 (raw타입은 컴파일 시에 Object로 대체)
2) Object로 대체한 후 타입 안정성 보존을 위해 필요하다면 type casting을 넣어준다
3) 다형성을 위해 Bridge Method를 만들 수도 있다
<pre>
<code>
// 컴파일 할 때 (타입 소거 전) 
public class Test&lt;T> {
    public void test(T test) {
        System.out.println(test.toString());
    }
}

// 런타임 때 (타입 소거 후)
public class Test {
    public void test(Object test) { 
// T 와 같이 타입을 직접 명시하지 않은 타입 매개 변수 - raw 타입, Object로 
        System.out.println(test.toString());
    }
}

// 런타임 시에 &lt;Integer> 타입이 소거되므로, Comparator의 compare 메소드의 매개변수 타입은 Object로 바뀔 것 
public class MyComparator implements Comparator&lt;Integer> {
   public int compare(Integer a, Integer b) {
   }

   //THIS is a "bridge method" 
   //매개변수가 Integer 타입의 compare 메소드를 사용할 수 있게 됩니다.
   public int compare(Object a, Object b) {
      return compare((Integer)a, (Integer)b);
   }
}
</code>
</pre>

제네릭은 타입소거라는 특징으로 컴파일러가 컴파일 타임에 타입을 추론할 수 있으며, <br />
이런 타입 추론 기능을 강력하게 사용하기 위해서는 런타임에 타입을 추론하는 Array 대신에 컴파일타임에 타입을 추론하는 List를 함께 사용해야 안정성을 보장 할 수 있다는 것

### 메서드도 형변환 없이 사용할 수 있는 편이 더 좋다 -> 막연하게 이해되나 이렇게 사용해야하는 이유와 어떤 상황에서 쓰면 좋을까요?
형변환이 생기면 생길 수록 성능 저하...가 생기기 때문에, 메서드도 가능한 형변환 없이, 제네릭 메소드를 사용해야한다는 것으로 이해가 됩니다ㅠㅠ...
제네릭 메서드를 사용하면 하나의 메서드로 모든 타입에 대응 할 수 있게 만들어 지기 때문에 재사용성도 좋아 지기 때문에...
형변환이 많이 필요할 경우 제네릭 메소드를 사용해야한다는 내용 같습니다... 


### 목표 타입과 목표 타입 추론은 뭘까요?
- 목표 타이핑(Target typing) <br />
  ex) long l = (int형 값) * (int형 값); <br />
  long 자료형의 변수 l에 int * int의 결과값이 대입된다 <br />
  컴파일러가 int * int 식을 long * long으로 자동 변환하여 연산을 수행하는 기능을 목표 타이핑 <br />
  결과 자료형에 따라 계산식을 자동 변환하여 연산을 수행하는 기능 <br />
  
- 목표 타입(Target type) <br />
  : The Target-Type of an expression is the data type that the Java Compiler expects depending on where the expression appears.<br />
  목표 타입은, 표현식이 나타나는 위치에 따라 자바 컴파일러가 예상하는 데이터 유형<br />

- 목표 타입 추론(Target type inference) <br />
  : 불필요한 코드의 장황함을 줄이기 위해,
  컨텍스트 정보를 기반으로 표현식의 지정되지 않은 데이터 유형을 자동으로 추론하는 프로세스
  
  <pre>
  <code>
  static &lt;T&gt; List&lt;T&gt; add(List&lt;T&gt; list, T a, T b) {
    list.add(a);
    list.add(b);
    return list;
  }

  List&lt;String&gt; strListGeneralized = add(new ArrayList&lt;>(), "abc", "def");
  List&lt;Integer&gt; intListGeneralized = add(new ArrayList&lt;>(), 1, 2);
  List&lt;Number&gt; numListGeneralized = add(new ArrayList&lt;>(), 1, 2.0);
  </code>
  </pre>
코드에서 ArrayList<> 는 타입 인수를 명시적으로 제공하지 않습니다. 따라서 컴파일러는 이를 유추해야 합니다.   
ArrayList&lt;String> , ArrayList&lt;Integer> 및 ArrayList&lt;Number> 세 가지 인스턴스의 인수를 추론합니다. <b>목표 타입 추론</b> <br />
추론 한 유형 String, Integer, Number <b>목표 타입</b>

출처 : https://www.baeldung.com/java-generalized-target-type-inference

### p. 187 ~ 188의 comparable<E>, Delayed, ScheduledFuture<V>의 관계에 대한 설명
ScheduledFuture&lt;V> extends Delayed <br />
Delayed extends Comparable&lt;Delayed> <br />
Comparable&lt;T> <br />
https://blog.benelog.net/2173103.html

### (아이템30) 메서드 선언에서 타입 매개변수 목록은 '이 메서드에서는 이러한 타입 매개변수만을 사용한다'고 선언하는 용도로 이해하면 되는 건가요?

### (아이템31, p.189 예시) 와일드카드를 사용했는데, 만약 복잡한 내부 로직을 구현해야 한다면 결국 특정 제네릭으로 바꿔서 구현해야 한다는 의미인가요?

### (공통) 타입 매개변수가 구체적으로 무엇이 있고 각각의 용도가 무엇인지 모르겠어요
- 널리 사용되는 타입 매개변수 이름 <br />
E - Element (배열이나 맵등의 컬렉션에 들어 가는 element) <br />
K - Key <br />
N - Number <br />
T - Type <br />
V - Value <br />
S,U,V etc. - 2nd, 3rd, 4th types <br />


# item 32.제네릭과 가변인수를 함께 쓸 때는 신중하라

- 가변인수로 제네릭을 받는 경우에, 힙오염이(heap pollution)이 발생할 수 있는 이유 <br />
: 가변인수 메소드를 호출하면, 가변인수를 담기 위한 배열이 하나 만들어지는데, <br />
실체화 불가 타입인 제네릭을 가변인수로 받는 경우(List<String>... 이나 T...) 실체화 불가 타입에 대한 배열이 만들어진다 <br />
  => 힙오염으로 이어짐 

- 제네릭 등 실체화 불가 타입의 배열은 원래 불가하다. 타입 안전성이 깨질 수 있음(런타임에 ClassCastExeption 발생할 수 있음) <br />
그래서 컴파일 타임에 경고로 알려줌 

- item28에서와 같이 제네릭 배열을 직접 만드는 것은 오류를 내지만, 가변인수에서 생성하는 건 경고로 끝내는 이유는 ? <br />
Arrays.asList(T... a), List.of(E ...)와 같이 자바 라이브러리도 이런 메소드를 여럿 제공하는 것과 같이 실무에서 유용하기 때문 

- @SafeVarargs annotation - 그 메소드가 타입 안전함을 보장 <br />
varargs 메소드를 만들거면 무조건 안전하게 만들라 <br />
1) varargs 매개변수 배열에 아무것도 저장하지 않는다 <br />
2) 그 배열(혹은 복제본) 을 신뢰할 수 없는 코드에 노출하지 않는다 

---

# item33. 타입 안전 이종 컨테이너를 고려하라 

- 자바 내에서 Map&lt;K,V>, Set&lt;E>등의 컨테이너에서 매개변수화할 수 있는 타입의 수는 제한.
- <b>타입 안전 이종 컨테이너 패턴</b> :여러 타입을 안전하게 담을 수 있는 컨테이너를 말한다. 
- Map&lt;Class&lt;?>, Object> 이런 식으로 컨테이너 자체가 아닌 컨테이너의 키를 타입으로 사용하고, 검색할 때에도 타입을 지정해서 불러오는 식
- Class(타임 토큰)를 사용하는 방식이기 때문에 List<String> 같은 건 저장할 수 없다. (List, List<String> 모두 같은 Class)
- Super type token (스프링의 ParameterizedTypeReference)
- getAnnotation 
- .asSubClass 
