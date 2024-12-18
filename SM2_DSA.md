---
title: SM2密钥生成以及数字签名
layout: default
math: katex
---


# SM2推荐椭圆曲线参数：

使用素数域256位椭圆曲线
椭圆曲线方程:  
$$y^2 = x^3 + ax + b$$
```
p= FFFFFFFE FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF 00000000 FFFFFFFF FFFFFFFF  
a= FFFFFFFE FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF 00000000 FFFFFFFF FFFFFFFC  
b= 28E9FA9E 9D9F5E34 4D5A9E4B CF6509A7 F39789F5 15AB8F92 DDBCBD41 4D940E93  
n= FFFFFFFE FFFFFFFF FFFFFFFF FFFFFFFF 7203DF6B 21C6052B 53BBF409 39D54123  
Gx= 32C4AE2C 1F198119 5F990446 6A39C994 8FE30BBF F2660BE1 715A4589 334C74C7  
Gy= BC3736A2 F4F6779C 59BDCEE3 6B692153 D0A9877C C62A4740 02DF32E5 2139F0A0  
```


# SM2 密钥对的生成和公钥的验证
## 密钥对的生成
1. 随机数发生器生成 d ∈ [1,n-2]  
2. G为基点，计算点 P = dG  
3. 密钥对 是 （d, P)。公钥为P，私钥为d。

## 公钥的验证
（Fp上）：
输入：公钥P 以及 椭圆曲线参数集  
输出： 有效/无效

1. 验证 P不是 无穷远点 O  
2. 验证 P的坐标$$(x_p,y_p)$$都是 区间 [0，p-1]中的整数  
3. 验证 $$Y_p^{2}  \equiv x_p^{3} + a x_p + b (mod \ p)$$  
4. 验证 $$ nP = O $$
5. 只有通过所有验证，才输出‘有效’，否则输出’无效‘  

# SM2数字签名
## SM2 数字签名生成 
待签名的 消息为$$M$$,  第一步要求出用户A的 $$Z_A$$  
先约定：  
- 可辩别标识 $$ID_A$$ ，这个标识的 比特长度为 $$entlen_A$$  
- $$ENTL_A$$ 是 整数  $$entlen_A$$  转换的 两个字节，  
> 比如 ID = 0x31323334353637383132333435363738。 这里比特数为128。  
于是 $$ENTL_A$$ = 0x0080  


$$Z_A  = H_{256}(ENTL _A|| ID_A || a || b || x_G || y_G|| x_A || y_A )$$，其中 $$x_A$$ 为公钥 $$P_A$$ 的X坐标，$$y_A$$ 为公钥$$P_A$$的Y坐标。
$$H_256$$即做一次 **SM3** 摘要。 
 
1. 置 
$$ \overline{M} = Z_A || M $$   
2. 计算 哈希值 $$ e = H_v(\overline{M}) $$ 即再做一次 SM3.  
3. 随机数生成 $$ k，k ∈ [1，n-1] $$  
4. 计算出点 $$ kG $$  
5. 计算 $$ r = ( e + kG.x ) mod \ n $$， 若 r = 0 或者 r + k = n则返回 3.  
6. 计算  $$ s = ((1+ d_A) ^ {-1} \cdot  ( k - r \cdot d_A) ) mod n $$ 若 s = 0,则返回 3.  
7. 签名就是 $$ (r,s) $$  

## SM2 数字签名验证 
收到了$$M'$$ 以及 $$(r',s')$$。 公钥为$$P_A$$  
1. 检验 $$ r ∈ [1,n-1] $$  
2. 检验 $$ s ∈ [1,n-1] $$  
3. 置  
$$ \overline{M‘} = Z_A || M’ $$   
> 注：若 $$Z_A$$不是 用户A对应的杂凑值，则验证直接不通过。  
4. 计算 哈希值 $$ e’ = H_v(\overline{M‘}) $$ 即再做一次 SM3.  
5. 计算 $$ t = ( r' +s' ) mod n $$。 若 t = 0 ，则验证不通过  
6. 计算 $$ s'G  + t P_A，记作(x_1'，y_1') $$  
7. 计算 $$ R= (e' + x_1') mod \ n $$ 检验 是否 $$ R = r' 成立$$。成立则验证通过，否则验证不通过。  

# 用户身份标识ID的默认值
《SM2密码算法使用规范 》 GB/T 35276-2017 中 如此描述：  
> 无特殊约定的情况下：$$ID_A$$的长度为**16**字节，  
$$\rm{ID_A  = 0x31323334353637383132333435363738}$$。此时 $$ENTL_A$$ = 0x0080。  
