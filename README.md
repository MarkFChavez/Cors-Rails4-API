#CORS

I decided to decouple my frontend web apps from Rails and I was in for a pleasant surprise.
I noticed in the web server console an options request and Rails did not know how to respond.

After some digging I realized it was a CORS pre-flight permissions check and the same origin
policy ruined my apps interaction with the API. To get around this, I set all option actions
to a method in my API controller that responds the way the browser expects and it sets
the headers. I don't support any option routes in my API besides the CORS response so it
shouldn't be a problem. I looked around for a solution and no one way worked out of the box
but this [gist](https://gist.github.com/dhoelzgen/cd7126b8652229d32eb4) pointed me in the right direction. 
It should be noted that my app uses token authentication so
allowing all origins won't be a problem under tight security, it just puts more pressure
on my authentication system. It should also be noted that I think there will be problems
with SSL for allowing all origins so the * may need to be changed in the future.

##API controller (my api end points all inherit from this controller)

```ruby
#APIController

before_action :authenticate_user
after_filter :cors_set_access_control_headers
skip_before_filter :authenticate_user, :only => [:route_options]

def route_options
  cors_preflight_check
end

private

  def authenticate_user
    #Do some cool stuff with tokens to identify the user
  end

  def cors_set_access_control_headers
    response.headers['Access-Control-Allow-Origin'] = '*'
    response.headers['Access-Control-Allow-Methods'] = 'POST, GET, PUT, PATCH, DELETE, OPTIONS'
    response.headers['Access-Control-Allow-Headers'] = 'Origin, Content-Type, Accept, Authorization, Token, Auth-Token, Email'
    response.headers['Access-Control-Max-Age'] = "1728000"
  end
     
  def cors_preflight_check
    if request.method == 'OPTIONS'
      request.headers['Access-Control-Allow-Origin'] = '*'
      request.headers['Access-Control-Allow-Methods'] = 'POST, GET, PUT, PATCH, DELETE, OPTIONS'
      request.headers['Access-Control-Allow-Headers'] = 'X-Requested-With, X-Prototype-Version, Token, Auth-Token, Email'
      request.headers['Access-Control-Max-Age'] = '1728000'  
      render :text => '', :content_type => 'text/plain'
    end
  end
```

And in my routes I do this at the end of the routing file

```ruby
#Routes

#Route this to wherever you put the cors_preflight_check
#This is to handle the CORS preflight request, it only catches the options action.
controller 'api/v1/api' do
  match '*unmatched_route', :to => 'api/v1/api#route_options', via: [:options]
end
```

Hope this helps anyone who still has this problem.
