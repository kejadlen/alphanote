require "pry"

namespace :sync do
  desc "Sync Kindle highlights from GoodReads"
  task :good_reads, [:url] do |t, args|
    url = args.fetch(:url)

    notes = GoodReads::Notes.download(url)
    s = notes.to_s.gsub(/â\u0080\u0099d/, ?')
    puts s
  end

  namespace :recipe do
    task :chef_steps
    task :serious_eats
  end
end

module GoodReads
  Notes = Struct.new(:title, :author, :notes) do
    def self.download(url)
      require "nokogiri"
      require "open-uri"
      require "uri"

      uri = URI(url)
      doc = Nokogiri::HTML(uri.open, nil, "UTF-8")

      title = doc.css("div.readingNotesBookDetailsContainer a")[0].text
      author = doc.css("div.readingNotesBookDetailsContainer a")[1].text

      notes = []
      loop do
        doc.css("div.noteHighlightContainer").each do |node|
          location = node.css("div.noteHighlightContainer__location a").text
          highlight = node.css("div.noteHighlightTextContainer__highlightText span").last.text
          note = node.css("div.noteContainer__noteText").text
          notes << Note.new(location, highlight, note)
        end

        next_page = doc.at_xpath('//a[contains(@class, "next_page") and not(contains(@class, "disabled"))]')
        break unless next_page

        href = URI(next_page["href"])
        uri.path = href.path
        uri.query = href.query
        doc = Nokogiri::HTML(uri.open, nil, "UTF-8")
      end

      new(title, author, notes)
    end

    def to_s
      s = <<~EOF
      ---
      tags: [ book ]
      ---
      # #{title}

      by #{author}

      #{notes.map(&:to_s).join("\n\n")}
      EOF
    end
  end

  Note = Struct.new(:location, :highlight, :note) do
    def to_s
      s = []
      s << "> #{highlight}" unless highlight.empty?
      s << "#{note}" unless note.empty?
      s.join("\n\n")
    end
  end
end
