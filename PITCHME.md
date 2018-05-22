---

# Argument Dependent Lookup

- Name lookup
- ADL
- Some pitfalls
- All examples are verified with gcc 5.4.0(online compiler)

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
    std::cout << d->X2 << std::endl; // OK!
    std::cout << Derived::X3 << std::endl; // Error!
}
```

---

##### Unqualified name lookup

```cpp
int X1 = 1;
int X3 = 3;

class B { public:
    static int X1;
    static int X2;
    static int X4;
};

class D : public B { public:
    void func(int X4)
    {
        int X2 = 2222;
        std::cout << X2 << std::endl; // local scope X2
        std::cout << X4 << std::endl; // X4 parameter
        std::cout << X1 << std::endl; // B::X1
        std::cout << X3 << std::endl; // ::X3
    }
};
```

---

##### ADL motivation

```cpp
namespace A { // Generic framework
    template<typename T> void test(T& t) {
        special_func(t);
    } }

namespace B { // Test related namespace
    class Test {};

    void special_func(Test& t) {
        std::cout<<"ADL: Special func" << std::endl;
    } }

int main() {
    B::Test t;
    A::test(t);
}
```

---

## ADL
ADL applies only to unqualified names that look like they name a nonmember function in a function call.
So ADL is NOT triggered in following cases:
- Name is qualified
- Name doesn't look like a function call
- Ordinary lookup finds name of member functin
- Ordinary lookup finds name of type(constructor)
- Name of the function to be called is enclosed in bracket(func ptr?)
- The list of function arguments contains only builtins

---

##### ADL and function overload

```cpp
namespace B {
    class Base;
    class Test; }

void special_func(B::Base& t) {
    std::cout<<"Global: special function" << std::endl;
}

namespace A { // Generic framework 
    template<typename T> void test(T& t) {
        special_func(t);
    } }

namespace B { // Particular domain
    class Base {};
    class Test : public Base {};

    void special_func(Test& t) {
        std::cout<<"ADL: special function" << std::endl;
    } }

int main()
{
    B::Base t1;
    B::Test t2;
    A::test(t1); // ::special_func(Not ADL)
    A::test(t2); // B::special_func(ADL)
}
```

---

##### ADL and refactoring

```cpp
namespace B {
    class Base {};
    class Test : public Base {};

    class Test1 {Test t;};
    void special_func(Test1& t) {
        std::cout<<"ADL: special function" << std::endl;
    } }

int main() {
    B::Base t1;
    B::Test t2;
    A::test(t1); // As before ::special_func(Not ADL)
    A::test(t2); // Now its non ADL function ::special_func
}
```

---

# More of ADL
Main purpose of ADL is to generate for Name Lookup additional scope.
- For pointer and array types, ADL considers underlying type.
- For enumeration types, ADL considers the namespace in which the enumeration is declared.
- For class members, the enclosing class is cosidered by ADL.
- For function types, ADL considers the namespaces and classes associated with all the parameter types and those associated with the return type.
- For class types (including union types) ADL consider the class type itself, the enclosing class, and any direct and indirect base classes.

---

## Additional informarion
Inspired by the book: C++ Templates: The Complete Guide

Now second addition with C++17 features
https://www.amazon.com/C-Templates-Complete-Guide-2nd/dp/0321714121

Code snippets:
- http://rextester.com/KYJIU95018
- http://rextester.com/SVK69998
- http://rextester.com/RSVCP95364
- http://rextester.com/CBVYC5858
- http://rextester.com/LHOV16189

