
# cypher导入csv文件
将节点csv文件和边集csv文件放于import文件夹中，
[csv文件读写](https://docs.python.org/zh-cn/3/library/csv.html)
```cypher
导入people.csv节点集  
load csv from "file:///people.csv" as line 
create (a:Author{name:line[0]})
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c29ee0163008467286daca2ad3ff48d3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
导入edges.csv边集
load csv from "file:///edges.csv" as line
match(a:Author),(b:Author)
where ID(a)=toInteger(line[0]) and ID(b)=toInteger(line[2])
create (a)-[r:friend{type:line[1]}]->(b)
return r
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/76250e6c56df4b7793479aa1ec7715b6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

# 函数ID()获取节点的id（neo4自动赋给节点的id）
![在这里插入图片描述](https://img-blog.csdnimg.cn/544c16865e8e4eed99a16e32a4cc1bb9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)


