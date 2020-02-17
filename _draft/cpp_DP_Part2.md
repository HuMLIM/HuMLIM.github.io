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