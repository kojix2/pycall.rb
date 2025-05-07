require "bundler/gem_helper"
require "rake/clean"

base_dir = File.join(File.dirname(__FILE__))

helper = Bundler::GemHelper.new(base_dir)
helper.install
spec = helper.gemspec

def run_extconf(build_dir, extension_dir, *arguments)
  cd(build_dir) do
    ruby(File.join(extension_dir, "extconf.rb"), *arguments)
  end
end

def make_command
  if RUBY_PLATFORM =~ /mswin/
    "nmake"
  else
    ENV["MAKE"] || find_make
  end
end

def find_make
  candidates = ["gmake", "make"]
  paths = ENV.fetch("PATH", "").split(File::PATH_SEPARATOR)
  exeext = RbConfig::CONFIG["EXEEXT"]
  candidates.each do |candidate|
    paths.each do |path|
      cmd = File.join(path, "#{candidate}#{exeext}")
      return cmd if File.executable?(cmd)
    end
  end
end

Dir[File.expand_path('../tasks/**/*.rake', __FILE__)].each {|f| load f }

require "rake/extensiontask"

# ExtensionTask for the main pycall extension module
Rake::ExtensionTask.new("pycall") do |ext|
  ext.lib_dir = "lib/pycall/ext"
  ext.ext_dir = "ext/pycall"
  
  # Support for BUILD_DIR environment variable
  if ENV["BUILD_DIR"]
    ext.tmp_dir = ENV["BUILD_DIR"]
  end
end

# ExtensionTask for spec_helper
Rake::ExtensionTask.new("pycall/spec_helper") do |ext|
  # Support for BUILD_DIR environment variable
  if ENV["BUILD_DIR"]
    ext.tmp_dir = ENV["BUILD_DIR"]
  end
end

desc "Run tests"
task :test do
  cd(base_dir) do
    ruby("test/run-test.rb")
  end
end

task default: :test

require "rspec/core/rake_task"
RSpec::Core::RakeTask.new(:spec) do |t|
  ext_dir = File.join(base_dir, "ext/pycall")
  t.ruby_opts = "-I#{ext_dir}"
  t.verbose = true
end

task default: :spec
task spec: :compile
