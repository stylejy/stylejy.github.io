---
layout: post
title: stringbuilder, stringbuffer
date: 2021-05-24
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

이번 블로그에서는 stringbuilder와 string buffer에 대해서 이어서 적어보려고 합니다.  


이전 블로그에서 확인하면 알 수 있지만 immutable한 string은 concat이나 + 기호로 변화시키려고 하면 변화되지 않고 계속 새로운 메모리를 소모합니다. 하지만 stringbuilder, stringbuffer는 mutable합니다.  
stringbuilder나 stringbuffer를 사용하여 내용을 변화시키면 새로운 메모리에 할당하지 않고, 처음 할당된 메모리에 변화된 내용이 저장됩니다.  

다음의 코드를 봅시다!  
```java
StringBuilder stringBuilder = new StringBuilder("Hello ");
StringBuffer stringBuffer = new StringBuffer("Hello ");

System.out.println("stringBuilder String : " + stringBuilder);
System.out.println("stringBuffer String : " + stringBuffer);
System.out.println();

System.out.println("stringBuilder identityHashCode : " + System.identityHashCode( stringBuilder));
System.out.println("stringBuffer identityHashCode : " + System.identityHashCode(stringBuffer));
System.out.println();

stringBuilder.append("World");
stringBuffer.append("World");

System.out.println("stringBuilder identityHashCode : " + System.identityHashCode( stringBuilder));
System.out.println("stringBuffer identityHashCode : " +System.identityHashCode( stringBuffer));
System.out.println();

System.out.println("stringBuilder String : " + stringBuilder);
System.out.println("stringBuffer String : " + stringBuffer);
System.out.println();
```
<br>
이렇게 stringBuilder와 stringBuffer를 선언하여 문자열("Hello")로 초기화하고 각각 문자열("World")을 append합니다.  
그리고 stringBuilder와 stringBuffer의 문자열들과 identityHashCode를 출력해보면 다음과 같이 확인할 수 있습니다.  
<br>

```
stringBuilder String : Hello 
stringBuffer String : Hello 

stringBuilder identityHashCode : 905654280
stringBuffer identityHashCode : 592179046

stringBuilder identityHashCode : 905654280
stringBuffer identityHashCode : 592179046

stringBuilder String : Hello World
stringBuffer String : Hello World
```
> stringBuilder와 stringBuffer의 문자열이 내용이 변했지만 참조값은 변하지 않았음을 알 수 있습니다!  
> stringBuilder와 stringBuffer는 mutable하다!  


그렇다면 stringBuilder와 stringBuffer의 차이점은 어떤 것이 있을까요?  
결론부터 말하자면, stringBuffer는 동기화를 지원하고 stringBuilder는 그렇지 못 합니다.  
> 동기화란?  
> 간단히 설명하면, 두 개 이상의 프로세스 혹은 스레드가 공유 자원을 접근 및 변경할 때, race condition이 발생하지 않도록 하는 것입니다.  

  
#### 코드를 한 번 까봅시다!  

StringBuilder와 StringBuffer는 모두 AbstractStringBuilder를 구현하고 있습니다. 하지만 특정 함수 구현 방법에 차이가 있습니다. 

-- StringBuilder.class 
```java
...
@Override
@HotSpotIntrinsicCandidate
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
...
```

-- StringBuffer.class 
```java
...
@Override
@HotSpotIntrinsicCandidate
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
...
```

append할 때에 StringBuilder는 synchronized 키워드가 없고, StringBuffer는 있는 것을 확인할 수 있습니다.  
<br>
#### 이제 정말 그런지 예제로 확인해봅시다!

```java
StringBuilder builder = new StringBuilder();
StringBuffer buffer = new StringBuffer();

// Thread 1 --> 0을 append하는 thread
Executors.newSingleThreadExecutor().execute(() -> {
    for (int i = 0; i < 10; i++) {
        builder.append(0);
        buffer.append(0);
        try {
            sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});

// Thread 2 --> 1을 append하는 thread
Executors.newSingleThreadExecutor().execute(() -> {
    for (int i = 0; i < 10; i++) {
        builder.append(1);
        buffer.append(1);
        try {
            sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});

sleep(10000);
System.out.println("stringBuilder string: " + builder);
System.out.println("stringBuffer  string: " + buffer);
```
stringbuilder(이후 builder)와 stringbuffer(이후 buffer)를 하나씩 만든 후에 thread 2개를 만듭니다.  
builder와 buffer에 0을 append하는 thread 1개 (이후 thread-0), 1을 append하는 thread 1개 (이후 thread-1)를 만듭니다.  
이럴 경우 콘솔에 어떻게 찍히는 지를 보면, 

```
stringBuilder string: 01101001010011001
stringBuffer  string: 01101001011001010101
```

stringBuilder의 길이가 17, stringBuffer의 길이가 20임을 확인할 수 있습니다.  2개의 thread가 각각 loop를 10번 돌았기 때문에 길이는 20이어야 합니다.  
stringBuilder는 17인 이유는 동기화를 지원하지 않기 때문입니다. thread-0과 thread-1이 동시에 builder에 append를 하여 둘 중 하나의 저장하는 연산이 생략되버린 것입니다.  
> builder에서 race codition이 발생했습니다.  
반면에, buffer는 동기화를 지원하기 때문에 thread-0이 연산할 때 lock을 걸어 thread-1이 연산하지 못 하도록 합니다. 그래서 생략되는 연산 없이 길이가 20이 될 수 있었습니다.    

#### 결론
멀티 스레드 환경에서는 StringBuffer를 사용하고, 단일 스레드 환경에서는 StringBuilder를 사용하는 것이 좋아보입니다.  
> 단일 스레드 환경에서는 StringBuilder가 StringBuffer보다 동일 연산에서 더 나은 성능을 보여줍니다.  
> lock을 걸지 않기 때문입니다.  



