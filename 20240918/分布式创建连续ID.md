# 背景与总结
* 因为bitmap 如果是任意long,会造成数据稀松，不够密集，对bitmap存储占用较大，所以需要重新编码成连续的正数。
* 利用多节点并发的能力编码

# 一、计算每一个partition的中，id在该partition的序号 -- table 1
### 用户id、该用户所在partition、该用户id在该partition的排序序号
select id,
partition,
row_number()over(PARTITION BY partition ORDER BY id) seq
from
(
	### 计算id所归属的partition
	select id,abs(hash(id)%500) partition
	from
	(
		### 去重复
		SELECT id
		FROM biao
		GROUP BY 1
	) t
) t

# 二、计算每一个partition开始计数的基础结果 -- table2

### 计算每一个partition上一个位置的最大值，即该partition位置的累计开始位置
select partition,
lag(partition_sum_cnt) over(PARTITION BY 1 ORDER BY partition) base
from
(
	### 计算从0到N，截止到每一个partition时，该partition之前已经存储了多少条数据
	select partition,
	sum(partition_cnt) over(PARTITION BY 1 ORDER BY partition) partition_sum_cnt
	FROM 
	(
		### 计算每一个partition有多少条数据
		SELECT abs(hash(id)%500) partition,sum(1) partition_cnt
		FROM
		(
			### 去重复
			SELECT id
			FROM biao
			GROUP BY 1
		) t
		GROUP BY abs(hash(id)%500)
	) t
) t
		
# 三、组合sql
select /*+ mapjoin(table2) */ table1.id,
table1.seq + nvl(table2.base,0) new_id
from table1
join table2 on table1.partition = table2.partition