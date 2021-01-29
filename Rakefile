namespace :book do
  def exec_or_raise(command)
    puts `#{command}`
    if (! $?.success?)
      raise "'#{command}' failed"
    end
  end

  # Variables referenced for build
  version_string = ENV['TRAVIS_TAG'] || `git describe --tags`.chomp
  if version_string.empty?
    version_string = '0'
  end
  date_string = Time.now.strftime("%Y-%m-%d")
  params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}'"

  desc 'build basic book formats'
  task :build => [:build_html, :build_epub, :build_pdf] do
    begin
        # Run check
        Rake::Task["book:check"].invoke

        # Rescue to ignore checking errors
        rescue => e
        puts e.message
        puts "Error when checking books (ignored)"
    end
  end

  desc 'build basic book formats (for ci)'
  task :ci => [:build_html, :build_epub, :build_pdf] do
    begin
        # Run check, but don't ignore any errors
        Rake::Task["book:check"].invoke
    end
  end

  desc 'generate contributors list'
  file 'book/contributors.txt' do
      puts "Generating contributors list"
      `git shortlog -s | grep -v -E "(Straub|Chacon|dependabot)" | cut -f 2- | column -c 120 > book/contributors.txt`
  end

  desc 'build HTML format'
  task :build_html => 'book/contributors.txt' do
      puts "Converting to HTML..."
      `bundle exec asciidoctor #{params} -a data-uri progit.asc`
      puts " -- HTML output at progit.html"

  end

  desc 'build Epub format'
  task :build_epub => 'book/contributors.txt' do
      puts "Converting to EPub..."
      `bundle exec asciidoctor-epub3 #{params} progit.asc`
      puts " -- Epub output at progit.epub"

  end

  desc 'build Mobi format'
  task :build_mobi => 'book/contributors.txt' do
      # Commented out the .mobi file creation because the kindlegen dependency is not available.
      # For more information on this see: #1496.
      # This is a (hopefully) temporary fix until upstream asciidoctor-epub3 is fixed and we can offer .mobi files again.

      # puts "Converting to Mobi (kf8)..."
      # `bundle exec asciidoctor-epub3 #{params} -a ebook-format=kf8 progit.asc`
      # puts " -- Mobi output at progit.mobi"

      # FIXME: If asciidoctor-epub3 supports Mobi again, uncomment these lines below
      puts "Converting to Mobi isn't supported yet."
      puts "For more information see issue #1496 at https://github.com/progit/progit2/issues/1496."
      exit(127)
  end

  desc 'build PDF format'
  task :build_pdf => 'book/contributors.txt' do
      puts "Converting to PDF... (this one takes a while)"
      `bundle exec asciidoctor-pdf #{params} progit.asc 2>/dev/null`
      puts " -- PDF output at progit.pdf"
  end

  desc 'Check generated books'
  task :check => [:build_html, :build_epub] do
    begin
        puts "Checking generated books"

        exec_or_raise('htmlproofer --check-html progit.html')
        exec_or_raise('epubcheck progit.epub')
    end
  end

  desc 'Clean all generated files'
  task :clean do
    begin
        puts "Removing generated files"

        FileList['book/contributors.txt', 'progit.html', 'progit.epub', 'progit.pdf'].each do |file|
            rm file

            # Rescue if file not found
            rescue Errno::ENOENT => e
            puts e.message
            puts "Error removing files (ignored)"
        end
    end
  end

end

task :default => "book:build"
