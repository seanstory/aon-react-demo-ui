# [Archives of Nethys Search Engine](https://seanstory.github.io/aon-react-demo-ui/)

A configurable, generic search UI for Archives of Nethys
Built with Elastic App Search's Reference UI.

### Updating configuration

The project can be configured via a JSON [config file](src/config/engine.json).

You can easily control things like...

- The Engine the UI runs against
- Which fields are displayed
- The filters that are used

If you would like to make configuration changes, there is no need to regenerate
this app from your App Search Dashboard!

You can simply open up the
[engine.json](src/config/engine.json) file, update the [options](#config),
and then restart this app.

### Configuration options <a id="config"></a>

The following is a complete list of options available for configuration in [engine.json](src/config/engine.json).

| option               | value type    | required/optional | source                                                                                                                                                                                          |
| -------------------- | ------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `engineName`         | String        | required          | Found in your App Search Dashboard.                                                                                                                                                             |
| `endpointBase`       | String        | required*         | (*) Elastic Enterprise Search deployment URL, example: "http://127.0.0.1:3002". Not required if using App Search on swiftype.com.                                                               |
| `hostIdentifier`     | String        | required*         | (*) Only required if using App Search on swiftype.com.                                                                                                                                          |
| `searchKey`          | String        | required          | Found in your App Search Dashboard.                                                                                                                                                             |
| `searchFields`       | Array[String] | required          | A list of fields that will be searched with your search term.                                                                                                                                   |
| `resultFields`       | Array[String] | required          | A list of fields that will be displayed within your results.                                                                                                                                    |
| `querySuggestFields` | Array[String] | optional          | A list of fields that will be searched and displayed as query suggestions.                                                                                                                      |
| `titleField`         | String        | optional          | The field to display as the title in results.                                                                                                                                                   |
| `urlField`           | String        | optional          | A field with a url to use as a link in results.                                                                                                                                                 |
| `sortFields`         | Array[String] | optional          | A list of fields that will be used for sort options.                                                                                                                                            |
| `facets`             | Array[String] | optional          | A list of fields that will be available as "facet" filters. Read more about facets within the [App Search documentation](https://www.elastic.co/guide/en/app-search/current/facets-guide.html). |

## Deploy


```bash
npm run deploy
```

## Where'd the data come from?
Originally, the data came from a scrape of the [Archives of Nethys](https://2e.aonprd.com/) website, for Pathfinder - 2nd Edition.
I scraped it using the [Elastic App Search Crawler](https://www.elastic.co/guide/en/app-search/current/crawl-web-content.html). When I scraped it, the Crawler was still in "beta", so I did a little extra cleanup to get the data formatted exactly as I wanted it.

First, I ran the crawler in local mode, with a config like:

```
max_indexed_links_count: 1000
max_extracted_links_count: 10000
max_url_length: 70
max_crawl_depth: 10
domain_allowlist:
  - https://2e.aonprd.com
seed_urls:
  - https://2e.aonprd.com/Actions.aspx
  - https://2e.aonprd.com/Afflictions.aspx
  - https://2e.aonprd.com/Ancestries.aspx
  - https://2e.aonprd.com/Archetypes.aspx
  - https://2e.aonprd.com/Backgrounds.aspx
  - https://2e.aonprd.com/Classes.aspx
  - https://2e.aonprd.com/Conditions.aspx
  - https://2e.aonprd.com/Creatures.aspx
  - https://2e.aonprd.com/Equipment.aspx?All=true
  - https://2e.aonprd.com/Feats.aspx
  - https://2e.aonprd.com/Hazards.aspx
  - https://2e.aonprd.com/Rules.aspx
  - https://2e.aonprd.com/Skills.aspx
  - https://2e.aonprd.com/SpellLists.aspx?Tradition=0
  - https://2e.aonprd.com/Traits.aspx
  - https://2e.aonprd.com/Rules.aspx?ID=1483
  - https://2e.aonprd.com/Rules.aspx?ID=748
  - https://2e.aonprd.com/Rules.aspx?ID=1587
  - https://2e.aonprd.com/Rules.aspx?ID=686
output_sink: file
output_dir: examples/output/aon
```


Next, I made a new dir:

```
mkdir examples/output/aon-cleaned/
```

Then, I cleaned the raw crawl JSON:

```
require 'json'
require 'nokogiri'

output_dir = 'examples/output/aon-cleaned/'
input_dir = 'examples/output/aon/'
Dir.foreach(input_dir) do |filename|
  next if filename == '.' or filename == '..'
  puts "Parsing #{filename}"
  json = File.read("#{input_dir}#{filename}")
  hsh = JSON.parse(json)
  parsed_data = Nokogiri::HTML.parse(hsh['content'])
  title = parsed_data.title.strip.gsub(' - Archives of Nethys: Pathfinder 2nd Edition Database','')
  categories = title.split(' - ')
  category = categories.size >= 2 ? categories[1] : nil
  sub_category = categories.size >= 3 ? categories[2] :nil
  common_keywords = ["Archives", "Nethys", "Wiki", "Archives of Nethys", "Pathfinder", "Official", "AoN", "AoNPRD", "PRD", "PFSRD", "2E", "2nd Edition"]
  keywords =  parsed_data.at('meta[name=keywords]')['content'].split(', ') - common_keywords
  description = parsed_data.at('meta[name=description]')['content']
  description = Nokogiri::HTML(description)&.text
  body_content = parsed_data.at_css('[id="ctl00_MainContent_DetailedOutput"]')&.text
  body_content = parsed_data.at_css('[id="ctl00_RadDrawer1_Content_MainContent_DetailedOutput"]')&.text unless body_content
  result = {
    :title => title,
    :category => category,
    :sub_category => sub_category,
    :keywords => keywords,
    :description => description,
    :body => body_content,
    :url => hsh['url']
  }
  output_json = JSON.pretty_generate(result)
  File.open("#{output_dir}#{filename}", 'w') { |file| file.write(output_json) }
  puts "Parsed #{filename}"
rescue StandardError => e
  puts "Failed to parse #{filename} because #{e.class}: #{e.message}"
end
```

Then, I used the [App Search Ruby Client](https://www.elastic.co/guide/en/enterprise-search-clients/ruby/current/app-search-api.html) to index the cleaned json

```
require 'elastic/enterprise_search'

host = <host>
key = <private key>

ent_client = Elastic::EnterpriseSearch::Client.new(host: host)
ent_client.app_search.http_auth = key
client = ent_client.app_search
engine_name = 'aon-non-crawl'
documents = []
batch = 1
output_dir = 'examples/output/aon-cleaned/'
Dir.foreach(output_dir) do |filename|
  next if filename == '.' or filename == '..'
  document = JSON.parse(File.read("#{output_dir}#{filename}"))
  document[:id] = filename
  documents << document
  if documents.size == 100
    puts "indexing batch #{batch}"
    client.index_documents(engine_name, documents: documents)
    documents = []
    batch = batch + 1
  end
end
client.index_documents(engine_name, documents: documents)
```

## License 📗

[Apache-2.0](https://github.com/elastic/app-search-reference-ui-react/blob/master/LICENSE.md) © [Elastic](https://github.com/elastic)

