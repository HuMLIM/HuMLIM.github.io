---
permalink: 
layout:
---
## Design Pattern part4
- adapter pattern
    - 이미 구현되어 있는 다른 클래스(또는 객체)를 인터페이스만 수정하여 사용
    - 호환성이 없던 클래스를 연결해서 사용 할 수 있다.
    ```cpp
    /*text_view.h*/
    class TextView
    {
        std::string data;
        std::string font;
    public:
        TextView(std::string s, std::string fo = "나눔고딕") : data(s), font(fo) {}

        void Show() { cout << data << endl; }
    };

    /*shape_editor.cpp*/
    #include "text_view.h"

    class Shape 
    {
    public:
        virtual void Draw() = 0;
    };

    // TextView를 도형 편집기에서 사용하기위해 인터페이스 변경 클래스 adapter(상속)
    class Text : public TextView, public Shape  // 다중 상속
    {
    public:
        Text(string s) : TextView(s) {}
        virtual void Draw() { TextView::Show(); }
    };

    // TextView를 도형 편집기에서 사용하기위해 인터페이스 변경 객체(Object) adapter(포함, 구성)
    class ObjectAdapter : public Shape
    {
        TextView* pView;    // 반드시 포인터나 참조로 받는다.
    public:
        ObjectAdapter(TextView* p) : pView(p) {}
        virtual void Draw() { pView->Show(); }
    };

    int main()
    {
        TextView tv;
        vector<Shape*> v;

        v.push_back(new Text("hello"));         // 클래스도 담을 수 있고
        //v.push_back(&tv);   // error : TextView의 객체는 Shape를 상속하고 있지 않으므로 그대로 담을 수 없다.
        v.push_back(new ObjectAdapter(&tv));    // 객체도 담을 수 있다.

        for (auto p : v)
            p->Draw();
    }
    ```
- STL adapter
    - Container adapter
        - STL에 있는 stack을 adapter를 이용해서 만들어 보자
        - Stack의 기능은 push, pop, top세가지만 있다고 가정함
    ```cpp
    #include <iostream>
    #include <list>
    #include <vector>
    #include <deque>

    using namespace std;
    /* 기본적인 adapter
    template<typename T> class Stack : public list<T>
    {
    public:
        void push(const T& a) { list<T>::push_back(a); }
        void pop() {list<T>::pop_back(); }
        T& top() { return list<T>::back(); }
    };
    */
    /* private 상속을 통해 list의 내부 함수를 사용하지 못하도록 한다.
    template<typename T> class Stack : private list<T>
    {
    public:
        void push(const T& a) { list<T>::push_back(a); }
        void pop() {list<T>::pop_back(); }
        T& top() { return list<T>::back(); }
    };
    */
    /* 포함으로 구현 
    template<typename T> class Stack
    {
        list<T> st;
    public:
        void push(const T& a) { st.push_back(a); }
        void pop() {st.pop_back(); }
        T& top() { st.back(); }
    };
    */
    /* container도 template 인자로 받아서 사용자에게 선택권을 준다. */
    /* default 인자값을 설정하여 사용성을 높인다. */
    template<typename T, typename C = deque<T>> class Stack
    {
        C st;   // 클래스 adapter : 포인터로 받지 않았기 때문에
    public:
        void push(const T& a) { st.push_back(a); }
        void pop() { st.pop_back(); }
        T& top() { return list<T>::back(); }
    };

    int main()
    {
        //Stack<int, vector<T>> s;
        //Stack<int, list<T>> s;
        Stack<int> s;
        s.push(10);
        s.push(20);

        //s.push_front(20);   // bad!!
        // list를 상속받았기 때문에 기본 adapter에서는 push_front도 사용 가능하게 된다.
        // 이 문제의 해결 방법:
        //  1. private로 list를 상송받는다.
        //  2. 상속이아닌 포함(구성)으로 구현 한다.
        cout << s.top() << endl;
    }
    ```
    - iterator Adapter
        - reverse_iterator
        - 기존 반복자의 동작을 거꾸로 동작
    ```cpp
    #include <iostream>
    #include <algorithm>
    #include <list>

    using namespace std;

    void foo(int a) { cout << a; }

    int main()
    {
        list<int> s = {1, 2, 3, 4};
        auto p1 = s.begin();
        auto p2 = s.end();

        reverse_iterator<list<int>::iterator> p3(p2); // --p2로 초기화
        //reverse_iterator<list<int>::iterator> p4(p1); // --p1로 초기화
        auto p4 = make_reverse_iterator(p1);
        
        //cout << *p3 << endl; // 4
        //++p3;
        //cout << *p4 << endl; // 1
        
        for_each(p1,p2,foo);    // 1234
        cout << endl;
        for_each(p3, p4, [](int a){ cout << a; });  // 4321
    }
    ```
- proxy stub 구조
    - proxy는 함수 호출을 명령코드로 변경하여 서버전달
    - stub은 명령코드를 다시 함수 호출로 변경하여 work수행

    <details><summary>ecource.dp.hpp</summary>

    ```cpp
    #ifndef ECOURSE_CO_KR
    #define ECOURSE_CO_KR

    #if _MSC_VER > 1000
    #pragma  once
    #endif

    #ifdef UNICODE
    #undef _UNICODE
    #undef UNICODE
    #endif

    #include <Windows.h>
    #include <iostream>
    #include <string>
    #include <sstream>
    #include <ctime>
    #include <tchar.h>
    #include <map>
    #pragma comment(lib, "kernel32.lib")
    #pragma comment(lib, "user32.lib")
    #pragma comment(lib, "gdi32.lib")
    using namespace std;

    namespace ecourse
    {
        namespace module
        {
    #ifdef _WIN32
    #define IOEXPORT __declspec(dllexport)
    #else
    #define IOEXPORT
    #endif

            void* ec_load_module(string path)
            {
                return reinterpret_cast<void*>(LoadLibraryA(path.c_str()));
            }
            void ec_unload_module(void* p)
            {
                FreeLibrary((HMODULE)p);
            }
            void* ec_get_function_address(void* module, string func)
            {
                return reinterpret_cast<void*>(GetProcAddress((HMODULE)module, func.c_str()));
            }
        }
        namespace file
        {
            typedef int(*PFENUMFILE)(string, void*);

            void ec_enum_files(string path, string filter, PFENUMFILE f, void* param)
            {
                BOOL b = SetCurrentDirectory(path.c_str());

                if (b == false)
                {
                    cout << "[DEBUG] " << path.c_str() << " directory does not exit" << endl;
                    return;
                }
                WIN32_FIND_DATA wfd = { 0 };
                HANDLE h = ::FindFirstFile(filter.c_str(), &wfd);
                do
                {
                    if (!(wfd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY) && !(wfd.dwFileAttributes & FILE_ATTRIBUTE_HIDDEN))
                    {
                        if (f(wfd.cFileName, param) == 0) break;
                    }
                } while (FindNextFile(h, &wfd));

                FindClose(h);
            }
        }

        namespace ipc
        {
            typedef int(*IPC_SERVER_HANDLER)(int, int, int);

            IPC_SERVER_HANDLER _proxyServerHandler;

            LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam)
            {
                if (msg >= WM_USER)
                    return _proxyServerHandler(msg - WM_USER, wParam, lParam);

                switch (msg)
                {
                case WM_DESTROY:
                    PostQuitMessage(0);
                    return 0;
                }
                return DefWindowProc(hwnd, msg, wParam, lParam);
            }

            HWND MakeWindow(string name, int show)
            {
                HINSTANCE hInstance = GetModuleHandle(0);
                WNDCLASS wc;
                wc.cbClsExtra = 0;
                wc.cbWndExtra = 0;
                wc.hbrBackground = (HBRUSH)(COLOR_WINDOW+1);
                wc.hCursor = LoadCursor(0, IDC_ARROW);
                wc.hIcon = LoadIcon(0, IDI_APPLICATION);
                wc.hInstance = hInstance;
                wc.lpfnWndProc = WndProc;
                wc.lpszClassName = _T("First");
                wc.lpszMenuName = 0;
                wc.style = 0;

                RegisterClass(&wc);

                HWND hwnd = CreateWindowEx(0, _T("First"), name.c_str(), WS_OVERLAPPEDWINDOW,
                    CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, 0, 0, hInstance, 0);

                ShowWindow(hwnd, show);
                return hwnd;
            }


            void ProcessMessage(IPC_SERVER_HANDLER handler)
            {
                _proxyServerHandler = handler;
                MSG msg;
                while (GetMessage(&msg, 0, 0, 0))
                {
                    TranslateMessage(&msg);
                    DispatchMessage(&msg);
                }
            }
            //------------------------------------------------------------------

            void ec_start_server(string name, IPC_SERVER_HANDLER handler, int show = SW_HIDE)
            {
                printf("[DEBUG MESSAGE] %s Server Start...\n", name.c_str());

                MakeWindow(name, show);
                ProcessMessage(handler);
            }

            int ec_find_server(string name)
            {
                HWND hwnd = FindWindow(0, name.c_str());

                if (hwnd == 0)
                {
                    printf("[DEBUG] ???? : %s Server?? ????? ???????.\n", name.c_str());
                    return -1;
                }
                return (long long)hwnd;
            }

            int ec_send_server(long long serverid, int code, int param1, int param2)
            {
                return SendMessage((HWND)serverid, code + WM_USER, param1, param2);
            }

        }
        using namespace file;
        using namespace module;
        using namespace ipc;
    }

    #endif	// ECOURSE_CO_KR
     ```
    </details>

    - server(stub) 만들기
    ```cpp
    #include <iostream>
    #include "ecource_dp.hpp"

    #include "ICalc.h"  // interface 도입

    using namespace std;
    using namespace ecource;

    //class Calc
    class Calc : public ICalc   // interface 도입
    {
    public:
        int Add(int x, int y) { return x + y; }
        int Sub(int x, int y) { return x - y; }
    };
    Calc calc;

    int dispatch(int code, int x, int y)
    {
        cout << "[DEBUG] " << code << x << y << endl;

        switch (code)
        {
        case 1: return calc.Add(x, y);
        case 2: return calc.Sub(x, y);
        }
        return -1;
    }

    int main()
    {
        ec_start_server("CalcService", dispatch);
    }
    ```
    
    - client(proxy) 만들기
        - proxy 장점
            - 사용자는 내부 로직을 알 필요 없이 함수 호출만으로 기능을 사용할 수 있다.
    ```cpp
    #include <iostream>
    #include "ecource_dp.hpp"
    // interface 도입: Server 클래스와 porxy 클래스가 동일한 함수를 가지고 있음을 보장
    #include "ICalc.h"  

    using namespace std;
    using namespace ecource;

    // Proxy
    //class Calc
    class Calc : public ICalc   // interface 도입
    {
        int server;
    public:
        Calc() { server = ec_find_server("CalcService"); }
        int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
        int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
    };

    int main()
    {
        Calc* pCalc = new Calc;

        cout << pCalc->Add(1,2) << endl;    // 3
        cout << pCalc->Sub(10,8) << endl;    // 2
    }
    ```
    ```cpp
    /* ICalc.h */
    struct ICalc
    {
        virtual int Add(int a, int b) = 0;
        virtual int Sub(int a, int b) = 0;

        virtual ~ICalc() {}
    };
    ```

    - proxy dll(동적모듈) 만들어서 배포하기
        - 사용자는 proxy의 구조 역시 몰라도 된다.
        - 단, proxy클래스의 객체를 생성하기 위한 약속된 함수가 필요하다. (CreateCalc)
        - g++ -shared CalcProxy.cpp -o CalcProxy.dll
    ```cpp
    /* CalcProxy.cpp */
    #include "ecource_dp.hpp"
    #include "ICalc.h"  

    using namespace ecource;

    class Calc : public ICalc   // interface 도입
    {
        int server;
    public:
        Calc() { server = ec_find_server("CalcService"); }
        int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
        int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
    };

    extern "C" __declspecc(dllexport) // windows 환경에서 필요한 내용 
    ICalc* CreateCalc() // CreateCalc 함수는 사용자와의 약속이므로 이름을 변경하면 안됨
    {
        return new Clac;
    }
    ```
    ```cpp
    #include <iostream>
    #include "ecource_dp.hpp"
    #include "ICalc.h"
    
    typedef ICalc* (*F)();

    int main()
    {
        // 동적 모듈 load
        void* addr = ec_load_module("CalcProxy.dll");   // 호환성을 위해 배포자는 dll이름을 바꾸면 안된다

        F f = (F)ec_get_function_address(addr, "CreateCalc")
        ICalc* pCalc = f(); // CreateCalc()

        cout << pCalc->Add(1,2) << endl;    // 3
        cout << pCalc->Sub(10,8) << endl;    // 2
    }
    ```
    - RefCount로 객체 관리하기
        - dll에 AddCount 와 DelCount를 만들고
        - 사용자가 객체를 생성했을때 AddCount호출 사용이 완료되면 delCount호출
        ```cpp
        /* ICalc.h */
        struct IRefCnt  // MS의 IUnkonwn : proxy 클래스에 모두 사용하니 공통의 기반 인터페이스로 분리함
        {
            virtual void AddCount() = 0;
            virtual void DelCount() = 0;
            virtual ~IRefCnt() {}
        };

        struct ICalc : public IRefCnt
        {
            virtual int Add(int a, int b) = 0;
            virtual int Sub(int a, int b) = 0;
            virtual ~ICalc() {}
        };

        /* CalcProxy.cpp */
        class Calc : public ICalc   // interface 도입
        {
            int server;
            int count = 0;
        public:
            Calc() { server = ec_find_server("CalcService"); }
            ~Calc() { cout << "~Clac()" << endl; }

            void AddCount() { ++count; }
            void DelCount() { if (--count == 0) delete this; }

            int Add(int a, int b) { return ec_send_server(server, 1, a, b); }
            int Sub(int a, int b) { return ec_send_server(server, 2, a, b); }
        };

        ```
    - smart pointer로 객체 관리하기
        - 진짜 포인터는 함수가 종료될때 아무일도 하지 않는다.
        - 스마트포인터를 사용하면 함수가 종료될때 소멸자를 호출하는 원리를 이용하여 
        - 사용자가 생성과 소멸을 신경쓰지 않도록 한다.