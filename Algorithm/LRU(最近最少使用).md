LRU（Least Recently Used）是一种常用的缓存淘汰策略，其核心思想是当缓存空间满时，移除最近最少使用的数据项，即最久没有被访问过的数据项。LRU算法广泛应用于操作系统和应用程序中的内存管理，特别是在缓存系统中。
##### **复杂度**
时间复杂度 O(1)
空间复杂度O(min(p,capacity)) p--put的调用次数
ref：[146. LRU 缓存 - 力扣（LeetCode）](https://leetcode.cn/problems/lru-cache/)
[146. LRU 缓存 - 力扣（LeetCode）](https://leetcode.cn/problems/lru-cache/solutions/2456294/tu-jie-yi-zhang-tu-miao-dong-lrupythonja-czgt/)
##### 优缺点
优点：高效性，所有操作都可以在O(1)时间复杂度内完成；简单易懂，适用于缓存管理等场景。
缺点：内存消耗较大，需要额外的空间存储哈希表和链表；链表操作虽然时间复杂度低，但相比数组实现的缓存需要更多的内存空间。
##### 应用
Redis：采用近似LRU算法，通过随机采样N个key，选择其中最久未使用的key进行淘汰，以降低维护全量键值对链表的开销。
MySQL Buffer Pool：对LRU算法进行改进，将链表分为young和old两个区域，分别存放最近访问的新页面和旧页面，以应对预读失效和Buffer Pool污染问题。
LRU算法通过维护数据项的使用顺序，有效地管理缓存空间，提高数据访问效率，在实际应用中发挥着重要作用。
##### 实现
LRU算法通常结合哈希表和双向链表来实现：
**哈希表**：用于存储键与数据项的映射，支持O(1)时间复杂度的查找操作。
**双向链表**：用于按顺序维护数据项的使用状态。链表头部表示最近使用的元素，尾部表示最久未使用的元素，插入、删除操作可以在O(1)时间内完成。
##### 工作原理
访问元素（get）：查找缓存项，若存在，返回其值，并将该元素移动到链表头部；若不存在，返回-1。
插入元素（put）：若缓存项已存在，更新其值并移动到链表头部；若不存在，插入新元素到链表头部。若缓存已满，移除链表尾部的元素（最久未使用的元素）。
![[lru.png]]
##### 模板
根据原理，首先实现双向链表
```
//头文件
struct DoubleNode {
	int key;
	int value;
	DoubleNode* prev;
	DoubleNode* next;
	DoubleNode();
	DoubleNode(int _key, int _value);
};
//实现
DoubleNode::DoubleNode()
{
	key = 0;
	value = 0;
	prev = nullptr;
	next = nullptr;
}

DoubleNode::DoubleNode(int _key, int _value)
{
	key = _key;
	value = _value;
	prev = nullptr;
	next = nullptr;
}
```
LRU类的实现
```
//声明
class LRU
{
	std::unordered_map<int, DoubleNode*> umap;
	int _capcity;
	int size;
	DoubleNode* head;
	DoubleNode* tail;
public:
	LRU();
	LRU(int capcity);
	~LRU();
	int get(int key);
	void put(int key, int value);
	void moveToHead(DoubleNode* node);
	void deleteNode(DoubleNode* node);
	void addToHead(DoubleNode* node);
	DoubleNode* deleteTail();
};

//实现
LRU::LRU()
{
	_capcity = 5;
	size = 0;
	head = new DoubleNode(-1, -1);
	tail = new DoubleNode(-1, -1);
	head->next = tail;
	tail->prev = head;
}

LRU::LRU(int capcity)
{
	_capcity = capcity;
	size = 0;
	head = new DoubleNode(-1, -1);
	tail = new DoubleNode(-1, -1);
	head->next = tail;
	tail->prev = head;
}

LRU::~LRU()
{
	delete head;
	delete tail;
}

int LRU::get(int key)
{
	//不存在
	if (!umap.count(key)) { return -1; }
	//存在，返回值
	DoubleNode* node = umap[key];
	moveToHead(node);
	return node->value;
}

void LRU::put(int key, int value)
{
	//不存在
	if (!umap.count(key)) {
		DoubleNode* node = new DoubleNode(key, value);
		size++;
		addToHead(node);
		umap[key] = node;
		//超过容量，删除最后的节点
		if (size > _capcity) {
			DoubleNode* temp = deleteTail();
			umap.erase(temp->key);
			delete temp;
			size--;
		}
	}
	else {
		//更新值
		if (umap[key]->value != value) {
			umap[key]->value = value;
		}
		moveToHead(umap[key]);
	}
}

//移除node的关系，放在最前面
void LRU::moveToHead(DoubleNode* node)
{
	deleteNode(node);
	addToHead(node);
}
//基本函数：删除节点的关系--移除
void LRU::deleteNode(DoubleNode* node)
{
	if (node == nullptr) { return; }
	node->next->prev = node->prev;
	node->prev->next = node->next;
}
//基本函数：把节点放到最前面
void LRU::addToHead(DoubleNode* node)
{
	node->prev = head;
	node->next = head->next;
	node->next->prev = node;
	head->next = node;
}

//移除最后节点的关系，返回该节点
DoubleNode* LRU::deleteTail()
{
	DoubleNode* node = tail->prev;
	deleteNode(node);
	return node;
}
```

##### 关键流程

双向链表、哈希表（unordered_map<int,doublenode*>）、头指针和尾指针、容量、当前大小

```
int get(int key)
void put(int key,int value)

void remove(doublenode* node) 移除节点的关系
void addtohead(doublenode* node) 增加节点到头
void movetohead(doublenode* node) 已有的移动到头

doublenode*  deleteTail() 删除末尾的节点 并返回
```

get函数返回key的值，如果不存在，返回-1.
实现方法：
1. 使用哈希表判断是否存在，不存在则返回-1.
2. 取出doublenode，调用movetohead，使该节点到头
3. 返回值

put() 函数--如果存在key更新value，不存在就增加一个新的key，如果整体大小大于capacity，删除最后一个
实现方法：
1. 判断存在key
2. 不存在，新建一个节点，addtohead，如果size>capacity，调用deletetail，删除
3. 存在，如果不一样就更新，movetohead