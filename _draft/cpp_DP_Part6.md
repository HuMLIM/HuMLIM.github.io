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
            Rect() {}
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