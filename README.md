# AfasRest

This 'wrapper gem' hooks a Ruby app (currently only tested with Rails) up to the [Afas](http://www.afas.com/) REST API ([docs](https://docs.afas.com/esign/), [API explorer](https://apiexplorer.afas.com/#/esign/restapi)) to allow for embedded signing.

## Installation

Add this line to your application's Gemfile:

    gem 'afas_rest'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install afas_rest

## Configuration

There is a bundled rake task that will prompt you for your Afas credentials including:

  * Username
  * Password
  * Integrator Key

and create the `config/initializers/afas_rest.rb` file in your Rails app for you. If the file was unable to be created, the rake task will output the config block for you to manually add to an initializer.

**Note** please run the below task and ensure your initializer is in place before attempting to use any of the methods in this gem. Without the initializer this gem will not be able to properly authenticate you to the Afas REST API.

    $ bundle exec rake afas_rest:generate_config

outputs:

    Please do the following:
    ------------------------
    1) Login or register for an account at https://demo.afas.net
         ...or their production url if applicable
    2) From the Avatar menu in the upper right hand corner of the page, click "Go to Admin"
    3) From the left sidebar menu, click "API and Keys"
    4) Request a new 'Integrator Key' via the web interface
        * You will use this key in one of the next steps to retrieve your 'accountId'

    Please enter your Afas username: someone@gmail.com
    Please enter your Afas password: p@ssw0rd1
    Please enter your Afas integrator_key: KEYS-19ddd1cc-cb56-4ca6-87ec-38db47d14b32

    The following block of code was added to config/initializers/afas_rest.rb

    require 'afas_rest'

    AfasRest.configure do |config|
      config.profit_username = 'someone@gmail.com'
      config.password        = 'p@ssw0rd1'
      config.api_key         = 'KEYS-19ddd1cc-cb56-4ca6-87ec-38db47d14b32'
      config.environment_key = '123456'
      #config.endpoint       = 'https://www.afas.net/restapi'
      #config.api_version    = 'v2'
    end


### Config Options

There are several other configuration options available but the two most likely to be needed are:

```ruby
config.endpoint       = 'https://afas.net/restapi'
config.api_version    = 'v2'
```

The above options allow you to change the endpoint (to be able to hit the production Afas API, for instance) and to modify the API version you wish to use.

## Usage

The afas\_rest gem makes creating multipart POST (aka file upload) requests to the Afas REST API dead simple. It's built on top of `Net::HTTP` and utilizes the [multipart-post](https://github.com/nicksieger/multipart-post) gem to assist with formatting the multipart requests. The Afas REST API requires that all files be embedded as JSON directly in the request body (not the body\_stream like multipart-post does by default) so the afas\_rest gem takes care of [setting that up for you](https://github.com/lafeber/afas_rest/blob/master/lib/afas_rest/client.rb#L397).

### Examples

* Unless noted otherwise, these requests return the JSON parsed body of the response so you can index the returned hash directly. For example: `template_response["templateId"]`.

#### Situations

**In the context of a Rails app**

This is how most people are using this gem - they've got a Rails app that's doing things with the Docusign API.  In that case, these examples assume you have already set up a afas account, have run the `afas_rest:generate_config` rake task, and have the configure block properly setup in an initializer with your username, password, integrator\_key, and account\_id.

**In the context of this gem as a standalone project**

Ideally this gem will be independent of Rails.  If that's your situation, there won't be a Rails initializer so your code will need to load the API authentication credentials.  You will want to do something like:

```ruby
load 'test/afas_login_config.rb'
client = AfasRest::Client.new
client.get_account_id
document_envelope_response = client.create_envelope_from_document( # etc etc
```
