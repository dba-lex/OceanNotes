#### Display BBU Information

```
MegaCli -AdpBbuCmd -aN
```

#### Display BBU Status Information

```
MegaCli -AdpBbuCmd -GetBbuStatus –aN
```

#### 查看充电状态    

```
MegaCli -AdpBbuCmd -GetBbuStatus -aALL |grep "Charger Status"
```

#### 显示BBU状态信息    

```
MegaCli -AdpBbuCmd -GetBbuStatus –aALL
```

#### 显示当前BBU属性    

```
MegaCli -AdpBbuCmd -GetBbuProperties –aALL
MegaCli -AdpBbuCmd -GetBbuCapacityInfo –aALL
MegaCli -AdpBbuCmd -GetBbuDesignInfo –aALL
```

#### 查看充电进度百分比    

```
MegaCli -AdpBbuCmd -GetBbuStatus -aALL |grep "Relative State of Charge"
```
