## 写在最前

可信考试科目一中经常会用到HashMap的使用场景，但C语言本身并不像Java或者Python语言一样，自带HashMap的实现API，但让我们每次遇到需要使用HashMap的场景时候都自己先构建一个Hash表结构也不太现实，目前通用的做法是，使用第三方库uthash，这个库在力扣和OJ上都是可以使用的。

## Uthash的使用

### 定义结构体

```C 
#include "uthash.h"
#include "uthash.h"
struct my_struct {
    int key;               
    int value;
    UT_hash_handle hh;     
};
/* 申明哈希表为NULL指针 */
struct my_struct *map = NULL; 
```

此处定义的结构体必须包含UT_hash_handle 成员，一般命名为hh，不需要初始化；

### 查找

使用HASH_FIND_INT查找表中元素，该宏需要传递三个参数

- 第一个参数users哈希表
- 第二个参数是**user_id的地址**
- 第三个参数是输出变量，返回表中能找到的对应成员的指针，找不到返回NULL

```C 
struct my_struct *find_user(int ikey) {  
    struct my_struct *s;  
HASH_FIND_INT(g_users, &ikey, s );  
return s;  
}  
```

### 新增

对于不同类型的成员，使用不同的宏来加进哈希表中，如下

| 宏           | 含义                     |
| ------------ | ------------------------ |
| HASH_ADD_INT | 添加的键值为int类型      |
| HASH_ADD_STR | 添加的键值为字符串类型   |
| HASH_ADD_PTR | 添加的键值为指针类型     |
| HASH_ADD     | 添加的键值可以是任意类型 |

具体使用方法如下：

```C 
void add_user(int user_id, char *name) {
    struct my_struct *s;
    /*重复性检查*/
    HASH_FIND_INT(users, &user_id, s); 
    if (s==NULL) {
      s = (struct my_struct *)malloc(sizeof *s);
      s->id = user_id;
      HASH_ADD_INT( users, id, s ); 
    }
    strcpy(s->name, name);
}
```

### 删除

要从哈希表中删除成员，先找到该成员是否存在于表中，然后使用HASH_DEL方式从表中移除成员，注意此时并不会释放该成员的内存。

```C 
void delete_user(int ikey) {  
    struct my_struct *s = NULL;  
    HASH_FIND_INT(users, &ikey, s);  
    if (s!=NULL) {  
      HASH_DEL(users, s);   
      free(s);              
    }  
}
```

### 统计个数

使用`HASH_COUNT`函数获取哈希表中元素个数

```C 
unsigned int num = HASH_COUNT(users); 
```

### 遍历

```C 
struct my_struct *s, *tmp;  
HASH_ITER(hh, users, s, tmp) {  
    printf("user ikey %d: value %s\n", s->ikey, s->value);  
    /* ... it is safe to delete and free s here */  
}
```

## 算法例题

学会了uthash 的使用方法后就可以愉快的在刷题的时候使用上了

### 两数之和

[1. 两数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/two-sum/)

使用UThash的题解如下：

```C 
struct HashTable{
    int key;
    int value;
    UT_hash_handle hh;
};

int* twoSum(int* nums, int numsSize, int target, int* returnSize){
    struct HashTable *map = NULL;
    for (int i = 0; i < numsSize; i++) {
        struct HashTable *temp = NULL;
        int number = target - nums[i];
        HASH_FIND_INT(map, &number, temp);
        if (temp == NULL) {
            temp = (struct HashTable *)malloc(sizeof(struct HashTable));
            temp->value = i;
            temp->key = nums[i]; 
            HASH_ADD_INT(map, key, temp);
        } else {
            int* res = malloc(2*sizeof(int));
            res[0] = i;
            res[1] = temp->value;
            *returnSize = 2;
            return res; 
        }
    }
    *returnSize = 0;
    return NULL;
}
```

