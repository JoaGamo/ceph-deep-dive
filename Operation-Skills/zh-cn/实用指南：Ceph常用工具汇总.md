

## 📋 常用工具(汇总简略不全版)

| 功能分类 | 主要命令 | 验证状态 | 使用频率 | 适用场景 | 风险级别 |
|---------|---------|---------|---------|---------|---------|
| **集群监控** | `ceph -s`, `ceph health(detail)`, `ceph df`, `ceph -w`| ✅ 已验证 | ⭐⭐⭐⭐⭐ | 日常监控、故障诊断 | 🟢 无风险 |
| **I/O监控** | `ceph iostat`(版本相关,N以上), `ceph -w`, `ceph status` | ✅ 已验证 | ⭐⭐⭐⭐ | 性能监控 | 🟢 无风险 |
| **OSD管理** | `ceph osd tree`, `ceph osd status`, `ceph osd out/in` | ✅ 已验证 | ⭐⭐⭐⭐ | OSD维护、容量管理 | 🟡 中等风险 (查询无风险) |
| **Monitor管理** | `ceph mon stat`, `ceph quorum_status`, `ceph mon add/remove` | ✅ 已验证 | ⭐⭐⭐ | 集群管理、高可用 | 🔴 高风险 (查询无风险)|
| **Manager管理** | `ceph mgr module enable/disable`, `ceph mgr stat` | ✅ 已验证 | ⭐⭐⭐ | 功能管理、仪表板 | 🟡 中等风险 (查询无风险) |
| **存储池管理** | `ceph osd pool create/delete`, `ceph osd pool set` | ✅ 已验证 | ⭐⭐⭐⭐ | 存储规划、配额管理 | 🔴 高风险 (查询无风险) |
| **PG管理** | `ceph pg stat`, `ceph pg repair`, `ceph pg scrub` | ✅ 已验证 | ⭐⭐⭐⭐ | 数据完整性、故障修复 | 🟡 中等风险 (查询无风险) |
| **认证管理** | `ceph auth list/create/del` | ✅ 已验证 | ⭐⭐⭐ | 安全管理、权限控制 | 🔴 高风险 (查询无风险) |
| **CRUSH管理** | `ceph osd crush tree`, `crushtool`, `ceph osd crush rule` | ✅ 已验证 | ⭐⭐ | 数据分布、故障域 | 🔴 高风险 (查询无风险) |
| **RBD管理** | `rbd create/rm`, `rbd snap create`, `rbd map/unmap` | ✅ 已验证 | ⭐⭐⭐⭐ | 块存储、快照管理 | 🟡 中等风险 |
| **CephFS管理** | `ceph fs status`, `ceph mds stat`, `ceph fs dump`, `ceph mds fail` | ✅ 已验证 | ⭐⭐⭐ | 文件系统、元数据 | 🟡 中等风险 (查询无风险) |
| **RGW管理** | `radosgw-admin user create`, `radosgw-admin bucket` | ✅ 已验证 | ⭐⭐⭐ | 对象存储、用户管理 | 🟡 中等风险  (查询无风险)|
| **配置管理** | `ceph config set/get`, `ceph tell` | 未验证 | ⭐⭐⭐⭐ | 参数调优、故障处理 | 🟡 中等风险 (查询无风险) |
| **性能分析** | `ceph osd perf`,`rbd perf image iostat`, cephfs-top | ✅ 已验证 | ⭐⭐⭐ | 性能测试、瓶颈分析 | 🟢 无风险 |
| **专用工具** | `ceph-objectstore-tool`, `ceph-bluestore-tool` | ✅ 已验证 | ⭐⭐ | 数据恢复、深度诊断 | 🔴 高风险 (查询无风险) |
| **故障排查** | `journalctl`, `ceph daemon dump`, 日志分析 | ✅ 已验证 | ⭐⭐⭐⭐ | 问题诊断、根因分析 | 🟢 无风险 |
| **备份恢复** | `ceph mon getmap`, `ceph auth export`, 数据导出 | ✅ 已验证 | ⭐⭐ | 灾难恢复、迁移 | 🟡 中等风险 (查询无风险) |




## 🔧 1. 集群状态监控

### 1.1 集群整体状态
```bash
# 查看集群状态（最常用）
ceph status
ceph -s

# 查看集群健康状态
ceph health
ceph health detail

# 实时监控集群状态
ceph -w

# 查看集群存储使用情况
ceph df
ceph df detail

# 查看集群版本信息
ceph version
ceph versions
```

### 1.2 集群性能监控
```bash
# 查看集群 I/O 统计
ceph iostat
watch "ceph iostat"

# 查看 OSD 性能统计
ceph osd perf

# 查看慢请求
ceph daemon osd.X dump_slow_requests
ceph daemon osd.X dump_historic_slow_ops

# PG 分布分析
ceph pg dump | grep ^pg | awk '{print $1,$15}' | sort
```

---

## 🗄️ 2. OSD 管理

### 2.1 OSD 基本操作
```bash
# 查看 OSD 状态
ceph osd stat
ceph osd dump
ceph osd tree

# 查看 OSD 使用情况
ceph osd df
ceph osd df tree

# 启动/停止 OSD
systemctl start ceph-osd@X
systemctl stop ceph-osd@X
systemctl restart ceph-osd@X

# 将 OSD 标记为 out/in
ceph osd out X
ceph osd in X

# 将 OSD 标记为 down/up
ceph osd down X
ceph osd up X
```

### 2.2 OSD 维护操作
```bash
# 安全移除 OSD（完整流程）
ceph osd out X                              # 标记为out，开始数据迁移
ceph osd safe-to-destroy X                  # 检查是否安全移除
ceph osd destroy X --yes-i-really-mean-it   # 销毁OSD
ceph osd crush remove osd.X                 # 从CRUSH map移除
ceph auth del osd.X                         # 删除认证信息
ceph osd rm X                               # 从集群移除

# 添加新 OSD
ceph-volume lvm create --data /dev/sdX
ceph-volume lvm create --data /dev/sdX --block.wal /dev/nvme0n1p1

# 替换故障 OSD
ceph osd destroy X --yes-i-really-mean-it
ceph osd new X

# 重新权衡 OSD
ceph osd reweight X 0.8                     # 临时权重调整
ceph osd crush reweight osd.X 2.0           # 永久权重调整
ceph osd reweight-by-utilization            # 按使用率自动调整权重
```

### 2.3 OSD 故障排查
```bash
# 查看 OSD 详细信息
ceph osd find X
ceph osd metadata X

# 查看 OSD 日志
journalctl -u ceph-osd@X -f
journalctl -u ceph-osd@X --since "1 hour ago"

# 修复 OSD
ceph osd scrub X
ceph osd deep-scrub X

# 查看 OSD 的 PG
ceph pg ls-by-osd X

# OSD 性能诊断
ceph daemon osd.X perf dump
ceph daemon osd.X dump_ops_in_flight
ceph daemon osd.X dump_blocked_ops
```

---

## 🏛️ 3. Monitor 管理

### 3.1 Monitor 基本操作
```bash
# 查看 Monitor 状态
ceph mon stat
ceph mon dump

# 查看 Monitor 仲裁状态
ceph quorum_status

# 添加 Monitor
ceph mon add <mon-id> <mon-addr>

# 移除 Monitor
ceph mon remove <mon-id>

# 查看 Monitor 映射
ceph mon getmap -o /tmp/monmap
monmaptool --print /tmp/monmap
```

### 3.2 Monitor 维护
```bash
# 启动/停止 Monitor
systemctl start ceph-mon@<hostname>
systemctl stop ceph-mon@<hostname>

# 压缩 Monitor 数据库
ceph tell mon.<mon-id> compact

# 同步 Monitor 时间
ceph time-sync-status

# Monitor 故障恢复
ceph-mon --extract-monmap /tmp/monmap --mon-data /var/lib/ceph/mon/ceph-<id>
```

---

## 👨‍💼 4. Manager 管理

### 4.1 Manager 基本操作
```bash
# 查看 Manager 状态
ceph mgr stat
ceph mgr dump

# 启用/禁用 Manager 模块
ceph mgr module enable <module>
ceph mgr module disable <module>

# 查看可用模块
ceph mgr module ls

# 故障转移 Manager
ceph mgr fail <mgr-id>
```

### 4.2 常用 Manager 模块
```bash
# Dashboard 模块
ceph mgr module enable dashboard
ceph dashboard create-self-signed-cert
ceph dashboard ac-user-create <username> -i <password-file>
ceph dashboard ac-role-add-scope-perms admin read-write pool
ceph config set mgr mgr/dashboard/server_addr 0.0.0.0
ceph config set mgr mgr/dashboard/server_port 8443

# Prometheus 模块
ceph mgr module enable prometheus
ceph config set mgr mgr/prometheus/server_addr 0.0.0.0
ceph config set mgr mgr/prometheus/server_port 9283

# Balancer 模块
ceph mgr module enable balancer
ceph balancer mode upmap
ceph balancer on
ceph balancer status

# Alerts 模块
ceph mgr module enable alerts
```

---

## 🗂️ 5. 存储池管理

### 5.1 Pool 基本操作
```bash
# 创建存储池
ceph osd pool create <poolname> <pg_num> [pgp_num]
ceph osd pool create rbd 128 128 replicated
ceph osd pool create mypool 64 64 erasure

# 删除存储池
ceph config set mon mon_allow_pool_delete true
ceph osd pool delete <poolname> <poolname> --yes-i-really-really-mean-it

# 查看存储池
ceph osd lspools
ceph osd pool ls detail

# 设置存储池配额
ceph osd pool set-quota <poolname> max_objects 1000
ceph osd pool set-quota <poolname> max_bytes 1TB

# 查看存储池统计
ceph osd pool stats
ceph osd pool stats <poolname>
```

### 5.2 Pool 参数配置
```bash
# 设置存储池副本数
ceph osd pool set <poolname> size 3
ceph osd pool set <poolname> min_size 2

# 设置 PG 数量
ceph osd pool set <poolname> pg_num 128
ceph osd pool set <poolname> pgp_num 128

# 设置存储池类型
ceph osd pool set <poolname> crush_rule <rule-name>

# 启用/禁用功能
ceph osd pool set <poolname> noscrub true
ceph osd pool set <poolname> nodeep-scrub true

# Pool 应用标签
ceph osd pool application enable <poolname> rbd
ceph osd pool application enable <poolname> cephfs
ceph osd pool application enable <poolname> rgw
```

---

## 📄 6. 放置组 (PG) 管理

### 6.1 PG 状态查看
```bash
# 查看 PG 状态
ceph pg stat
ceph pg dump
ceph pg dump --format json-pretty

# 查看异常 PG
ceph pg dump_stuck
ceph pg dump_stuck inactive
ceph pg dump_stuck unclean

# 查看特定 PG 信息
ceph pg <pgid> query
ceph pg map <pgid>
```

### 6.2 PG 修复操作
```bash
# 修复 PG
ceph pg repair <pgid>
ceph pg scrub <pgid>
ceph pg deep-scrub <pgid>

# 强制修复不一致的对象
ceph pg force-recovery <pgid>
ceph pg force-backfill <pgid>

# 查看 PG 的 OSD
ceph pg ls-by-pool <poolname>
ceph pg ls-by-primary <osd-id>

# PG 自动修复
ceph config set osd osd_scrub_auto_repair true
```

---

## 🔐 7. 认证管理

### 7.1 用户管理
```bash
# 查看认证信息
ceph auth list
ceph auth get client.admin

# 创建用户
ceph auth get-or-create client.<name> mon 'allow r' osd 'allow rw pool=<poolname>'
ceph auth get-or-create client.rbd mon 'profile rbd' osd 'profile rbd pool=rbd'

# 删除用户
ceph auth del client.<name>

# 导出/导入密钥
ceph auth export client.<name> > client.<name>.keyring
ceph auth import -i client.<name>.keyring
```

### 7.2 权限管理
```bash
# 修改用户权限
ceph auth caps client.<name> mon 'allow r' osd 'allow rw pool=rbd'

# 查看权限
ceph auth get client.<name>

# 生成客户端配置
ceph config generate-minimal-conf

# 常用权限模板
# 只读用户：mon 'allow r' osd 'allow r'
# RBD用户：mon 'profile rbd' osd 'profile rbd pool=rbd'
# CephFS用户：mon 'allow r' mds 'allow rw' osd 'allow rw pool=cephfs_data'
```

---

## 🏗️ 8. CRUSH Map 管理

### 8.1 CRUSH 基本操作
```bash
# 查看 CRUSH map
ceph osd tree
ceph osd crush tree

# 导出/导入 CRUSH map
ceph osd getcrushmap -o crushmap.bin
crushtool -d crushmap.bin -o crushmap.txt
crushtool -c crushmap.txt -o crushmap-new.bin
ceph osd setcrushmap -i crushmap-new.bin

# 创建/删除 bucket
ceph osd crush add-bucket <bucket-name> <bucket-type>
ceph osd crush remove <bucket-name>

# 移动 OSD 到不同位置
ceph osd crush move osd.X host=<hostname>
ceph osd crush move osd.X rack=<rackname>
```

### 8.2 CRUSH 规则管理
```bash
# 查看 CRUSH 规则
ceph osd crush rule ls
ceph osd crush rule dump

# 创建 CRUSH 规则
ceph osd crush rule create-simple <rule-name> <root> <type>
ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <device-class>

# 删除 CRUSH 规则
ceph osd crush rule rm <rule-name>

# 测试 CRUSH 映射
crushtool -i crushmap.bin --test --show-mappings --ruleset 0 --num-rep 3 --min-x 0 --max-x 100
```

---

## 🎯 9. RBD 管理

### 9.1 RBD 基本操作
```bash
# 创建 RBD 镜像
rbd create --size 1024 <poolname>/<image-name>
rbd create --size 10G --image-feature layering,exclusive-lock mypool/myimage

# 查看 RBD 镜像
rbd ls <poolname>
rbd info <poolname>/<image-name>
rbd du <poolname>/<image-name>

# 删除 RBD 镜像
rbd rm <poolname>/<image-name>
rbd trash mv <poolname>/<image-name>    # 移动到回收站
rbd trash restore <poolname>/<image-id>  # 从回收站恢复

# 调整镜像大小
rbd resize --size 2048 <poolname>/<image-name>
rbd resize --size 2G <poolname>/<image-name>

# 创建快照
rbd snap create <poolname>/<image-name>@<snap-name>
rbd snap ls <poolname>/<image-name>
rbd snap rm <poolname>/<image-name>@<snap-name>
```

### 9.2 RBD 高级操作
```bash
# 克隆镜像
rbd snap protect <poolname>/<parent-image>@<snap-name>
rbd clone <poolname>/<parent-image>@<snap-name> <poolname>/<child-image>

# 展平克隆镜像
rbd flatten <poolname>/<image-name>

# 导出/导入镜像
rbd export <poolname>/<image-name> image.raw
rbd export --export-format 2 <poolname>/<image-name> image.raw
rbd import image.raw <poolname>/<image-name>

# 镜像映射
rbd map <poolname>/<image-name>
rbd unmap /dev/rbdX
rbd showmapped

# 镜像特性管理
rbd feature enable <poolname>/<image-name> exclusive-lock
rbd feature disable <poolname>/<image-name> deep-flatten
```

---

## 🌐 10. CephFS 管理

### 10.1 CephFS 基本操作
```bash
# 创建文件系统
ceph fs new <fs-name> <metadata-pool> <data-pool>

# 查看文件系统
ceph fs ls
ceph fs status <fs-name>
ceph fs get <fs-name>

# 删除文件系统
ceph fs fail <fs-name>
ceph fs rm <fs-name> --yes-i-really-mean-it

# MDS 管理
ceph mds stat
ceph mds dump
ceph mds fail <mds-id>
ceph mds repaired <mds-id>
```

### 10.2 CephFS 客户端
```bash
# 挂载 CephFS
mount -t ceph <mon-addr>:/ /mnt/cephfs -o name=admin,secret=<key>
mount -t ceph <mon-addr>:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.secret

# 使用 ceph-fuse
ceph-fuse /mnt/cephfs
ceph-fuse /mnt/cephfs -o allow_other

# 查看客户端连接
ceph daemon mds.<id> client ls
ceph daemon mds.<id> session ls

# 踢除客户端
ceph daemon mds.<id> client evict id=<client-id>
```

### 10.3 CephFS 高级管理
```bash
# 子目录挂载
mount -t ceph <mon-addr>:/subdir /mnt/subdir -o name=admin

# 配置多个文件系统
ceph fs flag set enable_multiple true

# 目录配额
setfattr -n ceph.quota.max_files -v 10000 /mnt/cephfs/dir
setfattr -n ceph.quota.max_bytes -v 1000000000 /mnt/cephfs/dir
getfattr -n ceph.quota.max_files /mnt/cephfs/dir
```

---

## ☁️ 11. RGW 管理

### 11.1 RGW 基本操作
```bash
# 创建用户
radosgw-admin user create --uid=<user-id> --display-name="<display-name>"
radosgw-admin user create --uid=testuser --display-name="Test User" --email=test@example.com

# 查看用户
radosgw-admin user list
radosgw-admin user info --uid=<user-id>

# 删除用户
radosgw-admin user rm --uid=<user-id>

# 修改用户
radosgw-admin user modify --uid=<user-id> --display-name="New Name"

# 创建子用户
radosgw-admin subuser create --uid=<user-id> --subuser=<subuser-id> --access=full
```

### 11.2 RGW 密钥管理
```bash
# 创建访问密钥
radosgw-admin key create --uid=<user-id> --key-type=s3

# 删除访问密钥
radosgw-admin key rm --uid=<user-id> --key-type=s3 --access-key=<access-key>

# 生成Swift密钥
radosgw-admin key create --uid=<user-id> --key-type=swift --gen-secret
```

### 11.3 RGW 维护
```bash
# 检查 bucket 索引
radosgw-admin bucket check --bucket=<bucket-name>

# 重建 bucket 索引
radosgw-admin bucket reshard --bucket=<bucket-name> --num-shards=<num>

# 垃圾回收
radosgw-admin gc list
radosgw-admin gc process

# 查看使用统计
radosgw-admin usage show --uid=<user-id>
radosgw-admin usage show --show-log-entries=false

# Bucket 管理
radosgw-admin bucket list
radosgw-admin bucket stats --bucket=<bucket-name>
```

---

## 🔧 12. 配置管理

### 12.1 运行时配置
```bash
# 查看配置
ceph config dump
ceph config get <daemon-type>.<daemon-id> <option>
ceph config show <daemon-type>.<daemon-id>

# 设置配置
ceph config set <daemon-type> <option> <value>
ceph config set global osd_pool_default_size 3
ceph config set osd osd_max_backfills 1

# 重置配置
ceph config rm <daemon-type> <option>

# 查看配置历史
ceph config log
```

### 12.2 临时配置调整
```bash
# 临时设置配置（重启后失效）
ceph tell <daemon-type>.<id> config set <option> <value>
ceph tell osd.* config set debug_osd 10

# 查看配置
ceph tell <daemon-type>.<id> config show
ceph tell <daemon-type>.<id> config get <option>

# 批量配置
ceph tell osd.* config set osd_max_backfills 1
ceph tell mon.* config set debug_mon 5
```

---

## 📈 13. 性能分析专项

### 13.1 RBD 性能分析

#### RBD 性能测试工具
```bash
# 使用 rbd bench 进行性能测试
rbd bench --io-type write --io-size 4K --io-threads 16 --io-total 1G <pool>/<image>
rbd bench --io-type read --io-size 4K --io-threads 16 --io-total 1G <pool>/<image>
rbd bench --io-type readwrite --io-size 4K --io-threads 16 --io-total 1G <pool>/<image>

# 使用 fio 进行详细性能测试
fio --name=rbd-test --ioengine=rbd --pool=<pool> --rbdname=<image> \
    --rw=randwrite --bs=4k --iodepth=32 --numjobs=4 --runtime=60 --group_reporting

# 测试不同块大小的性能
for bs in 4k 8k 16k 32k 64k 128k; do
    echo "Testing block size: $bs"
    rbd bench --io-type write --io-size $bs --io-threads 16 --io-total 512M <pool>/<image>
done

# 顺序读写测试
fio --name=seq-write --ioengine=rbd --pool=test --rbdname=testimg \
    --rw=write --bs=1M --iodepth=1 --numjobs=1 --runtime=60
```

#### RBD 性能监控
```bash
# 实时监控 RBD I/O
rbd perf image iostat

# 查看 RBD 镜像统计
rbd du <pool>/<image>
rbd info <pool>/<image>

# 监控客户端 I/O 模式
ceph daemon client.<id> perf dump
ceph daemon client.<id> perf histogram dump

# 查看 RBD 缓存统计
ceph daemon client.<id> config show | grep rbd_cache
ceph daemon client.<id> perf dump | grep rbd_cache
```

#### RBD 性能调优参数
```bash
# 客户端缓存优化
ceph config set client rbd_cache true
ceph config set client rbd_cache_size 134217728           # 128MB
ceph config set client rbd_cache_max_dirty 100663296      # 96MB
ceph config set client rbd_cache_target_dirty 67108864    # 64MB
ceph config set client rbd_cache_max_dirty_age 10.0

# 预读优化
ceph config set client rbd_readahead_trigger_requests 10
ceph config set client rbd_readahead_max_bytes 524288     # 512KB
ceph config set client rbd_readahead_disable_after_bytes 52428800  # 50MB

# 并发优化
ceph config set client rbd_concurrent_management_ops 20
ceph config set client rbd_op_threads 1
```

### 13.2 CephFS 性能分析

#### CephFS 性能测试
```bash
# 使用 dd 进行基本性能测试
dd if=/dev/zero of=/mnt/cephfs/testfile bs=1M count=1000 oflag=direct
dd if=/mnt/cephfs/testfile of=/dev/null bs=1M iflag=direct

# 使用 iozone 进行全面性能测试
iozone -a -s 1G -r 4k -r 64k -r 1M -i 0 -i 1 -i 2 -f /mnt/cephfs/testfile

# 小文件性能测试
mkdir /mnt/cephfs/small_files_test
time for i in {1..10000}; do touch /mnt/cephfs/small_files_test/file_$i; done

# 元数据性能测试
time find /mnt/cephfs -name "*.txt" | wc -l
time ls -la /mnt/cephfs/large_directory/ | wc -l

# 并发文件操作测试
for i in {1..10}; do
    (for j in {1..1000}; do
        touch /mnt/cephfs/test_${i}_${j}
    done) &
done
wait
```

#### CephFS 性能监控
```bash
# MDS 性能统计
ceph daemon mds.<id> perf dump
ceph daemon mds.<id> session ls
ceph daemon mds.<id> status
ceph daemon mds.<id> ops

# 客户端性能统计
ceph daemon client.<id> perf dump
ceph daemon client.<id> client_metadata

# 查看 MDS 负载
ceph fs status <fsname>
ceph mds metadata <mds-id>

# 监控目录碎片化
ceph daemon mds.<id> dirfrag ls <path>
ceph daemon mds.<id> dirfrag split_info <path>
```

#### CephFS 性能调优
```bash
# MDS 缓存调优
ceph config set mds mds_cache_memory_limit 4294967296     # 4GB
ceph config set mds mds_cache_reservation 0.05
ceph config set mds mds_health_cache_threshold 1.5

# 客户端缓存调优
ceph config set client client_cache_size 134217728        # 128MB
ceph config set client client_cache_mid 0.75
ceph config set client fuse_default_permissions 0

# 并发调优
ceph config set mds mds_max_caps_per_client 1048576
ceph config set mds mds_session_timeout 60
ceph config set mds mds_session_autoclose 300

# 目录分片调优
ceph config set mds mds_bal_split_size 10000
ceph config set mds mds_bal_merge_size 50
```

### 13.3 OSD 性能分析

#### OSD 性能基准测试
```bash
# OSD 基准测试
ceph tell osd.<id> bench 1073741824 4194304              # 1GB测试，4MB块大小

# 使用 ceph-osd 直接测试
ceph-osd -i <id> --test-objectstore --test-objectstore-workload uniform_random

# 磁盘性能测试
dd if=/dev/zero of=/var/lib/ceph/osd/ceph-<id>/test bs=1M count=1000 oflag=direct
dd if=/var/lib/ceph/osd/ceph-<id>/test of=/dev/null bs=1M iflag=direct

# 网络性能测试
iperf3 -s    # 在目标OSD节点运行
iperf3 -c <target-osd-ip> -t 30 -P 4    # 在源节点运行

# 使用 rados bench 测试
rados bench -p <pool> 60 write --no-cleanup
rados bench -p <pool> 60 seq
rados bench -p <pool> 60 rand
rados -p <pool> cleanup
```

#### OSD 详细性能监控
```bash
# OSD 性能计数器
ceph daemon osd.<id> perf dump
ceph daemon osd.<id> perf histogram dump

# OSD 操作统计
ceph daemon osd.<id> dump_ops_in_flight
ceph daemon osd.<id> dump_blocked_ops
ceph daemon osd.<id> dump_op_pq_state

# OSD 存储统计
ceph daemon osd.<id> status
ceph daemon osd.<id> df

# 慢操作分析
ceph daemon osd.<id> dump_slow_requests
ceph daemon osd.<id> dump_historic_slow_ops

# OSD 恢复统计
ceph daemon osd.<id> dump_recovery_reservations
ceph daemon osd.<id> dump_backfill_reservations
```

#### OSD 性能调优参数
```bash
# 队列深度调优
ceph config set osd osd_op_queue mclock_scheduler
ceph config set osd osd_op_queue_mclock_anticipation_timeout 0.0

# 线程池调优
ceph config set osd osd_op_threads 8
ceph config set osd osd_disk_threads 1
ceph config set osd osd_recovery_threads 1

# 刷写调优
ceph config set osd osd_max_write_size 90
ceph config set osd osd_max_object_size 134217728
ceph config set osd osd_client_message_size_cap 524288000

# BlueStore 调优
ceph config set osd bluestore_cache_size 3221225472       # 3GB
ceph config set osd bluestore_cache_meta_ratio 0.4
ceph config set osd bluestore_cache_kv_ratio 0.4
ceph config set osd bluestore_min_alloc_size 4096

# 恢复调优
ceph config set osd osd_recovery_max_active 3
ceph config set osd osd_max_backfills 1
ceph config set osd osd_recovery_sleep 0.1
```

### 13.4 整体集群性能分析

#### 集群级性能监控
```bash
# 集群 I/O 统计
ceph iostat
watch "ceph iostat"

# PG 分布分析
ceph pg dump | grep ^pg | awk '{print $1,$15}' | sort

# 实时性能监控脚本
#!/bin/bash
while true; do
    echo "$(date) - IOPS: $(ceph iostat | grep -E 'read|write')"
    echo "$(date) - Health: $(ceph health)"
    sleep 5
done

# 集群延迟分析
ceph osd perf | grep -E "apply_latency|commit_latency"

# 网络延迟检查
for osd in $(ceph osd ls); do
    echo "OSD $osd latency:"
    ceph daemon osd.$osd dump_slow_requests | head -5
done
```

---

## 🛠️ 14. Ceph 专用工具集

### 14.1 数据恢复和诊断工具

#### ceph-objectstore-tool (对象存储工具)
```bash
# 列出对象存储内容
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> --op list

# 导出 PG 数据
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> \
    --op export --pgid <pg-id> --file /tmp/pg_export.dat

# 导入 PG 数据
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> \
    --op import --file /tmp/pg_export.dat

# 删除损坏的对象
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> \
    --op remove --pgid <pg-id> --oid <object-id>

# 修复对象存储
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> \
    --op fsck --debug

# 查看对象信息
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-<id> \
    --op info --pgid <pg-id> --oid <object-id>
```

#### ceph-kvstore-tool (键值存储工具)
```bash
# 列出键值对
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> list

# 获取特定键值
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> get <prefix> <key>

# 删除键值对
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> rm <prefix> <key>

# 压缩数据库
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> compact

# 备份/恢复键值存储
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> backup /tmp/kv_backup
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> restore /tmp/kv_backup

# 统计信息
ceph-kvstore-tool bluestore-kv /var/lib/ceph/osd/ceph-<id> stats
```

#### ceph-bluestore-tool (BlueStore 专用工具)
```bash
# 查看 BlueStore 信息
ceph-bluestore-tool show-label --dev /dev/sdX
ceph-bluestore-tool prime-osd-dir --dev /dev/sdX --path /var/lib/ceph/osd/ceph-X

# 修复 BlueStore
ceph-bluestore-tool fsck --path /var/lib/ceph/osd/ceph-<id>
ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-<id>

# 展示 BlueStore 元数据
ceph-bluestore-tool show-label --dev /dev/sdX
ceph-bluestore-tool show-label --path /var/lib/ceph/osd/ceph-<id>

# 蓝存储空间分析
ceph-bluestore-tool bluefs-stats --path /var/lib/ceph/osd/ceph-<id>
ceph-bluestore-tool bluefs-bdev-sizes --path /var/lib/ceph/osd/ceph-<id>

# 分配器工具
ceph-bluestore-tool free-dump --path /var/lib/ceph/osd/ceph-<id>
ceph-bluestore-tool free-score --path /var/lib/ceph/osd/ceph-<id>
```

### 14.2 监控和管理工具

#### crushtool (CRUSH 映射工具)
```bash
# 编译/反编译 CRUSH map
ceph osd getcrushmap -o crushmap.bin
crushtool -d crushmap.bin -o crushmap.txt
crushtool -c crushmap.txt -o crushmap_new.bin
ceph osd setcrushmap -i crushmap_new.bin

# 测试 CRUSH 规则
crushtool -i crushmap.bin --test --show-mappings --ruleset 0 --num-rep 3 --min-x 0 --max-x 100

# 分析 CRUSH 权重
crushtool -i crushmap.bin --show-choose-tries

# 验证 CRUSH map
crushtool -i crushmap.bin --check

# 显示 CRUSH 树结构
crushtool -i crushmap.bin --tree
```

#### monmaptool (Monitor 映射工具)
```bash
# 创建 Monitor map
monmaptool --create --add <mon-id> <mon-addr> --fsid <uuid> monmap.bin

# 查看 Monitor map
monmaptool --print monmap.bin

# 添加/删除 Monitor
monmaptool --add <mon-id> <mon-addr> monmap.bin
monmaptool --rm <mon-id> monmap.bin

# 生成新的 fsid
uuidgen
monmaptool --create --fsid $(uuidgen) monmap.bin

# 从现有集群提取
ceph mon getmap -o monmap.bin
monmaptool --print monmap.bin
```

#### osdmaptool (OSD 映射工具)
```bash
# 导出 OSD map
ceph osd getmap -o osdmap.bin
osdmaptool osdmap.bin --print

# 测试 PG 映射
osdmaptool osdmap.bin --test-map-pgs --pool <pool-id>

# 创建增量 map
osdmaptool --createsimple 3 osdmap.bin --with-default-pool

# 导出特定 epoch 的 map
ceph osd getmap <epoch> -o osdmap_<epoch>.bin

# 显示 OSD 树
osdmaptool osdmap.bin --tree
```

### 14.3 数据迁移和同步工具

#### rados (对象存储客户端)
```bash
# 基本对象操作
rados -p <pool> put <object> <file>
rados -p <pool> get <object> <file>
rados -p <pool> ls
rados -p <pool> rm <object>

# 批量操作
rados -p <pool> ls | xargs -I {} rados -p <pool> rm {}

# 基准测试
rados bench -p <pool> 60 write --no-cleanup
rados bench -p <pool> 60 seq
rados bench -p <pool> 60 rand

# 清理基准测试数据
rados -p <pool> cleanup

# 对象属性操作
rados -p <pool> setxattr <object> <attr> <value>
rados -p <pool> getxattr <object> <attr>
rados -p <pool> listxattr <object>

# 监视对象变化
rados -p <pool> watch <object>
```

#### rbd-mirror (RBD 镜像工具)
```bash
# 启用镜像
rbd mirror pool enable <pool> pool
rbd mirror pool enable <pool> image

# 配置镜像
rbd mirror image enable <pool>/<image> snapshot
rbd mirror image enable <pool>/<image> journal

# 查看镜像状态
rbd mirror pool status <pool>
rbd mirror image status <pool>/<image>

# 强制重新同步
rbd mirror image resync <pool>/<image>

# 故障转移
rbd mirror image promote <pool>/<image>
rbd mirror image demote <pool>/<image>
```

### 14.4 网络和通信工具

#### ceph-syn (同步测试工具)
```bash
# 测试文件系统同步
ceph-syn -c /etc/ceph/ceph.conf --syn workload1

# 自定义工作负载测试
ceph-syn -c /etc/ceph/ceph.conf --makedir /test/dir
ceph-syn -c /etc/ceph/ceph.conf --writefile /test/file 1048576

# 元数据操作测试
ceph-syn -c /etc/ceph/ceph.conf --mkdir /test
ceph-syn -c /etc/ceph/ceph.conf --rmdir /test
```

#### ceph-detect-init (初始化系统检测)
```bash
# 检测初始化系统
ceph-detect-init

# 检测系统信息
ceph-detect-init --verbose
```

### 14.5 日志和调试工具

#### ceph-post-file (日志上传工具)
```bash
# 上传日志文件进行分析
ceph-post-file /var/log/ceph/ceph-osd.0.log

# 上传带描述的日志
ceph-post-file -d "OSD crash investigation" /var/log/ceph/ceph-osd.0.log
```

#### ceph-crash (崩溃报告工具)
```bash
# 查看崩溃报告
ceph crash ls
ceph crash info <crash-id>

# 归档崩溃报告
ceph crash archive <crash-id>
ceph crash archive-all

# 生成崩溃报告
ceph crash post -i <crash-dump-file>

# 统计崩溃报告
ceph crash stat
```

#### ceph-volume (卷管理工具)
```bash
# LVM 方式创建 OSD
ceph-volume lvm create --data /dev/sdb
ceph-volume lvm create --data /dev/sdb --block.wal /dev/nvme0n1p1

# 准备和激活分离
ceph-volume lvm prepare --data /dev/sdb --osd-id 0
ceph-volume lvm activate 0 <osd-uuid>

# 查看 LVM 信息
ceph-volume lvm list
ceph-volume lvm list /dev/sdb

# 简单方式创建 OSD
ceph-volume simple scan /dev/sdb
ceph-volume simple activate <osd-id> <osd-uuid>
```

---

## 🚨 15. 故障排查

### 15.1 日志查看
```bash
# 查看各服务日志
journalctl -u ceph-mon@<hostname> -f
journalctl -u ceph-osd@<id> -f
journalctl -u ceph-mds@<hostname> -f
journalctl -u ceph-mgr@<hostname> -f

# 按时间范围查看日志
journalctl -u ceph-osd@<id> --since "2023-01-01 00:00:00" --until "2023-01-01 23:59:59"

# 调整日志级别
ceph tell osd.* config set debug_osd 10
ceph tell mon.* config set debug_mon 10
ceph tell mds.* config set debug_mds 10

# 查看实时错误
tail -f /var/log/ceph/ceph.log | grep -i error
tail -f /var/log/ceph/ceph-osd.*.log | grep -i slow
```

### 15.2 网络诊断
```bash
# 测试 OSD 之间连通性
ceph osd bench <osd-id> <bytes> <write|seq|rand>

# 查看网络连接
ceph daemon osd.<id> dump_ops_in_flight
ceph daemon osd.<id> dump_blocked_ops

# 检查集群网络
ceph features
ceph daemon osd.<id> dump_mempools

# 网络延迟测试
ping -c 10 <osd-node-ip>
iperf3 -c <osd-node-ip> -t 10
```

### 15.3 常见问题排查
```bash
# PG 不一致问题
ceph health detail | grep inconsistent
ceph pg <pgid> query
ceph pg repair <pgid>

# OSD 满载问题
ceph osd df | grep -E "(95|96|97|98|99)%"
ceph osd reweight-by-utilization

# 慢请求问题
ceph daemon osd.<id> dump_slow_requests
ceph daemon osd.<id> dump_historic_slow_ops
ceph config set osd debug_osd 10

# 时钟同步问题
ceph time-sync-status
ntpq -p

# 磁盘空间问题
ceph df
ceph osd df
df -h /var/lib/ceph/osd/
```

---

## 🔄 16. 备份恢复

### 16.1 数据备份
```bash
# 导出 Monitor map
ceph mon getmap -o monmap.bin

# 导出 OSD map
ceph osd getmap -o osdmap.bin

# 导出 CRUSH map
ceph osd getcrushmap -o crushmap.bin

# 备份 ceph.conf 和密钥
cp /etc/ceph/ceph.conf /backup/
cp /etc/ceph/ceph.client.admin.keyring /backup/

# 备份认证信息
ceph auth export > /backup/ceph_auth.txt

# 备份存储池信息
ceph osd pool ls detail > /backup/pools.txt
ceph pg dump > /backup/pg_dump.txt
```

### 16.2 灾难恢复
```bash
# 从单个 OSD 恢复数据
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-X \
    --export-remove --pgid <pgid> --file /tmp/pg_export

# 重建 Monitor
ceph-mon --mkfs -i <mon-id> --monmap monmap.bin --keyring mon.keyring

# 恢复认证信息
ceph auth import -i /backup/ceph_auth.txt

# 恢复 CRUSH map
ceph osd setcrushmap -i /backup/crushmap.bin

# OSD 数据恢复
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-X \
    --op import --file /tmp/pg_export
```

### 16.3 RBD 备份恢复
```bash
# RBD 快照备份
rbd snap create <pool>/<image>@backup-$(date +%Y%m%d)
rbd export <pool>/<image>@<snapshot> /backup/image_backup.raw

# RBD 增量备份
rbd export-diff <pool>/<image>@<snap1> <pool>/<image>@<snap2> /backup/diff.raw

# RBD 恢复
rbd import /backup/image_backup.raw <pool>/<new-image>
rbd import-diff /backup/diff.raw <pool>/<image>
```

---

## 📊 17. 运维脚本集合

### 17.1 集群健康检查脚本
```bash
#!/bin/bash
# cluster_health_check.sh
# Ceph 集群健康检查脚本

LOG_FILE="/var/log/ceph_health_check.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "=== Ceph Cluster Health Report ===" | tee -a $LOG_FILE
echo "Date: $DATE" | tee -a $LOG_FILE
echo

echo "1. Cluster Status:" | tee -a $LOG_FILE
ceph -s | tee -a $LOG_FILE
echo

echo "2. Health Details:" | tee -a $LOG_FILE
ceph health detail | tee -a $LOG_FILE
echo

echo "3. OSD Status:" | tee -a $LOG_FILE
ceph osd tree | grep -E "(osd\.|down|out)" | tee -a $LOG_FILE
echo

echo "4. PG Status:" | tee -a $LOG_FILE
ceph pg stat | tee -a $LOG_FILE
ceph health detail | grep -i pg | tee -a $LOG_FILE
echo

echo "5. Performance:" | tee -a $LOG_FILE
ceph iostat | head -5 | tee -a $LOG_FILE
echo

echo "6. Disk Usage:" | tee -a $LOG_FILE
ceph df | head -10 | tee -a $LOG_FILE
echo

echo "7. Slow Operations:" | tee -a $LOG_FILE
SLOW_OPS=$(ceph health detail | grep -c "slow ops")
echo "Slow operations count: $SLOW_OPS" | tee -a $LOG_FILE

if [ $SLOW_OPS -gt 0 ]; then
    echo "OSDs with slow operations:" | tee -a $LOG_FILE
    ceph health detail | grep "slow ops" | tee -a $LOG_FILE
fi
echo

echo "Health check completed at: $(date)" | tee -a $LOG_FILE
echo "========================================" | tee -a $LOG_FILE
```

### 17.2 性能监控脚本
```bash
#!/bin/bash
# performance_monitor.sh
# Ceph 性能监控脚本

LOG_FILE="/var/log/ceph_performance.log"
INTERVAL=30

while true; do
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    # 获取集群 I/O 统计
    iostat=$(ceph iostat | grep -E "client|recovery" | tr '\n' ' ')
    
    # 获取慢操作统计
    slow_ops=$(ceph health detail | grep -c "slow ops")
    
    # 获取 PG 状态
    pg_stats=$(ceph pg stat | grep "pgs:" | awk '{print $2 " " $3}')
    
    # 获取集群利用率
    cluster_usage=$(ceph df | grep "TOTAL" | awk '{print $6}')
    
    # 记录到日志
    echo "$timestamp - I/O: $iostat - Slow Ops: $slow_ops - PGs: $pg_stats - Usage: $cluster_usage" >> $LOG_FILE
    
    # 检查是否需要报警
    if [ $slow_ops -gt 10 ]; then
        echo "$timestamp - ALERT: High slow operations count: $slow_ops" >> $LOG_FILE
    fi
    
    sleep $INTERVAL
done
```

### 17.3 自动化维护脚本
```bash
#!/bin/bash
# auto_maintenance.sh
# Ceph 自动化维护脚本

LOG_FILE="/var/log/ceph_maintenance.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "$DATE - Starting maintenance tasks" >> $LOG_FILE

# 1. 定期清理陈旧的 PG 日志
echo "$DATE - Compacting OSD logs" >> $LOG_FILE
ceph tell osd.* compact 2>&1 >> $LOG_FILE

# 2. 检查并修复不一致的 PG
echo "$DATE - Checking for inconsistent PGs" >> $LOG_FILE
INCONSISTENT_PGS=$(ceph pg dump | grep inconsistent | awk '{print $1}')

if [ ! -z "$INCONSISTENT_PGS" ]; then
    echo "$DATE - Found inconsistent PGs: $INCONSISTENT_PGS" >> $LOG_FILE
    for pg in $INCONSISTENT_PGS; do
        echo "$DATE - Repairing PG: $pg" >> $LOG_FILE
        ceph pg repair $pg 2>&1 >> $LOG_FILE
        sleep 60
    done
else
    echo "$DATE - No inconsistent PGs found" >> $LOG_FILE
fi

# 3. 清理崩溃报告（保留最近30天）
echo "$DATE - Cleaning old crash reports" >> $LOG_FILE
CRASH_COUNT=$(ceph crash ls | wc -l)
if [ $CRASH_COUNT -gt 100 ]; then
    ceph crash archive-all 2>&1 >> $LOG_FILE
    echo "$DATE - Archived $CRASH_COUNT crash reports" >> $LOG_FILE
fi

# 4. 检查磁盘使用率
echo "$DATE - Checking disk usage" >> $LOG_FILE
HIGH_USAGE_OSDS=$(ceph osd df | awk '$6 > 85 {print $1 " " $6}')
if [ ! -z "$HIGH_USAGE_OSDS" ]; then
    echo "$DATE - WARNING: High disk usage OSDs:" >> $LOG_FILE
    echo "$HIGH_USAGE_OSDS" >> $LOG_FILE
fi

echo "$DATE - Maintenance tasks completed" >> $LOG_FILE
```

### 17.4 备份脚本
```bash
#!/bin/bash
# ceph_backup.sh
# Ceph 配置和元数据备份脚本

BACKUP_DIR="/backup/ceph/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

echo "Starting Ceph backup to $BACKUP_DIR"

# 备份集群配置
echo "Backing up cluster configuration..."
cp /etc/ceph/ceph.conf $BACKUP_DIR/
cp /etc/ceph/*.keyring $BACKUP_DIR/

# 导出各种映射
echo "Exporting cluster maps..."
ceph mon getmap -o $BACKUP_DIR/monmap.bin
ceph osd getmap -o $BACKUP_DIR/osdmap.bin
ceph osd getcrushmap -o $BACKUP_DIR/crushmap.bin

# 备份认证信息
echo "Backing up authentication data..."
ceph auth export > $BACKUP_DIR/ceph_auth.txt

# 备份存储池信息
echo "Backing up pool information..."
ceph osd pool ls detail > $BACKUP_DIR/pools.txt
ceph pg dump > $BACKUP_DIR/pg_dump.txt

# 备份集群状态
echo "Backing up cluster status..."
ceph -s > $BACKUP_DIR/cluster_status.txt
ceph health detail > $BACKUP_DIR/health_detail.txt
ceph osd tree > $BACKUP_DIR/osd_tree.txt

# 创建备份清单
echo "Creating backup manifest..."
ls -la $BACKUP_DIR > $BACKUP_DIR/backup_manifest.txt

echo "Backup completed successfully to $BACKUP_DIR"

# 清理旧备份（保留7天）
find /backup/ceph -type d -mtime +7 -exec rm -rf {} \; 2>/dev/null
```

### 17.5 故障恢复脚本
```bash
#!/bin/bash
# emergency_recovery.sh
# Ceph 紧急恢复脚本

LOG_FILE="/var/log/ceph_emergency_recovery.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "$DATE - Emergency recovery script started" >> $LOG_FILE

# 检查集群健康状态
HEALTH=$(ceph health)
echo "$DATE - Cluster health: $HEALTH" >> $LOG_FILE

if [[ $HEALTH == "HEALTH_ERR" ]]; then
    echo "$DATE - Cluster in ERROR state, attempting recovery" >> $LOG_FILE
    
    # 尝试修复所有不一致的 PG
    INCONSISTENT_PGS=$(ceph health detail | grep "inconsistent" | awk '{print $2}' | cut -d. -f1-2)
    
    for pg in $INCONSISTENT_PGS; do
        echo "$DATE - Attempting to repair PG $pg" >> $LOG_FILE
        ceph pg repair $pg 2>&1 >> $LOG_FILE
        sleep 30
    done
    
    # 检查并处理 down 的 OSD
    DOWN_OSDS=$(ceph osd tree | grep "down" | awk '{print $4}')
    
    for osd in $DOWN_OSDS; do
        echo "$DATE - Attempting to restart OSD $osd" >> $LOG_FILE
        systemctl restart ceph-osd@$osd 2>&1 >> $LOG_FILE
        sleep 10
    done
    
    # 强制进行 scrub
    echo "$DATE - Forcing scrub on all PGs" >> $LOG_FILE
    ceph pg scrub $(ceph pg ls | grep active | awk '{print $1}' | head -10) 2>&1 >> $LOG_FILE
    
elif [[ $HEALTH == "HEALTH_WARN" ]]; then
    echo "$DATE - Cluster in WARN state, checking details" >> $LOG_FILE
    ceph health detail >> $LOG_FILE
    
    # 处理常见警告
    SLOW_OPS=$(ceph health detail | grep -c "slow ops")
    if [ $SLOW_OPS -gt 0 ]; then
        echo "$DATE - Found slow operations, checking affected OSDs" >> $LOG_FILE
        ceph health detail | grep "slow ops" >> $LOG_FILE
    fi
fi

echo "$DATE - Emergency recovery script completed" >> $LOG_FILE
```

---

## 💡 最佳实践和技巧

### 日常运维检查清单
```bash
# 每日检查
1. ceph -s                    # 集群整体状态
2. ceph health detail         # 健康详情
3. ceph df                    # 存储使用情况
4. ceph osd df                # OSD使用情况
5. 检查服务状态 systemctl status ceph-*

# 每周检查
1. ceph pg dump_stuck         # 卡住的PG
2. ceph osd perf              # OSD性能
3. 日志检查 journalctl -u ceph-*
4. 磁盘健康检查 smartctl
5. 网络连通性测试

# 每月检查
1. ceph versions              # 版本一致性
2. crush map 优化检查
3. 性能基准测试
4. 备份验证
5. 容量规划评估
```

### 性能优化快速配置
```bash
# OSD 性能优化
ceph config set osd bluestore_cache_size 2147483648    # 2GB
ceph config set osd osd_memory_target 4294967296       # 4GB
ceph config set osd osd_op_threads 8
ceph config set osd osd_max_backfills 1

# 客户端优化
ceph config set client rbd_cache true
ceph config set client rbd_cache_size 67108864         # 64MB

# 恢复优化
ceph config set global osd_recovery_max_active 1
ceph config set global osd_recovery_sleep 0.1
```

---

**本文档持续更新，建议定期检查最新版本。如有任何问题或建议，欢迎反馈。**

*最后更新时间：2024年12月*
