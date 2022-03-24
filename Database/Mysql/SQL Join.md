#Mysql #SQL
# Join种类
*LEFT JOIN*、*RIGHT JOIN*、*INNER JOIN*、*OUTER JOIN*  的用法：
![[SQL Join的使用.png]]

# Group by
场景
- 患者(patient)和随访计划(plan)关联，一对多，也可能没有关联；
- 查询患者信息以及相关的计划信息（可能为 null）；
- 使用 left join 与 plan 关联，
