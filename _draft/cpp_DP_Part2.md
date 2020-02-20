---
permalink: 
layout:
---
## Design Pattern part2

- 변하지 않는 코드안에 있는 변해야 하는 부분은 분리 하는 것이 좋다.

- Template method pattern
    - 변해야 하는 부분(Validation 정책)을 별도의 가상 함수로 분리한다.
    - 변하는 것을 가상함수로 분리 할때의 장점
        - Validation 정책을 변경 하고 싶다면 파생 클래스를 만들어서 가상함수를 재정의 하면 된다.
    - 상속 기반 통신
    - 실행 시간의 정책을 교체 할 수 없어 유연성이 떨어진다
    ```cpp
    #include <conio.h>  // getch()사용을 위해

    //Edit 클래스는 모든 문자 입력 가능 클래스
    class Edit
    {
        string data;
    public: 
        //변해야 하는 부분의 가상함수
        virtual bool validate(char c)
        {
            return true;
        }

        string getData()
        {
            data.clear();

            while( 1 )
            {
                char c = getch();

                if ( c == 13 ) break;

                if ( validate(c) )  //변해야 하는 부분
                {
                    data.push_back(c);
                    cout << c;
                }
            }
            cout << endl;
            return data;
        }
    };

    // DigitEdit는 숫자만 입력 가능
    class DigitEdit : public Edit
    {
    public:
        virtual bool validate(char c)
        {
            return isdigit(c);
        }
    };

    int main()
    {
        Edit edit1;    //모든 문자 입력 객체
        DigitEdit edit2;  //숫자만(ex.전화번호) 입력 객체

        while( 1 )
        {
            string s1 = edit1.getData();
            cout << s1 << endl;

            string s2 = edit2.getData();
            cout << s2 << endl;
        }
    }

    ```

- Strategy pattern (Composition)
    - 변해야 하는 부분을 다른 클래스로 만들어 분리한다.
    - 인터페이스 기반 통신
    - 정책을 재 사용할 수 있고(다른 클래스에서 사용가능)
    - 실행 시간의 정책을 교체할 수 있다.
    ```cpp
    // validation을 위한 인터페이스
    struct IValidator
    {
        virtual bool valicate(string s, char c) = 0;
        virtual bool iscomplete(string s) { return true; }  //초기값은 true로 하고 필요한 class에서 재정의 하여 사용한다.

        virtual ~IValidator() {}
    };

    class Edit
    {
        string data;
        IValidator* pVal = 0;   //0은 유효성 검사 정책이 없다는 뜻이 된다.
    public:
        void setValidator(Ivalidator* p) { pVal = p; }

        string getData()
        {
            data.clear();

            while( 1 )
            {
                char c = getch();

                if ( c == 13 && //enter 입력
                    ( pVal == 0 || pVal->iscomplete(data) ) ) break;
                if ( pVal == 0 || pVal->validate(data, c) )
                {
                    data.push_back(c);
                    cout << c;
                }
            }
            cout << endl;
            return data;
        }
    };

    class LimitDigitValidator : public IValidator
    {
        int value;
    public:
        LimitDigitValidator(int n) : value(n) {}

        virtual bool validate(string s, char c)
        {
            return s.size() < value && isdigit(c);
        }

        virtual bool iscomplete(string s)
        {
            return s.size() == value;
        }
    };
    
    int main() 
    {
        Edit edit;

        // edit 클래스에 유효성 정책 연결
        LimitDigitValidator v1(5);
        edit.setValidator(&v1);

        // edit 클래스의 정책을 실시간에 변경 할 수 있다.
        LimitDigitValidator v2(10);
        edit.setValidator(&v2)

        while( 1 )
        {
            string s = edit.getData();

        }

    }
    ```
- Policy Base Design
    - Strategy pattern의 가상함수는 성능저하가 약간 있다.
    - 정책 클래스를 template 인자를 inline 함수로 교체하는 기술을 단위 전략 디자인(Policy Base Design) 이라고 한다
    - 컴파일 시간에 정책을 결정하기 때문에 실행 시간에 교체 할 수 없다.(코드에 유형에 따라 선택한다.)
    ```cpp
    template<typename T, typename ThreadModel> class List
    {
        ThreadModel tm; //동기화 정책을 담은 클래스
    public:
        void push_front(const T& a)
        {
            tm.Lock();  //ThreadModel(정책)에 따라 inline 함수로 치환되기 때문에 빠르다.
            //...리스트에 값을 저장하는 구현부라고 가정
            tm.Unlock();
        }
    };

    class MutexLock
    {
        //mutex m;
    public:
        inline void Lock() { cout << "mutex Lock" << endl; }
        inline void Unlock() { cout << "mutex Unlock" << endl; }
    };

    class NoLock
    {
        //mutex m;
    public:
        inline void Lock() {}
        inline void Unlock() {}
    };

    List<int, MutexLock> s1;    //전역변수는 tread에 안전하지 않다.
    List<int, NoLock> s2;

    int main()
    {
        s1.push_front(10);
        s2.push_front(10);
    }
    ```
||Stratege pattern|Policy Base Design|
|:--:|:--------------:|:----------------:|
| 속도 | 가상함수 기반 / 느리다 | 템플릿 인자 사용 / 인라인 치환 가능 / 빠르다 |
| 유연성 | 실행시간 정책 교체 가능 | 컴파일 시간 정책 교체 가능 |

- Application Framework
    - Application의 전체적인 흐름을 라이브러리가 가지고 있는 것을 application framework라고 함
    - 미리 정의 된 실행 흐름을 라이브러리 내부에 감추고 사용자는 파생 클래스를 만들고 사용자 클래스를 전역객체로 생성
    - MFC, QT, WxWidget, IOS, Android app등에서 사용
    
    ```cpp
    // 라이브러리 부분
    class CwinApp;  // 클래스 전방 선언 
                    // : 클래스가 완전히 선언되기 전이라도 포인터는 사용할 수 있도록 하는 방법

    CWinApp* g_app = 0;

    class CWinApp
    {
    public:
        CWinApp() { g_app = this; }
        virtual bool InitInstance() { return false; }
        virtual int ExitInstance() { return false; }
        virtual int Run() { return 0; }
    };

    int main()  
    {
        if (g_app->InitIstance() == ture)
            g_app->Run();
        g_app->ExitInstance();
    }
    // C/C++은 main함수가 객체가 될 수 없기 때문에 g_app을 이용하여 CWinApp의 member 함수 처럼 사용
    // main 함수가 CWinApp class 안에 있는 함수라면 template method와 형태
    ```

    ```cpp
    // 라이브러리 사용자
    /* 
    라이브러리 사용 가이드
        1. CWinApp의 파생 클래스를 만든다
        2. 사용자 클래스를 전역객체로 생성 한다.
    */
    class MyApp : public CWinApp
    {
        virtual bool InitInstance() 
        {
            cout << "Init Application" << endl;
            return true; 
        }

        virtual int ExitInstance()
        {
            cout << "Exit Application" << endl;
            return true;
        }
    };
    MyApp theApp;

    /* 
    호출 순서
        1. 전역변수 생성자.(MyApp 생성자)
        2. 기반 클래스 생성자 (CWinApp 생성자)  // CWinApp생성자에서 theApp의 주소를 g_app에 보관 한다.
        3. main 함수 실행
    ```
- 일반함수에서의 가변성
    - 변해야 하는 것(정책)을 함수의 인자화 한다.
    ```cpp
    void Sort(int* x, int sz)
    {
        for (int i = 0; i < sz-1; i++)
        {
            for (int j = i + 1; j < sz; j++)
            {
                if (x[i] < x[j])    // 변해야 하는 것
                    swap(x[i], y[j]);
            }
        }
    }

    void Sort_FP(int* x, int sz, bool(*cmp)(int, int ))
    {
        for (int i = 0; i < sz-1; i++)
        {
            for (int j = i + 1; j < sz; j++)
            {
                if (cmp(x[i], y[j]))
                    swap(x[i], y[j]);
            }
        }
    }

    template<typename T>
    void Sort_T(int* x, int sz, T cmp)
    {
        for (int i = 0; i < sz-1; i++)
        {
            for (int j = i + 1; j < sz; j++)
            {
                if (cmp(x[i], y[j]))
                    swap(x[i], y[j]);
            }
        }
    }

    bool cmp1(int a, int b) { return a < b; }
    bool cmp2(int a, int b) { return a > b; }

    int main()
    {
        int x[10] = { 1, 3, 5, 7, 9, 2, 4, 6, 8, 10 };
        Sort(x, 10);    // 정책(내림차순/올림차순)을 변경하려면 라이브러리를 수정해야 함
        Sort_FP(x, 10, &cmp1);    // 함수 포인터를 사용 
                                    // 사용자가 정책 변경이 가능하지만 함수포이터는 인라인 치환이 되지 않는 단점이 있다.
        Sort_T(x, 10, [](int a, int b) { return a > b; });  // Template을 사용하여 인라인 치환이 가능하다.
                                                            // 정책이 많을 경우 코드 메모리가 증가한다.

        for (auto n : x)
        {
            cout << n << end;
        }
    }
    ```
- State Pattern
    - template method(상속)는 객체에 대한 (상태)변화가 아닌 클래스에 의한 변화
    - stratege pattern은 클래스가 아닌 객체에 대한 (상태)변화
    - stratege pattern과 state pattern은 구조상 차이가 없어 보인다.(클래스 다이어그램 동일)

    | stratege pattern | state pattern |
    | :--------------: | :-----------: |
    | 다양한 알고리즘이 존재하고 적합한 알고리즘을 선택(변경) 하는 구조 | 다양한 상태가 존재하고 변화 상태에 따라 행위(동작)을 선택(변경) 하는 구조 | 
    