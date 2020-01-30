---
permalink: 
layout:
---
### cpp ref : random_device 클래스(C++11)
- 0 ~ 2^32(4294967295) 범위의 비결정적(Non-deterministic) 정수 난수 생성기
    > Non-deterministic : 한가지 행동에 여러가지 결과를 가질수 있음

    ```cpp
    //random_device class
    class random_device { // class to generate random numbers (from hardware where available)
    public:
        using result_type = unsigned int;

        explicit random_device() { // construct
        }

        explicit random_device(const string&) { // construct
        }

        _NODISCARD static constexpr result_type(min)() { // return minimum possible generated value
            return 0;
        }

        _NODISCARD static constexpr result_type(max)() { // return maximum possible generated value
            return static_cast<result_type>(-1);
        }

        _NODISCARD double entropy() const noexcept { // return entropy of random number source
            return 32.0;
        }

        _NODISCARD result_type operator()() { // return next value
            return _Random_device();
        }

        random_device(const random_device&) = delete;
        random_device& operator=(const random_device&) = delete;
    };
    ```
    - 사용 예제

    ```cpp
    #include <random>
    #include <iostream>

    using namespace std;

    int main()
    {
        random_device gen;

        cout << "entropy == " << gen.entropy() << endl;
        cout << "min == " << gen.min() << endl;
        cout << "max == " << gen.max() << endl;

        cout << "random value == " << gen() << endl;
        cout << "random value == " << gen() << endl;
    }
    ```
    <details><summary>OUTPUT</summary>

        entropy == 32
        min == 0  
        max == 4294967295  
        a random value == 2378414971
        a random value == 3633694716
    </details>

    - 단점
        - 느리다
