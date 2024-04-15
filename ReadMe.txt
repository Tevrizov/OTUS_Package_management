Домашнее задание
1) Создать свой DEB пакет (можно взять свое приложение, либо собрать, например,
апач с определенными опциями)
2) Создать свой репозиторий и разместить там ранее собранный DEB



#############################################################################################################################
1) Создание своего DEB пакета

1.Исрользую тестовую ВМ на базе Ubuntu 20.04.6 LTS



2.Вначале устанавливаю пакеты которые мне понадобятся для создания deb пакетов
     apt install dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev unzip cmake 




3.Включаю доступность исходников
     nano /etc/apt/sources.list
     раскомментируем строки начинающиеся  с deb-src
     sudo apt-get update




4.Создаю директорию для сборки и получаю исходники
    apt source nginx
        root@tep:~/custom-nginx# ll
        total 1952
        drwxr-xr-x  3 root root    4096 апр 14 15:43 ./
        drwx------  9 root root    4096 апр 14 15:40 ../
        drwxr-xr-x 10 root root    4096 апр 14 15:43 nginx-1.18.0/
        -rw-r--r--  1 root root  934300 ноя 15  2022 nginx_1.18.0-0ubuntu1.4.debian.tar.xz
        -rw-r--r--  1 root root    4273 ноя 15  2022 nginx_1.18.0-0ubuntu1.4.dsc
        -rw-r--r--  1 root root 1039530 апр 28  2020 nginx_1.18.0.orig.tar.gz
        root@tep:~/custom-nginx# ^C



5.Перехожу в директорю с модулями
    cd nginx-1.18.0/debian/modules/




6.В эту директорию скачиваем дополнительный модуль
    git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli



7. Согласно инструкции делаем:
    cd ngx_brotli/deps/brotli/
    mkdir out && cd out



8. Согласно инструкции на сайте https://github.com/google/ngx_brotli запускаем

cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast \
 -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections \
 -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native \ 
 -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..

 проверяем
 root@tep:~/custom-nginx/nginx-1.18.0/debian/modules/ngx_brotli/deps/brotli/out# echo $?
 0



9. собираю 
root@tep:~/custom-nginx/nginx-1.18.0/debian/modules/ngx_brotli/deps/brotli/out# cmake --build . --config Release --target brotlienc
Scanning dependencies of target brotlicommon
[  3%] Building C object CMakeFiles/brotlicommon.dir/c/common/constants.c.o
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/context.c.o
[ 10%] Building C object CMakeFiles/brotlicommon.dir/c/common/dictionary.c.o
[ 13%] Building C object CMakeFiles/brotlicommon.dir/c/common/platform.c.o
[ 17%] Building C object CMakeFiles/brotlicommon.dir/c/common/shared_dictionary.c.o
[ 20%] Building C object CMakeFiles/brotlicommon.dir/c/common/transform.c.o
[ 24%] Linking C static library libbrotlicommon.a
[ 24%] Built target brotlicommon
Scanning dependencies of target brotlienc
[ 27%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references.c.o
[ 31%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references_hq.c.o
[ 34%] Building C object CMakeFiles/brotlienc.dir/c/enc/bit_cost.c.o
[ 37%] Building C object CMakeFiles/brotlienc.dir/c/enc/block_splitter.c.o
[ 41%] Building C object CMakeFiles/brotlienc.dir/c/enc/brotli_bit_stream.c.o
[ 44%] Building C object CMakeFiles/brotlienc.dir/c/enc/cluster.c.o
[ 48%] Building C object CMakeFiles/brotlienc.dir/c/enc/command.c.o
[ 51%] Building C object CMakeFiles/brotlienc.dir/c/enc/compound_dictionary.c.o
[ 55%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment.c.o
[ 58%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment_two_pass.c.o
[ 62%] Building C object CMakeFiles/brotlienc.dir/c/enc/dictionary_hash.c.o
[ 65%] Building C object CMakeFiles/brotlienc.dir/c/enc/encode.c.o
[ 68%] Building C object CMakeFiles/brotlienc.dir/c/enc/encoder_dict.c.o
[ 72%] Building C object CMakeFiles/brotlienc.dir/c/enc/entropy_encode.c.o
[ 75%] Building C object CMakeFiles/brotlienc.dir/c/enc/fast_log.c.o
[ 79%] Building C object CMakeFiles/brotlienc.dir/c/enc/histogram.c.o
[ 82%] Building C object CMakeFiles/brotlienc.dir/c/enc/literal_cost.c.o
[ 86%] Building C object CMakeFiles/brotlienc.dir/c/enc/memory.c.o
[ 89%] Building C object CMakeFiles/brotlienc.dir/c/enc/metablock.c.o
[ 93%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict.c.o
[ 96%] Building C object CMakeFiles/brotlienc.dir/c/enc/utf8_util.c.o
[100%] Linking C static library libbrotlienc.a
[100%] Built target brotlienc
root@tep:~/custom-nginx/nginx-1.18.0/debian/modules/ngx_brotli/deps/brotli/out# 

root@tep:~/custom-nginx/nginx-1.18.0/debian/modules/ngx_brotli/deps/brotli/out# echo $?
0

10. Возвращаюсь в директорию с модулями
    cd ../../../..



11. Добавляю  --add-module=$(MODULESDIR)/ngx_brotli  в файл /root/custom-nginx/nginx-1.18.0/debian/rules



12. Редактирую файл /root/custom-nginx/nginx-1.18.0/debian/changelog изменя версию на nginx (1.18.0-0ubuntu1.4-custum) 



13. root@tep:~/custom-nginx/nginx-1.18.0# dpkg-buildpackage  -b


dpkg-checkbuilddeps: error: Unmet build dependencies: debhelper (>= 10) po-debconf libexpat-dev libgd-dev libgeoip-dev libhiredis-dev libluajit-5.1-dev libmaxminddb-dev libmhash-dev libpam0g-dev libperl-dev libssl-dev libxslt1-dev quilt

устанавливаю недостающие библиотеки

получаю следующую ошибку
dpkg-checkbuilddeps: error: Unmet build dependencies: debhelper (>= 10)

sudo apt-get install debhelper

после утановки недостающих библиотек команда dpkg-buildpackage  -b запустилась

.......
dpkg-deb --build debian/libnginx-mod-http-lua ..
dpkg-deb: building package 'libnginx-mod-http-lua' in '../libnginx-mod-http-lua_1.18.0-0ubuntu1.4-custum_amd64.deb'.
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../nginx_1.18.0-0ubuntu1.4-custum_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)
root@tep:~/custom-nginx/nginx-1.18.0# echo $?
0
root@tep:~/custom-nginx/nginx-1.18.0# 


14. Проверяем список пакетов
root@tep:~/custom-nginx# ll | grep cus
-rw-r--r--  1 root root   38200 апр 14 16:55 libnginx-mod-http-auth-pam_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   46028 апр 14 16:55 libnginx-mod-http-auth-pam-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   40620 апр 14 16:55 libnginx-mod-http-cache-purge_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   50824 апр 14 16:55 libnginx-mod-http-cache-purge-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   40092 апр 14 16:55 libnginx-mod-http-dav-ext_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   50268 апр 14 16:55 libnginx-mod-http-dav-ext-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   49776 апр 14 16:55 libnginx-mod-http-echo_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  289616 апр 14 16:55 libnginx-mod-http-echo-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   43428 апр 14 16:55 libnginx-mod-http-fancyindex_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   56268 апр 14 16:55 libnginx-mod-http-fancyindex-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   39472 апр 14 16:55 libnginx-mod-http-geoip_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   39976 апр 14 16:55 libnginx-mod-http-geoip2_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   49312 апр 14 16:55 libnginx-mod-http-geoip2-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   48612 апр 14 16:55 libnginx-mod-http-geoip-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   43628 апр 14 16:55 libnginx-mod-http-headers-more-filter_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  117484 апр 14 16:55 libnginx-mod-http-headers-more-filter-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   43040 апр 14 16:55 libnginx-mod-http-image-filter_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   55484 апр 14 16:55 libnginx-mod-http-image-filter-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root  183512 апр 14 16:55 libnginx-mod-http-lua_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 1581648 апр 14 16:55 libnginx-mod-http-lua-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   38672 апр 14 16:55 libnginx-mod-http-ndk_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   48548 апр 14 16:55 libnginx-mod-http-ndk-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   51676 апр 14 16:55 libnginx-mod-http-perl_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  146904 апр 14 16:55 libnginx-mod-http-perl-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   41284 апр 14 16:55 libnginx-mod-http-subs-filter_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   50240 апр 14 16:55 libnginx-mod-http-subs-filter-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   45048 апр 14 16:55 libnginx-mod-http-uploadprogress_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   58908 апр 14 16:55 libnginx-mod-http-uploadprogress-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   41440 апр 14 16:55 libnginx-mod-http-upstream-fair_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   52232 апр 14 16:55 libnginx-mod-http-upstream-fair-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   41216 апр 14 16:55 libnginx-mod-http-xslt-filter_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root   63900 апр 14 16:55 libnginx-mod-http-xslt-filter-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   71268 апр 14 16:55 libnginx-mod-mail_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  176908 апр 14 16:55 libnginx-mod-mail-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root  181768 апр 14 16:55 libnginx-mod-nchan_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 1309520 апр 14 16:55 libnginx-mod-nchan-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root  153020 апр 14 16:55 libnginx-mod-rtmp_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  615352 апр 14 16:55 libnginx-mod-rtmp-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   95768 апр 14 16:55 libnginx-mod-stream_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root  325696 апр 14 16:55 libnginx-mod-stream-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   33824 апр 14 16:54 nginx_1.18.0-0ubuntu1.4-custum_all.deb
-rw-r--r--  1 root root   26165 апр 14 16:55 nginx_1.18.0-0ubuntu1.4-custum_amd64.buildinfo
-rw-r--r--  1 root root   21702 апр 14 16:55 nginx_1.18.0-0ubuntu1.4-custum_amd64.changes
-rw-r--r--  1 root root   66044 апр 14 16:55 nginx-common_1.18.0-0ubuntu1.4-custum_all.deb
-rw-r--r--  1 root root  844380 апр 14 16:55 nginx-core_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 2809312 апр 14 16:55 nginx-core-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root   45260 апр 14 16:54 nginx-doc_1.18.0-0ubuntu1.4-custum_all.deb
-rw-r--r--  1 root root  868516 апр 14 16:55 nginx-extras_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 2918244 апр 14 16:55 nginx-extras-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root  847132 апр 14 16:55 nginx-full_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 2806976 апр 14 16:55 nginx-full-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
-rw-r--r--  1 root root  824812 апр 14 16:55 nginx-light_1.18.0-0ubuntu1.4-custum_amd64.deb
-rw-r--r--  1 root root 2538648 апр 14 16:55 nginx-light-dbgsym_1.18.0-0ubuntu1.4-custum_amd64.ddeb
root@tep:~/custom-nginx# 


15. если нужно их утстановить то
    dpkg -i *.deb


16. Что бы пакет не изменялся фиксируем его командой
    root@tep:~/custom-nginx# apt-mark hold nginx
    nginx помечен как зафиксированный.

###########################################################################################################################################

2) Создать свой репозиторий и разместить там ранее собранный DEB

1. Установить пакет dpkg-dev, если  ещё не установлен.
    sudo apt-get update
    sudo apt-get install -y dpkg-dev

2. Сложить нужные deb пакеты (из которых Вы хотите сделать Ваш локальный репозиторий) в некую папку.
    mkdir repo
    cp *.deb repo
    cd repo

3.root@tep:~/custom-nginx/repo# dpkg-scanpackages . /dev/null | gzip -9c > Packages
    dpkg-scanpackages: warning: Packages in archive but missing from override file:
    dpkg-scanpackages: warning:   libnginx-mod-http-auth-pam libnginx-mod-http-cache-purge libnginx-mod-http-dav-ext libnginx-mod-http-echo libnginx-mod-http-fancyindex libnginx-mod-http-geoip libnginx-mod-http-geoip2 libnginx-mod-http-headers-more-filter libnginx-mod-http-image-filter libnginx-mod-http-lua libnginx-mod-http-ndk libnginx-mod-http-perl libnginx-mod-http-subs-filter libnginx-mod-http-uploadprogress libnginx-mod-http-upstream-fair libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-nchan libnginx-mod-rtmp libnginx-mod-stream nginx nginx-common nginx-core nginx-doc nginx-extras nginx-full nginx-light
    dpkg-scanpackages: info: Wrote 27 entries to output Packages file.

    Получился файл Packages, это список пакетов репозитория!

4. Добавляю в файл sources.list ссылку на папку с пакетами, если она расположена локально, то она должна выглядеть примерно так:
    deb file:ПОЛНЫЙ_ПУТЬ_К_ВАШЕЙ ПАПКЕ ./

    nano /etc/apt/sources.list
    deb [allow-insecure=yes] file:/root/custom-nginx/repo  ./


    Если  репозиторий находится на другой машине в локальной сети, то нужно открыть доступ в папке с пакетами по http или compress_fragment_two_pass
    
5. apt-update

#######################################################################################################################################################
