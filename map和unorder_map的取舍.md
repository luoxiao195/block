在开发过程中，键值对型容器使用频率可以说是比较多的，当前C++中有两种该类型容器，map与unordered_map。这两种容器在不同场景下的作用是不同的，应用得当对优化性能有不小的帮助。

map是基于红黑树实现。红黑树作为一种自平衡二叉树，保障了良好的最坏情况运行时间，即它可以做到在O(log n)时间内完成查找，插入和删除，在对单次时间敏感的场景下比较建议使用map做为容器。比如实时应用，可以保证最坏情况的运行时间也在预期之内。

另红黑树是一种二叉查找树，二叉查找树一个重要的性质是有序，且中序遍历时取出的元素是有序的。对于一些需要用到有序性的应用场景，应使用map。

unordered_map是基于hash_table实现，一般是由一个大vector，vector元素节点可挂接链表来解决冲突来实现。hash_table最大的优点，就是把数据的存储和查找消耗的时间大大降低，几乎可以看成是常数时间；而代价仅仅是消耗比较多的内存。然而在当前可利用内存越来越多的情况下，用空间换时间的做法是值得的。

值得注意的是，在使用unordered_map设置合适的hash方法，可以获得良好的性能。

至于实际性能对比，这里不妨写测试来看看，到底两者之间性能如何。

```
编译release版本：g++ -o3 -o main main.cpp
随机操作和顺序操作比例：1:1
```

10w量级的耗时，可以看出，map在增删查三项上均弱于unordered_map，内存使用map略少，但不明显：

```
map insert time: 71.064000
map find time: 30.305000
map erase time: 45.373000
unordered_map insert time: 42.139000
unordered_map find time: 11.316000
unordered_map erase time: 31.453000

map内存：5096K
unordered_map内存：6712K
```

100w量级的耗时，结论同上。

```
map insert time: 955.625000
map find time: 574.289000
map erase time: 623.460000
unordered_map insert time: 575.636000
unordered_map find time: 166.449000
unordered_map erase time: 294.509000

map内存：47M
unordered_map内存：50M
```

**综上，抛出结论，在需要有序性或者对单次查询有时间要求的应用场景下，应使用map，其余情况应使用unordered_map。**

附上测试代码，：

```
#include <iostream>
#include <string>
#include <sstream>
#include <list>
#include <map>
#include <time.h>
#include <stdio.h>
#include <unistd.h>
#include <unordered_map>

using namespace std;

const int num = 100000;

class timer {
public:
	clock_t start;
	clock_t end;
	string name;
	timer(string n) {
		start = clock();
		name = n;
	}
	~timer() {
		end = clock();
		printf("%s time: %f \n", name.c_str(), 
			(end - start) * 1.0 / CLOCKS_PER_SEC * 1000);
	}
};

template<typename T> 
void insert(T & conta, string name) {
	srand((unsigned)time(NULL));  
	timer t1(name);
	for (int i = 0; i < num / 2; i++) {
		int key = rand();
		conta.insert(pair<int, int>(i, i));
		conta.insert(pair<int, int>(key, i));
	}

}

template<typename T>
void find(T & conta, string name) {
	srand((unsigned)time(NULL));  
	timer t1(name);
	for (int i = 0; i < num / 2; i++) {
		int key = rand();
		conta.find(key);
		conta.find(i);
	}
}

template<typename T>
void erase(T & conta, string name) {
	srand((unsigned)time(NULL));  
	timer t1(name);
	for (int i = 0; i < num / 2; i++) {
		conta.erase(i);
		int key = rand();
		conta.erase(key);
	}
}


void test_map() {
	map<int, int> m1;
	insert<map<int, int> >(m1, "map insert");
	find<map<int, int> >(m1, "map find");	
	erase<map<int, int> >(m1, "map erase");
}

void test_unordered_map() {
	unordered_map<int, int> m2;
	insert<unordered_map<int, int> >(m2, "unordered_map insert");	
	find<unordered_map<int, int> >(m2, "unordered_map find");
	erase<unordered_map<int, int> >(m2, "unordered_map erase");
}

int main(){
	test_map();
	test_unordered_map();
}
```