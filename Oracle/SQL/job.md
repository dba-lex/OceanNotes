

```
exec dbms_job.remove(70);
exec dbms_job.broken(70,true);
exec dbms_job.run(5);
exec dbms_job.interval(5,'sysdate+2');
exec dbms_job.next_date(5,sysdate+1/24);
```



```
exec dbms_ijob.remove(70);
exec dbms_ijob.broken(5,true);
exec dbms_ijob.run(5);
exec dbms_ijob.interval(5,'sysdate+2');
exec dbms_ijob.next_date(5,sysdate+5);
```



```
1.dbms_job只能在当前用户内创建job、修改和删除job，不能对其他用户的job进行操作;sys用户也无法用dbms_job管理其他用户的job。
2.dbms_ijob只能由sys用户去执行，拥有DBA权限的用户都没有权限去执行它。
3.通过dbms_ijob sys用户可以给其他用户创建job，且job在该用户下，在该用户内可以通过user_jobs视图看到。
4.通过dbms_ijob sys用户能够对其他用户中的job进行删除、修改。
5.sys用户通过dbms_ijob给X用户创建job，那么X用户对该job拥有修改和删除的权限。
```
