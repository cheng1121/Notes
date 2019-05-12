# vector
类似于string类，也是一种动态数组，可以在运行阶段设置vector对象的长度，可在末尾附加新数据，还可以在中间插入新数据。基本上，它是使用new创建动态数组的替代品。功能强大，效率较低

### vector的使用
1. 添加头文件vector
2. vector包含在命名空间std中，所以可以使用using编译指令、using声明或者std::vector
3. 模版使用不同的语法来指出它存储的数据类型
4. vector类使用不同的语法来指定元素数


```
#include <vector>
using namespace std; //如果不加命名空间std的话：使用std::vector<int> vi;
int main(int argc, const char * argv[]) {
    // insert code here...

    vector<int> vi; //创建一个类型为int 长度为0的向量
    
    vector<double> vn(10); //创建一个类型是double 长度为10的向量
    return 0;
}
```

# array
1. 于数组一样长度固定
2. 存储在栈（静态内存分配），而不是自由存储区，因此效率与数组相同，但更方便，更安全


### array的使用
```
#include <array>
using namespace std;

int main(int argc, const char * argv[]) {
    
    array<int, 10> ai;  //创建一个长度为10的int数组
    array<double, 5> ad ={10.0,23.9,1.2,2.1,3.3}; //创建数组并初始化

    
    return 0;
}
```

# 数组、vector和array的对比

```
//预处理器编译指令
#include <iostream>
#include <vector>
#include <array>
using namespace std;

int main(int argc, const char * argv[]) {
    //数组
    double a1[4] ={1.1,1.2,1.3,1.4};
    //vector
    vector<int> a2 ={1,2,3,4};
  
    //array
    array<double, 5> a3 ={10.0,23.9,1.2,2.1,3.3}; //创建数组并初始化

    array<double, 5> a4;
    a4 = a3;
    
    cout << "a1 =" << a1[2] << " at 内存地址 " << &a1[2]<< endl;
    cout << "a2 =" << a2[2] << " at 内存地址 " << &a2[2]<<endl;
    cout << "a3 =" << a3[2] << " at 内存地址 " << &a3[2]<<endl;
    cout << "a4 =" << a4[2] << " at 内存地址 " << &a4[2]<<endl;
    return 0;
}
结果：
a1 =1.3 at 内存地址 0x7ffeefbff560
a2 =3 at 内存地址 0x10050ff88
a3 =1.2 at 内存地址 0x7ffeefbff500
a4 =1.2 at 内存地址 0x7ffeefbff4d0
```
1. 无论是数组、vector对象还是array对象，都可以使用标准数组表示法来访问各个元素
2. 从地址可知,array对象和数组存储在相同的内存区域(栈中),vector对象存储在另一个区域(自由存储区)
3. 可以将一个array赋给另一个array对象；而数组必须逐元素赋值数据
    