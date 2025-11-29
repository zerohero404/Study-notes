CGroup 是 Control Groups 的缩写，是 Linux 内核提供的一种可以限制、记录、隔离进程组 (process groups) 所使用的物力资源 (如 cpu memory i/o 等等) 的机制。2007 年进入 Linux 2.6.24 内核，CGroups 不是全新创造的，它将进程管理从 cpuset 中剥离出来，作者是 Google 的 Paul Menage
默认情况下，如果不对容器做任何资源限制，容器会占用系统所有能提供给容器的资源，所以我们需要对 Docker 从内存、CPU、硬盘三个方面进行限制，但是硬盘基本不做限制
OOME:Out Of Memory Exception,一旦发生了 OOME，任何进程都可能会被杀死，包括 docker daemon 这个 docker 主进程，如果 docker daemon 被杀死，所有的容器都会关闭，所以 docker 调整了 docker daemon 的 OOME 优先级，以免被内核关闭

# 1. 内存资源限制
- 为应用做内存压力测试，理解正常业务需求下使用的内存情况，然后才能进入生产环境使用
- 一定要限制容器的内存使用上限
- 尽量保证主机的资源充足，一旦通过监控发现资源不足，就进行扩容或者对容器进行迁移
- 如果可以（内存资源充足的情况），尽量不要使用 swap，swap 的使用会导致内存计算复杂，对调度器非常不友好
- 设置方式<br>
在 docker 启动参数中，和内存限制有关的包括（参数的值一般是内存大小，也就是一个正数，后面跟着内存单位 b、k、m、g，分别对应 bytes、KB、MB、和 GB）
  - -m --memory：容器能使用的最大内存大小，最小值为 4m
  - --memory-swap：容器能够使用的 swap 大小
  - --memory-swappiness：默认情况下，主机可以把容器使用的匿名页（anonymous page）swap 出来，你可以设置一个 0-100 之间的值，代表 swap 使用的比例
  - --memory-reservation：设置一个内存使用的 soft limit （软限制，在内存资源足够的时候可以超过这个值，但是当内存资源不够的时候就会低于这个值）,设置值小于 –m 设置
  - --kernel-memory：容器能够使用的 kernel memory（容器内核内存） 大小，最小值为 4m
  - --oom-kill-disable：是否运行 OOM 的时候杀死容器。只有设置了 -m，才可以把这个选项设置为 false， 否则容器会耗尽主机内存，而且导致主机应用被杀死（不建议使用）
































