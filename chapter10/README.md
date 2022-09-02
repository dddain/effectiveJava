# 10장. 예외

***

# item 69. 예외는 진짜 예외 상황에만 사용하라. 
- 예외는 오직 예외 상황에서만 써야 한다. 절대 일상적인 제어 흐름용으로 사용해선 안됨 

# item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라.
- 호출하는 쪽에서 복구하리라 여겨지면 CheckedException
- 프로그래밍 오류를 나타낼 때는 RuntimeException <br />
=> 호출 가능 여부가 모호할 때는 RuntimeException 
  
# item 71. 필요 없는 검사 예외 사용은 피하라.
- CheckedException 이 안전성을 높이긴 하지만, 남용하면 오히려 부작용 
- 옵셔널을 반환해도 될지 고민해보고,
- 옵셔널만으로는 상황을 처리하기에 충분한 정보를 제공할 수 없을 때 RuntimeException

# item 72. 표준 예외를 사용하라.
- 예외도 다른 코드와 마찬가지로 코드를 재사용하는 것이 좋음 
- 표준 예외를 재사용하는 것이 협업, 유지보수에 좋으므로 표준 예외를 사용하자 

# item 73. 추상화 수준에 맞는 예외를 던져라.
- 수행하는 일과 관련 없어 보이는 예외가 튀어나오는 당황스러운일이 발생하지 않기 위해 <b> 예외 번역 / 예외 연쇄 </b>
- ex) List(상위 계층) <br />
  AbstractSequentialList에서 발생하는 예외 > 저수준 예외 <br />
<pre>
<code>
예외 번역
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
        // AbstractSequentialList 상위 계층인 List interface's method get에 명시된 필수사항 
    }
}

예외 연쇄 
class HigerLevelException extends Exception { // Exeption extends Throwable, cause를 Throwable(Throwable)까지 건넬수 있음 
    HigherLevelExceptio(Throwable cause) {
        super(cause);
    }
}
</code>
</pre>


# item 74. 메서드가 던지는 모든 예외를 문서화 하라. 
- 메서드가 던질 가능성이 있는 모든 예외를 문서화하자. JavaDoc의 @throws태그, 
- checked exception > 메서드 선언의 throws문에 선언, unchecked exception > 메서드 선언에는 기입하지 말자.

# item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라.
- e.printStackTrace(message); 에, 실패 관련 정보를 담으면 메시지 보고 더 빨리 에러를 찾을 수 있다 

# item 76. 가능한 한 실패 원자적으로 만들라.
- 호출된 메소드가 실패하더라도 해당 객체는 메소드 호출 전 상태를 유지하도록 하라. 