### 1 二维数组中的查找

> 在一个二维数组中（每个一维数组的长度相同)，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数.

**题目解析**:

- 这个题目比较好的解题思路是从**右上角或者左下角**开始找；这个是题目给定的每一行**从左到右递增**和每一列**从上到下递增**的原因；
- 例如，从右上角开始找，设置两个变量`row`，`col`分别代表行坐标和列坐标， 如果要找的数就是`target`，则直接返回；
- **如果`arr[row][col]` < target，那 row = row + 1，因为它左边的都会`arr[row][col]`小，这是因为列增加的性质**；
- **如果`arr[row][col]` > target，那 col = col - 1，因为它下面的都会`arr[row][col]`大，这是因为行增加的性质**；



![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/01_s.png)



**牛客通过代码**

```java
public class Solution {
    public boolean Find(int target, int [][] array) {    
        // r 表示 第一行 c 表示 第一行 最后一列，n 表示 总共多少列
        int r = 0,c=array[0].length-1,n=array.length-1;
        while(true){
            if(r>n||c<0){
                return false;
            }
            else if(array[r][c]==target){
                return true;
            }
            else if(array[r][c]<target){
                r++;
            }
            else if(array[r][c]>target){
                c--;
            }            
        }
    }
}
```

### 2.替换空格

> 请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。 

**问题背景**：

在接收网络字符流的时候，一般会进行字符转换。 输入的是一个 **字符缓冲流**`StringBuffer`。

**解法**：

因为替换的字符串长度不一样，数组插入耗时较久，所以不采取插入的方法。将 `StringBuffer`的长度设为扩展后的长度（原来的长度加上添加字符后的长度）。然后从后向前遍历，遇到空格，添加字符串

**牛客通过代码**

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	int length = str.length()-1,count=0;
        for(int i =0 ;i<=length;i++){
            if(str.charAt(i)==' ')
                count++;
        }
        int fi = length;
        int se = length+count*2;
        str.setLength(se+1);
        while( fi >=0){
            if(str.charAt(fi) ==' '){
                str.setCharAt(se--,'0');
                str.setCharAt(se--,'2');
                str.setCharAt(se--,'%');
            }else{
                str.setCharAt(se--,str.charAt(fi));
            }
            fi--;
        }
        
        return str.toString();
    }
}
```

### 3从尾到头打印链表

> 输入一个链表，按链表值从尾到头的顺序返回一个ArrayList



**解法：** 

- 先装入栈，在装入`Arraylist `

- 递归

  

**牛客通过代码**：

递归：

```java
import java.util.ArrayList;
public class Solution {
    ArrayList<Integer> arrayList=new ArrayList<Integer>();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode!=null){
            this.printListFromTailToHead(listNode.next);
            arrayList.add(listNode.val);
        }
        return arrayList;
    }
}  
```

使用栈:

```java
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        Stack<Integer> stack = new Stack();
        ListNode cur = listNode;
        while(cur!=null){
            stack.push(cur.val);
            cur = cur.next;
        }
         
        ArrayList<Integer> arr = new ArrayList();
        while(!stack.isEmpty()){
           arr.add(stack.pop());
        }
        return arr;
         
    }
}
```

### 4. 重建二叉树

> 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。 



**解析：**

树的问题 可以看作 是 递归问题

前序遍历序列 根 左 右

中序遍历序列 左 根 右

故前序的第一个根，在中序队列中找到根后，左边的为根的左子树，右边的为 根的右子树。

然后继续递归 查找。



**牛客通过代码：**

```java
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        /* 
        树的问题可以看作是 递归问题
        前序遍历序列  根 左 右
        中序遍历序列  左 根 右
        故前序的第一个为根，在中序队列中找到根后 左边的为 根的左子树，右边的为根的右子树
        */
        
        return build(pre,0,pre.length-1,in,0,in.length-1);
        
    }
    /*
    
    传入，前序序列，前序序列的 坐标，中序序列，中序序列的坐标
    */
    TreeNode build(int[] pre,int pl,int pr,int[] in,int il,int ir){
        // 如果 序列 左坐标 大于 右坐标  则返回 null
        if(pl>pr||il>ir){
            return null;
        }
        // 建立 根节点
        TreeNode root = new TreeNode(pre[pl]);
        // 寻找 左树长度
        int llen=0;
        while(in[il+llen]!=pre[pl]){
            llen++;
        }
        root.left = build(pre,pl+1,pl+llen,in,il,il+llen-1);
        root.right = build(pre,pl+llen+1,pr,in,il+llen+1,ir);
        return root;
    }
}
```



### 5.用两个栈实现队列

> 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。 



解析：

`push`的时候 ，加入第一个栈，`pop` 的时候，先查看 第二个栈是否有数，如果有数，就pop,

否则就 将 第一个栈 全部压入 第二个 栈



```java
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node); 
    }
    
    public int pop() {
        if(!stack2.isEmpty()){
            return stack2.pop();
        }
         while(!stack1.isEmpty()){
             int tmp = stack1.pop();
             stack2.push(tmp);
         }   
        return stack2.pop();
    }
}
```

 



### 6. 旋转数组的最小数字

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个**非减排序**的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。 



**解析**：

**非减排序的数组，将若干个元素搬到数组的末尾**： 意味 末尾的数组是 非减排序 ，如果非末尾元素`i` 大于 末尾元素，则 最小的数字 一定在 `i` 到末尾元素之间。如果`非末尾元素 `小于 末尾元素，则这个`非末尾元素` 一定在这些 若干元素中。。 所以 **末尾元素很重要**。 

采用 二分法解答这个问题

  *mid = low + (high - low)/2*

需要考虑三种情况：

-  **(1)array[mid] > array[high]:**

出现这种情况的array类似[3,4,5,6,0,1,2]，此时最小数字一定在mid的右边。

low = mid + 1

- **(2)array[mid] == array[high]:**

出现这种情况的array类似 [1,0,1,1,1] 或者[1,1,1,0,1]，此时最小数字不好判断在mid左边

还是右边,这时只好一个一个试 ，

high = high - 1

- **(3)array[mid] < array[high]:**

出现这种情况的array类似[2,2,3,4,5,6,6],此时最小数字一定就是array[mid]或者在mid的左

边。因为右边必然都是递增的。

high = mid

**注意这里有个坑：如果待查询的范围最后只剩两个数，那么mid** **一定会指向下标靠前的数字**

比如 array = [4,6]

array[low] = 4 ;array[mid] = 4 ; array[high] = 6 ;

如果high = mid - 1，就会产生错误， 因此high = mid

但情形(1)中low = mid + 1就不会错误



```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        if(array.length==0){
            return 0;
        }        
        int low = 0 ; int high = array.length - 1;   
        while(low < high){
            int mid = low + (high - low) / 2; 
            // 情况 1 
            if(array[mid] > array[high]){
                low = mid + 1;
             // 情况 2， 遍历
            }else if(array[mid] == array[high]){
                high = high - 1;
            }else{
                // 情况 3，
                high = mid;
            }   
        }
        return array[low];
    }
}
```



也可以 用 遍历的方法，查找 第一个小于 前面 的数，返回

```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        if(array.length==0){
            return 0;
        }         
        for(int i=0; i < array.length-1;i++){
            if(array[i+1]<array[i]){
                return array[i+1];
            }
        }
        return array[0];
    }
}
```



### 7 斐波那契数列

> 大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。
>
> n<=39

**解析**：

用动态规划求解，用两个数保存之前的值，因为当前值仅与之前的两个值相关

```java
public class Solution {
   
    public int Fibonacci(int n) {
        int f = 0, g = 1;
        while((n--)>0) {
            g += f;
            f = g - f;  //  c=f+g,f=g,g=c;  前三个式子 可以用g=c=f+g,f=g-f;表示
        }
        return f;
    }
}
```



### 8 跳台阶

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。 

**解析**：

当前台阶可以 由 跳一次 得到，也可以由跳二次得到。所以跳到 当前台阶 总共有f(n-1) +f(n-2) 次 。其中 f(1) =1,f(2)=2;

```java
public class Solution {
    
    public int JumpFloor(int target) {
          int f=1,g=2;
         if(target<=0)
             return 0;
        else if(target==1)
            return 1;
        else if(target ==2)
            return 2;
        target--;
          while(target-->0){
              g+=f;
              f=g-f;
         }
        
          return f;
    }
  
}
```

### 9 变态跳台阶

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。 

**解析**：

与之前相比的不同是，当前台阶可以由之前所有台阶得来

```java
public class Solution {
    int[] dp;
    public int JumpFloorII(int target) {
        dp = new  int[target+1];
        return rec(target); 
    }
    int rec(int n){
        if(n<1){
            return 0;
        }else if(n==1||n==2){
            return n;
        }
        if(dp[n]!=0){
            return dp[n];
        }
        for(int i =1;i<n;i++){
            dp[n]+=rec(i);
        }
        dp[n]+=1;
        return dp[n];     
    }    
}
```

### 10 矩形覆盖

> 我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？ 



**解析**：

与跳台阶相似，一个 2*2 的矩形 由 2 *  1 矩形组成 有两种方法。



```java
public class Solution {
    
    public int JumpFloor(int target) {
          int f=1,g=2;
         if(target<=0)
             return 0;
        else if(target==1)
            return 1;
        else if(target ==2)
            return 2;
          target--;
          while(target-->0){
              g+=f;
              f=g-f;
         }
        
          return f;
    }
  
}
```

## 11-20

### 11 二进制中1的个数

> 输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。 

**解析**： 

如果一个整数不为0，那么这个整数至少有一位是1。**如果我们把这个整数减1，那么原来处在整数最右边的1就会变为0，原来在1后面的所有的0都会变成1(如果最右边的1后面还有0的话)。其余所有位将不会受到影响。**

举个例子：一个二进制数1100，从右边数起第三位是处于最右边的一个1。减去1后，第三位变成0，它后面的两位0变成了1，而前面的1保持不变，因此得到的结果是1011.我们发现减1的结果是把最右边的一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做**与运算**，从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000.也就是说，把一个整数减去1，再和原整数做与运算，会把该整数最右边一个1变成0.那么一个整数的二进制有多少个1，就可以进行多少次这样的操作。

```java
public class Solution {
    public int NumberOf1(int n) {
        int count = 0;
        while(n!= 0){
            count++;
            n = n & (n - 1);
         }
        return count;
    }
}
```

### 12 数值的整数次方 

> 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。 

解析：

**少做几次乘法操作**，一个数值的整数次方 如 `a^b=a*a*a*a..*a` b个a相乘。也等于 

`n为偶数，a^n=a^n/``2``*a^n/``2``;n为奇数，a^n=（a^（n-``1``）/``2``）*（a^（n-``1``/``2``））*a`

时间复杂度 `O（logn）` 

**注意**：考虑所有情况

![](https://uploadfiles.nowcoder.com/files/20170906/3270776_1504711188579_E695B0E580BCE79A84E695B4E695B0E6ACA1E696B9.png)

![](https://uploadfiles.nowcoder.com/images/20170907/3270776_1504748011304_B16239181123036ECE974C89CC410F1F)

- 当底数为0且指数<0时 需进行错误处理
- 判断底数是否等于0，由于base 为double 型，不能直接用== 判断

```java
public class Solution { 
      boolean invalidInput=false;    
      public double Power(double base, int exponent) {     
        if(equal(base,0.0)&&exponent<0){
            invalidInput=true;
            return 0.0;
        }
        int absexponent=exponent;
        if(exponent<0)
            absexponent=-exponent;
        double res=getPower(base,absexponent);
        if(exponent<0)
            res=1.0/res;
        return res;
  }
    boolean equal(double num1,double num2){
        if(num1-num2>-0.000001&&num1-num2<0.000001)
            return true;
        else
            return false;
    }
    double getPower(double b,int e){
        if(e==0)
            return 1.0;
        if(e==1)
            return b;
        double result=getPower(b,e>>1);
        result*=result;
        if((e&1)==1)
            result*=b;
        return result;
    }
}

```

### 13 调整数组顺序使奇数位于偶数前面

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

**解法**：
- 要使相对顺序不变，则只交换相邻的奇偶数组。类似冒泡排序，遍历 i遍后，前 i 个顺序确定
- 再创建一个数组

第一种：
```java
public class Solution {
    public void reOrderArray(int [] array) {
        for (int i = 0; i < array.length;i++)
        {
            for (int j = array.length - 1; j>i;j--)
            {
                if (array[j] % 2 == 1 && array[j - 1]%2 == 0) //前偶后奇交换
                {
                    int tmp = array[j];
                    array[j]=array[j-1];
                    array[j-1] = tmp;
                }
            }
        }
    }
}
```

第二种
```java
public class Solution {
    public void reOrderArray(int [] array) {
        int count=0;
        int[] newarray = new int[array.length];
        for(int i=0;i<array.length;i++){
            if(array[i]%2==1)
                count++;
        }
        int fi=0;
        for(int i =0;i<array.length;i++){
            if(array[i]%2==1){
                newarray[fi++]=array[i];
            }
            else {
                newarray[count++]=array[i];
            }
        }
        System.arraycopy(newarray, 0, array, 0, array.length);
    }
}

```

### 14 链表中倒数第K个节点

> 输入一个链表，输出该链表中倒数第k个结点。

解析：
**快慢 指针**，一个快指针先走 k-1 步，然后两个指针一起走，当快指针走到结尾的时候，慢指针指向 倒数第K个节点。

```java
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        if (head == null || k == 0) //注意要特判
            return null;
        ListNode last = head,first=head;
        for(int i=0;i<k-1;i++){
           if(last == null) //k > len 特判
            return null;
            last = last.next;
        }
        while(last.next!=null){
             first=first.next;
             last=last.next;
        }
        return first;
    }
}
```

### 15 反转链表

> 输入一个链表，反转链表后，输出新链表的表头。

解析：

三个指针，一个指向 前节点，一个指向当前节点，一个指向后一个节点。

**反转节点**：

当前节点 的 next 指向 前一个节点，然后 前一个节点 指向当前节点，当前节点指向 下一个节点，

```java
public class Solution {
    public ListNode ReverseList(ListNode head) {
          ListNode pre = null,cur =head,next;
          while(cur!=null){
              next = cur.next;
              cur.next=pre;
              pre = cur;
              cur=next;
          }
        return pre;
    }
}
```

### 16 合并两个排序的链表

> 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

```java
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        if (list1 == null)
            return list2;
        if (list2 == null)
            return list1;
        ListNode head = new ListNode(-1),cur;
        cur = head;
        while(list1!=null&&list2!=null){
            if(list1.val>list2.val){
                cur.next=list2;
                list2=list2.next;
               
            }else{
                cur.next=list1;
                list1=list1.next;
               
            }
             cur=cur.next;
        }
        cur.next = (list1 == null)?list2:list1;
        return head.next;
 
    }
```

### 17 树的子结构

> 输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

解析：

递归

```java
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root2==null || root1==null){
            return false;
        }
        return aContainsB(root1,root2)||HasSubtree(root1.left,root2)||HasSubtree(root1.right,root2);
    }
    private boolean aContainsB(TreeNode A, TreeNode B) {
        if (B == null)  // B遍历完了, 说明可以
            return true;
        if (A == null)
            return false;
        // A != null && B != null 利用短路特性
        return A.val == B.val
                && aContainsB(A.left, B.left)
                && aContainsB(A.right, B.right);
    }
 }
```

### 18 二叉树的镜像

> 操作给定的二叉树，将其变换为源二叉树的镜像。

解析：
递归

```java
public class Solution {
    public void Mirror(TreeNode root) {
        if(root!=null){
            TreeNode tmp = root.left;
            root.left = root.right;
            root.right = tmp;
            Mirror(root.left);
            Mirror(root.right);
        }
    }
}
```

### 19 顺时针打印矩阵

> 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

解析：
- 使用矩阵分圈处理的方式，在矩阵中使用(ar,ac)表示左上角，(br,bc)表示矩阵的右下角；
- 每次只需要通过这四个变量打印一个矩阵，然后用一个宏观的函数来调用打印的局部的函数，这样调理更加清晰；


![f222f031e399e7ec929f11d05d78b77d.png](en-resource://database/2820:1)

代码:
```java
import java.util.ArrayList;
public class Solution {

    private ArrayList<Integer> res;

    public ArrayList<Integer> printMatrix(int[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0)
            return null;
        res = new ArrayList<>();
        int ar = 0, ac = 0, br = matrix.length - 1, bc = matrix[0].length - 1;
        while (ar <= br && ac <= bc)
            print(ar++, ac++, br--, bc--, matrix);
        return res;
    }

    private void print(int ar, int ac, int br, int bc, int[][] matrix) {
        if (ar == br)
            for (int j = ac; j <= bc; j++) res.add(matrix[ar][j]);
        else if (ac == bc)
            for (int i = ar; i <= br; i++) res.add(matrix[i][ac]);
        else {
            for (int j = ac; j < bc; j++) res.add(matrix[ar][j]);
            for (int i = ar; i < br; i++) res.add(matrix[i][bc]);
            for (int j = bc; j > ac; j--) res.add(matrix[br][j]);
            for (int i = br; i > ar; i--) res.add(matrix[i][ac]);
        }
    }
}
```

### 20 包含 min 函数的栈

解析：
用两个 栈，**一个 栈 从 栈底 到 栈顶 依次 减小**。

```java
import java.util.Stack;

public class Solution {
    Stack<Integer> stackdate = new Stack();
    Stack<Integer> stackmin =  new Stack();
    public void push(int node) {
        stackdate.push(node);
        //当 min 栈 栈顶 大于等于 push 进来的数字，则将 node 加入 min 栈
        if(stackmin.isEmpty()||stackmin.peek()>=node){
            stackmin.push(node);
        }
    }
    
    public void pop() {
        int tmp = stackdate.pop();
        if(tmp==stackmin.peek())
            stackmin.pop();
    }
    
    public int top() {
        return stackdate.peek();
    }
    
    public int min() {
        return stackmin.peek();
    }
}
```

### 21 栈的压入、弹出序列

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

**解析**：

借用一个辅助的栈，模拟进栈出栈 顺序。



```java
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        if(pushA.length == 0 || popA.length == 0)
            return false;
        Stack<Integer> s = new Stack<Integer>();
        //用于标识弹出序列的位置
        int popIndex = 0;
        for(int i = 0; i< pushA.length;i++){
            // 压入第一值
            s.push(pushA[i]);
            //如果栈不为空，且栈顶元素等于弹出序列
            while(!s.empty() &&s.peek() == popA[popIndex]){
                //出栈
                s.pop();
                //弹出序列向后一位
                popIndex++;
            }
        }
        return s.empty();
    }
}
```

### 22 从上往下打印二叉树

> 从上往下打印出二叉树的每个节点，同层节点从左至右打印。

解析：**层次遍历**

```java
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> list = new ArrayList();
        Queue<TreeNode> queue = new LinkedList<>();
        if (root == null)
            return list;
        queue.add(root);
        while(!queue.isEmpty()){
            TreeNode node = queue.poll();
            list.add(node.val);
            if(node.left!=null)
            queue.add(node.left);
            if(node.right!=null)
            queue.add(node.right);          
        }
            return list;
        }
    }

```

### 23 二叉搜索树的后序遍历序列

> 输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

解析：

二叉搜索树：

**左子树 < 根节点 < 右子树**

所以二叉搜索树的 后序序列 是 ：**对于一个序列S，最后一个元素是X，如果去掉最后一个元素的序列为T，那么T满足：T可以分成两段，前一段（左子树）小于x,后一段（右子树）大于x，且这两段（子树）都是合法的后序序列。**



用递归来做：

```java
链接：https://www.nowcoder.com/questionTerminal/a861533d45854474ac791d90e447bafd
来源：牛客网

public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
       // 如果 序列长度等于 0 ，则返回 false 
        if(sequence.length == 0){
            return false;
        }
        // 序列长度 等于 1  ，返回 true
        if(sequence.length == 1){
            return true;
        }
        
        return judge(sequence,0,sequence.length-1);
    }
     
    public boolean judge(int[] a,int start,int end){
    	// 如果 只剩 一个元素，则返回 true
        if(start >= end){
            return true;
        }
        // 找到 左子树的 最后一个节点 坐标
        int i = start;
        while(a[i] < a[end]){
            ++i;
        }
        // 判断右 子树 是否满足条件
        for(int j=i;j<end;j++){
            if(a[j] < a[end]){
                return false;
            }
        }
        // 分别判断 左 子树 和 右子树。
        return judge(a,start,i-1) && judge(a,i,end-1);
    }
}

```

### 24 二叉树中和为某一值的路径

> 输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到**叶结点**所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

解析：

深度优先遍历。

```java
public class Solution {
    
    private ArrayList<ArrayList<Integer>> listAll = new ArrayList<ArrayList<Integer>>();
    private ArrayList<Integer> list = new ArrayList<Integer>();
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
        // 如果 root==null ，则返回 
        if(root == null) return listAll;
        list.add(root.val);
        target -= root.val;
        // 如果符合条件，则加入到 list中。
        if(target == 0 && root.left == null && root.right == null)
            listAll.add(new ArrayList<Integer>(list));
        FindPath(root.left, target);
        FindPath(root.right, target);
        // 回退到上一个节点
        list.remove(list.size()-1);
        return listAll;
    }
}
```

### 25 复杂链表的复制

> 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

解析：

1. 复制每个节点，如：复制节点A得到A1，将A1插入节点A 后面
2. 遍历链表，A1->random = A->random->next ;
3. 将链表拆分成原链表和复制后的链表



```java
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
       if(pHead==null) return null;
        RandomListNode currNode = pHead;
        // 新建节点
        while(currNode!=null){
            RandomListNode node = new RandomListNode(currNode.label);
            node.next=currNode.next;
            currNode.next=node;
            currNode = node.next;
        }
        // 1. 遍历链表，A1->random = A->random->next ;
        currNode = pHead;
        while(currNode!=null){
            RandomListNode node  = currNode.next;
            if(currNode.random!=null){
                node.random=currNode.random.next;
            }
            currNode = node.next;
        }
        
        // 拆分
        RandomListNode pCloneHead = pHead.next;
        RandomListNode tmp;
        currNode = pHead;
    // 每个节点都指向 下下节点。
        while(currNode.next!=null){
            tmp=currNode.next;
            currNode.next=tmp.next;
            currNode=tmp;
        }
    return pCloneHead;
    }
}
```

另一种解法：

用 `HashMap` 保存，旧 节点--> 新节点 的映射

然后复制

```java
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        HashMap<RandomListNode,RandomListNode>  map = new HashMap();
        RandomListNode cur = pHead;
        while(cur!=null){
            map.put(cur,new RandomListNode(cur.label));
            cur = cur.next;
        }
        cur = pHead;
        while(cur != null){
            map.get(cur).next = map.get(cur.next);
            map.get(cur).random = map.get(cur.random);
            cur = cur.next;
        }
        return map.get(pHead);
    }
}
```

### 26 二叉搜索树与双向链表

> 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

解析：

二叉搜索树，中序遍历是 顺序序列。

将二叉搜索树转换成一个排序的双向链表，通过中序遍历还完成。

```java
	import java.util.Stack;
    public TreeNode ConvertBSTToBiList(TreeNode root) {
        if(root==null)
            return null;
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode p = root;
        TreeNode pre = null;// 用于保存中序遍历序列的上一节点
        boolean isFirst = true;
        while(p!=null||!stack.isEmpty()){
            while(p!=null){
                // 将 p 节点的 左节点都压进 栈中。
                stack.push(p);
                p = p.left;
            }
            p = stack.pop();
            if(isFirst){
                root = p;// 将中序遍历序列中的第一个节点记为root
                pre = root;
                isFirst = false;
            }else{
                pre.right = p;
                p.left = pre;
                pre = p;
            }      
            p = p.right;
        }
        return root;
    }
```

### 27 字符串的排列

> 输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

解析：

无重复值的全排列问题。

- 如果不存在 重复值：

1. 固定第一个字符，递归取得首位后面的各种字符串组合
2. 再把第一个字符与后面每一个字符交换，同样递归获得首位后面的字符串组合

- 存在重复值 

由于全排列就是从第一个数字起，每个数分别与它后面的数字交换，我们先尝试加个这样的判断——如果一个数与后面的数字相同那么这两个数就不交换了。
例如abb，第一个数与后面两个数交换得bab，bba。然后abb中第二个数和第三个数相同，就不用交换了。
是对bab，第二个数和第三个数不 同，则需要交换，得到bba。
由于这里的bba和开始第一个数与第三个数交换的结果相同了，因此这个方法不行。

**换种思维**，**对abb，第一个数a与第二个数b交换得到bab，然后考虑第一个数与第三个数交换，此时由于第三个数等于第二个数**
**所以第一个数就不再用与第三个数交换了。再考虑bab，它的第二个数与第三个数交换可以解决bba。此时全排列生成完毕！**

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashSet;
public class Solution {
    ArrayList<String> res;
    public ArrayList<String> Permutation(String str) {
        res = new ArrayList();
        if(str == null || str.length()==0)
            return res;
        permutation(str.toCharArray(),0);
        // 字典排序
        Collections.sort(res);
        return res;        
    }
    private void permutation( char[] num,int index){
        if(index == num.length-1){
            res.add(new String(num));
            return;
        }
        HashSet<Character> set =new HashSet();
        for(int i =index;i<num.length;i++){
            if(!set.contains(num[i])){
    		// set 保存 已经交换的过的  元素
            set.add(num[i]); 
            swap(num,i,index);
            permutation(num,index+1);
            swap(num,i,index);
            }
        }
    }
    private void swap(char[] num,int x ,int y){
        char tmp = num[x];
        num[x] = num[y];
        num[y] = tmp;
    }
}
```

### 28 数组中出现次数超过一半的数字

> 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

解析：

保存两个 值，一个值表示当前可能的 结果 `cur`，一个值计数`count`。

然后遍历，当 当前值等于`cur`时，`count++` ，否则`count--`。当`count==0` 时，`cur=` 当前的元素值。然后 `count=1` .

最后遍历查找该值是否超过一半。

```java
public class Solution {
    public int MoreThanHalfNum_Solution(int [] array) {
        if(array==null||array.length==0){
            return 0;
        }
        int cur=array[0],count=1;
        for(int i =1;i<array.length;i++){
             if(cur!=array[i]){
                count--;
                 if(count==0){
                     cur = array[i];
                     count++;
                 }
            }else{
                 count++;
             }
        }
        count=0;
        for(int i=0;i<array.length;i++){
            if(array[i]==cur)
                count++;
        }
        if(count*2>array.length)
            return cur;
        else
            return 0;
    }
}
```

### 29 最小的k个数

> 输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

解析：

利用快速排序来做，因为需要找 最小的k 个数，而这个 K 个数不需要排序。

快速排序：每次遍历 确定一个 数所在的位置`i`，在这个位置上，索引小于`i`的数小于 这个确定的 数，大于 `i` 的大于这个确定的数。故只要当 `i==(k-1)` 时，即可确定k个最小的数。但是不幸的是，快速排序中`i` 是随机的。最快 一次遍历就得到了最小的K个数。时间复杂度`O(n)` 。

```java
import java.util.ArrayList;
public class Solution {
    
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> res = new ArrayList<>();
        if (input == null || k <= 0 || k > input.length)
            return res;
        rec(input, 0, input.length - 1, k);
        for (int i = 0; i < k; i++)
            res.add(input[i]);
        return res;
    }
    
  // L = 序列左 坐标 R= 右 坐标
  private void rec(int[] arr, int L, int R, int k) {
        int border = partition(arr, L, R);
        if (k - 1 == border)//划分结束 可以返回退出了
            return;
        if (k - 1 < border) {
          // 如果 划分的边界 大于 k-1 
            rec(arr, L, border - 1, k);
        } else {
            rec(arr, border + 1, R, k);
        }
    }
    // 快速排序的一次遍历
    private  int partition(int[] input ,int l,int r){
        int temp=input[l],t,i=l;
        while(l!=r){
            while(input[r]>=temp&&r>l)
                r--;
            while(input[l]<=temp&&r>l)
                l++;
            if(r>l){
                t=input[r];
                input[r]=input[l];
                input[l]=t;
            }

        }
        input[i]=input[l];
        input[l]=temp;
        return l;
    }
}
```

### 30 连续子数组 的最大和

> HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)

解析：

使用动态规划，`F(i)`：以`array[i]` 为**末尾元素**的子数组的和的最大值，子数组的元素的相对位置不变。

`F(i) = max(F(i-1)+array[i],array[i])`

`res` : 所有子数组的和的最大值

`res = max(res,F(i)`)



```java
public class Solution {
    public int FindGreatestSumOfSubArray(int[] array) {
       // 先 创建一个 dp 的状态序列
        int[] dp = new int[array.length];
       // 表示最大值
        int max=0;
        // 一次遍历
        for(int i=0;i<array.length;i++){
            if(i==0){
                dp[i]=array[i];
                max = dp[i];
            }
            else{
                if((dp[i-1]+array[i])<array[i]){
                    dp[i]= array[i];
                }
                else{
                    dp[i] = dp[i-1]+array[i];
                }
                if(dp[i]>max)
                    max =dp[i];
            }
        }
        return max;
        
    }
}
```

### 31 整数中1出现的次数

> 求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

解析：

假如 n=534

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/assets/1555202832462.png)

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/assets/1555202889455.png)

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/assets/1555202908028.png)

**再举一个栗子: 统1～5246中1的个数** :

- 想到每一位和1的关系，拿5246，先是个位6，个位的变化范围是0~9，而这样的变化，会有524次，所以这里有524个1，又因为最后一次有个6，所以还要加一次，所以个位的1的个数是524+1 = 525；
- 再看十位，十位上的数字是4，所以同理，这个位数的上的1的个数应该是52 * 10，**注意这里不是52 \* 1，因为，10位上的数后面10-20之间有10个1，且最后4>1，所以还要加上10，所以十位上的1的个数是52 \* 10+10 = 530。这里要注意如果十位上的数字是1的话，就要看个位上的数是多少了，也就是10 ~ 20之间取多少个，这时候我们只要计算**`n%10+1`就行了。
- 然后同理推高位，可以得到1~5246中1的个数是`(524 * 1+1)+(52 * 10+10)+(5 * 100+100) +( 0 * 1000+1000) = 2655`个。

```java
public class Solution {
    public int NumberOf1Between1AndN_Solution(int n) {
        if (n <= 0) return 0;
        int res = 0;
        // base表示当前判断的位数、cur表示当前位、height表示高位
        int base = 1, cur, height = n;
        while (height > 0) {
            cur = height % 10;
            height /= 10;
            res += height * base; //先加上一开始的
            if (cur == 1)
                res += (n % base) + 1; //==1 就要看前面的了
            else if (cur > 1)
                res += base; //后面剩的，>1 还要+base
            base *= 10;
        }
        return res;
    }
}
```

### 32 丑数

> 把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。

解析：

为了保持丑数数组的顺序，可以维护三个队列 `q2`,`q3`,`q5` ，分别存放每次由上一个最小的没有用过的丑数乘以 2，3，5 得到的丑数：

过程：

1. 一开始第一个丑数为`1`，将 `1 * 2`放入`q2`，`1 * 3`放入`q3`，`1 * 5`放入`q5`；
2. 取三个队列中最小的为`2`，将`2 * 2 = 4`放入`q2`，`2 * 3 = 6`放入`q3`，`2 * 5 = 10`放入`q5`；
3. 取三个队列中最小的为`3`，将`3 * 2 = 6`放入`q2`，`3 * 3 = 9`放入`q3`，`3 * 5 = 15`放入`q5`；
4. 。。
5. 取三个队列中最小的为`6`，注意这里要将`q2、q3`都要弹出，因为`q2、q3`的对头都是`6`，然后。。。

然后我们只需要在丑数数组中取 `index-1` 个即可，由于只有一个索引，所以丑数数字可以用一个变量 `candi` 和一个索引 `count` 记录即可，代码如下：

```java
import java.util.*;

public class Solution {

    public int GetUglyNumber_Solution(int index) {
        if(index == 0)
            return 0;
        Queue<Integer> q2 = new LinkedList<>(), q3 = new LinkedList<>(), q5 = new LinkedList<>();
        int candi = 1, cnt = 1;
        while(cnt < index){ 
            q2.add(candi * 2); 
            q3.add(candi * 3);  
            q5.add(candi * 5);
            int min = Math.min(q2.peek(), Math.min(q3.peek(), q5.peek()));
            if(q2.peek() == min) q2.poll();
            if(q3.peek() == min) q3.poll();
            if(q5.peek() == min) q5.poll();
            candi = min;
            cnt++;
        }
        return candi;
    }
}
```

### 33 把数组排成最小的数

> 输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

解析：

这题关键是找到一个排序规则。然后数组根据这个规则就能排序成一个最小的数字。

也就是给出两个数字 a 和 b ，**我们需要确定一个规则判断a 和 b 哪个应该排在前面，而不是仅仅比较两个数字的值哪个更大。**

如果只看长度不行，但是只看数字大小也不行。 但是如果我们限定长度相同呢？

- 根据题目的要求，两个数字a和b 能拼接成数字`ab` 和 `ba ` 。如果`ab<ba` ，那么我们应该打印 ab,也就是 a 应该排在 b 的前面，我们定义此时 a 小于 b；`若 ab < ba,则 a<b`
- 反之，如果 `ba<ab` ，我们定义 b 小于 a。
- 如果 ab=ba,a等于b

如果直接用数字去拼接 nm，可能会溢出。但是我们可以直接用字符串拼接，然后定义字符串的比较贵重即可。

```java
import java.util.*;

public class Solution {
    public String PrintMinNumber(int[] numbers) {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < numbers.length; i++)
            list.add(numbers[i] + "");
        Collections.sort(list, (o1, o2) -> {
            return (o1 + o2).compareTo(o2 + o1);  //按照降序排列(第一个大于第二个返回1-->升序排列)
        });
        StringBuilder res = new StringBuilder();
        for (String s : list)
            res.append(s);
        return res.toString();
    }
}
```

### 34 第一个只出现一次的字符

> 在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到**第一个只出现一次的字符**，并返回它的位置， 如果没有则返回 -1（需要区分大小写）

解析：

因为 字符 `ascii`  在 `65~122`,所以开一个 58 的数组就可以了。

统计每个单词出现的次数，然后从头开始扫到第一个次数为`1` 返回就可以。

```java
public class Solution {
    public int FirstNotRepeatingChar(String str) {
        if (str == null || str.length() == 0) return -1;
        int[] counts = new int[58]; // 65('A') ~ 122 'z'
        for (int i = 0; i < str.length(); i++)
            counts[str.charAt(i) - 'A']++;
        for (int i = 0; i < str.length(); i++)
            if (counts[str.charAt(i) - 'A'] == 1)
                return i;
        return -1;
    }
}
```

### 35 数组中的逆序对

> 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007。

解析：

可以利用归并排序解决。

利用`归并排序` 的每次 `merge()` 过程：

- 因为 `meige()` 过程，处理的两个范围都有序的，即`[L, mid]`有序， `[mid+1, R]`有序；
- 我们可以在这里做手脚，当两部分出现`arr[p1] > arr[p2]`的时候，结果直接可以累加`mid - p1 + 1`个，这样就可以利用归并来降低复杂度；

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/35_s.png)

```java
public class Solution {
    private final int mod = 1000000007;
    public int InversePairs(int[] array) {
        if (array == null || array.length <= 1)
            return 0;
        return mergeRec(array, 0, array.length - 1);
    }

    public int mergeRec(int[] arr, int L, int R) {
        if (L == R)
            return 0;
        int mid = L + (R - L) / 2;
        return ((mergeRec(arr, L, mid) + mergeRec(arr, mid + 1, R))%mod + merge(arr, L, mid, R))%mod; // 正确
    }
    // [L, mid]有序， [mid+1, R]有序
    public int merge(int[] arr, int L, int mid, int R) {
        int[] help = new int[R - L + 1];
        int k = 0, sum = 0;
        int p1 = L, p2 = mid + 1;
        while (p1 <= mid && p2 <= R) {
            if (arr[p1] <= arr[p2]) {
                help[k++] = arr[p1++];
            } else {  //arr[p1] > arr[p2], 此时p2~R都小于arr[p1, mid]之间的元素，从这里求得逆序数
                sum += (mid - p1 + 1);
                sum %= mod;
                help[k++] = arr[p2++];
            }
        }
        while (p1 <= mid) help[k++] = arr[p1++];
        while (p2 <= R) help[k++] = arr[p2++];
        for (int i = 0; i < k; i++) arr[L + i] = help[i];
        return sum;
    }
}
```

### 36 两个链表的第一个公共节点

> 输入两个链表，找出它们的第一个公共结点。

解析：

- 第一种解法

使用 `Set`存储 `pHead` 中的所有节点，然后遍历 `pHead2`每一个节点，查看 `Set`中是否存放在该节点即可。时间复杂度O(N)，空间复杂度O(N)。



```java
import java.util.*;
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        HashSet<ListNode>set = new HashSet<>();
        for(ListNode cur = pHead1; cur != null; cur = cur.next)
            set.add(cur);
        for(ListNode cur = pHead2; cur != null; cur = cur.next)
            if(set.contains(cur))
                return cur;
        return null;
    }
}
```

 

- 第二种解法

O(N)时间，O(1)空间优化。

- 先求出两个链表的长度`n1、n2`；
- 记长度较长的链表为`p1`，较短的为`p2`；
- 先让`p1`走`abs(n1 - n2)`步；
- 然后让`p1、p2`一起走，当`p1 == p2`的时候，就是第一个公共节点；

```java
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        if (pHead1 == null || pHead2 == null) return null;
        int n1 = len(pHead1), n2 = len(pHead2);
        ListNode p1 = n1 > n2 ? pHead1 : pHead2;
        ListNode p2 = n1 > n2 ? pHead2 : pHead1;
        for(int i = 0; i < Math.abs(n1 - n2); i++) p1 = p1.next;
        for(; p1 != null && p1 != p2 ; p1 = p1.next, p2 = p2.next);
        return p1;
    }
    private int len(ListNode node) {
        int len = 0;
        for (ListNode p = node; p != null; p = p.next) len++;
        return len;
    }
}
```

### 37 数字在排序数组中出现的次数

> 统计一个数字在排序数组中出现的次数。

解析：

因为数组是排序的，所以O(N)的方法肯定不是好方法，所以想到二分。

其实思路很简单，我们只需要利用二分找到**第一个等于key的和最后一个等于key**的位置，就能得到key出现的次数。

而用二分得到这两个位置也很简单。

- 找到第一个 `=key` 的位置

数组中可能有重复 `key` ，我们要找到 的是第一个 `key的位置。`

和普通二分查找法不同的是在我们要`R=mid-1` 前的判断条件 不是 arr[mid]>key,而是 `arr[mid]>=key`

最后我们要判断`L` 是否越界(`L` 有可能等于`arr.length`),而且最后arr[L]是否等于 要找的 `key` 。

如果`arr[L]`不等于`key`，说明没有这个元素，返回`-1`；

- 最后一个 `=key` ，不存在 返回 `-1`。

和寻找第一个 `=key` 的很类似，不过是方向的不同而已：

数组中有可能有重复的`key`，我们要查找的是最后一个 `= key`的位置，不存在返回`-1`；

为了更加的直观的理解，和寻找第一个...的形成对比，这里是当`arr[mid] <= key`的时候，我们要去右边查找(`L = mid + 1`)，**同样是直观的理解，因为我们是要去找到最后一个 = key的，所以不仅仅是arr[mid] < key要去左边寻找，等于key的时候也要去左边寻找；**

同时我们也要判断`R`的下标的合法性，以及最后的`arr[R]`是否等于`key`，如果不等于就返回`-1`；

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/37_s.png)

```java
public class Solution {

    public int GetNumberOfK(int[] array, int k) {
        if (array.length == 0 || array == null)
            return 0;
        int L = firstEqual(array, k);
        int R = lastEqual(array, k);
        if (L != -1 && R != -1)
            return R - L + 1;
        return 0;
    }

    private int firstEqual(int[] arr, int key) {
        int L = 0, R = arr.length - 1; //在[L,R]查找第一个>=key的
        int mid;
        while (L <= R) {
            mid = L + (R - L) / 2;
            if (arr[mid] >= key)//因为要找第一个=的，所以也往左边
                R = mid - 1;
            else
                L = mid + 1;
        }
        if (L < arr.length && arr[L] == key)
            return L;
        return -1;
    }

    private int lastEqual(int[] arr, int key) {
        int L = 0, R = arr.length - 1;
        int mid;
        while (L <= R) {
            mid = L + (R - L) / 2;
            if (arr[mid] <= key)//因为要找最后一个，所以=往右边
                L = mid + 1;
            else
                R = mid - 1;
        }
        if (R >= 0 && arr[R] == key)
            return R;
        return -1;
    }
}

```

### 38 二叉树的深度

> 输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

解析：

1. 递归

递归的思路很简单。当前节点为根的树的高度 = 左右子树中高的那个 + 1 

```java
public class Solution {
    public int TreeDepth(TreeNode root) {
        if(root == null)
            return 0;
        return 1 + Math.max(TreeDepth(root.left), TreeDepth(root.right));
    }
}

```

2. 非递归

可以用层次遍历。来求树的层数(高度)。

- 每一个层的数量用一个变量 `count`统计，总的层数用`depth`统计；
- 同时，我们在当前层的时候，可以得知下一层的节点的数量(`queue.size()`)
- 然后再到了下一层的时候，就判断统计的数量 `count== nextlevelSize`,如果等于，就加一层 `depth++`；

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/38_s.png)

```java
import java.util.*;
public class Solution {
    public int TreeDepth(TreeNode root) {
        if(root == null)
            return 0;
        Queue<TreeNode>queue = new LinkedList<>();
        queue.add(root);
        int count = 0, nextLevelSize = 1;
        int depth = 0;
        while(!queue.isEmpty()){
            TreeNode cur = queue.poll();
            count++;
            if(cur.left != null) queue.add(cur.left);
            if(cur.right != null) queue.add(cur.right);
            if(count == nextLevelSize){
                count = 0;
                depth++;
                nextLevelSize = queue.size(); //下一层的节点的个数
            }
        }
        return depth;
    }
}

```

### 39 平衡二叉树

> 输入一棵二叉树，判断该二叉树是否是平衡二叉树。

解析：

1. 解法一

首先我们知道平衡二叉树是一颗空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一颗平衡二叉树。

我们可以使用一个获取树的高度的函数`depth()`。然后递归比较左右子树是不是平衡二叉树且左右子树的高度不超过`1`即可。

这里获取高度需要`logN`复杂度，主函数`isBalance`需要`O(N)`，所以总的时间复杂度为`N*logN`；

```java
public class Solution {
    
    public boolean IsBalanced_Solution(TreeNode root) {
        if(root == null)
            return true;
        return IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right) 
                && Math.abs(depth(root.left) - depth(root.right)) <= 1;
    }
    
    private int depth(TreeNode node) {
        if (node == null)
            return 0;
        return Math.max(depth(node.left), depth(node.right)) + 1;
    }
}

```

2. 解法二

上面的方法需要先求高度，然后再判断是不是平衡二叉树，能否一起做呢?

所以假如我们要判断一个以`x`开头的结点是否为一颗平衡二叉树，则要满足以下几点 :

- 它的左子树要给它一个左子树本身是不是平衡二叉树的信息；
- 它的右子树要给它一个右子树本身是不是平衡二叉树的信息；
- 它的左子树要给它左子树高度的信息；
- 它的右子树要给它右子树高度的信息；

所以我们知道上面的几点之后，可以完全按照上面写出一个递归结构函数，因为子树返回的信息中既包含高度信息，又包含子树是不是也是一颗平衡二叉树的信息，所以这里把这个信息封装成一个结构。

**这里和O(n\*logn)方法不同的是，我们求的是一个结构体，递归函数同时返回了两个信息: 高度和子树是否平衡，如果不平衡，我们就不再判断直接false了)。**

```java
public class Solution {

    public boolean IsBalanced_Solution(TreeNode root) {
        if (root == null)
            return true;
        return height(root) > -1;
    }

    private int height(TreeNode node) {
        if (node == null)
            return 0;
        int LH = height(node.left);
        int RH = height(node.right);
        if (Math.abs(LH - RH) > 1 || LH == -1 || RH == -1)
            return -1;
        return Math.max(LH, RH) + 1;
    }
}

```

### 40 数组中只出现一次的数字

> 一个整型数组里除了**两个数字**之外，其他的数字都出现了偶数次。请写程序找出这两个只出现一次的数字。

解析：

这题好方法是用 `位运算` 来做。

首先：

- 任何一个数异或自身等于` 0` ，任何一个数异或`0` 等于自身
- 所有在一个数组中，如果**只有一个数字**出现了一次，其他出现了两次，那么我们直接全部异或一遍，最终结果就是答案。

但是这一题的要求是有两个数字出现一次。**我们的做法是将原数组按照某种方式划分成两个子数组，使得每个子数组包含其中一个只出现一次的数字，而其他数字都成对出现两次，如何分别对两个子数组异或即可。**

**按照什么方式划分呢？**

我们还是从头到尾依次异或数组中的每一个数字，那么最终得到的结果就是两个只出现一次的数字的异或结果。

由于这两个数字肯定不一样，那么异或的结果肯定不为`0`，也就是说在这个结果数字的二进制表示中至少就有一位为 `1`。我们在结果数字中**找到第一个为 1 的位的位置**，记为第 `n` 位。现在我们以第 `n`位是不是 `1` 为标准把原数组中的数字分成两个子数组。

第一个子数组中每个数字的第`n` 位都是 `1`，而第二个子数组中每个数字的第 `n` 位都是 `0`。

> 由于我们分组的标准是数字中的某一位是 `1` 还是 `0`， 那么出现了两次的数字肯定被分配到同一个子数组。因为两个相同的数字的任意一位都是相同的，我们不可能把两个相同的数字分配到两个子数组中去，于是我们已经把原数组分成了两个子数组，每个子数组都包含一个只出现一次的数字，而其他数字都出现了两次。

过程：

- 每一个数字做异或运算之后，得到的结果用二进制表示是 `2 `(二进制0010)；
- 根据数字的倒数第二位是不是 1 分为两个数组。第一个子数组`{2, 3, 6, 3, 2}`、第二个子数组`{4, 5, 5}`；
- 分别对这两个子数组求异或，第一个子数组中只出现一次的数字是`6`，而第二个子数组中只出现一次的数字是 `4`。

代码

```java
public class Solution {
    // num1[0], num2[0]是返回的两个只出现一次的数
    public void FindNumsAppearOnce(int[] array, int num1[], int num2[]) {
        if (array.length < 2)
            return;
        int res = 0;
        for (int i = 0; i < array.length; i++)
            res ^= array[i];
        int digit = 1;
        while ((digit & res) == 0) digit *= 2; // digit的最高位为1
        for (int i = 0; i < array.length; i++) {
            if ((array[i] & digit) == 0)  //第一组 (第一个子数组)
                num1[0] ^= array[i];
            else                          //第二组 (第二个子数组)
                num2[0] ^= array[i];
        }
    }
}

```

## 41-50

### 41 和为S的连续正数序列

> 小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!

解析：

双指针思路：

- 两个指针 `small` 和 `big` 分别表示序列(窗口)的最小值和最大值。
- 首先把`small` 初始化为`1` ，`big` 初始化为2。
- 如果从`small`到 `big` 的序列的和大于 `s`，我们可以从序列中去掉较小的值，也就是增大 `small `的值。
- 如果从 `small` 到 `big` 的序列的和小于 `s`，我们可以增大`big`，让这个序列包含更多的数字。

**因为这个序列至少要有两个数字，我们一直增加 small 到 (1+s)/2 为止**。

> 过程：
>
> 先把 small 初始化为 1，big 初始化为2。此时介于 small 和 big 之间的序列是`{1， 2}`，序列的和为 3，小于9，记以我们下一步要让序列包含更多的数字。我们把 big 增加 1 变成 3，此时序列为`{1，2,， 3}`。由于序列的和是 6，仍然小于 9，我们接下来  再增加big 变成4，介于 small 和 big 之间的序列也随之变成`{1, 2, 3, 4}`。由于序列的和 10 大于 9，我们要删去去序列中的一些数字，于是我们增加 small 变成2，此时得到的序列是`{2, 3, 4}`，序列的和正好是 9。我们找到了第一个和为 9 的连续序列，把它打印出来。接下来我们再增加 big，重复前面的过程， 可以找到第二个和为 9 的连续序列`{4, 5}`。

代码：

```java
import java.util.ArrayList;

public class Solution {

    private ArrayList<ArrayList<Integer>> res;

    public ArrayList<ArrayList<Integer>> FindContinuousSequence(int sum) {
        res = new ArrayList<>();
        int small = 1, big = 2;
        int mid = (sum + 1) / 2;
        int curSum = small + big;
        while (small < mid) {                                  
           if (curSum == sum)
                packing(small, big);
            while (curSum > sum && small < mid) {//还要去找别的可能的连续序列
                curSum -= small;
                small++;
                if (curSum == sum) packing(small, big);
            }
            big++;
            curSum += big;
        }
        return res;
    }

    private void packing(int small, int big) {
        ArrayList<Integer> tmp = new ArrayList<>();
        for (int i = small; i <= big; i++)
            tmp.add(i);
        res.add(tmp);
    }
}

```

解法二：

结果序列是一个个 等差为1 的数列，而这个序列的中间值代表了平均值的大小。假设序列长度为n，那么这个序列的中间值可以通过 `S/n`得到。

- 满足条件的 n 分为两种情况
  - n 为奇数的时候，序列中间的数正好是序列的平均值，所以条件为 ：`n%2==1&& sum%n==0` 。 其中 `n%2==1` 表示 n 为奇数，`sum%n==0`表示中间值为正数。
  - n 为偶数时，序列中间两个数的平均值是序列的平均值，应该满足中间两个数的平均值的小数部分是 `0.5` ，所以条件为 `(sum % n) *2==n`
- 我们需要确定` n` 的遍历范围。根据等差数列的求和公式：`S=(1+n)*n/2`，得到$n<\sqrt{2S}$,我们从这个数从大到小遍历即可。



```java
import java.util.ArrayList;

public class Solution {

    public ArrayList<ArrayList<Integer>> FindContinuousSequence(int sum) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        for(int n = (int) Math.sqrt(2 * sum); n >= 2; n--){
//            if( (n%2 == 1 && sum%n == 0) || (n%2 == 0 && (sum%n)*2 == n)) { // 也可以
            if( (n%2 == 1 && sum%n == 0) || (n%2 == 0 && (sum/n + sum/n+1)*n/2 == sum)) {// 奇数的情况 | 偶数的情况
                ArrayList<Integer> tmp = new ArrayList<>();
                int start = sum/n-(n-1)/2;
                for(int k = start; k < start + n; k++) tmp.add(k);
                res.add(tmp);
            }
        }
        return res;
    }

    public static void main(String[] args){
        System.out.println(new Solution().FindContinuousSequence(100));
    }
}
```

### 42 和为S的两个数字(排序数组)

> 输入一个**递增排序**的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，**输出两个数的乘积最小的**。

解析：

思路

- 设两个指针`L、R`，分别是排序数组的开头和结尾；
- 然后下面就是两个指针`L、R`向中间靠拢的过程。① 如果`arr[L] + arr[R] > sum`，说明右边那个`arr[R]`大了，需要向左移动，看能不能找到更小的`arr[R]`来和`arr[L]`一起组成`sum`。② 同理，如果`arr[L] + arr[R] < sum`，说明左边那个`arr[L]`小了，需要向右移动，看能不能找到更大的`arr[L]`来和`arr[R]`一起组成`sum`。③否则等于就返回即可；
- 题目说要找到乘积最小的，可以发现，`L、R`隔的越远，`arr[L] * arr[R]`乘积越小，所以我们的做法没问题。

```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int[] array, int sum) {
        ArrayList<Integer> res = new ArrayList<>();
        int L = 0, R = array.length - 1;
        while(L < R){
            if(array[L] + array[R] == sum){
                res.add(array[L]);
                res.add(array[R]);
                return res;
            }
            if(array[L] + array[R] < sum) L++;
            else R--;
        }
        return res;
    }
}
```

### 43 左旋转字符串

> 汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列`S=”abcXYZdef”`,要求输出**循环左移3位**后的结果，即`“XYZdefabc”`。是不是很简单？OK，搞定它！

解析：

剑指Offer的解法:

- 将字符串分成两部分，第一部分记为前n个字符部分记为`A`，后面的部分记为`B`；
- 其实这个题目就是要你从`AB`转换到`BA`；
- 做法就是 (1)、先将A部分字符串翻转；(2)、然后将B字符串翻转；(3)、最后将整个字符串翻转；
- 也就是(ATBT)T = BA；

```java
ublic class Solution {
    public void reverse(char[] chs, int L, int R) {
        for (; L < R; L++, R--) {
            char c = chs[L];
            chs[L] = chs[R];
            chs[R] = c;
        }
    }
    public String LeftRotateString(String str, int n) {
        if (str == null || str.length() == 0) return "";
        if (n == str.length() || n == 0)
            return str;
        char[] chs = str.toCharArray();
        reverse(chs, 0, n - 1);
        reverse(chs, n, str.length() - 1);
        reverse(chs, 0, str.length() - 1);
        return new String(chs);
    }
}
```

### 44 反转单词顺序

> 牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，有一天他向Fish借来翻看，但却读不懂它的意思。例如，`“student. a am I”`。后来才意识到，这家伙原来把句子单词的顺序翻转了，正确的句子应该是`“I am a student.”`。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？
>
> 即:
>
> 输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串`"I am a student. "`，则输出`"student. a am I"`。

解析：

1. 方法一

直接从后面开始构造结果字符串即可，中间加上`" "`即可。很简单。

```java
public class Solution {
    public String ReverseSentence(String str) {
        if(str.trim().equals("")) return str;// 注意"    "这种空格多的情况
        StringBuilder sb = new StringBuilder();
        String[] strings = str.split(" ");
        for(int i = strings.length - 1; i > 0; i--)
            sb.append(strings[i]).append(" ");
        sb.append(strings[0]);
        return sb.toString();
    }
}
```

2.方法二

也很简单:

- 就是先把整个字符串先翻转一下，例如`"I am a student. "`翻转成`".tneduts a ma I"`；
- 然后再翻转每个单词中字符的顺序即可。
- 翻转某个字符的某个区间写成一个函数`reverse()`即可；

```java
public class Solution {
    public String ReverseSentence(String str) {
        if(str.trim().equals("")) return str;
        int n = str.length();
        char[] chs = str.toCharArray();
        // 1、先翻转整个字符串
        reverse(chs, 0, n-1);

        // 2、然后翻转其中的每一个单词
        for(int i = 0; i < n; ){
            while(i < n && chs[i] == ' ')i++; // 跳过空格
            int L = i, R = i;
            for(; i < n && chs[i] != ' '; i++, R++); // chs[R] = ' '
            reverse(chs, L, R-1); // notice is R - 1
        }
        return new String(chs);
    }
    // 翻转chs在[L, R]范围内的字符
    private void reverse(char[] chs, int L, int R){
        for(; L < R; L++, R--){
            char c = chs[L];
            chs[L] = chs[R];
            chs[R] = c;
        }
    }
}
```

### 45 扑克牌顺序

> 从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2~10为数字本身，A为1，J为11，Q为12，K为13，而大、小王可以看成任意数字。
>
> 如果牌能组成顺子就输出true，否则就输出false。**为了方便起见,你可以认为大小王是0。**

#### 1、思路一

**先把数组排序**。

由于 0 可以当成任意数字，我们可以用 0 去补满数组中的空缺。如果排序之后的数组不是连续的，即相邻的两个数字相隔若干个数字，但只要我们有足够的 0 可以补满这两个数字的空缺，这个数组实际上还是连续的。

举个例子，数组排序之后为`{0，1，3，4，5}`，在1和3之间空缺了一个 2，刚好我们有一个 0，也就是我们可以把它当成 2去填补这个空缺 。

分析了上面的问题之后，我们就可以整理出解题步骤:

- 首先把数组排序，再统计数组中 0 的个数，最后统计排序之后的数组中相邻数字之间的空缺总数；
- 如果空缺的总数小于或者等于 0 的个数，那么这个数组就是连续的，反之则不连续；
- 还需要注意: 如果数组中的**非 0 数字**重复出现，则该数组不是连续的。即如果一副牌里含有对子，则不可能是顺子；

```java
import java.util.Arrays;
public class Solution {
    public boolean isContinuous(int[] numbers) {
        if (numbers.length != 5)
            return false;
        Arrays.sort(numbers);
        int interval = 0, zero = 0;
        for (int i = 0; i < 4; i++) {
            if (numbers[i] == 0) {//写在循环前面, 且numbers[4]不需要判断了
                zero++;
                continue;// 记得不要判断下面的对子了，因为number[i+1]有可能也是0
            }
            if (numbers[i] == numbers[i + 1])
                return false;
            interval += numbers[i + 1] - numbers[i] - 1;
        }
        return zero >= interval;
    }
}
```

#### 2.思路二

思路二的判断点:

- 最大的牌到最小的牌的距离要小于`5`。
- 也是不能有重复的牌 (除`0`外)；

因为我们没有对数组排序，**这里用位操作来判断数组中是否含有重复的数字**。

```java
public class Solution {
    public boolean isContinuous(int[] numbers) {
        if (numbers.length != 5)
            return false;
        int max = -1, min = 14;
        int bit = 0; // 用bit判断是否相同
        for (int num : numbers) {
            if (num > 13 || num < 0) return false;
            if (num == 0) continue;
            if (((bit >> num) & 1) != 0) return false;
            bit |= (1 << num);
            if(num > max) max = num;
            if(num < min) min = num;
            if(max - min >= 5) return false;
        }
        return true;
    }
}
```

### 46 孩子们的游戏(圆圈中最后剩下的数)

> 每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)

解析：

利用取模

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/46_s4.png)

```java
import java.util.ArrayList;
public class Solution {
    public int LastRemaining_Solution(int n, int m) {
        if(n == 0 || m == 0)
            return -1;
        // 
        ArrayList<Integer> list = new ArrayList<>();
        for(int i = 0; i < n; i++) list.add(i);
        int pos = -1;
        while(list.size() > 1){
            pos = (pos + m) % list.size();
            list.remove(pos);
            pos--;
        }
        return list.get(0);
    }
}
```

### 47 求1+2+3+...+n

> 求`1+2+3+...+n`，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

解析

思路: **利用逻辑与的短路特性实现递归终止**。

- 当`n == 0`时，`n > 0 && (res += Sum_Solution(n-1)) > 0`只执行前面的判断，为`false`，然后直接返回`0`；
- 当`n > 0`时，递归`res += Sum_Solution(n-1)`，实现递归计算；

代码:

```java
public class Solution {
    public int Sum_Solution(int n) {
        int res = n;
        boolean b = n > 0 && (res += Sum_Solution(n-1)) > 0;
        return res;
    }
}
```

### 48不用加减乘除做加法

> 写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

解析：

如果不考虑进位的情况下，`a^b` 就是正确结果。因为`0 + 0 = 0(0 ^ 0)`，`1 + 0 = 1(1 ^ 0)`，`0 + 1 = 1(0 ^ 1)`，`1 + 1 = 0 (1 ^ 1)`。

例如：

```java
a : 001010101
b : 000101111
    001111010

```

只考虑进位的情况下，也就是只考虑 `a+b` 中进位产生的值的情况下，`(a & b) << 1`就是结果。因为第`i`位的结果只有可能是`i - 1`位都为`1`的情况下才会产生进位。

例如：

```java
a : 001010101
b : 000101111
    000001010 // 这是第一次 产生进位的情况

```

把完全不考虑进位和考虑进位的两种情况相加，就是最终的结果。也就是说一直重复这样的过程，直到最后的进位为 `0` ，则说明完成了相加。

例如：

```java
1、一开始的值:
a    :   001010101
b    :   000101111
    
2、上面两个异或和&<<1的值:
^    :   001111010
&<<1 :   000001010
    
3、上面两个异或和&<<1的值:
^    :   001110000
&<<1 :   000010100
    
4、上面两个异或和&<<1的值:
^    :   001100100
&<<1 :   000100000
    
5、上面两个异或和&<<1的值:
^    :   001000100
&<<1 :   001000000
    
6、上面两个异或和&<<1的值:
^    :   000000100
&<<1 :   010000000
    
7、上面两个异或和&<<1的值:
^    :   010000100
&<<1 :   000000000    (num2 == 0)
```

```java
public class Solution {
    public int Add(int num1, int num2) {
        int sum = num1, carry = 0;//一开始sum = num1的原因是如果num2 == 0,后面我直接返回sum，而不是num1
        while(num2 != 0){
            sum = num1 ^ num2;
            carry = (num1 & num2) << 1;
            num1 = sum;
            num2 = carry;
        }
        return sum;
    }
}
```

### 49 把 字符串转换成整数

> 将一个字符串转换成一个整数(实现Integer.valueOf(string)的功能，但是string不符合数字要求时返回0)，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。
>
> 实例：
>
> 输入
>
> > +2147483647
> >  1a33

> 
>
> 输出
>
> > ```
> > 2147483647
> >  0
> > ```

解析：

比较简单的模拟题。但是也要注意一些情况:

- 前面的空字符要去掉；
- 然后就是第一个如果是符号要判断；
- 还有一个就是判断溢出，这个问题可能有点麻烦，方式挺多，但是很多都有小问题；

```java
public class Solution {

    public int StrToInt(String str) {
        if (str == null || str.trim().equals(""))
            return 0;
        char[] chs = str.trim().toCharArray();//去除前面的空字符' '
        int res = 0;
        for (int i = (chs[0] == '-' || chs[0] == '+') ? 1 : 0; i < str.length(); i++) {
            if(chs[i] < '0' || chs[i] > '9') return 0; // < 48 || > 57
            int num = chs[i] - '0'; // chs[i] - 48
            int sum = res * 10 + num;
            if((sum - num)/10 != res) // 如果 sum超出范围了，这个表达式就回不到原来的res
                return 0;
            res = sum;
        }
        return chs[0] == '-' ? -res : res;
    }
}
```

## 51-60

### 50 数组中的重复数字

> **在一个长度为n的数组里的所有数字都在`0`到`n-1`的范围内**。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组`{2,3,1,0,2,5,3}`，那么对应的输出是**第一个重复**的数字`2`。

解析：

1. 直接用数组保存哪个数之前有没有出现过即可。因为 数在 `0~n-1` 之间，所以数组只需要开 n 即可。 空间复杂度是`O（N）` 

   ```java
   public class Solution {
       public boolean duplicate(int numbers[], int length, int[] duplication) {
           if(numbers == null || length == 0) return false;
           boolean[] used = new boolean[length];
           for(int num : numbers){
               if(used[num]){
                   duplication[0] = num;
                   return true;
               }
               used[num] = true;
           }
           return false;
       }
   }
   ```

   

2. 充分利用数只出现在`0 ~ n-1`之间，所以我们每次将`arr[ abs(arr[i])] `标记成**它的相反数**；下次，如果再发现一个`arr[i]` ，且`arr[ abs(arr[i])] < 0`，说明之前已经标记过了，所以可以返回`arr[i]`；

   ```java
   public class Solution {
       public boolean duplicate(int numbers[], int length, int[] duplication) {
           if(numbers == null || length == 0) return false;
           for(int i = 0; i < length; i++){
               int idx = Math.abs(numbers[i]);
               if(numbers[idx] >= 0)
                   numbers[idx] = -numbers[idx];
               else {
                   duplication[0] = idx;
                   return true;
               }
           }
           return false;
       }
   }
   ```



### 51 构建乘积数组

> 给定一个数组`A[0,1,...,n-1]`，请构建一个数组`B[0,1,...,n-1]`，其中B中的元素`B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]`。**不能使用除法**。

解析：

好的方法是**分别从两边开始乘**。

- 一开始从左往右累乘到`B[i]`，但是不要包括`A[i]` (也就是`A[0 ~ i-1]`)；
- 第二次从后往前累乘到`B[i]`，也不要包括`A[i]`(也就是`A[i+1 ~ n-1]`)；

看个例子：

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/51_s.png)

```java
public class Solution {
    public int[] multiply(int[] A) {
        int n = A.length;
        int[] B = new int[n];
        // 乘数
        int mul = 1;
        for (int i = 0; i < n; i++) {
            B[i] = mul;//先 =
            mul *= A[i];
        }
        mul = 1;
        for (int i = n - 1; i >= 0; i--) {
            B[i] *= mul;//先 *
            mul *= A[i];
        }
        return B;
    }
}
```

### 52 正则表达式匹配

> 请实现一个函数用来匹配包括`'.'`和`'*'`的正则表达式。模式中的字符`'.'`表示任意一个字符，而`'*'`表示它**前面的字符**可以出现任意次（**包含0次**）。
>
> 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串`"aaa"`与模式`"a.a"`和`"ab*ac*a"`匹配，但是与`"aa.a"`和`"ab*a"`均不匹配。

解析：

这种题目一般都会用动态规划。

递归的思路如下图，大致分三种情况

我们的`sl`和`pl`分别表示当前判断的两个串的长度。而对应的索引是`s[sl - 1]`和`p[pl-1]`。

- `s[sl - 1] = p[pl-1]`
- `p[pl - 1] = '.'`
- `p[pl - 1] = '*'`；这种情况具体又可以分几种情况，具体看下图。

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/52_s.png)

最后还要注意边界，当`s == "" && p == ""`的时候返回`true`，当`p=="" & s!= ""`的时候返回`false`。

当`s == "" && p != ""`的时候就要注意，如果`p[i-1] == '*'`则`dp[0][i] = dp[0][i-2]`，因为可以用`*`可以消去前一个字符。

虽然第三种情况，可以合起来考虑，代码会更简洁一些，但是这里个人认为还是写的清楚一点较好。

```java
public class Solution {

    private int[][] dp;

    public boolean match(char[] str, char[]  pattern) {
       // 保存中间状态
        dp = new int[str.length + 1][pattern.length + 1];
        return isMatch(str, pattern, str.length, pattern.length);
    }

    private boolean isMatch(char[] s, char[] p, int ls, int lp) {
        if (ls == 0 && lp == 0)
            return true;
       // dp[ls][lp]=1 表示 s[0-ls-1] 与 lp[0-lp-1] 匹配
        if (dp[ls][lp] != 0)
            return dp[ls][lp] == 1;
      // 如果模板字符串长度为 0 
        if (lp == 0)
            return false; 
        boolean res = false;
        if (ls == 0) {
            res = lp >= 2 && p[lp - 1] == '*' && isMatch(s, p, ls, lp - 2);
        } else {
          // 如果当前 p[lp-1]==s[ls-1] ，则两个指针都 左移，进行下一个匹配。 
            if (p[lp - 1] == '.' || p[lp - 1] == s[ls - 1]) {
                res = isMatch(s, p, ls - 1, lp - 1);
              //  如果 p[lp-1] == "*"， 则还要考虑前一位的情况，"*" 可以表示 0 个 或多个
            } else if (p[lp - 1] == '*') {
              //如果 前一位 是 "." ，则 分三种情况。
                if (p[lp - 2] == '.')
                    res = isMatch(s, p, ls - 1, lp - 1)
                            || isMatch(s, p, ls - 1, lp)
                            || isMatch(s, p, ls, lp - 2);
                else if (s[ls - 1] == p[lp - 2]) {
                  // 如果 前一位与 s[ls-1] 相等
                    res = isMatch(s, p, ls - 1, lp - 2) //这里和上面不同，不是ls-1, lp-1, 
                            || isMatch(s, p, ls - 1, lp)
                            || isMatch(s, p, ls, lp - 2);
                } else
                    res = isMatch(s, p, ls, lp - 2);
            }
        }
        dp[ls][lp] = res ? 1 : -1;
        return res;
    }
}
```

### 53 表示数值的字符串

> 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

解析：

模拟题，比较好的方式是用三个`bool` 变量 `sign dot E`记录前面是否已经出现了过`+/-、.、E/e`。

- 首先最后一个字符不能是`E、e、.、+、-`；
- 当`str[i] == E/e`时，先判断之前有没有出现过`E/e`，且`E`前面只能是数字；
- 当`str[i] == +/-`时，如果是前面以及出现了`+/-`，则这个`+/-`只能在`E/e`之后。如果第一次出现`+/-`，则必须出现在开头或者在`E/e`之后；
- 当`str[i] == '.'`时，判断`.`只能出现一次，且`.`不能出现在`E`之后；

```java
import java.math.BigDecimal;

class Solution {

    public boolean isNumeric(char[] str) {
        if (str == null || str.length == 0)
            return false;
        int n = str.length;
        // 最后一个不能为这些
        if (str[n - 1] == 'E' || str[n - 1] == 'e' || str[n - 1] == '.' || str[n - 1] == '+' || str[n - 1] == '-')
            return false;
        boolean sign = false, dot = false, E = false; // 是否出现 +/- 、.　、E/e
        for (int i = 0; i < str.length; i++) {
            if (str[i] == 'e' || str[i] == 'E') {
                if (E) return false; // 只能出现一个E
                if (i == str.length - 1) return false; // E后面一定要有东西
                if (i > 0 && (str[i - 1] == '+' || str[i - 1] == '-' || str[i - 1] == '.')) return false; //E 前面是数字
                E = true;
            } else if (str[i] == '-' || str[i] == '+') {
                // 第二次出现+- 必须在e之后
                if (sign && str[i - 1] != 'e' && str[i - 1] != 'E') return false; // 第二个符号必须在E的后面
                // 第一次出现+-符号，且不是在字符串开头，则也必须紧接在e之后
                if (!sign && i > 0 && str[i - 1] != 'e' && str[i - 1] != 'E') return false;
                sign = true;
            } else if (str[i] == '.') { // dot
                if (E || dot) return false; // E后面不能有小数点, 小数点不能出现两次 例如: 12e+4.3
                dot = true;
            } else if (str[i] < '0' || str[i] > '9') {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        System.out.println(new Solution().isNumeric(new char[]{'-', '.', 'E', '5'})); // false
        System.out.println(new Solution().isNumeric(new char[]{'-', '+'}));           // false
        System.out.println(new Solution().isNumeric(new char[]{'-', '.', 'E', '+'})); // false

        // 测试 科学计数法
        BigDecimal bd = new BigDecimal("-3.40256010353E11");
        System.out.println(bd.toPlainString());
    }
}
```

### 54 字符流中第一个不重复的字符

> 请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。如果当前字符没有出现一次的字符，返回 `#` 字符。

解析：

O(256)方法

使用一个队列，来记录 `c[ch]==1` 的，然后每次查询的时候，从对头取，直到取到`c[q.peek()] == 1`的，就是我们要的，**因为队列的先进先出，所以对头的一定是我们之前最早加入的**。

```java
import java.util.*;

public class Solution {

    private int[] c;
    private Queue<Character> q;

    public Solution() {
        c = new int[256];
        q = new LinkedList<>();
    }

    public void Insert(char ch) {
        //  如果 只出现 一次，则入队。 
        if (++c[ch] == 1) q.add(ch); // 将出现一次的入队
    }

    public char FirstAppearingOnce() {
       // 出队时判断一下是否为 一次
        while (!q.isEmpty() && c[q.peek()] != 1) q.poll();
        if (q.isEmpty()) return '#'; // 不能将这个放在上面，可能会空指针异常
        return q.peek();
    }
}
```

### 55 链表中环的入口节点

> 给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出`null`。

解析：

使用快慢指针

- 两个指针`fast` 、 `slow`，`fast` 一次走两步，`slow` 一次走一步。
- 如果有环，它们一定会在环内相遇.。如果走到了末尾则不存在环。

**如何找到环入口？**

- 先找到环的节点数n，然后让 一个快指针先走 n 步,然后快指针和慢指针再同步 一步一步走，第一次相遇的节点就是 环的入口。
- 在 判断一个链表有环时 ，用一个快指针和一个慢指针，如果两个相遇则在环中，然后再一步一步走，再次到这个节点的时候就走过了整个环



```java
public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
       // 如何判读一个 链表有环？
       /*
        定义两个指针，同时从头结点出发，一个快指针，一次走两个节点，一个慢指针，一次走一步，如果快指针和慢指针相遇则 存在环，
        如果快指针 走到了 末尾，则不存环。
       */
        /*
           如何找到环入口？
           1. 先找到环的节点数n，然后让一个快指针先走 n 步， 然后快指针和慢指针再同步 一步一步走，第一次相遇的节点就是 环的入口
           如何找到环的节点数？
           1，在 判断一个链表有环时 ，用一个快指针和一个慢指针，如果两个相遇则在环中，然后再一步一步走，再次到这个节点的时候就走过了整个环。
        */
        if (pHead==null||pHead.next==null)
            return null;
        ListNode fastnode=pHead.next,slownode=pHead;
        while(fastnode!=slownode){
            if(fastnode==null||fastnode.next==null)
                return null;
            fastnode=fastnode.next.next;
            slownode=slownode.next;
        }
        int count =1;
        slownode = slownode.next;
        while(slownode!=fastnode)
        {count++;
         slownode = slownode.next;
        }
        fastnode = pHead;
        slownode = pHead;
        while(count>0){
            fastnode=fastnode.next;
            count--;
        }
        while(slownode!=fastnode){
            slownode=slownode.next;
            fastnode=fastnode.next;
        }
        return slownode;
    }
}
```

### 56 删除链表中重复的节点

> 在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表`1->2->3->3->4->4->5` 处理后为 `1->2->5`。

解析：



用三个变量 `pre cur next` 操作即可。 注意用 dummyHead 操作会方便一些。

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/%E5%89%91%E6%8C%87Offer/images/56_s.png)

```java
public class Solution {
    
    public ListNode deleteDuplication(ListNode pHead){
        if(pHead == null) return null;
        if(pHead.next == null) return pHead;
        ListNode dummyHead = new ListNode(Integer.MAX_VALUE);
        dummyHead.next = pHead;
        ListNode pre = dummyHead, cur = pHead, next;
        while(cur != null){
            next = cur.next;
            if(next == null) break;
            if(cur.val == next.val){
                while(next != null && cur.val == next.val)//重复的一截
                    next = next.next;
                pre.next = next;// 减掉中间重复的
                cur = next;
            }else {
                pre = pre.next;
                cur = cur.next;
            }
        }
        return dummyHead.next;
    }
}
```

### 57 二叉树的下一个节点

> 给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

解析：

中序遍历， `左 根 右` 

三种情况：

- 如果该节点有 右孩子节点，则下一节点是右孩子的最左节点。
- 否则 如果该节点的父节点 不为空，而且该节点是它的 最节点，则下一个节点是 父节点。
- 如果 该节点 是父节点的右孩子，则向上查找。

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%AE%97%E6%B3%95/Tree/images/houji3.png)

```java
public class Solution {

    // next 最好写成 parent
    public TreeLinkNode GetNext(TreeLinkNode pNode) {
        if (pNode == null) return null;
        if (pNode.right != null) return getMostLeft(pNode.right); // 答案是: 右孩子的最左节点
        if (pNode.next != null && pNode.next.left != null && pNode.next.left == pNode) // 答案是: 父亲
            return pNode.next;
        while (pNode.next != null && pNode.next.right != null && pNode.next.right == pNode) //答案是不断的往上找
            pNode = pNode.next;
        return pNode.next;
    }
	//获得node的最左下角节点
    public TreeLinkNode getMostLeft(TreeLinkNode node) {
        while (node.left != null) node = node.left;
        return node;
    }
}
```

### 58 对称的二叉树

> 请实现一个函数，用来判断一颗二叉树是不是对称的。注意，**如果一个二叉树同此二叉树的镜像是同样的**，定义其为对称的。

解析：

递归思路。

- 首先根节点，只要 `pRoot.left` 和 `pRoot.right`对称即可
- 左右节点的值相等且对称子树`left.left `和`right.right` 对称，且`left.rigth和right.left也对称`。

```java
public class Solution {

    boolean isSymmetrical(TreeNode pRoot){
        return pRoot == null ? true : mirror(pRoot.left, pRoot.right);
    }

    boolean mirror(TreeNode left, TreeNode right) {
        if(left == null && right == null) return true;
        if(left == null || right == null) return false;
        return left.val == right.val 
                && mirror(left.left, right.right) 
                && mirror(left.right, right.left);
    }
}
```



### 59 按之字形顺序打印二叉树

> 请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

解析：

层次遍历，把偶数层翻转一下即可。

```java
import java.util.*;
public class Solution {

    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        // 最终结果 序列
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        if(pRoot == null) 
            return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(pRoot);
        boolean ok = false;
        ArrayList<Integer> list = new ArrayList<>();
        while(!queue.isEmpty()){
            int n = queue.size();
            ArrayList<Integer> tmp = new ArrayList<>();
            for(int i = 0; i < n; i++){
                TreeNode cur = queue.poll();
                tmp.add(cur.val);
                if(cur.left != null) queue.add(cur.left);
                if(cur.right != null) queue.add(cur.right);
            }
            if(ok) Collections.reverse(tmp);
            ok = !ok;
            res.add(tmp);
        }
        return res;
    }
}
```

### 60 把二叉树打印多行

> 从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

```java
import java.util.*;

public class Solution {
    ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        if(pRoot == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(pRoot);
        while(!queue.isEmpty()){
            int n = queue.size();
            ArrayList<Integer> tmp = new ArrayList<>();
            for(int i = 0; i < n; i++){
                TreeNode cur = queue.poll();
                tmp.add(cur.val);
                if(cur.left != null) queue.add(cur.left);
                if(cur.right != null) queue.add(cur.right);
            }
            res.add(new ArrayList<>(tmp));
        }
        return res;
    }
}
```

## 61-70

### 61 序列化二叉树

> 请实现两个函数，分别用来序列化和反序列化二叉树

解析：

将一个二叉树 序列化 为一个 字符串



- 序列化过程

通过前序遍历，将二叉树变成字符串，`null` 序列化 为 `#` 。每个 节点 之前用 `,` 隔开。

- 反序列化过程

序列化的 相反 过程。

```java
public class Solution {
    String Serialize(TreeNode root) {
        if(root == null)
            return "";
        StringBuilder sb = new StringBuilder();
        Serialize2(root, sb);
        return sb.toString();
    }
     
    void Serialize2(TreeNode root, StringBuilder sb) {
        if(root == null) {
            sb.append("#,");
            return;
        }
        sb.append(root.val+",");
        Serialize2(root.left, sb);
        Serialize2(root.right, sb);
    }
     
    int index = -1;
     
    TreeNode Deserialize(String str) {
        if(str.length() == 0)
            return null;
        String[] strs = str.split(",");
        return Deserialize2(strs);
    }  
     
    TreeNode Deserialize2(String[] strs) {
        index++;
        if(!strs[index].equals("#")) {
            TreeNode root = new TreeNode(0);     
            root.val = Integer.parseInt(strs[index]);
            root.left = Deserialize2(strs);
            root.right = Deserialize2(strs);
            return root;
        }
        return null;
    }
}
```

### 62 二叉搜索树的第k个结点

> 给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）    中，按结点数值大小顺序第三小结点的值为4。

解析：

中序遍历 然后 计数。中序遍历的的 非递归方法：

中序遍历的顺序 为 ： **左 根 右**

非递归的方法：先一直向左查找，将非叶子节点加到 栈中，当到叶子节点的时候，输出叶子节点（**左**）。然后从栈中获得栈顶元素，输出栈顶元素（**根**）。然后查找栈顶元素的 右节点（**右**）。继续查找。

```java
import java.util.*;

public class Solution {
    TreeNode KthNode(TreeNode pRoot, int k){
        Stack<TreeNode> stack = new Stack<>();
        TreeNode p = pRoot;
        int cnt = 0;
        while(!stack.isEmpty() || p != null){
            while(p != null){
                stack.push(p);
                p = p.left;
            }
            // 当前 p == null
            p = stack.pop();
            cnt++;
            if(k == cnt) 
                return p;
            p = p.right;
        }
        return null;
    }
}
```

### 63 数据流中的中位数

> 如何得到一个数据流中的中位数？**如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。**我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

解析：

利用 一个  `最大堆`和  一个 `最小堆` 。

**最小堆 的 堆顶最小，存的是最大的 `n/2` 个元素。**

**最大堆  的 堆顶最大，存的是最大的 `n/2` 个元素。**

最小堆的 的**堆顶**  > 最大堆的 **堆顶**。

```java
import java.util.PriorityQueue;

public class Solution {

    //堆顶最小，但是存的是最大的 n/2个元素
    private PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    //堆顶最大，但是存的是最小的 n/2个元素
    private PriorityQueue<Integer> maxHeap = new PriorityQueue<>((o1, o2) -> o2 - o1);

    public void Insert(Integer num) {
      // 如果 最大堆 为空，或者 该数 小于 最大堆最大值,说明 该数字在 数据流中 后一半。
        if(maxHeap.isEmpty() || num <= maxHeap.peek()){
            maxHeap.add(num);
        }else{
            minHeap.add(num);
        }
       // 如果 minHeap 多，就加入到 minHeap 。 
        if(minHeap.size() - maxHeap.size() > 1)
            maxHeap.add(minHeap.poll());
        else if(maxHeap.size() - minHeap.size() > 1){
            minHeap.add(maxHeap.poll());
        }
    }

    public Double GetMedian() {
        if(minHeap.size() > maxHeap.size())
            return 1.0 * minHeap.peek();
        else if(maxHeap.size() > minHeap.size())
            return 1.0 * maxHeap.peek();
        else
            return 1.0 * (minHeap.peek() + maxHeap.peek())/2;
    }
}
```

[PriorityQueue原理（堆排序）](C:\Users\Administrator\Documents\笔记\刷题\PriorityQueue原理.md)

### 64 滑动窗口的最大值

> 给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组`{2,3,4,2,6,2,5,1}`及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为`{4,4,6,6,6,5}`； 针对数组`{2,3,4,2,6,2,5,1}`的滑动窗口有以下6个：` {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}`。



解析：

**单调队列介绍**

单调队列一开始用来求出**数组的某个区间范围内求出最大值**。具体的做法如下

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/363_t.png)

准备一个双端队列，**队列遵守从左到右从大到小**（不能相等）的规则，然后当窗口决定加入一个数，则`R`往右移动一位，`L` 不动，并且**判断队列中的尾部的数是否比刚刚窗口加入的数小**，如果不是就加入，如果是就一直弹出队列尾部的数，直到比这个要加入的数大，就把这个数的下标加入双向链表，由于这里一开始队列为空直接加入。

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s.png)

再接下来判断 1 位置上 的  `4` 和 `2`位置的  `1`,它们都比 各自的队列尾部小，可以直接进入队列；

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s3.png)

再判断`3`位置上的`2`，由于比`1`大(也就是队列的尾部比这个数小)，所以把队列尾部弹出一个，`1`弹出，由于`4`比`2`大，就可以放`２`了:

![](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s4.png)

再看窗口中减数的逻辑，当`L`向右移动的时候，检查队列中的头部**有没有过期，也就是还在不在窗口，如果不在，就弹出，如图，弹出5，L向右移动**:

[![这里写图片描述](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s5.png)](https://github.com/ZXZxin/ZXBlog/blob/master/刷题/Other/LintCode/TwoPointer/images/362_s5.png)

然后如果`R`再向右移动一位，来到`4`位置上的`6`，队列中的数都比`6`小，所以都弹出:

[![这里写图片描述](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s6.png)](https://github.com/ZXZxin/ZXBlog/blob/master/刷题/Other/LintCode/TwoPointer/images/362_s6.png)

继续往后，直到最后:

[![这里写图片描述](https://github.com/ZXZxin/ZXBlog/raw/master/%E5%88%B7%E9%A2%98/Other/LintCode/TwoPointer/images/362_s7.png)](https://github.com/ZXZxin/ZXBlog/blob/master/刷题/Other/LintCode/TwoPointer/images/362_s7.png)

- 上面就是单调队列的窗口加数和减数的操作规则；
- 要注意队列从左到右一直是降序的；
- 要注意减数的判断过期规则；
- 所以在任意的窗口时刻，**队列头部就是这个窗口的最大值**；

总的来说，维持一个 `单调队列` ， 单调队列 中的 队头到 队尾 依次减小，而且 单调队列中 的数字索引应该都在滑动窗口中。队列中存放的   元素的  `索引`。

```java
import java.util.*;

public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size){
        ArrayList<Integer> res = new ArrayList<>();
        if(num == null || size < 1 || num.length < size) return res;
        LinkedList<Integer> qmax = new LinkedList<>();
        for(int i = 0; i < num.length; i++){ 
          // 将 小的都弹出。
            while(!qmax.isEmpty() && num[qmax.peekLast()] < num[i])
                qmax.pollLast();
            qmax.addLast(i);
           // 过期数据
            if(i - size == qmax.peekFirst()) qmax.pollFirst();
            if(i >= size-1) res.add(num[qmax.peekFirst()]);
        }
        return res;
    }
}
```

### 65 矩阵中的路径

> 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 `a b c e s f c s a d e e` 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

解析：

`dfs` = 深度优先遍历

边界判断条件 

`if (cur == str.length-1 && matrix[x * c + y] == str[cur]) return true;`

```java
public class Solution {

    final int[][] dir = {{-1, 0}, {0, 1}, {1, 0}, {0, -1}};

    private boolean vis[];
    private int r, c;

    private boolean dfs(char[] matrix, char[] str, int cur, int x, int y) {
        if (cur == str.length-1 && matrix[x * c + y] == str[cur]) return true;
        if (vis[x * c + y] || matrix[x * c + y] != str[cur]) return false;
        vis[x * c + y] = true;
        for (int i = 0; i < 4; i++) {
            int nx = x + dir[i][0];
            int ny = y + dir[i][1];
            if (nx >= 0 && nx < r && ny >= 0 && ny < c && !vis[nx * c + ny] &&
                    (dfs(matrix, str, cur + 1, nx, ny))) return true;
        }
        vis[x * c + y] = false;
        return false;
    }

    public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
        r = rows;
        c = cols;
        vis = new boolean[r * c];
        for (int i = 0; i < matrix.length; i++) {
            // x  行
            int x = i / c;
            // y 列
            int y = i % c;
            if (dfs(matrix, str, 0, x, y)) return true;
        }
        return false;
    }

    public static void main(String[] args) {
//        char[] matrix = {'a', 'b' ,'c' ,'e' ,'s' ,'f','c' ,'s','a' ,'d','e','e'};
        char[] matrix = {'A','A','A','A','A','A','A','A','A','A','A','A'};
//        char[] str = {'b','c','c','e','d'};
//        char[] str = {'a', 'b','c','d'};
        char[] str = {'A','A','A','A','A','A','A','A','A','A','A','A','A'};
        System.out.println(new Solution().hasPath(matrix, 3, 4, str));
    }
}
```

### 66 机器人的 运动范围

> 地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，**但是不能进入行坐标和列坐标的数位之和大于k的格子**。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

解析：

DFS 深度优先遍历

维持一个 访问数组，记录某个 坐标，是否为访问过。

**与上一题的区别**,上一题回溯时需要还原状态。

```java
public class Solution {
    public int movingCount(int threshold, int rows, int cols)
    {
        if(threshold<=0||rows<=0 || cols<=0 )
            return 0;
        boolean[] visited   = new boolean[rows*cols];
        movinhCore(threshold,rows,cols,0,0,visited);
        int sum = 0;
        for(int i =0;i<rows*cols;i++){
            if(visited[i]==true)
                sum++;
        }
        return sum;
    }
    void movinhCore(int threhold,int rows,int cols,int row, int col,boolean[] visited){				// 如果没有访问过
        if(row>=0 && row<rows && col>=0 && col<cols && !visited[row*cols+col] && sum1(row,col)<=threhold) {
            visited[row*cols+col] = true ;
            movinhCore( threhold,rows, cols, row,  col+1,visited); 
            movinhCore( threhold,rows, cols, row+1,  col,visited); 
            movinhCore( threhold,rows, cols, row,  col-1,visited);
            movinhCore( threhold,rows, cols, row-1,  col,visited); 
        }
    }
    
   int sum1(int row,int col){
       int sum = 0;
       while(row!=0){
           sum+=row%10;
           row = row/10;
       }
       while(col!=0){
           sum+=col%10;
           col = col/10;
       }
       return sum;
   
   }
}
```
### 67 LRU 缓存机制

> 运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。
>
> 获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
> 写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

解析：

用hashMap + 双端链表实现

```java
class DlinkedNode{
    int key;
    int value;
    DlinkedNode pre;
    DlinkedNode post;
    DlinkedNode(){}
    DlinkedNode(int key,int value){
        this.key  = key;
        this.value = value;
    }
```

类中 保存 双端 链表的  head 和 tail  指针。方便 在链表 头部和尾部 插入和 删除元素。

```java
head = new DlinkedNode();
head.pre = null;
tail = new DlinkedNode();
tail.post = null;
head.post=tail;
tail.pre = head;
```

hash 表中 保存 `HashMap<Integer,DlinkedNode> cache = new HashMap<>()` 插入元素的值和 其 在链表中的节点。

#### Put  操作



```
public void put(int key, int value) {
    DlinkedNode node = cache.get(key);
    if(node==null){
        node = new DlinkedNode(key,value);
        addHead(node);
        cache.put(key,node);
        count++;
        if(count>capacity){
            DlinkedNode tai  = popTail();
            cache.remove(tai.key);
            --count;
        }
    }else{
        node.value = value;
        movehead(node);

    }
} 
```

### get 操作

```java
public int get(int key) {
        DlinkedNode node = cache.get(key);
        if(node==null)
            return -1;
        this.movehead(node);
        return node.value;

    }
```

### 68 排序链表

> 在 *O*(*n* log *n*) 时间复杂度和常数级空间复杂度下，对链表进行排序。

解析：

 归并排序

```java
class Solution {
    public ListNode sortList(ListNode head) {
        return head ==null ? null : mergeSort(head);
    }
    private ListNode mergeSort(ListNode head){
      // 如果 只有一个节点 ，则返回该节点
        if(head.next == null )
            return head;
        // 否则查找 链表中间节点
        ListNode p = head,q = head, pre =null;
        // 寻找链表中间节点
        while(q!=null&&q.next!=null){
            pre =p;
            p = p.next;
            q = q.next.next;
        }
        pre.next=null;
        ListNode l = mergeSort(head);
        ListNode r = mergeSort(p);
        return merge(l,r);        
    }
    ListNode merge(ListNode l, ListNode r) {
        ListNode dummyHead = new ListNode(0);
        ListNode cur = dummyHead;
        while (l != null && r != null) {
            if (l.val <= r.val) {
                cur.next = l;
                cur = cur.next;
                l = l.next;
            } else {
                cur.next = r;
                cur = cur.next;
                r = r.next;
            }
        }
        if (l != null) {
            cur.next = l;
        }
        if (r != null) {
            cur.next = r;
        }
        return dummyHead.next;
    }
}
```

### 69 乘积最大子序列



> 给定一个整数数组 `nums` ，找出一个序列中乘积最大的连续子序列（该序列至少包含一个数）。

解析：

动态规划，当前元素为末尾的最大值 = 前一个元素的最大值*当前元素 与 当前元素 相比的最大值。

因为 整数数组中存在 正数和负数，所以还应该保存一个 最小值

```java
class Solution {
    public int maxProduct(int[] nums) {
      int max = Integer.MIN_VALUE, imax = 1, imin = 1;
        for(int i=0; i<nums.length; i++){
            if(nums[i] < 0){ 
              int tmp = imax;
              imax = imin;
              imin = tmp;
            }
            imax = Math.max(imax*nums[i], nums[i]);
            imin = Math.min(imin*nums[i], nums[i]);
            
            max = Math.max(max, imax);
        }
        return max;
        
    }
}
```



### 70  每日温度

>根据每日 气温 列表，请重新生成一个列表，对应位置的输入是你需要再等待多久温度才会升高超过该日的天数。如果之后都不会升高，请在该位置用 0 来代替。
>
>例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。
>
>提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。
>

解析：

对于  *temperatures*  数组 从后往前遍历，用一个 临时数组 保存 遍历过的温度的 索引。 遍历到当期温度后，在临时数组中遍历 大于该温度的索引，取最小的索引。

然后将这个最小的索引减去 当前索引。并将 当前温度索引保存到临时数组中。

```java
public int[] dailyTemperatures(int[] T) {
        int n = T.length;
        int[] res = new int[n];
        int[] tmp = new int [101];
        for(int i =n-1;i>=0;i--){
            int index=i;
            for(int j=T[i]+1;j<=100;j++){
                if(tmp[j]!=0)
                {
                    if(index==i){
                        index =tmp[j];
                    }
                    else{
                        index = Math.min(index,tmp[j]);
                    }
                }
            }
            
            res[i]=index-i;
            tmp[T[i]]=i;
        }
        return res;
    }
```

解法 2：

从后往前遍历，利用已有结果进行跳跃：

![](https://pic.leetcode-cn.com/0f16daf6fde5475d72cbb6e9efec1d66409590141b861c2cf62fd87394211a82-image.png)

假设已经知道75  之后的结果，当遍历到 75，先看 71 是不是比 75 大，如果不是，则 直接跳到 72 比较（因为71 位置上的结果是 2，意味比 他索引大2 的位置上数字比它大）。

```java
public int[] dailyTemperatures(int[] T) {
    int length = T.length;
    int[] result = new int[length];

    //从右向左遍历
    for (int i = length - 2; i >= 0; i--) {
        // j+= result[j]是利用已经有的结果进行跳跃
        for (int j = i + 1; j < length; j+= result[j]) {
            if (T[j] > T[i]) {
                result[i] = j - i;
                break;
            } else if (result[j] == 0) { //遇到0表示后面不会有更大的值，那当然当前值就应该也为0
                result[i] = 0;
                break;
            }
        }
    }
    return result;
}
```



### 71 回文字符串

> 给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。
>
> 具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被计为是不同的子串。

解析：

**中心扩展法**：

以当前节点 i 为中心节点，获取 以当前 点 i 的奇回文和偶回文。

```java
public int countSubstrings2nd(String s) {
    int result = 0;
    for (int i = 0; i < s.length(); i++) {
        //以当前点i位置，向两边扩展,以i i+1位置向两边扩展
        result += countSegment(s, i, i);
        result += countSegment(s, i, i + 1);
    }
    return result;
}


public int countSegment(String s, int start, int end) {
    int count = 0;
    //start往左边跑，end往右边跑，注意边界
    while (start >= 0 && end < s.length() && s.charAt(start--) == s.charAt(end++)) {
        count++;
    }
    return count;
}
```

**链表回文**

先定位到链表的中间位置，再将链表的后半段逆置。之后使用两个指针同时从链表头部和中间开始逐一向后遍历比较。时间复杂度为O(n)，空间复杂度为O(1)。

定位到 链表中间

```java
ListNode slow = head, fast = head;
while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
}
```

   此时 slow 指向的是中间节点，将后半段节点 翻转

```java
ListNode mid = slow.next;
slow.next = null;
ListNode right = reverseList(mid);
// 使用 三个指针 cur ,pre ,next 来翻转链表
private ListNode reverseList(ListNode head) {
    ListNode current = head, next, pre = null;
    while (current != null) {
        // 记录后继结点
        next = current.next;
        // 后继指针逆向
        current.next = pre;
        // 记录当前结点
        pre = current;
        // 下一结点成为当前结点
        current = next;
    }
    return pre;
}
```

再比较 前后两个链表，查看是否为回文

```java
while (left != null && right != null) {
        if (left.val != right.val) {
            return false;
        }
        left = left.next;
        right = right.next;
    }
    return true;
```

**栈回文**

根据栈的特性，将字符串全部压入栈 ，再依次将各个字符出栈，从而得到原字符串的逆串，然后一个个比较。

#### 最长回文子序列

>给定一个字符串s，找到其中最长的回文子序列。可以假设s的最大长度为1000。
>
>示例 1:
>输入:
>
>"bbbab"
>输出:
>
>4
>一个可能的最长回文子序列为 "bbbb"。

**DP 步骤版**

```java
public int longestPalindromeSubseq(String s) {     
        // dp[i][j]  表示 从i 到 j 最长回文子序列长度
        // dp[i][j] = dp[i+1][j-1] + 2  如果 s.charAt(i)==s.charAt(j)
        // dp[i][j] = Math.max(dp[i+1][j],dp[i][j-1])
        if(s==null|s.length()==0)
            return 0;
        int len  = s.length();
        int[][] dp = new int[len][len];
        for(int i=len-1;i>=0;i--){
            dp[i][i]=1;
            for(int j=i+1;j<len;j++){
                if(s.charAt(i)==s.charAt(j)){
                    dp[i][j]=dp[i+1][j-1]+2;
                }else{
                    dp[i][j] = Math.max(dp[i][j-1],dp[i+1][j]);
                    }
            }
        }
        return dp[0][len-1];
    }
```

### 72 任务调度器

 >给定一个用字符数组表示的 CPU 需要执行的任务列表。其中包含使用大写的 A - Z 字母表示的26 种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。CPU 在任何一个单位时间内都可以执行一个任务，或者在待命状态。
 >
 >然而，两个相同种类的任务之间必须有长度为 n 的冷却时间，因此至少有连续 n 个单位时间内 CPU 在执行不同的任务，或者在待命状态。
 >
 >你需要计算完成所有任务所需要的最短时间。

解析：

通过分析 可以知道最短时间与最高频率的字母有直接关系。

但是 有可能 出现 特殊情况，即存在一种特殊情况，例如：

输入: tasks = ["A","A","A","B","B","B","C","C","D","D"], n = 2
输出: 10
执行顺序: A -> B -> C -> A -> B -> D -> A -> B -> C -> D
此时如果按照上述方法计算将得到结果为 8，比数组总长度 10 要小，应返回数组长度。

```java
 public int leastInterval(char[] tasks, int n) {
        if(tasks.length==0) return 0;
        int ch[] = new int[256];

        for(char c : tasks){
            ch[c]++;
        }
        int max = Integer.MIN_VALUE;
        for(int i :ch){
            max =Math.max(max,i);
        }
        int  count=0;
        for(int i :ch){
            if(i==max) ++count;
        }
        return Math.max((n+1)*(max-1)+count,tasks.length);
    }
```

### 73 合并二叉树

>给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。
>
>你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。
>

解析：

递归，构造 二叉树。

```java
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
         if(t1 == null){
            return t2;
        }
        if(t2 == null){
            return t1;
        }
        TreeNode result = new TreeNode(t1.val + t2.val);
        result.left = mergeTrees(t1.left,t2.left);
        result.right = mergeTrees(t1.right,t2.right);
        return result;
    }  
```

### 74 最短无序连续子数组

> 给定一个整数数组，你需要寻找一个**连续的子数组**，如果对这个子数组进行升序排序，那么整个数组都会变为升序排序。
>
> 你找到的子数组应是**最短**的，请输出它的长度。

解析：

1. 另外申请一个空间，复制数组后排序，逐一比对，找到未就位的区间段。

```java
public int findUnsortedSubarray(int[] nums) {
    int len = nums.length;
    int[] tmp = (int[])nums.clone();
    Arrays.sort(tmp);
    int begin = 0, end = len-1;
    while(begin < end){
        int flag =1;
        if(nums[begin] == tmp[begin]){
            begin++;
            flag = 0;
        }
        
        if(nums[end] == tmp[end]){
            end--;
            flag = 0;
        }
        //  如果 左右都不相等
        if(flag == 1) break;
    }
    return end-begin>0?end-begin+1:0;
}  
```

2. 在开头遍历找到第一个降序的位置，往前的区域定义为S，从结尾遍历找到第一个升序的位置，往后的区域定义为E 这里还需要考虑有重复值得情况，因此S区域还要根据最后一个值前面是否有重复的情况，往前缩小S，E同理 然后在中间这段序列里边找到这里边的最大值，最小值。 使用最小值在S区域里，从开始找第一个大于最小值得位置s，使用最大值在E区域里，从尾开始找第一个小于最大值的位置e 最终的结果是e - s + 1 。

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        int s = -1;
        int e = -1;
        // 从前往后，找到第一个降序 的位置
        for(int i = 0; i < nums.length - 1; i++){
            if(nums[i] > nums[i + 1]){
                s = i;
                break;
            }
        }
        //	缩小 s ，加上 重复的位置
        while(s > 0 && nums[s] == nums[s - 1]) s--;
        //  是升序序列 
        if(s == -1) return 0;
        // 从后往前，找到第一个升序的位置。
        for(int i = nums.length - 1; i > 0; i--){
            if(nums[i] < nums[i - 1]){
                e = i;
                break;
            }
        }
        // E 同理 
        while(e < nums.length - 1 && nums[e] == nums[e + 1]) e++;
        
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;
        // 在 s-e 范围中，找到 局部最大值 和局部最小值
       for(int i = s; i <= e; i++){
           if(min > nums[i])
               min = nums[i];
           if(max < nums[i])
               max = nums[i]; 
       }
        // 在 s 区域内 找到第一个 大于 最小值的值
        int res_s = -1, res_e = -1;
        for(int i = 0; i <= s; i++){
            if(nums[i] > min){
                res_s = i;
                break;
            }
        }
        // 在大与 e 的区域内，找到 大于 最大值的值
        for(int i = nums.length - 1; i >= e; i--){
            if(nums[i] < max){
                res_e = i;
                break;
            }
        }
        return res_e - res_s + 1;
    }
}
```

### 75 和为k的子数组

>给定一个整数数组和一个整数 k，你需要找到该数组中和为 k 的**连续的子数组**的个数。
>
>示例 1 :
>
>输入:nums = [1,1,1], k = 2
>输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。

**解析**：

数组里有正负值，用 hash 表记录 前 0-i 的 sum 的个数。

```java
 public int subarraySum(int[] nums, int k) {
       // hash 表 记录 前0-i 的 sum 对应的个数
        Map<Integer,Integer> map = new HashMap<>();
        map.put(0,1);
        int sum=0;
        int ans=0;
       for(int num:nums){
           sum+=num;
           if(map.containsKey(sum-k))
               ans+=map.get(sum-k);
           map.put(sum,map.getOrDefault(sum,0)+1);
       }
       return ans;
    }
```

### 76 二叉树的直径

> 给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过根结点。

```java
    int max =0;
    public int diameterOfBinaryTree(TreeNode root) {
        if(root==null)
            return 0;
        dfs(root);
        return max;

    }  
    private void dfs(TreeNode root){
        if(root!=null){
            max = Math.max(max,height(root.left)+height(root.right));
            dfs(root.left);
            dfs(root.right);
        }
    }   
    private int height(TreeNode root){
        if(root==null)
            return 0;
        return Math.max(height(root.left),height(root.right))+1;
    }
```

### 77 验证二叉搜索树



>给定一个二叉树，判断其是否是一个有效的二叉搜索树。
>
>假设一个二叉搜索树具有如下特征：
>
>节点的左子树只包含小于当前节点的数。
>节点的右子树只包含大于当前节点的数。
>所有左子树和右子树自身必须也是二叉搜索树。

**解析**：

使用非递归方法 得到 二叉中序序列

```java
Stack<TreeNode> stack = new Stack<>();
TreeNode cur = root;
while(cur!=null||!stack.isEmpty()){
    while(cur!=null){
        stack.push(cur);
        cur = cur.left;
    }
    cur = stack.pop();
    dosomthing;
    cur = cur.right;
}
```

验证 二叉搜索树，只需要在中序遍历中，将 当前值与前一个值比较，如果大于等于，则正确，否则 失败。

也可以用 递归方法做。

中序遍历，记住前驱节点

```java
class Solution {
    TreeNode last=null;
    public boolean isValidBST(TreeNode root) {
        if(root==null)
            return true;
        if(!isValidBST(root.left)) return false;
        if(last!=null&&last.val>=root.val) return false;
        last =root;
        if(!isValidBST(root.right)) return false;
        return true;
    }
    
}
```

### 78 最长连续序列

>给定一个未排序的整数数组，找出最长连续序列的长度。
>
>要求算法的时间复杂度为 *O(n)*。

**解析**：

在 `set` 中 保存所有的序列，然后 遍历每个数字，以这个数字为中心在 set 中查找其他数字。

```java
    public int longestConsecutive(int[] nums) {
        Set<Integer> numsSet = new HashSet<>();
        for (Integer num : nums) {
            numsSet.add(num);
        }
        int longest = 0;
        for (Integer num : nums) {
            if (numsSet.remove(num)) {
                // 向当前元素的左边搜索,eg: 当前为100, 搜索：99，98，97,...
                int currentLongest = 1;
                int current = num;
                while (numsSet.remove(current - 1)) current--;
                currentLongest += (num - current);
		        // 向当前元素的右边搜索,eg: 当前为100, 搜索：101，102，103,...
                current = num;
                while(numsSet.remove(current + 1)) current++;
                currentLongest += (current - num);
        	    // 搜索完后更新longest.
                longest = Math.max(longest, currentLongest);
            }
        }
        return longest;
    }
```

### 79 二叉树的最近公共祖先

>给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
>百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”
>

解析：

利用 后序遍历

```java
 private TreeNode currentNode =null;
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        rec(root,p,q);
        return currentNode;
    }
    private boolean rec(TreeNode cur,TreeNode p,TreeNode q){
        if(cur==null)
            return false;
        int left = rec(cur.left,p,q)?1:0;
        int right = rec(cur.right,p,q)?1:0;
        int mid=0;
        if(cur==q||cur==p)
            mid=1;
        if(left+mid+right>=2)
            currentNode = cur;
        return left+mid+right>0;
    }
```

### 80 单词拆分

>给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。
>
>说明：
>
>拆分时可以重复使用字典中的单词。
>你可以假设字典中没有重复的单词。

解析：

动态规划，完全背包问题，dp[i] = dp[i]||dp[i-len] (len为 字典中单词的长度)，当 这个单词 与  字符串中字段相等时

```java
    public boolean wordBreak(String s, List<String> wordDict) {
      // 完全背包问题
        if(s==null||wordDict==null||wordDict.size()==0)
            return false;
        int n = s.length();
        boolean[] dp = new boolean[n+1];
        dp[0]=true;
        for(int i=1;i<=n;i++){
            for(String word : wordDict){
                int len = word.length();
                if(i>=len&&  word.equals(s.substring(i-len,i)))
                    dp[i] = dp[i]||dp[i-len];
            }
        }
        return dp[n];     
    }
```

### 81 不同的二叉搜索树

>给定一个整数 *n*，求以 1 ... *n* 为节点组成的二叉搜索树有多少种？

动态规划

- 假设n个节点存在二叉排序树的个数是G(n)，令f(i)为以i为根的二叉搜索树的个数，则

*G*(*n*)=*f*(1)+*f*(2)+*f*(3)+*f*(4)+...+*f*(*n*)

- 当i为根节点时，其左子树节点个数为i-1个，右子树节点为n-i，则

*f*(*i*)=*G*(*i*−1)∗*G*(*n*−*i*)

综合两个公式可以得到 卡特兰数 公式
*G*(*n*)=*G*(0)∗*G*(*n*−1)+*G*(1)∗(*n*−2)+...+*G*(*n*−1)∗*G*(0)

```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n+1];
        dp[0] = 1;
        dp[1] = 1;
        
        for(int i = 2; i < n + 1; i++)
            for(int j = 1; j < i + 1; j++) 
                dp[i] += dp[j-1] * dp[i-j];
        
        return dp[n];
    }
}
```

### 82 最大矩形

> 给定一个仅包含 0 和 1 的二维二进制矩阵，找出只包含 1 的最大矩形，并返回其面积。

解析：

维护一个从小大的 栈，

```java
    public int maximalRectangle(char[][] matrix) {
       int h = matrix.length;
        if(h<=0)
            return 0;
        int[] height = new int[matrix[0].length];
        int max=0;
        for(int i=0;i<h;i++){
            for(int j=0;j<matrix[0].length;j++){
                if(matrix[i][j]=='1')
                    height[j] = height[j]+1;
                else 
                    height[j]=0;
            }
            max = Math.max(largestRectangleArea(height),max);           
       }
        return max;
    }
    
     private int largestRectangleArea(int[] heights) {
        Stack < Integer > stack = new Stack <> ();
        stack.push(-1);
        int maxarea = 0;
        for (int i = 0; i < heights.length; ++i) {
            // 如果栈中元素 大于 待添加的元素
            while (stack.peek() != -1 && heights[stack.peek()] >= heights[i])
                maxarea = Math.max(maxarea, heights[stack.pop()] * (i - stack.peek() - 1));
            stack.push(i);
        }
        while (stack.peek() != -1)
            maxarea = Math.max(maxarea, heights[stack.pop()] * (heights.length - stack.peek() -1));
        return maxarea;
    }
```

### 83 最大正方形

>在一个由 0 和 1 组成的二维矩阵内，找到只包含 1 的最大正方形，并返回其面积。

解析：

动态规划。

1. 我们用 0 初始化另一个矩阵 dp，维数和原始矩阵维数相同；
2. dp(i,j) 表示的是由 1 组成的最大正方形的边长；
3. 从 (0,0)(0,0) 开始，对原始矩阵中的每一个 1，我们将当前元素的值更新为
   dp(*i*, *j*)=min(dp(*i*−1, *j*), dp(*i*−1, *j*−1), dp(*i*, *j*−1))+1
4. 我们还用一个变量记录当前出现的最大边长，这样遍历一次，找到最大的正方形边长 maxsqlenmaxsqlen，那么结果就是 maxsqlen^2maxsqlen 
  

```java
public class Solution {
    public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[][] dp = new int[rows + 1][cols + 1];
        int maxsqlen = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                if (matrix[i-1][j-1] == '1'){
                    dp[i][j] = Math.min(Math.min(dp[i][j - 1], dp[i - 1][j]), dp[i - 1][j - 1]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[i][j]);
                }
            }
        }
        return maxsqlen * maxsqlen;
    }
}
```

### 84 二叉树的最大路径和

>给定一个**非空**二叉树，返回其最大路径和。
>
>本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径**至少包含一个**节点，且不一定经过根节点。

**解析:**

对于 任意一个节点，如果最大和路径包含该节点，那么只可能是两种情况；

1. 其左右子树中所构成的和路径值较大的那个加上该节点的值后想父节点回溯构成最大路径
2. 左右子树都在最大路径中，加上该节点的值构成了最终的最大路径。

```java
private int ret = Integer.MIN_VALUE;
public int maxPathSum(TreeNode root){
    getMax(root);
    return ret;
}

private int getMax(TreeNode r){
    if(r==null) reutrn 0;
     int left = Math.max(0, getMax(r.left)); // 如果子树路径和为负则应当置0表示最大路径不包含子树
        int right = Math.max(0, getMax(r.right));
        ret = Math.max(ret, r.val + left + right); // 判断在该节点包含左右子树的路径和是否大于当前最大路径和
        return Math.max(left, right) + r.val;
}
```

### 85 单词搜索

> 给定一个二维网格和一个单词，找出该单词是否存在于网格中。
>
> 单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。
>

解析：

**dfs** + 回溯

```java
public boolean exist(char[][] board,String word){
    if(word == null || word.length==0)
        return true;
    if(board == null || board.length==0 || board[0].length == 0)
        return false;
    m = board.length;
    n = board[0].length;
    boolean[][] hasVisited = new boolean[m][n];
    for(int r=0;r<m;r++){
        for(int c=0;c<n;c++){
            if (backtracking(0, r, c, hasVisited, board, word)) {
                 return true;
            }
        }
    }
}

private boolean backtracking(int curlen,int r,int c,boolean[][] visited,final char[][] board,final String word){
    if (curLen == word.length()) {
        return true;
    }
     if (r < 0 || r >= m || c < 0 || c >= n
            || board[r][c] != word.charAt(curLen) || visited[r][c]) {

        return false;
    }
    
    visited[r][c] = true;

    for (int[] d : direction) {
        if (backtracking(curLen + 1, r + d[0], c + d[1], visited, board, word)) {
            return true;
        }
    }

    visited[r][c] = false;

    return false;
}
```

### 86 对称二叉树

>给定一个二叉树，检查它是否是镜像对称的。
>
>例如，二叉树 `[1,2,2,3,4,4,3]` 是对称的。

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root==null)
            return true;
        return isMirror(root.left,root.right);
    
    }
    
    // 左子树 与右子树是 镜像的 ，左子树的 左子树和 右子树的 右子树是 镜像的，左子树的右子树和 右子树的左子树是对应的。
    public boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    return (t1.val == t2.val)
        && isMirror(t1.right, t2.left)
        && isMirror(t1.left, t2.right);
}
}
```

### 87 子集

> 给定一组**不含重复元素**的整数数组 *nums*，返回该数组所有可能的子集（幂集）。

解析：

**dfs**  + 回溯

```java
 public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> subsets = new ArrayList<>();
    List<Integer> tempSubset = new ArrayList<>();
    for (int size = 0; size <= nums.length; size++) {
        backtracking(0, tempSubset, subsets, size, nums); // 不同的子集大小
    }
    return subsets;
}

private void backtracking(int start, List<Integer> tempSubset, List<List<Integer>> subsets,
                          final int size, final int[] nums) {

    if (tempSubset.size() == size) {
        subsets.add(new ArrayList<>(tempSubset));
        return;
    }
    for (int i = start; i < nums.length; i++) {
        tempSubset.add(nums[i]);
        backtracking(i + 1, tempSubset, subsets, size, nums);
        tempSubset.remove(tempSubset.size() - 1);
    }
}
```

### 88 颜色分类

>给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。
>
>此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。
>

解析： 本问题被称为 **荷兰国旗问题** ，其主要思想是给每个数字设定一种颜色，并按照荷兰国旗颜色的顺序进行调整。

我们用 三个指针来分别追踪0 的最右边界，2 的最左边界和当前考虑的元素。

本题的解法思路是沿着数组移动 cur 指针，若 nums[curr] = 0, 则将其与 nums[p0] 互换，若 nums[curr] = 2 ，则 与 nums[p2] 互换。

While curr <= p2 :

若 nums[curr] = 0 ：交换第 curr个 和 第p0个 元素，并将指针都向右移。

若 nums[curr] = 2 ：交换第 curr个和第 p2个元素，并将 p2指针左移 。

若 nums[curr] = 1 ：将指针curr右移。

实现

```java
    public void sortColors(int[] nums) {
        int zero=-1,one=0,two=nums.length;
        while(one<two){
            if(nums[one]==0){
                swap(nums,++zero,one++);
            }else if(nums[one]==2){
                swap(nums,--two,one);
            }else{
                one++;
            }
        }
    
    }    
    void swap(int[] nums,int i,int j){
        int tmp = nums[i];
        nums[i]=nums[j];
        nums[j]=tmp;
    }
```

### 89 目标和

> 给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。
>

**解析**：

在某个解中 正数和为 x, 负数和 的绝对值为 y ，则 x+y = sum， x-y =S，

所以 x = （sum+s)/2

所以就有在nums中选择一部分数（装与不装到背包中），让其和为x（总的容量为x），此时转化为了0-1背包问题,

同时可有sum+S为奇数，则一定没有解的推论，因为要除以2才是x的解

dp[i] 表示 和为 i  的数目。

```java
 public int findTargetSumWays(int[] nums, int S) {
    int sum = computeArraySum(nums);
    if (sum < S || (sum + S) % 2 == 1) {
        return 0;
    }
    int W = (sum + S) / 2;
    int[] dp = new int[W + 1];
    dp[0] = 1;
    for (int num : nums) {
        for (int i = W; i >= num; i--) {
            dp[i] = dp[i] + dp[i - num];
        }
    }
    return dp[W];
}

private int computeArraySum(int[] nums) {
    int sum = 0;
    for (int num : nums) {
        sum += num;
    }
    return sum;
}
```

### 90 最长有效括号

> 给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

**解析**：

使用栈存放 索引值。

```java
    public int longestValidParentheses(String s) {
        if(s==null||s.length()<=1)
            return 0;
        int max =0;
        // 存放索引值
        Stack<Integer> stack  = new Stack<>();
        int i =-1;
        for(char c : s.toCharArray()){
            i++;            
            if(c==')'&&!stack.isEmpty()&&s.charAt(stack.peek())=='('){
                stack.pop();
                if(stack.isEmpty())
                    max = i+1;
                else if(i-stack.peek()>max)
                    max = i-stack.peek();
            }else{
                stack.push(i);
            }
        }
        return max;
        
    }
```

### 91 找出字符串中所有字母的异位词

>给定一个字符串 **s** 和一个非空字符串 **p**，找到 **s** 中所有是 **p** 的字母异位词的子串，返回这些子串的起始索引。

解析：

**滑动窗口法**：

滑动窗口思想 可以解决 子串问题。

滑动窗口算法的思路是这样：

1、我们在字符串 S 中使用双指针中的左右指针技巧，初始化 left = right = 0，把索引闭区间 [left, right] 称为一个「窗口」。

2、我们先不断地增加 right 指针扩大窗口 [left, right]，直到窗口中的字符串符合要求（包含了 T 中的所有字符）。

3、此时，我们停止增加 right，转而不断增加 left 指针缩小窗口 [left, right]，直到窗口中的字符串不再符合要求（不包含 T 中的所有字符了）。同时，每次增加 left，我们都要更新一轮结果。

4、重复第 2 和第 3 步，直到 right 到达字符串 S 的尽头。

这个思路其实也不难，**第 2 步相当于在寻找一个「可行解」，然后第 3 步在优化这个「可行解」**，最终找到最优解。左右指针轮流前进，窗口大小增增减减，窗口不断向右滑动。

```java
string s, t;
// 在 s 中寻找 t 的「最小覆盖子串」
int left = 0, right = 0;
string res = s;

while(right < s.size()) {
    window.add(s[right]);
    right++;
    // 如果符合要求，移动 left 缩小窗口
    while (window 符合要求) {
        // 如果这个窗口的子串更短，则更新 res
        res = minLen(res, window);
        window.remove(s[left]);
        left++;
    }
}
return res;
```

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList<>();
        // 需要的字符
        Map<Character,Integer> needs = new HashMap<>();
        // 窗口值
        Map<Character,Integer> windows = new HashMap<>();
        for(char c : p.toCharArray()){
            needs.put(c,needs.getOrDefault(c,0)+1);
        }
        // 匹配数
        int match=0;
        int l=0,r=0;
        while(r<s.length()){
            char c = s.charAt(r);
            if(needs.containsKey(c)){
                windows.put(c,windows.getOrDefault(c,0)+1);
                if(windows.get(c).equals(needs.get(c))) {
                    match++;
                }
            }
            r++;
            while(match==needs.size()){
                   char lc = s.charAt(l);
                   if(r-l==p.length())
                       res.add(l);
                   if(needs.containsKey(lc)){
                       windows.put(lc,windows.get(lc)-1);
                       if(windows.getOrDefault(lc,0)<needs.get(lc))
                            match--;
                   }     
                   l++;
               }              
        }
        return res; 
    }
}
```

### 92.筷子

>A先生有很多双筷子。确切的说应该是很多根，因为筷子的长度不一，很难判断出哪两根是一双的。这天，A先生家里来了K个客人，A先生留下他们吃晚饭。加上A先生，A夫人和他们的孩子小A，共K+3个人。每人需要用一双筷子。A先生只好清理了一下筷子，共N根，长度为T1,T2,T3,……,TN.现在他想用这些筷子组合成K+3双，使每双的筷子长度差的平方和最小。(怎么不是和最小？？这要去问A先生了，呵呵)
>
>**输入**：
>
>输入文件共有两行，第一行为两个用空格隔开的整数，表示N,K(1≤N≤100, 0<K<50)，第二行共有N个用空格隔开的整数，为Ti.每个整数为1～50之间的数。
>
>**输出**
>
>输出文件仅一行。如果凑不齐K+3双，输出-1，否则输出长度差平方和的最小值。
>
>