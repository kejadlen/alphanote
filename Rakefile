require "pry"

namespace :download do
  desc "Download Kindle highlights from GoodReads"
  task :good_reads, [:url] do |t, args|
    url = args.fetch(:url, `pbpaste`)

    require "yaml"

    puts YAML.dump(
      GoodReads::Notes.download(url).to_h.transform_keys(&:to_s)
    )
  end

  namespace :recipe do
    task :chef_steps

    desc "Download recipe from Serious Eats"
    task :serious_eats, [:url] do |t, args|
      url = args.fetch(:url)

      puts SeriousEats::Recipe.download(url)
    end
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
          highlight = node.css("div.noteHighlightTextContainer__highlightText span").last.text rescue nil
          note = node.css("div.noteContainer__noteText").text

          highlight.gsub!(/â\u0080\u0099d/, ?')

          notes <<
            Note.new(location, highlight, note)
              .to_h
              .reject {|_,v| v.nil? || v.empty? }
              .transform_keys(&:to_s)
        end

        next_page = doc.at_xpath("//a[contains(@class, 'next_page') and not(contains(@class, 'disabled'))]")
        break unless next_page

        href = URI(next_page["href"])
        uri.path = href.path
        uri.query = href.query
        doc = Nokogiri::HTML(uri.open, nil, "UTF-8")
      end

      new(title, author, notes)
    end
  end

  Note = Struct.new(:location, :highlight, :note)
end

module SeriousEats
  Recipe = Struct.new(:url, :title, :tags, :about, :ingredients, :directions) do
    def self.download(url)
      require "erb"
      require "json"
      require "nokogiri"
      require "open-uri"
      require "uri"

      uri = URI(url)
      doc = Nokogiri::HTML(uri.open, nil, "UTF-8")

      linked_data = doc
        .xpath("/html/head/script[@type='application/ld+json']")
        .map(&:inner_text)
        .map {|raw| JSON.parse(raw) }
      recipe = linked_data.find {|json| json.fetch("@type") == "Recipe" }

      about = doc.at_xpath("//ul[contains(@class, 'recipe-about')]")
      active_time = about.at_xpath(".//span[text()='Active time:']/following-sibling::span").inner_text
      total_time = about.at_xpath(".//span[text()='Total time:']/following-sibling::span").inner_text

      new(
        url,
        recipe.fetch("name"),
        recipe.fetch("keywords").split(/,\s*/).map {|s| s.gsub(/\W+/, ?-) },
        {
          "Yield" => recipe.fetch("recipeYield"),
          "Active time" => active_time,
          "Total time" => total_time,
        },
        recipe.fetch("recipeIngredient"),
        recipe.fetch("recipeInstructions").map {|instruction| instruction.fetch("text") },
      )
    end

    def to_s
      # ERB.new(<<~EOF, trim_mode: "<>").result(binding)
      <<~EOF
        ---
        tags:
        #{%w[recipe].concat(tags).map {|t| "  - #{t}" }.join("\n")}
        ---

        ## [#{title}][serious-eats]

        [serious-eats]: #{url}

        #{about.map {|k,v| "- #{k}: #{v}" }.join("\n")}

        ### Ingredients

        #{ingredients.map {|i| "- #{i}" }.join("\n")}

        ### Directions

        #{directions.map {|s| "1. #{s}" }.join("\n\n")}
      EOF
    end
  end
end
