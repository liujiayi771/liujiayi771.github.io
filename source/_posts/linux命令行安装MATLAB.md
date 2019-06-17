---
title: linux命令行安装MATLAB
date: 2019-01-18 09:51:19
tags:
---

登录MATLAB的MathWorks，选择右上角的我的账户，点击许可证右侧的许可证管理，进入后选择安装和激活，之后选择“激活并获取许可证文件”，选择激活计算机，填入相关信息。其中主机ID即为电脑用于上外网的网卡的MAC地址，可以使用`ipconfig`进行查看，计算机登录名就是当前登录用户的用户名，不知道的话可以使用`whoami`指令进行查看。填写完成后会生成一个序列号，和一个lisence.lic文件，将文件上传至安装MATLAB的服务器上并记录激活序列号。

下载包含所有产品的完整MATLAB安装包，官网下载的只是一个100MB多的一个网络安装器，安装过程中才会下载需要的产品，这对于命令行安装是不适用的，命令行安装必须使用完整的离线安装包。之后需要填写silent安装模式下的安装配置文件，具体模板如下：
<!--more-->
```
##################################################################
##
## Use this file to specify parameters required by the installer at runtime.
##
## Instructions for using this file.
##
## 1. Create a copy of this template file and fill in the required
##    information.
##
## 2. Uncomment only those lines that start with a single '#'
##    and set the desired values. All allowed values for the
##    parameters are defined in the comments section for each
##    parameter.
##
## 3. Launch the installer from the command line, using the -inputFile option
##    to specify the name of your installer initialization file.
##
##    (Windows) setup.exe -inputFile <file_name>
##    (Mac/Unix) install -inputFile <file_name>
##
## NOTE:
##    If you want to run the activation application in silent
##    mode immediately after the installer completes, you must create
##    an activation initialization file and specify its name as the
##    value of the activationPropertiesFile= option. You can also
##    pass the name of the activation initialization file to the
##    installer using the -activationPropertiesFile command line
##    option.
##
##################################################################
##
##
## SPECIFY INSTALLATION FOLDER
##
## Example:
##        (Windows) destinationFolder=C:\Program Files\MATLAB\RXXXX
##        (Unix) destinationFolder=/usr/local/RXXXX
##        (Mac) destinationFolder=/Applications
##
## Set the desired value for destinationFolder and
## uncomment the line.

# destinationFolder=

##
## SPECIFY FILE INSTALLATION KEY
##
## Example: fileInstallationKey=xxxxx-xxxxx-xxxxx-xxxxx.....
##
## Set the desired value for fileInstallationKey and
## uncomment the line.
##
# fileInstallationKey=

##
## ACCEPT LICENSE AGREEMENT
##
## You must agree to the license agreement to install MathWorks products.
## The license agreement can be found in the license_agreement.txt file at the
## root level of the installation DVD.
##
## Example: agreeToLicense=yes
##
## Set agreeToLicense value to yes or no and
## uncomment the line.

# agreeToLicense=

##
## SPECIFY OUTPUT LOG
##
## Specify full path of file into which you want the results of the
## installation to be recorded.
##
## Example:
##            (Windows) outputFile=C:\TEMP\mathworks_<user_name>.log
##            (Unix/Mac) outputFile=/tmp/mathworks_<user_name>.log
##
## Set the desired value for outputFile and
## uncomment the line.

# outputFile=

## SPECIFY INSTALLER MODE
##
## interactive: Run the installer GUI, waiting for user input on all
##              dialog boxes.
##
## silent:      Run the installer without displaying the GUI.
##
## automated:   Run the installer GUI, displaying all dialog boxes, but only
##              waiting for user input on dialogs that are missing required
##              input.
##
## Set mode value to either interactive, silent, or automated and
## uncomment the line.

# mode=

## SPECIFY LENGTH OF TIME DIALOG BOXES ARE DISPLAYED
##
## Specify how long the installer dialog boxes are displayed, in milliseconds.
##
## NOTE: Use this value only if you set the installer mode to automated.
##
## By default, the dialog boxes display on the screen for one second.
##
## Example: (To specify a value of 1 second.) automatedModeTimeout=1000
##
## Set the desired value for automatedModeTimeout and
## uncomment the line.

# automatedModeTimeout=

## SPECIFY ACTIVATION PROPERTIES FILE (For non-network license types only)
##
## Enter the path to an existing file that contains properties to configure
## the activation process.

# activationPropertiesFile=

########## Begin: Options for Network License Types #########
##
## SPECIFY PATH TO LICENSE FILE (Required for network license types only)
##
## This value is required when installing either the License Manager or when
## installing as a Network End-User
## Example:
##            (Windows) licensePath=C:\TEMP\license.dat
##            (Unix) licensePath=/tmp/license.dat
## Set the desired value for licensePath and
## uncomment the line.

# licensePath=

## CHOOSE TO INSTALL LICENSE MANAGER (For network license types only)
##
## Installs license manager files to disk.
##
## NOTE: You only need to install the license manager files
## on your license server.
##
## Set lmgrFiles value to true or false and
## uncomment the line.

# lmgrFiles=

## INSTALL LICENSE MANAGER AS A SERVICE (For network license types only)
##
## Configure the license manager as a service on Windows.
##
## NOTE: Not applicable for Unix or Mac.
##
## NOTE: The lmgr_files option (set in previous step) must also be set to true.
##
## Set lmgrService value to true or false and
## uncomment the line.

# lmgrService=

########## End: Options for Network License Types #########



################# Begin - Windows Only Options ################
##
## CHOOSE TO SET FILE ASSOCIATIONS
##
## Set to true if you want the installer to associate file types used by MathWorks
## products to this version of MATLAB, or false if you do not want the installer to
## associate MathWorks file types with this version of MATLAB.
##
## Default value is true.
##
## Set setFileAssoc value to true or false and
## uncomment the line.

# setFileAssoc=

##
## CHOOSE TO CREATE WINDOWS DESKTOP SHORTCUT
##
## Set to true if you would like the installer to create a desktop shortcut icon
## when MATLAB is installed or false if you don't want the shortcut created.
##
## Set desktopShortcut value to true or false and
## uncomment the line.

# desktopShortcut=

## CHOOSE TO ADD SHORTCUT TO WINDOWS START MENU
##
## Set to true if you would like the installer to create a Start Menu shortcut
## icon when MATLAB is installed or false if you don't want the shortcut created.
##
## Set startMenuShortcut value to true or false and
## uncomment the line.

# startMenuShortcut=

## CREATE a MATLAB Startup Accelerator task
##
## The MATLAB Startup Accelerator installer creates a
## system task to preload MATLAB into the system's cache
## for faster startup.
##
## NOTE: By default, a MATLAB Startup Accelerator task will
## automatically be created.
##
## If you want a MATLAB Startup Accelerator task to be created,
## do not edit this section.
##
## Set createAccelTask value to false if you do not want to
## create an Accelerator task and uncomment the line.

# createAccelTask=

## Enable Login Named User  licensing
##
## Set to Yes to enable use of a Login Named User license for all users of this MATLAB installation
## Users must log in to their MathWorks Account when MATLAB starts.
##
## Example: enableLNU=yes
##
## NOTE: This flag is valid in silent installations only.

# enableLNU=yes



################ End - Windows Only Options ################


## SPECIFY PRODUCTS YOU WANT TO INSTALL
##
## By default, the installer installs all the products and
## documentation for which you are licensed. Products you are not licensed for
## are not installed, even if they are listed here.
##
## Note:
## 1. To automatically install all your licensed products, do not edit
##    any lines in this section.
##
## 2. To install a specific product or a subset of products for
##    which you are licensed, uncomment the line for the product(s) you want
##    to install.

#product.5G_Toolbox
#product.Aerospace_Blockset
#product.Aerospace_Toolbox
#product.Antenna_Toolbox
#product.Audio_System_Toolbox
#product.Automated_Driving_System_Toolbox
#product.Bioinformatics_Toolbox
#product.Communications_Toolbox
#product.Computer_Vision_System_Toolbox
#product.Control_System_Toolbox
#product.Curve_Fitting_Toolbox
#product.DO_Qualification_Kit
#product.DSP_System_Toolbox
#product.Data_Acquisition_Toolbox
#product.Database_Toolbox
#product.Datafeed_Toolbox
#product.Deep_Learning_Toolbox
#product.Econometrics_Toolbox
#product.Embedded_Coder
#product.Filter_Design_HDL_Coder
#product.Financial_Instruments_Toolbox
#product.Financial_Toolbox
#product.Fixed_Point_Designer
#product.Fuzzy_Logic_Toolbox
#product.GPU_Coder
#product.Global_Optimization_Toolbox
#product.HDL_Coder
#product.HDL_Verifier
#product.IEC_Certification_Kit
#product.Image_Acquisition_Toolbox
#product.Image_Processing_Toolbox
#product.Instrument_Control_Toolbox
#product.LTE_HDL_Toolbox
#product.LTE_Toolbox
#product.MATLAB
#product.MATLAB_Coder
#product.MATLAB_Compiler
#product.MATLAB_Compiler_SDK
#product.MATLAB_Distributed_Computing_Server
#product.MATLAB_Production_Server
#product.MATLAB_Report_Generator
#product.Mapping_Toolbox
#product.Model_Predictive_Control_Toolbox
#product.Model_Based_Calibration_Toolbox
#product.OPC_Toolbox
#product.Optimization_Toolbox
#product.Parallel_Computing_Toolbox
#product.Partial_Differential_Equation_Toolbox
#product.Phased_Array_System_Toolbox
#product.Polyspace_Bug_Finder
#product.Polyspace_Code_Prover
#product.Powertrain_Blockset
#product.Predictive_Maintenance_Toolbox
#product.RF_Blockset
#product.RF_Toolbox
#product.Risk_Management_Toolbox
#product.Robotics_System_Toolbox
#product.Robust_Control_Toolbox
#product.Sensor_Fusion_and_Tracking_Toolbox
#product.Signal_Processing_Toolbox
#product.SimBiology
#product.SimEvents
#product.Simscape
#product.Simscape_Driveline
#product.Simscape_Electrical
#product.Simscape_Fluids
#product.Simscape_Multibody
#product.Simulink
#product.Simulink_3D_Animation
#product.Simulink_Check
#product.Simulink_Code_Inspector
#product.Simulink_Coder
#product.Simulink_Control_Design
#product.Simulink_Coverage
#product.Simulink_Design_Optimization
#product.Simulink_Design_Verifier
#product.Simulink_Desktop_Real_Time
#product.Simulink_PLC_Coder
#product.Simulink_Real_Time
#product.Simulink_Report_Generator
#product.Simulink_Requirements
#product.Simulink_Test
#product.Spreadsheet_Link
#product.Stateflow
#product.Statistics_and_Machine_Learning_Toolbox
#product.Symbolic_Math_Toolbox
#product.System_Identification_Toolbox
#product.Text_Analytics_Toolbox
#product.Trading_Toolbox
#product.Vehicle_Dynamics_Blockset
#product.Vehicle_Network_Toolbox
#product.Vision_HDL_Toolbox
#product.WLAN_Toolbox
#product.Wavelet_Toolbox
```
我们需要填写的内容主要有：

```
destinationFolder=/usr/local/RXXXX
fileInstallationKey=xxxxx-xxxxx-xxxxx-xxxxx.....
agreeToLicense=yes
mode=silent
licensePath=/tmp/license.lic
```
该配置文件需要指定安装路径（不指定的话安装路径默认为/usr/local/MATLAB），同意协议，安装模式为silent，激活序列号，证书文件路径，最后可以指定需要安装的产品，按照模板中的内容将对应产品前的注释取消即可，若不知道需要哪些产品，不添加任何产品则会安装所有的产品。之后解压离线安装包，这里需要注意的是，离线安装包解压后里面很多可执行文件的权限会没有执行权限，直接安装的话会很快结束，无法正常安装，由于不方便将所有可执行文件都找出来添加执行权限，这里需要暴力使用`chmod -R 777 MATLAB_dir`将安装包解压文件夹下的所有文件的权限都变为777，这样才能正确安装，之后使用如下指令进行安装（installer_input.txt为之前写的配置文件的路径）：

```
./install.sh -inputFile installer_input.txt
```
安装完成后，还需要将license.lic文件移动到安装目录下的license目录当中完成激活。
之后可能会遇到libXt.so.6找不到的问题，该库文件是用于支持x11的，按道理说以命令行方式运行是不需要x11的，但是还是会报错，解决方法便是安装缺失的这个库

```
sudo apt install libxt6
```
