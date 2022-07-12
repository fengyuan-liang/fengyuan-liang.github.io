# AVL树

思维导图：

![image-20220408170620773](https://cdn.fengxianhub.top/resources-master/202204081706974.png)

## 1. 二叉搜索树复杂度

>我们来分析一下BST的复杂度：
>
>如下图左边部分，我们可以知道BST的`添加`、`删除`和`搜索`效率都非常的高，其复杂度与元素的个数没有关系，只与树的高度有关系，即复杂度为：O(h) ，h为树的高度，当BST为满二叉树时，其复杂度为O(logn)，n为元素个数，此时：`O(h)  == O(logn)`

![image-20220328134437941](https://cdn.fengxianhub.top/resources-master/202203281344175.png)

>但是如果是按照从小到大的顺序添加结点，如上图右边所示，可以看到这样的BST与链表是一样的，其复杂度`O(h)  == O(n)`
>
>我们称这样的BST退化成了链表
>
>🚩以上两种BST的的`效率有巨大的差距`，当n = 1000000（一百万）时，左边的BST最坏情况下只需要进行`20次查找`，右边的BST最坏情况下需要进行`一百万次查找`

除了添加元素可能会让BST退化成链表之外，删除也有可能会让BST退化成链表，如下图所示，当树的高度足够大时，也面临着上面的问题。

![image-20220328193628291](https://cdn.fengxianhub.top/resources-master/202203281936472.png)

## 2. BST的平衡

>有什么办法能够解决上面的问题呢？当我们的二叉树更加`平衡`时，就可以解决上面的问题，所谓的`平衡`就是当节点数量固定时，左右子树的高度越接近，这棵二叉树就越平衡（高度越低），如下图所示

![image-20220328200730904](https://cdn.fengxianhub.top/resources-master/202203282007037.png)

>最理想的平衡，就是像完全二叉树、满二叉树那样，高度是最小的

![image-20220328201046761](https://cdn.fengxianhub.top/resources-master/202203282010863.png)

## 3. 改进二叉搜索树

首先我们需要知道：

- 首先，节点的添加、删除顺序是无法限制的，可以认为是随机的
- 所以，改进方案是：在节点的添加、删除操作之后，想办法让二叉搜索树恢复平衡（减小树的高度）

举个栗子，我们将下图中左边的BST调整为右边的BST：

![image-20220328201643226](https://cdn.fengxianhub.top/resources-master/202203282016356.png)

可以看到这样的调整让BST的高度减少了1，并且没有改变BST的性质，这就是一种`有效调整`

那右边的BST可以继续调整吗？其实可以继续调整的，但是没有必要，因为如果接着继续调整节点的位置，会做过多的运算，这样的话`付出的代价可能会比较大`

所以我们的做法是：`用尽量少的调整次数达到适度平衡即可`

**一棵达到适度平衡的二叉搜索树，可以称之为：平衡二叉搜索树**

## 4. 平衡二叉搜索树（Balanced Binary Search Tree）

我们将平衡二叉搜索树简称为`BBST`

经典的BBST有：

- **AVL树**（Windows NT 内核中广泛使用）
- **红黑树**（红黑树的应用十分广泛，例如C++ STL库中的`map`、`set`；Java 的 `TreeMap`、`TreeSet`、`HashMap`、`HashSet`；Linux 的进程调度；Nginx 的 timer 管理）

一般也称它们为：自平衡的二叉搜索树（Self-balancing Binary Search Tree）

## 5. AVL树

首先，AVL树是最早发明的自平衡二叉搜索树之一，AVL 取名于两位发明者的名字，G. M. `A`delson-`V`elsky 和 E. M. `L`andis（来自苏联的科学家）

因为读音酷似艾薇儿，所以有人把AVL树念做“艾薇儿树”

### 5.1 AVL树的相关概念及特点

**平衡因子**（Balance Factor）：某结点的左右子树的高度差，即左子树高度减去右子树高度

例如：下图中7这个结点左子树高度为2，右子树高度为4，所以其平衡因子为 2 - 4 = -2

![image-20220328203941373](https://cdn.fengxianhub.top/resources-master/202203282039483.png)

**AVL树的特点：**

- 每个节点的平衡因子只可能是 1、0、-1（绝对值 ≤ 1，如果超过 1，称之为“失衡”）
- 每个节点的左右子树高度差不超过 1
- 因为每个结点的高度差不超过1，所以AVL树搜索、添加、删除的时间复杂度是 O(logn)

我们看一棵AVL树（每个结点上都标记了平衡因子）：

![image-20220328204416187](https://cdn.fengxianhub.top/resources-master/202203282044295.png)

### 5.2 普通BST和AVL树添加对比

我们往一棵普通的BST和一棵AVL树中添加同一组结点：35, 37, 34, 56, 25, 62, 57, 9, 74, 32, 94, 80, 75, 100, 16, 82

我们可以得到普通BST的AVL的结构区别如下：

![image-20220328204650378](https://cdn.fengxianhub.top/resources-master/202203282046557.png)

## 6. AVL树设计

### 6.1 继承结构

首先我们设计一个简单的继承结构：

其中RBTree指的是红黑树

<img src="https://cdn.fengxianhub.top/resources-master/202203282108764.png" style="zoom:50%"/>

### 6.2 普通BST添加导致失衡例子

我们先来看一个BST因为添加结点而导致失衡的例子

我们往下面的BST中添加13这个元素（注意下面的BST并不完整，只是其中的一部分）

![image-20220328212403202](https://cdn.fengxianhub.top/resources-master/202203282124285.png)

可以看到在添加13这个元素前，图片里的树是平衡的，因为任意结点的平衡因子都小于1，但是当我们添加13这个结点后，这棵树就会变成以下这样：

![image-20220328212218085](https://cdn.fengxianhub.top/resources-master/202203282122271.png)

可以看到当添加了元素之后，有三个结点处于不平衡的状态了，并且对于整棵二叉树有：

- 最坏情况：可能会导致所有祖先节点都失衡
- 父节点、非祖先节点，都不可能失衡

### 6.3 解决添加失衡——LL-右旋转(单旋)

我们看一个例子：

在图中展示的二叉树里n表示node，p表示parent、g表示grandparent。这棵本来是平衡的（看下面的辅助线），但是因为`n`结点添加了一个元素，现在导致`g`结点现在不平衡了

因为`g`结点不平衡，且是因为`g`结点的左子树的左子树（LL）让其不平衡，所以我们称旋转的方式为：`LL-右旋转`

![image-20220328224015655](https://cdn.fengxianhub.top/resources-master/202203282144450.png)

我们为了让`g`结点平衡，可以采取如下措施：

```java
g.left = p.right
p.right = g
让p成为这棵子树的根节点
```

结果如下（右边的树是整理后的）：

![image-20220328224015655](https://cdn.fengxianhub.top/resources-master/202203282240716.png)

根据上面的调整我们得到：

- 调整后的二叉树仍然是一棵二叉搜索树：T0 < n < T1 < p < T2 < g < T3
- 整棵树都已经达到平衡

但是在调整的过程中我们还需要注意的是：

- T2、p、g 的 parent 属性需要更新
- 先后更新 g、p 的高度

>来一张动图展示一下

![](https://cdn.fengxianhub.top/resources-master/202204051755162.gif)

>这些动画都可以在<a href="https://myusf.usfca.edu/~galles/visualization/Algorithms.html">国外的一个网址</a>上，可以自行尝试，我这里提供一个将国外网站汉化版的地址，这里资源是借鉴了别人的，然后<a href="https://cdn.fengxianhub.top/eureka-static/Visualization/index.html">放在自己的服务上了</a>，侵删😖，点进去就能体验了

![image-20220406223050515](https://cdn.fengxianhub.top/resources-master/202204062230789.png)

### 6.4 解决添加失衡——RR-左旋转(单旋)

下面这种情况的失衡，由于失衡结点`g`的`右子树的右子树`（RR）增加了一个结点，所以我们需要让`g`左旋转来维持平衡

![image-20220329142724771](https://cdn.fengxianhub.top/resources-master/202203291427939.png)

具体做法为：

```java
g.right = p.left
p.left = g
让p成为这棵子树的根节点
```

![image-20220329143140699](https://cdn.fengxianhub.top/resources-master/202203291431767.png)

同理，根据上面的调整我们得到：

- 调整后的二叉树仍然是一棵二叉搜索树：T0 < n < T1 < p < T2 < g < T3
- 整棵树都已经达到平衡

但是在调整的过程中我们还需要注意的是：

- T1、p、g 的 parent 属性需要更新
- 先后更新 g、p 的高度

>来一张动图演示一下

![202204051759188](https://cdn.fengxianhub.top/resources-master/202204051759188.gif)

### 6.5 解决添加失衡——LR-RR左旋转，LL右旋转(双旋)

我们看下面失去平衡的例子，结点`g`的`左子树的右子树`（LR）增加了一个结点，从而使`g`失去了平衡

![image-20220329143442889](https://cdn.fengxianhub.top/resources-master/202203291434946.png)

这里我们采取的措施是先让结点`P`左旋转，即

```java
p.right = n.left
n.left = p
让n成为父结点
```

结果如下：

![image-20220329144853194](https://cdn.fengxianhub.top/resources-master/202203291448256.png)

现在又回到`g`左子树的左子树（T0结点）不平衡的情况了，这种我们需要`LL-右旋转`，这里对`g`进行右旋转

```java
g.left = n.right
n.right = g
让n成为根结点
```

可以看到最后我们的树变得`平衡`了

![image-20220329145338273](https://cdn.fengxianhub.top/resources-master/202203291453327.png)

>来一张动图演示一下

![doubleRotate](https://cdn.fengxianhub.top/resources-master/202204051821236.gif)

### 6.6 解决添加失衡——RL-LL右旋转，RR左旋转(双旋)

结点`g`的`右子树的左子树`（RL）添加结点后让`g`失去平衡

![image-20220329145827019](https://cdn.fengxianhub.top/resources-master/202203291458083.png)

我们需要先对`p`结点进行`LL-右旋转`

```java
p.left = n.right
n.right = p
让n变为根结点
```

![image-20220329150127812](https://cdn.fengxianhub.top/resources-master/202203291501875.png)

现在只需要将`g`进行`RR左旋转`即可让整棵二叉树恢复平衡

```java
g.right = n.left
n.left = g
让n成为根结点
```

![image-20220329150336132](https://cdn.fengxianhub.top/resources-master/202203291503182.png)

### 6.7 删除元素导致失衡

除了添加结点可能会导致失衡，删除结点也同样会导致树失去平衡，例如我们现在要删除下面的结点`16`

![image-20220408143240431](https://cdn.fengxianhub.top/resources-master/202204081432532.png)

我们可以看到结点`16`被删除后整个二叉树会变成下图中的情况，很显然结点`15`的平衡因子为2，失去了平衡：

![image-20220408143405423](https://cdn.fengxianhub.top/resources-master/202204081434513.png)

这里有：

- 可能会导致父节点或祖先节点失衡（只有1个节点会失衡）

>其实看到上面失衡的情况，我们可以快速的发现，这种失衡可以通过`LL-右旋转`来解决，这种不是和添加结点失衡一样吗？
>
>但真的是一样的吗？我们看下面将失衡结点进行右旋转的栗子

![image-20220408151143834](https://cdn.fengxianhub.top/resources-master/202204081511948.png)

看上去好像达到平衡的效果，但是**如果我们假设绿色的结点刚开始是不存在的**，也就是下图的这种情况

![image-20220408151628296](https://cdn.fengxianhub.top/resources-master/202204081516495.png)

我们会发现在右边的树虽然达到了平衡的效果，但是整体的高度减少了1**，整体高度减少了就有可能会导致其父结点失去平衡**

- 如果绿色节点不存在，更高层的祖先节点可能也会失衡，需要再次恢复平衡，然后又可能导致更高层的祖先节点失衡
- 极端情况下，所有祖先节点都需要进行恢复平衡的操作，共 O(logn) 次调整

>同样的，我们删除元素导致失衡也有LL、RR、LR-RR、RL-LL几种情况，和添加结点导致的失衡是一样的处理方式

## 7. 代码实现AVL树添加操作

### 7.1 平衡前的准备

由于这里的代码必须要基于二叉搜索树，所以可以看一下我的上一篇文章：<a href="https://blog.csdn.net/fengxiandada/article/details/123735085">java实现，二叉搜索树(过程非常详细)</a>，可以把文章最后的代码直接copy！

首先我们需要判断一下我们什么时候需要调整我们的BST，很显然，**我们需要在BST添加元素后进行判断**，如果不平衡我们就需要将其调整为平衡状态。

因为我们设计了继承关系来复用代码，所以这里我们的做法是在`AVL`的父类`BinarySearchTree`的`add`方法中添加一个判断是否需要调整的方法`afterAdd`，然后由子类`AVL`来进行实现，这样能够尽可能多的进行代码复用

```java
/**
 * 判断添加结点后是否需要平衡二叉树
 * @param node 新添加的结点
 */
protected void afterAdd(Node<E> node){
}
```

`BinarySearchTree`里的add方法

```java
...省略add方法部分代码
    //添加元素
    Node<E> newNode = new Node<>(element, parent);
    if (cmp > 0) {
        parent.right = newNode;
    } else {
        parent.left = newNode;
    }
    size++;
    //判断是否需要平衡这棵二叉树
    afterAdd(newNode);
}
```

>由上面的分析我们可以得知，失衡结点只可能是当前添加结点的父结点或祖父结点，所以我们需要先找`最靠近当前结点的失衡结点`（可能会导致多个祖父结点失衡，但是我们只需要找到最近的那一个，解决它的失衡问题，其他结点就都会平衡）

首先我们需要在`Node`结点中添加高度属性`height`，但是我们要注意不要在`BinaryTree`中添加，因为这个属性是`AVL`特有的，所以我们通过继承的方式添加我们的属性

```java
/**
 * AVL树结点
 * @param <E>
 */
private static class AVLNode<E> extends BinaryTree.Node<E>{
    int height;//AVL树有平衡因子的概念，需要有高度
    public AVLNode(E element, Node<E> parent) {
        super(element,parent);
    }
}
```

但是这里就会产生一个问题，我们添加结点的时候调用的是父类`BinarySearchTree`的`add`方法，这里添加的时候我们是直接添加`BinarySearchTree`里面的`Node`，所以这里我们需要在`BinarySearchTree`中在暴露一个添加结点的接口

```java
/**
 * 创建结点
 * @param element 结点中的元素
 * @param parent 当前结点的父结点
 * @return 返回创建好的结点
 */
protected Node<E> createNode(E element,Node<E> parent){
    return new Node<>(element,parent);
}
```

我们需要将`add`方法中添加结点的代码改成这个方法

然后让`AVLTree`去重写这个方法，这样添加的就是`AVLTree`的结点了

```java
/**
 * 创建AVL树结点
 * @param element 结点中的元素
 * @param parent 当前结点的父结点
 * @return 返回AVL树的结点
 */
@Override
protected Node<E> createNode(E element, Node<E> parent) {
    return new AVLNode<>(element,parent);
}
```

当然我们还需要在我们的`AVLNode`结点中定义一个获得`平衡因子`的方法：

```java
/**
 * AVL树结点
 * @param <E>
 */
private static class AVLNode<E> extends BinaryTree.Node<E>{
    int height = 1;//AVL树有平衡因子的概念，需要有高度，根据BST的性质我们知道传入的结点一定会变成叶子结点，高度为1
    public AVLNode(E element, Node<E> parent) {
        super(element,parent);
    }
    //获取当前结点的平衡因子，即左子树高度 减去 右子树高度
    public int balanceFactor(){
        int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
        int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
        return leftHeight - rightHeight;
    }
}
```

再写一个判断平衡的方法：

```java
/**
 * 判断当前结点是否平衡
 * @param node 传入需要判断的结点
 * @return 返回结点的平衡情况 <br> tree 平衡 <br> false 不平衡
 */
private boolean isBalanced(Node<E> node){
    //先将传入的结点转为AVLNode，在进行判断，结点的平衡因子的绝对值小于等于1表示平衡
    return Math.abs(((AVLNode<E>)node).balanceFactor()) <= 1;
}
```

在结点中写更新该结点高度的方法：

```java
/**
 * AVL树结点
 * @param <E>
 */
private static class AVLNode<E> extends BinaryTree.Node<E>{
    int height;//AVL树有平衡因子的概念，需要有高度
    public AVLNode(E element, Node<E> parent) {
        super(element,parent);
    }
    //获取当前结点的平衡因子，即左子树高度 减去 右子树高度
    public int balanceFactor(){
        int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
        int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
        return leftHeight - rightHeight;
    }
    //更新当前结点的高度
    public void updateHeight(Node<E> node){
        int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
        int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
        this.height = 1 + Math.max(leftHeight,rightHeight);
    }
}
```

>现在我们需要写我们的`afteradd`方法了，由上面的结论我们可以知道，不平衡的结点只可能在其祖父结点中出现，且我们只需要改变离该结点最近的不平衡的祖父结点，所以我们循环判断其所有父结点，这里有：
>
>- 该结点平衡，更新这个结点的高度
>- 不平衡，执行平衡逻辑
>
>这里可能有的地方需要好好推敲，比如为什么在结点中写更新结点高度的方法，其实是为了不去递归获得高度，因为我们在循环的过程中本身就已经遍历了所有的父结点，这么做都是为了尽可能的提升效率，可以在后面完整代码中反复推敲

```java
/**
 * 重写平衡BST的逻辑
 * @param node 新添加的结点，也就是导致失衡的结点
 */
@Override
protected void afterAdd(Node<E> node) {
    //在其祖父结点中寻到最近的失衡结点
    while ((node = node.parent) != null){
        //判断当前结点是否平衡
        if(isBalanced(node)){
            //平衡，更新该结点的高度
            updateHeight(node);
        }else {
            //不平衡，该结点为离添加结点最近的不平衡的结点
            reBalance(node);
            //该结点恢复平衡后整棵树也将恢复平衡，直接退出循环
            break;
        }
    }
}
```

### 7.2 恢复平衡

>接下来就是整棵`AVL`树的精华了——恢复平衡

首先我们要知道如果程序能够到这里，说明一定找到了不平衡的父结点，例如下图中的`g`结点

![image-20220328224015655](https://cdn.fengxianhub.top/resources-master/202203282144450.png)

先找我们要找到`p`结点和`n`结点，其中，p结点应该是g结点左右子树中较高的结点，如果高度相同，应该返回与该结点同一侧的结点

我们在AVL的父类BST的结点中增加两个判断左右子结点的方法

```java
public boolean isLeftChild(){
    return parent != null && this == parent.left;
}
public boolean isRightChild(){
    return parent != null && this == parent.right;
}
```

对不平衡的原因进行分类，我们就可以先得出一个大致的框架出来

```java
/**
 * 让当前AVL树恢复平衡
 *
 * @param grand 离添加结点最近的不平衡的结点
 */
private void reBalance(Node<E> grand) {
    //分别获得parent结点和node结点
    Node<E> parent = ((AVLNode<E>) grand).tallerChild();
    Node<E> node = ((AVLNode<E>) parent).tallerChild();
    //对不平衡的原因进行分类
    if (parent.isLeftChild()) { //L
        if (node.isLeftChild()) {  //LL
        } else {  //LR
        }
    } else {  //R
        if (node.isLeftChild()) {  //RL
        } else {  //RR
        }
    }
}
```

接下来我们要做的就是完善里面的代码

我们将之前解决失衡的结论回顾一下：

- LL——g右旋转
- RR——g左旋转
- LR——p左旋转，g右旋转
- RL——P右旋转，g左旋转

我们定义两个用来旋转结点的方法

```java
/**
 * 对该元素进行左旋转
 * @param node 待旋转的结点
 */
private void rotateLeft(Node<E> node){
}
/**
 * 对该元素进行右旋转
 * @param node 待旋转的结点
 */
private void rotateRight(Node<E> node){
}
```

在将分类好的`reBalance`方法用这两个旋转的方法进行填充

```java
//先判断LL右旋转的情况
if (parent.isLeftChild()) { //L
    if (node.isLeftChild()) {  //LL
        rotateRight(grand);
    } else {  //LR
        rotateLeft(parent);
        rotateRight(grand);
    }
} else {  //R
    if (node.isLeftChild()) {  //RL
        rotateRight(parent);
        rotateLeft(grand);
    } else {  //RR
        rotateLeft(grand);
    }
}
```

>最后就是完善`rotateLeft`左旋和`rotateRight`右旋的代码了

我们回顾一下第六点里面左旋和右旋的思路

![image-20220405171057108](https://cdn.fengxianhub.top/resources-master/202204051710222.png)

左旋：

```java
g.right = p.left
p.left = g
让p成为这棵子树的根节点
T1、p、g 的 parent 属性需要更新
先后更新 g、p 的高度
```

右旋：

```java
g.left = p.right
p.right = g;
让p成为这棵子树的根节点
T2、p、g 的 parent 属性需要更新
先后更新 g、p 的高度
```

接下来用代码来实现一下：

```java
/**
 * 对该元素进行左旋转
 * @param grand 待旋转的结点
 */
private void rotateLeft(Node<E> grand){
    //获得parent结点
    Node<E> parent = grand.right;
    //将parent的左子结点取出
    Node<E> leftChild = parent.left;
    grand.right = leftChild;
    parent.left = grand;
    /*
     * 让parent结点成为当前子树的根结点
     * 这里有两步：
     *  1. 让parent的父结点指向grand的父结点
     *  2. 让grand父结点本来指向grand的指针指向parent,这里顺便更新了parent结点的父结点
     **/
    parent.parent = grand.parent;
    if(grand.isLeftChild()){
        grand.parent.left = parent;
    }else if(grand.isRightChild()){
        grand.parent.right = parent;
    }else {
        //当前结点没有父结点，即grand结点就是root结点
        root = parent;
    }
    /*
     * 一共需要更新三个结点的parent，grand、parent和leftChild结点
     * grand结点在上面第二步中已经更新了，所以这里我们还需要更新parent结点和leftChild结点的parent结点
     **/
    if(leftChild != null){
        leftChild.parent = grand;
    }
    //更新grand的parent结点
    grand.parent = parent;
    //更新高度，先比较矮的结点再更新较高的结点
    updateHeight(grand);
    updateHeight(parent);
}
```

```java
/**
 * 对该元素进行右旋转
 * @param grand 待旋转的结点
 */
private void rotateRight(Node<E> grand){
    //获得parent结点,即grand结点的左结点
    Node<E> parent = grand.left;
    //获得parent结点的右子结点，方便后面更新高度
    Node<E> rightChild = parent.right;
    //右旋
    grand.left = parent.right;
    parent.right = grand;
    //让parent结点成为当前树的根结点
    parent.parent = grand.parent;
    if(grand.isLeftChild()){
        grand.parent.left = parent;
    }else if(grand.isRightChild()){
        grand.parent.right = parent;
    }else {
        //当前结点没有父结点，即grand结点就是root结点
        root = parent;
    }
    //更新parent结点和grand结点的父结点和grand结点左子树或右子树的指向
    if(rightChild != null){
        rightChild.parent = grand;
    }
    grand.parent = parent;
    //更新高度，先比较矮的结点再更新较高的结点
    updateHeight(grand);
    updateHeight(parent);
}
```

其实我们会发现这两段代码重复度很高，我们将共有代码提取为一个方法，旋转的完整代码为：

```java
/**
 * 对该元素进行左旋转
 * @param grand 待旋转的结点
 */
private void rotateLeft(Node<E> grand){
    //获得parent结点
    Node<E> parent = grand.right;
    //将parent的左子结点取出
    Node<E> leftChild = parent.left;
    //左旋
    grand.right = leftChild;
    parent.left = grand;
    //旋转之后让parent结点成为根结点并更新grand、parent、child结点的高度
    afterRotate(grand,parent,leftChild);
}
/**
 * 对该元素进行右旋转
 * @param grand 待旋转的结点
 */
private void rotateRight(Node<E> grand){
    //获得parent结点,即grand结点的左结点
    Node<E> parent = grand.left;
    //获得parent结点的右子结点，方便后面更新高度
    Node<E> rightChild = parent.right;
    //右旋
    grand.left = rightChild;
    parent.right = grand;
    //旋转之后让parent结点成为根结点并更新grand、parent、child结点的高度
    afterRotate(grand,parent,rightChild);
}
/**
 * 旋转之后让parent结点成为根结点并更新grand、parent、child结点的高度
 */
private void afterRotate(Node<E> grand,Node<E> parent,Node<E> child){
    /*
     * 让parent结点成为当前子树的根结点
     * 这里有两步：
     *  1. 让parent的父结点指向grand的父结点
     *  2. 让grand父结点本来指向grand的指针指向parent,这里顺便更新了parent结点的父结点
     **/
    parent.parent = grand.parent;
    if(grand.isLeftChild()){
        grand.parent.left = parent;
    }else if(grand.isRightChild()){
        grand.parent.right = parent;
    }else {
        //当前结点没有父结点，即grand结点就是root结点
        root = parent;
    }
    /*
     * 一共需要更新三个结点的parent，grand、parent和leftChild结点
     * grand结点在上面第二步中已经更新了，所以这里我们还需要更新parent结点和leftChild结点的parent结点
     **/
    if(child != null){
        child.parent = grand;
    }
    //更新grand的parent结点
    grand.parent = parent;
    //更新高度，先比较矮的结点再更新较高的结点
    updateHeight(grand);
    updateHeight(parent);
}
```

### 7.3 测试

我这里有一个可以将二叉树打印出来的封装好的工具包，我这里使用它进行测试

我们首先在<a href="https://520it.com/binarytrees/">下面的网站</a>上随机生成一些结点并构成AVL树

![image-20220408002415279](https://cdn.fengxianhub.top/resources-master/202204080024527.png)

然后一下测试代码，将随机生成的数字添加到自己的AVL树中

```java
@Test
public void test01() {
    Integer[] data = {
            17, 29, 40, 48, 97, 59, 65, 39, 25, 94, 43, 91, 37, 64, 13, 66, 53, 28, 34
    };
    AVLTree<Integer> avl = new AVLTree<>();
    for(int i : data){
        avl.add(i);
    }
    //这是一个可以将树打印出来的工具，代码有点多就不贴出来了，可以滴滴我私发
    BinaryTrees.println(avl);
}
```

输出结果：

![image-20220408002620243](https://cdn.fengxianhub.top/resources-master/202204080026424.png)

将每个结点的父结点和高度也一起输出出来，可以看到高度也是没有问题的：

![image-20220408004800602](https://cdn.fengxianhub.top/resources-master/202204080048724.png)

## 8. 添加操作的统一平衡处理

在上面的代码中，我们采取的方式是每添加一个结点都会进行平衡判断并更新结点的高度，我们其实还有一种做法，我们不需要去判断LL、RR或者双旋的情况，**我们可以用一份代码进行统一的平衡处理**

我们先回顾一下所有失去平衡的情况：

这里我将所有的结点从左往右按字母表进行标号，根据BST性质我们知道：g > f > e > d > c > b > a

![image-20220408102524684](https://cdn.fengxianhub.top/resources-master/202204081025844.png)

他们修复平衡后的状态为：

![image-20220408102632766](https://cdn.fengxianhub.top/resources-master/202204081026913.png)

>我们可以惊奇的发现，所有不平衡的情况，平衡之后的状态都是一样的！
>
>所以我们只需要找到待平衡的二叉树的对应结点即可

这里我们统一操作的步骤为：

```java
让d成为整棵子树的根结点
处理d的左子树，即a-b-c结点
处理d的右子树，即e-f-g结点
处理b-d-f结点
```

>我们在仔细观察一下会发现，其实结点`a`和结点`g`无论怎么选择其parent结点及其parent结点的指向都是没有变化的，所以我们在处理的时候可以不处理这两个结点，在红黑树中也不用处理这两个结点，但为了让代码更加直观，下面我实现的代码还是对这两个结点进行了处理

代码实现一下：

```java
/**
 * 进行统一旋转处理
 *
 * @param r 根结点
 */
private void rotate(Node<E> r,  //子树的根结点
                    Node<E> a,
                    Node<E> b,
                    Node<E> c,
                    Node<E> d,
                    Node<E> e,
                    Node<E> f,
                    Node<E> g) {
    //让d成为这棵树的根结点
    d.parent = r.parent;
    if (r.isLeftChild()) {
        r.parent.left = d;
    } else if (r.isRightChild()) {
        r.parent.right = d;
    } else {
        //r没有父结点
        root = d;
    }
    //处理d的左子树，即a-b-c结点
    b.right = c;
    if (c != null) {
        c.parent = b;
    }
    b.left = a;
    if (a != null) {
        a.parent = b;
    }
    //更新b结点的高度
    updateHeight(b);
    //处理d的右子树，即e-f-g结点
    f.right = g;
    if (g != null) {
        g.parent = f;
    }
    f.left = e;
    if (e != null) {
        e.parent = f;
    }
    //更新f结点的高度
    updateHeight(f);
    //现在处理b-d-f结点
    d.left = b;
    d.right = f;
    b.parent = d;
    f.parent = d;
    //d的左右子树结点高度都更新了，d的高度也需要更新
    updateHeight(d);
}
```

在让树恢复平衡的`reBalance`方法中进行统一平衡处理，可以对着上面的图将结点传入即可！

```java
/**
     * 让当前AVL树恢复平衡
     *
     * @param grand 离添加结点最近的不平衡的结点
     */
private void reBalance(Node<E> grand) {
    //分别获得parent结点和node结点
    Node<E> parent = ((AVLNode<E>) grand).tallerChild();
    Node<E> node = ((AVLNode<E>) parent).tallerChild();
    //先判断LL右旋转的情况
    if (parent.isLeftChild()) { //L
        if (node.isLeftChild()) {  //LL
            //                rotateRight(grand);
            rotate(grand,node.left,node,node.right,parent,parent.right,grand,grand.right);
        } else {  //LR
            //                rotateLeft(parent);
            //                rotateRight(grand);
            rotate(grand,parent.left,parent,node.left,node,node.right,grand,grand.right);
        }
    } else {  //R
        if (node.isLeftChild()) {  //RL
            //                rotateRight(parent);
            //                rotateLeft(grand);
            rotate(grand,grand.left,grand,node.left,node,node.right,parent,parent.right);
        } else {  //RR
            //                rotateLeft(grand);
            rotate(grand,grand.left,grand,parent.left,parent,node.left,node,node.right);
        }
    }
}
```

测试一下！

![image-20220408115518177](https://cdn.fengxianhub.top/resources-master/202204081155310.png)

## 9. 代码实现AVL树删除元素

我们在6.7中提到过，在删除元素导致的失衡中，不能让整棵树的高度减少，接下来我们用代码实现一下

跟添加结点时的操作一样，我们在AVL树的父类`BinarySearchTree`删除结点时也要添加一个方法

```java
/**
 * 删除结点时判断是否进行平衡
 * @param node 删除的结点
 */
protected void afterRemove(Node<E> node){
}
```

我们需要在`remove`方法中插入`afterRemove`方法进行删除元素后判断是否需要平衡BST

插入的位置很值得推敲，首先我们知道删除的结点可能是度为0、1、2的结点，但是删除度为2的结点实际上是删除其前驱或者后继结点，这样就需要判断一下我们到底要传入的元素是哪一个了，我们这里插入的地方是remove函数的最后面，等待删除的结点真正被删除之后，我们再来判断是否需要进行恢复平衡的操作

我们在AVLTree中重写这个方法，这里需要注意的是由于父结点也可能会失去平衡，所以所有的父结点都要进行判断

代码就很简单了，只是比上面添加判断少了一行`break`的代码，因为其父结点也可能会失去平衡

```java
/**
 * 删除元素进行平衡
 * @param node 删除的结点
 */
@Override
protected void afterRemove(Node<E> node) {
    //在其祖父结点中寻到最近的失衡结点
    while ((node = node.parent) != null) {
        //判断当前结点是否平衡
        if (isBalanced(node)) {
            //平衡，更新该结点的高度
            updateHeight(node);
        } else {
            //不平衡，该结点为离添加结点最近的不平衡的结点
            reBalance(node);
        }
    }
}
```

**测试一下**：

我们准备一棵AVL然后删除一些数据

![image-20220408162405602](https://cdn.fengxianhub.top/resources-master/202204081624745.png)



测试代码

```java
@Test
public void test01() {
    Integer[] data = {
            17, 29, 40, 48, 97, 59, 65
    };
    AVLTree<Integer> avl = new AVLTree<>();
    for(int i : data){
        avl.add(i);
    }
    avl.remove(97);
    avl.remove(59);
    avl.remove(65);
    BinaryTrees.println(avl);
}
```

输出结果：

![image-20220408162523684](https://cdn.fengxianhub.top/resources-master/202204081625843.png)

可以看到删除元素后也能恢复平衡

## 10. 总结

![image-20220408163155272](https://cdn.fengxianhub.top/resources-master/202204081631433.png)























