# 常见异常

## Spring Data

### Could not instantiate id generator

```bash
ERROR 2193 --- [nio-8090-exec-5] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : 
    Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; 
    nested exception is org.springframework.dao.DataAccessResourceFailureException: Could not create JPA EntityManager; 
    nested exception is javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; 
    nested exception is org.hibernate.MappingException: Could not instantiate id generator [entity-name=zhe.zhuang.qmzsjmfs.model.entity.meta.BaseMeta]] with root cause
```

* 检查id generator的全限定名是不是填对了。







