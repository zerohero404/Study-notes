CGroup 是 Control Groups 的缩写，是 Linux 内核提供的一种可以限制、记录、隔离进程组 (process groups) 所使用的物力资源 (如 cpu memory i/o 等等) 的机制。2007 年进入 Linux 2.6.24 内核，CGroups 不是全新创造的，它将进程管理从 cpuset 中剥离出来，作者是 Google 的 Paul Menage
默认情况下，如果不对容器做任何资源限制，容器会占用系统所有能提供给容器的资源，所以我们需要对 Docker 从内存、CPU、硬盘三个方面进行限制，但是硬盘基本不做限制
OOME:Out Of Memory Exception,一旦发生了 OOME，任何进程都可能会被杀死，包括 docker daemon 这个 docker 主进程，如果 docker daemon 被杀死，所有的容器都会关闭，所以 docker 调整了 docker daemon 的 OOME 优先级，以免被内核关闭

































