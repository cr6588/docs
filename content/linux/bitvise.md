---
title: "Bitvise"
date: 2018-10-17T09:22:38+08:00
draft: true
---
    
    #bat通过脚本上传文件，cenos7.tlp保存的profile,cenos7.pub保存的host key，sftp.bat脚本
    sftpc -profile="D:\cenos7.tlp" -hostKeyFile="d:\cenos7.pub" -cmdFile="doc\bat\sftp.bat"
    #sftp.bat
    #:覆盖复制文件
    put -o erp-spring-boot\* /data/dockerbuild/erp
    put -o erp-spring-boot\lib\erp-common-1.0.0.jar /data/dockerbuild/erp/lib
    put -o erp-spring-boot\lib\erp-external-1.0.0.jar /data/dockerbuild/erp/lib
    put -o erp-spring-boot\lib\erp-facade-1.0.0.jar /data/dockerbuild/erp/lib
    #:不覆盖复制文件
    put -r erp-spring-boot\lib\* /data/dockerbuild/erp/lib
