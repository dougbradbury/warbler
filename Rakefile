#--
# Copyright (c) 2010-2012 Engine Yard, Inc.
# Copyright (c) 2007-2009 Sun Microsystems, Inc.
# This source code is available under the MIT license.
# See the file LICENSE.txt for details.
#++

begin
  require 'bundler'
  gem_helper = Bundler::GemHelper.new(Dir.pwd)
  gem_helper.install
  gemspec = gem_helper.gemspec
  require 'bundler/setup'
rescue LoadError
  puts $!
  puts "Please install Bundler and run 'bundle install' to ensure you have all dependencies"
end

require 'rake/clean'
CLEAN << "pkg" << "doc"

require 'spec/rake/spectask'

Spec::Rake::SpecTask.new do |t|
  t.spec_opts ||= []
  t.spec_opts << "--options" << "spec/spec.opts"
end

Spec::Rake::SpecTask.new("spec:rcov") do |t|
  t.spec_opts ||= []
  t.spec_opts << "--options" << "spec/spec.opts"
  t.rcov_opts ||= []
  t.rcov_opts << "-x" << "/gems/"
  t.rcov = true
end

require 'spec/rake/verify_rcov'

RCov::VerifyTask.new(:rcov => "spec:rcov") do |t|
  t.threshold = 100
end

task :default => :spec

desc "Compile and jar the Warbler Java helper classes"
begin
  require 'ant'
  task :jar => :compile do
    ant.jar :basedir => "pkg/classes", :destfile => "lib/warbler_jar.jar", :includes => "*.class"
  end

  directory "pkg/classes"
  task :compile => "pkg/classes" do |t|
    ant.javac :srcdir => "ext", :destdir => t.prerequisites.first,
    :source => "1.5", :target => "1.5", :debug => true,
    :classpath => "${java.class.path}:${sun.boot.class.path}",
    :includeantRuntime => false
  end
rescue LoadError
  task :jar do
    puts "Run 'jar' with JRuby >= 1.5 to re-compile the java jar booster"
  end
end

# Make sure jar gets compiled before the gem is built
task Rake::Task['build'].prerequisites.first => :jar

task :warbler_jar => 'pkg' do
  ruby "-rubygems", "-Ilib", "-S", "bin/warble"
  mv "warbler.jar", "pkg/warbler-#{Warbler::VERSION}.jar"
end

task :build => :warbler_jar

require 'rdoc/task'
RDoc::Task.new(:docs) do |rd|
  rd.rdoc_dir = "doc"
  rd.rdoc_files.include("README.rdoc", "History.txt", "LICENSE.txt")
  rd.rdoc_files += gemspec.require_paths
  rd.options << '--title' << "#{gemspec.name}-#{gemspec.version} Documentation"
  rd.options += gemspec.rdoc_options
end

task :release => :docs do
  config = YAML.load(File.read(File.expand_path("~/.rubyforge/user-config.yml"))) rescue nil
  if config
    dir  = "/var/www/gforge-projects/#{gemspec.rubyforge_project}/#{gemspec.name}"
    dest "#{config["username"]}@rubyforge.org:#{dir}"
    sh %{rsync -av --delete doc/ #{dest}}
  end
end
