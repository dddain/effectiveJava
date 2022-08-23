# 9장. 일반적인 프로그래밍 원칙

***

# item 65. 리플렉션보다는 인터페이스를 사용하라. 
- 사용 이유 : 런타임에 메소드, 클래스, 인터페이스의 행위를 검사 혹은 수정하기 위해 <br />
  (반대로, 컴파일 타임의 타입 검사 불가)
- 하지만, 
    - 대비 코드를 정확하게 작성해두지 않으면 런타임 오류가 발생할 수 있으며, 
    - 성능이 떨어진다 : 리플렉션을 통한 메소드 호출은 일반 메소드 호출보다 훨씬 느리다.
    
=> 강력한 기능이지만, 단점이 많으므로 꼭 필요한 경우에만 사용하도록 

--- 

# item 66. 네이티브 메서드는 신중히 사용하라. 
- 네이티브 메서드 : 네이티브 프로그래밍 언어로 작성한 메서드 
<pre>
<code>
Native language is a language that can run on the platform without converting 
it first, or to do anything with it. Example is C/C++ on Symbian. 
(But, actually it have to converted to binary machine code first)

Managed language is a language that must be converted or interpreted 
before it can be run on the platform. Example is Java on Windows, or .NET

Dynamic language is a language that allows you to convert some a string 
to an integer without a lot of things. Example is python.
</code>
</pre>
- 성능을 개선해주는일이 많지 않다. 
- 가비지 컬렉터가 네이티브 메모리 자동 회수하지 못함.
- 네이티브 메서드와 자바 코드 사이 접착코드 작성 필요. 
=> 네이티브 메서드 또한 꼭 필요한 경우에만 사용하도록 
  
--- 

# item 67. 최적화는 신중히 하라.
- 빠른 프로그램보다 좋은 프로그램을 !
- API를 설계할 때 성능에 주는 영향을 반드시 고려하라 !
- 잘 설계된 API는 성능도 좋다. 그러니 성능을 위해 API를 왜곡하는 것은 안 좋은 생각 !
- 각각의 최적화 시도 전후로 성능을 측정하라 
- 최적화 도구 > 프로파일링도구, jmh
=> 이것도 왠만하면 하지말기 

# item 68. 일반적으로 통용되는 명명규칙을 따르라 
