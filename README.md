# Archives of Nethys Search Engine

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


## License 📗

[Apache-2.0](https://github.com/elastic/app-search-reference-ui-react/blob/master/LICENSE.md) © [Elastic](https://github.com/elastic)

