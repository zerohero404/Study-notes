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
<img width="913" height="322" alt="Linux：虚拟化42" src="https://github.com/user-attachments/assets/0908f452-3364-408a-937c-96574240cdcf" />

# 2. CPU 资源限制
Docker 提供的 CPU 资源限制选项可以在多核系统上限制容器能利用哪些虚拟 CPU。而对容器最多能使用的 CPU 时间有两种限制方式

- 有多个 CPU 密集型的容器竞争 CPU 时，设置各个容器能使用的 CPU 时间相对比例
- 以绝对的方式设置容器在每个调度周期内最多能使用的 CPU 时间
- CPU 限制方式
  - --cpuset-cpus=""
    - 允许使用的 CPU 集，你的服务器有多少线程就有多少个虚拟 CPU，这个值就是从 0 到你线程减一，例如 0 就是第一个虚拟 CPU，0,3 就是第一个和第四个虚拟CPU，0-3 就是第一个到第四个虚拟 CPU
  - -c,--cpu-shares=0
    - CPU 共享权值（相对权重），默认值 1024，例如两个容器一个是 1024 一个是2048，那么服务器的 CPU 权重就是 1:2
  - --cpuset-mems=""
    - 允许在上执行的内存节点（MEMs）
  - --cpu-period=0
    - 即可设置调度周期，CFS 周期的有效范围是 1ms~1s，对应的–cpu-period的数值范围是1000~1000000，此命令常与 --cpu-quota=0 命令搭配使用
  - --cpu-quota=0
    - 设置在每个周期内容器能使用的 CPU 时间，容器的 CPU 配额必须不小于 1ms，即 --cpu-quota 的值必须 >= 1000，单位微秒
    - 此命令与 --cpu-period=0 搭配使用，举例
      - docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:16.04 /bin/bash # 这个命令就是设置这个容器只使用 50% 的虚拟 CPU，因为 --cpu-quota=25000 是 --cpu-period=50000 的一半
      - docker run -it --cpu-period=10000 --cpu-quota=20000 ubuntu:16.04 /bin/bash # 这个命令就是设置这个容器使用 200% 的虚拟 CPU，也就是两个 CPU ，因为 --cpu-quota=25000 是 --cpu-period=50000 的两倍
  - --cpus
    - 能够限制容器可以使用的主机 CPU 个数，并且还可以指定如 1.5 之类的小数，是随机使用虚拟 CPU，但总数一定是设置的值，比如 1.5 ，可能是 0、2、4 三个虚拟 CPU 各给 0.5 组成 1.5

- 限制实验
  - docker run --name stress -it --rm -m 256m lorel/docker-stress-ng:latest stress -vm 2
    - -m 256m stress 是一个无限占用资源的镜像，这条命令就是限制容器内存不超过 256 m
  - docker run --name stress -it --rm --cpus 2 lorel/docker-stress-ng:latest stress --cpu 8
    - --cpus 2 限制容器不超过 200%，--cpu 8 使用8线程
  - docker run --name stress -it --rm --cpuset-cpus 0 lorel/docker-stress-ng:latest stress --cpu 8
    - --cpuset-cpus 0 设置容器的使用第一个虚拟 CPU
  - docker run --name stress -it --rm --cpu-period=10000 --cpu-quota=20000 lorel/docker-stress-ng:latest stress --cpu 8
    - 设置容器使用两个虚拟 CPU，也就是 200%






























