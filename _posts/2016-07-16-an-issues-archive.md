---
layout: post
title: An Issues Archive
updated: 2016-07-13 23:48:32
category: tech
---

## A Personal Record

To remember what I've accomplished and learned this summer & help me recount or reuse what I did.

<div class="divider"></div>

*In chronological order*

***Note:*** Bigger tickets will be put in their own post!!

<div class="divider"></div>

# 1. Adjust popup time on Email Capture Modal

* Literally going into the **coffeescript** file for the email capture modal and changing the timeout from 5000ms to 12000ms.

* Learned about the git workflow (PRs & merging into beta & whatnot) and navigating the codebase (ish) though!

# 2. Update publisher lead gen destinations

* Learned about **Sidekiq**: enqueues jobs such as sending out emails.

* In development, all emails are caught in **MailTrap**, which you can log into via **Bitium**.

* **Redis** server needs to be running for jobs to enqueue. Broke my first & learned about **specs** -- you always need to update or add specs for changes that are more than just simple UI things.

# 3. on iOS BBB and Powered by StackCommerce footer logos blocked by Add To Cart button

* Simple changing margins in CSS on mobile settings using Javascript

# 4. Add button to resend order confirmation email

* Wow! I got to create a cool button on the Admin page using Bootstrap styles!!! Fun stuff

* Learning **erb** and how to use buttons to execute a method:

```ruby
<%= link_to "Resend Confirmation Email", resend_confirmation_email_admin_order_path(@order), class: "button", method: :post %>
```

* Setting up routes and the method in the appropriate controller to execute the task:

```ruby
config/routes.rb

member do
  post :resend_confirmation_email
end
```

```ruby
app/controllers/admin/orders_controller.rb

def resend_confirmation_email
  @order.send_confirmation_email
  redirect_to admin_user_order_path(@order.user, @order)
end
```

* Creating specs to test a new feature!!!
* Extra special because I had to figure out some Sidekiq stuff

```ruby
spec/controllers/admin/orders_controller_spec.rb

describe 'resend_confirmation_email' do
  Sidekiq::Testing.fake! do
    it 'enqueues job onto Sidekiq mailer' do
      admin = create :user, :admin
      sign_in_as admin

      order = create :order, :w_item

      expect{post :resend_confirmation_email, id: order.id}.to (
      change(Sidekiq::Extensions::DelayedMailer.jobs, :size).by(1))
    end
  end

  it 'redirects to /users/:user_id/orders/:id' do
    admin = create :user, :admin
    order = create :order

    sign_in_as admin

    get :show, id: order.id

    expect(response).to redirect_to(admin_user_order_path(order.user, order))
  end
end
```

* My first like, kind of grown-up spec where I made a real feature and tested it and everything. Learned a lot and even marked it as my PR I was proud of just because of the struggles it took.

# 5. Remove product icon in bundles

* Literally just removed like a div

# 6. JS example for Sales API

* Barragan basically did this for me during one of our pairing sessions lol

* Used [jsfiddle](https://jsfiddle.net/kv67ks0c/) & Bootstrap to consume our API endpoint and display sales nicely:

```javascript
var publisher_id = 1;
var per_page = 8;
var url = '//api.stacksocial.com/v0/search/sales?publisher_id=' + publisher_id + '&per_page=' + per_page;

var source = $("#sales-template").html();
var template = Handlebars.compile(source);

$.get(url, function(data) {
	$('body').append(template(data));
});
```
* I honestly didn't know how to get the json stuff so that was cool (we just copied an example)

* We used [Bootstrap cards](http://v4-alpha.getbootstrap.com/components/card/#content-types) to make them nice little sales displays

* Also used Handlebars to increase readability / make it look more like a template:

**Handlebars**: a templating language used with html. These two lines compile the template in javascript:

```javascript
var source = $("#sales-template").html();
var template = Handlebars.compile(source);
```

The template is then delivered to the browser in the `<script>` tag. In this example, our templated variables help us extract the data from the JSON object received from the API call.

JSON received from API call:
![ApiCall]({{site.baseurl}}/assets/images/issues-archive/sales-api.png)

How we used Handlebars to extract the `title`, `subtitle`, and `main_image` from each sales in the JSON and put them into pretty cards:

```
<script id="sales-template" type="text/x-handlebars-template">
  <div class="container-fluid">
    <div class="row">
    	{% raw %} {{#each sales}}
      	<div class="col-xs-4">
         <div class="card">
           <img class="card-img-top" src="{{main_image}}">
          <div class="card-block">
            <h6 class="card-title">{{title}}</h4>
            <p class="card-text">{{subtitle}}</p>
          </div>
         </div>
       </div>
      {{/each}} {% endraw %}
    </div>
  </div>
</script>
```

The final result:

![Result]({{site.baseurl}}/assets/images/issues-archive/sales-api-example.png)

<div class="divider"></div>

# 7. Reset password email not branded

* Mostly looking and copying. Adjusting the ClearanceMailer to use the same BaseMailer that the TransactionalMailer was using to get the header, footer, and nice formatting in the order confirmation etc. emails

```ruby
class ClearanceMailer < BaseMailer
  def change_password(user_id, publisher_id)
    @current_publisher = Publisher.find(publisher_id)
    @user = User.find(user_id)

    mail to: @user.email,
    	 from: "#{@current_publisher.platform_name} <#{@current_publisher.support_email}>",
    	 subject: 'Change your password',
    	 "X-From-Name" => @current_publisher.platform_name
  end
end
```

* Then once again kind of "cherry-picking", if you will (I'm probably totally using it wrong), the erb view from the order confirmation emails to have the content & button needed for reset password.

# 8. Convert all state machines to statesman

* My first big, meaty ticket!!! See [here]({{site.baseurl}}/2016/07/14/statesman.html) for full post.

# 9. Add Academies section to stackcommerce

* Ez front-end stuff, ya feel

* Adding new assets to use

* Once again kind of looking at existing divs and copying & modifying until it looked like the mock

* ADD PIC

# 10. StackCommerce vendor signup does not acknowledge sign up

* Literally the confirmation modal already existed (for instructor sign up), so I just had to add the div to the vendor signup view and the JS and everything was already there to get it working

* ADD PIC

# 11. Add custom fields to FB feed product catalog

* This was really simple bc Nick showed me the file for the rss builder, I just added a new xml tag with the label and data that they wanted.

* Did write a new method in the decorator to calculate appropriate amounts, reusing / looking at old methods. (These methods can only be called if you decorate the sale first!)

* Remember to always update corresponding specs!

```ruby
@sales.each do |sale|
  xml.item do
    xml.tag! 'g:id', sale.id
    xml.tag! 'g:availability', facebook_sale_availability(sale)
    xml.tag! 'g:condition', 'new'
    xml.tag! 'g:description', sale.title
    xml.tag! 'g:product_type', facebook_product_type(sale)
    xml.tag! 'g:image_link', sale.main_image
    xml.tag! 'g:link', sale_url(sale)
    xml.tag! 'g:title', sale.name
    xml.tag! 'g:price', "#{sale.price} USD"
    xml.tag! 'g:brand', current_publisher.name
    xml.tag! 'g:expiration_date', sale.expires_at
    xml.tag! 'g:custom_label_0', sale.promotion_percentage
    xml.tag! 'g:custom_label_1', sale.discount_in_dollars
  end
end
```

Those last two custom labels are mine!

# 12. Collect source info for users and orders

* Another kind of big, interesting ticket, which I had no idea how to approach but turned out to be pretty simple.

* See [here]({{site.baseurl}}/2016/07/14/utm-tracking.html) for full post

# 13. Update SEO meta description

* Literally Talysson had already made a PR for it, so I just mimicked his changes and added the different titles / descriptions for the different verticals. And Hsieh corrected my syntax / style. Ez.

* **Meta title / description**: the title shows up on your browser window, and the title & description show when your site is listed on Google searches. Can be found by viewing page source.

* Basically we needed different titles & descriptions for each vertical (since they have 3 now) so here's the syntax I used:

```ruby
@title = if current_publisher.vertical == 'lifestyle'
  "#{current_publisher.platform_name}: Modern & Stylish Goods for Men"
elsif current_publisher.vertical == 'academy'
  "#{current_publisher.platform_name}: Online Courses from Top Instructors"
else
  "#{current_publisher.platform_name}: The Hottest Tech Deals, Delivered Daily"
end
```

# 14. Product fields directly injects HTML into page

* WIP, big meaty one again yay!!! The ticket title is not very informative.
