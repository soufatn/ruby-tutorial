Using Sauce Labs with Ruby
============

Sauce Labs provides over 130 platforms to run Selenium tests against, including Linux, OSX, Windows (XP, 7, 8), iOS and Android.  The Sauce Gem makes it easy to take your existing tests, or write new ones, and run them with Sauce.

Once you're set up, you'll write tests like this:

```ruby
Sauce.config do |c|
  c[:browsers] = [
    ["Windows 7", "Internet Explorer", "9"],
    ["Mac", "Firefox", "17"]
  ]
end

describe "Wikipedia's Ramen Page" do
  it "Should mention the inventor of instant Ramen" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Ramen"
    click_button "searchButton"
    page.should have_content "Momofuku Ando"
  end 
end

describe "Wikipedia's Miso Page" do
  it "Should mention a favorite type of Miso" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Miso"
    click_button "searchButton"
    page.should have_content "Akamiso"
  end 
end

```

And get an integration test that runs each example group in parallel, 
first against IE 9 on Windows 7, then Firefox 17 on OSX.  Your tests 
are run in real browsers on a real operating system, running on a 
freshly baked VM.  Once they're complete, screenshots, video, Selenium 
log and a log of passes and failures can be seen and shared.

This example uses [Capybara](http://jnicklas.github.com/capybara/) ~> 2.0.3 and RSpec with Rails 3.2.x and Ruby 1.9.3, but Sauce Labs also works great against any Ruby web stack, and with [Test::Unit](https://saucelabs.com/docs/ondemand/getting-started/env/ruby/se2/mac), [Cucumber](https://github.com/sauce-labs/sauce_ruby/wiki/Cucumber-and-Capybara), and most other testing frameworks... right down to vanilla [WebDriver](http://code.google.com/p/selenium/wiki/RubyBindings).

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
  gem 'sauce', '~> 2.4.0'
  gem 'sauce-connect', '~> 2.3.0'
  gem 'capybara', '~> 2.0.3'
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
  c[:browsers] = [
    ["Windows 7", "Internet Explorer", "9"],
    ["Mac", "Firefox", "17"]
  ]
end
```

Check out [our platforms page](http://saucelabs.com/docs/browsers) and pick which ones you'd like to test against.

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

Writing your tests
-----------------

Phew!  That's all your setup done.  You're ready to write your tests.

Any tests in spec/selenium will get run against multiple browsers, but
because we want the Capybara DSL included, we're going to put our tests in
the spec/features directory.  We can still turn on the Sauce voodoo by
tagging our example groups with `ruby :sauce => true`, like this:

    mkdir ./spec/requests
    vim ./spec/requests/ramen_spec.rb

```ruby
require "spec_helper"

describe "Wikipedia's Ramen Page", :sauce => true do
  it "Should mention the inventor of instant Ramen" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Ramen"
    click_button "searchButton"
    page.should have_content "Momofuku Ando"
  end 
end
```
That's one spec, how about another?

    vim ./spec/requests/miso_spec.rb

```ruby
require "spec_helper"

describe "Wikipedia's Miso Page", :sauce => true do
  it "Should mention a favorite type of Miso" do
    visit "http://en.wikipedia.org/"
    fill_in 'search', :with => "Miso"
    click_button "searchButton"
    page.should have_content "Akamiso"
  end 
end
```

And that's everything!

Running your tests
------------------

`rake parallel:spec[2]`.

It's that simple (Thanks in part to the [parallel_tests](https://github.com/grosser/parallel_tests) gem)

You should get the following output:

    Finished in 1 minute 18.14 seconds
    1 example, 0 failures

    Randomized with seed 40484

    .

    Finished in 1 minute 19.89 seconds
    1 example, 0 failures

    Randomized with seed 56147


    2 examples, 0 failures

The `2 examples, 0 failures` line means the tests are passing, congratulations!

Your tests ran in two groups simultaneously, each test running on each browser in turn.  Running in parallel makes your build faster, running across multiple browsers makes your product reach a wider audience.

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
has useful info.

<!-- SAUCE:INCLUDE:get-support -->
