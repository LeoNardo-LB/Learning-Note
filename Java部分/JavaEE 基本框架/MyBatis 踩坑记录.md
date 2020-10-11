## 动态sql中的test

本想判断传入的text的值是否为空串

此处 `<if test="text != ''">` 为是否为空字符 `‘’`

需要修改成 `<if test='text != ""'>`

```xml
<select id="selectLimitPage" resultMap="userMap">
    select * from t_user
    <where>
        <if test='text != ""'>
            and login_account like '%${text}%'
        </if>
    </where>
    limit #{offset},#{size}
</select>
```

