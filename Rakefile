require 'puppetlabs_spec_helper/rake_tasks'

# load optional tasks for releases
# only available if gem group releases is installed
begin
  require 'puppet_blacksmith/rake_tasks'
  require 'voxpupuli/release/rake_tasks'
  require 'puppet-strings/tasks'
rescue LoadError
end

PuppetLint.configuration.log_format = '%{path}:%{line}:%{check}:%{KIND}:%{message}'
PuppetLint.configuration.fail_on_warnings = true
PuppetLint.configuration.send('relative')
PuppetLint.configuration.send('disable_140chars')
PuppetLint.configuration.send('disable_class_inherits_from_params_class')
PuppetLint.configuration.send('disable_documentation')
PuppetLint.configuration.send('disable_single_quote_string_with_variables')

exclude_paths = %w[
  pkg/**/*
  vendor/**/*
  .vendor/**/*
  spec/**/*
]
PuppetLint.configuration.ignore_paths = exclude_paths
PuppetSyntax.exclude_paths = exclude_paths

desc 'Auto-correct puppet-lint offenses'
task 'lint:auto_correct' do
  PuppetLint.configuration.fix = true
  Rake::Task[:lint].invoke
end

desc 'Run acceptance tests'
RSpec::Core::RakeTask.new(:acceptance) do |t|
  t.pattern = 'spec/acceptance'
end

desc 'Check for trailing whitespace'
task :trailing_whitespace do
  puts "\nChecking for trailing whitespace"
  Dir.glob('**/*.md', File::FNM_DOTMATCH).sort.each do |f|
    next unless f !~ /^(modules\/|acceptance\/|vendor\/|\.vendor\/|spec\/fixtures\/|pkg\/)/
    puts "  #{f}"
    cnt = 0
    File.foreach(f) do |line|
      cnt += 1
      if line =~ %r{\s\n$}
        puts "  #{f} has trailing whitespace on line #{cnt}"
        exit 1
      end
    end
  end
end

desc 'Run tests metadata_lint, release_checks, trailing_whitespace'
task test: [
  :metadata_lint,
  :release_checks,
  :trailing_whitespace
]

desc "Run main 'test' task and report merged results to coveralls"
task test_with_coveralls: [:test] do
  if Dir.exist?(File.expand_path('../lib', __FILE__))
    require 'coveralls/rake/task'
    Coveralls::RakeTask.new
    Rake::Task['coveralls:push'].invoke
  else
    puts 'Skipping reporting to coveralls.  Module has no lib dir'
  end
end

desc 'Print supported beaker sets'
task 'beaker_sets', [:directory] do |_t, args|
  directory = args[:directory]

  metadata = JSON.load(File.read('metadata.json'))

  (metadata['operatingsystem_support'] || []).each do |os|
    (os['operatingsystemrelease'] || []).each do |release|
      beaker_set = if directory
                     "#{directory}/#{os['operatingsystem'].downcase}-#{release}"
                   else
                     "#{os['operatingsystem'].downcase}-#{release}-x64"
                   end

      filename = "spec/acceptance/nodesets/#{beaker_set}.yml"

      puts beaker_set if File.exist? filename
    end
  end
end

begin
  require 'github_changelog_generator/task'
  GitHubChangelogGenerator::RakeTask.new :changelog do |config|
    version = Blacksmith::Modulefile.new.version
    config.future_release = "v#{version}" if version =~ %r{^\d+\.\d+.\d+$}
    config.header = "# Changelog\n\nAll notable changes to this project will be documented in this file.\nEach new release typically also includes the latest modulesync defaults.\nThese should not affect the functionality of the module."
    config.exclude_labels = %w[duplicate question invalid wontfix wont-fix modulesync skip-changelog]
    config.user = 'voxpupuli'
    metadata_json = File.join(File.dirname(__FILE__), 'metadata.json')
    metadata = JSON.load(File.read(metadata_json))
    config.project = metadata['name']
  end
rescue LoadError
end
# vim: syntax=ruby
