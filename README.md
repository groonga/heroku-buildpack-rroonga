# Heroku buildpack: Rroonga

This is a Heroku buildpack of [Rroonga](http://ranguba.org/#about-rroonga).

## Usage:

    heroku create --buildpack https://codon-buildpacks.s3.amazonaws.com/buildpacks/groonga/rroonga.tgz

Add `rroonga` entry to your `Gemfile`:

    gem "rroonga"

Create `groonga/init.rb` that initializes your Groonga database. You can refer your Groonga database path by `ENV["GROONGA_DATABASE_PATH"]`.

Here is a sample `groonga/init.rb`:

```ruby
require "groonga"

Groonga::Database.open(ENV["GROONGA_DATABASE_PATH"])

# Define schema
Groonga::Schema.define do |schema|
  schema.create_table("Sites",
                      :type => :hash,
                      :key_type => :short_text) do |table|
    table.short_text("title")
    table.text("description")
  end
end

# Add data
sites = Groonga["Sites"]
sites.add("http://www.ruby-lang.org/",
          :title => "Ruby Programming Language",
          :description => "The official Web site of Ruby.")
sites.add("http://groonga.org/",
          :title => "Groonga - An open-source fulltext search engine and column store",
          :description => "The official Web site of Groonga.")

# Create indexes. We can use offline index construction by creating indexes
# after we add data. Offline index construction is 10 times faster rather
# than online index construction.
#
# See also:
#   * Online index construction: http://groonga.org/docs/reference/indexing.html#online-index-construction
#   * Offline index construction: http://groonga.org/docs/reference/indexing.html#offline-index-construction
Groonga::Schema.define do |schema|
  schema.create_table("Terms",
                      :type => :patricia_trie,
                      :key_type => :short_text,
                      :normalizer => "NormalizerAuto",
                      :default_tokenizer => "TokenBigram") do |table|
    table.index("Sites.title")
    table.index("Sites.description")
  end
end
```

Then push them to Heroku.

    git push heroku master
