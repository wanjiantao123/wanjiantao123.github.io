> 时间：2020-12-27  16：33

# 关于Maybatis中的left jion问题

#### 需求：在供求信息页面进行全局模糊查询

XXXMapper.xml中代码：

```xml
<!--根据给定条件查询供求表中内容-->	
<select id="findAll" parameterType="cn.lz.management.suppy.model.SuppyModelVo" resultType="cn.lz.management.suppy.model.SuppyModelVo">
			SELECT
			s.id,
			s.name,
			u.user_name userName,
			s.create_time createTime,
			s.industry,
			s.status 
		    FROM
			supply_demand s
			LEFT JOIN user_info u ON u.id = s.create_user 
		    WHERE
			u.id = s.create_user and s.active = 1 
		  <if test="industry !=null and industry != ''">
		  	and s.industry= #{industry}
		  </if>
		  <if test="name !=null  and name != ''">
		  	and s.name like concat('%',#{name},'%')
		  </if>
	      <if test="status !=null  and status != ''">
	      	and s.status =#{status}
	      </if>
	      <if test="userName !=null  and userName != ''">
	      	and u.user_name like concat('%',#{userName},'%')
	      </if>
	      ORDER BY createTime DESC
	</select>
```

总结： LEFT JOIN 关键字会从左表 (supply_demand） 那里返回所有的行，即使在右表 (user_info) 中没有匹配的行。可以联想到： RIGHT JOIN 关键字会右表 (user_info) 那里返回所有的行，即使在左表 (supply_demand) 中没有匹配的行。  

具体的sql语句执行效果：

![1587628437072](images/1587628437072.png)

图_1 左连接效果