XCODEROOT = %x[xcode-select -print-path].strip
SIMSDKPATH = Dir["#{XCODEROOT}/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator*.sdk/"].sort.last
IOSSDKPATH = Dir["#{XCODEROOT}/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS*.sdk/"].sort.last

task :verify_sysroot => [SIMSDKPATH, IOSSDKPATH]

file "ios_build_config.rb" do

  open('ios_build_config.rb', 'w') do |config_file|

    config_file.puts <<__EOF__
MRuby::Build.new do |conf|
  toolchain :clang

  conf.gembox 'default'
end

SIM_SYSROOT="#{SIMSDKPATH}"
DEVICE_SYSROOT="#{IOSSDKPATH}"

=begin
MRuby::CrossBuild.new('ios-simulator') do |conf|
  conf.bins = []

  conf.gembox 'default'

  conf.cc do |cc|
    cc.command = 'xcrun'
    cc.flags = %W(-v -sdk iphoneos clang -miphoneos-version-min=5.0 -arch x86_64 -isysroot \#{SIM_SYSROOT} -g -O3 -Wall -Werror-implicit-function-declaration)
  end

  conf.linker do |linker|
    linker.command = 'xcrun'
    linker.flags = %W(-v -sdk iphoneos clang -miphoneos-version-min=5.0 -arch x86_64 -isysroot \#{SIM_SYSROOT})
  end
end
=end

MRuby::CrossBuild.new('ios-simulator-x86_64') do |conf|
  conf.bins = []

  conf.gembox 'default'

  conf.cc do |cc|
    cc.command = 'xcrun'
    cc.flags = %W(-sdk iphoneos clang -miphoneos-version-min=5.0 -arch x86_64 -isysroot \#{SIM_SYSROOT} -g -O3 -Wall -Werror-implicit-function-declaration)
  end

  conf.linker do |linker|
    linker.command = 'xcrun'
    linker.flags = %W(-sdk iphoneos clang -miphoneos-version-min=5.0 -arch x86_64 -isysroot \#{SIM_SYSROOT})
  end
end

MRuby::CrossBuild.new('ios-armv7') do |conf|
  conf.bins = []

  conf.gembox 'default'

  conf.cc do |cc|
    cc.command = 'xcrun'
    cc.flags = %W(-sdk iphoneos clang -arch armv7 -isysroot \#{DEVICE_SYSROOT} -g -O3 -Wall -Werror-implicit-function-declaration)
  end

  conf.linker do |linker|
    linker.command = 'xcrun'
    linker.flags = %W(-sdk iphoneos clang -arch armv7 -isysroot \#{DEVICE_SYSROOT})
  end
end

MRuby::CrossBuild.new('ios-armv7s') do |conf|
  conf.bins = []

  conf.gembox 'default'
  conf.cc do |cc|
    cc.command = 'xcrun'
    cc.flags = %W(-sdk iphoneos clang -arch armv7s -isysroot \#{DEVICE_SYSROOT} -g -O3 -Wall -Werror-implicit-function-declaration)
  end

  conf.linker do |linker|
    linker.command = 'xcrun'
    linker.flags = %W(-sdk iphoneos clang -arch armv7s -isysroot \#{DEVICE_SYSROOT})
  end
end

MRuby::CrossBuild.new('ios-arm64') do |conf|
  conf.bins = []

  conf.gembox 'default'
  conf.cc do |cc|
    cc.command = 'xcrun'
    cc.flags = %W(-sdk iphoneos clang -arch arm64 -isysroot \#{DEVICE_SYSROOT} -g -O3 -Wall -Werror-implicit-function-declaration)
  end

  conf.linker do |linker|
    linker.command = 'xcrun'
    linker.flags = %W(-sdk iphoneos clang -arch arm64 -isysroot \#{DEVICE_SYSROOT})
  end
end
__EOF__

  end

end

directory "MRuby.framework/Versions"
directory "MRuby.framework/Versions/1.0.0/"
directory "MRuby.framework/Versions/1.0.0/Headers"
directory "MRuby.framework/Versions/1.0.0/Resources"

task "MRuby.framework/Versions/1.0.0/" => "MRuby.framework/Versions" do

  Dir.chdir("MRuby.framework/Versions/") do
    File.symlink("1.0.0", "Current")
  end
  Dir.chdir("MRuby.framework/") do
    File.symlink("Versions/Current/Headers", "Headers")
    File.symlink("Versions/Current/MRuby", "MRuby")
    File.symlink("Versions/Current/Resources", "Resources")
  end
end

task :build_mruby => "ios_build_config.rb" do
  Dir.chdir("mruby") do
    ENV['MRUBY_CONFIG'] = "../ios_build_config.rb"
    sh "rake"
  end
end

directory "bin"

file "bin/mirb" => [:build_mruby, "bin"] do
  FileUtils.cp "mruby/build/host/bin/mirb", "bin/mirb"
end

file "bin/mrbc" => [:build_mruby, "bin"] do
  FileUtils.cp "mruby/build/host/bin/mrbc", "bin/mrbc"
end

file "bin/mruby" => [:build_mruby, "bin"] do
  FileUtils.cp "mruby/build/host/bin/mruby", "bin/mruby"
end

file "MRuby.framework/Versions/Current/MRuby" => [:build_mruby, "MRuby.framework/Versions/1.0.0/"] do
  #sh "#{IOSSDKPATH}/../../usr/bin/lipo -arch x86_64 mruby/build/ios-simulator/lib/libmruby.a -arch x86_64 mruby/build/ios-simulator-x86_64/lib/libmruby.a -arch arm64 mruby/build/ios-arm64/lib/libmruby.a -arch armv7 mruby/build/ios-armv7/lib/libmruby.a -arch armv7s mruby/build/ios-armv7s/lib/libmruby.a -create -output MRuby.framework/Versions/Current/MRuby"
  sh "#{IOSSDKPATH}/../../usr/bin/lipo -arch x86_64 mruby/build/ios-simulator-x86_64/lib/libmruby.a -arch arm64 mruby/build/ios-arm64/lib/libmruby.a -arch armv7 mruby/build/ios-armv7/lib/libmruby.a -arch armv7s mruby/build/ios-armv7s/lib/libmruby.a -create -output MRuby.framework/Versions/Current/MRuby"
end

task :mruby_headers => [:build_mruby, "MRuby.framework/Versions/1.0.0/Headers"] do
  FileUtils.cp_r "mruby/include/.", "MRuby.framework/Versions/Current/Headers/"

  sh "sed -i '' 's/mruby\\.h/..\\/mruby\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/*"
  sh "sed -i '' 's/mruby\\/khash\\.h/..\\/mruby\\/khash\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/*"
  sh "sed -i '' 's/mruby\\/irep\\.h/..\\/mruby\\/irep\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/proc.h"
  sh "sed -i '' 's/mruby\\/irep\\.h/..\\/mruby\\/irep\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/dump.h"
  sh "sed -i '' 's/mruby\\/object\\.h/..\\/mruby\\/object\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/value.h"
  sh "sed -i '' 's/mruby\\/compile\\.h/..\\/mruby\\/compile\\.h/g' MRuby.framework/Versions/Current/Headers/mruby/irep.h"
end
 
#task :all => [:verify_sysroot, "bin/mirb", "bin/mrbc", "bin/mruby", "MRuby.framework/Versions/Current/MRuby", :mruby_headers]
task :all => [:verify_sysroot, "bin/mrbc", "MRuby.framework/Versions/Current/MRuby", :mruby_headers]

task :default => :all

task :clean => "ios_build_config.rb" do
  Dir.chdir("mruby") do
    ENV['MRUBY_CONFIG'] = "../ios_build_config.rb"
    sh "rake clean"
  end
  FileUtils.rm_f ["ios_build_config.rb", "bin/mirb", "bin/mrbc", "bin/mruby"]
  FileUtils.rm_rf "MRuby.framework"
end
