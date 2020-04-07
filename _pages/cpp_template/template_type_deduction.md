---
permalink: /categories/cpp_template/template_type_deduction
layout: single
---
### class template type deduction(추론)
---
- Type dedcution 기본
    - 함수 template은 인자의 type을 보고 template의 type deduction이 가능하지만
    - class template은 type deduction이 불가능 하다.(단 C++17 부터는 가능)
    - class의 type을 명시하지 않고 사용하기 위해서는 user가 type을 명시 해야 한다. (C++17)
        ```cpp
        
        template<typename T> class Vector
        {
            T* buff;
        public:
            Vector() {}
            Vector(int sz, T initValue) {}

            template<typename Container> Vector(Container& c) {} //복사 생성자
        }
        //user define deduction guide
        Vector()->Vector<int>;  // v3를 위한 deduction guide
                                // compiler가 type을 결정할 수 있도록
                                // user가 type정보 명시

        template<typename Container>
        Vector(Container&)->Vector<typename Container::value_type>  // v4를 위한 deduction guide

        int main ()
        {
            Vector<int> v1(10, 3); // 일반적인 class template 사용 방식
                                   // 생성할 때 type을 넘겨준다.
            Vector v2(10, 3);      // C++17부터 가능한 방법

            Vector v3; 

            list<int> s = {1, 2, 3};
            // list s = {1, 2, 3}   // (C++17)
            Vector v4(s);
        }
        ```
        > v4의 type을 결정할 때 사용된 [typename]()과 [value_type]()은 링크 참조

    - 즉, C++17부터는 class template도
        1. 생성자의 인자를 통해서
        2. 생성자로 추론 할 수 없는 경우 user define deduction guide를 통해 type deduction이 가능하다.   
            ==> 단, 비추. error발생 되는 것이 더 나을 수도 있다.

- Object Generator
    - C++17 이전 버전에서 class template이 type deduction이 되지 않는 불편함을 개선하는 방법
        ```cpp
        template<typename T> std::list<T>* make_list(int sz, T value)
        {
            return new std::list<T>(sz, value);
        }

        int main()
        {
            auto p1 = new std::list<int>(10, 1); // class template을 사용하면 <int>를 생략 할 수 없다.
            auto p2 = make_list(10, 1); // 함수 template이므로 인자를 전달할 필요 없이
                                        // 1로 부터 int를 추론한다.
        }
        ```
    - 이와 같은 원리로 C++ 표준에서는 pair와 tuple을 만들기 쉽도록 make_pair(), make_tuple()같은 함수를 제공한다.

- identity<>
    - 인자 추론을 막기 위한 방법(반드시 template인자를 user가 넘겨주어야 한다.)
        ```cpp
        template<typename T> void foo(typename identity<T>::type a) {}

        foo<int>(3); // T는 int이고 identity<T>::type도 int가 되어 OK
        foo(3);      // identity가 class template이므로 컴파일러가 3을 가지고 type을 추론 할 수 없다. ERROR
        ```
    - 사용예 : [perfect forwarding]()

- template argument type deduction
    - 규칙 1. template 인자가 value type 일 때
        - 인자가 가진 const, volatile, reference 속성을 제거하고 T type을 결정 한다.
        - type의 const가 아닌 인자의 const속성이 제거되는 것임
    
