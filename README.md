# xcodebuild.sh
use xcodebuild.sh to auto archive ipa and upload to PGY or Appstore

.sh detail:
 
#脚本所在的目录必须和WorkSpace或者说工程主目录所在的目录在同一个目录层级中

#上传蒲公英
uploadPGY(){

    echo "~~~~~~~~~~~~选择完成是否上传蒲公英(输入序号)~~~~~~~~~~~~~~~"
    echo "  1 上传"
    echo "  2 不上传"

    read uploadPara
    sleep 0.5
    upload="$uploadPara"

    # 判读用户是否有输入
    if [ -n "$upload" ]
    then
    if [ "$upload" = "1" ]
    then

    echo "~~~~~~~~~~~~即将进行上传蒲公英~~~~~~~~~~~~~~~"

    echo "****** 开始上传IPA包到蒲公英 ******"

    filePath=$1/$2.ipa
    echo "~~~~~~~~~~~~filePath为 $filePath~~~~~~~~~~~~"
    U_key="your u_key"
    APP_KEY="your app_key"
    if [ -e "${filePath}" ]; then
    echo "进入上传"
    curl -F "file=@${filePath}" \
    -F "uKey=${U_key}" \
    -F "_api_key=${APP_KEY}" \
    "http://www.pgyer.com/apiv1/app/upload"
    echo "****** IPA包上传到蒲公英成功 ******"
    else
    echo "IPA包不存在 上传蒲公英失败"
    fi
    
    elif [ "$upload" = "2" ]
    then
    echo "~~~~~~~~~~~~打包结束~~~~~~~~~~~~~~~"
    fi
    fi
}


#配置参数

Workspace_Name="***.xcworkspace"

#工程名字
Project_Name="***"

#配置打包方式Release或者Debug
Configuration="Release"

#在终端中提示 根据输入的序号不同，打包成不同版本的ipa
echo "~~~~~~~~~~~~选择打包环境(输入序号)~~~~~~~~~~~~~~~"
echo "  1 Debug"
echo "  2 Release"

# 读取用户在终端中输入并存到变量里
read parameter_dev
sleep 0.5
method_dev="$parameter_dev"

# 判读用户是否有输入
if [ -n "$method_dev" ]
then
if [ "$method_dev" = "1" ]
then
Configuration="Debug"
echo "~~~~~~~~~~~~即将进行Debug环境打包~~~~~~~~~~~~~~~"

elif [ "$method_dev" = "2" ]
then
Configuration="Release"

echo "~~~~~~~~~~~~即将进行Release环境打包~~~~~~~~~~~~~~~"
fi
fi

#基础主路径
BUILD_PATH=./build

#不同版本的基础子路径
#dev
DEV_PATH=${BUILD_PATH}/dev
#adHoc
ADHOC_PATH=${BUILD_PATH}/adHoc
#appStore
APPSTORE_PATH=${BUILD_PATH}/appStore
#enterprise
ENTERPRISE_PATH=${BUILD_PATH}/enterprise


#配置打包结果输出的路径
#dev版本
DevProjectOutPath=${DEV_PATH}/devOut
#AdHoc版本
AdHocProjectOutPath=${ADHOC_PATH}/adHocOut
#AppStore版本
AppStoreProjectOutPath=${APPSTORE_PATH}/appStoreOut
#企业版本
EnterpriseProjectOutPath=${ENTERPRISE_PATH}/enterpriseOut


#加载各个版本的plist文件
DEVExportOptionsPlist="./ExportOptions/ExportOptions-dev.plist"
ADHOCExportOptionsPlist="./ExportOptions/ExportOptions-adhoc.plist"
AppStoreExportOptionsPlist="./ExportOptions/ExportOptions-appStore.plist"
EnterpriseExportOptionsPlist="./ExportOptions/ExportOptions-enterprise.plist"


#在终端中提示 根据输入的序号不同，打包成不同版本的ipa
echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "  1 dev"
echo "  2 adHoc"
echo "  3 AppStore"
echo "  4 Enterprise"

# 读取用户在终端中输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]
then
if [ "$method" = "1" ]
then
#这里都执行命令中是在xcworkspace工程中执行的，如果工程不是xcworkspace，可以把-workspace的内容删掉，加入了证书和描述文件，如果不需要请删除
#如果用户选择的是1，就执行dev脚本
#首先清除原来的文件夹
rm -rf ${BUILD_PATH}
#创建文件夹，路径需要一层一层创建，不然会创建失败
mkdir ${BUILD_PATH}
mkdir ${DEV_PATH}
#打包输出的文件
mkdir ${DevProjectOutPath}
#编译
xcodebuild archive -workspace $Workspace_Name -scheme $Project_Name -configuration $Configuration -archivePath ${DEV_PATH}/$Project_Name-dev.xcarchive -allowProvisioningUpdates
#打包
xcodebuild  -exportArchive -archivePath ${DEV_PATH}/$Project_Name-dev.xcarchive -exportOptionsPlist $DEVExportOptionsPlist -exportPath ${DevProjectOutPath}

#上传操作
uploadPGY ${DevProjectOutPath} ${Project_Name}


elif [ "$method" = "2" ]
then
#这里都执行命令中是在xcworkspace工程中执行的，如果工程不是xcworkspace，可以把-workspace的内容删掉，加入了证书和描述文件，如果不需要请删除
#如果用户选择的是2，就执行adhoc脚本
#首先清除原来的文件夹
rm -rf ${BUILD_PATH}
#创建文件夹，路径需要一层一层创建，不然会创建失败
mkdir ${BUILD_PATH}
mkdir ${ADHOC_PATH}

#打包输出的文件
mkdir ${AdHocProjectOutPath}

#编译
xcodebuild archive -workspace $Workspace_Name -scheme $Project_Name -configuration $Configuration -archivePath ${ADHOC_PATH}/$Project_Name-adhoc.xcarchive -allowProvisioningUpdates

xcodebuild  -exportArchive -archivePath ${ADHOC_PATH}/$Project_Name-adhoc.xcarchive -exportOptionsPlist $ADHOCExportOptionsPlist -exportPath ${AdHocProjectOutPath}

#上传操作
uploadPGY ${AdHocProjectOutPath} ${Project_Name}

elif [ "$method" = "3" ]
then
#如果用户选择的是3，就执行appstore脚本
#首先清除原来的文件夹
rm -rf ${BUILD_PATH}
#创建文件夹，路径需要一层一层创建，不然会创建失败
mkdir ${BUILD_PATH}
mkdir ${APPSTORE_PATH}

#打包输出的文件
mkdir ${AppStoreProjectOutPath}

xcodebuild archive -workspace $Workspace_Name -scheme $Project_Name -configuration $Configuration -archivePath ${APPSTORE_PATH}/$Project_Name-appstore.xcarchive -allowProvisioningUpdates

xcodebuild  -exportArchive -archivePath ${APPSTORE_PATH}/$Project_Name-appstore.xcarchive -exportOptionsPlist $AppStoreExportOptionsPlist -exportPath ${AppStoreProjectOutPath}

#验证ipa是否打包成功
if [ -e $AppStoreProjectOutPath/$Project_Name.ipa ]; then
echo '----ipa包已生成----'
open $AppStoreProjectOutPath
echo '----打包ipa完成----'
echo '**---------------**'
echo '****开始发布ipa包****'
echo '**---------------**'
#验证后上传到App Store
# 将-u 后面的XXX替换成自己的AppleID的账号，-p后面的XXX替换成自己的密码
altoolPath="/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool"
"$altoolPath" --validate-app -f ${exportIpaPath}/${scheme_name}.ipa -u XXX -p XXX -t ios --output-format xml
"$altoolPath" --upload-app -f ${exportIpaPath}/${scheme_name}.ipa -u  XXX -p XXX -t ios --output-format xml
else
echo '----ipa包导出失败----'
fi


elif [ "$method" = "4" ]
then
#如果用户选择的是4，就执行企业脚本
#首先清除原来的文件夹
rm -rf ${BUILD_PATH}
#创建文件夹，路径需要一层一层创建，不然会创建失败
mkdir ${BUILD_PATH}
mkdir ${ENTERPRISE_PATH}

#打包输出的文件
mkdir ${EnterpriseProjectOutPath}

xcodebuild archive -workspace $Workspace_Name -scheme $Project_Name -configuration $Configuration -archivePath ${ENTERPRISE_PATH}/$Project_Name-enterprise.xcarchive -allowProvisioningUpdates

xcodebuild  -exportArchive -archivePath ${ENTERPRISE_PATH}/$Project_Name-enterprise.xcarchive -exportOptionsPlist $EnterpriseExportOptionsPlist -exportPath ${EnterpriseProjectOutPath}

#上传操作
uploadPGY ${EnterpriseProjectOutPath} ${Project_Name}

else
#如果是其他输入，则在终端中提示参数无效并退出
echo "参数无效...."
exit 1
fi
fi


