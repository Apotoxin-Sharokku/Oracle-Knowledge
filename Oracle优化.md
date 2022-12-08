<a name="3d725f4d"></a>
# Oracle优化
<a name="FLWBs"></a>
## [1. 表结构优化](#3b14b513)
<a name="yd5xD"></a>
## [2. 分区表优化](#54bd2780)
<a name="plGEG"></a>
## [3. 索引优化](#NTNZo)
<a name="oKA9P"></a>
## [4. 代码优化](#seFDW)
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
&nbsp; 
---

<a name="3b14b513"></a>
### 表结构优化      
-- [get_back](#3d725f4d) -- 
> 设计表的时候，**尽量满足第三范式** 
当然有时候也可以 **为了效率降低要求，增加冗余用空间换时间** 
例如游戏商品表里面有四个字段:  
       +  游戏道具   
       +  单个售价   
       +  销售量     
       +  总金额     
其中总金额可以由单个售价和销售量相乘获得,为冗余字段 
但是这个字段可以提升查询速度,因此可根据需求做适当保留 
















<a name="54bd2780"></a>
### 分区表优化     
-- [get_back](#3d725f4d) --
分区表的类型可以分为以下五种：         
  - [范围分区](#dc5c9f03) -        
  - [列表分区](#48849fe0) -        
  - [组合分区](#d6ea6238) -        
  - [散列分区](#9b72e6d9) -        
  - [复合分区](#339fb890) -        
<a name="dc5c9f03"></a>
#### 范围分区
- [get_back](#54bd2780) -
> 对字段的值的范围进行划分 
> >>>>>>>>>>>>>>>>
> - 一般用于对《时间》进行范围划分        
可根据日、月、季、年进行划分        
> - 也可对《合格率》进行范围划分         
例如次日留存率，7日留存率          


<a name="48849fe0"></a>
#### 列表分区
- [get_back](#54bd2780) -
> 根据某个字段的具体值进行划分
> >>>>>>>>>>>>>>>>
> 例如：游戏版本、礼包名称


<a name="d6ea6238"></a>
#### 组合分区
- [get_back](#54bd2780) -
> 范围+列表的合成体
> >>>>>>>>>>>>>>>>
> 例：游戏类别 + 游戏具体名字


<a name="9b72e6d9"></a>
#### 散列分区
- [get_back](#54bd2780) -
> 是根据字段的hash值进行均匀分布 尽可能实现各分区所散列的数据相等


<a name="339fb890"></a>
#### 复合分区
- [get_back](#54bd2780) -
> 范围分区 + 散列分区 
> 只有一个大类，没有明显区分的小类
> >>>>>>>>>>>>>>>>
>  例：服务器人数分布
















<a name="NTNZo"></a>
### 索引优化
-- [get_back](#3d725f4d) --
索引的类型可以分为以下五种：

1. [唯一索引](#KMVJJ)
2. [普通索引](#gQ7V0)
3. [组合索引](#VZJ0m)
4. [位图索引](#s8JtB)
5. [函数索引](#PZ2Pk)
<a name="KMVJJ"></a>
#### 唯一索引
- [get_back](#NTNZo) -
> 当某列任意两行数据不相同
> >>>>>>>
>  uid


<a name="gQ7V0"></a>
#### 普通索引
- [get_back](#NTNZo) -
> 少量重复的数据 一般可满足大部分索引的需求


<a name="VZJ0m"></a>
#### 组合索引
- [get_back](#NTNZo) -
> 当多列经常一起出现在WHERE条件中，创建索引


<a name="s8JtB"></a>
#### 位图索引
- [get_back](#NTNZo) -
> 当一列有大量重复数据时建立
> >>>>>>>
> 省份、性别、游戏类型、游戏平台


<a name="PZ2Pk"></a>
#### 函数索引
- [get_back](#NTNZo) -
> 在WHERE 条件语句中包含函数或表达式时建立















<a name="seFDW"></a>
### 代码优化
-- [get_back](#3d725f4d) --

1. [避免索引失效](#pIeBV)
2. [Union 和 Union All](#D07C3)
3. [绑定变量](#HsG9k)
4. [查询避免使用 *](#Qlfx9)
5. [减少数据库访问次数](#RrxUU)
6. [用 where 替换 having](#aV0QB)
7. [执行计划优化](#ULYYc)
<a name="pIeBV"></a>
#### 1. 避免索引失效
- [get_back](#seFDW) -
**a. 避免隐式转换**
> 表的字段类型要与查询内容一致
> 例：id  varchar2(200)
>        where  id = 1234
>        string  >>   number


**b. 避免模糊查询 %在前**
>      -- %王冰冰%    索引失效
>      -- %王冰冰       索引失效
>      +  王冰冰%      索引不失效
> 可用 instr  替代 like
> 或用并行提高查询速度
> SELECT  /*+PARALLEL(employee,2)*/ 
>                  * 
> FROM employee 
> WHERE last_name LIKE '%c%


**c. 避免使用不等于操作符  <>  !=  **
> 用 or 语法替代不等号进行查询
> where id <> 100
>              ↓↓↓
> where id >100 or id<100


**d. 避免对索引使用计算或函数**
> + - * /     is null    is not null    
> or   连接的不是同一个字段，索引失效
> sum()   trunc()


**e. 组合索引使用遵循最左前缀原则**
> (类似过桥，桥头，桥身，桥尾)
> 范围索引后面的字段会失效
> index cctv ( id, name, job )
> ------
> where id=52 and name=wbb and job=jizhe
> where id=52 and job=jizhe
> where id>52 and name=wbb and job=jizhe
> where name=wbb and job=jizhe
> ------
> id, name, job 均走索引
> id 走索引
> id 走索引
> 均不走索引



<a name="D07C3"></a>
#### 2. Union 和 Union All
- [get_back](#seFDW) -
> 当where后面有or且多个不同字段时
> 索引无法使用
> 此时可将多个条件Union 或 Union All




<a name="HsG9k"></a>
#### 3. 绑定变量
- [get_back](#seFDW) -
> 可以将SQL语句写进存储过程，
> 在存储过程中使用变量，
> 来达到共享SQL语句的目的。
> 存储过程是已经编译过的SQL语句的集合，也可以达到减少编译时间的目的




<a name="Qlfx9"></a>
#### 4. 避免查询使用 *
- [get_back](#seFDW) -
> ORACLE数据库在解析的过程中，
> 会将 * 一次转换成所有的列名，
> 这个工作是通过查询数据字典完成的
> 这将增加额外资源消耗。




<a name="RrxUU"></a>
#### 5. 减少数据库访问次数
- [get_back](#seFDW) -
> 当执行每条SQL语句时，ORACLE数据库在内部执行了许多工作：
>        解析SQL语句，
>        估算索引的利用率，
>        绑定变量，
>        读取数据块等，
> 由此可见，减少访问数据库的次数，就能实际上减少ORACLE 数据库的工作量。
> >>>>>>>>>>
> 例：合成宽表
>         多条SQL合为一条



<a name="aV0QB"></a>
#### 6. 用 where 替换 having
- [get_back](#seFDW) -
> 避免使用 HAVING 子句，
> HAVING 只会在检索出所有记录之后才对结果集进行过滤
> 这个处理需要排序，总计等操作。
> 如果能通过 WHERE 子句限制记录的数目，那就能减少这方面的开销。




<a name="ULYYc"></a>
#### 7. 执行计划优化
- [get_back](#seFDW) -

