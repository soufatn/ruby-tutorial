Using Sauce Labs with Ruby
============

Once you're set up, you'll write tests like this:

```ruby
Sauce.config do |c|
  c[:browsers] = [["Windows 7", "Internet Explorer", "9"]]
end

describe "Sauce Labs Browser Documentation" do
  it "Displays Ruby code only when Ruby selected" do
    visit "https://saucelabs.com/docs/browsers"
    select('ruby', :from => 'lang-chooser')
    caps = page.find(:xpath, "//p[contains(., 'caps = Selenium::WebDriver::Remote::Capabilities')]")
    caps.visible?.should be_true
  end
end
```

And get an integration test in IE9 on Windows 7 with screenshots, video and a log of passes and failures.

This example uses [Capybara](http://jnicklas.github.com/capybara/) and RSpec with Rails 3.2.x and Ruby 1.9.3, but Sauce Labs also works great against any Ruby web stack, and with [Test::Unit](https://saucelabs.com/docs/ondemand/getting-started/env/ruby/se2/mac), [Cucumber](https://github.com/sauce-labs/sauce_ruby/wiki/Cucumber-and-Capybara), and most other testing frameworks... right down to vanilla [WebDriver](http://code.google.com/p/selenium/wiki/RubyBindings).

We're working on making this tutorial as clear, simple, and relevant
as possible. If you run into any problems, or have questions or
suggestions, please don't hesitate to email help@saucelabs.com!

What You'll Need
----------------

In your Gemfile:

```ruby
group :test, :development do
  # These are the target gems of this tutorial
  gem 'rspec-rails', '~> 2.12'
  gem 'sauce', '~> 2.3'
  gem 'capybara', '~> 1.0'
  gem 'parallel_tests'
end
```

Setting up RSpec
-----------

From your `$RAILS_ROOT`, generate a ./spec directory, a ./spec/spec_helper.rb file, and a warm, fuzzy feeling of productivity by executing:

    rails generate rspec:install

Inside the newly created spec/spec_helper.rb, just under the other `require` statements, we'll add Capybara and the Sauce gem, and tell Capybara to use Sauce Labs for all tests (by default, it's only used for tests marked :js => true):

```ruby
require 'capybara/rails'
require 'capybara/rspec'
require 'sauce/capybara'

Capybara.default_driver = :sauce
```

Next we can add the following block to configure which browsers we want to use:

```ruby
Sauce.config do |c|
  c[:browsers] = [["Windows 7", "Internet Explorer", "9"]]
end
```

Check out [this list of device/OS/browser platforms](http://saucelabs.com/docs/browsers) and pick which ones you'd like to test against.

Setting up the Sauce Gem
-------------------------

<!-- SAUCE:LOGIN -->

Keep your Sauce Labs credentials out of your repositories and available to all your Sauce Labs tools using projects by adding them as environment variables.

<!-- SAUCE:BEGIN_PLATFORM:MAC|LINUX -->

Open `~/.bash_profile` and add the following lines:

```bash
export SAUCE_USERNAME=<!-- SAUCE:USERNAME -->
export SAUCE_ACCESS_KEY=<!-- SAUCE:ACCESS_KEY -->
```

You'll then need to re-load that profile with `source ~/.bash_profile`
<!-- SAUCE:END_PLATFORM -->
<!-- SAUCE:BEGIN_PLATFORM:WIN -->
Open your environment variables settings window (Instructions [here](http://www.itechtalk.com/thread3595.html)) and set the following variables:

    Name: SAUCE_USERNAME
    Value: <!-- SAUCE:USERNAME -->

    Name: SAUCE_ACCESS_KEY
    Value:  <!-- SAUCE:ACCESS_KEY -->
<!-- SAUCE:END_PLATFORM -->

Writing your test
-----------------

Phew!  That's all your setup done.  You're ready to write your tests.

We're going to put our test in the spec/requests directory so that rspec includes the Capybara DSL:

    mkdir ./spec/requests
    vim ./spec/requests/browser_docs_spec.rb

```ruby
require "spec_helper"

describe "Sauce Labs Browser Documentation" do
  it "Displays Ruby code only when Ruby selected" do
    visit "https://saucelabs.com/docs/browsers"
    select('ruby', :from => 'lang-chooser')
    caps = page.find(:xpath, "//p[contains(., 'caps = Selenium::WebDriver::Remote::Capabilities')]")
    caps.visible?.should be_true
  end
end
```

And that's everything!  

You can run your tests as usual with `rake spec:requests`, but to speed things up, we highly recommend parallelizing your tests with [parallel_tests](https://github.com/grosser/parallel_tests). By running tests in parallel on Sauce, you can do builds in a fraction of the time. To run in parallel, just use `rake parallel:spec`.

Either way, you should get the following output:

    $ rake parallel:spec
    .

    Finished in 24.31 seconds
    1 example, 0 failures

    Randomized with seed 6006

The `1 example, 0 failures` line means the test is passing, congratulations!

Check out the results, including a command log, screenshots, and video of the browser executing the test, on your [account page](https://saucelabs.com/account).

What's Next?
------------
**Capybara Resources**

Now that you have an example to work with, it's time to write a test for your web app! For more info on how to write Capybara tests, we recommend the excellent [Capybara README](https://github.com/jnicklas/capybara).

**Tunnel to your local machine with Sauce Connect**

If you need to test a staged site behind your firewall, that's no problem: check out [Sauce Connect](http://saucelabs.com/docs/connect).

To use Sauce Connect with the Sauce gem, simply add it to your Gemfile:

```ruby
gem 'sauce-connect'
```

then enable it in your Sauce.config block:

```ruby
Sauce.config do |config|
  start_tunnel_for_parallel_tests(c)
  # or, if you're not using parallel_tests, do this instead:
  # config[:start_tunnel] = true
end
```

**parallel_tests considerations**

If you find that you run into problems with all your tests using the same db at once, or otherwise need some careful setup, the 
[parallel_tests instructions](https://github.com/grosser/parallel_tests)
have useful info.

<!-- SAUCE:INCLUDE:get-support -->
