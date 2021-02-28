---
layout: post
title: Spring jdbcTemplate.queryForList 报错：Incorrect column count:expected 1, actual 7
tags: Java基础
---
# 问题描述
记录一个坑，在工程之前封装了Dao层，时间过得有点久了，使用的时候遇到了一个报错信息

Caused by: org.springframework.jdbc.IncorrectResultSetColumnCountException: Incorrect column count: expected 1, actual 7
```java 
// 封装代码
public <T> List<T> queryForListClassType(String sql, Class<T> classType, Object... objects) {
        logger.debug("sql>>>" + sql + ",objects>>>" + getParasLog(objects));
        DBContextHolder.setDBType(DatabaseConfig.DATA_SOURCE_READER);
        return jdbcTemplate.queryForList(sql, classType, objects);
    }
   
 //调用代码
 String sql = "select * from t_pay_sandbox_user" where deleted ='0'";
 List<SandBoxUser> users = queryForListClassType(sql, SandBoxUser.class, null); 
```
比较坑的是，编译时通过的，并没有报错，但是 `jdbcTemplate.queryForList` 这个方法的实现接收的是一个基础的类型和String类型，`getSingleColumnRowMapper` 就是只能结果是只能有一列，

```java
	@Override
	public <T> List<T> queryForList(String sql, Class<T> elementType, @Nullable Object... args) throws DataAccessException {
		return query(sql, args, getSingleColumnRowMapper(elementType));
	}
```

# 解决方案
使用 `jdbcTemplate.query` 方法，封装Dao层如下，这样就可以把查询的结果映射到传入的Class类中了
```java
    public <T> List<T> queryForListEntityClass(String sql, Class<T> entityClass, Object... objects) {
        logger.debug("sql>>>" + sql + ",objects>>>" + getParasLog(objects));
        DBContextHolder.setDBType(DatabaseConfig.DATA_SOURCE_READER);
        return jdbcTemplate.query(sql, BeanPropertyRowMapper.newInstance(entityClass), objects);
    }
```