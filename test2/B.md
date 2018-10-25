# 实验步骤:
- 第1步:以system登录到pdborcl，创建角色con_res_viewwl和用户new_userwl，并授权和分配空间(其中：授权new_userwl用户访问users表空间，空间限额是50M)：

![tu](./1-1.png)

- 第2步：新用户new_userwl连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。

![tu](./1-2.png)

- 第3步：用户hr连接到pdborcl，查询new_userwl授予它的视图myview：

![tu](./1-2.png)

