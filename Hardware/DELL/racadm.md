

racadm lclog export -f Mylog.xml
racadm -r 10.94.161.119 -u root -p calvin lclog export -f Mylog.xml



### racadm获取TSR

```
#racadm techsupreport collect
Job ID = JID_121422476593
RAC1154: The requested operation is initiated.
Run the RACADM jobqueue sub-command, using the job id to check the status of
the requested operation.

[root@qdata-com1 /root]
#racadm jobqueue view -i JID_121422476593
---------------------------- JOB -------------------------
[Job ID=JID_121422476593]
Job Name=TSR_Collect
Status=Completed
Start Time=[Not Applicable]
Expiration Time=[Not Applicable]
Message=[SYS018: Job completed successfully.]
Percent Complete=[100]
----------------------------------------------------------

用以上命令查询collect进度，等待Status变为Completed

[root@qdata-com1 /root]
#racadm techsupreport export -f xxx.zip
Tech Support Report exported successfully.
```
