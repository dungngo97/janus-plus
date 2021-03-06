ROOT_PATH = File.expand_path(File.join(File.dirname(__FILE__)))
$: << File.join(ROOT_PATH, 'janus', 'ruby')

require 'janus'
require 'fileutils'
include Janus

desc "link ViM configuration files."
task :link_vim_conf_files do
  %w[ vimrc gvimrc tmux.conf tigrc ctags].each do |file|
    dest = expand("~/.#{file}")
    unless File.exist?(dest)
      require 'rbconfig'
      source = expand("../janus/vim/#{file}", __FILE__)
      if RbConfig::CONFIG['host_os'] =~ /mswin|mingw/
        require 'open3'
        dest.gsub!(?/, ?\\)
        source.gsub!(?/, ?\\)
        puts "cmd.exe /c mklink #{dest} #{source}"
        stdin, stdout, stderr, wait_thr = Open3.popen3('cmd.exe', "/c mklink #{dest} #{source}")
        wait_thr.value.exitstatus
      else
        ln_s(source, dest)
      end
    end
  end
end

task :link_git_conf_files do
  require_relative "janus/git/lib/setup"
  Git::Setup.new.perform
end

namespace :dev do
  desc "Update submodules"
  task :update_submodules do
    sh "git submodule foreach 'git fetch origin && git reset --hard origin/master'"
  end

  # Taken from RefineryCMD
  # https://github.com/resolve/refinerycms/blob/master/core/lib/tasks/refinery.rake
  desc 'Removes trailing whitespace across the entire application.'
  task :whitespace do
    require 'rbconfig'
    if RbConfig::CONFIG['host_os'] =~ /linux/
      sh %{find . -name '*.*rb' -o -name '*.*vim' -exec sed -i 's/\t/ /g' {} \\; -exec sed -i 's/ *$//g' {} \\; }
    elsif RbConfig::CONFIG['host_os'] =~ /darwin/
      sh %{find . -name '*.*rb' -o -name '*.*vim' -exec sed -i '' 's/\t/ /g' {} \\; -exec sed -i '' 's/ *$//g' {} \\; }
    else
      puts "This doesn't work on systems other than OSX or Linux. Please use a custom whitespace tool for your platform '#{RbConfig::CONFIG["host_os"]}'."
    end
  end
end

desc "Create necessary folders."
task :folders do
  Janus::VIM.folders.each do |folder|
    mkdir_p folder
  end
end

task :clean do
  puts "Cleaning the janus folder"
  `git clean -xdf -- janus`
  `git ls-files --exclude-standard --others -- janus`.split("\n").each do |untracked|
    FileUtils.rm_rf File.expand_path(untracked.chomp, File.dirname(__FILE__))
  end
end

task :fetch_origin do
  puts "Fetching latest changes"
  sh "git remote update origin"

  puts "Cleaning the janus folder"
  sh "git reset --hard origin/master"
end

task :fetch_local, [:branch] do |task, args|
  puts "Fetching latest changes"
  sh "git remote update local"

  puts "Cleaning the janus folder"
  sh "git reset --hard local/#{args[:branch]}"
end

task :fetch_submodules do
  puts "Synchronising submodules urls"
  `git submodule sync`

  puts "Updating the submodules"
  `git submodule update --init`
end

task :vim_plug do
  sh "vim +PlugInstall +qall"
  sh "reset"
end

task :install => [:folders, :link_vim_conf_files, :link_git_conf_files, :vim_plug] do
  # Dummy task, real work is done with the hooks.
end

desc "Install or Update Janus."
task :default do
  Rake::Task["clean"].invoke
  Rake::Task["fetch_origin"].invoke
  Rake::Task["fetch_submodules"].invoke
  sh "rake install"
end

def blank?(value)
  return true if value.nil?
  value == ""
end

desc "Install or Update Janus using the local repo"
task :local, [:repo, :branch] do |task, args|
  if blank?(args[:repo])
    puts "Repo is not provided"
    exit 1
  end

  if blank?(args[:branch])
    puts "Branch is not provided"
    exit 2
  end

  system "git remote rm local"
  sh "git remote add local #{args[:repo]}"
  Rake::Task["clean"].invoke
  Rake::Task["fetch_local"].invoke(args[:branch])
  Rake::Task["fetch_submodules"].invoke
  sh "rake install"
end
