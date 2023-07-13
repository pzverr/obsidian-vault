##### Установка зависимостей
```sh
sudo apt-get install -y build-essential gdb lcov pkg-config libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev lzma lzma-dev tk-dev uuid-dev zlib1g-dev
```

##### Качаем
[XZ compressed source tarball](https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tar.xz)
Например вот так:
```sh
wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tar.xz
```

##### Распаковываем
```sh
tar xvf python3.10.tar.xz
```

##### Конфигурим
```sh
./configure --enable-optimizations
```
по-умолчанию префикc /usr/local

##### Мейкаем
```sh
sudo make -j4 altinstall
```
*-j4 использовать 4 ядра*

