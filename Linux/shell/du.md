
- 前15大的文件

```
du -xB M /var | sort -rn | head -n 15
du -xB M --max-depth=2 /var | sort -rn | head -n 15
```
