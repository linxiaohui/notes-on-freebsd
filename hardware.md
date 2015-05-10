# 系统管理

## 查看系统硬件

   * 查找硬件信息
```bash
dmesg | grep attached	（列出没有被驱动的硬件）
pciconf –lvbce
sysctl -a | grep "^dev\."
devinfo -vr
```
   * 查看CPU
```bash
sysctl hw.model
sysctl  hw.ncpu
dmidecode		(DMI table decoder)
dmidecode –t processor
dmesg | grep "CPU:"
```
   * CPU温度  
在**/boot/loader.conf** 中加入：
```
acpi_ibm_load="YES"
coretemp_load="YES"
acpi_video_load="YES"
```
重启后执行`sysctl -a | grep temper`
输出如
```
hw.acpi.thermal.tz0.temperature: 49.0C
dev.cpu.0.temperature: 52.0C
dev.cpu.1.temperature: 52.0C
dev.cpu.2.temperature: 52.0C
dev.cpu.3.temperature: 52.0C
```
   * 查看内存
```
dmesg | grep "real memory" | awk -F '[( )]' '{print $2,$4,$7,$8}'
```
   * 查看硬盘
     * ```dmesg | grep "sector" | awk '{print $1,$2}'```
     * 查看硬盘信息
```
diskinfo -vt /dev/ada0
disklabel /dev/ada0  
disklabel /dev/ada0s2
```
     * 查看硬盘详细分区信息 & 读写状况  
`gstat`

   * 查看CP支持指令集合
```
grep -i features /var/run/dmesg.boot 
```
输出如
```
Features=0xbfebfbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,
CMOV,PAT,PSE36,CLFLUSH,DTS,ACPI,MMX,FXSR,SSE,SSE2,SS,HTT,TM,PBE>  
Features2=0x98e3fd<SSE3,DTES64,MON,DS_CPL,VMX,SMX,EST,TM2,SSSE3,CX16,xTPR,
PDCM,SSE4.1,SSE4.2,POPCNT>
  AMD Features=0x28100800<SYSCALL,NX,RDTSCP,LM>
  AMD Features2=0x1<LAHF>
```
   * 查看当前操作系统开启的指令集合
```
cd /usr/share/mk
make -V CPUTYPE
输出如core2   （实质上是现实make.conf中定义的CPUTYPE）
make -V MACHINE_CPU
输出如ssse3 sse3 amd64 sse2 sse mmx
```
**备注**：根据**man make**，`make –V`在哪个目录执行是无关的。