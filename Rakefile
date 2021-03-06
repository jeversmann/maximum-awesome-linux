require 'tmpdir'

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end

def link_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.exists?(symlink_path)
    # Symlink already configured properly. Leave it alone.
    return if File.symlink?(symlink_path) and File.readlink(symlink_path) == original_path
    # Never move user's files without creating backups first
    number = 1
    loop do
      backup_path = "#{symlink_path}.bak"
      if number > 1
        backup_path = "#{backup_path}#{number}"
      end
      if File.exists?(backup_path)
        number += 1
        next
      end
      mv symlink_path, backup_path, :verbose => true
      break
    end
  end
  ln_s original_path, symlink_path, :verbose => true
end

# instructions from http://www.webupd8.org/2011/04/solarized-must-have-color-paletter-for.html
def solarize(color)
  Dir.chdir 'gnome-terminal-colors-solarized' do
    sh "./set_#{color}.sh"
  end

  step 'fix ls-colors'
  Dir.chdir do
    sh "wget --quiet --no-check-certificate https://raw.github.com/seebi/dircolors-solarized/master/dircolors.ansi-#{color}"
    sh "mv dircolors.ansi-#{color} .dircolors"
    sh 'eval `dircolors .dircolors`'
  end
end

namespace :install do

  desc 'Apt-get Update'
  task :update do
    step 'apt-get update'
    sh 'sudo apt-get update -q'
  end

  desc 'Install Vim'
  task :vim do
    step 'vim'
    sh 'sudo apt-get install -q vim'
    link_file 'vim', '~/.vim'
    link_file 'vimrc', '~/.vimrc'
    link_file 'vimrc.local', '~/.vimrc.local'
  end

  desc 'Install zsh'
  task :zsh do
    step 'zsh'
    sh 'sudo apt-get install -q zsh'
    # Need to chsh
  end

  desc 'Install oh-my-zsh'
  task :oh_my_zsh => :zsh do
    step 'oh-my-zsh'
    sh 'git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh' unless Dir.exist? "#{Dir.home}/.oh-my-zsh"
    link_file 'zshrc', '~/.zshrc'
  end

  desc 'Install tmux'
  task :tmux do
    step 'tmux'
    sh 'sudo apt-get install -q tmux'
    link_file 'tmux.conf', '~/.tmux.conf'
  end

  desc 'Install ctags'
  task :ctags do
    step 'ctags'
    sh 'sudo apt-get install -q ctags'
  end

  # https://github.com/ggreer/the_silver_searcher
  desc 'Install The Silver Searcher'
  task :the_silver_searcher do
    step 'the_silver_searcher'
    sh 'sudo apt-get install -q build-essential automake pkg-config libpcre3-dev zlib1g-dev liblzma-dev'
    sh 'git clone https://github.com/ggreer/the_silver_searcher.git' unless Dir.exists? 'the_silver_searcher'
    Dir.chdir 'the_silver_searcher' do
      sh 'git pull'
      sh './build.sh > /dev/null'
      sh 'sudo make install'
    end
  end

  desc 'Download Solarized'
  task :solarized do
    step 'solarized'
    sh 'sudo apt-get install -q gnome-terminal'
    sh 'git clone https://github.com/sigurdga/gnome-terminal-colors-solarized.git' unless File.exist? 'gnome-terminal-colors-solarized'
  end

  desc 'Install Solarized Dark'
  task :solarized_dark => :solarized do
    solarize 'dark'
  end

  desc 'Install Solarized Light'
  task :solarized_light => :solarized do
    solarize 'light'
  end
end

desc 'Install the things that make this computer mine.'
task :default => [
  'install:update',
  'install:vim',
  'install:tmux',
  'install:ctags',
  'install:the_silver_searcher',
  'install:zsh',
  'install:oh_my_zsh'
] do

  step 'git submodules'
  sh 'git submodule update --init'

  step 'solarized dark or light'
  puts ' Run: "rake install:solarized_dark" or "rake install:solarized_light"'
  puts ' You may need to close your terminal and re-open it for it to take effect.'
end
