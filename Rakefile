# frozen_string_literal: true

require 'open-uri'
require 'shellwords'
require 'bundler/audit/task'
require 'rubocop/rake_task'

task default: %i[format lint]

desc 'Lint sources'
task lint: %i[lint:rubocop:autocorrect]

namespace :lint do
  RuboCop::RakeTask.new(:rubocop)
end

desc 'Format sources'
task format: %i[format:text]

namespace :format do
  desc 'Format text, YAML, and Markdown sources with prettier'
  task :text do
    sh 'npm run fmt'
  end
end

desc 'Format sources'
task fmt: %i[fmt:text]

namespace :fmt do
  desc 'Format text, YAML, and Markdown sources with prettier'
  task :text do
    sh 'npm run fmt'
  end
end

Bundler::Audit::Task.new

namespace :release do
  link_check_files = FileList.new('**/*.md') do |f|
    f.exclude('node_modules/**/*')
    f.exclude('**/target/**/*')
    f.exclude('**/vendor/*/**/*')
    f.include('*.md')
    f.include('**/vendor/*.md')
  end

  link_check_files.sort.uniq.each do |markdown|
    desc 'Check for broken links in markdown files'
    task markdown_link_check: markdown do
      command = ['npx', 'markdown-link-check', '--config', '.github/markdown-link-check.json', markdown]
      sh command.shelljoin
      sleep(rand(1..5))
    end
  end
end

RUST_VERSION = '1.78.0'

namespace :toolchain do
  desc 'Sync Rust toolchain to all sources'
  task sync: %i[sync:dockerfiles]

  namespace :sync do
    desc 'Sync the Rust toolchain to all Dockerfiles'
    task :dockerfiles do
      regexp = /^ARG RUST_VERSION=(.+)$/
      next_rust_version = "ARG RUST_VERSION=#{RUST_VERSION}"

      dockerfiles = FileList.new('**/Dockerfile')

      failures = dockerfiles.map do |file|
        contents = File.read(file)

        if (existing_version = contents.match(regexp))
          File.write(file, contents.gsub(regexp, next_rust_version)) if existing_version != RUST_VERSION
          next
        end

        puts "Failed to update #{file}, ensure there is a RUST_VERSION arg specified" if Rake.verbose
        file
      end.compact

      raise 'Failed to update some RUST_VERSION args' if failures.any?
    end
  end

  desc 'Output the current toolchain version'
  task :version do
    puts RUST_VERSION
  end
end
