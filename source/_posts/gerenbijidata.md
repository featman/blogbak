title: 复习笔记-数据结构  
date: 2015-3-29 15:31:49
categories: 总结复习
tags: [知识点整理,数据结构] 
---
总结复习，数据结构
<!--more-->
#### 1.[可视化演示](http://blog.csdn.net/bigleo/article/details/41219647)
> [可视化数据结构](http://www.cs.usfca.edu/~galles/visualization/Algorithms.html )   
> [C++实现的各种算法演示](http://people.cs.pitt.edu/~kirk/cs1501/animations/)   
> [很酷的各种排序演示](http://sorting.at/)   
> [很有创意的排序比较（匈牙利 Sapientia 大学的 6 种排序算法舞蹈视频](http://top.jobbole.com/1539/)）  

![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/33611161.jpg)

#### 快速排序所谓的最好情况和最坏情况不一样，如何才是最好，如何才是最坏？
最好情况是，选取到那个基准正好为比较中间的值，比如一个序列，10，23，45，8，9，这样就比较好，彼此到次数少，所以时间复杂度会变小，为nlog2n，
最坏情况是，选取到这个序列有一定到顺序，这样再排序到时候，就会使得比较次数增加（每一轮比上次少1个），时间复杂度o(n2)
####  所谓的稳定与不稳定是什么意思？
假如待排序的一个序列中有A和B两个元素相等，如果在排序的过程中，有可能交换两个元素到位置，那么说这个排序算法不稳定。如果没有交换次序，那么就说这个排序算法是稳定的。
#### 1.冒泡排序：O（n2），稳定。
从前往后依次比较两个数的大小，如果按递减的顺序排序的话，那么比较一趟下来，最小的数应该位于最后一个数上。然后再从第一个继续比较，第二趟完了，会把次小的放在倒数第二个，以此类推。
#### 2.选择排序：O（n2），不稳定。
首先应选择第一个为基准，开始第一堂排序，之后的所有值与这个基准进行比较，如果a[i]小于基准，那么把a[i]定位新的基准，在用新的基准比较后边的值，一直到最后为止，这时候，把新的基准和第一个交换为止，第一趟结束。然后同理进入第二趟。
#### 3.快速排序：好的时候是O（nlogn）,坏的时候是O（n2），不稳定
任取一个值，一般取第一个，然后定义两个游标，一个游标指向最左边，另一个游标指向最右边，然后右边的游标先动，从右往左查找比第一个数小的值，找到就停止。然后左边的开始往右边查找一个比第一个大的数，找到为止，然后开始交换两个游标指向的值，直到左右游标相遇了。这时候再把第一个值与游标目前为止的值互换一下，进入下一次递归调用。
在算法实现的过程中需要注意的问题：
- a.由于进行的是递归调用，下边还需要使用right和left，所以right和left不能直接参与本次循环，需要引入r和l，如r=right,l=left
- b.第一层循环while(l<r)，这时候不能在下边的设置为if(arr[r] >= arr[index]) 因为如果是if判断，只是简单判断第一次是否成立，并不能保证找到了比基准小的值，所以需要把条件变为while(arr[r] >= arr[index]) 
- c.由b改成while()之后，需要再条件中添加一个条件l<r，因为如果不添加那么就会一直减下去。
- d.在递归调用的时候，必须要有一个判断退出的，不然会无线循环。if(l>r) return;

```
#include "General.h"
/*
 *  快速排序所谓的最好情况和最坏情况不一样，如何才是最好，如何才是最坏？
 *	最好情况是，选取到那个基准正好为比较中间的值，比如一个序列，10，23，45，8，9，这样就比较好，彼此到次数少，所以时间复杂度会变小，为nlog2n，
 *	最坏情况是，选取到这个序列有一定到顺序，这样再排序到时候，就会使得比较次数增加（每一轮比上次少1个），时间复杂度o(n2)
 */
void quickSort(int arr[], int left, int right){

	if(left > right) return;//递归必须要有退出的语句！
	int index = left;
	int l = left, r = right;
	while(l < r){
		while(arr[r]>arr[index] && l<r) r--;
		while(arr[l]<arr[index] && l<r) l++;
		int tmp = arr[r];
		arr[r] = arr[l];
		arr[l] = tmp;
	}
	int temp = arr[l];
	arr[l] = arr[index];
	arr[index] = temp;


	quickSort(arr,left,l-1);
	quickSort(arr,l+1,right);
}

```

#### 4.[堆排序](http://www.cnblogs.com/mengdd/archive/2012/11/30/2796845.html)：O（nlogn），不稳定
分为两个过程，一个是建立堆的过程，一个是拿出祖先节点后，重新调整堆的过程。d
建立堆之后，祖先节点就是一个最值了，把这个最值拷贝到另外一个数组的第一个位置，然后在原来的数组上（即堆上），删除根节点，然后现在的数组就需要重新调整为一个新的堆，在把这个堆的第一个元素取出，放到另外一个数组的第二个位置。以此类推。


```
#include "General.h"


void adjustHeap(int arr[],int start,int end){
	//int tmp = arr[start];
	int i;
	//每次调整3个，如果传入进来的节点start,i,i+1，已经是一个有序的序列，那么循环执行一次就会结束。
	//如果传入进来的三个节点顺序不对，那么先调整这三个节点，然后再进入下一次循环以i或者i+1作为新的父节点进行调整，有可能调整到叶子节点，直到满足break。
	//NOTICE:for中一定要i<=end，而下边第一条执行if一定要有i<end，
	//考虑叶子节点只有一个左孩子的情况，此时也需要判断父亲和左孩子的大小，因为左孩子没有兄弟，所以不能跟i+1比较，会出现不可预料的后果。
	for(i=2*start+1;i<=end;i*=2){
		if(arr[i] < arr[i+1] && i<end) i++;//选择大的子节点//这里的大于小于就确定了是大根堆还是小根堆
		if(arr[i] < arr[start]) break;//选择子节点和父节点中较大的一个
		int temp = arr[i];
		arr[i] = arr[start];
		arr[start] = temp;
		start = i;//为下一次节点调整做准备，下次循环中start就是“父节点”。
	}
}



//这里的n传入的是个数，不是数组下标最大值
void heapSort(int arr[], int n){
	int i ;
	//建立一个初始堆，这个堆是一个原始的序列
	//此循环从n/2开始，是要从中间部分向上逐渐调整。
	//循环最终得到一个完整的大根堆/小根堆
	for(i=n/2;i>=0;i--){//这里i有=0，是因为根节点也要和左右调整
		adjustHeap(arr,i,n-1);
	}


	//每次取出根节点，放置序列的尾部，空间复杂度o1
	//这样的话，在大根堆输出的序列就是一个递增的序列
	for(i=n-1;i>0;i--){//这里i没有=0，是因为下边一直在和arr[0]换，arr[0]不需要和本身换。
		int tmp = arr[0];
		arr[0] = arr[i];
		arr[i] = tmp;
		//交换完之后还要调整剩下的堆，这个堆从上往下调整，直到再次选出其余节点中最值
		adjustHeap(arr,0,i-1);//i-1是因为每次交换的都是最后一个和第一个，所以需要从0调整到倒数第二个。
	}
}
```
#### 5.二分查找(复杂度log2n，最大次数为log2n+1)
在给定的一个有序递增序列中，拿指定值和中间值进行比较，如果相等，那么返回中间值，如果不等，则需要判断指定值和中间值的大小情况，如果小于中间值，那么要找的值在左边，所以递归调用本身，但是参数的上界变为mid-1.大于中间值，同理。需要注意的地方是循环的条件是Low<=high，因为相等的情况还是有一个节点的。

```
//查找的出是序列必须是连续存储，比如数组，链表不适合。另外被查找的序列应该有序，如递增

//递归查找
int binSearch(int arr[],int mete,int start,int end){
	int n = (start+end)/2;//key
	if(start > end) return -1;//递归要有退出
	if(arr == NULL) return -1;
	if(arr[n] == mete) return n;
	if(arr[n] < mete)
		return binSearch(arr,mete,n+1,end);
	else
		return binSearch(arr,mete,0,n-1);
	return -1;
}

//循环查找
int binSearchOut(int arr[],int mete,int start,int end){
	if(arr == NULL) return -1;

	while(start <= end){
		int n = (start+end)/2;
		if(mete == arr[n])
			return n;
		else if(mete > arr[n])
			start = n+1;
		else
			end = n-1;
	}
	return -1;
}
```

#### 6.二叉树的遍历

```
typedef struct BiTree {
    int data;
    struct BiTree *lchild,*rchild;
}treenode,*BiTree;
```

- 前序遍历：根-》左-》右  
- 中序遍历：左-》根-》右  
- 后序遍历：左-》右-》根  
> 以上的均代表一个节点的情况，例如根左右说的就是treenode中的，data(根)，lchild（左)，rchild(右)。

```
void zhongxu(BiTree T){
    zhongxu(T->lchild);
    T->data;
    zhongxu(T->rchild);
}x
```
由左、中序列或者中、右序列 均可以推出二叉树，但是只知道左右并不能推出二叉树。
##### 6.1[线索化二叉树](http://blog.chinaunix.net/uid-26548237-id-3476920.html)
#### 7.图——深度优先搜索遍历
任取一个顶点，找出刚访问过的顶点的第一个未被访问的邻接点，然后这个邻接点为新的节点，重复上述过程，直到没有邻接点为止。这时候往上层回退，一直到访问完所有的顶点。每个节点仅访问一次。

算法：
需要一个数组，该数组记录节点是否被访问，如果被访问，则值为1，初始值为0.
#### 8.图——广度优先搜索遍历
类似于树的按层次遍历
任取一个顶点，依次访问这个顶点的所有邻接点。然后再分别从这些邻接点出发依次访问每个“邻接点”的邻接点。并使“先被访问的顶点的邻接点”先于“后被访问的顶点的邻接点”。
#### 9.赫夫曼编码
官方定义：以N种字符出现的频率作权计算来设计一颗赫夫曼树，由此得到的二进制前缀编码便成为赫夫曼编码。

前缀编码：任何一个字符都不是另一个字符的编码的前缀，满足这个关系就是前缀编码。

二叉树设计前缀编码，规定左分支标记为0，右分支标记为1.这样每个字符的最终编码就是从根节点到字符的所走的路径。

赫夫曼树的构建：
首先我们必须要知道的是每个字符的权值（如概率、出现的次数等等，这些都可以作为权值）

然后我们在给出的字符权值组成一个序列，从这个序列中先选两个最小的（比如1和2），然后作为左右孩子节点

接着，算出左右孩子节点的父节点权值（左右孩子权值相加）。

算出过后，把（1和2）从原来的序列中删除，把新的父节点权值加入到序列中（把1+2=3加入到序列中）

最后就重复做选小的，然后相加，然后删除，在继续做，直到序列中没有为止。

[（例子）](http://www.cnblogs.com/Jezze/archive/2011/12/23/2299884.html)

赫夫曼树的定义：哈夫曼树又称最优二叉树，是一种带权路径长度最短的二叉树

权值大的离根越近，权值小的离根越远。也就是权值大的，编码位数少。

每个字符的编码值并不是唯一的，因为在上述构建过程中，左右孩子节点的选取就不是固定的，所以编码值不唯一。
但是字符所在的层次是固定的。也就是说位数是确定的。
#### 10.哈希表
无顺序、基于数组
优点，查找速度相当快，O1
缺点，无法遍历，扩展性差（数组无法扩展）。
#### 10.[平衡二叉树AVL与红黑树](http://www.cnblogs.com/BeyondAnyTime/archive/2012/08/27/2659163.html)
http://www.bbniu.com/matrix/ShowApplication.aspx?id=149   
红黑树的应用：
> 红黑树的应用比较广泛，主要是用它来存储有序的数据，它的时间复杂度是O(lgn)，效率非常之高。
> 例如，Java中的TreeSet和TreeMap，C++ STL中的set、map，以及Linux虚拟内存的管理，都是通过红黑树去实现的。
> 这里大致介绍下，红黑树和AVL树的差异。AVL树也是特殊的二叉树，它的特性是“任何节点的左右子树的高度之差不超过1”。基本上，用到红黑树的地方都可以用AVL树(自平衡二叉查找树)去替换。但是一般情况下，在执行添加、删除节点时，AVL树比红黑树执行的操作更多一些，效率更低一些；而且红黑树也是相对平衡的二叉树(从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点)。因此，红黑树的效率会高更一点。 
http://www.csdn123.com/html/itweb/20130813/57700_57702_57699.htm

#### 11.[左旋和右旋](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/71866125.jpg)

#### 12.再说 树

{二叉树：二叉树，（二叉排序树，AVL，红黑树），二叉堆，哈夫曼树}

{多叉树：B树、B+树}
#### 13.[红黑树](http://www.csdn123.com/html/itweb/20130813/57700_57702_57699.htm)
> 查找、插入、删除 时间复杂度为log N。

红黑树的特性：（属于二叉排序树的一种，包含其特性）
- a。树中的节点，要么是黑色的，要么是红色的。
- b。根节点一定是黑色的。
- c。红色节点的孩子节点一定为黑色的。
- d。每个叶子节点都是黑色的。
- e。任意一个节点到所有该节点的子孙节点的路径中包含相同数量的黑色节点

对x进行左旋，意味着，将“x的右孩子”设为“x的父亲节点”；即，将 x变成了一个左节点(x成了为z的左孩子)！。 因此，左旋中的“左”，意味着“被旋转的节点将变成一个左节点”。
![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/37266332.jpg)
#### 14.[二叉排序树](http://www.cnblogs.com/zhuyf87/archive/2012/11/09/2763113.html)（查找时间复杂度为On，BST）

左子树小于父亲节点、右子树大于父亲节点。左右子树依然有序。
#### 15.平衡二叉查找树（AVL，时间复杂度O（logN）  ）
要求对于每个节点来说，它的左右子树的高度之差不能超过1，否则就要进行节点之间的旋转，使之达到平衡状态。
#### 16.为什么需要AVL或者R.B Tree
AVL和红黑树都为二叉排序树的一种，想象如果只有二叉排序树，极端情况下，二叉排序树只有1，2,3,4,5，这五个数字组成了一个单方向的二叉树，仔细看可以看出，其实成为了一个链表，链表的查找效率低下，需要一个高效的查找数据结构，AVL和红黑树可以实现。
#### 17.二叉堆分为大根堆和小根堆，大根堆中父节点的值永远大于子节点的值。
#### 18.string实现


```


#include "MyString.h"


MyString::MyString(){
 string = new char[1];
 *string = '\0';
}

MyString::MyString(const char* str){
 if(str == NULL){
  string = new char[1];
  *string = '\0';
 }else{
  int len = strlen(str);
  string = new char[len+1];
  strcpy(string,str);
 }
}

MyString::MyString(const MyString& ms){
 int len = strlen(ms.string);
 this->string = new char[len+1];
 strcpy(this->string,ms.string);
}

MyString::~MyString(){
 delete[] this->string;
}

MyString& MyString::operator=(const char* str){
 int len = strlen(str);
 this->string = new char[len+1];
 strcpy(this->string,str);
 return *this;
}

MyString MyString::operator+(const MyString& ms){
 int len = strlen(ms.string);
 MyString str;
 str.string = new char[len+strlen(this->string)+1];
 strcpy(str.string,this->string);
 strcat(str.string,ms.string);
 return str;
}

char MyString::operator[](unsigned int index){
 char chr;
 chr = this->string[index];
 return chr;
}

bool MyString::operator==(const MyString &ms){
 int res = strcmp(ms.string,this->string);
 if (!res){
  return true;
 }else{
  return false;
 }
}

ostream& operator<<(ostream& out,const MyString& ms){
 out<<ms.string;
 return out;
}
```
#### 19.atoi 实现

```
int my_atoi(const char* str){ 
int res = 0; 
int flag = 1; 
if(!str) return 0;

while(*str == ' ') str++;

if(*str == '-') { 
flag = -1; 
str++; 
}

while(*str >= '0' && *str <= '9'){ 
res = *str-'0'+res*10; 
str++; 
}

return res*flag; 
} 
```



#### 20.itoa 实现

```
char* reverse(char* str){ 
	char tmp; 
	char* p = str; 
	char* q = str; 
	while(*p) p++; 
	p--; 


	while(p>q){ 
		tmp = *p; 
		*p = *q; 
		*q = tmp; 
		p--; 
		q++; 
	} 
	return str; 
} 




char* my_itoa(int val){ 
	char flag = 0; 
	int i = 0; 
	static char str[100] = "\0"; 
	if(!val) return 0; 
	if(val < 0) { 
		flag = '-'; 
		val = val*(-1); 
	} 
	while(val){ 
		str[i++] = val%10+'0'; 
		val /= 10; 
	} 
	str[i] = flag; 
	return reverse(str); 
} 
```
