# mp4v2_trackdump_poc

## Project address

https://github.com/enzo1982/mp4v2

## POC

[Poc file](https://github.com/10cksYiqiyinHangzhouTechnology/mp4v2_trackdump_poc/blob/main/id_000005%2Csig_08%2Csrc_000166%2B000357%2Ctime_3137250%2Cexecs_3545598%2Cop_splice%2Crep_16)

[afl_trackdump](https://github.com/10cksYiqiyinHangzhouTechnology/mp4v2_trackdump_poc/blob/main/afl_mp4trackdump)

[asan_trackdump](https://github.com/10cksYiqiyinHangzhouTechnology/mp4v2_trackdump_poc/blob/main/asan_mp4trackdump)

## Problem

There has a FPE(Floating Point Exception) in mp4trackdump.cpp:54, function DumpTrack(). Attackers cause denial of service through carefully constructed malicious files.

```
54              msectime /= timescale;
```
I use gdb debug this bug:

```bash
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x427a9e <DumpTrack(void*, unsigned int)+190>:       xor    ecx,ecx
   0x427aa0 <DumpTrack(void*, unsigned int)+192>:       mov    edx,ecx
   0x427aa2 <DumpTrack(void*, unsigned int)+194>:       mov    rdi,QWORD PTR [rbp-0x68]
=> 0x427aa6 <DumpTrack(void*, unsigned int)+198>:       div    rdi
   0x427aa9 <DumpTrack(void*, unsigned int)+201>:       mov    QWORD PTR [rbp-0x38],rax
   0x427aad <DumpTrack(void*, unsigned int)+205>:       cmp    QWORD PTR [rbp-0x38],0x0
   0x427ab2 <DumpTrack(void*, unsigned int)+210>:       jne    0x427aed <DumpTrack(void*, unsigned int)+269>
   0x427ab8 <DumpTrack(void*, unsigned int)+216>:       movabs rax,0x442de8
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe030 --> 0x0
0008| 0x7fffffffe038 --> 0xf80d10 --> 0xf815a0 --> 0x7ffff7f9ec58 --> 0x7ffff7e2fe60 (<mp4v2::platform::io::File::~File()>:     push   rbp)
0016| 0x7fffffffe040 --> 0x0
0024| 0x7fffffffe048 --> 0x7ffff7ec221c (<MP4ReadProvider(char const*, MP4FileProvider const*)+204>:    mov    rax,QWORD PTR [rbp-0x20])
0032| 0x7fffffffe050 --> 0xf80d10 --> 0xf815a0 --> 0x7ffff7f9ec58 --> 0x7ffff7e2fe60 (<mp4v2::platform::io::File::~File()>:     push   rbp)
0040| 0x7fffffffe058 --> 0x1f7fc9130
0048| 0x7fffffffe060 --> 0x7fffffffe100 --> 0x7fffffffe1f0 --> 0x0
0056| 0x7fffffffe068 --> 0x7ffff7ed2a4e (<MP4FindTrackId(MP4FileHandle, uint16_t, char const*, uint8_t)+158>:   mov    ecx,DWORD PTR [rbp-0x58])
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGFPE
0x0000000000427aa6 in DumpTrack (mp4file=0xf80d10, tid=0x1) at /root/mp4v2/build/mp4v2/util/mp4trackdump.cpp:54
54              msectime /= timescale;
gdb-peda$ p timescale
$1 = 0x0
gdb-peda$
```
you can see 'timescale' is 0.It cause the SIGFPE.

## ASAN report

```
(base) ➜  build git:(main) ✗ ./mp4trackdump ../../out1/default/crashes/id:000005,sig:08,src:000166+000357,time:3137250,execs:3545598,op:splice,rep:16

==2100718==ERROR: AddressSanitizer: FPE on unknown address 0x56363644d728 (pc 0x56363644d728 bp 0x7ffd22170500 sp 0x7ffd221704a0 T0)

#0 0x56363644d727 in DumpTrack(void*, unsigned int) (/root/mp4v2/build/mp4v2/build/mp4trackdump+0x2727)
#1 0x56363644e76a in main (/root/mp4v2/build/mp4v2/build/mp4trackdump+0x376a)
#2 0x7f647b7e6082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
#3 0x56363644d54d in _start (/root/mp4v2/build/mp4v2/build/mp4trackdump+0x254d)
```
