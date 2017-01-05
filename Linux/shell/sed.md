
```
^ 表示一行的开头。如：/^#/ 以#开头的匹配。
$ 表示一行的结尾。如：/}$/ 以}结尾的匹配。
\< 表示词首。 如 \<abc 表示以 abc 为首的詞。
\> 表示词尾。 如 abc\> 表示以 abc 結尾的詞。
. 表示任何单个字符。
* 表示某个字符出现了0次或多次。
[ ] 字符集合。 如：[abc]表示匹配a或b或c，还有[a-zA-Z]表示匹配所有的26个字符。如果其中有^表示反，如[^a]表示非a的字符
```


- 在每一行最前面加点东西

```
sed 's/^/#/g' pets.txt
```

- 每一行最后面加点东西

```
sed 's/$/ --- /g' pets.txt
```

- 替换

```
my字符串替换成Hao Chen’s
sed "s/my/Hao Chen's/g" pets.txt

只替换第三行中的内容
sed "3s/my/your/g" pets.txt

只替换第3到第6行的文本
sed "3,6s/my/your/g" pets.txt

只替换每一行的第一个s
sed 's/s/S/1' my.txt

只替换每一行的第二个s
sed 's/s/S/2' my.txt

只替换第一行的第3个以后的s
sed 's/s/S/3g' my.txt
```

- 多个匹配

```
第一个模式把第一行到第三行的my替换成your，第二个则把第3行以后的This替换成了That
sed '1,3s/my/your/g; 3,$s/This/That/g' my.txt
sed -e '1,3s/my/your/g' -e '3,$s/This/That/g' my.txt
```

- i命令(insert插入行)

```
在第1行前插入一行
sed "1 i This is my monkey, my monkey's name is wukong" my.txt
```

- a命令(append插入行)

```
在最后一行后追加一行
sed "$ a This is my monkey, my monkey's name is wukong" my.txt

匹配到/fish/后就追加一行
sed "/fish/a This is my monkey, my monkey's name is wukong" my.txt
```

- c命令(替换匹配行)

```
把第二行替换为指定内容
sed "2 c This is my monkey, my monkey's name is wukong" my.txt

把匹配行替换为指定内容
sed "/fish/c This is my monkey, my monkey's name is wukong" my.txt
```

- d命令(删除匹配行)

```
sed '2d' my.txt
sed '2,$d' my.txt
sed '/fish/d' my.txt
```

-
