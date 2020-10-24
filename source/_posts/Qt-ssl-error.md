---
title: Qt ssl error
date: 2020-06-29 16:35:45
tags: Qt 
---


Qt 5.14.2 Ubutun 16.04.6 运行时报错ssl相关
类似下面这样的报错信息
<!--More-->
```
qt.network.ssl: QSslSocket: cannot resolve CRYPTO_num_locks
qt.network.ssl: QSslSocket: cannot resolve CRYPTO_set_id_callback
qt.network.ssl: QSslSocket: cannot resolve CRYPTO_set_locking_callback
qt.network.ssl: QSslSocket: cannot resolve ERR_free_strings
qt.network.ssl: QSslSocket: cannot resolve EVP_CIPHER_CTX_cleanup
qt.network.ssl: QSslSocket: cannot resolve EVP_CIPHER_CTX_init
qt.network.ssl: QSslSocket: cannot resolve sk_new_null
qt.network.ssl: QSslSocket: cannot resolve sk_push
qt.network.ssl: QSslSocket: cannot resolve sk_free
qt.network.ssl: QSslSocket: cannot resolve sk_num
qt.network.ssl: QSslSocket: cannot resolve sk_pop_free
qt.network.ssl: QSslSocket: cannot resolve sk_value
qt.network.ssl: QSslSocket: cannot resolve SSL_library_init
qt.network.ssl: QSslSocket: cannot resolve SSL_load_error_strings
qt.network.ssl: QSslSocket: cannot resolve SSL_get_ex_new_index
qt.network.ssl: QSslSocket: cannot resolve SSLv3_client_method
qt.network.ssl: QSslSocket: cannot resolve SSLv23_client_method
qt.network.ssl: QSslSocket: cannot resolve SSLv3_server_method
qt.network.ssl: QSslSocket: cannot resolve SSLv23_server_method
qt.network.ssl: QSslSocket: cannot resolve X509_STORE_CTX_get_chain
qt.network.ssl: QSslSocket: cannot resolve OPENSSL_add_all_algorithms_noconf
qt.network.ssl: QSslSocket: cannot resolve OPENSSL_add_all_algorithms_conf
qt.network.ssl: QSslSocket: cannot resolve SSLeay
qt.network.ssl: Incompatible version of OpenSSL
```
自己下载 openssl 编译一遍 然后复制到qt 的库引用位置就行
为了明确引用版本 在项目里 执行
```
    qDebug() << QSslSocket::supportsSsl() << QSslSocket::sslLibraryBuildVersionString() << QSslSocket::sslLibraryVersionString();
```

我自己的环境运行结果如下

```
true "OpenSSL 1.1.1d  10 Sep 2019" "OpenSSL 1.1.1h-dev  xx XXX xxxx"
```

查看一下需要引用的版本，我使用的Qt 5.14.2 使用的是 1.1.1 的版本

在 [GitHub](https://github.com/openssl/openssl)上找到 openssl ,clone 到本地，找到需要的分支，比如我的需要的分支是 OpenSSL_1_1_1-stable，切换到这个分支

```
git checkout -b openssl1.1.1 origin/OpenSSL_1_1_1-stable
```
然后编译

```
./config enable-shared
make -j4
```

然后在把编译后生成的lib 拷贝到 qt 引用目录

```
cp libssl.so* libcrypto.so* ~/Qt5.14.2/5.14.2/gcc_64/lib/ -a
```
注意后面要换成你机器上安装Qt的目录

再次编译项目就可以了就没有报错了


#### 参考

- [https://blog.csdn.net/u010168781/article/details/85632637](https://blog.csdn.net/u010168781/article/details/85632637)
- [https://bugreports.qt.io/browse/QTBUG-68156?focusedCommentId=424062&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel](https://bugreports.qt.io/browse/QTBUG-68156?focusedCommentId=424062&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel)
- [https://www.tal.org/tutorials/openssl_1_1_1_and_qt_5_12_4](https://www.tal.org/tutorials/openssl_1_1_1_and_qt_5_12_4)
- [https://www.cnblogs.com/cute/p/12617952.html](https://www.cnblogs.com/cute/p/12617952.html)
- [https://github.com/openssl/openssl](https://github.com/openssl/openssl)