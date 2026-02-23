require 'fileutils'
require 'rake/clean'
require 'kramdown'
require 'liquid'
require 'yaml'

ROOT_DIR = File.dirname(__FILE__)
SITE_DATA = YAML.safe_load(File.read('site_config.yaml'))
TEMPLATE_DIR = File.join(ROOT_DIR, 'templates')

LIQUID_ENVIRONMENT = Liquid::Environment.build(
  :file_system => Liquid::LocalFileSystem.new(TEMPLATE_DIR),
)
LIQUID_OPTIONS = { environment: LIQUID_ENVIRONMENT }

TEMPLATES = {}
def load_template template
  if not TEMPLATES[template]
    template_file = File.join(TEMPLATE_DIR, template.pathmap('%n.liquid'))
    source = File.read(template_file)
    TEMPLATES[template] = Liquid::Template.parse(source, LIQUID_OPTIONS)
  end
  TEMPLATES[template]
end

DEFAULT_TEMPLATE_NAME = SITE_DATA['default_template'] || 'base'
DEFAULT_TEMPLATE = load_template DEFAULT_TEMPLATE_NAME

# From Jekyll, https://github.com/jekyll/jekyll
# available under MIT license
YAML_FRONT_MATTER_REGEXP = %r!\A(---\s*\n.*?\n?)^((---|\.\.\.)\s*$\n?)!m.freeze

KRAMDOWN_OPTS = { :header_links => true, :syntax_highlighter => nil }

task :default => %w[build]

task :build => %w[public html]

markdown_files = FileList.new('pages/**/*.md') { |fl|
  fl.exclude(/^readme/i)
  fl.exclude('~*')
}.to_a.to_h { |path|
  [path.pathmap('%{^pages,public}d/%n.html'), path]
}
task :html => markdown_files.keys

rule '.html' => [
  proc { |tn| markdown_files[tn] },
] do |t|
  source = File.read t.source
  basename = t.source.pathmap('%n')
  dir = t.source.pathmap('%{^pages,public}d')
  directory dir
  FileUtils.mkdir_p dir

  data = {}
  if source =~ YAML_FRONT_MATTER_REGEXP
    source = Regexp.last_match.post_match
    data = YAML.safe_load(Regexp.last_match 1)
  end

  template_name = data['template'] || DEFAULT_TEMPLATE_NAME
  begin
    template = load_template template_name
  rescue Errno::ENOENT
    STDERR.puts "Error: `#{t.source}' requested template `#{template_name}', but corresponding file not found."
    template = DEFAULT_TEMPLATE
  end

  doc = Kramdown::Document.new(source, KRAMDOWN_OPTS)
  data['filename'] = basename
  data['body'] = doc.to_html
  data['site'] = SITE_DATA

  rendered = template.render!(data)
  File.write(t.name, rendered)
end

directory 'public'
file 'public' do
  cp Dir['static/*'], 'public'
end

CLOBBER << 'public'

task :serve => :build do
  require 'webrick'

  root = File.expand_path(File.join(ROOT_DIR, 'public'))

  server = WEBrick::HTTPServer.new :Port => 8080, :DocumentRoot => root
  trap 'INT'  do server.shutdown end
  trap 'TERM' do server.shutdown end
  server.start
end
