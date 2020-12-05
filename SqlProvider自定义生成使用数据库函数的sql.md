> 拼接sql，可能持久层框架不能提供拼接复杂的sql的方法，比如用到了数据库的函数


#### Dao中供service调用的方法

```java
    @SelectProvider(type = SqlProvider.class, method = "brandSelectListOrderByNameLength")
    List<BrandCategory> brandSelectListOrderByNameLength(@Param("page") DefaultPage page, @Param("criteria") GeneratedCriteria criteria, @Param("entity") BrandCategory entity);
```

#### 生成sql的暗箱操作
拼接想要的sql，比如这个length(列名)就不能在框架中拼接出来

在dao中添加内部类
```java
class SqlProvider extends BaseSqlProvider{
        public String brandSelectListOrderByNameLength(Map<String, Object> params){
            final String sql = "select * from " + Tables.BR_BRAND_CATEGORY + " where status = 1 ";
            return conditionHandler(sql, params).executeSQL().concat(" order by length(name) asc");
        }
    }
```