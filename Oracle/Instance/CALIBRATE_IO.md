```
set serveroutput on;

declare
  v_max_iops BINARY_INTEGER;
  v_max_mbps BINARY_INTEGER;
  v_act_lat  BINARY_INTEGER;
begin
  dbms_resource_manager.CALIBRATE_IO(num_physical_disks => 1,
                                     max_latency        => 10,
                                     max_iops           => v_max_iops,
                                     max_mbps           => v_max_mbps,
                                     actual_latency     => v_act_lat);
  dbms_output.put_line('max iops : ' || v_max_iops);
  dbms_output.put_line('max mbps : ' || v_max_mbps);
  dbms_output.put_line('actual latency : ' || v_act_lat);
end;
/
```
