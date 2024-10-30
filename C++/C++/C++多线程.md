

## 1. 进程与线程

- **程序: **是指令和数据的有序集合，其本身没有任何运行的含义，是一个静态的概念。
- **进程:** 是程序的一次执行过程。每个独立执行的程序都是一个进程，也就是"==正在运行的程序=="。
  - 例如： MySQL、QQ、LOL 等。
- **线程:** 是进程中的实际运行的最小单位，每个运行的程序都是一个进程，在一个进程中还可以有多个执行单元同时运行，这些执行单元可以看做程序执行的一条条线索，被称为线程。
- 一个进程中至少有一个线程（主线程），同时还能开启多个子线程（多线程）
- C++11之前是没有提供线程库的，Windows上需要调用 CreateThread 创建线程，Linux下需要调用 clone 或者 pthread_create 来创建线程。
- C++11开始提供了 thread 线程类，使用该类必须包含 `<thread>` 头文件 





---

## 2. thread线程库的使用

- `main` 函数可以被称作主线程
- `thread` 类可以创建子线程
- 通过主线程来开启子线程 
  - 在主线程中实例化 `thread` 对象，并将要执行的函数传入，即在子线程中执行函数
  - `join()` :  阻塞主线程，让子线程执行完毕之后，再执行主线程中剩下的代码。 如果没有调用 `join` 函数，主程序执行完毕并没有接到子线程执行完毕的信号，就会报错。



### 2.1 线程创建

`thread`构造函数：

- `thread();`  默认构造函数，构造一个空线程对象，没有关联任何线程函数，即没有启动任何线程。 

- `thread(fn, args1,args2...);`  构造一个线程对象，并关联线程函数 fn, args1, args2...为线程函数的参数。

- `thread(thread&& other );`  移动构造函数，调用成功之后该线程对象会转移。

  > 参数为右值引用，也可以使用move转为右值引用：
  >
  >  `thread(move(thread& other));`

- `thread( const thread& ) = delete;`  拷贝构造函数(被禁用)，意味着线程无法拷贝。

其他常用函数：

- `get_id()`：获取线程id。 
- `joinable()`：线程是否还在执行，joinable代表的是一个正在执行中的线程。
- `join()`：该函数调用后会阻塞主线程，当子线程结束后，主线程继续执行。
- `detach()`：在创建线程对象后马上调用，用于把被创建线程与线程对象分离开，主线程不与子线程汇合，各自独立自行。

```c++
#include <thread>						//头文件
std::thread t(function_name, args...);	//基本语法
```

> 第一个参数是函数名(即函数指针)
>
> 第二个参数是那个函数需要的参数，没有参数就省略，如果多个参数就在后面依次加入



### 2.2 等待线程

如果不等待线程，那么创建线程之后，主函数(当前函数，不一定非是主函数)和该线程是“同时”执行的，如果主函数先结束，那么就会停止该线程

使用`t.join()`，后就会在此**阻塞主函数**，等待线程结束才能执行主函数的下面指令。

```c++
t.join();
```

> 从join方法的源码来看，join方法的本质调用的是Object中的wait方法实现线程的阻塞，wait就是pv信号量中的p，但是我们需要知道的是，调用wait方法必须要获取锁，所以join方法是被synchronized修饰的，synchronized修饰在方法层面相当于synchronized(this),this就是previousThread本身的实例。



### 2.3 分离线程

有时候我们可能不需要等待线程完成，而是希望它在后台运行。这时候我们可以使用`t.detach()`方法来分离线程

这样**即使主函数先运行结束，该线程会在后台继续运行直到结束**

```c++
t.detach();
```



### 2.4 判断是否可等待

- `joinable()` 方法判断此线程是否在执行当中，即是否可以执行`join`和`detach`方法

- `joinable()`方法返回一个布尔值，如果线程可以被`join()`或`detach()`，则返回true，否则返回false。如果我们试图对一个不可加入的线程调用`join()`或`detach()`，则会抛出一个`std::system_error`异常。

```c++
t.joinable();
```



### 2.5 线程睡眠

1. C++11之前并未提供专门的休眠函数，C语言的`sleep、usleep`函数其实是系统提供的函数，不同的系统函数的功能还要些差异。

   - 在Windows系统中，sleep的参数是毫秒：`sleep(2*1000);` // sleep for 2 seconds
   - 在类Unix系统中，sleep()函数的单位是秒：`sleep(2);` // sleep for 2 seconds

2. C++11开始，C++标准库提供了专门的线程休眠函数，使得代码可以独立于不同的平台。

   - `std::this_thread::sleep_for`，让线程休眠指定时间，比如：想要一个线程休眠 1000 ms

     ```c++
     std::this_thread::sleep_for(std::chrono::millseconds(1000));
     ```

   - `std::this_thread::sleep_untill`，会阻塞当前线程直至未来某个时间点到达；

```c++
//线程休眠函数: std::this_thread::sleep_for
//三种方式：都是睡眠1秒
std::this_thread::sleep_for(1s);		//线程睡眠1秒

std::this_thread::sleep_for(chrono::milliseconds(1000));	

std::this_thread::sleep_for(chrono::seconds(1));
```

```c++
//线程休眠函数: std::this_thread::sleep_untill
// 5秒之后执行该进程
// chrono::steady_clock::now() : 获取当前时间点的时间戳（ms级）
this_thread::sleep_until(chrono::steady_clock::now() + 5000ms);
```



### 2.6 获取当前线程id

```c++
//获取当前线程(在线程内部)的id
std::cout << "当前线程的ID" << std::this_thread::get_id() << std::endl;
//通过线程对象调用线程id
std::cout << "线程t的ID" << t.get_id() << std::endl;
```



### 2.7 传递参数

#### 1. 传递普通数据类型

```c++
#include<iostream>
#include<thread>

void foo(int x) {
	std::cout << x << std::endl;
}

int main(){
	int a=1;
	std::thread t(foo,a);
	if(t.joinable()){	//判断线程是否可等待
		t.join();		//等待线程
	}
	//t.detach();		//分离线程
	return 0;
}
```



#### 2. 传递引用

传==引用类型==的参数必须加上`std::ref()`；因为创建线程实际传入`foo`的参数是此处的参数的副本(拷贝构造)，是浅拷贝(值拷贝)，普通类型和指针无影响，引用和对象类型就有影响了

```c++
void foo(int& x) {
	x+=1;
}

int main(){
	int a=1;
    //传引用类型的参数必须加上std::ref()；实际传入foo的参数是此处的参数的副本(拷贝构造)，
    //即它是浅拷贝:普通类型和指针无影响，引用和对象类型就有影响了
	std::thread t(foo,std::ref(a));
	if(t.joinable()){	
		t.join();	
	}
	return 0;
}
```



#### 3. 传递类成员函数

1. 使用栈对象传递，注意传递参数写法，函数部分填写`&A::foo`（显式声明函数指针），参数填写对象的地址(指针)`&a`，因为类成员函数默认隐含了一个`this指针`即当前对象的指针，所以需要传递对象指针

   ```c++
   class A {
   public:
       void foo() {
           std::cout << "hello" << std::endl;
       }
   };
   int main() {
       A a;
       std::thread t(&A::foo, &a); //成员函数的入口地址，参数是对象的地址(指针)
   
       //如果这里释放了对象a，那么线程执行会有问题，就有了下面使用智能指针
       t.join();
       return 0;
   }
   ```

2. 使用智能指针来管理对象类型

   ```c++
   class A {
   public:
       void foo() {
           std::cout << "hello" << std::endl;
       }
   };
   int main() {
       std::shared_ptr<A> a = std::make_shared<A>();
       std::thread t(&A::foo, a);		//a是shared_ptr类的对象，因为重载了*和->使得就像使用指针一样
       std::thread t2(&A::foo, *a);	//对a解引用得到a管理的数据指针
       std::thread t3(&A::foo,g.get())	//返回a中管理的数据地址
      
       t.join();
       t2.join();
       t3.join();
       return 0;
   }
   ```

   > 3种方式都可以

3. 传递的是类私有成员函数时，声明了友元函数就可以在线程中使用类私有成员函数

   ```c++
   class A {
   private:
   	friend void thread_foo();	//声明了友元函数就可以在线程中使用类私有成员函数
       void foo() {
           std::cout << "hello" << std::endl;
       }
   };
   void thread_foo(){
   	std::shared_ptr<A> a = std::make_shared<A>();
       std::thread t(&A::foo, a);
       t.join();	//当前函数内等待
   }
   
   int main() {
   	thread_foo();
       return 0;
   }
   ```



#### 4. 注意局部变量释放时机

因为函数内部的线程是该函数和线程同时进行的，如果该函数传递的堆类型的参数释放过早，将会导致线程获取参数值是错误的

1. 在`join`(即线程结束后再释放参数内存)

   ```c++
   void foo(int* x) {
   	std::cout << *x << std::endl;
       //delete x    第二种方式，在调用函数内部释放
   }
   
   int main() {
   	//改变释放堆内存时机，解决传输堆变量输出问题
   	int* a = new int(1);
   	//std::thread t(foo, std::ref(a));  
   	std::thread t1(foo, a);
   
   	//delete a;			//在线程结束之前就释放内存参数将会导致错误
   	if (t1.joinable()) {
   		t1.join();
   	}
   	delete a;			//第一种方式:要在线程执行完之后释放
   }
   ```

2. 使用智能指针进行管理堆内存参数

   ```c++
   void foo(int* x) {
   	std::cout << *x << std::endl;
   }
   int main(){
   	std::shared_ptr<int> b = std::make_shared<int>(2);
   	std::thread t2(foo, b.get());
   	t2.join();
   }
   ```

   

### 2.8 多子线程

- 多子线程即多次使用 thread 实例化出多个线程，每个线程都调用一个函数即可
- 多个线程互相独立，不会互相干扰

```c++
void f4_1()
{
	for (int i = 0; i < 100; i++)
	{
		cout << "线程1: " << i << endl;
	}
}
void f4_2()
{
	for (int i = 0; i < 100; i++)
	{
		cout << "线程2: " << i << endl;
	}
}

int main()
{
	thread t1(f4_1);
	thread t2(f4_2);

	t1.join();
	t2.join();

	cout << "over" << endl;
	return 0;
}
```



### 2.9 移动构造

1. `thread( thread&& other );`  移动构造函数，调用成功之后该线程对象会转移
2. `thread`中拷贝赋值构造被删除了，注意移动构造参数是右值引用，所以可以使用`move`将左值转为右值引用

```c++
void fn5() {
	cout << "fn5" << endl;
}
int main() {
	thread t1(fn5);
	thread t2(fn5);
	// 线程转移构造
	thread t3(move(t2));

	cout << t1.get_id() << endl;
	//由于t2转移到了t3所以t2的id是0
	cout << t2.get_id() << endl;
	cout << t3.get_id() << endl;

	t1.join();
	if (t2.joinable()) t2.join();
	t3.join();
}
```



---

## 3. 同步与互斥

### 3.1 同步

**进程/线程同步**：并发性带来了异步性，有时候需要同步解决这种异步问题。有的进程/线程之间需要相互配合的完成工作，各进程/线程推进需要遵循一定的先后顺序



### 3.2 互斥

**进程/线程互斥**：对临界资源的访问，需要互斥的进行，即同一时间段内只允许一个进程/线程访问该资源

**数据共享问题**：多个线程访问同一个数据并进行修改时，会数显数据竞争问题，数据竞争可能会导致程序崩溃、产生未定义的结果，或者得到错误的结果。

**解决办法**：为了避免数据竞争问题，需要使用==互斥机制==来确保多个线程之间对共享数据的访问是安全的。常见的同步互斥机制包括互斥量、条件变量、原子操作等

**线程安全**：如果多线程程序每一次的运行结果和单线程运行的结果是一样的，那么就称该程序是线程安全的

**问题案例**:

```c++
#include <iostream>
#include <thread>
int shared_data = 0;
void func() {
    for (int i = 0; i < 100000; ++i) {
        shared_data++;
    }
}
int main() {
    std::thread t1(func);
    std::thread t2(func);
    t1.join();
    t2.join();
    std::cout << "shared_data = " << shared_data << std::endl;    //输出的结果每次都不一样
    return 0;
}
```



### 3.3 互斥量

#### 1. 概念

互斥量（`mutex`）是一种用于实现多线程同步的机制，用于确保多个线程之间对共享资源的访问互斥。互斥量通常用于保护共享数据的访问，以避免多个线程同时访问同一个变量或者数据结构而导致的数据竞争问题。

#### 2. 使用

1. 导入库`include<mutex>`，使用`mutex`定义信号量
2. 对信号量进行操作

互斥量提供了两个基本操作：`lock()` 和 `unlock()`。当一个线程调用 `lock()` 函数时，如果互斥量当前没有被其他线程占用，则该线程获得该互斥量的所有权，可以对共享资源进行访问。如果互斥量当前已经被其他线程占用，则调用 `lock()` 函数的线程会被阻塞，直到该互斥量被释放为止。

> 就是计操里的信号量PV操作，mutex.lock()就是P(mutex)，即上锁(信号量减1)，mutex.unlock()就是V(mutex)，即解锁(信号量+1)，以此来实现同步

案例：

```c++
#include <iostream>
#include <thread>
#include <mutex>		//导入互斥量库
int shared_data = 0;
std::mutex mtx;			//定义互斥量
void func(int n) {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();		//P操作，上锁
        shared_data++;        
        std::cout << "Thread " << n 
        << " increment shared_data to " << shared_data << std::endl;
        mtx.unlock();	//V操作，解锁
    }
}
int main() {
    std::thread t1(func, 1);
    std::thread t2(func, 2);

    t1.join();
    t2.join();    
    std::cout << "Final shared_data = " << shared_data << std::endl;    
    return 0;
}
```

#### 3. 联想信号量

**信号量实现同步**：

![在这里插入图片描述](file://F:\SoftWares\StudyWindow\XingMa\408\source\images\操作系统\20200318214033750.png?lastModify=1720364170)

**信号量实现互斥**：

![在这里插入图片描述](file://F:/SoftWares/StudyWindow/XingMa/408/source/images/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/20200318213342662.png?lastModify=1720364211)





---

## 4. 死锁

### 4.1 死锁概念

**死锁定义**：各进程/线程互相等待对方手里的资源，导致各进程被阻塞，无法进行推进

**死锁原因**：请求和释放资源的顺序不当，就会产生死锁



### 4.2 互斥量死锁

**问题**：

假设有两个线程 T1 和 T2，它们需要对两个互斥量 mtx1 和 mtx2 进行访问，而且需要按照以下顺序获取互斥量的所有权：

\- T1 先获取 mtx1 的所有权，再获取 mtx2 的所有权。

\- T2 先获取 mtx2 的所有权，再获取 mtx1 的所有权。

如果两个线程同时执行，就会出现死锁问题。



**解决**：可以让两个线程按照相同的顺序获取互斥量的所有权

```c++
#include <iostream>
#include <thread>
#include <mutex>
std::mutex mtx1, mtx2;
void func1() {
	 for (int i = 0; i < 100000; ++i) {
        mtx1.lock();  
        mtx2.lock(); 
        mtx1.unlock();
        mtx2.unlock();
    }
}
void func2() {
	 for (int i = 0; i < 100000; ++i) {
        //如果上锁的顺序不当就会产生死锁
        mtx1.lock();  
        mtx2.lock(); 
        mtx1.unlock();
        mtx2.unlock();
    }
}
int main() {
	std::thread t1(func1);
    std::thread t2(func2);
    t1.join();
    t2.join();
    return 0;
}

```



---

## 5. lock_guard和unique_lock

这两种和**智能指针的实现原理**类似，使用封装的模板类来进行管理，可以不用自己定义加锁和解锁，对象析构时自动解锁

### 5.1 lock_guard

#### 1. 定义

`std::lock_guard` 是 C++ 标准库中的一种互斥量封装类，用于保护共享数据，防止多个线程同时访问同一资源而导致的数据竞争问题。

#### 2. 特点

1. 当构造函数被调用时，该互斥量会被自动锁定。
2. 当析构函数被调用时，该互斥量会被自动解锁。
3. `std::lock_guard` 对象不能复制或移动，因为它没有拷贝构造函数和赋值函数(delete了同unique_ptr)，因此它只能在局部作用域中使用。



#### 3. 案例

```c++
#include<iostream>
#include<thread>
#include<mutex>

int data = 0;
std::mutex mtx;
void func() {
	for (int i = 0; i < 1000; i++) {
		//构建lock_guard,构造时枷锁，析构时解锁
		std::lock_guard<std::mutex> lg(mtx);
		data++;
	}
}

int main() {
	std::thread t1(func);
	std::thread t2(func);
	t1.join();
	t2.join();

	std::cout << data << std::endl;
    //结果正确
}
```



### 5.2 unqiue_lock

#### 1. 定义

`std::unique_lock` 是 C++ 标准库中提供的一个互斥量封装类，用于在多线程程序中对互斥量进行加锁和解锁操作。

#### 2. 特点

它的主要特点是可以对互斥量进行更加(**比lock_guard功能更多)**灵活的管理，包括延迟加锁、条件变量、超时等。

#### 3. 构成

##### 构造函数

1. `unique_lock() noexcept = default`：默认构造函数，创建一个未关联任何互斥量的 `std::unique_lock` 对象。
2. `explicit unique_lock(mutex_type& m)`：构造函数，使用给定的互斥量 `m` 进行初始化，并对该互斥量进行加锁操作，析构时自动解锁同`lock_guard`。
3. `unique_lock(mutex_type& m, defer_lock_t) noexcept`：构造函数，使用给定的互斥量 `m` 进行初始化，但不对该互斥量进行加锁操作。
4. `unique_lock(mutex_type& m, try_to_lock_t) noexcept`：构造函数，使用给定的互斥量 `m` 进行初始化，并尝试对该互斥量进行加锁操作。如果加锁失败，则创建的 `std::unique_lock` 对象不与任何互斥量关联。
5. `unique_lock(mutex_type& m, adopt_lock_t) noexcept`：构造函数，使用给定的互斥量 `m` 进行初始化，并假设该互斥量已经被当前线程成功加锁。

##### 成员函数

1. `lock()`：尝试对互斥量进行加锁操作，如果当前互斥量已经被其他线程持有，则当前线程会被阻塞，直到互斥量被成功加锁。

2. `try_lock()`：尝试对互斥量进行加锁操作，如果当前互斥量已经被其他线程持有，则函数立即返回 `false`，否则返回 `true`。

3. `try_lock_for(const std::chrono::duration<Rep, Period>& rel_time)`：尝试对互斥量进行加锁操作，如果当前互斥量已经被其他线程持有，则当前线程会被阻塞，直到互斥量被成功加锁，或者超过了指定的时间。

4. `try_lock_until(const std::chrono::time_point<Clock, Duration>& abs_time)`：尝试对互斥量进行加锁操作，如果当前互斥量已经被其他线程持有，则当前线程会被阻塞，直到互斥量被成功加锁，或者超过了指定的时间点。

   > try_lock_until的时间是某个具体的时间点，而try_lock_for是当前时间点之后的时间段内。具体功能一样
   >
   > 而且这两个返回值和try_lock的返回值一样都是bool类型

5. `unlock()`：对互斥量进行解锁操作。



#### 4. 案例

```c++
#include<iostream>
#include<thread>
#include<mutex>

int data = 0;
std::timed_mutex mtx;  //只有时间锁支持对时间进行操作，普通的锁不支持的
void func() {
	for (int i = 0; i < 2; i++) {
		//参数defer_lock表示构造时不加锁
		std::unique_lock<std::timed_mutex> lg(mtx, std::defer_lock);
		//延迟枷锁，如果两秒之内不能加锁那么会直接撤销
		if (lg.try_lock_for(std::chrono::seconds(2))) {
			//线程延迟一秒
			std::this_thread::sleep_for(std::chrono::seconds(1));
			data++;
		}
	}
}

int main() {
	std::thread t1(func);
	std::thread t2(func);
	t1.join();
	t2.join();
	std::cout << data << std::endl;
	//结果是3而不是4，因为第一个线程，他会占用mtx两秒(两次循环各延迟一秒)，
    //所以第二个线程的第一个循环会放弃执行
}
```



### 5.3 SAC

- `Compare-And-Swap (CAS)` 是用于多线程以实现同步的原子指令。它将存储位置的内容与给定值进行比较，当它们逐位相等，才将该存储位置的内容修改为新的给定值。**整个流程为一个原子操作**。

- `mutex`互斥锁比较重，对于系统消耗有些大，C++11的`thread`类库提供了针对简单类型的原子操作类，如`std::atomic_int，atomic_long，atomic_bool`等，这些值的增减都是基于`CAS`操作的，既保证了线程安全，效率还非常高

```c++
#include <atomic>

atomic_int num2 = 0;
void fn9(){
	for (int i = 0; i < 10000000; i++){
		num2++;
	}
}

int main(){
	thread t1(fn9);
	thread t2(fn9);

	if (t1.joinable()) t1.join();
	if (t2.joinable()) t2.join();
	cout << num2 << " end" << endl;
	return 0;
}
```

> 使用这些原子操作类可以不用考虑互斥问题