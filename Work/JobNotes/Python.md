python脚本语法：

1. 类

   ```python
   class NewPacket:
       def __init__(self, **kwargs):# 构造函数，self为当前对象，kwargs为构造时传来的参数(字典)
           
       def saveDebPack(self):		# 类中的成员函数，手动指明this
           
       def __del__(self):			# 析构函数
       
       
   class UI(NewPacket):	# 继承NewPacket类
       def __init__(self, **kwargs):
           self.softName2 = kwargs.get("name")	# python动态的定义成员变量，直接用self.名
           
          	softId = ""		# 定义局部变量
           if self.softName2 == "chenxinsd":
               softId = "com.vsecure.chenxinsd"
           
           kwargs.update({"name": softId}) # update用于合并字典，如果本身有key，则替换value
           
   		self.plats = copy.deepcopy(PLAT_LIBS) # 深拷贝，需要导入(import copy)模块
           
           super().__init__(**kwargs) # 调用父类的构造函数(_init_)
       
   ```

   > 在函数定义中使用 `**kwargs` 表示接收所有额外的关键字参数，这些参数会以字典的形式存储在 `kwargs` 中

2. 获取某个字典对应的值

   ```python
   self.libc = kwargs.get("libc")
   ```

3. 条件判断 `if ... else`

   ```python
   if self.mkdirBuildDir():        # 非0整数即为true
       return 1
   else:
   	return 0
   
   if copyFile(self.originQtPath + qlib, insRtlPath) and patchelf:		# and连接条件
   if not len(self.originQMakePath):									# not否定条件
   ```

4. 打印 `print`

5. 循环 `for ... in ...`

   ```python
   for module in self.modules:
   ```

6. 异常处理 `try ... finally ...`

   ```python
   try:
       file = open("example.txt", "r")
       content = file.read()
       print(content)
   finally:			
       file.close()
       print("文件已关闭")
   ```

   > 1. 不管`try`中有没有异常，都会走`finally`
   > 2. `finally` 不能单独使用，必须搭配 `try`。

7. 资源管理`with`，通常用来替代 `try...finally` 结构，在处理文件时，不需要手动调用`close()`

   ```python
   with open("example.txt", "r") as file:
       content = file.read()
       print(content)
   ```

   > 1. `open("example.txt", "r")` 返回一个文件对象。
   > 2. `as file` 将文件对象赋值给变量 `file`。
   > 3. 在 `with` 块结束后，文件会自动关闭，无需显式调用 `file.close()`。

8. 导入模块

   ```python
   from common.ui_sm import *		# common为包名(文件夹), ui_sm为模块(文件ui_sm.py)
   
   # *表示表示导入该模块中定义的所有“公共名称”（通常是变量、函数、类等）
   
   volatile.pack_args.CORP	# 同样，访问"模块"公共字典/集合/列表，通过上述方式
   ```

9. 三元运算符 `<true> if <condition> else <false>`

   ```python
   self.rtl = kwargs.get("rtl") if kwargs.get("rtl") is not None else 0
   ```

10. 列表  `[]` ： 一种有序、可变的集合，可以**存储任意类型**的元素（数字、字符串、对象等）

    ```python
    my_list = [1, 2, 3, "hello", True]
    print(my_list[0])  # 输出 1
    my_list.append(4)  # 添加元素
    print(my_list)     # 输出 [1, 2, 3, "hello", True, 4]
    print(len(my_list)) # 输出列表长度
    ```

11. 字典 `{}` : 存储键值对形式`{key: value}`（**键不能重复，值可以重复**）

    ```python
    my_dict = {"name": "Alice", "age": 25}
    print(my_dict["name"])  # 输出 "Alice"
    my_dict["city"] = "Beijing"  # 添加键值对
    print(my_dict)          # 输出 {"name": "Alice", "age": 25, "city": "Beijing"}
    ```

12. 集合(set) `{}` : 存储单个元素(**任意类型，但是不可重复**)

    ```python
    my_set = {1, 2, 3, 3}  # 重复的 3 会被去重
    print(my_set)          # 输出 {1, 2, 3}
    my_set.add(4)          # 添加元素
    print(my_set)          # 输出 {1, 2, 3, 4}
    ```

13. 动态执行字符串形式的 Python 表达式 `eval()`

    ```python
    eval(expression, globals=None, locals=None)
    
    x = 10
    result = eval("x + 5")
    print(result)  # 输出 15
    
    result = eval("NewPacket()")	# 实例化类
    eval(args.Independent + "(**tDict)") # 实例化类并且传入参数(有参构造)，tDict为字典
    
    result = eval("my_func()") 		# 调用函数
    ```

    > 调用函数和类的实例化时需要带`()`, 否则返回的只是函数或者对象本身，没有触发执行

14. 关于为什么在类实例化以及在构造函数中使用`**`来接收参数

    1. 实例化时加 `**`：因为传入的是一个字典（或类似对象），需要**解包**成关键字参数以**匹配 init 的参数名**。不加 `**`，字典会被当作单一参数，导致参数不匹配

       ```python
       class MyClass:
           def __init__(self, name, age):
           
       params = {"name": "Alice", "age": 25}
       instance = MyClass(**params)  # 解包字典为 name="Alice", age=25
       ```

    2. 在 __init__ 函数中加 `**`：在类的 __init__ 方法中定义 `**kwargs`，是为了让类能够接**受任意数量的关键字参数，并将它们打包成一个字典**

       ```python
       class MyClass:
           def __init__(self, name, **kwargs):
               self.name = name
               self.extras = kwargs  # 存储额外参数
       
       params = {"name": "Bob", "age": 30, "city": "Beijing"}
       instance = MyClass(**params)
       print(instance.name)      # 输出 Bob
       print(instance.extras)    # 输出 {'age': 30, 'city': 'Beijing'}
       ```

15. 关于继承之后，重写函数的调用问题：Python 使用**动态绑定**（dynamic dispatch），Python 的方法解析是基于实例的类型和继承链动态决定的

    比如：`UI_UOS`继承了`UI`类，那么当在`UI_UOS`中执行`super().__init__(**kwargs)`调用了父类的构造时：

    1. 父类的`self`始终都是`UI_UOS`的`self`，**实例类型是最开始的那个对象**

    2. 在父类的构造中，可以使用子类中已定义过的成员(用的是同一个指针)

    3. 如果有子类重写了成员函数，那么会根据self的实例调用其重写的函数，如果没有找上一层的

       > 寻找重写函数的方向：ui_uos --> ui --> package  

    ```python
    class UI(Package):
        def __init__(self, **kwargs):
            print("UI __init__")
            print(self.name)  			# 如果是ui_uos的实例，可以访问其成员变量，如果不是则空
            self.createAfter()			# 如果self是ui_uos的实例，那么调用的是uos重写的函数
    		
        def createAfter(self):
            print("UI createAfter")
    
    class UI_uos(UI):
        def __init__(self, **kwargs):
            print("UI_uos __init__")
            self.name = "jjj";
            super().__init__(**kwargs)
    
        def createAfter(self):
            print("UI_uos createAfter")
    
    # 测试代码
    obj = UI_uos(key="value")
    ```

