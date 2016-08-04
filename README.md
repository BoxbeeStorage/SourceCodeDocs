The Boxbee software platform was created by the following people:

* Kristoph Matthews (CTO)
* Angie Yang (Software Engineer)([GitHub](https://github.com/AnGiEYaNg)) ([LinkedIn](https://www.linkedin.com/in/angi3yang))
* Calvin Choi (Software Engineer) ([GitHub](https://github.com/cschoi3)) ([LinkedIn](https://www.linkedin.com/in/choicalvin))
* Kareem Kwong (Software Engineer) ([GitHub](https://github.com/kwngo)) ([LinkedIn](https://www.linkedin.com/in/kwngo))
* Brandon Roberts (Strategy) ([LinkedIn](https://www.linkedin.com/in/brandon-roberts-0137b164))
* Jahanzeb Khan (Designer) ([Website](http://lowfatgraphics.com))
* Elieser Duran (Design) ([Behance](https://www.behance.net/eduran03))

<h1>Purchase A License</h1>

For more information or to purchase a license, please email licensing@boxbee.com

<h1>ARCHITECTURE</h1>

![Boxbee Overall Architecture](https://dl.dropboxusercontent.com/s/7ne8dz7ejispen1/Screenshot%202016-07-10%2012.08.26.png)

#Overview 

The Boxbee platform was built as a multi-tenant system, with a separate front-end and back-end architecture. 

#Multi-tenancy
Boxbee was originally designed to run multiple storage companies, who in turn had multiple customers. This means that the first level of tenant is the Company. The second level of tenant is the Customer. 
 
If your company will be using this software to run other on demand storage companies, then the software will work as is, according to the two tiers of tenants listed above. 

For most companies however, you are acquiring this software solely to run your own on demand storage business and only care about the single tenant- the Customer. In this case, you have two options: 1) Modify the source code so that the Company model is completely removed from the app, along with all of its dependencies, or 2) Much easier- seed a single Company record that represents your company and make sure all of your Product(s) and Customers belong to that Company. More detail on this will be covered in the Models section.

Note in the diagram above that we have a load balancer. This is a lightweight instance running [HAProxy](http://www.haproxy.org/) contained within Docker, and hosted on Heroku. This is primarily designed for multitenancy, but is relevant to the "frontend and backend" separated architecture that we have. You will later need to reconfigure HAProxy, along with your Server's (e.g. Heroku) domain properties, and DNS (e.g. DNSimple) CNAME records. See the Getting Started section for more info. 

* In Boxbee's current setup, any traffic that is directly requesting the API server begins with the host _developer.boxbee.com_. When any of the front-end clients (the two AngularJS apps) need to make a request to read/write data to the API server, they use this host. You can use your own DNS provider to configure "developer" to whatever subdomain you want, and "boxbee.com" to whatever domain you want. Just make sure that you've adjusted this in the load balancer's HAProxy.cfg and on your server. 

* Any traffic whose requests begin with host www.boxbee.com go to the boxbee home page, which happens to be on the same client-side Angular app as the admin portal (boxbee-home). If you are running multiple storage companies as a software company, this should be set to www.yourdomain.com, on HAProxy, Your Server, and with your DNS management company. If you're not (most companies), comment out this route. 

* Any traffic whose requests begin with host admin.boxbee.com go to the boxbee admin portal app. Again, per above, you can change this to yourdomain.com

* Any traffic whose requests begin with host some-subdomain.boxbee.com go to the customer portal app. This is the marketing page for end storage consumers to go to find out about the product, make an order, and go into their dashboard. If you are only running your own on demand storage company and not licensing software to other companies, then this is irrelevant to you. Rather, you should change some-subdomain to a subdomain like "www", or "customer".

# Frontends and Backend

As hinted in the above section and the title schematic image, the Boxbee API consists of a divided frontend and backend services architecture, rather than a single monolithic app. The Boxbee API represents the server and all requests involving reading/writing data in the entire platform are directed to this app. 

There are two frontend (read "client-side") apps: Boxbee-home and Customer-portal. 

* Boxbee-home (a.k.a. "Admin Portal") is used to host Boxbee.com and the Admin portal (the back-office tool, where parameters of your on demand storage business are configured). This is built in the AngularJS client-side framework and makes requests and is authenticated by the Boxbee API (aka "Server").

*Customer-portal is used to host the customer portal, which represents the marketing site (which advertises your storage business and allows customers to make purchases and manage their subscription). This is also built in AngularJS and communicates with the Boxbee API backend. Both this and the admin portal client side apps are hosted on NodeJS/Express on Heroku and are referenced in the Load Balancer's HAproxy.cfg file. 

NOTE: Both of these client side apps, should you choose to use them, must be re-wired to use the Boxbee API. This intends changing the base URL to the API to whatever your domain is. You are not required to use our frontends and are welcome to build your own web experience from scratch, while still utilizing the backend API.

<h1>GETTING STARTED</h1>

#Requirements 
## Devops 
* A rails compatible server such as Heroku, AWS, Digital Ocean, etc.

##Software 
* Ruby version 2.2.3
* Rails version 4.2.4
* PosgreSQL (pg version 0.18.4)
* Imagemagick (Run `brew install imagemagick`)
* REDIS database (Run `brew install redis`)
* HaProxy running on Docker (Step by step [resource](https://pilot.co/blog/hosting-multiple-heroku-apps-on-a-single-domain/) for setting this up)

##Services
In addition to installing the basic stack above, you must have accounts with the following services (exceptions noted). Most of these services are either free or pay per use. If you don't find certain services below to be relevant to your application, feel free to not use them. However, you will have to remove references to these services in the code, usually visible in the code with the format of `Figaro.env.SERVICE_NAME`

* New Relic APM - for application and downtime monitoring 
* Sendgrid - for sending out system emails to customers 
* Stripe Connect Managed Accounts - for credit card processing and billing 
* Honeybadger - for server error reports 
* Amazon Web Services (AWS) account, specifically S3 for file/image storage 
* A REDIS server (e.g. RedisToGo, etc.)
* Mapbox (for mapping and address validation, etc.)
* Geonames (for properly assigning customers to time zones) 
* Firebase (a cloud database for temporarily storing and querying billing data from Stripe)

# Getting Up And Running

##Boxbee API  (boxbee-api repo)
It is recommended that you use Heroku as your app's server. If not, the below instructions need to be adjusted for your server.

1. Provided that [Homebrew](http://brew.sh) is installed, run the following commands to allow for image processing on your machine:
    * `brew install cairo`
    * `brew install imagemagick`

2. Run `bundle install` to install all of the Ruby gems in the Gemfile.

3. Run `bundle exec figaro install`. This will place an application.yml file in your /config directory and .gitignore the file.

4. *Load application.yml:* Application.yml is a secret file that contains all of your secret credentials for the abovementioned services, among other things. This should NEVER be committed to version control. Included in your repository is a sample_application.yml for guidance as to what keys and values to put in the actual application.yml file. 

5. It's possible that you'll get a complaint that your pg database could not be found. Make sure Postgres is running in the background and that you have run `rake db:create`.

6. Run `bundle exec rake db:migrate` 

7. Deploy to production. If you look at the Procfile, there are 3 separate processes running. If you're using Heroku, make sure to scale up a dyno for each process to make sure everything runs.

8. Seeding. It is recommended that you seed the database on your staging server with some appropriate data to be able to start using and testing out the API endpoints right away. We provide an easy way to put in randomized data, by running `bundle exec rake db:seed`. This will draw from the db/seeds.rb file, which you should review anyway, to see the necessary sequence that data needs to be added in, in order for the API to work correctly. If you elect not to automatically seed the database with this sandbox data, DO see our seeding section to understand at minimum what data the API needs to be able to start properly. 

###Finished
At this point you should be able to start making requests to the API.

##Boxbee Admin Portal

Boxbee admin portal is an AngularJS app served on an Express server. It uses Gulp as a build tool and Browserify for module management. 

Gulp is a toolkit that helps you automate painful or time-consuming tasks in your development workflow. Browserify is a pre-processor for javascript which allows you to use npm modules client side. Boxbee admin portal supports Node >= 5.0.0.

1. Run `npm install` and `npm install -g gulp`

2. Once installed, run `gulp`. This will run a number of tasks (e.g. concat, bundle, etc.) as well as start the local server. See `gulpfile.js` to see the source code for the tasks.

3. Visit `localhost:3010` to see the live app.

#### Hooking up your API
Boxbee admin portal can be hooked up to your backend API through the Angular factory at `src/common/API/API.js`. In the file, replace productionHost with the domain of your API as well as the development and production API links.

    module.exports =/*@ngInject*/ function() {
       var API = {};
       var productionHost = "your-production-app.com";
        # Check if productionHost can be found
        if (window.location.hostname.toLowerCase().search(productionHost) < 0) {
         # Development API link    
	     API.API_URL = 'api.your-development-app.com';
         } else {
         # Production API link
	     API.API_URL = 'api.your-production-app.com';
	}

	return API;
    };

#### Hooking up Stripe
Boxbee admin portal can be hooked up with Stripe through the Angular factory at `src/common/API/Stripe.fact.js`. Simply replace `Stripe.setPublishableKey('your-publishable-key-here');` with your publishable key.

#### Hooking up Mapbox
Boxbee admin portal relies on the Mapbox API for some of its routing capabilities. Requests to Mapbox are proxied through the Express server. To configure Mapbox in development, set the key in `src/env/development.js`. In production, config vars can be set on heroku using the command `heroku config:set MAPBOX_KEY=MY_PRODUCTION_API_KEY` or whatever is required by your app server.

The API key is also required in the Angular map directive at `src/components/admin/operations/map/Map.dir.js`. It can be configured near the top of the file at `var MAPBOX_KEY = "YOUR_MAPBOX_API_KEY";`

##Customer Portal (customer-portal repo)

Boxbee customer portal is an AngularJS app served on an Express server. The structure of the app is very similar to that of the admin portal. It also uses Gulp as a build tool and Browserify for module management. Boxbee customer portal supports Node >= 5.0.0.

1. Run `npm install` and `npm install -g gulp`

2. Once installed, run `gulp`. This will run a number of tasks (e.g. concat, bundle, etc.) as well as start the local server. See `gulpfile.js` to see the source code for the tasks.

3. Visit `localhost:3030` to see the live app.

#### Hooking up your API
Boxbee customer portal can be hooked up to your backend API through the Angular factory at `src/common/API/API.js`. In the file, replace productionHost with the domain of your API as well as the development and production API links.

    module.exports =/*@ngInject*/ function() {
       var API = {};
       var productionHost = "your-production-app.com";
        # Check if productionHost can be found
        if (window.location.hostname.toLowerCase().search(productionHost) < 0) {
         # Development API link    
	     API.API_URL = 'api.your-development-app.com';
         } else {
         # Production API link
	     API.API_URL = 'api.your-production-app.com';
	}

	return API;
    };

#### Multitenancy
The Boxbee platform was built as a multi-tenant system, with a separate front-end and back-end architecture. If your company will be using this software to run other on demand storage companies, mapping the customer portal hostname to localhost will be required to view a properly seeded customer portal in development. On OSX, this can be done by changing the file at `/etc/hosts`. For windows, this is generally at `/windows/system32/drivers/etc`.

    ##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    # 127.0.0.1	localhost
    # ::1             localhost

    ::1 your-customer-portal-url.com
    255.255.255.255 broadcasthost
    ::1 localhost`
    




<h1>FILE ORGANIZATION</h1>

The Boxbee API follows a standard Rails file structure. 

* *app:* Contains all of the standard model and controller files according to Rails conventions. Note that API controllers all inherit from the ApiController class rather than ApplicationController and are namespaced to api/v1 in the folder structure as well as the routes themselves. 

* *app/jobs:* Contains all of the background jobs to be processed by Resque. We use [Resque](https://github.com/resque/resque) to process all background jobs and they are performed on a different (non-web) process. The job queue is stored in a REDIS database.

* *app/mailers:* All of the mailer classes for system generated emails 

* *app/models:* All of the models of objects in the database as well as their business logic. Note that there are a couple of non-Active Record models here, namely StripeSync, and all of its children. As you'll see in the "Billing" section of this Wiki, billing data is queried and stored to/from Stripe. To allow for better and faster queries, we've generated our own set of models and logic.

* *app/policies:* We use the [Pundit gem ](https://github.com/elabs/pundit) to maintain a set of access rules and scope server response data according to authorization level. The policies corresponding to each model are in this folder.

* *app/uploaders:* We use the [Carrierwave Gem ](https://github.com/carrierwaveuploader/carrierwave) to upload files received by the server to Amazon S3. The uploader classes that are mounted to the relevant models are in this folder. Note that because this is a JSON API (not multipart form), all images are accepted only in Base64 format and are parsed into .png files, etc. on the server and then uploaded using carrierwave. See app/controllers/api_controller.rb to see the method that parses the base64 string into an image. 

* *app/views:* Because this is an API and not a web page server, the use of this folder is very limited. Most of the html files you'll see here are related to the mailers.

* *config/initializers:* This folder contains all of the files that run when the server is started, and are largely related to the large array of external services we use for this app. Among the optional ones that can be deleted are: premailer, mixpanel (just delete any instances of `TRACKER` elsewhere in the app), and slack (just remove any instances of `SLACK_SIGNUP_NOTIFIER` and `SLACK_CRITICAL_NOTIFIER` elsewhere in the app). 

* *config/clock.rb:* We use the [Clockwork Gem](https://github.com/tomykaira/clockwork) as a sort of CRON job/scheduler tool to run background processes at regular intervals. You may adjust these to intervals of your liking. Note that the Slack-related processes are optional and can be removed. 

* *config/puma.rb:* We use the Puma application server for our Rails application server. Here you can configure properties of the server such as thread concurrency, etc. 

* *config/routes.rb:* All the routes for the rails application. Note the namespacing (api/v1) for the API related paths. We also have an AdminRestriction class that only allows admin access to the /resque route, which is a portal for you to manage and view all of your running background jobs. You can also see that we've instructed Devise to use our own home-grown controllers for sessions, passwords, and unlocks. See the "Authentication" section of this Wiki for more information. 

* *config/application.yml* If you followed the "Getting Started" guide in this Wiki you will have an application.yml file. Please see the sample_application.yml file for instructions on how to construct this file with all of the credentials of your different server. Note that we do not use secrets.yml in this app because this file replaces it. 

* *db:* This folder contains all of the database related files for migration, and seeding etc. It is strongly recommended that on your first run with this software, and/or on your staging server, you run `rake db:seed` to seed the data, and take a look at the sequence of how data is entered into the database. This app has a large number of models, with an even larger number of relationships and sequence-sensitive interdependencies, and looking at the seeds.rb file will help understand this a bit more. 

* *lib:* This folder contains a lot of business logic, helper files, and RAKE tasks. Starting with _assets_, this folder contains images for all of the ItemTypes that can be stored by a customer. Add as many images as you'd like here, but just realize they need to match the parameterized name of the ItemType itself in the model. E.g. "Large Table" matches with "large-table.png". Secondly, the "billing.rb" file is the billing routine, which looks at the customer activity over the last month and bills them accordingly. Lastly, "reports.rb" runs revenue reports on demand and generates a CSV with gross and net revenues broken down by accounting code, over a particular time frame. 

* *Procfile:* The Procfile follows the UNIX process model. There should be three processes here: Web, Resque, and Clock. Web tells the app to use Puma as its application server and will serve all of your web requests. Resque is a dedicated process for running background jobs. Clock is a dedicated process for timing and launching background jobs. 

<h1>SEEDING STAGING APP</h1>

The staging app is seeded by a seeding script, `seeds.rb`, and requires some other processes as well. This is written for Heroku, so translate as necessary for your server stack if you are not using Heroku. This should not be run on your production or development environments and is only so that you can experience a fully seeded database with fake, but functional data. 

1. Run `heroku pg:reset DATABASE_URL -a boxbee-api-staging`

2. Run `heroku run -a boxbee-api-staging rake db:migrate`

3. Run `heroku run -a boxbee-api-staging rake db:seed` to begin seeding the database.

4. Run `heroku run -a boxbee-api-staging rails c` to start the console, then queue the Appointment Cutoff Modification Job with `Resque.enqueue(AppointmentModificationCutoffJob)`. The tasks can be watched at http://boxbee-api-staging.herokuapp.com/resque/overview.

5. Run `heroku run -a boxbee-api-staging rake seeding:refresh` for the first time. This will group appointments into routes, and should run automatically every day from this point on to create new customers, appointments, etc. 

6. Run `heroku run -a boxbee-api-staging rails c` to start the console, then index the search engine by running `SearchSuggestion.index_main`

<h1>SEEDING PRODUCTION APP</h1>

When the production app is created, certain seeding procedures need to be done to prepare the app for operation. In addition, after each new company signs up with a product, subsequent seedings need to happen during the onboarding process. 

#Procedures for initializing the app
1. *Generate super company:* The super company is the first company in the database. This company represents "Boxbee Inc."
2. *Generate super user:* The super user belongs to the super company above and has the highest level of access in the Boxbee API
3. *Create allowed item_types:* A list of item_types belonging to the super company. Whenever a subsequent company signs up for Boxbee, the available item_types to choose from inherit from the super company's item_types.
4. *Generate default product data:* The default product information (at draft stage and at demo stage) need to be seeded into the super company's first and only product. All subsequent products will inherit from this first product. 

##RAKE task to run all aforementioned procedures

Use the command `rake seeding:initialize` to run all of the procedures above AFTER editing the seeding.rake file manually to reflect your business- OR just do all of the above procedures manually. Only run this on a blank slate database.

<h1>DEVOPS</h1>

* Database Backups. We recommend that you perform regular database backups. We use Heroku's Postgres addon for this. 

* Application Monitoring. To keep track of application performance and uptime, we've put in the configuration files already for NewRelic APM (free). Please make sure the signals are sending to New Relic. 

* Error Monitoring. To check for and diagnose internal server errors, we use HoneyBadger. 


* Deployment and Continuous Integration. We deploy to Heroku and have every push to Github automatically trigger Circle CI to run. If you wish to do this, you'll have to set it up yourself by linking Circle CI to your Github account. 

<h1>MODELS</h1>

Please see the individual model files themselves to learn more about their methods, capabilities, etc. This is a quick reference guide. 

# Address-related models

## AddressNoteOption
AddressNoteOptions are default address note templates provided by a company to give to their customers. Instead of requiring customers to type in a new note, they can select from some pre-saved templates. 

## AddressNote 
An AddressNote is a note that is attached to an Address to help inform the driver or customer service agent any special details about the address (e.g. "doorbell doesn't work").

## Address 
An address can belong to either a User or a Company (via a Warehouse). 

# Appointment-related models 

## Appointment 
An Appointment represents an event that is scheduled for a certain date and time with a customer. An appointment can have the following statuses: :scheduled, :confirmed, :started, :ignored, :completed, :canceled, and :failed

* Scheduled: This is when an Appointment has just been created

* Confirmed: This occurs automatically when an Appointment's cutoff time has been reached. After this point, in the customer portal, a customer cannot edit the appointment themselves, but can ask Customer Service to do it if it's allowed. 

* Started: This occurs when the Driver is en route to the appointment

* Ignored: This happens when the Driver or Warehouse worker marks the Appointment as ignored. 

* Completed: This occurs when the Driver marks the Appointment as completed after serving the customer. 

* Canceled: This occurs when the Driver marks the Appointment as canceled. 

* Failed: This occurs when the Driver marks the Appointment as failed after a no-show, and the customer will be billed. 

## AppointmentCutoff
As with most logistics companies, you can set cutoff times for various actions. There are 3 types of AppointmentCutoffs for a particular Product: :creation, :modification, and :cancellation. 

* Creation: The amount of lead time required before an Appointment can be scheduled. 

* Modification: The amount of lead time required before an Appointment can be modified.

* Cancellation: The amount of lead time required before an Appointment can be canceled. 

## AppointmentDetail 
An AppointmentDetail specifies what type of transit event (:pickup, :delivery) and request type (:empty, :packed) takes place during an appointment. An Appointment can have many AppointmentDetails.

## AppointmentHold 
When a Customer is in the checkout process but has not finished, a time slot is reserved via the AppointmentHold model. AppointmentHolds expire after 10 min unless otherwise configured, or a Customer makes an Appointment successfully. 

## AppointmentNote
Like AddressNote, AppointmentNote represents a text note that pertains specifically to that Appointment. 


# StripeSync models 
As mentioned in the "Billing" Section of this Wiki, StripeSync is a series of models built in house by Boxbee for syncing Stripe data to Boxbee and being able to query billing information easily, quickly, and intuitively.

## StripeSync
StripeSync is the model that all the below models inherit from and contains all of the shared business logic and query methods. All the models below are named after their corresponding Stripe Objects. [See the Stripe API Documentation](https://stripe.com/api/docs) for more information. 

Note that all of the methods on the classes below are scoped to a Stripe managed account on Stripe Connect and thus are scoped to a particular Company model. 

## BalanceTransaction
The transactions that occurred on the account of interest.

## Card 
The credit card object belonging to a customer. 

## Charge
The charge object corresponding to a BalanceTransaction, Refund, or Invoice. Note that Boxbee's API never issues a Charge. We instead create InvoiceItems, and then a single Invoice on the 1st day of every month. Stripe then routinely creates a Charge object corresponding to the Invoices shortly after the Invoices are created. 

## Invoice
Invoices consist of several InvoiceItems (aka "Line Items"). Once an Invoice is created in Stripe, Stripe auto creates a Charge. We generate only 1 invoice per month per customer so that customers aren't billed too many times. We found that this reduces churn and encourages long term customers. Taxes are applied to the Invoice level.

## InvoiceItem
InvoiceItems, also called "lines" or "line items", represent the different items and amounts that the customer is to be charged for.  

## Refund 
Refunds are issued to a customer on a particular charge/invoice. Refunds can only be issued once a corresponding Charge is issued and only up to the amount of the not-yet-refunded amount of the invoice. 

## Subscription
Subscription is actually NOT a StripeSync model, but we're including it in this section because of its relevance. Rather, Subscriptions are stored and maintained locally in ActiveRecord, and is used for the monthly billing routine to keep track of subscription status for a particular customer. 


# Company-related models 
## Company
A Company is as the name suggests, a Company offering storage service. Please read the multitenancy subsection of the "Getting Started" section of this Wiki. If you are only using this software for your own storage business (e.g. not sublicensing this to other storage companies), your company should be the only Company record in this model. 

## CompanyDetail
There is a one-to-one relationship between a Company and its detail. Because a detail contains very sensitive information that we don't necessarily want showing up in all responses that involve Company data, we've stored this in a separate model called CompanyDetail. 

# CustomerItem-related models
## CustomerItem
A CustomerItem is simply something that you're storing that belongs to the customer. It could be a box, a sofa, a hanger of clothes, etc. They belong to and derive their name, information, and image from the ItemType parent model. 

## CustomerItemHistory
A history/breadcrumb trail for a CustomerItem
## Asset
The image(s) or file(s) related to a CustomerItem that can be seen in their dashboard. 

# Employee-related models
## User
A User is a user who can be logged into the system and often has the roles of :super_user, :admin, :driver, :warehouse_worker, :customer_service (you can add/remove as many as you'd like). This is for all users, but you'll notice that customers have their own model, Customer, who belongs to this model (User). This is because a Company can have many Products, and the User associated with a Customer of one Product should be able to log in as a different Customer of another Product. 
 
## EmployeeInvitation
When onboarding a new employee (User), you can generate an EmployeeInvitation, which gets emailed to an employee's email, and they can subsequently register. 

# Inventory-related models 
## Inventory 
An inventory represents an item that is sitting in storage/the warehouse. It can have an `inventory_type` attribute of either "container" or "customer_item". A "container" is something that you deliver to your customer (e.g. a "box"). If that container has been packed and returned to storage, the inventory_type becomes "customer_item". 

## InventoryHistory
A breadcrumb trail of that Inventory item. 

# Product-related models 
## Product 
A Product defines a product that a customer can sign up for and use. It has its own set of marketing information attributes, website, pricing, and more unique characteristics. It has the following statuses: :draft, :demo, :active. When the status moves to :active, the app will interface with your Live account (no longer your Test account), so be careful. 

## Faq
If you're using our client side frontends, use this Model to store question and answer pairs belonging to a particular Product. 

## ProductServiceArea
For a particular Company, you will set certain ServiceAreas that you're able to serve with your fleet. You may however have multiple Products in that Company and have only a subset of those ServiceAreas apply to each Product. You can do that with this many-to-many model. 

# Route-related models 
## Route
A Route represents a sequence of Appointments for a particular date, for a particular Vehicle. It has the following statuses: :draft, :confirmed, :ready, :outbound, :inbound, and :completed. 
 
* Draft: Set when the Route is first created

* Confirmed: When the transit supervisor sets the Route status to confirmed 

* Ready: When the transit supervisor sets the Route status to ready. Can only happen if there is at least one Appointment associated with the Route. 

* Outbound: When the Driver indicates that he has started the Route on the mobile app.

* Inbound: When the Driver indicates that he has ended the Route 

* Completed: When the Driver has offload all of the Payload onto the Staging area and has marked the Route as complete on his app. 

## RouteHistory
A breadcrumb trail history of the Route.


# ServiceArea-related models 
## ServiceArea
A named area of service for a particular company, at the moment defined by its ServiceAreaZips. 

## ServiceAreaSchedule
A schedule for a ServiceArea, including hours of operation, day of the week, slot window duration, and max number of appointments allowed for a particular slot. 

## ServiceAreaZip
A zip code served by a ServiceArea. 

# Warehouse-related models 
## Warehouse 
A Warehouse belonging to a Company and its associated location 

## WarehouseTask
A task with item and pick/put Location in the warehouse so that the warehouse worker knows where and what items to find to load a vehicle. 

## Vehicle 
A series of attributes describing a vehicle and its status. This vehicle can be associated to a Route and thus a driver. 

## Location
A physical location for where an item exists or can exist in a Warehouse. Currently it is given as a freeform string to be compatible with your facility's naming conventions. 

# Other models 
## AccountingCode
Currently the app comes with three default accounting codes for storage, transit, and misc, respectively. You can add more as you please (see app/models/product.rb). This is namely for tax and revenue reporting purposes. 

## Barcode 
This is currently not hooked up to any controllers because at the time of writing, we wanted to give companies flexibility to use their own barcoding systems. However, this model contains a number of methods to print both CSVs and PDFs of barcodes while eliminating replication of barcode numbers in your warehouse.
 
## ContactDetail
Because Users can have multiple ways of being contacted, this model was created to keep track of all of those attributes and belongs_to a User. 

## CustomerFee
This model keeps track of the transit-related fees that a Customer locked into at the time of making an Appointment and belongs to an Appointment record. This helps keep track of pricing that was agreed to a Customer after you changed pricing for new Customers.

## Customer 
A Customer is a unique combination of a User and a Product. In other words, the same person has only one User account with your Company but can have access to multiple products by being a different Customer for each one. 

## Holiday 
If your Company has company-wide holidays, you can put them here so that the availability schedule your customers see reflects this. 

## ItemType 
An ItemType represents the types of items that your Customer can store. 

## Onboarding 
This is irrelevant for most companies. It was intended for onboarding new Companies on to the Boxbee platform. 

## Payload 
Payloads represent items that have been loaded on to a Vehicle. There is one Payload for each CustomerItem (e.g. 1 packed plastic box) or each set of ItemTypes (e.g. 50 plastic boxes).

## Plan 
Plans represent a Product's subscription offerings in terms of price and allowable ItemType that can be stored. 
 
## Report 
This contains the business logic for generating a revenue report with gross and net revenues, credit card processing fees, taxes, etc. 

## Role 
This is set by the Rolify gem and needs very little configuration. 

## ScheduleAdjustment 
If you want to make an adjustment to a ServiceAreaSchedule (for example more bookable appoitments per time slot or extended hours during a busy season), you can do that with this model. 

## SearchSuggestion
To make data instantly searchable and on hand for Customer Service agents in your company, we generated an indexed Redis-based store for searching. Use methods here for quick search. 

## Slot 
This model contains methods for generating time slots and determining time slot availability by month and by date so that you're customers can view availability and book appropriately. 

## Tax 
Tax records belong to a Product and are scoped to particular zip codes to accommodate for the fact that taxes for various types of businesses/activities can vary across county lines. 

<h1>AUTHENTICATION AND AUTHORIZATION</h1>

The Boxbee API uses a modified [Devise](https://github.com/plataformatec/devise) and [Pundit](https://github.com/elabs/pundit) setup for authentication and authorization, respectively. 

Because this is an API and not a web app, we're not storing session cookies to log in and out users. Instead, we use Simple Web Tokens to authenticate users. When a Customer or a User (e.g. an employee) is created, a `token` attribute is automatically generated for that customer or user. When a request is sent from one of the client-side apps on their behalf to the API, it must contain this token. The API (via app/api_controller.rb), then checks to see if a valid user corresponds to that token. That token must be provided every time a request is made (that requires authentication)

To allow this to work well with Devise, we have modified controllers for sessions, passwords, and unlock, located in the app/controllers/api/v1/users folder. Check them out for more information on how this works. 

Lastly, Pundit is used to control what roles have access to what endpoints, as well as what portion of the data to scope to the requestor. This is all done using the policy classes contained in the app/policies folder. 

Both Devise and Pundit work with models (User and Customer) that contain various roles. These roles are governed by the [Rolify Gem](https://github.com/RolifyCommunity/rolify) and can be adjusted to your liking. Just remember to adjust policies every time you adjust roles, because they go hand in hand. 

<h1>Billing and Stripe</h1>

Before reading this it's highly recommend that you develop a basic understanding of [Stripe Connect](https://stripe.com/connect). It is designed for multitenant platforms such as ours. 

# Multitenancy

As mentioned in the "Overall Architecture" Section of this Wiki, the Boxbee API is a multi-tenant platform. The first tier of tenant is the Company, and the second tier is the Customer and User. This was designed with the intention of serving multiple companies, who each had their own storage business, their own billing account, and their own set of customers. For most reading this, they will only be ONE company with ONE account, and ONE set of customers and users (aka employees). If you fall into the second category, all you need to do is create one Company record, which will in turn create one managed account on Stripe, who will bill your customers' credit cards. This is the quickest solution, which requires no modification to the source code. If however, you want to deal directly with Stripe and not use their Stripe Connect platform (recommended for the longer term if you're using this software for your own business and not licensing to other storage companies), you will need to rewire all of the models to work directly with Stripe.  

##Stripe Sandbox 
When a Company is created (for most licensees, you will be doing this only Once, for your own company), a corresponding Stripe Account is created in Stripe. It defaults to your Stripe Test Account to begin with, and as soon as your first product's `status` attribute becomes 'active', it switches to your Stripe Live Account. Go to the Product and Company models in the source code for more info. 

# Billing 
Stripe will bill your customers' credit cards on the first every month, and takes into account their activity over the last month. The logic for this billing routine can be found in lib/billing/biller.rb. The frequency for which this routine is to be run can be found in config/clock.rb.

# Querying 
You will find that you will need to do complicated and very rapid queries for customer service, and reporting invoices to customers etc.. Stripe's current endpoints are rather slow and don't have all the querying functionality you'd typically want to see. We've thus engineered a much better solution that regularly syncs data from Stripe into a Firebase NOSQL database. Then a model for all of the main Stripe objects (Card, Customer, Invoice, InvoiceItem, Subscription, BalanceTransaction) is created locally, with some simple, active_record-like querying methods and relationship queries, so you can use all of the Stripe objects like they are local models in your app. Go to the model files by the same name as the list above in app/models or see our "Models" section in this Wiki for more info. 

<h1>APPOINTMENT CART LOGIC</h1>

The Appointment Model has a `process_cart(cart, created_by, company_id)` method that is the way to tell the API to generate a new order for a Customer. The created_by argument should be the id of the User representing the Customer. The company_id should be the id of the Company that the Customer is making an order to. You can see an example of how this is done in app/controllers/api/v1/appointments.rb. The cart is the most important input parameter in this method and the following is a guide to constructing the cart object:

The cart is actually an array of JSON objects. Each JSON object in the array is an instruction to the process_cart method to do things like create/edit Appointments/Details, Subscriptions, CustomerItems etc. Note that the conditions for what instruction to give depends on whether the corresponding Plan's ItemType's requires_empty_delivery or requires_empty_pickup attribute are true or false. 

## Create an Appointment
```js
//Case 1: empty_delivery needed; empty_pickup needed
//New plan
{
  "plan_id": 1,
  "action": "new_plan"
  "quantity": 5, //if customer is already subscribed, this is the "topup" amount
  "request_type": "empty",
  "transit_type": "delivery"
}
//Request pickup (applies to picking up empties as well)
{
  "customer_item_id": 34459,
  "request_type": "empty", //can be either
  "transit_type": "pickup",
  "action": "update_customer_item"
}
//Request delivery
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "delivery",
  "action": "update_customer_item"
}

//Case 2: ONLY empty_pickup needed
//New plan
{
  "plan_id": 1,
  "action": "new_plan"
  "quantity": 5,
  "request_type": "packed",
  "transit_type": "pickup"
}
//Request delivery
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "delivery",
  "action": "update_customer_item"
}
//Request pickup (ONLY applies to empty pickup at end of lifecycle)
{
  "customer_item_id": 34459,
  "request_type": "empty",
  "transit_type": "pickup",
  "action": "update_customer_item"
}

//Case 3: ONLY empty_delivery needed
//New plan
{
  "plan_id": 1,
  "action": "new_plan",
  "quantity": 5,
  "request_type": "empty",
  "transit_type": "delivery"
}
//Request pickup (ONLY applies to packed pickup)
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "pickup",
  "action": "update_customer_item"
}
//Request delivery
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "delivery",
  "action": "update_customer_item"
}

//Case 4: Requires NEITHER empty_delivery NOR empty_pickup
//New plan
{
  "plan_id": 1,
  "action": "new_plan"
  "quantity": 5,
  "request_type": "packed",
  "transit_type": "pickup"
}
//Request delivery
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "delivery",
  "action": "update_customer_item"
}
```

## Update an Appointment
When updating a booking, a similar cart is required to instruct BMS on how
to change a booking and its related objects (active plans, active form factors, etc.).
Find below the different types of JSON objects that can comprise an array and form a cart.

_NOTE:_ Make sure to follow the conventions in the "Create a Booking" section above
for the different cases (requires_empty_pickup == false, requires_empty_delivery == true, etc.)

```js
/* Active Form Factors */
//Request pickup (for an item that is with user, and the customer wants to send to storage)
{
  "customer_item_id": 34459,
  "request_type": "packed", //depends on Cases 1-4 above (empty/packed)
  "transit_type": "pickup",
  "action": "update_customer_item"
}
//Request item (for an item that is in storage, but the customer wants it to be delivered home)
{
  "customer_item_id": 34459,
  "request_type": "packed",
  "transit_type": "delivery",
  "action": "update_customer_item"
}
//Remove an active form factor from a booking
{
  "customer_item_id": 34459,
  "action": "remove_customer_item"
}
/* Plans */
//Add a new plan to an existing booking
{
  "plan_id": 1,
  "action": "new_plan"
  "quantity": 5,
  "request_type": "packed", //again, request and transit types depend on above cases 1-4
  "transit_type": "pickup"
}
//Remove a plan that was requested as part of an existing booking
{
  "plan_id": 1,
  "action": "remove_plan",
}
//Change of ordered plan quantity for an existing appointment
{
  "plan_id": 1,
  "quantity": 5,
  "action": "update_plan"
}
//Add quantity to a plan that the customer is already subscribed to (for a NEW booking)
{  
  "plan_id": 1,
  "action": "new_plan"
  "quantity": 5, //if customer is already subscribed, this is the "topup" amount
  "request_type": "empty",
  "transit_type": "delivery"
}
```

<h1>DiSCLAIMERS</h1>

* No representations are being made that the source code here is 100% working. You will need to make tweaks here and there to get it working in the way you want it to. Hopefully the built in error messages by Boxbee and Rails, combined with documentation here and inline in the code are sufficient to get it working in the way you want it to. 

* Certain sections, such as the Mailer classes in app/mailers, and their corresponding views, are not going to be 100% complete for your business needs. Your business is unique and you're going to want to communicate with your customers in a certain way, with specific from/subject lines, etc. so please take a closer look at these and customize them to your liking. 

* While we've implemented RSpec and a few model/controller tests in this build, the tests are very limited. We encourage you to build your own tests as you build features. 



