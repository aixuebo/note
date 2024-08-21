# 背景与总结


# 一、基础语法含义
from
(
) table1 lateral view explode(split('1,2,3,4',',')) table2 as later_view_col

注意
* table1表示from的表别名
* table2表示lateral view的表别名。
* later_view_col 表示table2的一个字段叫later_view_col

# 二、追加增加固定枚举值
## 1.膨胀5倍，为每一次碰撞追加一个枚举值，比如表示时段

select table1.poi_id,
table2.later_view_col
from
(
  select poi_id
  from table
  limit 2
) table1 lateral view explode(split('1,2,3,4',',')) table2 as later_view_col

