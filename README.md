# setup sending Email by action mailer and our gmail account.

By this method we do not use mailgun but we using our gmail account as a SMTP account.

So we have to change our gmail security  as following:

by this link:

```
https://accounts.google.com/DisplayUnlockCaptcha

```
and by checking or gmail inbox and review email of

Review blocked sign-in attempt

and change :

 "allowing access to less secure apps"

 link of that email.

 or by this link:

 ```
https://myaccount.google.com/lesssecureapps

 ```


# Sending Emails Using ActionMailer and Gmail

first we need to copy

In : config/environments/development.rb:

```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

```
Important:by any change in config we need to restart the server  if we want to run code.

and in : config>environment>development

change false to true for:

config.action_mailer.raise_delivery_errors = true



1-rails g scaffold user name:string email:string

2-rake db:migrate

3- rails g mailer example_mailer

4- in app/mailers/example_mailer.rb

```
class ExampleMailer < ActionMailer::Base


  def sample_email(user)
    @user = user
    mail(to: @user.email, subject: 'Sample Email')
  end
end

```

5- in app/views/example_mailer, Create a file sample_email.html.erb

```
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Hi <%= @user.name %></h1>
    <p>
      Sample mail sent using smtp.
    </p>
  </body>
</html>

```
6- create sample_email.text.erb in the app/views/example_mailer directory.

```
Hi <%= @user.name %>
Sample mail sent using smtp.

```
7- In the development environment we can use ActionMailer Preview to test our application.

test/mailers/previews/example_mailer_preview.rb

```
# Preview all emails at http://localhost:3000/rails/mailers/example_mailer
class ExampleMailerPreview < ActionMailer::Preview
  def sample_mail_preview
    ExampleMailer.sample_email(User.first)
  end
end

```

8- We will do so by using the gem "figaro"

9- bundle exec figaro install

10- /config/application.yml

```
gmail_username: 'username@gmail.com'
gmail_password: 'Gmail password'

```

/config/environments/production.rb

```
config.action_mailer.delivery_method = :smtp
# SMTP settings for gmail
config.action_mailer.smtp_settings = {
 :address              => "smtp.gmail.com",
 :port                 => 587,
 :user_name            => ENV['gmail_username'],
 :password             => ENV['gmail_password'],
 :authentication       => "plain",
:enable_starttls_auto => true
}

```

We just need to add :

```

ExampleMailer.sample_email(@user).deliver

```
to the create method in app/controllers/users_controller.rb.

The create method in users_controller.rb should look something like:

```
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save

      # Sends email to user when user is created.
      ExampleMailer.sample_email(@user).deliver

      format.html { redirect_to @user, notice: 'User was successfully created.' }
      format.json { render :show, status: :created, location: @user }
    else
      format.html { render :new }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end

```

# Sending emails using ActionMailer and Mailgun through SMTP

1- /config/application.yml

```
api_key: 'API Key'
domain: 'Domain'
username: 'Default SMTP Login'
password: 'Default Password'
gmail_username: 'username@gmail.com'
gmail_password: 'gmail password'

```

2- /config/environments/production.rb

As we want to use mailgun as a SMTP need to replace the gmail config in development and production with following:

```
config.action_mailer.delivery_method = :smtp
# SMTP settings for mailgun
ActionMailer::Base.smtp_settings = {
  :port           => 587,
  :address        => "smtp.mailgun.org",
  :domain         => ENV['domain'],
  :user_name      => ENV['username'],
  :password       => ENV['password'],
  :authentication => :plain,
}

```

We may receive an error like:

554 Sandbox subdomains are for test purposes only. Please add your own domain or add the address to authorised recipients in Account Settings.

it means code is right but need to work on mailgun.Mailgun needs to our domain or authorised receiptance emails to avoid spam emails.


# Sending emails using ActionMailer and Mailgun through Mailgun’s APIs

1-

```

gem 'mailgun-ruby'

we have to replace perviouse sending method with this one:

```

2- app/mailers/example_mailer.rb

```
class ExampleMailer < ActionMailer::Base

  def sample_email(user)
    @user = user
    mg_client = Mailgun::Client.new ENV['api_key']
    message_params = {:from    => ENV['gmail_username'],
                      :to      => @user.email,
                      :subject => 'Sample Mail using Mailgun API',
                      :text    => 'This mail is sent using Mailgun API via mailgun-ruby'}
    mg_client.send_message ENV['domain'], message_params
  end
end

```
