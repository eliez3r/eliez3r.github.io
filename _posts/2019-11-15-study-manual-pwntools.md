---
title: "[python]pwntools 정리"
tags: [python, manual, pwntools]
author: eli_ez3r
key: 20191115
modify_date: 2019-11-15
article_header:
  type: cover
  image:
    src: /assets/img/manual/pwntools_logo.png
---

## 0x00. pwntools?

pwntools는 말 그대로 포너블 툴 킷으로 익스플로잇을 편하게 하기 위해서 사용됩니다.

파이썬으로 작성되어 있으며 `from pwn import *`을 선언해서 사용할 수 있습니다. Gallopsled라는 팀에 의해서 만들어 졌다고 하네요.

pwntools에서 지원해 주는 것은 assemble, disassemble, rop, elf, ssh등이 있습니다.

-----

## 0x01. Install

pwntools은 파이썬의 모듈이기 때문에 기본적으로 파이썬이 설치되어 있어야 한다.

```
# pip install pwntools
```

리눅스에 설치를 진행한다면 `libcapstone-dev`를 설치해주어야 disasm에러가 발생하지 않는다.

```
# apt-get update
# apt-get install libcapstone-dev
```

-----

## 0x02. Usage

#### 1) import

pwntools를 사용하기 위해서 파이썬 스크립트 상단에 import해주어야 한다.

```python
from pwn import *
```

-----

### 1. 대상과 연결하기

#### 2) remote()

nc로 원격지에 연결하기 위한 코드이다. ip는 string이고 port는 int이다.

```python
# remote(ip, port)
conn = remote("localhost", 8888)
```



##### 비교

```python
# 기존 파이썬에서 소켓에 연결할 때의 코드
from socket import *

s = socket(AF_INET, SOCK_STREAM)
s.connect('localhost', 8888)
```



#### 3) process()

로컬에 있는 파일을 연결하기 위한 코드이다.

```python
# process(path)
p = process("./example")
```



#### 4) ssh()

ssh 연결을 위한 코드이다.  username, ip, password는 string이고 port는 int이다.

port와 password는 `=`를 붙여준다. port는 default로 `22`이고, password는 default로 `guest`이다.

```python
# ssh(username, ip, port, password)
conn = ssh("username", "localhost", port=8888, password="example")
```



##### example

```python
from pwn import *

conn = ssh("fd", "pwnable.kr", port=2222, password="guest")
print conn['ls']

sh = conn.run('/bin/sh')
sh.sendline("echo hi")

conn.close()
```

-----

### 2. 대상으로부터 데이터 받기

#### 1) recv()

연결된 대상으로 부터 데이터를 받는 코드이다.

```python
conn = remote("localhost", 8888)
message = conn.recv()
message = conn.recv(4)		# 4byte 만큼 받아온다.
```



#### 2) recvline()

연결된 대상으로 부터 데이터 **한 줄**을 받아온다.

```python
conn = remote("localhost", 8888)
message = conn.recvline()
```



#### 3) recvuntil(value)

괄호안에 있는 부분까지 데이터를 받는다.

```python
conn = remote("localhost", 8888)
message = conn.recvuntil("\n")	# 개행까지 데이터를 받아온다.
```

-----

### 3. 대상으로 데이터 보내기

#### 1) send()

연결된 대상으로 데이터를 보낸다. 

```python
conn = remote("localhost", 888)
conn.send(value)
```

> _read() 함수에 사용



#### 2) sendline(value)

연결된 대상으로 데이터를 **한 줄** 보낸다.

```python
conn = remote("localhost", 8888)
conn.sendline(value)
```

> _scanf(), gets(), fgets 등에 사용

-----

### 4. packing  관련 함수

#### 1) p32(value)

32bit little endian 방식으로 packing 해주는 함수. `p32(value, endian='big')`을 하면 big endiang으로 패킹해준다.

```python
tmp = p32(0x12345678)	# \x78\x56\x34\x12
tmp = p32(abcd)			# \x44\x43\x42\x41
```



#### 2) p64(value)

64bit little endian 방식으로 packing 해주는 함수. `p64(value, endian='big')`을 하면 big endiang으로 패킹해준다.

```python
tmp = p64(0x12345678)	# \x00\x00\x00\x00\x78\x56\x34\x12
tmp = p64(abcd)			# \x00\x00\x00\x00\x44\x43\x42\x41
```

-----

### 5. unpacking 관련 함수

#### 1) u32(str)

32bit little endian 방식으로 unpacking해주는 함수. 반환 값은 `int형`이다. 그리고 str에는 패킹된 `string`이 들어가야 한다.

```python
tmp = u32("\x78\x56\x34\x12")	# 305419896(0x12345678)
```



#### 2) u64(str)

64bit little endian 방식으로 unpacking해주는 함수. 반환 값은 `int형`이다. 그리고 str에는 패킹된 `string`이 들어가야 한다.

```python
tmp = u64("\x00\x00\x00\x00\x78\x56\x34\x12")	# 305419896(0x12345678)
```

-----

### 6. interactive 함수

#### 1) interactive()

쉘과 직접적으로 명령을 전송, 수신할 수 있는 함수

```python
conn = remote("localhost", 8888)
# ... code ...
conn.interactive()
```

-----

### 7. ELF 함수

#### 1) elf(path)

ELF에 바이너리를 인식시켜 적용되어 있는 보호기법을 보여주고, plt, got같은 elf 정보등을 가져올 수 있는 함수

```python
elf = ELF("./filename")
plt = elf.plt
got = elf.got
symbols = elf.symbols
```

```
>>> from pwn import *
>>> elf = ELF("./bof")
[*] '/root/pwnable/bof/bof'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

```
>>> plt = elf.plt
>>> print plt
{u'__gmon_start__': 1296, u'puts': 1264, u'system': 1280, u'__stack_chk_fail': 1232, 
u'__cxa_finalize': 1248, u'__libc_start_main': 1312, u'gets': 1216}
```

```
>>> got = elf.got
>>> print got
{u'__gmon_start__': 8212, u'puts': 8204, u'_Jv_RegisterClasses': 8176, 
u'system': 8208, u'__stack_chk_fail': 8196, u'__cxa_finalize': 8200, 
u'__libc_start_main': 8216, u'gets': 8192}
```

```
>>> symbols = elf.symbols
>>> print symbols
{'': 8228, u'_IO_stdin_used': 1928, u'_init': 1140, 
u'dtor_idx.6161': 8232, u'got.__libc_start_main': 8216, 
u'__libc_start_main': 1312, u'__dso_handle': 8224, 
u'_edata': 8228, u'data_start': 8220, 
u'__init_array_end': 7936, u'_fini': 1896, 
u'got.system': 8208, u'_start': 1328, u'__FRAME_END__': 2220, 
u'puts': 1264, u'plt.system': 1280, u'__DTOR_END__': 7948, 
u'got.puts': 8204, u'system': 1280, u'__JCR_END__': 7952, u'_end': 8236, u'__do_global_ctors_aux': 1840, 
u'__libc_csu_init': 1712, u'got.gets': 8192, 
u'_Jv_RegisterClasses': 8176, u'main': 1674, 
u'plt.__gmon_start__': 1296, u'plt.__cxa_finalize': 1248, 
u'_fp_hw': 1924, u'__CTOR_LIST__': 7936, 
u'got._Jv_RegisterClasses': 8176, u'_DYNAMIC': 7956, 
u'__libc_csu_fini': 1824, u'plt.gets': 1216, 
u'__cxa_finalize': 1248, u'frame_dummy': 1520, 
u'__DTOR_LIST__': 7944, u'plt.puts': 1264, 
u'_GLOBAL_OFFSET_TABLE_': 8180, u'func': 1580, 
u'completed.6159': 8228, u'gets': 1216, 
u'__gmon_start__': 1296, u'__CTOR_END__': 7940, 
u'got.__stack_chk_fail': 8196, u'__do_global_dtors_aux': 1392, 
u'got.__gmon_start__': 8212, u'__stack_chk_fail': 1232, 
u'__init_array_start': 7936, u'got.__cxa_finalize': 8200, 
u'plt.__libc_start_main': 1312, u'plt.__stack_chk_fail': 1232, 
u'__JCR_LIST__': 7952, u'__bss_start': 8228, 
u'__i686.get_pc_thunk.bx': 1575, u'__data_start': 8220}
```

-----

