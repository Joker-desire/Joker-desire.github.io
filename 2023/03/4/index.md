# sqlalchemy新增数据后获取递增ID报错解决


## 需求

通过`Sqlalchemy ORM`进行新增数据，有个新的需求是：新增数据后，需要获取到当前数据自动递增的ID。

## 处理

1. 通过`sqlacodegen`生成Model文件

    ```python
    class DailyStatistic(ModelDB2):
        __tablename__ = 'daily_statistics'
    
        id = Column(INTEGER(11), primary_key=True, nullable=False)
        ...
    
    ```

2. 编写新增逻辑代码

    ```python
    ef add_by_schemas(db: Session, param: DailyStatisticSchemas):
        daily = DailyStatistic(**dict(param))
        db.add(daily)
        db.commit() # 在commit后，模型对象会自动补充递增的ID，故可以直接使用.id获取递增的ID
        return daily.id
    ```

### 运行报错了😱

    ```bash
    sqlalchemy.orm.exc.ObjectDeletedError: Instance '<DailyStatistic at 0x1376a7b20>' has been deleted, or its row is otherwise not present.
    ```
    
    怎么会提示没有这个属性呢？我不李姐😷

### 排查问题

1. 查看数据表是否是自增ID

    ```sql
    CREATE TABLE `daily_statistics` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      ...
    ) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=latin1 COMMENT='日报统计表';
    ```

2. 查看`sqlacodegen`生成的Model文件

    ```python
    class DailyStatistic(ModelDB2):
        __tablename__ = 'daily_statistics'
    
        id = Column(INTEGER(11), primary_key=True, nullable=False)
        ...
    ```

3. 比对不一样的地方

    比对发现，在`sqlacodegen`生成的Model文件中，并没有自动递增的属性配置

### 解决问题

更改自动生成的Model文件ID属性，增加`autoincrement=True`配置

```python
class DailyStatistic(ModelDB2):
    __tablename__ = 'daily_statistics'

    id = Column(INTEGER(11), primary_key=True, nullable=False, autoincrement=True, comment='主键id')
```

更改后，再次进行添加，无报错✌️

## 总结

1. 不能完全相信自动生成的Model文件
2. 再处理获取自增字段时，要优先检查下Model文件是否有`autoincrement=True`配置

