> 时间：2020-10-2  16：07

# MySQL的时间格式化相关

需求：根据用户权限  预约时间和会议时间的模糊查询条件查询预约、会议室、预约设备的全部信息

具体XXXMapper.xml的代码：

```xml
<select id="findreservation" parameterType="java.lang.String" resultMap="selectreservationResultMap">
	select
		mr.*,m.*,sbe.*
		from
		Meeting_Reservation mr,Meeting m,subscribe_equipment sbe
		where  
		
		
		mr.meetingId=m.resourceID and sbe.meetingResrvationId=mr.resourceID and mr.status=1
		
		
		<if test="createUser != null">
        and (mr.reservationTime>DATE_FORMAT(now(), '%Y-%m-%d'))and mr.createUser=#{createUser}  or (mr.reservationTime=DATE_FORMAT(now(), '%Y-%m-%d') 
        and
        STR_TO_DATE(mr.endTime,'%H:%i')>STR_TO_DATE(#{nowTime},'%H:%i')and mr.createUser=#{createUser} and mr.meetingId=m.resourceID and sbe.meetingResrvationId=mr.resourceID and mr.status=1)
        </if>
       
        <if test="reservationTime !=null">
        	and mr.reservationTime=#{reservationTime}
        </if>
        <if test="meetingName !=null">
        	and mr.meetingName Like CONCAT('%',#{meetingName},'%')
        </if>
         ORDER BY mr.reservationTime DESC,STR_TO_DATE(mr.startTime,'%H:%i') DESC

		
</select>
```

总结有STR_TO_DATE()和DATE_FORMAT()的Mysql函数，前一个是对VARCHAR类型的字符串进行格式化，后一个是对DATETIME类型的时间进行格式化，其中：

| 字符 |     表示      |
| ---- | :-----------: |
| %Y   |   年（4位）   |
| %m   |  月（00-12）  |
| %d   |  日（00-31）  |
| %H   | 小时（00-23） |
| %i   | 分钟（00-59） |
| %s   |  秒（00-59）  |

