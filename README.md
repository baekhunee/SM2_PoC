# 实验名称
SM2_PoC

# 实验简介
Google Password Checkup. PoC impl of the scheme, or do implement analysis by Google

# 实验完成人
权周雨 

学号：202000460021 

git账户名称：baekhunee

# 运行指导
先运行SM2_PoC_server.py，再运行SM2_PoC_client.py，两个文件均可直接运行。

# 实验原理
根据课件指导，具体流程如下所示：

![image](https://user-images.githubusercontent.com/105578152/181284188-32bf9ded-ad79-434c-a2a2-836ff61aca35.png)

# 核心代码说明
## SM2_PoC_server.py
### Process data info
```
# 从客户端接收数据
k,addr = client.recvfrom(1024)
k = int(k.decode(),16)
v,addr = client.recvfrom(1024)
v = int(v.decode(),16)

# create key-value table
table = []
for i in range(20000 ,20000 + pow(2,16)):
    h = sm3._hash(str(i) + str(i))
    table.append(h)   
V = []
for m in table:
    if m[:2] == k:
        V.append((pow(int(m,16),d))%n)

print("V:",V)
```

### Find the data set
```
# 计算H_ab
H_ab=(pow(v,d))%n

# 向客户端发送H_ab与data set
client.sendto(hex(H_ab).encode(),addr)
json_v = json.dumps(V)
client.sendto(json_v.encode('utf-8'),addr)
client.close()
```

## SM2_PoC_client.py
### User input name and password: (u, p)
```
# 客户端计算key-value
h = sm3._hash(str(name)+str(password))
k = h[:2]
v = str((pow(int(h,16),d))%n)

# 向服务端发送k与v
addr = (HOST, PORT)
client.sendto(k.encode('utf-8'), addr)
client.sendto(v.encode('utf-8'), addr)
```
### Username and password detection
```
# 从服务端接收H_ab与data set S
H_ab,addr = client.recvfrom(1024 * 5)
H_ab = int(H_ab.decode(),16)
json_v,addr = client.recvfrom(1024 * 5)
json_v = json_v.decode('utf-8')
S = json.loads(json_v)
print("S:",S)

# 计算并检查H_b是否在S中
H_b = (pow(H_ab,sm3.inv(d,n-1)))%n
tmp = 0
for item in S:
    b = item
    if b == H_b:
        tmp = 1
if tmp == 0:
    print('Account:',name,'Safe!')
else:
    print('Account:',name,'Divulged')
```
# 测试截图
## 服务端
![image](https://user-images.githubusercontent.com/105578152/181287378-7951f957-67b0-4111-a9ed-0b90ae72fe74.png)

## 客户端
![image](https://user-images.githubusercontent.com/105578152/181287349-06523efb-47fc-434f-a62e-90920c2d9292.png)
