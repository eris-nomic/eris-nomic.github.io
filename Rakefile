require 'rake/clean'
require 'kramdown'

DEFAULT_TEMPLATE = 'base.template.html'

task :default => %w[build]

task :build => %w[public html]

markdown_files = FileList.new('**/*.md', '**/*.markdown') do |fl|
  fl.exclude(/^readme/i)
  fl.exclude('~*')
end
task :html => markdown_files.pathmap('public/%n.html')

def md_to_html(t)
  template = t.source.pathmap('templates/%n.template.html')
  template = DEFAULT_TEMPLATE if not File.exists? template
  opts = { :header_links => true, :syntax_highlighter => nil, template: }
  text = File.read t.source
  doc = Kramdown::Document.new(text, opts)
  File.write(t.name, doc.to_html)
end

rule '.md' => '.html' do |t|
  md_to_html t
end
rule '.markdown' => '.html' do |t|
  md_to_html t
end

directory 'public'
file 'public' do
  cp Dir['static/*'], 'public'
end

CLOBBER << 'public'
