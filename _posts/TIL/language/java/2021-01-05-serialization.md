---
title:  "[Java] 직렬화 (Serialization)"
date: 2021-01-05
excerpt: ""
tags: [til, java, language]
categories: [til/language/java]
---

이 글은 Oracle 포스트를 읽고 개인적으로 정리&느낀점을 정리한 글입니다.
https://www.oracle.com/technical-resources/articles/java/serializationapi.html
{: .notice--info}

Java 플랫폼은 메모리에서 reusable한 객체를 생성하도록 허용하고 있다. 이러한 객체들이 **지속적으로 존재하게 하려면 (이하 persistent라는 형용사로 대체한다 ㅎㅎ) 어떻게 해야할까?** 저장해두었다가, 꺼내야할 상황이 있을 수도 있다. 네트워크 통신을 해서 Object를 송수신한다던가, 캐싱을 한다던가 말이다. `Object Serialization`을 통해서 객체를 `flattern`해서 강력한 방식으로 재사용할 수 있다.

직렬화에 관해서 **Basic Protocol, Customizing Protocol, Creating Our Own Protocol** 세 단계로 단계적으로 학습해보자!

## 1. 기본 Mechanism

기본부터 시작해보자. Java에서 객체를 지속하게 하려면 `java.io.Serializable` 인터페이스를 implement해서 직렬화를 가능케 해야 한다. 이렇게 하면 객체가 byte로 flatten 될 수 있으며, 추후 다시 되돌아올 수 있게 된다.

``` java
public class PersistentTime implements Serializable {
  private Date time;

  public PersistentTime() {
    time = Calendar.getInstance().getTime();
  }

  public Date getTime() {
    return time;
  }
}
```

여기서 `Serializable` 인터페이스는 완벽히 empty 하다. 아무것도 없다는 뜻이다. 그저 객체가 직렬화될 수 있다는 `marker` 인터페이스로 이해하면 된다.

이렇게 만들어진 객체를 어떻게 지속할까? 방법은 `java.io.ObjectOutputStream` class가 해준다. 이 class stream은 파일 시스템이나 소켓에 쓰기를 가능하게 해준다.

다음은, 파일스트림에 객체를 쓰고 다시 읽어오는 예제이다.

``` java
PersistentTime time = new PersistentTime();
FileOutputStream fos = null;
ObjectOutputStream out = null;

try {
  fos = new FileOutputStream(filename);
  out = new ObjectOutputStream(fos);
  out.writeObject(time); // serialization mechanism 발동! 객체가 byte로 변환된다.
  out.close();
} catch(IOException ex) {
  ex.printStackTrace();
}
```

`out.writeObject(time)` 에서 serialization mechanism이 시작되고 객체가 byte로 변환된다. 다시 이를 restore 하려면 스트림을 `readObject()`로 다시 읽으면 된다.

이렇게 하면 보존된 객체를 저장하고, 다시 그 객체를 읽어들일 수 있다.

## 1-1. Nonserializable 객체

`Serializable` 만 implement 하면 객체를 보존할 수 있다. 하지만 특정 Type은 보존되지 않아야 할 수도 있다. `Thread`, `OutputStream`, `Socket` 등이 그 예다. 그냥 생각만 해봤을 때 스레드가 내 컴퓨터에서 영원이 보존되고 있다고 생각해보자. None sense at All. 따라서 `transient`를 붙여서 이 항목은 보존하지 않을 것이라 명시해주면 된다.

``` java
public class PersistentAnimation implements Serializable, Runnable {
	transient private Thread animator;
	private int animationSpeed;

	public PersistentAnimation(int animationSpeed) { ... }
 }
```


**Rule #1: 객체를 persistant 하게 유지하기 위해서는 `Serilizable`을 implement해야 한다.** <br/>
**Rule #2: nonserializable field에는 `transient`를 붙여주자**
{: .notice--info}

## 2. Customize 기본 Protocol

이러한 기본 Serialize 규약에서, 나만의 규약을 Customizing할 수도 있다.

위에서 계속 예시를 들었던 `PersistentAnimation`을 보면, `animationSpeed`를 계속 기록을 해둔다. 이 객체를 fileStream을 써서 저장해두고 다시 꺼낼때, 다시 해당 시점부터 `start()`를 재개하고 싶다고 가정해보자.

생성자에서 한번 start 하므로, 객체를 다시 restore 해도 start는 되지 않는다. 그래서 아래와 같은 기능을 구현하자.

- 생성자에서 startAnimation()을 하게 만든다.
- `readObject` 할때 startAnimation()을 다시 하게 만든다.

``` java
public class PersistentAnimation implements Serializable, Runnable {

	transient private Thread animator;
	private int animationSpeed;

	public PersistentAnimation(int animationSpeed) {
		this.animationSpeed = animationSpeed;
		startAnimation();
	}

	private void writeObject(ObjectOutputStream out) throws IOException {
		out.defaultWriteObject();
	}

	private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
		// our "pseudo-constructor"
		in.defaultReadObject();
		// now we are a "live" object again, so let's run rebuild and start
		startAnimation();
	}

	private void startAnimation()
	{
		animator = new Thread(this);
		animator.start();
	}
 }
```

`private void writeObject(ObjectOutputStream out) throws IOException;`

`private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException;`

위 두 메소드를 선언해서, Serialize 가 시작될 때, **java에서 자동으로 이 메소드를 수행하게 만드는 것이다.** (커스터마이징 하는 것이다!)

객체가 `Serializable`를 implement 하고 있는지 체크하고, 저 두 private 메서드가 있는지 확인한 뒤, 있다면 stream class가 파라미터로 넘겨져서 작동하게 된다

이러한 커스터마이징은 여러군데에서 응용될 수 있다. Encryption/Decryption을 할수도 있으며, version 관리에 쓰일 수도 있겠다!
{: .notice--info}

## Stop That Serialization!

만약 super class가 serializable 한데, 그를 상속한 sub class 는 serializable 하지 않게 하기 위해서는 어떻게 해야할까?

``` java
private void writeObject(ObjectOutputStream out) throws IOException {
	throw new NotSerializableException("Not today!");
}
private void readObject(ObjectInputStream in) throws IOException {
	throw new NotSerializableException("Not today!");
}
```

또 `writeObject` 와 `readObject` 메서드를 빌려서 Exception을 떨구기만 하면 된다!

## 3. Create Your Own Protocol : the Externalizable Interface

`Serializable` 인터페이스 대신에 `Externalizable` 인터페이스를 implement하면 아무 제약 없이 나만의 프로토콜을 만들 수 있다. 예를 들어, 내가 PDF 파일을 읽는 방법을 알고 있다면, PDF-specific 한 프로토콜을 `writeExternal` 과 `readExternal`에 넣어서 활용하면 된다.

## 참고 : Caching Objects in the Stream

`ObjectOutputStream`에 객체의 상태를 N번 변경하는 코드이다.
``` java
ObjectOutputStream out = new ObjectOutputStream(...);
MyObject obj = new MyObject(); // must be Serializable
obj.setState(100);
out.writeObject(obj); // saves object with state = 100
obj.setState(200);
out.writeObject(obj); // does not save new object state
```

마지막에 바꾼 내용은 stream 에 저장되지 않는다. 왜냐하면 stream은 초기의 객체를 참조하고 있기 때문이다.
이러한 문제를 해결하려면 항상 stream을 닫아주거나 `ObjectOutputStream.reset()` 으로 초기화해준 다음에 사용해야 한다.

## 참고 : Version Control

Serialize 한 객체 파일이 있고, 이 객체에 필드를 추가한 경우, 기존에 Serialize한 객체를 읽으려고 하면 어떻게 될까?
불행히도 `java.io.InvalidClassException` 이 발생한다. 왜냐하면 모든 `persistent-capable` 클래스는 자동으로 UID를 가지고 있기 때문이다.

만약 객체의 field를 추가하거나, 삭제했을 경우에도 serializable하게 하려면 UID를 고정적으로 가지고 있으면 된다.

``` java
static final long serialVersionUID = 10275539472837495L;
```

다만 객체의 구조 자체를 바꾼다거나, 객체가 implement 하고 있던 Serializable 인터페이스를 제거해버리면 이 UID로도 커버 못하기 때문에 에러가 발생하겠지요?


## 결론

이렇게 `Serialization` 에 대해 나름 깊게 알아봤다. Dto를 생성할 때 Serializable를 implement한 객체를 봤었고, UID도 봤었다. 그 때는 복붙하기에 급급했는데.. 이렇게 자세히 알게 되니까 후련하기도 하다.

백기선님 유튜브에서 이 글이 재밌다고 한번 보라고 해서 봤는데 잘 본것 같다!

## 참고문서
- [Oracle 공식문서](https://www.oracle.com/technical-resources/articles/java/serializationapi.html)
- [백기선님 유튜브](https://www.youtube.com/watch?v=xBVPChbtUhM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=3)
