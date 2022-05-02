安裝 redis
========
__安裝__
```bash
yum -y install redis
```

__啟動__
```
systemctl start redis
```
在linux環境中啟用很順利，但在docker中可能會遇到錯誤
> Failed to get D-Bus connection: Operation not permitted

因為container不是在privileged mode  
可參考: https://stackoverflow.com/questions/50393525/failed-to-get-d-bus-connection-operation-not-permitted
```
docker run -d -it --privileged {Image ID} /usr/sbin/init
```

安裝 php redis library
========
在實際linux環境中安裝出現的問題較少，以下為在docker環境中安裝步驟

1.
```bash
pecl install redis
```
報錯:
> No releases available for package "pecl.php.net/redis"  
> install failed


2.
```bash
pecl update-channels 
```
成功後再下`pecl install redis`
報錯:
> running: phpize
> Can't find PHP headers in /opt/rh/rh-php73/root/usr/include/php  
> The rh-php73-php-devel package is required for use of this command.  
> ERROR: phpize' failed


3.
```bash
yum install rh-php73-php-devel
```
安裝成功後再安裝redis，`pecl install redis`
報錯:
> checking for igbinary includes... configure: error: Cannot find igbinary.h  
> ERROR: /var/tmp/redis/configure --with-php-config=/opt/rh/rh-php73/root/usr/bin/php-config  
> --enable-redis-igbinary=?yes --enable-redis-lzf=yes --enable-redis-zstd=yes' failed


4.
```bash
pecl install igbinary
```
報錯:
> running: make  
> sh: make: command not found  
> ERROR: make' failed


5.
```bash
yum -y install make
```
安裝完再安裝
```bash
pecl install igbinary
```
安裝完會提示要加入擴展
> configuration option "php_ini" is not set to php.ini location  
> You should add "extension=igbinary.so" to php.ini

加完後再次安裝redis，`pecl install redis`
報錯:
> checking for libzstd files in default path... not found  
> configure: error: Please reinstall the libzstd distribution  
> ERROR: /var/tmp/redis/configure --with-php-config=/opt/rh/rh-php73/root/usr/bin/php-config  
> --enable-redis-igbinary=yes --enable-redis-lzf=yes --enable-redis-zstd=yes' failed


6.
此步驟是依據參考連結而來  
應該是多餘的，不過從缺什麼裝什麼的角度來說  
缺libzstd下`pecl install zstd`應該是合理會想到的步驟
```bash
pecl install zstd
```
> configuration option "php_ini" is not set to php.ini location  
> You should add "extension=zstd.so" to php.ini

安裝完擴展還是找不到libzstd path


7.
```bash
yum install epel-release
yum install libzstd-devel
```
安裝完最後再次安裝
```bash
pecl install redis
```
安裝完提醒要加擴展
> configuration option "php_ini" is not set to php.ini location  
> You should add "extension=redis.so" to php.ini


8.
php.ini加完擴展後會報錯
> PHP Warning:  PHP Startup: Unable to load dynamic library 'redis.so' (tried: /opt/rh/rh-php73/root/usr/lib64/php/modules/redis.so (/opt/rh/rh-php73/root/usr/lib64/php/modules/redis.so: undefined symbol: php_json_decode_ex), /opt/rh/rh-php73/root/usr/lib64/php/modules/redis.so.so (/opt/rh/rh-php73/root/usr/lib64/php/modules/redis.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0

這是因為和json擴展加載順序造成衝突，將php.ini的`extension=redis.so`拿掉  
改在php.d文件夹下建立新文件 __redis.ini__
```
; Enable redis extension module
extension=redis.so
```


Check Install
========
確認 php.d位置 `php -i|grep php.d`  
確認擴展已經加載 `php -m`



-----------
參考: https://blog.csdn.net/lengyue1084/article/details/109670427
