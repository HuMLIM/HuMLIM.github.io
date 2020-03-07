---
permalink: 
layout:
---
## Design Pattern part5

- Observer pattern
    - 관찰자(Observer)는 루프를 돌며 변화를 관찰 하거나
    - 관찰자의 대상(Subject)이 변화를 통보하는 방식
    - 객체 사이의 1:N의 종속성을 정의하고 
    - 한 객체의 상태가 변하면 다른 객체에 통보하며 자동으로 수정이 일어나게 한다.
    ![observer_pattern](../_img/observer_pattern.png)
    ![observer_diagram](../_img/observer_diagram.png)
    ```cpp
    #include <iostream>
    #include <vector>
    using namespace std;

    class Subject; // IGraph에서 사용하기 위한 전방 선언

    struct IGraph
    {
    public:
        virtual void update(Subject*) = 0;
        virtual ~IGraph() {}
    };

    struct Subject
    {
        vector<IGraph*> v;
    public:
        void attach(IGraph* p) { v.push_back(p); }
        void detach(IGraph* p) { /*TODO*/ }

        void notify()
        {
            for (auto p : v)
            {
                p->update(this);
            }
        }
    };

    class Table : public Subject
    {
        int data;
    public:
        int getData() { return data; }
        void setData(int d)
        {
            data = d;
            notify();
        }
    };

    class PieGraph : public IGraph
    {
    public:
        virtual void update(Subject* p)
        {
            // table에 접근하여 data를 가져온다.
            // p는 table의 인터페이스이므로 캐스팅 필요
            int n = static_cast<Table*>(p)->getData();
            Draw(n);
        }

        void Draw(int n)
        {
            cout << "Pie Graph: ";
            for (int i = 0; i < n; i++)
            {
                cout << "*";
            }
            cout << endl;
        }
    };

    class BarGraph : public IGraph
    {
    public:
        virtual void update(Subject* p)
        {
            int n = static_cast<Table*>(p)->getData();
            Draw(n);
        }

        void Draw(int n)
        {
            cout << "Bar Graph: ";
            for (int i = 0; i < n; i++)
            {
                cout << "*";
            }
            cout << endl;
        }
    };

    int main()
    {
        BarGraph bg;
        PieGraph pg;
        Table t;
        t.attach(&bg);
        t.attach(&pg);

        while (1)
        {
            int n;
            cin >> n;
            t.setData(n);
        }
    }
    ```

- Container 설계기술
    - Object(기반클래스) 포인터를 저장하는 컨테이너
        - 장점
            1. 코드 메모리가 증가하지 않는다.
        - 단점
            1. 타입 안정성이 떨어진다.
                > s.push_front(new Point);  
                > s.push_front(new Rect);  
                > // 타입이 달라도 같은 객체안에 push가 가능하다.
            2. 컨테이너에서 요소를 꺼낼 때 반드시 캐스팅이 필요하다.
                > Point* p = static_cast<Point*>(s.front());
            3. int, double 등의 primitive type은 저장할 수 없다.
            별도의 Integer등의 타입이 필요하다.
                > s.push_back(10) // error  
                > s.push_back(new Integer(10)); // ok

    - template 기반 컨테이너
        - 장점
            1. 타입 안정성이 뛰어나고
                > 다른 타입사용시 build error
            2. 컨테이너에서 요소를 꺼낼 때 캐스팅이 필요 없다.
            3. int, double등의 primitive type을 저장할 수 있다.
        - 단점
            1. 타입에 따라 코드가 생성 되므로 코드 메모리가 증가 한다.

    - thin template 기반 컨테이너
        - 템플릿에 의한 코드 중복을 줄이기 위한 기술
        - void* 등으로 내부 자료구조를 구성하고,
        - 캐스팅을 위한 템플릿을 제공한다.
        ```cpp
        template<typemane T> class slist : public slistImp
        {
        public:
            inline void push_front(T n) { slistTmp::push_front((void*)n); }
            inline T front() { return static_cast<T>(slistImp::front()); }
        }
        ```
        - [C++ idioms](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms)

