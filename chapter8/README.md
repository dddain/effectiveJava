# 8장. 메서드
(메서드 설계할 때 주의할 점, ex- 매개변수, 반환값, 메서드 시그니처 설계, 문서화)

***

# item 49. 매개변수가 유효한지 검사하라.
- @throws, @exception 태그 - 이름이 다를 뿐 동일하게 사용
  <pre>
  <code>
    /**
     * 파일 삭제
     *
     * @throws java.lang.SecurityException 보안 관련
     * @exception FileNotFoundException 지정된 파일을 찾을 수 없습니다
     */
    public void deleteFile() throws SecurityException {

    }
  </code>
  </pre>
- requireNonNull
: null체크를 위한 메소드, <br />
  입력된 값이 null 이면 NPE를 발생시키며, 그렇지 않으면 입력값을 반환 <br />
  <b>사용 이유 </b> <br />
  - 빠른 실패 (오류는 가능한 한 빨리)
  - 명시성과 가독성 
  
- assert 
: private 메소드 시에는 유효한 값만이 메서드에 넘겨질 것을 보장할 수 있으므로, 
  assert를 사용하여 매개변수 유효성을 검증할 수 있다고 하였는데,<br />
 
  <pre>
  <code>
  public static void notNull(@Nullable Object object, Supplier<String> messageSupplier) {
		if (object == null) {
			throw new IllegalArgumentException(nullSafeGet(messageSupplier));
		}
	} 
	// null일 시에 예외처리하는 메소드 notNull 
  
  /**
  * Assert that an object is null.
  * Assert.isNull(value, "The value must be null");
  * Params:
  * object – the object to check
  * message – the exception message to use if the assertion fails
  * Throws:
  * IllegalArgumentException – if the object is not null
  */
  public static void isNull(@Nullable Object object, String message) {
    if (object != null) {
  	  throw new IllegalArgumentException(message);
    }
  }

  /**
  * Assert that an object is {@code null}.
  * @deprecated as of 4.3.7, in favor of {@link #isNull(Object, String)}
  */
    @Deprecated
    public static void isNull(@Nullable Object object) {
      isNull(object, "[Assertion failed] - the object argument must be null");
    }
    // null일 시에 예외처리하는 메소드 isNull
  </code>
  </pre>

=> 메서드나 생성자를 작성할 때에, 유효성 검사 비용이 지나치게 높거나(Collection.sort(list)) 계산 과정에서 암묵적으로
  검사가 수행될 때 제외하고, 위의 두 태그를 사용해 제약들을 문서화하고 메서드 코드 시작 부분에 명시적으로 검사하는 습관을 기르자.

***
# item 50.적시에 방어적 복사본을 만들라. 
=> 클라이언트 단으로 객체값을 리턴해줄때, 객체 레퍼런스를 함께 넘기면 레퍼런스를 통해 내부 값을 변경할 수 있으니 복사본을 만들어 사용 및 리턴해야한다. <br/>

외부 공격으로부터 Period 인스턴스의 내부를 보호하려면,
1) 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야함
2) 가변 필드의 방어적 복사본을 반환
  - 방어적 복사란 ? 새로운 객체로 감싸서 복사해주는 방법 <br />
    외부와 내부에서 주소값을 공유하는 인스턴스의 관계를 끊어주기 위함 (ex- 외부에서 변경한 객체가 내부에 영향이 끼치지 않도록) <br />
    외부와의 관계 끊기 <br />

<pre>
<code>
  // setter 변경 취약점 -> 생성자에 방어적 복사 적용 (해당 예시가 setter보다 builder를 써야하는 이유의 예가 되는듯 함)
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime()); // 방어적 복사
    this.end = new Date(end.getTime()); // 방어적 복사
    if (validation(this.start, this.end)) {
        throw new IllegalArgumentException("");
    }
  }
</code>
</pre>

<pre>
<code>
  // get으로 인스턴스를 가져와 변경할 경우 취약점 -> getter 메소드에서 방어적 복사 적용
  public Date getStart() {
      return new Date(start.getTime());
  }
  
  public Date getEnd() {
      return new Date(end.getTime());
  }
</code>
</pre>
https://velog.io/@miot2j/%EC%96%95%EC%9D%80%EB%B3%B5%EC%82%AC-%EA%B9%8A%EC%9D%80%EB%B3%B5%EC%82%AC-%EB%B0%A9%EC%96%B4%EC%A0%81-%EB%B3%B5%EC%82%AC%EB%9E%80

*** 
# item 51. 메서드 시그니처를 신중히 설계하라. 
### 메서드 이름을 신중히 짓자.
- 명명규칙 (item 68)
  - 클래스, 인터페이스 - (철자규칙) Pascal, (문법규칙) 명사(구)
  - 메서드, 필드 - (철자규칙) camel, (문법규칙) 동사 
  ... 

### 편의 메서드를 너무 많이 만들지 말자.

### 매개변수 목록은 짧게 유지 (4개 이하가 좋다). 
- 매개변수가 4개 이상이 넘어가면 아래 방법 고려
  - 여러 메서드로 쪼갠다. (직교성이 높도록.?)
  - 매개변수 여러 개를 묶어주는 도우미 클래스 
  - 빌더 패턴을 사용하여 매소드 호
***
# item 52. 다중정의는 신중히 사용하라
- Overloading 메서드는 정적으로 선택된다. 
  - 컴파일 타임, 재정의되기 전에 컴파일 타임에 실제 타입이 아닌, 그 객체가 어떤 타입으로 넘어가는지를 보고 어떤 메서드가 호출될지 결정됨 
- Override 메서드는 동적으로 선택된다. 
  - 런타임, Override된 메서드가 재정의되고, 호출 되었을 때에 
=> 매개변수 수가 같은 overloading은 하지 않는 것이 좋다. 

***

Q. <b> 파라미터 null 체크 보통 어떻게 하세요 ? </b>
( 이건 질문 아니구 걍 스터디 시간에 모든 분들께 들어보고 싶은 내용임당 )
<br /> 자바 7에 추가된 java.util.Objects.requireNonNull 메소드는 유연하고 사용하기도 편하니~ (299p)
<pre>
<code>
  if(param == null) { throws new BizException("check param"); }
  
  Objects.requireNonNull(param, "check param");
  Optional.ofNullable(getNames()).orElseThrow(() -> new Exception());
  Assert.notNull(param, "check param");
</code>
</pre>

Q. <b> clone( ) 메소드의 주의사항에 대해서 알고 싶어요!! </b>
<br /> 관련 내용 : 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone 을 사용해서는 안된다 (304p)
<br /> => 
<pre>
<code>
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime()); // 방어적 복사
    this.end = new Date(end.getTime()); // 방어적 복사
    if (validation(this.start, this.end)) {
        throw new IllegalArgumentException("");
    }
  }
</code>
</pre>

clone을 사용하여 복사본을 만들 수도 있지만, 생성자의 매개변수가(start, end) final클래스가 아니어서, <br />
확장될 수 있는 타입이면, 방어적 복사본을 만들 때 재정의된(다른값) clone이 호출될 수 있으므로, 사용하면 안된다. <br />
clone 메소드는 깊은 복사(실제 값을 복사, 새로운 메모리 공간에 복사, 재정의) 
clone 메소드가 사용자가 정의한 값이 아닐 수 있다.
즉, clone이 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있어 사용하면 안된다.
<br />
<pre>
<code>
CopyObject original = new CopyObject("dain", 20);
CopyObject copy = original.clone();

copy.setName("java");
original.getName(); // dain
copy.getName(); // java 
</code>
</pre>
기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는게 최고' <br />
단, 배열만은 clone 메소드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다. <br />

Q. 코드 49-2 재귀 정렬용 private 도우미 함수에서 assert가 어떻게 동작하는지 시연을 보고 싶습니다. 
<br /> => 
- 거짓인 경우시 JVM이 AssertionError를 발생시킴
- AssersionError : 응용 프로그램에서 복구할 수 없는 조건을 나타내기 위한 것, 이를 처리하거나 복구하려 하지 말 것
- 즉, Catch로 잡는 행위 등을 하려 하지 말 것

Q. P.308 편의 메서드에 대한 설명 <br />
=> 편의를 위한 메소드, 
Collections 안에 있는 모든 메서드 (swap, min, max)등, 없어도 기능 상 문제가 없어 
그대로 사용할 수 있다. 
다른 연산들로도 표현될 수 있는 메소드.
<pre>
<code>
  private static void swap(Object[] arr, int i, int j) {
    Object tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
  }
</code>
</pre>

Q. P.310 카드게임을 클래스로 만든다고 가정했을 때 rank, suit매개변수를 묶는 도우미 클래스를 만들어 하나의 매개변수로 주고받는 API의 샘플이 궁금합니다.
<pre>
<code>
	@RequiredArgsConstructor
	private static class Card {
		private final Integer rank;
		private final String suit;
	}

	public void example() {
		List<Card> cardList = new ArrayList<>();
		cardList.add(new Card(1, "circle"));
		cardList.add(new Card(2, "square"));
		cardList.add(new Card(3, "space"));

		ex2(cardList);
	}

	private List<Card> ex2(List<Card> list) {
		assert list != null;
		assert list.size() > 5; // AssertionError 발생

		return list;
	}
</code>
</pre>

Q. p.317 다중정의 해소 알고리즘(?)을 고려할 사항이 위의 예시말고 다른것이 있을지 궁금합니다.

