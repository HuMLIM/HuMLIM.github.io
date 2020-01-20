### template instantiation draft page
---
- Instantiation(인스턴스화)
    - complier에 의해 실제 code가 생성 되는 것을 인스턴스화(instantiation)라고 한다.
    - template는 틀을 제공해 주는 것이고 그것이 사용 되었을 때 인스턴스화 한다.
    - template은 명시적(explicit) 혹은 암시적(implicit)으로 인스턴스 화 할 수 있다.
        - 명시적 인스턴스화
            1. code 생성을 명시적으로 지시한다.
            2. 함수/클래스 선언과 유사한 형태로 작성된다.
                ```cpp
                template<typename T> class Test
                {
                public:
                    void foo();
                    void goo();
                };

                template class Test<int>;   //foo, goo 모두 인스턴스화
                template void Test<double>::foo();  //foo 만 인스턴스화

                template<typename T> T square(T a)
                {
                    return a * a;
                }

                //인자를 통해 type 추론이 가능한 함수 template은
                //아래와 같은 형태로 type생략하여 명시적으로 표현 가능하다.
                template int square<int>(int);
                template int square<>(int);
                template int square(int);   
                
                int main() 
                {
                }
                ```

        - 암시적 인스턴스화
            1. 템플릿을 사용할때 인스턴스화가 자동으로 발생하며, 사용하지 않은 멤버함수는 인스턴스 화 되지 않는다.(lazy instantiation)
            2. 사용자가 타입을 직접 전달 하거나 컴파일러가 타입을 추론 하여 생성한다.(template argument type deduction)
                ```cpp
                template<typename T> class Test
                {
                public:
                    void foo() {}
                    void goo() {}
                };

                template<typename T> T square(T a)
                {
                    return a * a;
                }

                int main()
                {
                    int n1 = square<int>(3);    //기본적인 함수 template의 사용
                    int n2 = square(3);         //인자를 통해 type을 추론

                    Test<int> t1;       // 기본적인 클래스 template의 사용
                    t1.foo();           // foo만 인스턴스화 된다(goo 인스턴스화x)(lazy instantiation)
                }
                ```