---
layout: post
title: Visual Regression Testing Rails Mailers with Percy.io
---

Recently in work we've started using [Percy.io](https://percy.io/) to catch visual regressions in our UI. It's a great tool - it allows us to catch regressions in CSS or HTML that we would have only caught previously with manual testing. The service costs us $100 a month for 3 developers - compare that to the salary of a dedicated QA and it's a no brainer.

One missing piece of the puzzle which doesn't come out of the box is testing Rails mailers. We have quite a few HTML emails that we send out which are prone to the same visual regression problems that our  UI is. The set up is relatively easy using [action mailer previews](http://guides.rubyonrails.org/4_1_release_notes.html#action-mailer-previews).

ActionMailer::Previews are only mounted in your development environment by default. We need to mount them in the test environment in order to take screenshots with Percy:

```ruby
  config.action_mailer.show_previews = true
  # the path to the directory of your ActionMailer::Previews
  config.action_mailer.preview_path = Rails.root.join("spec/mailers/previews")
```

Percy doesn't load iframes by default and mailer previews run inside an iframe so we need to change the default Percy settings in `rails_helper.rb`:

```ruby
config.before(:suite) do
  Percy::Capybara.use_loader(::Percy::Capybara::Loaders::SprocketsLoader, include_iframes: true)
  Percy::Capybara.initialize_build
end
```

Lastly, we can write an integration spec that uses meta programming to iterate over all of our mailers:

```ruby
require 'spec_helper'

describe 'Mail Preview' do
  it 'previews all the mails', js: true do
    # set up any data required for the mailer previews
    create(:user)

    # The action mailer preview classes don't load unless we visit a mailer url to begin with
    visit '/rails/mailers/'

    ActionMailer::Preview.subclasses.each do |preview_class|
      # build the urls for the previews
      (preview_class.instance_methods - Object.methods).each do |method|
        slug = "#{preview_class.name.underscore.gsub('_preview', '')}/#{method}"
        begin
          visit "/rails/mailers/#{slug}"
          within_frame(find('[name="messageBody"]')) do
            expect(page).to have_selector('table.body')
            Percy::Capybara.snapshot(page, name: slug)
          end
        rescue AbstractController::ActionNotFound
          puts "Couldnt visit #{slug}"
        end
      end
    end
  end
end

```
