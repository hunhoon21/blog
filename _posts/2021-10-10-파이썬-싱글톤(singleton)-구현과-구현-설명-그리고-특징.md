안녕하세요, 이번에는 파이썬에서 싱글톤을 3가지 방식으로 직접 구현해보고 각 특징에 대해서 이야기해보고자 합니다.

* [싱글톤이란](#싱글톤이란)
* [파이썬에서 싱글톤 구현](#파이썬에서-싱글톤-구현)
  * [1. 클래스 메서드를 이용한 구현](#1.-클래스-메서드를-이용한-구현)
    * [1.1 구현 설명](#1.1-구현-설명)
    * [1.2 특징](#1.2-특징)
  * [2. `__new__` 메서드를 이용한 구현](#2.-`__new__`-메서드를-이용한-구현)
    * [2.1 구현 설명](#2.1-구현-설명)
    * [2.2 특징](#2.2-특징)
  * [3. metaclass를 이용한 구현](#3.-metaclass를-이용한-구현)
    * [3.1 구현 설명](#3.1-구현-설명)
    * [3.2 특징](#3.2-특징)
* [참고자료](#참고자료)

## 싱글톤이란
싱글톤 패턴을 따르는 객체는 프로세스에서 하나만 존재하는 객체입니다. 즉, 생성자를 여러번 호출하더라도 같은 객체가 불려지게 됩니다. 언제 불리든 같은 데이터를 전달해주어야 하는 Database Manager를 구현하거나 전역에서 같은 객체를 사용하고 싶은 경우 싱글톤 패턴을 사용할 수 있습니다. 자세한 내용은 [위키피디아](https://en.wikipedia.org/wiki/Singleton_pattern)를 참고하시기 바랍니다 :)

## 파이썬에서 싱글톤 구현
파이썬에서 직접 싱글톤을 구현해보고 각 구현한 방식에 대한 설명과 특징에 대해 이야기합니다.

### 1. 클래스 메서드를 이용한 구현
첫번째는 클래스 메서드를 이용하여 싱글톤을 구현하는 방식입니다. 파이썬에서 속성(Attribute)를 다루는 오픈소스 프레임워크 `Traitlets`에서도 해당 방식으로 싱글톤을 [구현](https://github.com/ipython/traitlets/blob/main/traitlets/config/configurable.py)하고 있습니다. 참고로 우리가 자주 사용하는 `Jupyter`에서도 내부는 `Traitlets` 를 사용하여 싱글톤을 구현합니다.
아래는 `Traitlets`에서 [구현](https://github.com/ipython/traitlets/blob/main/traitlets/config/configurable.py) 한 코드입니다.
```python
class Singleton():
    _instance = None

    @classmethod
    def _walk_mro(cls):
        """Walk the cls.mro() for parent classes that are also singletons

        For use in instance()
        """
        for subclass in cls.mro():
            if issubclass(cls, subclass) and \
                    issubclass(subclass, Singleton) and \
                    subclass != Singleton:
                yield subclass

    @classmethod
    def clear_instance(cls):
        """unset _instance for this class and singleton parents.
        """
        if not cls.initialized():
            return
        for subclass in cls._walk_mro():
            if isinstance(subclass._instance, cls):
                # only clear instances that are instances
                # of the calling class
                subclass._instance = None

    @classmethod
    def instance(cls, *args, **kwargs):
        # Create and save the instance
        if cls._instance is None:
            inst = cls(*args, **kwargs)
            # Now make sure that the instance will also be returned by
            # parent classes' _instance attribute.
            for subclass in cls._walk_mro():
                subclass._instance = inst

        if isinstance(cls._instance, cls):
            return cls._instance
        else:
            raise MultipleInstanceError(
                "An incompatible sibling of '%s' is already instanciated"
                " as singleton: %s" % (cls.__name__, type(cls._instance).__name__)
            )

    @classmethod
    def initialized(cls):
        """Has an instance been created?"""
        return hasattr(cls, "_instance") and cls._instance is not None


class MyClass(Singleton):
    pass


my_class = MyClass.instance()
my_class is MyClass.instance()
# True
```

#### 1.1 구현 설명
* `_walk_mro` 클래스메서드는 간단히 말해 내가 만든 클래스의 부모 클래스들(싱글톤 클래스까지)을 얻어내는 메서드라고 볼 수 있다. 내가 만든 클래스와 부모 클래스들이 같은 클래스 변수 (`_instance`)를 갖게 하기 위한 용도로 사용된다.
* `initialized` 클래스메서드는 해당 클래스가 싱글톤 객체를 가지고 있는지를 확인하기 위한 용도이다.
* `clear_instance` 클래스메서드는 해당 클래스가 가지고 있는 싱글톤 객체를 지우기 위한 용도이다.
* `instance` 클래스메서드는 실제로 싱글톤 객체를 만들고 클래스 변수(`_instance`)에 저장한다. 그리고 해당 싱글톤 객체를 리턴하여 싱글톤 방식을 구현해낸다.

#### 1.2 특징
* Lazy-Evaluation 기법을 이용하여 생성 시점에 싱글톤 객체를 생성한다.
* call 속도도 빠르고, C/C++에서 자주 사용하는 방식과 비슷하다고 한다.
* 상속을 받아 구현하기 때문에 오버라이딩 문제가 생길 수 있다.
* 보통의 객체 생성과 다르게 `Class.instance()` 구문을 이용하기 때문에 사용자가 헷갈릴 수 있다.

### 2. `__new__` 메서드를 이용한 구현
```python
class Singleton():
  _instance = None
  def __new__(cls, *args, **kwargs):
    if not isinstance(cls._instance, cls):
        cls._instance = super().__new__(cls, *args, **kwargs)
    return cls._instance


class MyClass(Singleton):
    pass

my_class = MyClass()
my_class is MyClass()
# True
```

#### 2.1 구현 설명
* `__new__` 메서드에서 생성된 객체를 클래스 변수 `_instance` 에 담아둔다.
* 이후에 객체 생성을 하는 경우에는 클래스 변수에 담긴 객체를 전달해준다.
* `__init__` 메서드는 객체에 속성을 할당하는 역할을 하고 실제 객체 생성 자체는 `__new__` 메서드에서 진행된다는 점을 알아야 한다.

#### 2.2 특징
* 일반적인 클래스처럼 사용할 수 있기 때문에 심리적인 장벽(?)이 낮다
* 위와 같이 상속을 받아 구현하기 때문에 본인이 만드는 클래스가 `__new__` 메서드를 오버라이딩 하는 경우에 문제가 생긴다.
* 싱글톤 구현 방식 중에 느린편에 속한다.

### 3. metaclass를 이용한 구현
```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]


class MyClass(metaclass=Singleton):
    pass


class MySecondClass(metaclass=Singleton):
    pass


a = MyClass()
a is MyClass()
# True

b = MySecondClass()
b is MySecondClass()
# True

print(Singleton._instances)
# {
#     <class '__main__.MyClass'>: <__main__.MyClass object at 0x1118d9be0>,
#     <class '__main__.MySecondClass'>: <__main__.MySecondClass object at 0x110869b80>
# }

```

#### 3.1 구현 설명
* 클래스의 클래스인 메타클래스를 사용한 구현방식이다.
* 메타클래스 `Singleton`의 클래스 변수에 싱글톤 객체들이 담기게 된다.
* 내가 만든 클래스가 생성하면서 메타클래스를 사용하게 되고, 메타클래스의 `__call__` 메서드를 거치게 된다.

#### 3.2 특징
* 메타클래스를 사용하기 때문에 오버라이딩에 대한 걱정을 하지 않아도 된다.
* 메타클래스를 사용하는만큼 진입장벽이 꽤 높다.
  * 메타클래스를 잘 설명한 [스택오버플로우](https://tech.ssut.me/understanding-python-metaclasses/) 에서 메타클래스에 대해 아래와 같이 표현한다.
  > 메타클래스는 99%의 사용자는 전혀 고려할 필요가 없는 흑마법입니다. 만약 당신이 이게 정말로 필요할지에 대한 의문을 갖는다면, 필요하지 않습니다. (이게 진짜로 필요한 사람은 이게 진짜로 필요하다고 알고 있으면서, 왜 필요한지에 대해 설명할 필요가 없는 사람들입니다.)

## 참고자료
* [traitlets repository](https://github.com/ipython/traitlets)
* [여러가지 싱글톤(singleton) 구현방법](https://wikidocs.net/3693)
* [[Python] 파이썬으로 Singleton 패턴 구현하기](https://jroomstudio.tistory.com/41)
* [파이썬 싱글턴 패턴](https://velog.io/@kimsehwan96/%ED%8C%8C%EC%9D%B4%EC%8D%AC-%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4)