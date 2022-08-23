# 6장 열거 타입과 애너테이션 

- 클래스의 일종인 <b>Enum</b> , 인터페이스의 일종인 <b>Annotation</b> <br />
특수한 목적의 참조 타입 두 가지

***

# item 34. int 상수 대신 enum 타입을 사용하라 
- 클래스인 Enum은 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다 

<pre>
<code>
public enum Apple {
    FUJI(3)
}

=>

public enum Apple {
    public static final Apple FUJI = new Apple(3);
}
</code>
</pre>

- enum 상수는 값 뿐만 아니라 로직도 가지고 있을 수 있다. <i>abstract</i> 를 이용하여
<pre>
<code>
case PLUS : return x + y;

=> abstract method 사용
PLUS("+") {
    public double apply(double x, double y) { 
    return x + y;
},

=> lambda 를 쓰면 더 깔끔 
PLUS("+", (x,y) -> x+y),

public abstract double apply(double x, double y);
</code>
</pre>

- 전략 열거 타입 패턴 (strategy enum pattern)
  (기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 좋음)

***

# item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라 

- Enum의 API 문서에 ordinal 메서드가 "EnumSet, EnumMap과 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다"
와 같이 서술되어 있는 것 처럼, 왠만하면 사용하지 말고 열거 타입 상수의 숫자 인스턴스 필드에 저장해서 위치 파악하자

***

# item 36. 비트 필드 대신 EnumSet을 사용하라

- 비트 필드란 ? 정수형이나 그 외의 자료형을 비트 단위로 저장할 수 있는 
<pre>
<code>
public class Text {
  public static final int STYLE_BOLD = 1 << 0; // 01 << 0 = 01(2) = 1
  public static final int STYLE_ITALIC = 1 << 1; // 01 << 1 = 10(2) = 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 01 << 2 = 100(2) = 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 01 << 3 = 1000(3) = 8

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int style) {
      System.out.println("Style : " + style);
    }
}


public static void main(String args[]) {
  Text text = new Text();
  text.applyStyles(Text.STYLE_UNDERLINE | Text.STYLE_STRIKETHROUGH);
  <b>// 12, 비트별 더해서</b>
}
</code>
</pre>

- 자바의 비트 필드 활용을 찾아보려 했으나 잘 찾아지지 않고 C언어의 구조체에서 주로 쓰이는 것 같다.<br /> <br />

- C언어에서 비트 필드 구조체 사용 
<pre>
<code>
#include <stdio.h>

struct Flags {
    unsigned int a : 1;     // a는 1비트 크기
    unsigned int b : 3;     // b는 3비트 크기
    unsigned int c : 7;     // c는 7비트 크기
};

int main()
{
    struct Flags f1;    // 구조체 변수 선언

    f1.a = 1;      //   1: 0000 0001, 비트 1개
    f1.b = 15;     //  15: 0000 1111, 비트 4개
    f1.c = 255;    // 255: 1111 1111, 비트 8개

    printf("%u\n", f1.a);    //   1:        1, 비트 1개만 저장됨
    printf("%u\n", f1.b);    //   7:      111, 비트 3개만 저장됨
    printf("%u\n", f1.c);    // 127: 111 1111, 비트 7개만 저장됨

    return 0;
}
</stdio.h>
</code>
</pre>


### EnumSet 
- EnumSet은 추상클래스, 
- JDK에서는 RegularEnumSet, JumboEnumSet 2가지의 EnumSet 구현체를 제공하고 있음
- RegularEnumSet : 비트 벡터를 표현하기 위해 단일 long자료형, long의 각 비트는 enum값 <br />
열거형 i 번째 비트에 저장되므로 값이 있는지 여부를 쉽게 알 수 있다.  
(64비트 데이터 -> 64개 원소 저장 가능)
(- 비트 벡터 : 중복되지 않는 정수 집합을 비트로 나타내는 방식 ex) {1,2,6}  <br />
  &nbsp;&nbsp;&nbsp;&nbsp;   0 1 2 3 4 5 6 <br />
  => 0 1 1 0 0 0 1)
- JumboEnumSet : long 요소의 배열을 비트 벡터로 사용 -> 64개 이상의 원소. 저장된 배열 인덱스를 <br />
찾기 위해 몇가지 추가 계산을 수행하므로, 속도는 RegularEnumSet이 더 빠를 듯 

***

# item 37. ordinal 인덱싱 대신 EnumMap을 사용하라 
<pre>
<code>
Class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL } 
  final String name;
  final LifeCycle lifeCycle;

  Plant (String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }
}

Set&lt;Plant&gt;[] plantsByLifeCycle = (Set&lt;Plant&gt;[]) new Set[Plant.LifeCycle.values().length];
for(int i=0; i&lt;plantsByLifeCycle.length; i++) {
  plantsByLifeCycle[i] = new HashSet<>(); // heap pollution 

  for(Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p); 
    // enum의 ordinal메소드로는, 정확한 값을 plantsByLifeCycle에 넣지 못할 것 
  }

  // 결과 
  for (int i=0; i&lt;plantsByLifeCycle.length; i++) {
    // Plant.LifeCycle.values()[i], plantsByLifeCycle[i]
    // XX 
  }
}
</code>
</pre>

## EnumMap 
: 내부에서 배열 사용, Map의 타입 안정성 + 배열의 성능 모두 얻어냄 
<pre>
<code>
Map&lt;Plant.LifeCycle, Set&lt;Plant&gt;&gt; plantsByLifeCycle = new EnumMap&lt;&gt;(Plant.LifeCycle.class);
for(Plant.LifeCycle lc : Plant.LifeCycle.values()) {
  plantsByLifeCycle.put(lc, new HashSet&lt;&gt;());
}
for(Plant p : garden) {
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}
// 안전하지 않은 형변환은 쓰지 않고
// map의 키인 열거 타입이 그 자체로 출력용 문자열을 제공 
</code>
</pre>

***

# item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 
- 연산 코드용 인터페이스(ex) double apply(double x, double y) 를 정의하고, 열거타입(BasicOperation, ExtendedOperation)이 이 인터페이스를 구현하게 하라.
- 중복량이 적을 때 연산 코드용 인터페이스를 확장하여 사용하면 되지만, 공유하는 기능이 많다면 그 부분은 별도의 도우미 클래스나 정적 도우미 메서드로 분리해야한다.

***

# item 39. 명명 패턴보다 애너테이션을 사용하라
- @Test ( @Retention(RetentionPolicy.RUNTIME), @Target(ElementType.METHOD))
- 배열 방식 대신 반복 가능 애너테이션을 적용할 것 
  - isAnnotationPresent vs getAnnotationByType
  
- 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다. 애너테이션 사용할 것 !
  - 오타가 나면 ...
  - 올바른 프로그램 요소에서만 사용되리라 보증이...
  - 프로그램 요소를 매개변수로 전달할 마땅한 방법이...

***


Q. item36. 223p가장 맨 밑~ 224p : 
정수 상수보다 열거 타입을 선호하는 프로그래머 중에도 상수 집합을 주고 받아야 할 때는 여전히 비트 필드를 사용하기도 한다. <br />
자바에서 비트 필드를 활용하는 예가 있을까요? 검색하면 C, C++ 에서만 활용하는 예제가 나와서 자바에서 비트필드를 활용하는지 궁금합니다. 
=> 보통 비트 단위 작업은 임베디드 시스템, 하드웨어 드라이버, 네트워크 프로토콜, 이진 파일 형식 및 문자 인코딩, 암호화 등 하드웨어
제어 수준에서 사용하여 보통의 프로그래머가 사용할 일은 없습니다.

Q. item38. 235p : 
자바 라이브러리도 이번 아이템에서 소개한 패턴을 사용한다. 그 예로, java.nio.file.LinkOption 열거 타입은 
CopyOption과 OpenOption 인터페이스를 구현했다. <br />
<pre>
<code>
public enum LinkOption implements OpenOption, CopyOption {
    /**
     * Do not follow symbolic links.
     *
     * @see Files#getFileAttributeView(Path,Class,LinkOption[])
     * @see Files#copy
     * @see SecureDirectoryStream#newByteChannel
     */
    NOFOLLOW_LINKS;
}
</code>
</pre>
=> LinkOption 열거 타입에 해당 내용만 있어서,, 어떻게 CopyOption, OpenOption 인터페이스를 구현하여
이번 아이템에서 얘기한 것과 같이 CopyOption, OpenOption이 어떻게 정의되었고, LinkOption이 어떻게 이 인터페이스를 구현하는지 궁금합니다.!!

=> StandardOpenOption, StandardCopyOption enum 클래스가 더 적합해보임 


***

# item 40. @Override 애너테이션을 일관되게 사용하라 
- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자.
- 일관되게 사용하라 => 재정의할 의도였으나 실수로 새로운 메서드를 추가했을 때 경고를 주므로.
- 추상메서드를 정의할 때엔 애너테이션을 달지 않아도 되지만 (단다고 해서 해로울 것 없으니 달자).

# item 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라 
- 아무 메서드도 담고 있지 않고, 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스 : 마커 인터페이스 
- EX ) Serializable 
- 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, <br />
  마커 애너테이션은 그렇지 않다. <br />
  마커 애너테이션을 사용했다면 컴파일타임에 잡을 수 있다. <br />
  serializable 마커 인터페이스는 그 대상이 직렬화 할 수 있는 타입인지를 확인, but Object를 받도록 설계되어 있어 런타임에 잡음. <br />
  마커 인터페이스를 사용하는 주요 이유 중 하나가 컴파일타임 오류 검출인데 serializable 마커 인터페이스는 이 이점을 살리지 못함. <br />
  ⇒ 마커 인터페이스, 마커 애너테이션 모두 컴파일 타임에 에러 잡기 위함인데, serializable은 object를 받도록 설계되어 있어 런타임에야 에러를 잡는다.. 그 얘긴가
 
<p>
- 새로 추가하는 메서드 없이 단지 타입 정의가 목적 : 마커 인터페이스 <br />
- 클래스나 인터페이스 외의 요소에 마킹 or 애너테이션을 적극 활용하는 프레임워크의 일부로 그 마커를 편힙하고자 : 마커 애너테이션
</p>

