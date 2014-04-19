blue-omniauth-meetup
====================

## Summary ##
The following content was used for a Rails Meetup at Blue1647 in Chicago on
4/20/2014

Covers integrating with OmniAuth (without Devise) using the Twitter and Github
providers. Also covers integrating with Devise and OmniAuth using those same
providers.

## Questions to ask before starting ##

- Will you want to support usernames/passwords as well as a 3rd party logins such
  as twitter? If not, which one will you need?
- If integrating with 3rd parties, will you allow users to connect via multiple
  providers simultaneously or just one?
- If connecting with 3rd parties, will you use them simply for logging in, or
  will you also want to pull/post data on users behalf?

## Standalone Omniauth ##
Omniauth can be used with or without Devise (or another authentication
provider).  The steps and specific configurations are slightly different, but
the overall idea is the same.

**High level steps for integration Omniauth**

1.   create your applications for providers you wish to support
2.   add gem 'omniauth-twitter' for twitter
3.   add gem 'omniauth-github' for github
4.   add and configure an omniauth.rb initializer file (with scopes)
5.   add route(s) to handle the callback response from provider
6.   create a vanilla sessions controller
7.   add a create action that will handle the callback
8.   create model to store user fields returned from provider
9.   create and/or find user based on provider and uid returned from provider
10.  override the default password required validator in user model
11.  set session for logged in user
12.  create helper method to handle current\_user
13.  add route for destroying session
14.  add link for destroying session
15.  add action destroy for destroying session




## OmniAuth with Devise ##

**High level steps for integrating Omniauth with Devise**

1.   starting with a devise application that already has a user model
2.   create your applications for providers you wish to support
3.   add gem 'omniauth-twitter' for twitter
4.   add gem 'omniauth-github' for github
5.   configure the omniauth initializers within devise initializer (with scopes)
6.   add the omniauthable module to user model (see rake routes)
7.   create a omniauth_callback that inherits from Devise::OmniauthCallbacksController
8.   add a provider action that will handle the callback (i.e def github)
9.   extend model to store user fields returned from provider
10.  create and/or find user based on provider and uid returned from provider
-   override the default password required validator in user model
-   set session for logged in user
-   create helper method to handle current\_user
-   add route for destroying session
-   add link for destroying session
-   add action destroy for destroying session


```ruby
# 8. add a provider action that will handle the callback (i.e def github)
def github
  auth = request.env['omniauth.auth']
  user = User.find_by(provider: auth['provider'], uid: auth['uid']) || User.create_with_omniauth(auth)

  if user.persisted?
    flash[:success] = "successfully logged in"
    sign_in_and_redirect user, notice: "signed in"
  else
    redirect_to :root, notice: "failed to save user"
  end
end
```

```ruby
# 9. extend model to store user fields returned from provider
rails g migration add_omniauth_to_user provider:string uid:string
```

```ruby
# 10. create and/or find user based on provider and uid returned from provider
def self.create_with_omniauth(auth)
  create! do |user|
    user.provider = auth['provider']
    user.uid = auth['uid']
    user.name = auth['info']['name']
    user.email = auth['info']['email']
  end
end
```


