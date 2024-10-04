# Starting a new Rails application

## 1. Generate the project

If needed, check the Rails command line [documentation](https://guides.rubyonrails.org/command_line.html). 
```sh 
rails new my_app --database=postgresql --skip-test --skip-jbuider --css tailwind 
```

## 2. Setting up RSpec

Instead of using default Rails Minitest, we're going with the best of the best!

[Documentation](https://github.com/rspec/rspec-rails/) you know, just in case.

Add the following into Gemfile:

```ruby 
group :development, :test do
  # Integrates the Rails testing helpers into RSpec
  gem 'rspec-rails', '~> 7.0.0'
end
```

Next, in your project directory:

```sh 
$ bundle install
$ rails generate rspec:install
$ bundle binstubs rspec-core
```
## 3. (Optional) Setting up Devise

Ensure these instructions are up to date, reference the [documentation](https://github.com/heartcombo/devise).

Add these to the `Gemfile`:

```ruby
# Flexible authentication solution for Rails
gem 'devise'
```

## 4. (Optional) Setting up Letter Opener

A simple gem to check sent e-mail from your application.

## 5. (Optional) Setting up Pundit


## 6. Tweaking RSpec

Now we just change some configurations in `spec/rails_helper.rb`

Uncomment the line bellow. This tells Rspec to look into specs support directory and load all those files, so it let us create helpers and things like that

```ruby
Rails.root.glob('spec/support/**/*.rb').sort_by(&:to_s).each { |f| require f }
```

The add these to the configuration block:

```ruby
config.global_fixtuers = :all

# Uncomment this line if you'll use ActiveJob
# config.include ActiveJob::TestHelper

# Uncomment this line if you'll use ActionMailbox
# config.include ActionMailbox::TestHelper

# Uncomment the lines bellow AFTER you install Devise, if you need it
# config.include Devise::Test::IntegrationHelpers, type: :feature
# config.include Devise::Test::ControllerHelpers, type: :controller
```

(Optional) This next step we'll create a helper that will put Devise into test mode, when using request_spec, giving us
a method to sign users in and out.

Create this file in `spec/support/request_spec_helper.rb`:

```ruby
module RequestSpecHelper
  include Warden::Test::Helpers

  def self.included(base)
    base.before(:each) { Warden.test_mode! }
    base.after(:each) { Warden.test_reset! }
  end

  def sign_in(resource)
    login_as(resource, scope: warden_scope(resource))
    resource
  end

  def sign_out(resource)
    logout(warden_scope(resource))
  end

  def json
    JSON.parse(response.body).with_indifferent_access
  end

  private

  def warden_scope(resource)
    resource.class.name.underscore.to_sym
  end
end

RSpec.configure do |config|
  config.include RequestSpecHelper, type: :request
  config.before(:each, type: :request) { host! "my_app.test" }
end
```

## 7. Tweaking Rails configs

Now let's remove some bloat that Rails generators create for us, and add Gzip.

Add the following to `config/application.rb`:

```ruby 
config.generators do |g|
  g.skip_routes true
  g.helpers false
  g.test_famework :rspec, fixture: false
  g.helper_specs false
  g.contoller_specs false
  g.system_tests false
  g.view_specs false
end

# Gzip all responses
config.middleware.use Rack::Deflater
```
