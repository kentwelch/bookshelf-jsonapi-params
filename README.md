# bookshelf-jsonapi-params [![Build Status](https://travis-ci.org/scoutforpets/bookshelf-jsonapi-params.svg?branch=master)](https://travis-ci.org/scoutforpets/bookshelf-jsonapi-params)

The [JSON API spec](https://jsonapi.org/format) defines standard parameters to be used when refining result sets via filtering, sparse fieldsets, paging, etc. This [Bookshelf.js](https://github.com/tgriesser/bookshelf) plugin adds a method to your models that can be called to automatically refine the results of your queries based on the aforementioned parameters.

## Installation

Install the package via `npm`:

```sh
$ npm i --save bookshelf-jsonapi-params
```

## Usage

Require and register the `bookshelf-jsonapi-params` plugin:

```js
const bookshelf = require('bookshelf')(knex);
const jsonApiParams = require('bookshelf-jsonapi-params');

bookshelf.plugin(jsonApiParams);
```

After the plugin has been registered, the `fetchJsonApi` method can be called on your model or collection. Here's an example:

```js

// Please see the API for more information
// on available options
const options = {
    fields: {
        person: ['firstName', 'lastName']
    },
    page: {
        limit: 10
    },
    sort: ['-lastName']
};

// Returns a list of people
Person
    .forge()
    .fetchJsonApi(options)
    .then((result) => {
        return result;
    });
```

The above example would yield something like the following when transformed to JSON API:

```js
"data": [
    {
      "type": "person",
      "id": "1",
      "attributes": {
        "firstName": "Cookie",
        "lastName": "Monster"
    },
    {
      "type": "person",
      "id": "1",
      "attributes": {
        "firstName": "Baby",
        "lastName": "Bop"
    }
      ...
      [8 more results]
]
```

Note that only the `firstName` and `lastName` attributes are returned, the results are ordered by `lastName` descending, and only 10 results were returned.

## API

`fetchJsonApi(options)` - calling `fetchJsonApi` with no options is just like a plain call to `Bookshelf.fetch()` or `Bookshelf.fetchAll()` and isn't useful. Please specify `options`.

Option    | Description
:------------- | :-------------
filter _object_  | Filters a result set based specific field. Example: `/pets?filter[name]=max` would only return pets named max.
fields _object_   | Limits the fields returned as part of the record. Example: `/pets?fields[pets]=name` would return pet records with only the name field rather than every field.
include _array_  | Returns relationships as part of the payload. Example: `/pets?include=owner` would return the pet record in addition to the full record of its owner.
page _object_  | Paginates the result set. Example: `/pets?page[limit]=25&page[offset]=0` would return the first 25 records.
sort _array_     | Sorts the result set by specific fields. Example: `/pets?sort=-weight,birthDate` would return the records sorted by `weight` descending, then `birthDate` ascending

See the **[specific section of the JSON API spec](http://jsonapi.org/format/#fetching-includes)** that deals with these parameters for more information.

### Pagination and Sorting
Under the hood, this plugin uses the excellent [bookshelf-page](https://github.com/anyong/bookshelf-page) plugin. That said, please see the available options that can be passed in via the `page` parameter.

#### Default Pagination Parameters
If you'd like your result sets to be paginated by default without having to add pagination options to each call, you can set the default pagination parameters when registering the plugin:

```js
bookshelf.plugin(jsonApiParams, {
    pagination: { limit: 25 }
});
```

_Note:_ pagination options passed into the `fetchJsonApi` will override the defaults.

### Parsing URL Parameters
You can obviously parse all of the URL parameters yourself, but I would highly recommend using the fantastic [node-jsonapi-query-parser](https://github.com/kideh88/node-jsonapi-query-parser) module to handle this for you. The object produced by this module can be passed directly to this plugin without modification.

### Transforming Bookshelf Models to JSON API
Once you have your Bookshelf model, how do you transform it to a JSON API payload? I'd recommend checking out our [jsonapi-mapper](https://github.com/scoutforpets/jsonapi-mapper) module, which will allow you automatically transform your Bookshelf model to a JSON API-compliant payload.

## License

MIT