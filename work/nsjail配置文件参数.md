```shell
# 配置的名称，用于日志或管理，帮助识别是哪一套配置
name: "default_jail"
# 配置的文字说明
description: "This is a sample nsjail config converted from proto"

# 启动模式
# ONCE 执行一次性的脚本或命令
# STANDALONE_EXECVE 单进程直接执行
# LISTEN_TCP 
mode: ONCE
# 设置NsJail内部看到的主机名 
hostname: "NSJAIL"
# 启动程序的工作路径
cwd: "/"

# 设置为true的时候，可以不改变主机的根目录
no_pivotroot: false
port: 0
bindhost: "::"
max_conns: 0
max_conns_per_ip: 0

# 程序最长运行时间
time_limit: 600
daemon: false
# 限制cpu数
max_cpus: 0
# 控制CPU调度的优先级
nice_level: 19

log_file: "/dev/null"
log_level: INFO

# 控制是否将外部的env注入到jail
keep_env: false
# 传入的环境变量
envar: "DISPLAY"

# 控制是否保留 root 权限
keep_caps: false
# 指定保留哪些 capability
cap: "CAP_SYS_PTRACE"

# 禁用一些错误输出
silent: false
# 阻止 jail 进程创建新会话
skip_setsid: false
# 防止输出错误信息到控制台
stderr_to_null: false
pass_fd: 0
disable_no_new_privs: false
forward_signals: false
disable_tsc: false


# 限制程序使用的系统资源
rlimit_as: 4096
rlimit_as_type: VALUE
rlimit_core: 0
rlimit_core_type: VALUE
rlimit_cpu: 600
rlimit_cpu_type: VALUE
rlimit_fsize: 1
rlimit_fsize_type: VALUE
rlimit_nofile: 32
rlimit_nofile_type: VALUE
rlimit_nproc: 1024
rlimit_nproc_type: SOFT
rlimit_stack: 8
rlimit_stack_type: SOFT
rlimit_memlock: 64
rlimit_memlock_type: SOFT
rlimit_rtprio: 0
rlimit_rtprio_type: SOFT
rlimit_msgqueue: 1024
rlimit_msgqueue_type: SOFT

disable_rl: false

persona_addr_compat_layout: false
persona_mmap_page_zero: false
persona_read_implies_exec: false
persona_addr_limit_3gb: false
persona_addr_no_randomize: false

# 隔离 jail 与外部系统的用户、网络、进程空间
clone_newnet: true
clone_newuser: true
clone_newns: true
clone_newpid: true
clone_newipc: true
clone_newuts: true
clone_newcgroup: true
clone_newtime: false

uidmap {
  inside_id: ""
  outside_id: ""
  count: 1
  # 控制root映射权限
  use_newidmap: false
}
gidmap {
  inside_id: ""
  outside_id: ""
  count: 1
  use_newidmap: false
}

# 挂载/proc
mount_proc: true

mount {
  src: "/bin/bash"
  dst: "/bin/bash"
  is_bind: true
  rw: false
  mandatory: true
}

seccomp_policy_file: "/path/to/policy.txt"
seccomp_log: false

cgroup_mem_max: 0
cgroup_mem_memsw_max: 0
cgroup_mem_swap_max: -1
cgroup_mem_mount: "/sys/fs/cgroup/memory"
cgroup_mem_parent: "NSJAIL"

cgroup_pids_max: 0
cgroup_pids_mount: "/sys/fs/cgroup/pids"
cgroup_pids_parent: "NSJAIL"

cgroup_net_cls_classid: 0
cgroup_net_cls_mount: "/sys/fs/cgroup/net_cls"
cgroup_net_cls_parent: "NSJAIL"

cgroup_cpu_ms_per_sec: 0
cgroup_cpu_mount: "/sys/fs/cgroup/cpu"
cgroup_cpu_parent: "NSJAIL"

cgroupv2_mount: "/sys/fs/cgroup"
use_cgroupv2: false
detect_cgroupv2: false

iface_no_lo: false
iface_own: "eth0"

macvlan_iface: "eth0"
macvlan_vs_ip: "192.168.0.2"

exec {
  path: "/bin/bash"
  arg: "-c"
  arg: "echo Hello from nsjail"
}

```





