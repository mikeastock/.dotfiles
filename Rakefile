require "tmpdir"

################################################################################
# Constants and helpers
################################################################################

COLORS = {
  red:    "1;31",
  yellow: "1;33",
}

INSTALL_TASKS = %w[
  links
  shell
]

##
# Each key corresponds to a file in the +files+ directory, and each value is the
# destination of the symlink.
#
MAPPINGS = {
  "Xresources"         => "~/.Xresources",
  "aliases"            => "~/.aliases",
  "bash_aliases"       => "~/.bash_aliases",
  "bashrc"             => "~/.bashrc",
  "bin"                => "~/bin",
  "gemrc"              => "~/.gemrc",
  "gitconfig"          => "~/.gitconfig",
  "gitignore_global"   => "~/.gitignore_global",
  "itermcolors"        => "~/.itermcolors",
  "rspec"              => "~/.rspec",
  "tmux.conf"          => "~/.tmux.conf",
  "tmuxinator"         => "~/.tmuxinator",
  "vim"                => %w[~/.vim ~/.nvim],
  "vimrc"              => %w[~/.vimrc ~/.nvimrc],
  "vimrc.plugins"      => "~/.vimrc.plugins",
  "xinitrc"            => "~/.xinitrc",
  "zsh"                => "~/.zsh",
}

LINUX_MAPPINGS = {
  "tag-linux/zsh/"     => "~/.zsh",
  "tag-linux/zshrc"    => "~/.zshrc",
}

MAC_MAPPINGS = {
  "tag-mac/zprofile"   => "~/.zprofile",
  "tag-mac/zshrc"      => "~/.zshrc",
}

PREFERRED_SHELL = "zsh"

##
# Wraps Kernel#warn with support for colors.
#
alias :kernel_warn :warn
def warn(msg, color=:red)
  kernel_warn "\e[#{COLORS[color]}m#{msg}\e[m"
end

def force?
  @force ||= build_force
end

def build_force
  ENV["FORCE"] == "yes"
end

##
# Symlinks +src+ file or directory to +target+
#
def link_file(source, target)
  source = "#{pwd}/files/#{source}"
  target = File.expand_path target
  rm_rf target if File.exists?(target) and force?
  if File.directory? target
    warn "#{target} is a directory. I'm not symlinking that unless you use force=yes", :yellow
  else
    FileUtils.mkdir_p File.dirname(target)
    FileUtils.ln_s(source, target, force: force?)
  end
rescue
  warn "Couldn't create #{target} because it exists. Use `force=yes` to overwrite."
end

def all_mappings
  if linux?
    MAPPINGS.merge(LINUX_MAPPINGS)
  elsif mac?
    MAPPINGS.merge(MAC_MAPPINGS)
  end
end

def linux?
  Gem::Platform.local.os == "linux"
end

def mac?
  Gem::Platform.local.os == "darwin"
end

################################################################################
# Tasks
################################################################################

task default: :update_and_force

desc "Same as install, but overwrites any existing files."
task :force do
  ENV["force"] = "yes"
  Rake::Task[:install].invoke
end

desc "Run these tasks in order: #{INSTALL_TASKS.join(' ')}"
task install: INSTALL_TASKS

desc "Symlink config files to appropriate locations. (force=yes to overwrite)"
task :links do
  all_mappings.each do |source, target|
    if target.is_a? Array
      target.each do |targetlet|
        link_file(source, targetlet)
      end
    else
      link_file(source, target)
    end
  end
end

desc "Set preferred shell"
task shell: :packages do
  `sudo chsh -s $(which #{PREFERRED_SHELL}) $(whoami)`
end

# Because this may update the Rakefile, we depend on the update task, then we
# actually exec a call to rake in the shell.
desc "[Default] Update repository and run force task"
task update_and_force: :update do
  exec "rake force" if $? == 0
end

desc "Update repository"
task :update do
  `git fetch --prune`
  `git pull --rebase origin master`
end