#
# ruby脚本配置说明
# 1.在workspace文件的root目录下，指定好具体的module工程和framework工程的目录，打开workspace文件，创建对应的module和framework
# 2.先执行framework工程的iOS Deployment Target配置，将版本号和ruby脚本最终指定的version保持一致
# 3.再执行子工程项目的配置，仅针对target配置更新Header Search Path配置和Link Binary With Library库引用配置
# 4.跟着同步修改子工程项目配置中的iOS Deployment Target配置，将版本号和ruby脚本最终指定的version保持一致
# 5.最后修改#{target}Tests和#{target}UITests两个单元测试的iOS Deployment Target配置，同样和ruby脚本的version保持一致
# 6.最后同步更新pod中所引用的第三方插件及私有库插件的引用
# ps1: 需要获取build_settings的key时，可以通过修改value查看.xcodeproj文件的变化，或者更简单一点，通过选中具体需要修改的内容，command+c进行复制即可
# ps2: Cocoapods-Xcodeproj-API文档地址：https://www.rubydoc.info/gems/xcodeproj
# ps3: 编写ruby时，建议使用其他非Xcode编译器，例如Sublime Text等含有ruby-sdk环境的编译器，方便脚本编写和方法引用
#

# 定义方法1: 校验文件夹是否存在，不存在则创建
def file_exists(name)
  exists = FileTest::exists?(name)
  if exists == false
    # 目录不存在时需要创建
    puts "#{name}目录不存在，开始创建..."
    Dir.mkdir name 
  end
  return name
end

# 定义方法2: 获取目录下所有子工程名称
def dir_children(dir_name)
  # 获取所有子目录名称，并过滤可能存在的空字符串
  dirs = Dir.children(dir_name).each{|x| x.sub!(/\..*/,"")}
  projects = dirs.reject { |e| e.to_s.empty? }
  puts "#{dir_name}: #{projects}"
  return projects
end



# 指定最小版本号
version = "11.0"
platform :ios, version

# 忽略插件的warning
use_frameworks!
inhibit_all_warnings!

# 指定父级的workspace文件
root_path = File.basename(Dir.pwd)
workspace "#{root_path}.xcworkspace"

# 获取当前的工作路径
frameworks_file = file_exists("XXFrameworks")
module_file = file_exists("XXModule")

# 获取所有frameworks名称，并过滤可能存在的空字符串
frameworks = dir_children(frameworks_file)

# 获取所有module名称，并过滤可能存在的空字符串
targets = dir_children(module_file)

# 保底配置
build_inherited = "$(inherited)"

# linker-framework名称
fw_name = "-framework"

# 判空
if frameworks.nil? == false
  # 配置各个Framework工程
  frameworks.each do |f|
    target f do
      # 配置frameworks工程执行路径
      fxcodeproj_path = "#{frameworks_file}/#{f}/#{f}.xcodeproj"
      project fxcodeproj_path
      xproject = Xcodeproj::Project.open(fxcodeproj_path)
      xproject.targets.each do |x|
        build_configurations = x.build_configurations
        build_configurations.each do |config|
          # 最低支持的版本号
          config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = version
          # 保存配置
          xproject.save
        end
      end
      # pod第三方插件
      pod 'RxSwift'
    end
  end
end

# 判空
if targets.nil? == false
  # 配置各个项目工程
  targets.each do |t|
   target t do
    # 配置子工程执行路径
    pxcodeproj_path = "#{module_file}/#{t}/#{t}.xcodeproj"
    project pxcodeproj_path
    # 打开子工程的.xcodeproj文件
    xproject = Xcodeproj::Project.open(pxcodeproj_path)
    xproject.targets.each do |x|
     # 遍历第一目标文件后，执行修改build_settings
     build_configurations = x.build_configurations
      build_configurations.each do |config|
        if x.name == "#{t}"
            header_settings = [];
            linker_flags = []
            if header_settings.include?(build_inherited) == false
                header_settings.push(build_inherited)
            end
            if linker_flags.include?(build_inherited) == false
                linker_flags.push(build_inherited)
            end
            frameworks.each do |f|
              framework_path = "$(PODS_CONFIGURATION_BUILD_DIR)/#{f}.framework"
              search_path = "#{framework_path}/Headers"
              if header_settings.include?(search_path) == false
                # 添加配置
                header_settings.push(search_path)
              # 添加Link Binary With Library库引用
              linker_flags.push("#{fw_name}")
              linker_flags.push("\"#{f}\"")
              end
            end
          # 更新Header Search Paths配置
          config.build_settings["HEADER_SEARCH_PATHS"] = header_settings
          # 更新Link Binary With Library库引用配置
          config.build_settings["OTHER_LDFLAGS"] = linker_flags
          # 统一配置最小版本号
          config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = version
        end
        # 单元测试-方法测试配置
        if x.name == "#{t}Tests"
          # 最低支持的版本号
          config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = version
        end
        # 单元测试-UI测试配置
        if x.name == "#{t}UITests"
          # 最低支持的版本号
          config.build_settings["IPHONEOS_DEPLOYMENT_TARGET"] = version
        end
        # 保存配置
        xproject.save
      end
    end
    # pod第三方插件
    pod 'RxSwift'
   end
  end
end
