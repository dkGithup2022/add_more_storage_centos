

##  리눅스 - 운영중인 파일시스템의 추가 용량 확보하기 

리눅스의 directory usage 가  100%가 된 상황에서 디스크 추가 공간을 확보하는 과정과 주의점에 대해 다룹니다.

- 중간에 리눅스의 스토리지 관리 구조에 대해 개념적인 내용을 다룹니다.

- 스토리지 용량확보의 실습과 커맨드에 대해 보실 분은 "3. 실습과 커맨드 " 부분부터 보시면 됩니다. 


- 아래 이미지 상황을 해결하는 개념적 지식과 절차에 대해 다룹니다. 

</br>


<img width="555" alt="스토리지_100_df_h" src="https://github.com/dkGithup2022/add_more_storage_centos/assets/104286091/744740dd-4d66-4656-b820-e1310cf6e763">



</br>

---------

### 목차

1. 스토리지 고갈이 골치아픈 이유
   - 이것은 절대 하지 마세요. 
   - 커맨드가 안먹을때, 진짜 진짜 진짜 진짜 최후의 방법


2. 리눅스가 디스크를 인식하는 방법
   - 오버뷰 
   - 용어정리 : 파티션, pv, lv, lvm 등 
   - 그래서 어느 커맨드로 어떤 용량이 확보가 되면 되는거냐


3. 실습과 커맨드 

</br>

개인이 서비스를 운영하다보면 가끔 서비스를 짧게는 1주, 길게는 1달 이상 방치한 채로 두게 될 때도 있습니다. 그리고 이러한 상황에서 종종 로그파일, 로그성 데이터로 인해 스토리지 고갈( disk usage 100%)를 만듭니다.


글쓴이가 이번에 개인적으로 운영하는 파이프라인과 db 를 1달이상 방치해서 생긴 일을 해결하는 과정과 공부내용에 대해 작성합니다.


</br>


*** 주의 *** 
1. 해당 글은 XFS 파일 시스템을 사용하는 CentOs 7 기준으로 작성되었습니다. 다른 환경이라면 커맨드가 다를 수 있습니다. ( 특히 lvm 확장은 커맨드 자체가 다른데, 이것은 아래에서 설명 )
2. 더 세련된 방법이 있을 수 있습니다. 잘못되거나 더 좋은 방법이 있다면 이슈에 남겨주세요. 


</br>


---------


</br>

### 스토리지 고갈이 골치아픈 이유.

 리눅스에서 운영하다보면 여러 문제가 있을 수 있지만 아래의 두가지 이유로 스토리지 고갈은 더 골치가 아픕니다.
 
- 어플리케이션이 실행이 안됨
  - db 데이터 때문에 storage가 100%가 되어도, db 의 내용을 지울 수 없습니다. db 실행에 필요한 로깅 작업이 안되서 db 자체가 켜지지 않습니다.
  

- 몇몇 리눅스 커맨드가 작동하지 않음.
  -  위와 마찬가지 이유로, "쓰는" 작업을 동반하는 커맨드는 실행되지 않을 수 있습니다.
  - 스토리지를 추가하기 위해 디스크를 추가할때, 커맨드 중에 config 를 적는 커맨드가 존재 합니다. 
  - 위와 같은 문제로 고통을 받으신 다면 아래의 "진짜 진짜 진짜 진짜 최후의 방법"을 참고해주세요.


<img width="594" alt="커맨드_불발_vgextend" src="https://github.com/dkGithup2022/add_more_storage_centos/assets/104286091/0e6a69a1-2877-49a7-9e09-52657e3a294a">

ㄴ> 리눅스 커맨드도 동작 안되는게 있다. db 는 db 자체가 안켜짐 .

</br>


#### 이것은 절대 하지 마세요. 

 - db 의 로그를 지우지 마세요.
   - 아마 du 를 통해서 볼 수 있는 가장 사이즈가 큰 로그 파일은 db 로그 파일일 겁니다. 근데 이거 지우면 db 가 부팅 단계에서 에러가 발생 할 수 있습니다. ( 가령, Postgresql 의 wal 로그 지우면 추가적인 조작 없이는 실행 자체가 안됩니다. )

    

</br>

#### 진짜 진짜 진짜 진짜 최후의 방법 : 양심적인 도큐먼트 지우기 

스토리지를 확보하기 위한 커맨드를 실행하기 위해 추가적인 스토리지가 필요할 수 있습니다.
추가적인 pv 를 확보하는 과정에 "쓰기" 연산이 있는데, 스토리지가 100% 라면 실행되지 않습니다.

그러면 파일을 지워서 추가적인 공간을 임시로 확보해야 하는데, centOS 기준 만만한 폴더가 하나 있습니다. 

바로 /usr/share/doc 라고 리눅스 커맨드의 실행파일의 설명서가 html 형태로 저장된 디랙토리가 있습니다.
여기 디렉토리 하나 지우면 커맨드 한번 실행할 정도의 용량은 확보 됩니다....

찝찝해도 급한 불부터 꺼야죠 


- 링크 : https://unix.stackexchange.com/questions/180400/is-it-safe-to-empty-usr-share-doc

- 저는 아래 이미지 처럼 지웠습니다. 

1) 디렉토리  
<img width="1122" alt="usr_share_doc_지우기_1" src="https://github.com/dkGithup2022/add_more_storage_centos/assets/104286091/9cc578b4-d5bc-40b6-958d-c5aaed159545">


2) 지우기
   <img width="658" alt="usr_share_doc_지우기_2" src="https://github.com/dkGithup2022/add_more_storage_centos/assets/104286091/d75eaa77-ca46-4176-a2fe-ff2ca100722a">


</br>


---------


</br>


### 리눅스가 디스크를 인식하는 과정

</br>

#### 오버 뷰

<img width="980" alt="스토리지_파일시스템_구조_도식화" src="https://github.com/dkGithup2022/add_more_storage_centos/assets/104286091/0d0f9235-56c0-4beb-83f1-185f6b51d0bf">

/* image by dankim , 암때나 말없이 가져가시고 틀린거 있음 코맨트 주세요.  */


</br>


연결된 하드디스크가 filesystem 으로 인식되기 까진 위와 같은 구조를 거쳐야 합니다. 

가장 아래가 물리적으로 가까운 부분이고 실제 cd, cp 등의 커맨드로 파일을 조작하는 파일 시스템의 경우 상대적으로 high-level 이다. 

file system 상에서 추가적인 디스크가 인식되게 하려면 아래의 순서로 작업을 진행하게 된다.

1. fdisk & n command  : 추가적인 파티션 생성
2. pvs : 물리 볼륨 생성 확인, fdisk & n 결과가 조회되지 않는다면 pvcreate
3. volume 그룹에 추가
4. logical volumne 에 추가 스토리지 확장 
5. 파일 시스템 포맷 xfs_growfs, resize2fs


이러면 짜잔, 당신이 db 를 운영 중이건 로그를 모으고 있건 해당 파일시스템의 추가적인 용량이 확인 됨을 확인하실 수 있을 것입니다. 디테일한 진행상황은 아래 "실습" 페이지에 이미지와 함께 다룹니다 .


</br>


#### 단어 정리 

1. 물리디스크
   1. 생김새 : /dev/sda
   2. 설명 : 진짜 물리디스크, ssd 든 hdd 든 선연결해서 컴퓨터에 달아 놓으면 /dev/sda 과 같은 형식으로 리눅스에 인식된다. 처음에는 인식만 된 것이고 filesystem으로 사용할 수는 없는 상태이다. 


2. 파티션:
   1. 생김새 : /dev/sda1, /dev/sda2
   2. 설명 : 인식된 물리디스크를 원하는 크기로 잘라서 관리하는 것이다. 이 기능이 있어야 1TB 스토리지 2개를 운영한다고 했을때, 1.5TB 파일시스템 1개, 0.5TB 스토리지 1개 스타일의 운영을 아래의 lvm 기능과 합쳐서 사용할 수 있다.  



3. pv: 
   1. 생김새 : /dev/sda1, /dev/sda2
   2. 설명 : 파티션을 lvm에서 사용할 수 있게 메타데이터를 달아 놓은 것  



4. vg:
    1. 생김새 : centos 등 lvm 에서 설정한 이름.
   2. lvm 에 의해 관리되는 물리적인 파티션 ( pv ) 의 그룹 .



6. lv : 
   1. 생김새: /dev/centos/root 등 lvm 에서 설정한 이름
   2. lvm 에서 관리되는 논리적인 스토리지 관리단위, 실제 파일시스템을 올릴 때, lv 기준으로 올리게 된다.
      가령, lv의 용량을 200GB 확보해야 200GB 가 가용한 폴더를 생성 가능하다. 


5. lvm : pv ~ lv 단계에서 스토리지를 관리하는 리눅스의 기능 .


</br>


#### 그래서 어느 커맨드로 어떤 용량이 확보가 되면 되는거냐


- df-h 가 아래처럼 용량이 있으면 된다.


혼동할 수 있는 비슷한 커맨드 결과로는 lsblk, fdisk 등이 있다. 위의 그림을 보고 어느 포인트까지 도달했는지 확인하는 지표로 삼자.

</br>

Case1 : lsblk 

이거는 파티션을 토폴로지까지 보여주는 커맨드이다. "각 fileSystem이 얼마의 용량을 확보했는가" 보다는 low 한 부분을 조회하는 커맨드이다. 여기서 예상대로 나왔다면 파티션은 잘 인식된다고 보면 된다.



```agsl

[root@lan_centos7_postgresql_02 ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0   800G  0 disk 
├─sda1            8:1    0     1G  0 part /boot
├─sda2            8:2    0   499G  0 part 
│ ├─centos-root 253:0    0 516.1G  0 lvm  /
│ └─centos-swap 253:1    0   7.9G  0 lvm  [SWAP]
└─sda3            8:3    0   100G  0 part 
  └─centos-root 253:0    0 516.1G  0 lvm  /

```

</br>

Case2 : fdisk 

이것도 디스크와 파티션 정보를 출력한다. 여기서 예상대로 나왔다면 파티션은 잘 인식된다고 보면 된다.

```agsl
[root@lan_centos7_postgresql_02 ~]# fdisk -l

Disk /dev/sda: 859.0 GB, 858993459200 bytes, 1677721600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b974b
```

</br>

----

</br>


### 실습과 커맨드 

1. df -h : ( 문제 확인 ) 파일 시스템 별 이용 상황 조회 

```agsl
root@lan_centos7_postgresql_02 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G   89M  3.8G   3% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root  617G  517G  101G  84% /
/dev/sda1               1014M  194M  821M  20% /boot
tmpfs                    783M     0  783M   0% /run/user/0
```

</br>


2. fdisk -l : 파티션 및 물리 디스크 크기 조회 

```agsl
root@lan_centos7_postgresql_02 ~]# fdisk -l

Disk /dev/sda: 859.0 GB, 858993459200 bytes, 1677721600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b974b

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200  1048575999   523238400   8e  Linux LVM
/dev/sda3      1048576000  1258291199   104857600   83  Linux
/dev/sda4      1258291200  1468006399   104857600   83  Linux

```

</br>

3. pvs : 물리 파티션 조회 

```agsl
[root@lan_centos7_postgresql_02 ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree 
  /dev/sda2  centos lvm2 a--  <499.00g     0 
  /dev/sda3  centos lvm2 a--  <100.00g     0 
  /dev/sda4  centos lvm2 a--  <100.00g 49.99
```

</br>

4. fdisk 파티션 인식 

```agsl
/* 위에는 이미 만들었지만 없다면 해당 명령어로 생성한다 .*/
fdisk /dev/sda

[root@lan_centos7_postgresql_02 ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (3 primary, 0 extended, 1 free)
   e   extended
Select (default e): p
Selected partition 4
First sector (1258291200-1677721599, default 1258291200): 
Using default value 1258291200
Last sector, +sectors or +size{K,M,G} (1258291200-1677721599, default 1677721599):  +100G           
Partition 4 of type Linux and of size 100 GiB is set

Command (m for help): p

Disk /dev/sda: 859.0 GB, 858993459200 bytes, 1677721600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b974b

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200  1048575999   523238400   8e  Linux LVM
/dev/sda3      1048576000  1258291199   104857600   83  Linux
/dev/sda4      1258291200  1468006399   104857600   83  Linux

Command (m for help): w
The partition table has been altered!

```

</br>

/* 이 시점에선 아직 마운트된 볼륨으론 안보임    */

```agsl
[root@lan_centos7_postgresql_02 ~]# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda3
[root@lan_centos7_postgresql_02 ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree  
  /dev/sda2  centos lvm2 a--  <499.00g      0 
  /dev/sda3  centos lvm2 a--  <100.00g <50.00g
```


</br>

5. pv 확인 

( 글쓴이 환경에선 pvcreate 없어도 파티션만드니까 바로 pv 로 인식이 되었음. 만약 pvdisplay 조회가 안된다면 pvcreate 로 별도로 생성  )


```agsl
root@lan_centos7_postgresql_02 ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <499.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              127743ㅍㅎㅇ
  Free PE               0
  Allocated PE          127743
  PV UUID               egbkGW-OZsD-k3Lm-Qqx2-GIAU-nhyu-qa9KAa
   
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               centos
  PV Size               100.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              25599
  Free PE               12799
  Allocated PE          12800
  PV UUID               c8YRiD-I5Fk-dXqA-6n4n-XdBZ-2RJm-898hfk
   
  "/dev/sda4" is a new physical volume of "100.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda4
  VG Name               
  PV Size               100.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               hiwMXi-X0Zx-zgWp-BFrp-m6L6-sdEM-6ycwMM
```

6. vg 확인 및 확장 

- vg display

```agsl

[root@lan_centos7_postgresql_02 ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  9
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               598.99 GiB
  PE Size               4.00 MiB
  Total PE              153342
  Alloc PE / Size       140543 / <549.00 GiB
  Free  PE / Size       12799 / <50.00 GiB
  VG UUID               e99U2a-SlFj-fSYF-L345-eXJi-SWoU-QKZtMk

```

</br>


- vg extend 
```
[root@lan_centos7_postgresql_02 doc]# vgextend centos /dev/sda4
  Volume group "centos" successfully extended
```


/* 주의 : 이 단계에서 아래와 같은 에러가 난다면 위의 "진짜 진짜 진짜 최후의 수단을 참고하세요." */

```agsl
[root@lan_centos7_postgresql_02 ~]# vgextend centos /dev/sda4
  Couldn't create temporary archive name.
```

위처럼 vggroup 으로 파티션이 인식이 된다면 이제 lv 로 용량을 합칠 준비가 된거다.

</br>

7. mount 명령어로 현재 사용할 파일시스템 종류 확인

용량을 합치기 전에 현재 lvm 에서 사용하는 파일시스템을 조회한다.

글쓴이 환경에선 xfs 로 나온다. 

```agsl
[root@lan_centos7_postgresql_02 /]# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=3992356k,nr_inodes=998089,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,mode=755)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,seclabel,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/centos-root on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=25,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10044)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=800836k,mode=700)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
```

</br>

8. lvm 추가 사이즈 할당 

- lvdisplay 로 lv path 확인하기

```
[root@lan_centos7_postgresql_02 /]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                h75itL-DeDx-fRkn-V5mU-i27Q-OW1W-vpEjZF
  LV Write Access        read/write
  LV Creation host, time localhost, 2022-06-27 19:31:36 +0900
  LV Status              available
  # open                 2
  LV Size                <7.88 GiB
  Current LE             2016
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                PXXAJW-qfdO-d0NZ-8r3D-bDsj-t7CQ-VKKBAb
  LV Write Access        read/write
  LV Creation host, time localhost, 2022-06-27 19:31:37 +0900
  LV Status              available
  # open                 1
  LV Size                516.12 GiB
  Current LE             132127
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

</br>

- lv 용량 확장 
  - 같은 vg group 의 용량에 남은게 있다면 확장이 가능하다..

   
```agsl
[root@lan_centos7_postgresql_02 mnt]# lvextend -L +100G /dev/centos/root
  Size of logical volume centos/root changed from 516.12 GiB (132127 extents) to 616.12 GiB (157727 extents).
```

</br>

9. 파일시스템 크기 조정 

위와 같은 작업을 거쳐도, df 명령어엔 아직도 500GB 가 최대의 크기로 나온다. lv 에 용량을 한다음 파일시스템 크기 조정하는 커맨드를 입력해야 합니다.

</br>

- 파일 시스템 용량 조회 : 아직 lv 에 추가할당한 용량이 적용되지 않았다.
```agsl
[root@lan_centos7_postgresql_02 mnt]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                   tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                   tmpfs     3.9G   89M  3.8G   3% /run
tmpfs                   tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs       517G  517G  332K 100% /
```

</br>

- 파일시스템 크기조정 : xfs_growfs /dev/mapper/centos-root

```agsl
[root@lan_centos7_postgresql_02 mnt]# xfs_growfs /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=512    agcount=42, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=135298048, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 135298048 to 161512448


[root@lan_centos7_postgresql_02 mnt]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G   89M  3.8G   3% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root  617G  517G  101G  84% /
/dev/sda1               1014M  194M  821M  20% /boot
tmpfs                    783M     0  783M   0% /run/user/0
```


파일시스템 크기조정까지 마쳐야 정상적으로 파일시스템의 용량이 확보됨을 확인할 수 있습니다.

이상으로 운영중인 서버의 스토리지 공간 확보 내용에 대해 마치겠습니다 :) 