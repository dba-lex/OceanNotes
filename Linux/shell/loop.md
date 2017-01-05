
## for循环

```
#!/bin/bash

for i in 1 2 3 4 5
do
echo "Welcome $i times"
done
```

```
for i in `cat /home/oracle/user.txt`
do
sh user_ddl.sh $i
done
```

```
#!/bin/bash
for (( c=1; c<=5; c++ ))
do
echo "Welcome $c times..."
done
```

```
for i in *.txt
do
mv "$i" "$i.bak"       
done
```



## while循环

```
i=1
while[ "$i" -le 5 ]
do
echo $i
i=$((i + 1 ))
done
```

```
#!/bin/sh
while read line;do
echo $line;
done < /etc/hosts;
```
