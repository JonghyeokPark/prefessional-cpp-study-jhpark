# Ch2. 스트링과 스트링 뷰 다루기

c++ std::string 클래스, std::string_view와 raw string literal 에 대해 다룬다.

## 스트링 리터럴

- 스트링 리터럴 (e.g. `"something"`)은 내부적으로 메모리의 **읽기 전용** 영역에 저장됨.
- 스트링 리터럴을 여러번 참조해도 컴파일러는 해당 리터럴에 대해 메모리 공간을 단 하나만 할당함 (literal pooling).
- 변수에 대입할 수는 있지만, 변수에 저장하면 안됨. `const char*` 사용

## raw 스트링 리터럴 

- 여러 줄에 걸쳐 작성한 스트링 리터럴, 이스케이스 시퀀스를 일반 텍스트로 취급함.

  ```
  // escape sequence 불필요

  const char *str = "Hello "World"!"    // error
  const char *str = "Hello \"World\"!"  // escpae sequence 사용

  const char *str = R"(Hello "World"!)";
  ```

  ```
  // 띄어쓰기도 가능

  const char *str = "Line 1\n Line 2";

  const char *str = R"(Line1
  Line2)";
  ```

## `std::string` 

- C Style 스트링의 단점

  ```
  1. 스트링 데이터 타입에 대한 고차원 기능을 구현하려면 힘듦.
  2. 찾기 힘든 메모리 버그가 발생하기 쉬움.
  3. C++의 객체지향적인 특성을 제대로 활용하지 못함.
  4. 프로그래머가 내부 표현 방식을 이해해야함.
  ```

- `std::string` 클래스 활용 

  ```
  /* 스트링 append 연산 */
  string A('12');
  string B('34');
  C = A + B;            // C = '1234' 

  /* 스트링 비교 연산 */
  char * a = "12";
  char b[] = "12";

  if (a == b) {
    ...                // c에서는 포인터 값 비교하기 때문에 항상 false 반환
  }
  ```

- 메모리 누수 발생하지 않음.
- 오버로딩을 통해 연산자를 원하는 방식으로 동작할 수 있음. 
- `c_str()` 함수를 통해 c언어에 대한 호환성 보장 가능.
- c++17 부터는 `non-const` 스트링에 대해 `c_str()` 호출하면 `char*`를 리턴하도록 변경하였음.

## `std::string` 리터럴

- 스트링 리터럴을 사용하기 위해서는 문자열 뒤에 `s`를 사용한다. 
- `using namespace std` 또는 `using namespace std::string_literals`를 추가해야 함.

```
using namespace std::string_literals

auto string1 = "Hello World";   // const char* 타입
auto string2 = "Hello World"s;  // std::string 타입
```

## 숫자 변환

- 숫자를 string으로 변환 : `to_string()` 함수를 사용하면 됨. (다양한 타입에 맞게 오버로딩 되어있음.)
- string을 숫자로 변환 : `base` 변환할 수의 밑, `idx` 변환되지 않은 부분의 맨 앞에 있는 문자 인덱스 

  ```
  int stoi(const string& str, size_t idx=0, int base=10);
  long stol(const string& str, size_t idx=0, int base=10);
  unsigned long stoul(const string& str, size_t idx=0, int base=10);

  ...

  ```

## 로우레벨 숫자 변환

- `<charconv>` 헤더파일에 정의됨. 
- 메모리 할당 관련 작업은 안해주기 때문에 호출한 측에서 버퍼를 할당하는 방식으로 사용해야 함.

  ```

  // 정상적으로 반환되면 ptr은 끝에서 두번째 문자를 가르키고, 
  // 그렇지 않으면 last와 같음. (ec == errc::value_too_large)

  struct to_chars_result {
    char *ptr;
    int ec;
  }

  to_chars_result to_chars (char* first, char* last, IntegerT value, int base = 10);

  ```
- c++17 구조적 바인딩과 연동한 예제

  ```
  std::string out(10, ' ');
  auto [ptr, ec] = std::to_chars(out.data(), out.data() + out.size(), 12345);
  if (ec == std::errc()) { /* ok */ }
  ```

- 부동소숫점 표현 

  ```
  enum class chars_format {
    scientific,      
    fixed,
    hex,
    general = fixed | scientific
  }
  ```
- 스트링을 숫자로 표현할 수 있음 `from_chars_result from_chars(cost char* first, const char *last, IntegerT value , int base=10)`

## `std::string_view` 

- `const char*` 매개변수로 지정하면, `.data()` 또는 `.c_str()` 함수를 통해 변환해야함 : `std::string` 객체지향 속성과 헬퍼 메소드 사용하지 못함.
- `std::string &` 매개변수로 지정하면 항상 `std::string`만 사용해야 함. (리터럴 복사본 오버헤드 문제 발생)
- `std::string_view`를 사용하면 `const string&` 대신 사용가능, string을 복사하지 않기 때문에 오버헤드도 없음.

  ```

  string_view string_view_func (string_view x) {
    return x;
  }

  void string_view_test (const string& test) { /* ... */ }

  // 잘못된 함수 호출 예
  string_view_test(string_view_func("My TEST"));

  // 올바른 예
  string_view_test(string_view_func("MyTest").data());
  string_view_test(string(string_view_func("MyTest")));

  ```

## string_view 리터럴

- `sv`를 사용하면 `string_view` 리터럴을 선언할 수 있음.

  ```
  using namespace std::string_view_literals;
  
  auto sv = "my string_view literal"sv;
  ```