require 'date'

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

namespace :post do
  desc 'create a new post'
  task :new, [:title] do |t, args|
    raise 'must provide title' unless args[:title]

    title = args[:title].strip
    file_title = title.gsub(/\W/, '-').downcase
    time = Time.now

    File.open("./_posts/#{time.to_date}-#{file_title}.md", 'w') do |f|
      f.puts <<~END
               ---
               layout: post
               title:  #{title}
               date:   #{time.to_s}
               tags:   []
               ---
             END
    end
  end
end

def syscall(command_args)
  system(*command_args, out: $stdout, err: $stderr)
end
