desc 'serve the site locally'
task :serve do
  syscall('bundle exec jekyll serve')
end

desc 'build the site'
task build: :clean do
  syscall('bundle exec jekyll build')
end

desc 'deploy the site'
task deploy: :build do
  syscall('rsync -vcrz --delete-after _site/ ssh.nine27.com:nine27.com')
end

desc 'cleanup build site'
task :clean do
  syscall('rm -rf _site')
end

def syscall(command_args)
  system(*command_args, out: $stdout, err: $stderr)
end
