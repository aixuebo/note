# 背景与总结
## 1.cube模型有很多的问题
* 不好运维，大量的groupping set组合，根本不知道要修改哪块。
* 计算耗时久，影响业务就绪失效。
* 冗余的组合计算，业务用不到，白计算。
* 查询时，配置也麻烦，要注意什么场景要填充-1.
* 开发过程也有坑点，要确保在cube之前，字段都要有默认值。
* 数据不如明细准确率高，并且明细表容易排查问题。cube模型天然不容易排查问题。（生成的sql也不好排查，需要每一个维度都赋予-1，sql很乱）

# 一、如何利用doris的bitmap，提升查询性能的同时，减少cube的开发
## 1.将字符串转换成bigint，然后编码成密集型编码 或者bigint直接编码成密集型编码
编码参考 TODO
## 2.分桶不要设置太多，集中在20-50之间，并且要分散。确保一个数据快有300M
## 3.正交分桶的方式，减少shuffle传输bitmap大文件，而是传输去重复结果，然后在reduce阶段进行sum即可。
相关材料待整理完善，可未来搜索"正交分桶"

# 二、实践中会遇到的问题
## 1.业务需求就是需要维度组合多，比如组合到AOI+坑位粒度，那就很细了。
比如一个场景下，日粒度，去重复后的维度组合有1.5亿，那基本上不用cube是不可行的，doris的bitmap支持不了。

## 2.如何改进
* 在hive表中分析所有维度group by后，有多少条数据，找到影响最大的维度。
比如找到坑位和小时两个维度取消掉，就从1.5亿 --> 300万，这就是进步一大截。
* 如果有一些维度去不掉，业务就是会这么看数，那用rollup + 看板模块独立的方式支持。
比如明细模块可以支持坑位+小时粒度，而其他模块都不支持，因此其他模块查询性能天然就会快，因为他会走特定的rollup。

## 3.实践抓手指标
尽量控制dt表内，单rollup控制在1000万以内，最好300万-500万左右。

# 三、rollup原则
## 1.针对筛选器搜索维度进行配置。
因为rollup会生成新的key组合的聚合内容，因此如果不加rollup，假设有1000万数据量，那么加了之后，就只有20万的搜索条件聚合结果了，查询就会快一些。
## 2.有必要分析用户使用习惯，前缀匹配的方式创建rollup吗
没必要，除非明确用户高频查询条件。
因为走了rollup && 前缀索引有5万；走了rollup && 无前缀索引有20万。因此加不加rollup的性能提升很明显，前缀索引的提升就不明显了。
考虑顺序还得先去了解用户的查询习惯，比较费时间，roi较低，不建议做。
## 3.rollup的顺序如何控制
分区字段、核心常用的查询优先。其他的无需考虑顺序。

rollup 语法
ALTER TABLE table ADD ROLLUP rollupName(

)
