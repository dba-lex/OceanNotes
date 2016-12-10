


$ dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbName ORCL -sid ORCL -SysPassword oracle -SystemPassword oracle -emConfiguration NONE -storageType ASM -asmSysPassword oracle -diskGroupName DG8



/home/grid/soft/grid/runInstaller -silent -responseFile /home/grid/soft/grid_install.rsp -showProgress -ignorePrereq

/home/oracle/soft/database/runInstaller -silent -responseFile /home/oracle/soft/db_install.rsp -showProgress -ignorePrereq

su - grid -c '/opt/grid/products/11.2.0/cfgtoollogs/configToolAllCommands RESPONSE_FILE=/home/grid/soft/grid_install.rsp'
