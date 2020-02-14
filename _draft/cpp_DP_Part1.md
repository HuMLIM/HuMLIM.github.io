---
permalink: 
layout:
---
### Design Pattern
- Design Pattern 이란? 자주 사용하는 코딩 패턴에 이름을 부여 한 것

- Protected constructor
    ```cpp
    class Animal
    {
    protected:
        Animal() {}
    };

    class Dog : public Animal
    {
    public:
        Dog() {}    //Dog() : Animal() {}
    };

    int main()
    {
        Animal a;   // error.
        Dog    d;   // ok
    }
    ```
    > Dog의 생성자가 먼저 호출 되고 생성자의 안에서 기반 클래스인 Animal의 생성자를 호출한다.

    > 생성자를 protected에 만드는 이유
    > - 자기자신은 객체로 만들 수 없으나 파생 클래스의 객체는 만들수 있도록 하기 위해
    > - 즉, 추상적인 개념은 객체로 존재 할 수 없으나 현실세계에서 객체가 존재 하는 경우

- protected destructor
    ```cpp
    class Car
    {
    public:
        Car() {}
        void Destory() { delete this; }
    protected:
        ~Car() { std::cout << "~Car" << std::endl; }
    };

    int main()
    {
        //Car c;          // 스택에 객체를 만들 수 없다.
        Car* p = new Car; // 힙에는 객체를 만들 수 있다.
        //delete p;       // protected에 접근 불가하여 error
        p->Destroy();     // 멤버함수를 이용하여 객체 파괴
    }
    ```
    > 객체를 스택에 만들지 못하게 하는 기법  
    > 참조계수 기반의 객체 수명 관리 기법에서 주로 사용

- Upcasting
    ```cpp
    class Animal
    {
        int age;
    public:
        virtual void Cry() { std::cout << "Animal Cry" << std::endl; }
    };

    class Dog : public Animal
    {
        int color;
    public:
        //override
        virtual void Cry() override { std::cout << "Dog Cry" << std::endl; }    //override생략 가능하지만 오타 방지를 위해 붙이는 것이 좋은 습관!
    };

    int main()
    {
        Dog d;

        Dog*    p1 = &d;    // ok
        double* p2 = &d;    // error
        Animal* p3 = &d;    // ok

        p3->Cry();      // Dog Cry 출력, 
                        // vertual이 아닐 경우 Animal Cry 출력 
    }
    ```
    > 기반 클래스의 타입의 포인터(참조)로 파생 클래스 객체를 가리킬 수 있다.

    > 파생클래스가 재정의(override) 한 함수가 호출되게 하려면 반드시 가상함수(virtual)로 만들어야 한다.

    - Upcasting 사용 예
    > ![upcasting 사용 예](../_img/composite_pattern.png)

- abstract class
    ```cpp
    class Shape //추상 클래스
    {
    public:
        virtual void Draw() = 0; // 순수 가상함수
    };

    class Rect : public Shape
    {
    public:
        virtual void Draw() {} // 구현부를 제공하면 추상 아님
    };

    int main()
    {
        Shape s; //error 추상클래스는 일반 객체 생성이 되지 않는다.
        Shape* p; // ok

        Rect r; // Draw() 구현 함수가 없으면 error
    }
    ```
    > 순수 가상 함수(pure virtual function)
    > - 함수 선언 뒤에 "=0"을 표기한 가상함수
    > - 함수의 구현부가 없다.

    > 추상 클래스(abstrcat class)
    > - 순수 가상 함수를 한 개 이상 가지고 있는 클래스

    > 추상 클래스 특징
    > - 객체를 생성할 수 없다.
    > - 포인터 변수는 만들 수 있다.
    > - 순수 가상함수의 구현부를 제공하지 않은 경우 파생 클래스도 추상 클래스 이다.
    > - 파생 클래스에게 특정 함수를 반드시 구현하라고 지시하기위해 사용된다.

- interface 와 coupling  
    새로운 것이 추가 되었을때 기존의 코드가 수정되는 것은 좋지 않은 Design이다.  
    - OCP(Open Close Principle)
        > 기능확장에는 열려있고(Open), 코드 수정에는 닫혀 있어야(Close), 한다는 이론(Principle)
    ```cpp
    class Camera
    {
    public:
        void take() { cout << "take picture" << endl; }
    };

    class HDCamera
    {
    public:
        void take() { cout << "take HD picture" << endl; }
    };

    class People
    {
    public:
        void useCamera(Camera* p) { p->take(); }
        void useHDCamera(HDCamera* p) { p->take(); }
    };

    int main()
    {
        People p;
        Camera c;
        p.useCamera(&c);

        HDCamera hc;
        p.useCamera(&hc);
    }
    ```
    > Camera class와 People class는 강한 결합관계
        > - 객체가 서로 상호 작용할 때 서로에 대해서 잘 알고 있는 것(?)
        > - 교체/확장이 불가능한 경직된 디자인
  
    ```cpp
    //규칙 : 모든 카메라는 아래 인터페이스로 구현해야 한다.

    /* 
    class ICamera
    {
    public:
        virtual void take() = 0;
        virtual ~ICamera() {}    // 누군가의 부모역할을 해야하므로 가상 소멸자를 써야한다??
    };
    */
    // C++에는 interface keyword가 없기 떄문에 아래와 같이 struct로도 많이 사용한다.(public keyword생략 가능)
    struct ICamera
    {
        virtual void take() = 0;
        virtual ~ICamera() {}
    };

    class Camera : public ICamera
    {
    public:
        void take() { cout << "take picture" << endl; }
    };

    class People
    {
    public:
        void useCamera(ICamera* p) { p->take(); }

    };
    ```
    > 약한 결합 관계
        > - 객체가 서로 상호 작용하지만, 서로에 대해 잘 알지 못하는 것.
        > - 교체/확장이 가능한 유연한 디자인
        > - 객체는 상호간에 인터페이스를 통해서 통신 해야 함
        > - Client는 구현에 의존하지 말고 인터페이스에 의존

- Design pattern example
    ```cpp
    class Shape
    {
    public:
        virtual void Draw() { cout << "Draw Shape" << endl; } // 1. 파생 클래스의 공통의 특징은 반드시 기반 클래스에도 있어야 한다.
                                                              // 2. 파생 클래스에서 재정의 되는 함수는 반드시 가상 함수로 만들어야 한다.
        // 자신의 복사본을 만드는 가상함수 : Prototype design pattern
        virtual Shape* Clone() { return new Shape(*this); }
    };

    class Rect : public Shape
    {
    public:
        virtual void Draw() { cout << "Draw Rect" << endl; } // 파생 클래스에서는 virtual keyword는 필요없으나
                                                             // 나중에 헷갈릴 수 있으니 적어주자
        virtual Shape* Clone() { return new Rect(*this); }                                                     
    };

    class Circle : public Shape
    {
    public:
        virtual void Draw() { cout << "Draw Circle" << endl; }
        virtual Shape* Clone() { return new Circle(*this); } 
    };

    int main()
    {
        vector<Shape*> v; // 공통의 기반 클래스가 있다면 하나의 컨테이너에서 관리가 가능하다.

        while( 1 )
        {
            int cmd;
            cin >> cmd;

            if ( cmd == 1 ) v.push_back( new Rect );
            else if ( cmd == 2 ) v.push_back( new Circle );
            else if ( cmd == 8 )
            {
                cout << "index >>";
                int k;
                cin >> k;
                // k 번째 도형의 복사본을 v에 추가한다.
                v.push_back( v[k]->Clone() );
            }
            else if ( cmd == 9 )
            {
                for (auto p : v)
                    p->Draw();  // 다형성(polymorphism) : 동일한 함수(메서드)호출이 상황(객체)에 따라 다르게 동작하는 것
            }
        }
    }
    ```
- template method
    - 공통적으로 변하지 않는 전체적인 흐름을 기반 클래스에서 제공하고(public, final)
    - 변해야 하는 부분을 가상함수(private 또는 protected)로 제공해서 파생 클래스가 재정의 할 수 있도록 한다.
    ```cpp
    /* Draw 할때 lock을 걸어주는 동작을 만들고 싶다면? */
    class Shape
    {
    protected:
        //virtual void DrawImp() { cout << "Draw Shape" << endl; }
        virtual void DrawImp() = 0; //파생 클래스에서 무조건 재정의 해야한다면 순수 가상함수로 만드는 것이 더 좋다.
    public:
        virtual void Draw() final {         // 파생 클래스에서 재정의 하지 못하도록 final keyword를 사용한다.
            cout << "mutex lock" << endl;   // *여기서는 cout으로 표현만 한다.
            //cout << "Shape Rect" << endl;   // 변해야 하는 부분
            DrawImp();
            cout << "mutex unlock" << endl; 
        }
        //virtual Shape* Clone() { return new Shape(*this); }
        virtual Shape* Clone() = 0; //파생 클래스에서 무조건 재정의 해야한다면 순수 가상함수로 만드는 것이 더 좋다.
    };

    class Rect : public Shape
    {
    public:
        virtual void DrawImp() { cout << "Draw Rect" << endl; } // 변해야 하는 부분은 파생 클래스에서 재정의 한다.
        virtual Shape* Clone() { return new Rect(*this); }                                                     
    };
    ```