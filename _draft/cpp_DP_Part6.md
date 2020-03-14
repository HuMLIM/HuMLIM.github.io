---
permalink: 
layout:
---
## Design Pattern part6

- 객체 생성 방법
    - 사용자가 직접 객체 생성
        - 객체 생성에 제약이 없으며 자유롭다.
        ```cpp
        class Shape
        {
        public:
            virtual ~Shape() {}
        };
        class Rect : public Shape
        {
        public:            
        };

        int main()
        {
            Rect r;              // stack
            Shape* p = new Rect; // heap
        }
        ```

    - static 멤버 함수를 사용한 객체 생성
        - 객체 생성을 한곳에서 수행하여 제약 조건을 만들 수 있다.
            > 생성 개수 제한, 자원 공유
        - 객체 생성 함수를 다른 함수의 인자로 전달 할 수 있다.
        ```cpp
        class Shape
        {
        public:
            virtual ~Shape() {}
        };
        class Rect : public Shape
        {
            Rect() {}   // 생성자를 private로 만들어 자유로운 객체 생성을 막는다.
        public:
        // static 함수로 만들어야 객체의 생성이 한곳에서 이루어지며 함수의 인자로 사용이 가능하다.
            static Shape* Create()
            {
                /* 이곳에 제약조건 추가 */ 
                return new Rect; 
            }
            void Draw() {}
        };

        void CreateAndDraw(Shape* (*f)())
        {
            Shape* p = f();
            p->Draw();
        }
        int main()
        {
            Shape* p = Rect::Create();
            CreateAndDraw(&Rect::Create());
        }
        ```
    - 객체 생성을 위한 전용 클래스
        - 객체 생성을 한곳에 집중 시키면서
        - 객체 본래의 기능과 객체 생성을 위한 코드를 분리한다.
        - 추상 팩토리(Abstract Factory) 패턴
        ```cpp
        class Shape
        {
        public:
            virtual ~Shape() {}
        };
        class Rect : public Shape
        {
            Rect() {}
        public:
            void Draw() {}
            friend class ShapeFactory
        };

        class ShapeFactory
        {
        public:
            Shape* CreateShape(int type)
            {
                Shape* p = 0;
                swich (type)
                {
                case 1: p = new Rect; break;
                //...
                }
                return p;
            }
        };

        int main()
        {
            ShapeFactory factory;
            Shape* p = factory.CreateShape(1);
        }
        ```
    
    - 기존에 존재하던 객체를 복사해서 새로운 객체 생성
        - prototype pattern
        - clone() 가상 함수
        ```cpp
        class Shape
        {
        public:
            virtual ~Shape() {}
            virtual Shape* clone() = 0;
        };
        class Rect : public Shape
        {
            Rect() {}
        public:
            void Draw() {}
            virtual Shape* clone() { return new Rect(*this); }
        };

        int main()
        {
            Shape* p = new Rect;
            Shape* p2 = p->clone();
        }
        ```

- Singleton pattern
    - 클래스의 인스턴스는 오직 하나임을 보장
    - 이에 대한 접근은 어디에서든지 하나로만 통일

    - 방법 1: static 멤버 data
    ```cpp
        class Cursor
    {
        int x, y;

    private:
        Cursor() {} // 생성자는 private로
        Cursor(const Cursor&) = delete; // 복사 생성자 삭제
        Cursor operator=(const Cursor&) = delete; // 복사 대입 생성자 삭제

        static Cursor instance;
    public:
        static Cursor getInstance()
        {
            return instance;
        }
    };

    Cursor Cursor::instance; // static 멤버변수의 외부 초기화로 인해 사용하지 않아도 항상 생성

    int main() { Cursor& c1 = Cursor::getInstance(); }
    ```
    - 방법2: static 지역 변수(Mayer's 싱글톤)
    ```cpp
    class Cursor
    {
        int x, y;

    private:
        Cursor() {} // 생성자는 private로
        Cursor(const Cursor&) = delete; // 복사 생성자 삭제
        Cursor operator=(const Cursor&) = delete; // 복사 대입 생성자 삭제
    public:
        static Cursor getInstance()
        {
            static Cursor instance; // static지역 변수 사용으로 사용하지 않으면 생성하지 않는다.(lazy initialization)
            return instance;
        }
    };
    int main() { Cursor& c1 = Cursor::getInstance(); }
    ```
    - 방법3: new로 생성
    ```cpp 
    class Cursor
    {
        int x, y;

    private:
        Cursor() {} // 생성자는 private로
        Cursor(const Cursor&) = delete; // 복사 생성자 삭제
        Cursor operator=(const Cursor&) = delete; // 복사 대입 생성자 삭제

        static Cursor* pInstance;
    public:
        static Cursor getInstance()
        {
            if (pInstance == 0)
                pInstance = new Cursor; // 멀티 스레드 프로그램에서 동기화 문제가 발생 된다.
            return *pInstance;
        }
    };

    Cursor* Cursor::pInstance = 0;

    int main() { Cursor& c1 = Cursor::getInstance(); }
    ```
    - DCLP(Double Check Locking Pattern)
        - new를 이용한 singleton pattern에서 동기화 문제는 lock을 사용하여 방지한다.
        - getInstance가 호출될 때 마다 lock이되는 것을 방지 하기위해 if문을 2번 사용하는 DCLP 기법을 사용한다.
        ```cpp
        static mutex m;
        static Cursor getInstance()
        {
            if(pInstance = 0)
            {
                m.lock();
                if (pInstance == 0)
                    pInstance = new Cursor; // 멀티 스레드 프로그램에서 동기화 문제가 발생 된다.
                    // Cursor의 생성 순서
                    //    1. temp = sizeof(Cursor)
                    //    2. Cursor::Cursor() 호출
                    //    3. pInstance = temp;
                    // 컴파일러의 최적화에 의해 pInstance = sizeof(Cursor)로 수행되면 
                    // 생성자가 호출 되기전에 다른 스레드에서 pInstance를 사용 할 가능성이 있다.
                    // atomic을 사용하여 문제를 해결한다.
                m.unlock();
            }

            return *pInstance;
        }
        ```

    - Singleton 코드의 재사용
        - 방법 1: 매크로를 사용
        ```cpp
        #define MAKE_SINGLETON(classname)                   \
        private:                                            \
            classname() { }                                 \
            classname(const classname&) = delete;           \
            void operator=(const classname&) = delete;      \
        public:                                             \
            static classname& getInstance()                 \
            {                                               \
                static classname instance;                  \
                return instance;                            \
            }

        class Cursor
        {
            int x, y;

            MAKE_SINGLETON(Cursor)
        };
        ```
        - 방법 2: 상속을 이용 : CRTP(Curiously Recurring Template Pattern)
        ```cpp
        template<typename TYPE> class Singleton
        {
        protected:
            Singleton() { }
            Singleton(const Singleton&) = delete;
            void operator=(const Singleton&) = delete;
        public:
            static TYPE& getInstance()
            {
                static TYPE instance;
                return instance;
            }
        };

        class Mouse : public Singleton<Mouse>
        {

        };
        ```
- Factory 
    - 일반적인 factory
    ```cpp
    class ShapeFactory
    {
        MAKE_SINGLETON(ShapeFactory)
    public:
        Shape* CreateShape(int type )
        {
            Shape* p = 0;
            if      ( type == 1 ) p = new Rect;
            else if ( type == 2 ) p = new Circle;

            return p;
        }
    };

    int main() 
    {
        ShapeFactory& factory = ShapeFactory::getInstance();
        Shape* p = factory.CreateShape(1);
    }
    ```
    - 제품 등록형 factory

    - 제품 자동 등록형 factory
    
    - 클래스가 아닌 객체를 등록하는 factory
        - 자주 사용하는 견본품을 등록해서 복사 생성 한다.
        - prototype 패턴이라고 한다.