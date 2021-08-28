# Ch1. C++ 표준 라이브러리 초단기 속성 코스

c++과 표준 라이브러리 핵심요소, 문법 기쵸 부분이여서 c++-17 버전에서 새롭게 소개된 문법이나 
중요한 내용만 간략하게 remind하는 수준으로 작성함.

## namespace

- namespace는 코드에서 이름이 서로 충돌되는 문제를 해결하기 위해 나온 개념임.
- `::` 스코프 지정 연산자 (scope resolution operator)를 이용함.
- `using` 지사자를 이용하면 namespace 앞 접두어를 생략할 수 있음. 
- 한 소스파일에서 `using` 지시자를 여러 개 지정할 수 있지만 남용하면 안된다. 
- 헤더 파일안에는 절대로 `using` 문을 작성하면 안된다. 
이렇게 하면, 해당 헤더 파일을 include하는 모든 파일에서 using 문으로 지정한 방식으로 호출해야함.
- 중첩된 namespace (nested namespace) 
  - 아래 코드 처럼 복잡하게 선언하지 않고, c++17 부터는 nested namespace를 통해 간결하게 선언할 수 있음. (`AAA::BBB::CCC`)
  ```
    namespace AAA {
      namespace BBB {
        namespace CCC {
          ...
        }
      }
    }
  ```
- 네임스페이스 앨리어스 (namespace alias)
  - 길이가 긴 namespace를 다른 이름으로 또는 간략하게 표현 가능
  ```
    namespace XXX = AAA::BBB::CCC
  ```

| 궁긍한점 

- 다른 헤더파일에 중복된 이름의 namespace를 호출하는 경우 ?
- nested namespace에서 이름이 중복되는 경우? 
- 표준 라이브러리 지원 함수 (예: `ceil()`)와 동일한 이름의 함수가 정의된 namespace상에서 표준 라이브러리 함수를 사용하고 싶은 경우?



## 열거 타입 (enumerated type)
- 변수에 지정할 수 있는 값의 범위를 엄격하게 제한하며, 아래 코드와 같이 각 멤버에 해당하는 정수 값을 지정할 수 있음. 지정하지 않으면, 이전 멤버의 값에서 1증가시킨 값이 자동으로 대입됨.
```
  enum PiceType {PieceTypeKing = 1, PieceTypeQueen, PieceTypeRock=10, PieceTypePawn}

  # PieceTypeQueen = 2, PieceTypePawn = 11
```

## enum class
- `enum`은 원래 타입을 엄격하게 따지지 않음. 항상 정수로 해석되기 때문에 선언한 형태와 관계 없이 `enum` 타입을 서로 비교할 수 있음. 
- type-safe한 `enum`을 사용하기 위해서는 `enum class`를 사용함.
- `enum class`로 정의한 열거 타입 값들은 자동으로 스코프가 확장되지 않아서 스코프 지정 연사자로 명시해줘야 함.
- `enum class`로 정의한 열거 타입 값은 자동으로 정수 타입으로 변환되지 않음.
  ```
  # 잘못된 코드
  if (PieceType::Queen == 2) {...}

  # 명시적 형변환을 해줘야 함.
  if (static_cast<int>(PieceType::Queen) == 2) {...}

  # 내부 표현 타입 변경
  enum class PieceType : unsigned long {
    King = 1,
    Queen,
    Rook = 10,
    Pawn
  };
  ```

## if 문의 initializer (초기자)
- c++17 부터 추가된 기능
- initializaer에서 선언한 변수는 if문 밖에서는 사용할 수 없음.

```
# 구조
if ( <이니셜라이저> ; <조건문> ) { <본문> }

# 예시
if (Employee employee = GetEmployee(); employ.salary > 1000) {
  ...
}
```

## switch fallthrough
- switch문으로 조건으로 지정한 값과 일치하는 case문이 없으면 `break`문 나올때까지 실행하는데, 이때 `break`문이 없다면 다음에 나오는 case문도 계속 실행하는 것
- 버그가 발생하기 쉬움, 컴파일러에서는 경고 메시지 발생하기도 함. c++17부터 `[[fallthrough]]` 속성 지정하면 의도적으로 fallthrough 방식으로 작성했다고 컴파일러에게 알려줄 수 있음.

```
switch (backgroundColor) {
  case Color::DarkBlue:
    foo();
    [[fallthrough]];
  case Color::Black:
    bar();
    break;
};
```

## switch 문의 initializer (초기자)
- if문과 동일
```
switch (<이니셜라이져> ; <표현식>) { <본문> }
```

## 함수 리턴 타입 추론 (`auto`)
- c++14 부터 함수 리턴 타입을 컴파일러가 알아서 지정할 수 있음. (`auto` 키워드)
  ```
  auto addNumbers(int num1, int num2) {
    return num1 + num2;
  }
  ```

## 배열 크기 확인
- 스택 기반 C 스타일의 배열 크기는 c++17부터 `std::size()` 함수로 알아낼 수 있음.
  ```
  #include <array>

  # c style
  unsigned int arraySize = sizeof(myArray) / sizeof(myArray[0]);

  # c++ style
  unsigend int arraySize = std::size(myArray);
  ```

## 구조적 바인딩 (Structured Binding)
- 여러개의 변수를 선언할 때 배열, 구조체, 페어 또는 튜플의 값으로 초기화 가능함.
- 구조적 바인딩을 사용하려면 반드시 `auto`를 사용해야 함.
- 배열뿐만 아니라, 모든 멤버가 `non-static` 이면서 `public`으로 선언된 데이터구조라면 `struct`, `pair`, `tuple` 등 적용 가능함.

  ```
  std::array<int , 3> values {11, 22, 33};

  # 왼쪽 선언할 변수 갯수와 오른쪽에 나온 표현식 갯수가 동일해야 함.
  auto [x, y, z] = values;
  ```

## 범위기반 for문
- 컨테이너에 담긴 원소에 대해 반복문을 실행하는데 편리하게 사용가능함.
- C스타일의 loop, `initialize list`, `array`, `vector`, 그리고 `begin()` , `end()` 메소드 (컨테이너 반복자) 가 정의된 모든 타입에 적용가능함.
- for 문을 돌면서 모든 원소에 대한 **복제본**을 출력함. (복제본 안쓰려면 참조자 사용)

  ```
  std::array<int, 4> arr = {1, 2, 3, 4};
  for (int i : arr) {
    std::cout << i << std::endl; 
  } 
  ```

## 스마트 포인터
- 스마트 포인터로 지정한 객체가 스코프를 벗어나면, 자동으로 메모리가 해제됨.
- `std::unique_ptr` : return 문을 실행하거나 exception이 발생해도 항상 할당된 메모리는 **해제 가능** 함.
  
  ```
  # c++14 부터 가능
  auto anEmplyoee = make_unique<Employee>();

  # 배열선언 예제
  auto employees = make_unique<Employee[]>(10);
  cout << "Salary: << employee[0].salary << endl;
  ```

- `shared_ptr` : 데이터 **공유** 가능, 대입연산이 발생할 때마다 레퍼런스 카운트 증가 `make_shared<>`사용. 

  ```
  auto anEmployee = make_shared<Employee>();
  if (anEmployee) {
    cout << "Salary: << anEmployee->salary << endl;
  }

  # 배열을 저장할 때는 shared_ptr 사용 불가
  shared_ptr<Employee[]> employees(new Employee[10]);
  cout << "Salary: << employees[0].salary << endl;
  ```

## const
- c++ 에서는 프로그램 실행동안 변경하면 안되는 값에 `#define`으로 선언하는 대신 `const`로 정의하는것이 바람직함.
- `#define`은 **전처리기**가 처리하고 `const`는 컴파일러가 처리한다. 전처리기는 텍스트 매칭 수준의 작업만 수행하는 반면, 컴파일러는 c++ 코드 문맥 안에서 처리하기 때문에 `const`로 정의할 대상에 타입이나 스코프를 적용할 수 있는 장점이 있음.

## const 매개변수
- `non-const` 변수를 `const` 변수로 캐스팅할 수 있음. 
- 함수 매개변수 변경되지 않도록 보장할 수 있음.
  ```
  void mysteryFunction(const std::string* someString) {
    *someString = "Test"; // compile error !
  }

  int main()
  {
    std::string myString = "The String";
    mysteryFunction(&myString);
    return 0;
  }
  ```

## 레퍼런스
- c++ 에서는 값 전달 (pass by value) 방식보다 레퍼런스 (참조) 전달 방식 (pass by reference)을 제공함.
- 원본에 대한 포인터만 전달되기 때문에 원본 자체를 복제할 필요가 없어서 성능이 우수함.

  ```
  void addOne_v1(int i) {
    i++; // 복제본 (정수 값)이 전달됐기 때문에 원본에는 아무런 영향 안끼침
  }

  void addOne_v2(int &i) {
    i++; // 원본 변수가 변경됨
  }

  int main() {
    addOne_v1(3); // 가능함.
    addOne_v2(3); // compile error! (literal 지정 불가)
  }
  ```

## const 레퍼런스 전달방식
- 원본에 대한 값이 복제되지도 않고, 원본 변수가 변경되지도 않는 장점을 모두 취할 수 있음.
  ```
  void printString(const std:;string& myString) {
    std::cout << myString << endl;
  }

  int main() {
    std::string someString = "Hello World";
    printString(someString);
    printString("wow"); // 리터럴 전달 가능!
    return 0;
  }
  ```

## auto
- `auto` 지정하면 레퍼런스와 `const` 한정자가 사라진다. 
`const` 반환하는 함수 값을 `auto` 타입으로 지정한 변수에 저장하면 값이 복제되버리는 경우가 있음. 
- `const autu&`를 사용하면 됨.

  ```
  const std::string& foo();

  auto f1 = foo();          // const 한정자 사라짐, 값 복제됨.
  auto& f2 = foo();         // cosnt 한정자 사라짐, 값 복제 안됨.
  const auto&  f3 = foo();  // ok
  ```

## delctype
- 인수로 지정한 표현식의 타입을 알 수 있음.
- **template** 사용할 때 아주 효과적임.
  ```
  delctype(foo()) f2 = foo();
  ```

## 유니폼 초기화
- `struct`, `class` 모두 **중괄호 초기화** 하도록 통일됨. (c++11 이후)
  ```
    class CircleClass {
      public:
        CircleClass(int x, int y, double radius)
          : mX(x), mY(y), mRadius(radius) {}

      private:
        int mX, mY;
        double mRadius;
    }

    CircleClass m1 = {10,10,2.5};

    # 등호 생략 가능
    CircleClass m2{10, 10,2.5};
  ```

- zero 초기화 가능
  ```
  int d{}; // 유니폼 초기화, d = 0으로 초기화됨.
  ```
- 축소변환 (narrowing) 방지 가능 

## 직접리스트 초기화와 복제리스트 초기화
- c++17 이전까지는 복제/직접 리스트 모두 `initialize_list<>`로 처리하였음.
- c++17 부터는 `auto`는 직접리스트 초기화에 대해 값 하나만 추론이 가능하다.
  ```
  # 복제리스트 초기화
  auto a = {11};        // initialize_list<int>
  auto b = {11, 12};    // initizliae_list<int>

  # 직접리스트 초기화
  auto c = {11};        // int
  auto d = {11, 12};    // error!
  ```














