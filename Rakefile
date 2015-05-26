require 'yaml'
TARGET = 'target'

ENGINE = YAML.load_file('config/engines.yaml')
FLOWS = YAML.load_file('config/flows.yaml')

task :clean do
  rm_rf TARGET
end

#LANG = ENV['LANG'][0..1]
LANG='pt'


FileList["udc/#{LANG}/**/*.adoc","features/**/*.adoc"].each do |f|
  basedir = f.split(/\//)[0]
  
  ["asciidoc-docbook45", "asciidoctor-docbook45", "asciidoctor-docbook5"].each do |asciidoc_to_docbook|
    
    asciidoc_to_docbook_dir = "#{TARGET}/#{asciidoc_to_docbook}/#{f}"
    asciidoc_to_docbook_adoc = "#{asciidoc_to_docbook_dir}/#{asciidoc_to_docbook}.adoc"
    asciidoc_to_docbook_xml  = asciidoc_to_docbook_adoc.ext('xml')
    directory asciidoc_to_docbook_dir => f
    file asciidoc_to_docbook_adoc => [f,asciidoc_to_docbook_dir]  do |t|
      cp_r "#{File.dirname(f)}/.", File.dirname(t.name)
      File.rename "#{asciidoc_to_docbook_dir}/#{File.basename(f)}", t.name
    end

    file asciidoc_to_docbook_xml => [asciidoc_to_docbook_adoc] do |t|
      Dir.chdir(File.dirname(t.name)) do
        system "#{ENGINE[asciidoc_to_docbook]} #{File.basename(t.name).ext('adoc')}"
        rm "#{File.basename(t.name).ext('adoc')}"
      end    
    end
    desc 'Build feature docbook file'
    task "#{basedir}:#{asciidoc_to_docbook}" => asciidoc_to_docbook_xml
    
    desc "Build feature docbook with all engines"
    task "#{basedir}:docbook" => asciidoc_to_docbook_xml
    
    
    ["dblatex-pdflatex", "dblatex-xetex", "dbcontext-context", "fopub"].each do |docbook_to_pdf|
      flow="#{asciidoc_to_docbook}-#{docbook_to_pdf}"
      next if FLOWS['ignore'].include? flow
      pdf_dir  = "#{TARGET}/#{f}"
      pdf_file = "#{pdf_dir}/#{docbook_to_pdf}-from-#{asciidoc_to_docbook}.pdf"
      directory pdf_dir
      file pdf_file.ext('xml') => [asciidoc_to_docbook_xml, pdf_dir] do |t|
        cp_r "#{File.dirname(asciidoc_to_docbook_xml)}/.", File.dirname(t.name)
        File.rename "#{pdf_dir}/#{File.basename(asciidoc_to_docbook_xml)}", t.name
      end
      file pdf_file => [pdf_file.ext('xml')] do |t|
        Dir.chdir(File.dirname(t.name)) do
          system "#{ENGINE[docbook_to_pdf]} #{File.basename(t.name).ext('xml')}"
        end
      end
      
      desc "Build pdf using this toolchain"
      task "#{basedir}:#{asciidoc_to_docbook}:#{docbook_to_pdf}" => pdf_file
      
      desc "Build these pdfs"
      task "#{basedir}:pdf" => [pdf_file]
      
    end
  end
end