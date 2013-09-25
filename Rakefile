def install_github_bundle(user, package)
  unless File.exist? File.expand_path("~/.vim/bundle/#{package}")
    sh "git clone https://github.com/#{user}/#{package} ~/.vim/bundle/#{package}"
  end
end

def apt_update
  sh "sudo apt-get update"
end

def apt_install name
  %x{sudo apt-cache search #{name}}
  sh "sudo apt-get install #{name}"
end

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end

def get_backup_path(path)
  number = 1
  backup_path = "#{path}.bak"
  loop do
    if number > 1
      backup_path = "#{backup_path}#{number}"
    end
    if File.exists?(backup_path) || File.symlink?(backup_path)
      number += 1
      next
    end
    break
  end
  backup_path
end


def link_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.exists?(symlink_path) || File.symlink?(symlink_path)
    if File.symlink?(symlink_path)
      symlink_points_to_path = File.readlink(symlink_path)
      return if symlink_points_to_path == original_path
      # Symlinks can't just be moved like regular files. Recreate old one, and
      # remove it.
      ln_s symlink_points_to_path, get_backup_path(symlink_path), :verbose => true
      rm symlink_path
    else
      # Never move user's files without creating backups first
      mv symlink_path, get_backup_path(symlink_path), :verbose => true
    end
  end
  ln_s original_path, symlink_path, :verbose => true
end

namespace :install do
  desc 'Install Vundle'
  task :vundle do
    step 'vundle'
    install_github_bundle 'gmarik','vundle'
    sh 'vim -c "BundleInstall" -c "q" -c "q"'
  end

  namespace :colors do
    desc "Solarized gnome terminal"
    task :gnome_terminal do
      sh 'colors/gnome-terminal-colors-solarized/set_dark.sh'
    end

    desc "Solarized DirColors"
    task :dircolors do
      link_file 'colors/dircolors-solarized/dircolors.ansi-universal', '~/.dircolors'
    end

    desc "Solarized Guake"
    task :guake do
      sh 'colors/guake-colors-solarized/set_dark.sh'
    end
  end
end


desc 'Install these config files.'
task :default do
  step 'git submodules'
  sh 'git submodule update --init'

  step "apt_update"
  apt_update
  step "install vim"
  apt_install :vim
  step "install tmux"
  apt_install :tmux
  step "install guake"
  apt_install :guake
  step "install xclip"
  apt_install :xclip

  # TODO install gem ctags?
  # TODO run gem ctags?

  step 'symlink'
  link_file 'vim'                   , '~/.vim'
  link_file 'tmux.conf'             , '~/.tmux.conf'
  link_file 'vimrc'                 , '~/.vimrc'
  link_file 'vimrc.bundles'         , '~/.vimrc.bundles'
  unless File.exist?(File.expand_path('~/.vimrc.local'))
    cp File.expand_path('vimrc.local'), File.expand_path('~/.vimrc.local'), :verbose => true
  end
  unless File.exist?(File.expand_path('~/.vimrc.bundles.local'))
    cp File.expand_path('vimrc.bundles.local'), File.expand_path('~/.vimrc.bundles.local'), :verbose => true
  end

  # Install Vundle and bundles
  Rake::Task['install:vundle'].invoke

  step "color scheme"
  puts
  puts 'Manualy command bellow to install the color scheme'
  puts 'rake install:colors:gnome_terminal # if you are using gnome_terminal'
  puts 'rake install:colors:guake # if you are using guake terminal'
  puts
  puts 'enjoy !'
end
