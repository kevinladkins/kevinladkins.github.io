---
layout: post
title:  "Building User Authentication with React/Rails"
date:   2017-07-11 19:12:20 -0400
---


For my final Flatiron project, I wanted to figure out how to authenticate users in my React/Redux/Rails app. With the help of a couple of incredibly informative resources -- Luke Ghenco's <a href="http://instruction.learn.co/video_lectures/131">epic e-commerce site video lectures</a> and a series of <a href="http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/">posts</a> at <a href="http://www.thegreatcodeadventure.com/">The Great Code Adventure</a> -- I managed to pull it off. Here's a quick rundown of how it worked.

**Rails Backend:**

The basic concept is that you want your API to issue an encrypted token at the start of each session that can be used to authenticate requests between the client and the server. Specifically, it issues a <a href="https://jwt.io/">JWT token</a>, which allows you to encrypt a user's information and embed it in a JSON object that can be included in every request requiring authentication. On the Rails side of things, you'll need to add the <a href="https://github.com/jwt/ruby-jwt">jwt gem</a> to your Gemfile to generate your tokens. 

The basic functioning of the JWT gem is this: you feed your **payload** (i.e., the data you want to encrypt), **authorization secret** (a string stored as an environment variable), and an **algorithm** (the default one seems to be 'HS256') into the **JWT.encode** method...

```
JWT.encode({payload}, secret, algorithm)
```

...which in practical terms will look something like this: 

```
JWT.encode({user_id: 1}, ENV['AUTH_SECRET'], "HS256")
```

(For setting the secret, the <a href="https://github.com/bkeepers/dotenv">dotenv-rails gem</a> provides an easy method of storing environment variables during development. You can then create a `.env` file, add it to your `.gitignore`, and set your secret there: 

```
export AUTH_SECRET:super_secret_authorization_code
```
)

The encoded token looks something like this: `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.nUM5gQIT4OeDnK5vskito5BKTGfgBUFxijQ5CmPL0FQ`, which, if you squint, you'll see consists of the three parts -- the **header** (which tells the type of token and the hashing algorithm being used), the **payload** (the info you encoded), and the **signature**. (I'm not going to pretend that all this encryption magic doesn't pretty quickly ascend above my paygrade; you can read all about it from way smarter people<a href="https://jwt.io/introduction/"> here.</a>)

To decode, you just feed the same info into JWT.decode, albeit in a slightly different format: 

```
token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.nUM5gQIT4OeDnK5vskito5BKTGfgBUFxijQ5CmPL0FQ"

JWT.decode(token, ENV['AUTH_SECRET'], true, {algorithm: 'HS256'})
```

...which will return the following: 

```
 [{"user_id"=>1}, {"typ"=>"JWT", "alg"=>"HS256"}] 
```

(I'll confess that I haven't looked into what that `true` or the `{algorithm: 'HS256}` is all about, but that's how it works).

Both of the sources I cribbed from suggested wrapping this functionality in an `Auth` class and storing it in an `auth.rb` file in `app/lib`; if you do so, you'll need to add the `/lib` directory to your Rails app in `config/application.rb`:

```
module YourAppName
  class Application < Rails::Application
    config.autoload_paths << Rails.root.join('lib')
  end
end
```

You can then build out an Auth class that provides a simple interface for encoding and decoding your tokens: 

```
require 'jwt'

class Auth
  ALGORITHM = 'HS256'

  def self.issue(payload)
    JWT.encode(payload, auth_secret, ALGORITHM)
  end

  def self.decode(token)
    JWT.decode(token, auth_secret, true, {alogrithm: ALGORITHM}).first
  end

  def self.auth_secret
    ENV['AUTH_SECRET']
  end
end
```


You can now access these methods in your Sessions controller to issue tokens to returning users (the example below also uses the <a href="https://github.com/codahale/bcrypt-ruby">bcrypt gem</a> to encrypt passwords):

```
class SessionsController < ApplicationController

  def create
    @user = User.find_by(email: user_params[:email])
    if !@user
      render json: {errors: ["user not found"]},
      status: 500
    elsif @user && @user.authenticate(user_params[:password])
      jwt = Auth.issue({user_id: @user.id})
      render json: {user: {id: @user.id}, token: jwt}
    else
      render json: {errors: ["password does not match provided email"]},
      status: 500
    end
  end

  private

  def user_params
    params.require(:user).permit(:email, :password)
  end

end
```

The salient point is that if your user exists in the database and provides a valid password, this little bit...

```
jwt = Auth.issue({user_id: @user.id})
render json: {user: {id: @user.id}, token: jwt}
```
 
...will generate a JSON response that looks like this:

```
{
  "user": {
    "id": 1
  },
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.nUM5gQIT4OeDnK5vskito5BKTGfgBUFxijQ5CmPL0FQ"
}
```

This token can then be stored in your React App and used to authenticate further requests -- assuming, of course, that you've built an **authentication method** in your Application Controller. My two guides for this process approached this logic in different ways, but the basic idea is:

1. Retrieve the token from the <a href="https://stackoverflow.com/questions/17396611/what-is-the-env-variable-in-rack-middleware">environment hash</a>.
2. Decode the token to retrieve the user's id.
3. Use the id to set the Current User. 

The simplest version of that logic looks like this: 

```
class ApplicationController < ActionController::Base

	
	def authenticate
      if request.env["HTTP_AUTHORIZATION"]
          token = request.env["HTTP_AUTHORIZATION"].split(" ").last
          decoded = Auth.decode_token(token)
          @user_id = decoded[0]["user_id"]
      else
        render json: {errors:
           {message: "You must include a JWT token!"}
        }, status: 403
      end
    end
	
	def current_user
      @user ||= User.find_by(id: @user_id) if @user_id
   end
		
 end
```

To break it down further: 

1. `token = request.env["HTTP_AUTHORIZATION"].split(" ").last` -- this step pulls the token from the request header -- which looks something like this: `'AUTHORIZATION': Bearer: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.nUM5gQIT4OeDnK5vskito5BKTGfgBUFxijQ5CmPL0FQ` -- and splits off the token itself (`.split(" ").last`).
2. `decoded = Auth.decode_token(token)` -- decodes the token
3. `@user_id = decoded[0]["user_id"]`  -- retrieves the user's id from the returned JSON object and stores it in an instance variable (`@user_id`)...
4. ....which sets the value of `current_user` : `@user ||= User.find_by(id: @user_id) if @user_id`

And with that, you can now slap `before_action :authenticate` on any routes that require authentication. 


**React Client**

Over on the front end, it's pretty simple. All you do is: 

1. Retrieve the token from the reponse to your login request
2. Store the token in <a href="https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage">session storage</a> for later retrieval (or in <a href="https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage">local storage</a> if you prefer; see links for the difference between the two)

Basically, you'll do something like this (assuming you're using redux's Thunk middleware): 

```
export function login(userDetails) {
    return fetch('/login', {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({user: userDetails})
    }).then(response => {
      if (response.token) {
        sessionStorage.setItem('jwt', response.token);
      }
    }).catch(error => {
        throw(error);
     });
  };
```

It's just your basic post request, sending the user's credentials (passed in here as `userDetails`, containing email and password), then -- assuming success -- retrieving the token from the response `response.token`, and storing it in session storage (`sessionStorage.setItem('jwt', response.token)`).

The token will now be accessible from anywhere in your React app via `sessionStorage.jwt`, which you can use to include the token in any requests to authenticated routes:

```
export function saveSources(user_id, source_ids) {
  return fetch(`/users/${user_id}`, {method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'AUTHORIZATION': `Bearer: ${sessionStorage.jwt}`
  },
  body: JSON.stringify({user:
      {sources: source_ids}
    })
  }).then(response => response.json())
  .catch(error => {
   throw(error);
  });
}
```

This is an example from my app, which uses the Rails API to save user preferences (i.e. the `source_ids` being passed into this request). The key bit is this right here: 

```
headers: {
    'Content-Type': 'application/json',
    'AUTHORIZATION': `Bearer: ${sessionStorage.jwt}`
  },
```

This is where your `authenticate` method in your Application Controller will find your token as described above. 

A key thing to note here is that the token is not being stored in your Rails database; all Rails knows is how to *decode* it; it finds the user based on the payload encrypted within. That means that all that's required to "log out" is for your React client to remove the token from session storage (via `sessionStorage.removeItem('jwt')`):

```
export function logOutUser(e) {
    e.preventDefault()
    sessionStorage.removeItem('jwt')
}
```
You can then tie this method into whatever button or link you're using for Logout purposes; in my app, that looks like this:

```
<span
        className="navlink"
        onClick={(e) => {logout(e)}}>
		Logout
</span>
```

So: that's the quick (and hopefully coherent!) rundown of the process. For *way* more detail and insight, I highly recommend learning from those who taught me:

<a href="http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/">JWT Authentication with React + Redux</a>

<a href="http://www.thegreatcodeadventure.com/jwt-auth-in-rails-from-scratch/">JWT Auth in Rails, From Scratch</a>



 












