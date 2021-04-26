# ti-ubuntu12.04 交叉编译环境

#===================================================

# sysroot
/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr

# 安装目录
cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..

# 查找位置
cmake -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..
#===================================================
#===================================================

#0. cmake-3.9.6
#===================================================
#1. aws-c-common
### CMakeLists.txt 
# set(CMAKE_C_COMPILER "arm-linux-gnueabihf-gcc")
# set(CMAKE_CXX_COMPILER "arm-linux-gnueabihf-g++")
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu99")

### include/aws/common/atomics_gnu.inl
//#    pragma GCC diagnostic ignored "-Wpedantic"

cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr ..

#===================================================
#2. openssl-1.1.1c
./config no-asm --sysroot="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi" --prefix="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local"

#===================================================
#3. aws-encryption-sdk-c
cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..

#===================================================
#4. libsodium-1.0.18
sudo apt-get install autoconf libtool
./configure --with-sysroot="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi" --host=arm-linux --prefix="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local"

#===================================================
#5. mbedtls


cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..

#===================================================
#6. tinycbor
make CC="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/i686-arago-linux/usr/bin/arm-linux-gnueabihf-gcc --sysroot=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi" DESTDIR=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi

#===================================================
# libmodbus
sudo apt-get install autoconf automake libtool
./autogen.sh
./configure --with-sysroot="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi" --host=arm-linux --prefix="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local"

make CC="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/i686-arago-linux/usr/bin/arm-linux-gnueabihf-gcc --sysroot=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi" DESTDIR=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi
#===================================================

#7. lws-iot-sdk
cmake -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..




--ti nim lang 开发环境设置
make DESTDIR=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/i686-arago-linux


#8.
删除CMakelists.txt中的Werror
cmake -DUA_ARCHITECTURE=posix -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..

cmake -DUA_ARCHITECTURE=posix -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local -DCMAKE_PREFIX_PATH:PATH=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local ..

#9.
设置Makefile的INSTALL_TOP为 /home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi/usr/local

make posix CC="/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/i686-arago-linux/usr/bin/arm-linux-gnueabihf-gcc --sysroot=/home/ti/ti-sdk-am335x-evm-06.00.00.00/linux-devkit/sysroots/armv7ahf-vfp-neon-3.2-oe-linux-gnueabi"