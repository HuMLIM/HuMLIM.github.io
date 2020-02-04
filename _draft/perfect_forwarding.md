
- perfect forwarding : 래퍼함수가 인자를 받아서 원본 함수에게 완벽하게 전달하는 개념
```cpp
#include <iostream>
using namesapce std;

void goo(int& a) { cout << "goo" << endl; a = 30; }
void foo(int  a) { cout << "foo" << endl; }

template<typename F, typename A>
//void chronometry(F f, A arg) // arg를 인자로 받을 경우 perfect forwarding이 되지 않는다.
//void chronometry(F f, A& arg) // arg가 상수(rvalue)일 경우 error
void chronometry(F f, const A& arg) // const& 타입을 일반 &로 받을 수 없다. goo에서 errro
{
    f(arg);
}

int main()
{
    int n = 0;
    chronometry(&goo, n);
    chronometry(&foo, 5);

    cout << n << endl; // n이 30이 나와야 perfect forwarding
}
```