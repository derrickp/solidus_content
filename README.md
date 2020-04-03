SolidusContent
==============

Introduction goes here.

Installation
------------

Add solidus_content to your Gemfile:

```ruby
gem 'solidus_content'
```

Bundle your dependencies and run the installation generator:

```shell
bundle
bundle exec rails g solidus_content:install
```

Usage
-----

```rb
home_entry_type = SolidusContent::EntryType.create!(
  name: :home,
  content_provider_name: :json,
  options: { path: 'app/content/home' }
)

home = SolidusContent.create!(
  content_type: home_entry_type,
  slug: :default,
)
```

Inside `app/views/spree/home/index.html.erb`:

```erb
<% data = SolidusContent::Entry.data_for(:home, :default) %>

<h1><%= data[:title] %></h1>
```

Configuration
-------------

Configure SolidusContent in an initializer:

```rb
# config/initializers/solidus_content.rb

SolidusContent.configure do |config|
  # your configuration goes here...
end
```

Available Content Providers
---------------------------

### RAW

This is the most simple provider, its data will come directly from the entry
options.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'raw',
)
entry = SolidusContent::Entry.create(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
  options: {title: "Hello World!", body: "My first post!"}
)
```

### JSON

Will fetch the data from a JSON file within the directory specified by the
`path` entry-type option and with a basename corresponding to the entry `slug`.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'json',
  options: {path: 'data/posts'}
)
entry = SolidusContent::Entry.create(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
)
```

```json
// [RAILS_ROOT]/data/posts/2020-03-27-hello-world.json
{"title": "Hello World!", "body": "My first post!"}
```

_NOTE: Absolute paths are taken as they are and won't be joined to `Rails.root`._

### YAML

Will fetch the data from a YAML file within the directory specified by the
`path` entry-type option and with a basename corresponding to the entry `slug`.

If there isn't a file with the `yml` extension, the `yaml` extension will be tried.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'yaml',
  options: {path: 'data/posts'}
)
entry = SolidusContent::Entry.create(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
)
```

```yaml
# [RAILS_ROOT]/data/posts/2020-03-27-hello-world.yml

title: Hello World!
body: My first post!
```

_NOTE: Absolute paths are taken as they are and won't be joined to `Rails.root`._

### Solidus static content

To retrieve the page we have to pass the page `slug` to the entry options.
If the page slug is the same of the entry one, we can avoid passing the options.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'solidus_static_content'
)

entry = SolidusContent::Entry.create!(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
  options: { slug: 'XXX' } # Can be omitted if the page slug is the same of the entry
)
```

### Contentful

To fetch the data we have to create a connection with Contentful passing the
`contentful_space_id` and the `contentful_access_token` to the entry-type options.

Will fetch the data from Contentful passing the `entry_id` entry option.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'contentful',
  options: {
    contentful_space_id: 'XXX',
    contentful_access_token: 'XXX'
  }
)

entry = SolidusContent::Entry.create!(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
  options: { entry_id: 'XXX' }
)
```

### Prismic

To fetch the data we have to create a connection with Prismic passing the
`api_entry_point` to the entry-type options.

If the repository is private, you have to also pass the `api_token` to the entry-type options.

Will fetch the data from Prismic passing the `id` entry option.

```rb
posts = SolidusContent::EntryType.create(
  name: 'posts',
  content_provider_name: 'prismic',
  options: {
    api_entry_point: 'XXX',
    api_token: 'XXX' # Only if the repository is private
  }
)

entry = SolidusContent::Entry.create!(
  slug: '2020-03-27-hello-world',
  entry_type: posts,
  options: { id: 'XXX' }
)
```

Registering a content provider
==============================

To register a content-provider, add a callable to the configuration under the
name you prefer. The

```rb
SolidusContent.config.content_providers[:json] = ->(input) {
  dir = Rails.root.join(input.dig(:type_options, :path))
  file = dir.join(input[:slug] + '.json')
  data = JSON.parse(file.read, symbolize_names: true)

  input.merge(data: data)
}
```

The `input` passed to the content-provider will have the following keys:

- `slug`: the slug of the content-entry
- `type`: the name of the content-type
- `provider`: the name of the content-provider
- `options`: the entry options
- `type_options`: the content-type options

The `output` of the content-provider is the `input` hash augmented with the
following keys:

- `data`: the content itself
- `provider_client`: (optional) the client of the external service
- `provider_entry`: (optional) the object retrieved from the external service

In both the input and output all keys should be symbolized.

Testing
-------

First bundle your dependencies, then run `rake`. `rake` will default to building the dummy app if it does not exist, then it will run specs, and [Rubocop](https://github.com/bbatsov/rubocop) static code analysis. The dummy app can be regenerated by using `rake test_app`.

```shell
bundle
bin/rake
```

When testing your application's integration with this extension you may use its factories.
Simply add this require statement to your spec_helper:

```ruby
require 'solidus_content/factories'
```

Releasing
---------

Your new extension version can be released using `gem-release` like this:

```shell
bundle exec gem bump -v VERSION --tag --push --remote upstream && gem release
```

Copyright (c) 2020 Nebulab, released under the New BSD License
