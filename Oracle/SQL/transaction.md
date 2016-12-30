
### 回滚中的死事务

```
select ADDR,KTUXEUSN,KTUXESLT,KTUXESQN,KTUXESIZ from x$ktuxe where KTUXECFL='DEAD';
```

### 估计死事务回滚时间

```
declare
 l_start number;
 l_end    number;
 begin
   select ktuxesiz into l_start from x$ktuxe where  KTUXEUSN=10 and KTUXESLT=39;
   dbms_lock.sleep(60);
   select ktuxesiz into l_end from x$ktuxe where  KTUXEUSN=10 and KTUXESLT=39;
   dbms_output.put_line('time est Day:'|| round(l_end/(l_start -l_end)/60/24,2));
 end;
  /
```
