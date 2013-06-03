Using Sauce Labs with Ruby
============

The sauce gem makes it easy to take new or existing Selenium or Capybara tests and run them against a wide range of browsers on Windows (XP, 7, 8), OS X, Linux, iOS and Android. 

This example uses [Capybara](http://jnicklas.github.com/capybara/) and RSpec with Rails 3 and Ruby 1.9, but Sauce Labs also works great against any Ruby web stack, and with [Test::Unit](https://saucelabs.com/docs/ondemand/getting-started/env/ruby/se2/mac), [Cucumber](https://github.com/sauce-labs/sauce_ruby/wiki/Cucumber-and-Capybara), and most other testing frameworks... right down to vanilla [WebDriver](http://code.google.com/p/selenium/wiki/RubyBindings).

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

Your tests are run in real browsers on a real operating system, in a dedicated, single-use VM.
Once they're complete, screenshots, video, Selenium log and a log of
passes and failures can be seen and shared.

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
  gem 'sauce', '~> 3.0.0.beta.1'
  gem 'capybara', '~> 2.0.3'
  gem 'parallel_tests'
end
```

Setting up RSpec
-----------

From your `$RAILS_ROOT`, generate a ./spec directory, a ./spec/spec_helper.rb file, and a warm, fuzzy feeling of productivity by executing:

    $ rails generate rspec:install
    
Set up your spec_helper and create a template sauce_helper, by running:
    $ rake sauce:install

Now, open the new spec/spec_helper.rb file, add a `require "sauce/capybara"` and configure your desired test platforms:

```ruby
require "sauce"
require "sauce/capybara"

Sauce.config do |c|
  c[:browsers] = [ 
    ["Windows", "Firefox", "18"],
    ["Linux", "Chrome", nil],
    ["Mac", "Firefox", "19"],
    ["Mac", "Firefox", "17"]
  ]
end
```

Check out [our platforms page](http://saucelabs.com/docs/platforms) for available platforms (130+ and counting!).

Inside the newly created spec/spec_helper.rb, just under the other `require` statements, we'll add requires for capybara/rails and capybara/rspec, and tell Capybara to use Sauce Labs for all tests (by default, it's only used for tests marked :js => true):

```ruby
require 'capybara/rails'
require 'capybara/rspec'

Capybara.default_driver = :sauce
```

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

Because we want the Capybara DSL included, we're going to put our tests in
the spec/features directory.  We can still turn on the Sauce voodoo by
tagging each describe block ('example group' in RSpec-lish)  with `:sauce => true`, like this:

    $ mkdir ./spec/features
    $ vim ./spec/features/ramen_spec.rb

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

    $ vim ./spec/features/miso_spec.rb

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

`$ rake sauce:spec`

It's that simple (Thanks in part to the excellent [parallel_tests](https://github.com/grosser/parallel_tests) gem.)
Your tests will run once for every platform, taking advantage of Sauce Labs to run as concurrently as possible.

You should output much like the following:

```
20 processes for 8 specs, ~ 0 specs per process
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
[Connecting to Sauce Labs...]
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38
Sauce Connect 3.0-r25, build 38

[snip]A LOT OF TEST DATA[/snip]

8 examples, 0 failures

Took 196.657036 seconds
```

The `8 examples, 0 failures` lines means your tests are running against each browser, and passing, congratulations!

Running in parallel makes your build faster, and doing it with multiple browsers helps you reach a wider audience.

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

When using Capybara and Sauce Connect, you need to specify the port Capybara spins up on.  Pick your favorite port:

```bash
80, 443, 888, 2000, 2001, 2020, 2222, 3000, 3001, 3030, 3333, 4000, 4001, 4040, 4502, 4503, 5000, 5001, 5050, 5555, 6000, 6001, 6060, 6666, 7000, 7070, 7777, 8000, 8001, 8003, 8031, 8080, 8081, 8888, 9000, 9001, 9080, 9090, 9999, 49221
```
And set the server port in spec_helper.rb:

```ruby
Capybara.server_port = your_chosen_port
```

**parallel_tests considerations**

If you find that you run into problems with all your tests using the same db at once, or otherwise need some careful setup, the 
[parallel_tests instructions](https://github.com/grosser/parallel_tests)
has useful info.

<!-- SAUCE:INCLUDE:get-support -->
