# Mongo安装&配置

## 系统配置

```shell
# Linux配置（临时）
echo never >>  /sys/kernel/mm/transparent_hugepage/enabled
echo never >>  /sys/kernel/mm/transparent_hugepage/defrag

# Linux配置（永久）
vim /etc/rc.local
# 添加
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
# 设置允许执行
chmod +x /etc/rc.d/rc.local
```

[Operations Checklist](https://docs.mongodb.com/manual/administration/production-checklist-operations/)