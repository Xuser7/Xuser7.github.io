
$$
线性表
\begin{cases}
    顺序存储——顺序表\\
    链式存储
    \begin{cases}
        单链表（指针实现）\\双链表（指针实现）\\循环链表（指针实现）\\静态链表（数组实现）
    \end{cases}\\
\end{cases}\\
$$

## 2.1 线性表的定义和基本操作

### 2.1.1   线性表的定义

线性表是具有相同数据类型的n个数据元素的有限序列。除第一个元素外，每个元素有且仅有一个直接前驱。除最后一个元素外，每个元素有且仅有一个直接后继。

### 2.1.2   线性表的基本操作

```c
InitList(&L) //初始化表。构造一个空表

Length(L) //求表长。返回线性表L的长度，即L中数据元素的个数

LocateElem(L,e) //按值查找。在表L中查找具有给定关键字值的元素e

GetElem(L,i) //按位查找。获取表L中第i个位置的元素的值

ListInsert(&L,i,e) //插入操作。在表L的第i个位置上插入指定元素e

ListDelete(&L,i,&e) //删除操作。删除表L中第i个位置的元素，并且用e返回删除元素的值

PrintList(L) //输出操作，按前后顺序输出线性表L的所有元素值

Empty(L) //判空操作。若L为空表，则返回true，否则返回false

DestroyList(L) //销毁操作。销毁线性表，并且释放线性表L所占用的内存空间
```

## 2.2   线性表的顺序表示

### 2.2.1   顺序表的定义

- 定义：线性表的顺序存储，它是用一组连续的存储单元依次存储线性表中的数据元素，从而使得逻辑上相邻的两个元素在物理位置上也相邻。（特点：逻辑顺序与其存储的物理顺序相同）

- 随机存取：每个数据元素的存储位置都和顺序表的起始位置相差 和该元素的位序成正比 的常数.
    
    `LOC(List) + (n-1)*sizeof(ElemType)` （起始地址）+（位序偏移量）*（元素存储空间）

    注意到n-1，因为线性表的位序是从1开始的，而数组中元素的下标是从0开始的

- 存储结构代码

    ```c
    //静态分配的顺序表
    #define MAXSIZE     50      //定义线性表的最大长度
    typedef struct{
        ElemType data[MAXSIZE]; //顺序表的元素
        int length;             //顺序表的当前长度
    }SeqList;

    //动态分配的顺序表
    #define INITSIZE    100     //表长度的初始定义
    typedef struct{
        ElemType *data;         //指示动态分配数组的指针
        int MaxSize,length;     //数组的最大容量和当前个数
    }SeqList;
    ```

### 2.2.2   顺序表上基本操作的实现

- 初始化

    ```c
    SeqList L;      //声明一个顺序表

    //初始化静态分配的顺序表
    //静态分配在声明顺序表时，就已经为其分配了数组空间，因此只需要将顺序表的当前长度设为0
    void InitList(SeqList &L){
        L.length = 0;               //顺序表初始长度为0
    }

    //初始化动态分配的顺序表
    //动态分配的初始化为顺序表分配一个预定义的数组空间，并将顺序表的当前长度设为0。MaxSize指示顺序表当前分配的存储空间大小
    void InitList(SeqList &L){
        L.data = (ElemType*)malloc(INITSIZE*sizeof(ElemType));//分配存储空间
        L.length = 0;               //顺序表初始长度为0
        L.MaxSize = INITSIZE;       //初始化存储容量
    }

    ```

- 插入：若插入位置i合法，则将第i个元素及其后的所有元素依次往后移动一个位置，腾出一个空位给新元素

    ```c
    //平均时间复杂度：O(n)
    bool ListInsert(SeqList &L,int i,ElemType e){
        if(i<1||i>L.length+1)           //判断i的范围是否有效
            return false;
        if(L.length>=MaxSize)           //当前存储空间已满，不能插入
            return false;
        for(int j=L.length;j>=i;j--)    //将第i个元素及之后的元素后移
            L.data[j]=L.data[j-1];
        
        L.data[i-1]=e;                  //在位置i处放入e                              
        L.length++;                     //线性表长度加1
        return true;
    }
    ```

- 删除

    ```c
    //平均时间复杂度：O(n)
    bool ListDelete(SeqList &L,int i,ElemType &e){
        if(i<1||i>L.length)         //判断i的范围是否有效
            return false;
        e=L.data[i-1];              //将被删除的元素赋值给e返回
        for(int j=i;j<=length;j++)  //将第i个位置后的元素前移
            L.data[j-1]=L.data[j];  
        L.length--;                 //线性表长度减1   
        return true;
    }
    ```

- 按值查找

    ```c
    //平均时间复杂度：O(n)
    int LocateElem(SeqList &L,ElemType e){
        int i;
        for(i=0;i<L.length;i++)
            if(L.data[i]==e)
                return i+1;     //下标为i的元素值等于e,返回其位序i+1
        return 0;               //退出循环，说明查找失败
    }
    ```

## 2.3   线性表的链式表示

### 2.3.1   单链表的定义



- 单链表：使用任意的存储单元来存储线性表中的数据元素。为了建立不同存储空间数据元素之间的线性关系，对每个链表结点，除存放元素自身的信息外，还需要存放一个指向其后继的指针。data为数据域，next为指针域（存放后继结点的地址）

    | data | next |
    |------|------|
    | 数据域 | 指针域 |

    ```c
    typedef struct LNode{       //定义单链表结点类型
        ElemType data;          //数据域
        struct LNode *next;     //指针域
    }LNode,*LinkList;
    ```

- 非随机存取：利用单链表可以解决顺序表需要大量连续存储单元的缺点，但附加的指针域，也存在浪费存储空间的缺点。由于单链表的元素离散地分布在存储空间中，因此是非随机存取的存储结构，不能直接找到表中某个特定结点，查找特定结点时，需要从表头开始遍历。

- 链表头结点：头结点是带头结点链表中的第一个结点，结点data域通常不存储信息。引入头结点，可以带来两个优点。

    1.由于第一个数据结点的位置被存放在头结点的指针域next中，因此在链表的第一个数据结点的操作和表中其他位置上的操作一致，避免了该位置的特殊处理。

    2.链表通常使用头指针来标识，指出链表的起始地址。若不引入头结点，那么头指针为NULL时表示空表。引入头结点后，无论链表是否为空，链表头指针都是指向头结点的非空指针，使得空表和非空表的处理得到了统一。

    `头指针(L或head) --> 头节点(data|next) --> 第一个数据结点(data|next) --> NULL`

### 2.3.2   单链表基本操作

- 单链表的初始化

    ```c
    //带头结点的单链表初始化
    bool InitList(LinkList &L){
        L=(LNode*)malloc(sizeof(LNode));    //创建头结点
        L->next = NULL;                     //头结点后暂时还没有元素
        return true;
    }

    //不带头结点的单链表初始化
    bool InitList(LinkList &L){
        L=NULL;
        return true;
    }
    ```

- 求表长

    ```c
    int Length(LinkList L){
        int len=0;              //计数变量
        LNode *p=L;
        while(p->next!=NULL){
            p=p->next;
            len++;              //每访问一个结点，计数加1
        }
        return len;
    }
    ```

- 查找结点

    ```c
    //按序号查找
    LNode *GetElem(LinkList L,int i){
        LNode *p=L;                 //指针p指向当前扫描到的结点
        int j=0;                    //记录当前结点的位序，头结点是第0个结点
        while(p!=NULL&&j<i>){       //循环找到第i个结点
            p=p->next;
            j++
        }
        return p;                   //返回第i个结点的指针
    }

    //按值查找
    LNode *LocateElem(LinkList L,ElemType e){
        LNode *p=L->next;
        while(p!=NULL&&p->data!=e)      //从第一个结点开始查找数据为e的结点
            p=p->next;
        return p;                       //找到后返回该结点指针，否则返回NULL
    }

    ```

- 插入结点操作：先检查插入位置的合法性，然后找到待插入位置的前驱，再在其后插入。

    ```c
    bool ListInsert(LinkList &L,int i,ElemType e){
        LNode *p=L;                                 //指针p指向当前扫描到的结点
        int j=0;                                    //记录当前结点的位序，头结点是第0个结点
        while(p!=NULL&&j<i-1){                      //循环找到第i-1个结点
            p=p->next;
            j++;
        }
        if(p==NULL)                                 //i值不合法
            return false;

        LNdoe *s=(LNode*)malloc(sizeof(LNode));
        s->data=e;
        s->next=p->next;
        p->next=s;
        return true;
    }
    ```

- 删除结点操作：先检查删除位置的合法性，然后找到被删除结点的前驱，再删除结点。

    ```c
    bool ListDelete(LinkList &L,int i,ElemType &e){
        LNode *p=L;
        int j=0;

        while(p!=NULL&&j<i-1){                      //循环找到第i-1个结点
            p=p->next;
            j++;
        }

        if(p==NULL||p->next==NULL)                  //i值不合法
            return false;

        LNode *q=p->next;                           //令q指向被删除结点
        e=q->data;                                  //用e返回元素的值
        p->next=q->next;                            //将*q结点从链中“断开”
        free(q);
        return true;
    }
    ```

- 头插法建立单链表：从空表开始，生成新结点，并将读取到的数据存放到新结点的数据域中，然后将新结点插入到当前链表的表头。读入数据的顺序与生成的链表中元素的顺序是相反的，可用来实现链表的逆置。

    ```c
    //逆向建立单链表
    LinkList List_HaedInsert(LinkList &L){
        LNode *s; int x;                        //设元素类型为整型
        L=(LNode*)malloc(sizeof(LNode));        //创建头结点
        L->next=NULL;                           //初始为空链表
        scanf("%d",&x);                         //输入结点的值
        while(x!=9999){                         //输入9999表示结束
            s=(LNode*)malloc(sizeof(LNode));    //创建新结点
            s->data=x;
            s->next=L->next;
            L->next=s;                          //将新结点插入表中，L为头指针
            scanf("%d",&x)
        }
        return L;
    }
    ```

- 尾插法建立单链表：头插法建立的链表结点次序与输入数据相反，若希望顺序一致，可以采用尾插法。该方法将新结点插入到当前链表的表尾，因此必须增加一个尾指针r，使其始终指向当前链表的尾结点。

    ```c
    //正向建立单链表
    LinkList List_TailInsert(LinkList &L)}{
        int x;                                  //设元素类型为整型
        L=(LNode*)malloc(sizeof(LNode));        //创建头结点
        LNode *s,*r=L;                          //r为表尾指针
        scanf("%d",&x);                         //输入结点的值
        while(x!=9999){                         //输入9999表示结束
            s=(LNode*)malloc(sizeof(LNode));
            s->data=x;
            r->next=s;
            r=s;                                //r指向新的表尾结点
            scanf("%d",&x);
        }
        r->next=NULL;                           //尾结点指针置空
        return L;
    }
    ```

### 2.3.3   双链表

单链表结点中只有一个指向其后继的指针，使得单链表只能从前往后依次遍历。要访问某个结点的前驱（插入、删除时），只能从头开始遍历，访问前驱的时间复杂度为O(n)。为了克服这个缺点，引入了双链表，双链表有两个指针prior和next，分别指向其直接前驱和直接后继。

```c
//双链表的结点类型描述
typedef struct DNode{
    ElemType data;              //数据域
    struct DNode *prior,*next;  //前驱和后继指针
}DNode,*DLinklist
```

- 双链表的插入操作

    `s->next=p->next;`

    `p->next->prior=s;`

    `s->prior=p;`

    `p->next=s;`

- 双链表的删除操作

    `p->next=q->next;`

    `q->next->prior=p`;

    `free(q);`

### 2.3.4   循环链表

- 循环单链表：循环单链表和单链表的区别在于，表中最后一个结点的指针不是NULL，而是改为指向头结点，从而整个链表形成一个环。循环单链表判空条件：头结点的next域等于头指针L。

- 循环双链表：和循环单链表类似，不同之处在于，循环双链表中，头结点prior指针还要指向表尾结点。循环双链表判空条件：头结点的prior和next域都等于L。

### 2.3.5   静态链表

静态链表是用数组来描述线性表的链式存储结构，结点也有data数据域和next指针域，但和上述动态链表不同的是，这里的指针域是结点在数组中的（数组下标），故next指针域并不是指针，而是数值。

```c
//静态链表结构类型的描述
#define MAXSIZE     50  //静态链表的最大长度
typedef struct{
    ElemType data;      //存储数据元素
    int next;           //下一个元素的数组下标
}SLinkList[MAXSIZE];
```

静态链表以`next==-1`作为其结束的标志。静态链表的操作与动态链表相同，只需要修改指针域next，不需要移动元素。但总体而言，静态链表没有单链表使用起来方便，但在不支持指针的语言中，是一种方法。

## 附：链表代码题

```C
/*-----------------------一律使用带头节点的链表，方便操作-----------------------*/

//结点结构体 
typedef struct LNode{
    int data;
    struct LNode *next;
}LNode,*List;

//初始化申请
List p = (List)malloc(sizeof(LNode));//链表概念，已是指针形式
LNode *p = (LNode*)malloc(sizeof(LNode));//节点概念，变量名定义指针
```

### 逆置带头节点的单链表

```C
List Reverse(List head)
{
    LNode *p,*q,*r;//三指针p q r
    p = head->next;
    q = p->next;
    p->next = NULL;

    while(q){
        //pqr位置按顺序排列
        r = q->next;
        //逆指
        q->next = p;
        //后移
        p = q;
        q = r;

        head->next = p;
    }
    return head;
}
```

### 带头结点的单链表删除重复元素

```C
void Delete(List &L)
{
    LNode *p = L->next,*q,*prep;

    while(p!=NULL)
    {
        prep = p;
        q = p->next;
        while(q!=NULL)
        {
            if(p->data==q->data){
                prep->next = q->next;
                free(q);
                q = prep->next;
            }
            else{
                prep=q;
                q=q->next;
            }
        }
        p=p->next;
    }
}
```

### 合并递增有序的的A链表与B链表，形成递增有序的新链表C

```C
List Merge(List &A,List &B)
{
    List C=(List)malloc(sizeof(LNode));
    //pc指针指向c尾结点
    LNode *pc=C;
    //pa，pb访问a,b的每个节点
    LNode *pa=A->next,*pb=B->next;

    //循环添加小元素
    while(pa&&pb){
        if(pa->data<=pb->data)
        {
            pc->next=pa;
            pc=pa;
            pa=pa->next;
        }
        else
        {
            pc->next=pb;
            pc=pb;
            pb=pb->next; 
        }
    }
    //其中一表已空，另外一表属于直接并入
    if(pa){
        pc->next=pa;
    }
    if(pb){
        pc->next=pb;
    }
    return C;
}
```