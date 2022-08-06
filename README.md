# fish_shooter

# 同级目录依赖:

boost
yylib

# 客户端 engine: 
axis

# 项目创建命令行:
axis new -p game.fish_shooter.cpp -l cpp fish_shooter_cpp
axis new -p game.fish_shooter.lua -l lua fish_shooter_lua

# cmake 修改细则

cmake_minimum_required(VERSION 3.20)

set(CMAKE_CXX_STANDARD 20)
include_directories(${AXIS_ROOT_PATH}/../boost/boost)
include_directories(${AXIS_ROOT_PATH}/../yylib/src)

list(APPEND GAME_SOURCE
	 ${RUNTIME_SRC_ROOT}/Classes/.......
	 ${RUNTIME_SRC_ROOT}/Classes/.......
	 ${AXIS_ROOT_PATH}/../xxxxx/......
 )

# 编译方式

windows 利用 cmake 高版本 产生项目文件
android 首先升级 AndroidStudio 以及各种 toolchains, sdk, ndk, cmake 到最高版本, 并记录版本号

当前部分版本号: 
sdk 33
ndk 25.0.8775105
cmake 3.22.1

进入到 向导项目 proj.android 目录
	打开 gradle.properties
	根据记录的版本信息, 先修改一波版本号.  例如    33  19  33
	
	打开 build.gradle
	将 7.2.0 改为 7.2.2  ( 也有可能是 7.4.2 ) ( 不在这一步修改应该也行，通过 android studio 弹出的 升级 AGP 提示一步步来 )
	
进入到 向导项目 proj.android/app 目录
	打开 build.gradle
	删除前 3 行 import
	
	打开 AndroidManifest.xml
	在 <activity 里加一行  android:exported="true"
	
进入到 engine 目录 axis\core\platform\android\libcocos2dx
	打开 axistools.gradle
	找出 NDK CMAKE 版本字样, 改为最新 (  findNDK & findCMake 函数  )

# 已知问题

当前 axis 的项目 死活会要求 安装 30.0.2 build tools 并且找不到地方修改

注意 ignore 自动创建的  SharedLoader.java  和  run.bat

	
