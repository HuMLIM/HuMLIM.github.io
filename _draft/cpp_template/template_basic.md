### cpp template draft page
---
tamplate 기본  
- 기본 모양
    - 함수 template
    ```cpp
    template<typename T>
    T square(T a)
    {
        return a * a;
    }
    ```
    - 클래스 template
    ```cpp
    template<typename T>    //template<class T> 의 형태도 가능함
    class Complex
    {
        T re, im;
    public:
        Complex(T r, T i) : re(r), im(i) {}
    };
    ```
- Template 장점
    - macro는 전처리기가 code를 생성하기 때문에 인자를 통해 type을 추론 할 수 없다.
    - 필요한 형태(type)의 code만 생성하기 때문에 overhead를 감소 시킬 수 있다.(e.g. int로 가능한 코드를 double로 생성하는 overhead 감소)

- Instantiation
    - complier에 의해 실제 code가 생성 되는 것을 instantiation(인스턴스화)라고 한다.
    - template는 틀을 제공해 주는 것이고 그것이 사용 되었을 때 instantiation 한다.
    - instantiation은 명시적(explicit)/암시적(implicit) instantiation가 있다.
        - 명시적 instantiation  
            1. code 생성을 명시적으로 지시, 함수/클래스 선언과 유사한 형태로 작성된다.
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

        - 암시적 instantiation
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