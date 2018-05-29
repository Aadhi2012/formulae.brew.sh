require "rake"
require "rake/clean"
require "json"
require "date"

task default: :dump

desc "Dump formulae data"
task :formulae do
  sh "brew", "ruby", "script/generate.rb"
end

def generate_analytics?
  json_file = "_data/analytics/build-error/30d.json"
  return true unless File.exist?(json_file)

  json = JSON.parse(IO.read(json_file))
  end_date = Date.parse(json["end_date"])
  end_date < Date.today
end

def setup_analytics_credentials
  ga_credentials = ".homebrew_analytics.json"
  return unless File.exist?(ga_credentials)

  ga_credentials_home = File.expand_path("~/#{ga_credentials}")
  return if File.exist?(ga_credentials_home)

  FileUtils.cp ga_credentials, ga_credentials_home
end

def setup_formula_analytics_cmd
  ENV["HOMEBREW_NO_AUTO_UPDATE"] = "1"
  unless `brew tap`.include?("homebrew/formula-analytics")
    sh "brew", "tap", "Homebrew/formula-analytics"
  end

  sh "brew", "formula-analytics", "--setup"
end

def setup_analytics
  setup_analytics_credentials
  setup_formula_analytics_cmd
end

desc "Dump analytics data"
task :analytics do
  next unless generate_analytics?

  setup_analytics

  %w[build-error install install-on-request os-version
     core-build-error core-install core-install-on-request].each do |category|
    case category
    when "core-build-error"
      category = "all-core-formulae-json --build-error"
      category_name = "build-error/homebrew-core"
    when "core-install"
      category = "all-core-formulae-json --install"
      category_name = "install/homebrew-core"
    when "core-install-on-request"
      category = "all-core-formulae-json --install-on-request"
      category_name = "install-on-request/homebrew-core"
    else
      category_name = category
    end
    FileUtils.mkdir_p "_data/analytics/#{category_name}"
    %w[30 90 365].each do |days|
      next if days != "30" && category_name.include?("homebrew-core")
      sh "brew formula-analytics --days-ago=#{days} --json --#{category} " \
        "> _data/analytics/#{category_name}/#{days}d.json"
    end
  end
end

desc "Dump formulae and analytics data"
task dump: %i[formulae analytics]

desc "Build the site"
task jekyll: :dump do
  sh "bundle", "exec", "jekyll", "build"
end

CLEAN.include FileList["_site"]
