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
hostname: "NSJAIL"
cwd: "/"

no_pivotroot: false
port: 0
bindhost: "::"
max_conns: 0
max_conns_per_ip: 0

time_limit: 600
daemon: false
max_cpus: 0
nice_level: 19

log_file: "/dev/null"
log_level: INFO

keep_env: false
envar: "DISPLAY"

keep_caps: false
cap: "CAP_SYS_PTRACE"

silent: false
skip_setsid: false
stderr_to_null: false
pass_fd: 0
disable_no_new_privs: false
forward_signals: false
disable_tsc: false

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
  use_newidmap: false
}
gidmap {
  inside_id: ""
  outside_id: ""
  count: 1
  use_newidmap: false
}

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





