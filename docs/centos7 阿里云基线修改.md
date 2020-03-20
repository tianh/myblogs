# centos7 阿里云基线修改

标签（空格分隔）： 基线 

---


```
grep "PASS_MIN_DAYS" /etc/login.defs
sed -i  "/^PASS_MIN_DAYS/c PASS_MIN_DAYS   7" /etc/login.defs
chage --mindays 7 root
grep "PASS_MIN_DAYS" /etc/login.defs
grep "ClientAlive" /etc/ssh/sshd_config
sed -i  "/#ClientAliveCountMax/c ClientAliveCountMax 2" /etc/ssh/sshd_config
sed -i  "/#ClientAliveInterval/c ClientAliveInterval 600" /etc/ssh/sshd_config
grep "ClientAlive" /etc/ssh/sshd_config
ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow
chown root:root /etc/passwd /etc/shadow /etc/group /etc/gshadow;chmod 0644 /etc/group;chmod 0644 /etc/passwd;chmod 0400 /etc/shadow;chmod 0400 /etc/gshadow
ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow
grep "password    sufficient" /etc/pam.d/system-auth
grep "password    sufficient" /etc/pam.d/password-auth
sed -i "/password    sufficient/ s/$/ remember=5/" /etc/pam.d/system-auth
sed -i "/password    sufficient/ s/$/ remember=5/" /etc/pam.d/password-auth
grep "password    sufficient" /etc/pam.d/system-auth
grep "password    sufficient" /etc/pam.d/password-auth
egrep "minclass|minlen" /etc/security/pwquality.conf
sed -i "/# minlen/c minlen=10" /etc/security/pwquality.conf
sed -i "/# minclass/c minclass=3" /etc/security/pwquality.conf
egrep "minclass|minlen" /etc/security/pwquality.conf
```

## 解析
```
# 设置密码修改最小间隔时间 | 身份鉴别

描述

设置密码修改最小间隔时间，限制密码更改过于频繁

检查提示

--

加固建议

在 /etc/login.defs 中将 PASS_MIN_DAYS 参数设置为7-14之间,建议为7：

PASS_MIN_DAYS 7
需同时执行命令为root用户设置：

chage --mindays 7 root
操作时建议做好记录或备份


设置SSH空闲超时退出时间 | 服务配置
描述

设置SSH空闲超时退出时间,可降低未授权用户访问其他用户ssh会话的风险

检查提示

--

加固建议

编辑/etc/ssh/sshd_config，将ClientAliveInterval 设置为300到900，即5-15分钟，将ClientAliveCountMax设置为0-3之间。

ClientAliveInterval 600
ClientAliveCountMax 2
操作时建议做好记录或备份


设置用户权限配置文件的权限 | 文件权限
描述

设置用户权限配置文件的权限

检查提示

--

加固建议

执行以下5条命令

chown root:root /etc/passwd /etc/shadow /etc/group /etc/gshadow
chmod 0644 /etc/group  
chmod 0644 /etc/passwd  
chmod 0400 /etc/shadow  
chmod 0400 /etc/gshadow  
操作时建议做好记录或备份


检查密码重用是否受限制 | 身份鉴别
描述

强制用户不重用最近使用的密码，降低密码猜测攻击风险

检查提示

--

加固建议

在/etc/pam.d/password-auth和/etc/pam.d/system-auth中password sufficient pam_unix.so 这行的末尾配置remember参数为5-24之间，原来的内容不用更改，只在末尾加了remember=5。

操作时建议做好记录或备份

# 密码复杂度检查 | 身份鉴别
描述

检查密码长度和密码是否使用多种字符类型

检查提示

--

加固建议

编辑/etc/security/pwquality.conf，把minlen（密码最小长度）设置为9-32位，把minclass（至少包含小写字母、大写字母、数字、特殊字符等4类字符中等3类或4类）设置为3或4。如：

minlen=10
minclass=3
```


