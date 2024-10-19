<a id="readme-top"></a>

# Rails 7 default configuration

This is my guide on how to start a Rails application the way I like and usually use.

Remember to add and commit every step!

## 1. Generate the project

If needed, check the Rails command line [documentation](https://guides.rubyonrails.org/command_line.html).

```sh
rails new my_app --database=postgresql --skip-test --skip-jbuider --css tailwind 
```

> COMMIT

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 2. Setting up RSpec

Instead of using default Rails Minitest (which we removed with `--skip-test`), we're going with the best of the best (or because it's the one I learned in...)!

[Documentation](https://github.com/rspec/rspec-rails/) you know, just in case.

Add the following into `Gemfile`'s group `:development, :test`:

```ruby 
# Integrates the Rails testing helpers into RSpec [https://github.com/rspec/rspec-rails]
gem 'rspec-rails'
```

Next, in your project directory:

```sh 
bundle install
rails generate rspec:install
bundle binstubs rspec-core
# Now we can run our tests with bin/rspec
```

### 2.1. Tweaking RSpec

First we'll add into `.rspec` the line bellow, because we're not masochists:

```sh
--format documentation
```

Now we just change some configurations in `spec/rails_helper.rb`

Uncomment the line bellow. This tells Rspec to look into specs support directory and load all those files, so it let us create helpers and things like that

```ruby
Rails.root.glob('spec/support/**/*.rb').sort_by(&:to_s).each { |f| require f }
```

The add these to the configuration block:

```ruby
config.global_fixtuers = :all

# Comment this line if you won't use ActiveJob
config.include ActiveJob::TestHelper

# Comment this line if you won't use ActionMailbox
config.include ActionMailbox::TestHelper

# Comment the lines bellow if you won't use Devise
config.include Devise::Test::IntegrationHelpers, type: :feature
config.include Devise::Test::ControllerHelpers, type: :controller
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

> COMMIT

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 3. (Optional) Factory Bot

Ensure these instructions are up to date, reference the [documentation](https://github.com/thoughtbot/factory_bot_rails).

Add to `Gemfile`, inside `:development, :test`:

```ruby
# Use factory_bot_rails as a fixtures replacement with a straightforward definition syntax. [https://github.com/thoughtbot/factory_bot_rails]
gem 'factory_bot_rails'
```

Run on terminal:

```sh
bundle install
```

Now, let's configure our test suite to include factory_bot_rails methods. Add to `spec/rails_helper.rb`

```ruby
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 4. (Optional) Setting up Devise

Ensure these instructions are up to date, reference the [documentation](https://github.com/heartcombo/devise).

Add these to the `Gemfile`: 

```ruby
# Flexible authentication solution for Rails [https://github.com/heartcombo/devise]
gem 'devise'
```

Run the thingy to install it:
```sh 
bundle install
```

Add these to `app/controllers/application_controller.rb` to protect all routes by default and allow some extra params:

```ruby
before_action :configure_permitted_parameters, if: :devise_controller?
before_action :authenticate_user!

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.permit(:sign_up, keys: %i[first_name last_name terms_and_conditions])
  devise_parameter_sanitizer.permit(:account_update, keys: %i[first_name last_name])
end
```

Next, we'll install Devise and generate a `User` model:

```sh 
rails generate devise:install
```

Devise will mix some letters into some steps you should do, just follow it.

After following what's written in Devices terminal, run:

```sh 
rails generate devise User
```

We'll go to the generated `User` migration and add two additional fields for `first_name` and `last_name`. At this point we can decide what Devise modules you want to enable.

```ruby
t.string :first_name,       null: false, default: ""
t.string :last_name,        null: false, default: ""
```

### 4.1 Tweaking some Devise configs

Here are some configurations that I like to change. They are all at `config/initializers/devise.rb`:

```ruby
# If you already have the domain, just it it already man...
config.mailer_sender = 'please-change-me-at-config-initializers-devise@example.com'

# Uncomment the line bellow. No one deserves to keep loging again and again
# config.remember_for = 2.weeks

# Uncomment these lines to avoid people trying to brute force an account
config.lock_strategy = :failed_attempts

config.unlock_keys = [:email]

config.unlock_strategy = :both

config.maximum_attempts = 20

config.unlock_in = 1.hour

config.last_attempt_warning = true
```

If you want some locales for devise, here are some [devise locales](https://github.com/heartcombo/devise/wiki/I18n). Just create a `config/devise.your_language.yml` it.

> COMMIT

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 5. (Optional) Setting up Letter Opener

A simple gem to check sent e-mail from your application. [Documentation](https://github.com/fgrehm/letter_opener_web), why not?

Add this to `Gemfile`'s group `:development`:

```ruby
  # Use letter_opener_web to preview email in the default browser instead of sending it [https://github.com/fgrehm/letter_opener_web]
  gem 'letter_opener_web'
```

Add this to your `routes.rb`:

```ruby 
mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
```

Then set the delivery method in `config/environments/development.rb`

```ruby
config.action_mailer.delivery_method = :letter_opener

config.action_mailer.perform_deliveries = true
```
> Guess what?

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## 6. (Optional) Setting up Pundit

Pundit provides us a set of helpers which guide you in leveraging User/Admin policies. As always, [Docs](https://github.com/varvet/pundit).

Add to `Gemfile`:

```ruby
# Object oriented authorization for Rails applications [https://github.com/varvet/pundit]
gem 'pundit'
```

Run generator:

```sh 
rails g pundit:install
```

> Come on. You know what you should be doing here.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

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

I’m a fan of Rails credentials, even with the [12 plumbus](https://12factor.net/), but I do not like the idea of committing it.
So let's ignore all `.key` and `.yml.enc` files withing `config/`:

```sh
# Ignore master key for decrypting credentials and more.
/config/**/*.key

#Ignore any credentials inside config folder
/config/**/*.yml.enc
```

> Commit to not commit!

