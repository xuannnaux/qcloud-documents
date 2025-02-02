本文以已绑定到代理、存在加密数据的MySQL元数据为例，介绍如何解除绑定和删除元数据。

## 元数据配置现状
* `casbtest`库下`t1`表中存在三个敏感字段`name`、`phone`和`address`。
* 数据库地址为:`172.16.48.12:3306`，已绑定的代理地址为`172.16.0.30:10101`。
* 数据库中的三个敏感字段`name`、`phone`和`address`已使用CASB代理配置加密。
  ![](https://qcloudimg.tencent-cloud.cn/raw/92986b8ca38db36b8f7b50138e3e2130.png)
* 针对代理账号`root`，`phone`字段上已配置脱敏规则。
  ![](https://qcloudimg.tencent-cloud.cn/raw/3b478a4402256cb97138bd30f0865a65.png)
* 数据库中存在已通过CASB加密后的数据。
  * 直连数据库查询: 所有敏感字段已加密。
  ![](https://qcloudimg.tencent-cloud.cn/raw/e9579bc9b5bfa5ef35e4a10190d7c24d.png)
  * 通过代理查询：`phone`字段已脱敏，其余字段自动解密为明文。 
  ![](https://qcloudimg.tencent-cloud.cn/raw/d9262f4ff91b6c47eacc60165af739b9.png)

## 步骤1：配置敏感字段加密策略的工作模式，不加密增量数据
1. 参考[策略管理](https://cloud.tencent.com/document/product/1303/64619)，配置敏感字段的工作模式为`读解密写不加密`。
   ![](https://qcloudimg.tencent-cloud.cn/raw/1c1ac0358b17f6cd9e56544c0b986847.png)
2. 通过代理写入的增量数据时，可以正常写入并自动解密和脱敏。
   ![](https://qcloudimg.tencent-cloud.cn/raw/b77ba39c04f9c202798a5b9f0fa38883.png)
3. 直连DB查询，代理写入的增量数据为明文存储。
   ![](https://qcloudimg.tencent-cloud.cn/raw/729aa29652ab838cb32555b37a9b684f.png)

>! 加密字段作为查询条件时，代理仅使用密文值作为查询条件，通过代理无法匹配到明文存储的增量数据。

## 步骤2：全量解密存量数据
1. 参考[任务管理](https://cloud.tencent.com/document/product/1303/64622)，新建全量解密任务，全量解密敏感字段。
   ![](https://qcloudimg.tencent-cloud.cn/raw/a32fd9d12cb94a27f33c2abf15c70711.png)
2. 全量解密任务完成后，直连DB查询，敏感字段的所有数据均已解密为明文。
    ![](https://qcloudimg.tencent-cloud.cn/raw/b55f6a5fbb4a03bd17c53c130224b40c.png)

## 步骤3：切换数据库连接
1. 数据库中的所有数据均解密成明文后，业务可以将数据库连接由CASB代理地址切换为数据库地址，后续所有数据库操作直连数据库处理。

## 步骤4：策略清理和解除绑定
1. 清理加密策略。参考[删除加密策略](https://cloud.tencent.com/document/product/1303/64619#.E5.88.A0.E9.99.A4.E7.AD.96.E7.95.A5)文档，删除元数据上所有已配置的加密策略。
2. 清理代理账号配置的脱敏策略。参考[删除脱敏策略](https://cloud.tencent.com/document/product/1303/56902) 文档，删除元数据对应的`所有代理账号`上已配置脱敏策略。
3. 解除代理和元数据绑定。参考[代理资源管理](https://cloud.tencent.com/document/product/1303/64636#.E8.A7.A3.E7.BB.91.E4.BB.A3.E7.90.86.E5.92.8C.E5.85.83.E6.95.B0.E6.8D.AE)文档，解除元数据和代理的绑定。

## 步骤5：删除元数据

1. 清理完策略和代理绑定后，即可从CASB系统中删除元数据。