# figurine-urban-system

Epic Ruby On Rails 7 SaaS Image sharing app with Strapi CMS/Rails/PostgreSQL ORM-Backend with Redis Caching.

Frontend:Bootstrap 5, Font Awesome 5, Devise, Noticed, Announcements, Rollup, CSS Bundling, JS Bundling, Madmin Admin, GLightbox, Hotwire, Stimulus JS and Action Cable/Cable Ready & Request JS .

# JS & CSS Bundling with Rollup - no webpack/webpacker

* https://stimulus.hotwired.dev/

* https://docs.stimulusreflex.com/

# Combine with Hotwire and Strada #

* https://hotwired.dev/

Style kept minimal so Bootstrap 5 can be swapped out for other CSS or UI if required

* Ruby/Rails/Node/Postgres

- [Ruby](https://www.ruby-lang.org/en/) 3.0.0
- [Rails](https://rubyonrails.org/) 7.0.0 up
- [NodeJS] (https://nodejs.org/es/blog/release/v16.0.0/) 16.0.0
- [PostgreSQL](https://www.postgresql.org/docs/14/index.html) 14.00

## SET UP, ALL steps to be done except step 9 (optional) ##

After first set email & db credentials in .env files (cp .env.example to .env and fill in with your env details in both rails root and backend folders)

## Install deps from rails root

1. $ bundle install/update (if change any gems)
2. $ yarn  (engine node v 16.0.0)
3. $ bin/rails css:install:bootstrap  (dont overwrite application.bootstrap)

## Create DB from rails root folder

4. $ bin/rails db:create && cd backend

## IMPORTANT STRAPI - IN BACKEND folder Set db credentials in rails & backend .env file (postgres) copy .env.example > .env and enter details , then install deps ##

Ensure that Strapi & the Strapi admin/DB Tables are set up first, in backend folder terminal run "yarn develop" after first running yarn for the node_modules, then set an admin user in Gui same as Rails admin below once, cmd below.

5. $ yarn && yarn develop - Install deps, set up admin & admin user, content types (in admin Gui), run and watch files

## Open new terminal in rails root
6. $ bin/rails db:migrate
7. $ bin/rails db:schema:load  (will also change strapi admin tables in db to cascade)

8. Is set up to send basic user signup mail with gmail, alter smtp or other Email credentials (.env, application_mailer, devise.rb [noreply] & development.rb), then test signup confirmation emails by signing up & confirming user email
   Use same admin user as Strapi and set true in rails console with User.first.update :admin => true
   
9. $ rails stimulus_reflex:install (OPTIONAL, extra config needed see link above)

10. $ rails dev:cache (check, should be cached, if not run again)

OPTIONAL but best to have.
11. $ rails g madmin:install  (if not, have to alter nav)
12. $ rails g madmin:views  (already there styled for bootstrap overwrite requires restyle)

13. $ rails generate favicon  (put your favicon.png in assets/images, along with favicon.json in config folder)
    (see https://realfavicongenerator.net/)  for more info.
    
14. $ rails g rspec:install  (for testing)

15. Run Importmap

# examples 

 ./bin/importmap
 ./bin/importmap pin bootstrap  (pin bootstrap & popper, ie import)
 ./bin/importmap json (show maps)

## Start server! ##

16. $ rails s

(For other cmds see  related docs/links)

# Example other command

$ rails action_text:install (already done here)

(When create new files or migrations run Rubocop to correct and add frozen_string_literal offenses)

## Author

Paul Anthony McGowan - xhostcom@gmail.com

https://www.patreon.com/xhostcom

#################################################

### Strapi CMS Back End ###

1. Is self contained/hosted & is in root folder (backend) with schema screenshots & own subfolders
2. Uses the same Postgresql database as the Rails App, & values in own .env file
3. See its Readme for usage and also more docs at:

https://strapi.io/resource-center

#################################################

### Multi Tenancy Support ###

Add domain and subdomain to your tenant account table (Skip if your rails app already did this)

rails generate multi_tenant_support:migration YOUR_TENANT_ACCOUNT_TABLE_OR_MODEL_NAME

# Say your tenant account table is "accounts"
rails generate multi_tenant_support:migration accounts

# You can also run it with the tenant account model name
# rails generate multi_tenant_support:migration Account

rails db:migrate
Create an initializer

rails generate multi_tenant_support:initializer
Set tenant_account_class_name to your tenant account model name in multi_tenant_support.rb

- config.tenant_account_class_name = 'REPLACE_ME'
+ config.tenant_account_class_name = 'Account'
Set host to your app's domain in multi_tenant_support.rb

- config.host = 'REPLACE.ME'
+ config.host = 'your-app-domain.com'
Setup for ActiveJob or Sidekiq

If you are using ActiveJob

- # require 'multi_tenant_support/active_job'
+ require 'multi_tenant_support/active_job'
If you are using sidekiq without ActiveJob

- # require 'multi_tenant_support/sidekiq'
+ require 'multi_tenant_support/sidekiq'
Add belongs_to_tenant to all models which you want to scope under tenant

class User < ApplicationRecord
  belongs_to_tenant :account
end
Usage
Get current
Get current tenant through:

MultiTenantSupport.current_tenant
Switch tenant
You can switch to another tenant temporary through:

MultiTenantSupport.under_tenant amazon do
  # Do things under amazon account
end
Set current tenant global
MultiTenantSupport.set_tenant_account(account)
Temp set current tenant to nil
MultiTenantSupport.without_current_tenant do
  # ...
end
3 protection states
MultiTenantSupport.full_protected?
MultiTenantSupport.allow_read_across_tenant?
MultiTenantSupport.unprotected?
Full protection(default)
The default state is full protection. This gem disallow modify record across tenant by default.

If MultiTenantSupport.current_tenant exist, you can only modify those records under this tenant, otherwise, you will get some errors like:

MultiTenantSupport::MissingTenantError
MultiTenantSupport::ImmutableTenantError
MultiTenantSupport::NilTenantError
MultiTenantSupport::InvalidTenantAccess
ActiveRecord::RecordNotFound
If MultiTenantSupport.current_tenant is missing, you cannot modify or create any tenanted records.

If you switched to other state, you can switch back through:

MultiTenantSupport.turn_on_full_protection

# Or
MultiTenantSupport.turn_on_full_protection do
  # ...
end
Allow read across tenant for super admin
You can turn on the permission to read records across tenant through:

MultiTenantSupport.allow_read_across_tenant

# Or
MultiTenantSupport.allow_read_across_tenant do
  # ...
end
You can put it in a before action in SuperAdmin's controllers

Turn off protection
Sometimes, as a super admin, we need to execute certain maintenatn operations over all tenant records. You can do this through:

MultiTenantSupport.turn_off_protection

# Or
MultiTenantSupport.turn_off_protection do
  # ...
end
Set current tenant acccount in controller by default
This gem has set a before action set_current_tenant_account on ActionController. It search tenant by subdomain or domain. Do remember to skip_before_action :set_current_tenant_account in super admin controllers.

Feel free to override it, if the finder behaviour is not what you want.

Override current tenant finder method if domain/subdomain is not the way you want
You can override find_current_tenant_account in any controller with your own tenant finding strategy. Just make sure this method return the tenat account record or nil.

For example, say you only want to find tenant with domain not subdomain. It's very simple:

class ApplicationController < ActionController::Base
  private

  def find_current_tenant_account
    Account.find_by(domain: request.domain)
  end
end
Then your tenant finding strategy has changed from domain/subdomain to domain only.

upsert_all
Currently, we don't have a good way to protect this method. So please use upser_all carefully.

Unscoped
This gem has override unscoped to prevent the default tenant scope be scoped out. But if you really want to scope out the default tenant scope, you can use unscope_tenant.

Console
Console does not allow read across tenant by default. But you have several ways to change that:

Set allow_read_across_tenant_by_default in the initialize file

console do |config|
  config.allow_read_across_tenant_by_default = true
end
Set the environment variable ALLOW_READ_ACROSS_TENANT when call consoel command

ALLOW_READ_ACROSS_TENANT=1 rails console
Manual change it in console

$ rails c
$ irb(main):001:0> MultiTenantSupport.allow_read_across_tenant
Testing
Minitest (Rails default)
# test/test_helper.rb
require 'multi_tenant_support/minitet'
RSpec (with Capybara)
# spec/rails_helper.rb or spec/spec_helper.rb
require 'multi_tenant_support/rspec'
Above code will make sure the MultiTenantSupport.current_tenant won't accidentally be reset during integration and system tests. For example:

With above testing requre code

# Integration test
test "a integration test" do
  host! "apple.example.com"

  assert_no_changes "MultiTenantSupport.current_tenant" do
    get users_path
  end
end

# System test
test "a system test" do
  Capybara.app_host = "http://apple.example.com"

  assert_no_changes "MultiTenantSupport.current_tenant" do
    visit users_path
  end
end
Code Example
Database Schema
create_table "accounts", force: :cascade do |t|
  t.bigint "domain"
  t.bigint "subdomain"
end

create_table "users", force: :cascade do |t|
  t.bigint "account_id"
end
Initializer
# config/initializers/multi_tenant_support.rb

MultiTenantSupport.configure do
  model do |config|
    config.tenant_account_class_name = 'Account'
    config.tenant_account_primary_key = :id
  end

  controller do |config|
    config.current_tenant_account_method = :current_tenant_account
  end

  app do |config|
    config.excluded_subdomains = ['www']
    config.host = 'example.com'
  end

  console do |config|
    config.allow_read_across_tenant_by_default = false
  end
end
Model
class Account < AppplicationRecord
  has_many :users
end

class User < ApplicationRecord
  belongs_to_tenant :account
end
Controler
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id]) # This result is already scope under current_tenant_account
    @you_can_get_account = current_tenant_account
  end
end
