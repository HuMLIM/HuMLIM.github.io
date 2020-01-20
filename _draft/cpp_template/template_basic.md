### template basic draft page
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

