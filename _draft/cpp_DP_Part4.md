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

    // TextView를 도형 편집기에서 사용하기위해 인터페이스 변경 클래스 adapter
    class Text : public TextView, public Shape  // 다중 상속
    {
    public:
        Text(string s) : TextView(s) {}
        virtual void Draw() { TextView::Show(); }
    };

    // TextView를 도형 편집기에서 사용하기위해 인터페이스 변경 객체(Object) adapter
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
