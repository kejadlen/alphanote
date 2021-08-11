require "pry"

def suppress_warnings
  original_verbosity = $VERBOSE
  $VERBOSE = nil
  yield
  $VERBOSE = original_verbosity
end

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
    desc "Download ChefSteps recipe"
    task :chef_steps, [:url] do |t, args|
      url = args.fetch(:url, `pbpaste`)
      puts ChefSteps.download(url)
    end

    desc "Download Serious Eats recipe"
    task :serious_eats, [:url] do |t, args|
      url = args.fetch(:url, `pbpaste`)

      require "capybara"
      require "selenium/webdriver"

      Selenium::WebDriver::Firefox::Binary.path = "/Applications/Firefox\ Developer\ Edition.app/Contents/MacOS/firefox"
      Capybara.default_driver = :selenium

      require "capybara/dsl"
      suppress_warnings do
        include Capybara::DSL
      end

      suppress_warnings do
        visit url
      end

      # about = find(:xpath, "//ul[contains(@class, 'recipe-about')]")
      # active_time = about.find(:xpath, ".//span[text()='Active time:']/following-sibling::span").text
      # total_time = about.find(:xpath, ".//span[text()='Total time:']/following-sibling::span").text
      active_time = find(:xpath, ".//span[text()='Active: ']/following-sibling::span").text
      total_time = find(:xpath, ".//span[text()='Total: ']/following-sibling::span").text

      json = find(:xpath, "/html/head/script[@type='application/ld+json']", visible: false)['innerHTML']
      recipe = JSON.parse(json)
      puts Recipe.new(
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
      ).to_s
    end

    desc "Download New York Times recipe"
    task :nyt, [:url] do |t, args|
      url = args.fetch(:url, `pbpaste`)
      puts NYT.download(url)
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
          highlight&.gsub!(/â\u0080\u0099d/, ?')
          note = node.css("div.noteContainer__noteText").text

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

ChefSteps = Struct.new(:url, :raw) do
  Step = Struct.new(:title, :directions)

  def tags
    raw.fetch("tagList")
  end

  def title
    raw.fetch("title")
  end

  def equipment
    raw.fetch("equipment").map {|e| e.dig("equipment", "title") }
  end

  def timing
    raw.fetch("timing")
  end

  def _yield
    raw.fetch("yield")
  end

  def ingredients
    raw.fetch("ingredients").map {|i|
      title, quantity, unit, note = i.fetch_values(*%w[ title quantity unit note ])
      quantity = unit == "ea" ? quantity.to_i : quantity.to_f
      out = ""
      out << quantity.to_s << " " unless quantity.zero?
      out << title
      out << ", " << note unless note.empty?
      out << ", as needed" if unit == "a/n"
      out
    }
  end

  def steps
    raw.fetch("steps")
      .reject {|s| s.fetch("isAside") || s.fetch("hideNumber") }
      .map {|s| Step.new(*s.fetch_values("title", "directions")) }
  end

  def to_s
    require "erb"
    ERB.new(<<~EOF, trim_mode: "<>").result(binding)
      ---
      tags:
      <% tags.each do |tag| %>
        - <%= tag %>
      <% end %>
      ---

      ## [<%= title %>][chef-steps]

      [chef-steps]: <%= url %>

      - Equipment:
      <% equipment.each do |equipment| %>
        - <%= equipment %>
      <% end %>
      - Timing: <%= timing %>
      - Yield: <%= _yield %>

      ### Ingredients

      <% ingredients.each do |ingredient| %>
      - <%= ingredient %>
      <% end %>

      ### Directions

      <% steps.each do |step| %>
      1. <%= step.title %>

      <%= step.directions.split("\n\n").map {|d| "  \#{d}" }.join("\n\n") %>


      <% end %>
    EOF
  end

  def self.download(url)
    require "json"
    require "open-uri"
    require "uri"

    uri = URI(url)
    uri.path = File.join("/api/v0", uri.path)
    json = JSON.parse(uri.open.read)
    new(url, json)
  end
end

class JSON_LD
  def self.download(url)
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

    Recipe.new(
      url,
      recipe.fetch("name"),
      recipe.fetch("keywords").split(/,\s*/).map {|s| s.gsub(/\W+/, ?-) },
      {
        "Yield" => recipe.fetch("recipeYield"),
        "Active time" => active_time(doc),
        "Total time" => total_time(doc),
      },
      recipe.fetch("recipeIngredient"),
      recipe.fetch("recipeInstructions").map {|instruction| instruction.fetch("text") },
    )
  end
end

Recipe = Struct.new(:url, :title, :tags, :about, :ingredients, :directions) do
  def self.from(url)
    recipe = if block_given?
               yield
             else
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
               linked_data.find {|json| json.fetch("@type") == "Recipe" }
             end

    new(
      url,
      recipe.fetch("name"),
      recipe.fetch("keywords").split(/,\s*/).map {|s| s.gsub(/\W+/, ?-) },
      {
        "Yield" => recipe.fetch("recipeYield"),
        "Active time" => active_time(doc),
        "Total time" => total_time(doc),
      },
      recipe.fetch("recipeIngredient"),
      recipe.fetch("recipeInstructions").map {|instruction| instruction.fetch("text") },
    )
  end

  def to_s
    <<~EOF
      ---
      tags:
      #{%w[recipe].concat(tags).map {|t| "  - #{t}" }.join("\n")}
      ---

      ## [#{title}][source]

      [source]: #{url}

      #{about.map {|k,v| "- #{k}: #{v}" }.join("\n")}

      ### Ingredients

      #{ingredients.map {|i| "- #{i}" }.join("\n")}

      ### Directions

      #{directions.map {|s| "1. #{s}" }.join("\n\n")}
    EOF
  end
end

class NYT < JSON_LD
  def self.active_time(doc)
    "N/A"
  end

  def self.total_time(doc)
    doc.at_xpath("//span[@class='recipe-yield-value']").inner_text
  end
end
