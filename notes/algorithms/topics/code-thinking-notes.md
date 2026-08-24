# 数组

## 1. 二分查找

对于边界的确定，`[left, right]`应确保更新`right`时右边界的有效性。
如当规定为左闭右闭区间时，应写`while (left <= right)`因为要与规定的`[left, right]`保持一致，即`left`与`right`可以取等。同时`arr[middle] > target`时更新右边界`right = middle - 1`，因为`middle`是基于`right`和`left`得到的，而它俩还要保持`[left, right]`的关系。

同理，如果定义的是`[left, right)`左闭右开的区间，应写`right = arr.size()`因为定义的是左闭右开区间，所以后续`while (left < right)`、`right = middle`也不再减1了，因为left也取不到right。

常规写法的二分（三段判断），只查找一个数的时候，如果该数存在`middle`是查询的结果，`left`是其左邻，`right`是其右邻，`right`大于`left`；如果不存在打破`while`循环，`left`比`right`大1，两者之间是其应该插入的位置，由于不能占用左侧`right`所在的数，所以其实就是在`left`所在索引处插入

### [704. 二分查找](https://leetcode.cn/problems/binary-search/description/)

```cpp
// [left, right]
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;
        while (left < right) {
            int middle = left  + ((right - left) / 2);
            if (nums[middle] > target) {
                right = middle;
            } else if (nums[middle] < target) {
                left = middle + 1;
            } else {
                return middle;
            }
        }

        return -1;
    }
};

// [left, right)
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int middle = left + ((right - left) >> 1);
            if (nums[middle] > target) {
                right = middle;
            } else if (nums[middle] < target) {
                left = middle + 1;
            } else {
                return middle;
            }
        }

        return -1;
    }
};
```



### [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/description/)

二分查找中，如果该排序数组中没有目标值最终`left`返回的就是这个数应该插入的下标，`right`就是其左边下标。

边界情况：当插入数比数组中所有数都小，也就说要插入第一个位置时，`left=0, right=-1`。

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int middle = left + ((right - left) / 2);
            if (nums[middle] > target) {
                right = middle - 1;
            } else if (nums[middle] < target) {
                left = middle + 1;
            } else {
                return middle;
            }
        }

        return left;
    }
};
```



### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description/)

两段判断，当小于或等于时都更新，可以一直更新直到获取到边界为止。为什么要这样写呢？因为循环不成立的条件是`left > right`所以找右边界的时候要一直（等于的时候）增大`left`，找左边界的时候要一直减小`right`。总之不管是左边界还是右边界，打破`left <= right`的时候永远都是`left`比`right`大1，`left`在右，`right`在左

**问题：**

> 什么只有在`rightBorder - leftBorder > 1`时才算有而不是大于等于？

因为规定的左边界和右边界的值不包含`target`。假设数组`5 7 7 8 8`求`6`是否在其中，那么最终`rightBorder = 1, leftBorder = 0`，显然是会漏判的。那么这种情况不属于情况3，直接归于情况2`return {-1, -1}`。

打破循环也没有找到数组中有目标值，最终就是`leftBorder - rightBorder < 1`，因为最终查到的的该数应插入的位置，且`left`和`right`相邻，`leftBorder`和`rightBorder`也相邻。

> 为什么更新`rightBorder` 要通过`left`更新，跟新`leftBorder`要通过`right`更新？

因为更新`rightBorder`是从左往右更新的，`left`是一直变化直到打破循环的那个数，如果在更新`rightBorder`的时候通过`right`赋值很可能会出现`right`从不更新的情况。比如当数组为空的时候，`right = left = 0`，那么最终也不会通过`right`的变化更新`rightBorder`的值，反而是通过`left`最终比`right`大1来更新。左边界同理。

> 情况1具体是什么状态？

情况是判断的是某个数在范围大于整个数组或小于整个数组，而且条件要用`||`。假设该数是大于整个数组的，那么右边界会得到一个值，而左边界是通过更新`right`来得到的，但是`right`自始至终都没有更新过所以最终`leftBorder = -3`。该数小于整个数组同理，总会有一个边界值等于-3.



**寻找右边界**

当循环打破，找到右边界时，`right`比`left`小1。因为最后一次`left`多更新了1，所以要以`left`的形式返回结果的话要写成`left - 1`；要以`right`的形式返回结果的话直接写成`right`就可以了

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        int rightBorder = -2;

        while (left <= right) {
            int mid = (left + right) >> 1;
            cout << "mid: " << mid << endl;
            cout << "nums[mid]: " << nums[mid] << endl;
            if (nums[mid] > target) {
                right = mid - 1;
                cout << "r: " << right << endl;
                rightBorder = right;
                cout << "rB: " << rightBorder << endl;
            } else {
                left = mid + 1;
                cout << "l: " << left << endl;
            } 
        }
        cout << "latst r: " << right << endl;
        cout << "latst l: " << left << endl;
        return {99, rightBorder - 1};
    }

};
```



**寻找左边界**

就是不断削减右边界`right`，只要大于或等于`target`就让`right-1` 。最后一次更新的时候`right`会多减1然后小于`left`打破循环，`left`的值就是左边界。

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        int leftBorder = -2;

        while (left <= right) {
            int mid = (left + right) >> 1;
            cout << "mid: " << mid << endl;
            cout << "nums[mid]: " << nums[mid] << endl;
            if (nums[mid] >= target) {
                right = mid - 1;
                cout << "r: " << right << endl;
            } else {
                left = mid + 1;
                cout << "l: " << left << endl;
                leftBorder = left;
                cout << "rB: " << leftBorder << endl;
            } 
        }
        cout << "latst r: " << right << endl;
        cout << "latst l: " << left << endl;
        return {leftBorder, 99};
    }
};
```



**解析**

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {

        int rightBorder = getRightBorder(nums, target);
        int leftBorder = getLeftBorder(nums, target);

        if (leftBorder == -3 || rightBorder == -3)
        {
            cout << "情况1" << endl;
            cout << "lb: " << leftBorder << endl;
            cout << "rb: " << rightBorder << endl;
            return {-1, -1};
        }
        if (rightBorder - leftBorder > 1)
        {
            cout << "情况3" << endl;
            return {leftBorder + 1, rightBorder - 1};
        }
        
        cout << "情况2" << endl;
        return {-1, -1};
    }

private:
    int getRightBorder(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        int rightBorder = -3;

        while (left <= right) {
            int mid = (left + right) >> 1;
            if (nums[mid] > target) {
                right = mid - 1;
            } else {
                left = mid + 1;
                rightBorder = left;
                cout << "rightBorder = left = " << rightBorder << endl;
            }
        }

        cout <<"rightBorder: " << rightBorder << endl;
        cout << "l: " << left << endl;
        cout << "r: " << right << endl;
        return rightBorder;
    }

private:
    int getLeftBorder(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        int leftBorder = -3;

        while (left <= right) {
            int mid = (left + right) >> 1;
            if (nums[mid] >= target) {
                right = mid - 1;
                leftBorder = right;
                cout << "leftBorder = right = " << leftBorder << endl;
            } else {
                left = mid + 1;
            }
        }

        cout <<"leftBorder: " << leftBorder << endl;
        cout << "l: " << left << endl;
        cout << "r: " << right << endl;
        return leftBorder;
    }
};
```





### [69. x 的平方根](https://leetcode.cn/problems/sqrtx/)

关于获取int的最大值还可以用内置常量`INT_MAX`在库文件`<limits.h>`中。

`right`最终得到就是下取结果，而`left`得到是上取整结果。

```cpp
class Solution {
public:
    int mySqrt(int x) {
        int left = 0;
        int right = 0x7FFFFFFF;

        while (left <= right) {
            long long middle = (left + right) / 2; // middle用long,可能会溢出
            if (middle * middle > x) {
                right = middle - 1;
            } else if (middle * middle < x) {
                left = middle + 1;
            } else {
                return middle;
            }
        }

        return right;
    }
};
```

关于位运算，我们在程序中写的就是以补码的形式给计算机的。经过测试可以得到结论，正数原码取反+1可以得到其负数。

```cpp
// 二进制
int y = (1 << 31) - 0b00000000000000000000000000000010;
int x = (1 << 31) - 1;
// int x = 0b10000000000000000000000000000000; // int表示的最大负数
// int x = 0b11111111111111111111111111111111; // int表示的最大小数
// int x = 0b00000000000000000000000000001001; // +9二进制
// int x = 0b11111111111111111111111111110111; // -9二进制
cout << "x = " << x << endl;
cout << "y = " << y << endl;
```



### [367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/)

**方法一：二分**

- 时间复杂度：$O(logn)$，其中 $n$ 为正整数 $num$ 的最大值。
- 空间复杂度：$O(1)$。

一个数的的平方根小于等于 `(x / 2) + 1` ，当x=1时就是一个极端情况，所以加1不能省。

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        int left = 0;
        int right = (num / 2) + 1;
        while (left <= right) {
            long long mid = (left + right) >> 1;
            long long square = mid * mid;
            if (square > num) {
                right = mid - 1;
            } else if (square < num) {
                left = mid + 1;
            } else {
                return true;
            }
        }
        return false;
    }
};
```



**方法二：数学规律**

* 时间复杂度：$O(\sqrt{n})$ 。
* 空间复杂度：$O(1)$。

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        int num1 = 1;
        while (num > 0) {
            num -= num1;
            num1 += 2;
        }
        return num == 0;
    }
};
```



**方法三：暴力**

* 时间复杂度：$O(\sqrt{n})$ ，其中 $n$ 为正整数 $num$ 的最大值。我们最多需要遍历 $n+1$ 个正整数。
* 空间复杂度：$O(1)$ 。

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        long long x = 1, square = 1;
        while (square <= num) {
            if (square == num) {
                return true;
            }
            x ++ ;
            square = x * x;
        }
        
        return false;
    }
};
```



## 2. 移除元素

### [27. 移除元素](https://leetcode.cn/problems/remove-element/)

**方法一：暴力**

* 时间复杂度：$O(n^{2})$ 
* 空间复杂度：$O(1)$ 

`vector`当数组用就行，每次遍历到目标值就把数组中所有数往前覆盖一个

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int size = nums.size();
        for (int i = 0; i < size; i ++ )
            if (nums[i] == val)
            {
                for (int j = i + 1; j < size; j ++ ) nums[j - 1] = nums[j];
                i -- ;
                size -- ;
            }
        
        return size;
    }
};
```

**方法二：双指针**

* 时间复杂度：$O(n)$ 
* 空间复杂度：$O(1)$ 

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex < nums.size(); fastIndex ++ )
        {
            if (val != nums[fastIndex])
            {
                nums[slowIndex ++ ] = nums[fastIndex];
            }
        }

        return slowIndex;
    }
};
```



### [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

**方法一：双指针**

- 时间复杂度：*O*，其中 *n* 是数组的长度。快指针和慢指针最多各移动 *n* 次。
- 空间复杂度：*O*(1)。只需要使用常数的额外空间。

删除重复的数要避免第0个数就是，且判断是否通过的循环条件是`i < slow`，所以`slow`和`fast`最好都从1开始，而且用`fast - 1`作为判断条件避免溢出

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int slow = 1;
        for (int fast = 1; fast < nums.size(); fast ++ ) {
                if (nums[fast - 1] != nums[fast]) {
                nums[slow ++ ] = nums[fast];
            }
        }        
        
        return slow;
    }
};
```



**方法二：暴力**

* 时间复杂度：$O(n^{2})$ 
* 空间复杂度：$O(1)$ 

不改变数组大小的情况下。当出现重复的就把后面所有的覆盖前面一个，就是数组中的所有元素往前移动一位

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int size = nums.size();
        for (int i = 1; i < size; i ++ )
        {
            if (nums[i - 1] == nums[i])
            {
                for (int j = i - 1; j < size; j ++ )
                {
                    if (j + 1 < size){
                        nums[j] = nums[j + 1];
                    }
                }

                i -- ;
                size -- ;
            }
        }
        return size;
    }
};
```



### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

**方法一：暴力**

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int size = nums.size();
        for (int i = 0; i < size; i++) {
            int flag = 0;
            if (nums[i] == 0) {
                flag = nums[i];
                for (int j = i; j + 1 < size; j ++ ){
                    nums[j] = nums[j + 1];
                }
                nums[size - 1] = flag;
                size -- ;
                i -- ;
            }
        }
    }
};
```



**方法二：双指针 1.0**

快慢指针先把非0的排一遍，再把剩余的部分置上0

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int size = nums.size();
        int slow = 0;
        int cnt = 0;
        for (int fast = 0; fast < size; fast ++ ){
            if (nums[fast] != 0){
                nums[slow] = nums[fast];
                slow ++ ;
            } else if (nums[fast] == 0) {
                cnt ++ ;
            }
        }
        
        for (int i = slow; i < slow + cnt; i ++ )
        {
            nums[i] = 0;
        }
    }

};
```



**方法三：双指针2.0**

类似快排的思想，中间点就是0，把不等于0的放左边，等于0的放右边。不等于0的时候就自己和自己交换，等于零的时候就跳过，且慢指针会指到最新一个在0的位置然后交换

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int j = 0;
        for (int i = 0; i < nums.size(); i ++ ) {
            if (nums[i] != 0) {
                int tmp = nums[i];
                nums[i] = nums[j];
                nums[j ++ ] = tmp;
            }
        }
    }
};
```


用库函数`swap` 交换

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int size = nums.size();
        int slow = 0, fast = 0;
        while (fast < size) {
            if (nums[fast] != 0) {
                swap(nums[slow], nums[fast]);
                slow ++ ;
            }
            fast ++ ;
        }
    }
};
```



### [844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/description/)

不等于退格符的时候慢指针要自减，并且要判断慢指针是否大于0，以免数组越界

```cpp
class Solution {
public:
    bool backspaceCompare(string s, string t) {
        int len_s = s.length(), len_t = t.length();
        int fast_s = 0, slow_s = 0, fast_t = 0, slow_t = 0;

        while (fast_s < len_s) {
            if (s[fast_s] != '#') {
                s[slow_s ++ ] = s[fast_s];
            } else if (slow_s > 0) {
                slow_s -- ;
            }
            fast_s ++ ;
        }

        while (fast_t < len_t) {
            if (t[fast_t] != '#') {
                t[slow_t ++ ] = t[fast_t];
            } else if (slow_t > 0){
                slow_t -- ;
            }
            fast_t ++ ;
        }

        s = s.substr(0, slow_s);
        t = t.substr(0, slow_t);
        cout << s << endl;
        cout << t;

        return s == t;
    }
};
```



## 3. 有序数组的平方

### [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/)

**方法一：暴力排序**

* 时间复杂度$O(n + nlog{n})$ 

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i ++ ) nums[i] *= nums[i];
        sort(nums.begin(), nums.end());
        return nums;
    }
};
```



**方法二：双指针**

* 时间复杂度：$O(n)$ 

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int size = nums.size();
        vector<int> res(size, 0);
        int i = 0, j = size - 1, k = size - 1;
        while (i <= j) {
            if (nums[i] * nums[i] > nums[j] * nums[j]) {
                res[k] = nums[i] * nums[i];
                i ++ ;
                k -- ;
            } else {
                res[k] = nums[j] * nums[j];
                j -- ;
                k -- ;
            }
        }
        return res;
    }
};
```



## 4. 长度最小的子数组

### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

**方法一：暴力**

* 时间复杂度：$O(n^{2})$ 
* 空间复杂度：$O(1)$ ，没有改变空间大小，只是对数据进行操作

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int res = INT_MAX;
        int sum = 0;
        int subLength = 0;
        for (int i = 0; i < nums.size(); i ++ ) {
            sum = 0;
            for (int j = i; j < nums.size(); j ++ ) {
                sum += nums[j];
                // 因为是连续的所以一旦大于就不用再加了
                if (sum >= target) {
                    subLength = j - i + 1;
                    res = res < subLength ? res : subLength;
                    break;
                }
            }
        }

        return res == INT_MAX ? 0 : res;
    }
};
```



**方法二：双指针（移动窗口）**

* 时间复杂度：$O(n)$ ，不要看有while就是$O(n^{2})$ ，要看元素的整体操作次数。每个元素滑动窗口后进来一次出去一次，一共最多就是2n次，所以是$O(n)$ 的。
* 空间复杂度：$O(1)$ 

精髓在于`sum -= nums[i ++ ]`一旦大于目标值，就更新初始坐标和sum值。
while循环在条件终止的时候sum<target了，如果此时结果sum-=nums[i++]会不会有影响？其实是不会的，以数据`[3,1,1]`，目标值5为例，结束的时候虽然走了while循环但是res已经记录更新了，所以最后即使sum=5-3=2也没有影响，返回的结果仍然是res=2-0+1=3，如果数据是`[3,1,1,4]`那么sum的变化就是sum=2, sum=6, sum=5, sum=4最终循环结束的时候就是sum<target而此时res已经更新了

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int res = INT_MAX;
        int sum = 0;
        int subLength = 0;
        int i = 0;
        for (int j = 0; j < nums.size(); j ++ ) {
            sum += nums[j];
            while (sum >= target) {
                subLength = j - i + 1;
                res = res < subLength ? res : subLength;
                sum -= nums[i ++ ];
            }
        }

        return res == INT_MAX ? 0 : res;
    }
};
```



### [904.水果成篮](https://leetcode.cn/problems/fruit-into-baskets/)

* 时间复杂度：$O(n)$ 
* 空间复杂度：$O(1)$ 

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int size = fruits.size();
        unordered_map<int, int> cnt;

        int ans = 0, left = 0;
        for (int right = 0; right < size; right ++ ) {
            cnt[fruits[right]] ++ ; // 增加当前水果计数

            // 当窗口内水果种类超过2种时，开始收缩窗口的左边界
            while (cnt.size() > 2) {
                unordered_map<int, int>::iterator it = cnt.find(fruits[left]);
                it->second -- ; // 减少当前窗口左侧水果的数量
                if (it->second == 0) {
                    cnt.erase(it); // 如果数量减少到0，删除该键值对
                }
                
                left ++ ; // 移动左边界
            }

            ans = max(ans, right - left + 1); // 更新最大值
        }

        return ans;
    }
};
```



### [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/submissions/573883022/)

* 如果把`s = "ADOBECODEBANC", t = "ABC"` 改成`s="ADOBECODEBNC"`把A删去一个，那么while就不会再找到，也就是说ans_right和ans_left的值被永远保存了，除非后面有再符合的字符，很巧妙。
* 其次if判断也是有必要的。后续找到的子串长度要是大于前先找到的就不更新。
* ans_left初始值设为-1是为了标记不存在的情况，如果用0为初始值的话假设s和t都是一个字符，则在输出的时候会用到0就会造成麻烦。也不用担心if判断中的情况，因为在找到一次值后他们就会被更新掉，不会产生影响

```cpp
class Solution {
    bool is_convered(int cnt_s[], int cnt_t[]) {
        for (int i = 'A'; i <= 'Z'; i ++ ){
            if (cnt_s[i] < cnt_t[i]) return false;
        }

        for (int i = 'a'; i <= 'z'; i ++ ) {
            if (cnt_s[i] < cnt_t[i]) return false;
        }

        return true;
    }

public:
    string minWindow(string s, string t) {
        int size = s.length();
        int ans_left = -1, ans_right = size;        
        int cnt_s[128] = {}, cnt_t[128] = {};

        for (char c : t) cnt_t[c] ++ ;

        int left = 0;
        for (int right = 0; right < size; right ++ ) {
            cnt_s[s[right]] ++ ;
            while (is_convered(cnt_s, cnt_t)) {
                if (right - left + 1 <= ans_right - ans_left + 1) {
                    ans_right =  right; cout << "ans_right = " << ans_right << endl;
                    ans_left = left; cout << "ans_left = " << ans_left << endl;
                }

                cnt_s[s[left]] -- ;
                left ++ ;
            }
        }

       return ans_left < 0 ? "" : s.substr(ans_left, ans_right - ans_left + 1);
    }
};
```

## 5. 螺旋矩阵II

### [59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

* 时间复杂度：$O(n^2)$ ：模拟遍历二维矩阵的时间。
* 空间复杂度：$O(n^2)$ ：这段代码的空间复杂度不是 *O*(1)，而是*O*(*n*2)，因为需要一个 *n*×*n* 的矩阵来存储结果。

1. 如果n是奇数的话，(n/2, n/2)就是中心点。
2. 当n为偶数时循环次数是n/2；当n为奇数时循环次数也是n/2，不过中心点需要自己手动计入。

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n, vector<int>(n, 0));
        int start_x = 0, start_y = 0;
        int loop = n / 2;
        int mid = n / 2;
        int offset = 1;
        int cnt = 1;
        int i, j;
        while (loop -- ) {
            i = start_x;
            j = start_y;

            for (j; j < n - offset; j ++ )
            {
                res[i][j] = cnt ++ ;
            }

            for (i; i < n - offset; i ++ )
            {
                res[i][j] = cnt ++ ;
            }

            for (j; j > start_y; j -- ) {
                res[i][j] = cnt ++ ;
            }

            for (i; i > start_y; i -- ) {
                res[i][j] = cnt ++ ;
            }

            start_x ++ ;
            start_y ++ ;

            offset ++ ;
        }

        if (n % 2) res[mid][mid] = cnt; 
        return res;
    }
};
```



### [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

* 时间复杂度：$O(mn)$ ，m、n是矩阵的长宽。
* 空间复杂度：$O(mn)$ 

思路就是每走一圈就把四条边更新一次，当边界交叉时（l > r, u > d）循环就结束，也就循环完毕得到答案了。

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> ans;
        if (matrix.empty()) return ans;

        int u = 0;
        int d = matrix.size() - 1;
        int l = 0;
        int r = matrix[0].size() - 1;
        
        while (true)
        {
            for (int i = l; i <= r; i ++ ) ans.push_back(matrix[u][i]);
            if ( ++ u > d) break;
            for (int i = u; i <= d; i ++ ) ans.push_back(matrix[i][r]);
            if ( -- r < l) break;
            for (int i = r; i >= l; i -- ) ans.push_back(matrix[d][i]);
            if ( -- d < u) break;
            for (int i = d; i >= u; i -- ) ans.push_back(matrix[i][l]);
            if ( ++ l > r) break;
        }

        return ans;
    }
};
```



## 6. 区间和

[题目链接](https://kamacoder.com/problempage.php?pid=1070)

* 时间复杂度：$O(n + m)$ ，n是数组长度，m是查询次数。后续的查询都是$O(1)$ 的。
* 空间复杂度：$O(n)$ 构建前缀和p[]的大小。

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    int n, a, b;
    cin >> n;
    vector<int> p(n);
    vector<int> ans(n);
    
    int presum = 0;
    for (int i = 0; i < n; i ++ )
    {
        cin >> ans[i];
        presum += ans[i];
        p[i] = presum;
    }
    
    while (cin >> a >> b) {
        int sum;
        if (a == 0) sum = p[b]; // 由于下面有用到a-1, 所以对a进行特殊判定
        else sum = p[b] - p[a - 1];
        cout << sum << endl;
    }
}
```



## 7. 开发商购买土地

[题目链接](https://kamacoder.com/problempage.php?pid=1044)

* 时间复杂度：$O(nm) + O(nm)+O(nm)=O(3nm)=O(nm)$ 
* 空间复杂度：$O(n \times m)$ 

题目分析：题中说了“只允许将区域按横向或纵向划分成两个子区域”就是要么横着一刀切，要么竖着一刀切，别被后面一句”而且每个子区域都必须包含一个或多个区块“误导了，毕竟前面说了只能分成两个子区域。

```cpp
#include <iostream>
#include <vector>
#include <math.h>
#include <climits>

using namespace std;

int main()
{
    int n, m;
    cin >> n >> m;
    vector<vector<int>> vec(n, vector<int>(m, 0));
    int sum_value = 0;
    for (int i = 0; i < n; i ++ )
    {
        for (int j = 0; j < m; j ++ )
        {
            cin >> vec[i][j];
            sum_value += vec[i][j];
        }
    }
    
    int res = INT_MAX;
    
    int cnt = 0;
    for (int i = 0; i < n; i ++ ) 
    {
        for (int j = 0; j < m; j ++ )
        {
            cnt += vec[i][j];
            // pre_horizontal[i]错在是每一行的和, 并没有叠加上一行的和
            if (j == m - 1) res = min(res, abs(sum_value - cnt - cnt)); 
        }
    }
    
    cnt = 0;
    for (int j = 0; j < m; j ++ )
    {
        for (int i = 0; i < n; i ++ )
        {
            cnt += vec[i][j];
            if (i == n - 1) res = min(res, abs(sum_value - cnt - cnt));
        }
    }
    
    cout << res << endl;
    return 0;
}
```



# 链表

## 1. 移除链表元素

### [203. 移除链表元素](https://leetcode.cn/problems/remove-linked-list-elements/)

**方法一：虚拟头结点**

* 时间复杂度：O(n)
* 空间复杂度：O(1)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* cur = dummyHead;
        while(cur->next != nullptr)
        {
            if (cur->next->val == val)
            {
                ListNode* tmp = cur->next;
                cur->next = cur->next->next;
                delete tmp;
            }
            else
            {
                cur = cur->next;
            }
        }

        head = dummyHead->next;
        delete dummyHead;
        return head;
    }
};
```



**方法二：递归**

递归结束标志就是head==nullptr，也就是直到最后一个结构体的后面，此时返回给倒数第二层head->next=最后一层，拿当节点head的val判断是否等于val，如果等于返回刚刚递归得到的结果就是head->next，反之返回当前节点

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        if (head == nullptr) return head;
        head->next = removeElements(head->next, val);
        return head->val == val ? head->next : head;
    }
};
```



## 2. 设计链表

### [707. 设计链表](https://leetcode.cn/problems/design-linked-list/)

* `addAtTail` 。在尾部插入节点就直接 while cur->next != nullptr

* 虚拟表头head，本身就是个val=0,next=nullptr的指针结构体

* `addAtTail` ，循环条件是cur->next != nullptr，当等于的时候就说明走到了最后一个位置，这时就可以插入了

* `addAtIndex`。用while index -- ，增加一个节点在index之前，cur等于_dummyHead，所以第一次走到下一个就是头结点。比如index=1，cur就走一次即指到head，下标的0，后插入，也即1之前
* `deleteAtIndex`，同理，删除节点函数也是走到要删除位置的前一个点

```cpp
class MyLinkedList {
public:
  struct LinkedNode {
        int val;
        LinkedNode* next;
        LinkedNode(int val):val(val), next(nullptr){}
    };

    MyLinkedList() {
        _dummyHead = new LinkedNode(0);
        _size = 0;
    }
    
    int get(int index) {
        if (index > (_size - 1) || index < 0) return -1;
        LinkedNode* cur = _dummyHead->next;
        while (index -- ) cur = cur->next;
        return cur->val;
    }
    
    void addAtHead(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        newNode->next = _dummyHead->next;
        _dummyHead->next = newNode;
        _size ++ ;
    }
    
    void addAtTail(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while (cur->next != nullptr) {
            cur = cur->next;
        }
        cur->next = newNode;
        _size ++ ;
    }
    
    void addAtIndex(int index, int val) {
        if (index > _size) return;
        if (index < 0) index = 0;

        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while (index -- ) {
            cur = cur->next;
        }
        newNode->next = cur->next;
        cur->next = newNode;
        _size ++ ;
    }
    
    void deleteAtIndex(int index) {
        if (index > _size - 1 || index < 0) return;

        LinkedNode* cur = _dummyHead;
        while (index -- ) cur = cur->next;
        LinkedNode* tmp = cur;
        cur->next = cur->next->next;
        _size -- ;
    }

    void printLinkedNode(int x) {
        LinkedNode* cur = _dummyHead;
        while (cur->next != nullptr) {
            cout << cur->next->val << endl;
            cur = cur->next;
        }
        cout << endl;
    }
private:
    int _size;
    LinkedNode* _dummyHead;
};


/**
 * Your MyLinkedList object will be instantiated and called as such:
 * MyLinkedList* obj = new MyLinkedList();
 * int param_1 = obj->get(index);
 * obj->addAtHead(val);
 * obj->addAtTail(val);
 * obj->addAtIndex(index,val);
 * obj->deleteAtIndex(index);
 */
```



## 3. 翻转链表

### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

**方法一：双指针**

* 时间复杂度：O(n)。和链表长度成正比，如果链表长n，那时间复杂度就是O(n)的
* 空间复杂度：O(1)。使用的额外空间不随链表长度而变化，一些变量如pre、cur等都是占固定内存大小

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr;
        ListNode* cur = head;
        ListNode* tmp = nullptr;
        while (cur != nullptr) {
            tmp = cur->next;
            cur->next = pre;
            pre = cur;
            cur = tmp;
        }

        return pre;
    }
};
```



**方法二：递归**

* 时间复杂度：O(n)
* 空间复杂度：O(n)

1. 最终返回的结果其实就是pre，指向反转后链表的头部
2. ` if (cur == nullptr) return pre;` 为什么不写 cur->next == nullprt？会报错`runtime error: member access within null pointer of type 'ListNode'` 因为cur已经是一个空指针了再在空指针中访问成员就会报错

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverse(ListNode* pre, ListNode* cur) {
        if (cur == nullptr) return pre;
        ListNode* tmp = cur->next;
        cur->next = pre;

        ListNode* reverse_head = reverse(cur, tmp);
        return reverse_head;
    }

    ListNode* reverseList(ListNode* head) {
        return reverse(nullptr, head);
    }
};
```



## 4. 两两交换链表中的节点

### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

* 时间复杂度：O(n)
* 空间复杂度：O(1)

每次循环结束，cur直到下一轮要交换点的前一个点，作为“头节点”。比如第一轮结束后指向`1`，所以循环里的条件的是cur->next != nullptr，而不是cur != nullptr

现在cur1->next 是指2号点，再加一个->next就是它的->next属性指到的是3的地址，改成指向1的地址，没问题

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode* dumyHead = new ListNode(0);
        dumyHead->next = head;
        ListNode* cur = dumyHead;

        while(cur->next != nullptr && cur->next->next != nullptr) {
            ListNode* tmp = cur->next;
            ListNode* tmp1 = cur->next->next->next;

            cur->next = cur->next->next;
            cur->next->next = tmp;
            cur->next->next->next = tmp1;

            cur = cur->next->next;
        }        

        ListNode* res = dumyHead->next;
        return res;
    }
};
```



## 5. 删除链表的倒数第N个节点

### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* fast = dummyHead;
        ListNode* slow = dummyHead;
        while (n -- )
        {
            fast = fast->next;
        }
        fast = fast->next;

        while (fast != nullptr)
        {
            slow = slow->next;
            fast = fast->next;
        }
        slow->next = slow->next->next;
        return dummyHead->next;
    }
};
```



## 6. 链表相交

### [面试题 02.07. 链表相交](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        int lenA = 0, lenB = 0;
        ListNode* curA = headA;
        ListNode* curB = headB;

        while (curA != nullptr)
        {
            lenA ++ ;
            curA = curA->next;
        }

        while (curB != nullptr)
        {
            lenB ++ ;
            curB = curB->next;
        }

        // 重置cur的位置
        curA = headA;
        curB = headB;
        if (lenA < lenB)
        {
            swap(lenA, lenB);
            swap(curA, curB);
        }

        int diff = lenA - lenB;
        while (diff -- )
        {
            curA = curA->next;
        }

        while (curA != nullptr)
        {
            if (curA == curB) return curA;
            curA = curA->next;
            curB = curB->next;
        }

        return nullptr;
    }
};
```



## 环形链表

### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

while循环里的第二个条件写，fast->next != NULL 就行了，因为没有fast->next自然也就没有fast->next->next了，直接写fast->next->next会有空指针异常

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while (fast != NULL && fast->next != NULL) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast) {
                ListNode* index1 = fast;
                ListNode* index2 = head;
                while (index1 != index2) {
                    index1 = index1->next;
                    index2 = index2->next;
                }
                return index1;
            }
        }
        return NULL;
    }
};
```



# 哈希表

## 1. 有效的字母异位词

### [242. 有效的字母异位词](https://leetcode.cn/problems/valid-anagram/)

**方法一：暴力**

直接比较两字符串。不过我这个写法有问题，处理不了 s=aa t=a 的情况

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        int len_s = s.size();
        int len_t = t.size();

        if (len_s < len_t) {
            swap(len_s, len_t);
            swap(s, t);
        }

        bool flag;
        for (int i = 0; i < len_s; i ++ ) {
            flag = false;
            for (int j = 0; j < len_t; j ++ ) {
                if (s[i] == t[j]) flag = true;
            }

            if (flag == false) return flag;
        }

        return flag;
    }
};
```



**方法一：哈希表**

* 时间复杂度：O(n)
* 空间复杂度：O(1)

1. 记录每个字符串中字母出现的次数

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        int len_s = s.size();
        int len_t = t.size();

        if (len_s < len_t) {
            swap(len_s, len_t);
            swap(s, t);
        }

        int sign_s[26] = {0};
        int sign_t[26] = {0};

        for (int i = 0; i < len_s; i ++ ) {
            int idx = s[i] - 'a';
            sign_s[idx] ++ ;     
        }

        for (int i = 0; i < len_t; i ++ ) {
            int idx = t[i] - 'a';
            sign_t[idx] ++ ;     
        }

        for (int i = 0; i < 26; i ++ ) {
            if (sign_s[i] != sign_t[i]) return false;
        }

        return true;
    }
};
```



2. 优化写法

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        int record[26] = {0};
        for (int i = 0; i < s.size(); i ++ ) {
            record[s[i] - 'a'] ++ ;
        }

        for (int i = 0; i < t.size(); i ++ ) {
            record[t[i] - 'a'] -- ;
        }

        for (int i = 0; i < 26; i ++ )
        {
            if (record[i] != 0) return false;
        }

        return true;
    }    
};
```



### [383. 赎金信](https://leetcode.cn/problems/ransom-note/)

**方法一：暴力**

双重循环，找到一样的字符就删去，最终如果长度为0说明符合

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        for (int i = 0; i < magazine.size(); i ++ ) {
            for (int j = 0; j < ransomNote.size(); j ++ ) {
                if (ransomNote[j] == magazine[i]) {
                    ransomNote.erase(ransomNote.begin() + j);
                    break;
                }
            }
        }

        if (ransomNote.size() == 0) return true;
        return false;
    }
};
```



**方法二：哈希表**

与上一道题不同，这里最后判断的时候是record[i] > 0，因为只要ransoNote小于等于magazine就可以

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int record[26] = {0};
        for (int i = 0; i < ransomNote.size(); i ++ )  {
            record[ransomNote[i] - 'a'] ++ ;
        }

        for (int i = 0; i < magazine.size(); i ++ )  {
            record[magazine[i] - 'a'] -- ;
        }
        
        for (int i = 0; i < 26; i ++ ) {
            if (record[i] > 0) return false;
        }

        return true;
    }
};
```



### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

**方法一：排序**

* 时间复杂度：O(nklogk)，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度。需要遍历 n 个字符串，对于每个字符串，需要 O(klogk) 的时间进行排序以及 O(1) 的时间更新哈希表，因此总时间复杂度是 O(nklogk)。
* 空间复杂度：O(nk)，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度。需要用哈希表存储全部字符串。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;
        for (string str : strs) {
            string key = str;
            sort(key.begin(), key.end());
            mp[key].push_back(str);
        }

        vector<vector<string>> ans;
        for (auto i = mp.begin(); i != mp.end(); i ++ ) {
            ans.push_back(i->second);
        }

        return ans;
    }
};
```



**方法二：计数**

* string字符串类型的数据可以使用push_back方法，作用是末尾添加字符
* 对于string字符串类型的数据，push_back一个int型数据时，如果是在ASCII码中是一个不可见的控制字符则会显示成\x01、\x02。不论如何，他们会形成一个独特的字符串，用来标记其中字符出现几次，类似a10b2c0d1
* `key.push_back(i + 'a');`每次key记录的都是26个字母出现的次数，是对strs字符串数组中的每一个字符串中每一个字母出现字数的统计

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;
        vector<vector<string>> ans;

        for (string str : strs) {

            int record[26] = {0};
            for (char c : str) {
                record[c - 'a'] ++ ;
            }

            string key = "";
            for (int i = 0; i < 26; i ++ ) {
                key.push_back(i + 'a');
                key.push_back(record[i]);
            }
            mp[key].push_back(str);
        }

        for(pair<string, vector<string>> x : mp) {
            ans.push_back(x.second);
        }

        return ans;
    }
};
```



mwa

```cpp
class Solution {
public:
    bool judge(string a, string b) {
        int record[26] = {0};
        for (int i = 0; i < a.size(); i ++ ) {
            record[a[i] - 'a'] ++ ;
        }

        for (int i = 0; i < b.size(); i ++ ) {
            record[b[i] - 'a'] -- ;
        } 

        for (int i = 0; i < 26; i ++ ) {
            if (record[i] != 0) return false;
        }

        return true;
    }

    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        int n = strs.size();
        vector<vector<string>> res;
        // vector<vector<string>> res(n, vector<string>(n, ""));
        // cout << strs.size();
        // if (strs.size() == 1) {
        //     res[0][0] = strs[0];
        //     return res;
        // }

        for (int i = 0; i < strs.size(); i ++) {
            for (int j = 1; j < strs.size(); j ++ ) {
                if (judge(strs[i], strs[j])) {
                    res.push_back(*(strs.begin() + j));
                    strs.erase(strs.begin() + j);
                }
            }
            res.push_back(strs[i]);
        }

        return res;
    }
};
```



### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

**方法一：滑动窗口 + 哈希表**

```cpp
class Solution {
public:
    bool judge(int cnt_s[], int cnt_p[]) {
        for (int i = 0; i < 26; i ++ ) {
            if (cnt_s[i] != cnt_p[i]) return false;
        }

        return true;
    }

    vector<int> findAnagrams(string s, string p) {
        int len_s = s.size(), len_p = p.size();
        int cnt_s[26] = {0}, cnt_p[26] = {0};
        vector<int> ans;	
	if (len_s < len_p) return ans;

        for (int i = 0; i < len_p; i ++ ) {
            cnt_p[p[i] - 'a'] ++ ;
        }

        for (int i = 0; i < len_p; i ++ ) {
            cnt_s[s[i] - 'a'] ++ ;
        }
	
	if (judge(cnt_s, cnt_p)) ans.push_back(0);

        for (int i = len_p; i < len_s; i ++ ) {
            cnt_s[s[i - len_p] - 'a'] -- ;
            cnt_s[s[i] - 'a'] ++ ;

            if (judge(cnt_s, cnt_p)) {
                ans.push_back(i - len_p + 1);
            }
            
   
        }

        return ans;
    }
};
```





mwa。不是时间超时就是空间超内存限制

```cpp
class Solution {
public:
    // 排序
    bool judge(string cp_s , string p) {
        sort(cp_s.begin(), cp_s.end());
        sort(p.begin(), p.end());
        return cp_s == p;
    }
    
    // 哈希表
    bool judge(string cp_s , string p) {
        unordered_map<string, int> mp;
        string key = cp_s;
        sort(key.begin(), key.end());
        mp[key] = 1;
        mp[p] ++ ;
        if (mp[key] > 1) return true;
        return false;
    }

	// 哈希表
    bool judge(string cp_s , string p) {
        int record[26] = {0};
        for (int i = 0; i < cp_s.size(); i ++ ) {
            record[cp_s[i] - 'a'] ++ ;
        }

        for (int i = 0; i < p.size(); i ++ ) {
            record[p[i] - 'a'] -- ;
        }

        for (int i = 0; i < 26; i ++ )
        {
            if (record[i] != 0) return false;
        }

        return true;
    }


    vector<int> findAnagrams(string s, string p) {
        int len_s = s.size();
        int len_p = p.size();
        vector<int> ans(0);

        int j = 0;
        for (int i = 0; i < len_s; i ++ ) {
            if (i - j + 1 == len_p) {
                string cp_s = s;
                cp_s = s.substr(j, len_p);

                bool flag = judge(cp_s, p);

                if (flag) {
                    ans.push_back(j);
                }
                j ++ ;
            }
        }

        return ans;
    }
};
```



## 2. 两个数的交集

### [349. 两个数组的交集](https://leetcode.cn/problems/intersection-of-two-arrays/)

**方法一：unorder_set**

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> result_set;
        unordered_set<int> nums_set(nums1.begin(), nums1.end());
        for (int num : nums2) {
            if (nums_set.find(num) != nums_set.end()) {
                result_set.insert(num);
            }
        }
    
        return vector<int>(result_set.begin(), result_set.end());
    }
};
```



**方法二：哈希表**

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> ans;
        int cnt[1010] = {0};
        for (int i = 0; i < nums1.size(); i ++ ) {
            cnt[nums1[i]] ++ ; // 方式二 cnt[nums1[i]] = 1
        }

        for (int i = 0; i < nums2.size(); i ++ ) {
            if (cnt[nums2[i]] != 0) { // 方式二 cnt[num2[i]] == 1
                ans.insert(nums2[i]);
            }
        }

        return vector<int> (ans.begin(), ans.end());
    }
};
```



### [350. 两个数组的交集 II](https://leetcode.cn/problems/intersection-of-two-arrays-ii/)

**方法一：哈希表 +unordered_set**

```cpp
class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        int cnt[1010] = {0};
        int cnt2[1010] = {0};
        vector<int> ans(0);

        for (int i = 0; i < nums1.size(); i++) {
            cnt[nums1[i]]++;
        }

        for (int i = 0; i < nums2.size(); i++) {
            cnt2[nums2[i]]++;
        }

        unordered_set<int> num1(nums1.begin(), nums1.end());
        unordered_set<int> num2(nums2.begin(), nums2.end());

        for (int num : num1) {
            if (num2.find(num) != num2.end()) {
                int tmp = min(cnt[num], cnt2[num]);
                while (tmp--)
                    ans.push_back(num);
            }
        }

        return ans;
    }
};
```



## 3. 快乐数

### [202. 快乐数](https://leetcode.cn/problems/happy-number/)

```cpp
class Solution {
public:
    int getSum(int n) {
        int sum = 0;
        while (n) {
            sum += (n % 10) * (n % 10);
            n /= 10;
        }

        return sum;
    }

    bool isHappy(int n) {
        unordered_set<int> set;
        while (1) {
            int sum = getSum(n);
            if (sum == 1) {
                return true;
            }

            if (set.find(sum) != set.end()) {
                return false;
            } else {
                set.insert(sum);
            }
            n = sum;
        }
    }
};
```



## 4. 两数之和

### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

**方法一：哈希表**

写法一
先判断 iter 是否存在如果不存在判断条件还写成 iter->second != 0 就是对一个不存在的pair取second操作，必然是错的

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> mp;
        for (int i = 0; i < nums.size(); i ++ ) {
            auto iter = mp.find(target - nums[i]);
            if (iter != mp.end()) {
                return {iter->second, i};
            }

            mp.insert(pair<int, int>(nums[i], i));
        }

        return {};
    }
};
```



写法二

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> ht;
        for (int i = 0; i < nums.size(); i ++ ) {
            auto iterator = ht.find(target - nums[i]);
            if (iterator != ht.end()) {
                return {iterator->second, i};
            }
            ht[nums[i]] = i;
        }

        return {};
    }
};
```



Wrong Answer
因为本题不仅需要查询target-nums[i]是否在集合中，还要知道该元素的下标，所以需要key value的形式存储

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_set<int> ans(nums.begin(), nums.end());
        vector<int> res;

        for (int i = 0; i < nums.size(); i ++ ) {
            if (ans.find(target - nums[i]) != ans.end()) {
                res.push_back(i);
            }
        }

        return res;
    }
};
```



## 5. 四数相加 II

### [454. 四数相加 II](https://leetcode.cn/problems/4sum-ii/)

```cpp
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int, int> umap;
        for (int a : nums1) {
            for (int b : nums2) {
                umap[a + b] ++ ;
            }
        }

        int count = 0;
        for (int c : nums3) {
            for (int d : nums4) {
                if (umap.find(0 - (c + d)) != umap.end()) {
                    count += umap[0 - (c + d)];
                }
            }
        }

        return count;
    }
};
```



## 6. 赎金信

### [383. 赎金信](https://leetcode.cn/problems/ransom-note/)

* 时间复杂度：O(n)
* 空间复杂度：O(1)

不要使用map，在本题的情况下map的空间消耗要比数组大一些，因为map要维护红黑树或者哈希表，还要做哈希函数，这是费时的，数据量打的话就能体现出差别了，所以这里使用数组更加简单有效

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int cnt1[26] = {0};
        int cnt2[26] = {0};
        for (int i = 0; i < ransomNote.size(); i ++ ) {
            cnt1[ransomNote[i] - 'a'] ++ ;
        }

        for (int i = 0; i < magazine.size(); i ++ ) {
            cnt2[magazine[i] - 'a'] ++ ;
        }

        for (int i = 0; i < 26; i ++ ) {
            if (cnt1[i] > cnt2[i]) return false;
        }
        
        return true;
    }
};
```



## 7. 三数之和

### [15. 三数之和](https://leetcode.cn/problems/3sum/)

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ans;
        sort(nums.begin(), nums.end());
        

        for (int i = 0; i < nums.size(); i ++ ) {
            if (nums[i] > 0) return ans;

            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int left = i + 1;
            int right = nums.size() - 1;
            while (left < right) {
                if (nums[i] + nums[left] + nums[right] > 0) right -- ;
                else if (nums[i] + nums[left] + nums[right] < 0) left ++ ;
                else {
                    ans.push_back(vector<int>{nums[i], nums[left], nums[right]});

                    while (left < right && nums[right] == nums[right - 1]) right -- ;
                    while (left < right && nums[left] == nums[left + 1]) left ++ ;

                    right -- ;
                    left ++ ;
                }
            }
        }

        return ans;
    }
};
```



## 8. 四数之和

### [18. 四数之和](https://leetcode.cn/problems/4sum/)

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> ans;
        sort(nums.begin(), nums.end());
        
        for (int k = 0; k < nums.size(); k ++ ){
            if (nums[k] > target && nums[k] > 0) break;

            if (k > 0 && nums[k] == nums[k - 1]) continue;

            for (int i = k + 1; i < nums.size(); i ++ ) {
                if (nums[k] + nums[i] > target && nums[k] + nums[i] > 0) break;

                if (i > k + 1 && nums[i] == nums[i - 1]) continue;

                int left = i + 1;
                int right = nums.size() - 1;
                while (left < right) {
                    if ((long) nums[k] + nums[i] + nums[left] + nums[right] > target) right -- ;
                    else if ((long) nums[k] + nums[i] + nums[left] + nums[right] < target) left ++ ;
                    else {
                        ans.push_back(vector<int>{nums[k], nums[i], nums[left], nums[right]});

                        while (left < right && nums[right] == nums[right - 1]) right -- ;
                        while (left < right && nums[left] == nums[left + 1]) left ++ ;

                        left ++ ;
                        right -- ;
                    }
                }
            }
        }

        return ans;
    }
};
```



# 字符串

## 1. 反转字符串

### [344. 反转字符串](https://leetcode.cn/problems/reverse-string/)

**方法一：使用swap()**

条件`i < size() / 2`可以换成`i < j`

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        for (int i = 0, j = s.size() - 1; i < s.size() / 2; i ++ , j -- ) {
            swap(s[i], s[j]);
        }
    }
};
```



**方法二：使用reverse**

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        reverse(s.begin(), s.end());
    }
};
```



## 2. 反转字符串 II

### [541. 反转字符串 II](https://leetcode.cn/problems/reverse-string-ii/)

**方法一**

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {
        for (int i = 0; i < s.size(); i += (2 * k)) {
            if (i + k <= s.size()) {
                reverse(s.begin() + i, s.begin() + i + k);
            } else {
                reverse(s.begin() + i, s.end());
            }
        }

        return s;
    }
};
```



**方法二**

手写反转函数

```cpp
class Solution {
public:
    void myReverse(string& s, int start, int end) {
        for (start, end; start < end; start ++ , end -- ) {
            swap(s[start], s[end]);
        }
    }

    string reverseStr(string s, int k) {
        for (int i = 0; i < s.size(); i += (2 * k)) {
            if (i + k <= s.size()) {
                myReverse(s, i, i + k - 1);
            } else {
                myReverse(s, i, s.size() - 1);
            }
        }

        return s;
    }
};
```



Wrong Answer

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {

        int cnt = 0, idx = 0, cyc = 0;
        for (int i = 0; i < s.size(); i ++ ) {
            cnt ++ ;
            if (cnt == 2 * k) {
                cout << "1" << endl;
                reverse(s.begin() + idx, s.begin() + idx + k);

                idx = i + 1;
                cnt = 0;
                cyc ++ ;

            } else if (cyc * 2 * k + 2 * k > s.size()) {
                cout << "yy" << endl;
                cout << idx;
                reverse(s.begin() + idx, s.end());
            }
        }
        
        return s;
    }
};
```



**Java**

解法3

```cpp
class Solution {
    public String reverseStr(String s, int k) {
        char[] ch = s.toCharArray();
        for (int i = 0; i < ch.length; i += (2 * k)) {
            if (i + k <= ch.length) {
                reverse(ch, i, i + k - 1);
            } else {
                reverse(ch, i, ch.length - 1);
            }
            
        }

        return new String(ch);
    }

    public void reverse(char[] ch, int i, int j) {
        for (; i < j; i++, j--) {
            char temp = ch[i];
            ch[i] = ch[j];
            ch[j] = temp;
        }
    }
}
```



## 3. 替换数字

### [54. 替换数字](https://kamacoder.com/problempage.php?pid=1064)

**方法一**

使用额外空间。string 重载了 “ + ” 所以可以写成 `ans += s[i]`，而vector却没有

```cpp
#include <iostream>

using namespace std;

bool judge(char c) {
    for (int i = 'a'; i <= 'z'; i ++ ) {
        if (c == i) {
            return true;
        }
    }
    return false;
}

int main(){
    string s;
    cin >> s;
    
    string ans = "";
    for (int i = 0; i < s.size(); i ++ ) {
        if (judge(s[i])) {
            ans += s[i];
        } else {
            ans += "number";
        }
    }
    
    cout << ans;
    return 0;
}
```



**方法二：双指针**

```cpp
#include <iostream>

using namespace std;

int main() {
    string s;
    while (cin >> s) {
        int sOldIndex = s.size() - 1;
        int count = 0;
        for (int i = 0; i < s.size(); i ++ ) {
            if (s[i] >= '0' && s[i] <= '9') {
                count ++ ;
            }
        }

        s.resize(s.size() + count * 5);
        int sNewIndex = s.size() - 1;
        while (sOldIndex >= 0) {
            if (s[sOldIndex] >= '0' && s[sOldIndex] <= '9') {
                s[sNewIndex -- ] = 'r';
                s[sNewIndex -- ] = 'e';
                s[sNewIndex -- ] = 'b';
                s[sNewIndex -- ] = 'm';
                s[sNewIndex -- ] = 'u';
                s[sNewIndex -- ] = 'n';
            } else {
                s[sNewIndex -- ] = s[sOldIndex];
            }
            sOldIndex -- ;
        }
    }
    
    cout << s;
    
    return 0;
}
```



# 动态规划

数字三角形

**问题描述**

给定一个如下图所示的数字三角形，从顶部出发，在每一结点可以选择移动至其左下方的结点或移动至其右下方的结点，一直走到底层，要求找出一条路径，使路径上的数字的和最大。

>   ​				7
>
>   ​			3	8
>
>   ​		8	1	0
>
>   ​	2	7	4	4
>
>   4	5	2	6	5



**输入格式**

第一行包含整数n，表示数字三角形的层数。

接下来n行，每行包含若干整数，其中第i行表示数字三角形第i层包含的整数。



**输出格式**

输出一个整数，表示最大的路径数字和。



**数据范围**

1≤n≤5001≤n≤500,
−10000≤三角形中的整数≤10000



**输入样例：**

>5
>
>7
>
>3	8
>
>8	1	0
>
>2	7	4	4
>
>4	5	2	6	5



**输出样例：**

> 30





```c
#include <stdio.h>

int max(int x,int y);

int main()
{
	
	int arr[500][500];
	int brr[500][500];
    
    //先初始化
    for (i=0;i<500;i)
	{
		for (j=0;j<=i;j)
		{
			brr[i][j]=0;
		}
	}
    
    //录入数字
	int n = 0;
	int i = 0;
	int j = 0;
	scanf("%d", &n);
	for (i = 0;i < n;i++)
	{
		for (j = 0;j <= i;j++)
		{
			scanf("%d", &arr[i][j]);
		}
	}
    
    //规划
	brr[0][0] = arr[0][0];
	for (i = 1;i < n;i++)
	{
		for (j = 0;j <= i;j++)
		{
			if (j == 0)
			{
				brr[i][j] = brr[i - 1][j]+ arr[i][j];
			}
			else if (j != 0 && j != i)
			{
				brr[i][j] = max(brr[i - 1][j - 1], brr[i - 1][j]) + arr[i][j];
			}
			else
			{
				brr[i][j] = brr[i - 1][j - 1] + arr[i][j];
			}
		}
	}
    
    //排序
    int t=0;
	for (j=0,i=1;i<n;i++,j++)
	{	
		if (brr[n-1][j]>=brr[n-1][j+1])
			{
				t=brr[n-1][j];
				brr[n-1][j]=brr[n-1][j+1];
				brr[n-1][j+1]=t;
			}
	}
	
	printf("%d",brr[n-1][n-1]);
}

int max(int x,int y)
{
  return x>y?x:y;
}
```



# 杂

每次递归调用都会在栈上分配一个新的栈帧。由于递归会调用 n 次（其中 n 是链表中节点的数量），因此递归栈的最大深度为 n。**递归调用栈** 是指在函数递归调用过程中，系统维护的一个数据结构，用于存储每个函数调用的状态和上下文信息。每次函数调用都会创建一个新的栈帧，栈帧中保存了该调用的局部变量、参数、返回地址等信息。

## 题单

**数组**

* [904.水果成篮](https://leetcode.cn/problems/fruit-into-baskets/)
* [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/submissions/573883022/)

**双指针**

- [27.移除元素](https://programmercarl.com/0027.移除元素.html)
- [15.三数之和](https://programmercarl.com/0015.三数之和.html)
- [18.四数之和](https://programmercarl.com/0018.四数之和.html)
- [206.翻转链表](https://programmercarl.com/0206.翻转链表.html)
- [142.环形链表II](https://programmercarl.com/0142.环形链表II.html)
- [344.反转字符串](https://programmercarl.com/0344.反转字符串.html)
- [54. 替换数字](https://kamacoder.com/problempage.php?pid=1064)

## 语法操作

### 数组、字符串

```cpp
#include <iostream>

using namespace std;

int main()
{
    // 数组 字符串
    // string s = "a";
    // cout << s.size() << " " << s.length() << endl;
    // s = s.substr(0,2);
    // cout << s;
    // cout << s[0];
    
    // erase()
    // string s = "abcedefg";
    // cout << s << endl;
    // s.erase(s.begin() + 2);
    // cout << s;

    // int arr[] = {1, 2};
    // cout << sizeof arr / sizeof arr[0];
    // cout << arr.size(); // 错
    // cout << strlen(arr); .// 错

    // int arr[10] = {0};
    // for (int i = 0; i < sizeof arr / sizeof arr[0]; i ++ ) cout << arr[i] << " ";
    
    // reverse 是左闭右开区间 [)
    // string str = "abcdefg";
    // reverse(str.begin(), str.begin() + 2);
    // cout << str;

    return 0;
}
```





### vector

```cpp
#include <vector>
#include <iostream>

using namespace std;

int main() {
    // 使用指定大小的构造函数初始化, 默认初始化为6个int类型的默认值，即0
    vector<int> res(6); 
    vector<int> res(6, -1);
    
    // 迭代器 
    // vector<int> arr = {1, 2, 3, 3, 5};
    // vector<int>::iterator x = arr.begin();
    // cout << *x;
    // arr.push_back(100);
    // arr[5] = 101;
    // for (int i = 0; i < arr.size(); i ++ ) cout << arr[i] << endl;
    
    // []加入的数不算size
    // vector<int> brr(0);
    // brr.push_back(2);
    // brr[0] = 1;
    // brr[1] = 2;
    // cout << brr.size() << endl;
    
    // 初始化二维vector
    // vector<vector<int>> res(n, vector<int>(n, 0));
    // 获取二维vector第二维的大小
    // int x = res[0].size();
    
    return 0;
}
```



### unordered_map

```cpp
#include <unordered_map>
int main() {
    unordered_map<int, int> my_hash_map;
    my_hash_map[1] = 10;
    my_hash_map[2] = 20;
    my_hash_map[3] = 30;

    unordered_map<int, int>::iterator it;
    
    it = my_hash_map.find(3);
    int x = it->second;
    int y = it->first;
    cout << x << " " << y << " " << &it << endl;

    for (pair<int, int> x : my_hash_map) cout << x.second << endl;

    it = my_hash_map.find(2);
    my_hash_map.erase(it);
    for (pair<int, int> x : my_hash_map) cout << x.second << endl;
	
    // vector容器不初始化大小不能通过下标索引和[]符号赋值
    unordered_map<int, int> hash_table;
    hash_table[0] = 1;
    int x = hash_table[0];
    cout << x;
}
```



### `<cmath> <math.h>`

```cpp
#include <iostream>
#include <math.h>

using namespace std;

int main()
{
    // 平方
    int x = 7;
    int pow_x = pow(x, 2);
    cout << pow_x << endl;
    
    // 平方根
    int sqrt_x = sqrt(pow_x);
    cout << sqrt_x << endl;
    
    return
}
```



### `<climits> <limits.h>` 

```cpp
#include <iostream>
#include <limits.h>
#include <climits>

using namespace std;

int main()
{
	// 极限
    cout << INT_MIN << endl;
    cout << INT_MAX << endl;
    int x = 0x7fffffff;
    cout << x;
}
```



### 单链表

```cpp
#include <iostream>
using namespace std;

// 定义 ListNode 结构体
struct ListNode {
    int val;  // 节点上存储的元素
    ListNode *next;  // 指向下一个节点的指针
    ListNode(int x) : val(x), next(NULL) {}  // 节点的构造函数
};

int main() {
    // 创建单个节点
    ListNode* head = new ListNode(1);  // 创建头节点，值为1
    ListNode* second = new ListNode(2);  // 创建第二个节点，值为2
    ListNode* third = new ListNode(3);  // 创建第三个节点，值为3

    // 将节点连接成链表
    head->next = second;  // 头节点指向第二个节点
    second->next = third;  // 第二个节点指向第三个节点

    // 遍历链表并输出节点值
    ListNode* current = head;  // 从头节点开始
    while (current != NULL) {

        cout << "自己地址: " << current << endl;
        cout << current->val << " ";  // 输出当前节点的值
        ListNode* tmp = current->next;
        cout << "next地址: " << tmp << endl;

        current = current->next;  // 移动到下一个节点
    }
    cout << endl;

    // 翻转链表
    // ListNode* pre = nullptr;
    // ListNode* tmp = head->next;
    // cout << "tmp = head->next: " << tmp << endl;

    // head->next = pre;
    // cout << "head->next = pre: " << head->next << endl;
    
    // cout << "tmp = head->next: " << tmp << endl;
    // pre = head;
    // head = tmp;



    // 调换节点
    // ListNode* dummy = new ListNode(0);

    // cout << "dummy: " << dummy << endl;
    // cout << "dummy->next: " << dummy->next << endl;
    // dummy->next = head;

    // ListNode* cur1 = dummy;
    // cout << "cur1: " << cur1 << endl;
    // cout << "cur1->next: " << cur1->next << endl;

    
    // ListNode* tmp = cur1->next;
    // cout << "tmp(1) = cur1->next " << tmp << endl;
    // cout << "cur1->next->next: " << cur1->next->next << endl;
    // ListNode* tmp1 = cur1->next->next->next;
    // cout << "tmp1(3) = cur1->next->next->next: " << tmp1 << endl;
    // cout << endl;

    // cur1->next = cur1->next->next;
    // cout << "cur1->next = cur1->next->next: " << cur1->next << endl;
    // cout << "tmp(1) = cur1->next " << tmp << endl;

    // cout << "cur1->next->next: " << cur1->next->next << endl;
    // cur1->next->next = tmp; 

    // cur1->next->next->next = tmp1; 

    // cur1 = cur1->next->next;



    return 0;
}
```





## 十大排序

**[977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/)**

**方法一：选择排序**

* 时间复杂度：$O(n^{2}$)
* 空间复杂度： $O(n)$ 在这个实现中，首先创建了一个新的向量 `res`，它的大小等于输入向量 `nums` 的大小。这表示我们需要额外的空间来存储 `res` 中的所有元素。由于 `res` 的大小直接依赖于输入向量 `nums` 的大小 `n`，因此我们说这个额外的空间需求是线性的，即 O(n)。

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int size = nums.size();
        vector<int> res(size, 0);
        
        for (int i = 0; i < size; i ++ ) res[i] = nums[i] * nums[i];
        
        for (int i = 0; i < size; i ++ ) {
            for (int j = i + 1; j < size; j ++ ) {
                if (res[i] >= res[j]) swap(res[i], res[j]);
            }
        }

        return res;
    }
};
```



**方法二：快速排序**

* 时间复杂度：$O(n)$ 

```cpp
class Solution {
public:
    void quick_sort(int l, int r, vector<int>& res) {
        if (l >= r) return;

        int x = res[l];
        int i = l - 1, j = r + 1;
        while (i < j) {
            do i ++ ; while (res[i] < x);
            do j -- ; while (res[j] > x);
            if (i < j) swap(res[i], res[j]);
        }

        quick_sort(l, j, res);
        quick_sort(j + 1, r, res);
    }

    vector<int> sortedSquares(vector<int>& nums) {
        int size = nums.size();
        vector<int> res(size, 0);
        for (int i = 0; i < size; i ++) res[i] = nums[i] *nums[i];
        quick_sort(0, size - 1, res);
        
        return res;
    }
};
```
