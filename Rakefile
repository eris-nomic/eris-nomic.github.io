require 'rake/clean'
require 'kramdown'
require 'liquid'
require 'yaml'

def template_path template
  template.pathmap('templates/%n.html.liquid')
end

DEFAULT_TEMPLATE = 'base'
templates = {}
templates[DEFAULT_TEMPLATE] = Liquid::Template.parse(File.read(template_path DEFAULT_TEMPLATE))

# From Jekyll, https://github.com/jekyll/jekyll
# available under MIT license
YAML_FRONT_MATTER_REGEXP = %r!\A(---\s*\n.*?\n?)^((---|\.\.\.)\s*$\n?)!m.freeze

KRAMDOWN_OPTS = { :header_links => true, :syntax_highlighter => nil }

task :default => %w[build]

task :build => %w[public html]

markdown_files = FileList.new('**/*.md', '**/*.markdown') do |fl|
  fl.exclude(/^readme/i)
  fl.exclude('~*')
end
task :html => markdown_files.pathmap('public/%n.html')

rule '.html' => [
  proc { |tn| tn.sub(/\.html/, '.md').sub(/^public\//, '') }
] do |t|
  source = File.read t.source
  basename = t.source.pathmap('%n')

  data = {}
  if source =~ YAML_FRONT_MATTER_REGEXP
    source = Regexp.last_match.post_match
    data = YAML.safe_load(Regexp.last_match 1)
  end

  template = data['template'] || basename
  if not templates[template]
    template_file = template_path template
    if File.exist?(template_file)
      templates[template] = Liquid::Template.parse(File.read(template_file))
    else
      template = DEFAULT_TEMPLATE
    end
  end

  doc = Kramdown::Document.new(source, KRAMDOWN_OPTS)
  data['filename'] = basename
  data['body'] = doc.to_html

  File.write(t.name, templates[template].render(data))
end

directory 'public'
file 'public' do
  cp Dir['static/*'], 'public'
end

CLOBBER << 'public'
