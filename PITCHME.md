---
theme: "white"
#customTheme: "style2"
---

# Argument Dependent Lookup

- Name lookup
- ADL

*All examples are verified with gcc 5.4.0(online compiler)*

---

## Name Lookup

- "Qualified name" lookaup
- "Unqualified name" lookup
- ADL motivation

---

##### Qualified name lookup

```cpp
class Base { public:
    static int X1;
    int X2 = 2;
};
class Derived : public Base {};

int Base::X1 = 1;
int X3 = 3;

void func(Derived* d)
{
    std::cout << Derived::X1 << std::endl; // OK!
    std::cout << d->X2 << std::endl;       // OK!
    std::cout << Derived::X3 << std::endl; // Error!
}
```

---

##### Unqualified name lookup

```cpp
int X3 = 3;
int X4 = 4;

class Base { public:
    static int X1;
    static int X2;
    static int X3;
};

class Derived : public Base { public:
    void func(int X2)
    {
        int X1 = 1;
        std::cout << X1 << std::endl; // local scope X1
        std::cout << X2 << std::endl; // function scope
        std::cout << X3 << std::endl; // class scope (B::X3)
        std::cout << X4 << std::endl; // ::X4
    }
};
```

---

##### ADL motivation

```cpp
namespace Utils {
    template<typename T> void DoSomething(T& t) {
        SpecialFunc(t); // T has to provide SpecialFunc
    } }

namespace Domain {
    class ConcreteClass {};

    void SpecialFunc(ConcreteClass& t) {
        std::cout<<"ADL: Special func" << std::endl;
    } }

int main() {
    Domain::ConcreteClass t;
    Utils::DoSomething(t);
}
```

---

## ADL
ADL generates additional scope for Name Lookup and then regular overloading rules are applied.
- For pointer and array types, ADL considers underlying type.
- For enumeration types, enclosing namespace is considered.
- For class members, the enclosing class is cosidered by ADL.

---

##### More of ADL scope
- For function types, ADL considers:
  - namespaces and classes associated with all the parameter types
  - and those associated with the return type
- For class types (including union types) ADL consider:
  - the class type itself
  - the enclosing class
  - and any direct and indirect base classes

---

##### When ADL is trigered
ADL applies only to unqualified names that looks like they name a nonmember function in a function call.

So ADL is **NOT** triggered in following cases:
- Name is qualified
- Name doesn't look like a function call
- Ordinary lookup finds name of member functin
- Ordinary lookup finds name of type(constructor)
- The list of function arguments contains only builtins

---

##### ADL example1

```cpp
class Base {};
namespace Domain1 {
    class ConcreteClass : public Base {};
    void SpecialFunc(ConcreteClass*, int) {
        std::cout<<"Domain1" << std::endl;
    } }

namespace Domain2 {
    enum ConcreteEnum{TestEnum = 0};
    void SpecialFunc(Base*, ConcreteEnum) {
        std::cout<<"Domain2" << std::endl;
    } }

namespace Utils {
    template<typename T, typename E> void DoSomething(T* t, E e) {
        SpecialFunc(t,e); // T or E have to provide SpecialFunc
    } }

int main() {
    Domain1::ConcreteClass t;
    Utils::DoSomething(&t, Domain2::TestEnum);
}
```

---
##### ADL example1(result)
\> Domain1
---

##### ADL example2

```cpp
class Base{};
class FancyName { public: FancyName(Base&, int) {
    std::cout<<"FancyName::FancyName" << std::endl;}};

namespace Domain {
    class ConcreteClass : public Base {};
    int FancyName(ConcreteClass*, int) { std::cout<<"Domain::FancyName" << std::endl; }
    void SomethingImpl(ConcreteClass*) {std::cout<<"Domain::SomethingImpl" << std::endl;}}

namespace Utils {
    class Helper { public:
      template<typename T> void DoSomething(T& t) {
        auto v = FancyName(t, 5);  
        SomethingImpl(t);
      }
      void SomethingImpl(Base& t) {std::cout<<"Utils::Helper::SomethingImpl" << std::endl;}
    }; }

int main() {
    Domain::ConcreteClass t;
    Utils::Helper h;
    h.DoSomething(t);
}
```

---
##### ADL example2(result)
- \> FancyName::FancyName

- \> Utils::Helper::SomethingImpl

---

##### ADL example3

```cpp
void SpecialFunc(double) { std::cout<<"Global::Double" << std::endl; }
void SpecialFunc(int) { std::cout<<"Global::Int" << std::endl; }

namespace Domain {
    class MyDouble {public: MyDouble(double){}};
    class MyInt {public: MyInt(int){}};
    typedef int Int;
    typedef MyDouble Double;

    void SpecialFunc(Double t) { std::cout<<"Domain::Double" << std::endl; }
    void SpecialFunc(Int t) { std::cout<<"Domain::Int" << std::endl; } }

namespace Utils {
    template<typename T> void DoSomething(T t) {
        SpecialFunc(t);
    } }

int main() {
    Utils::DoSomething<Domain::Double>(5.0);    
    Utils::DoSomething<Domain::Int>(5);
}
```

---
##### ADL example3(result)
\> Domain::Double

\> Global::Int

---

## Additional informarion
The book: C++ Templates: The Complete Guide
https://www.amazon.com/C-Templates-Complete-Guide-2nd/dp/0321714121

Code snippets:
- http://rextester.com/KYJIU95018
- http://rextester.com/SVK69998
- http://rextester.com/RSVCP95364
- http://rextester.com/ZGPTC88744
- http://rextester.com/EMQGY80139
- http://rextester.com/EEHGY18233

