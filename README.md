# GoWebApp
Basic Web Application in Golang

This project demonstrates how to structure and build a website using the Go language without a framework.

## Quick Start

To download, run the following command:

~~~
go get github.com/josephspurrier/gowebapp
~~~

Start MySQL and import database/database.sql to create the database, tables, and populate one of the tables.

Open config/config.json and edit the MySQL section so the connection information matches your MySQL instance.

Build and run from the root directory. Open your web browser to: http://localhost. You should see the welcome page.

Navigate to the login page, and then to the register page. Create a new user and you should be able to login. That's it.

## Overview

The web app has a public home page, authenticated home page, login page, register page, and about page. 

The entrypoint for the web app is gowebapp.go. The file loads the initial configuration, 
starts the session, connects to the database, precompiles all the templates, assigns 
the routes, attaches the middleware, and starts the web server.

The front end is built using Foundation with a few small changes to fonts and spacing. The flash 
messages are customized so they show up at the bottom right of the screen.

All of the error and warning messages should be either displayed either to the 
user or in the console. Informational messages are displayed to the user via 
flash messages that disappear after 5 seconds. The flash messages are controlled 
by javascript in the static folder.

## Structure

The project is organized into the following folders:

~~~
config		- config values and matching structs
controller	- page logic organized by HTTP methods (GET, POST)
database	- sql file for database creation and matching structs
middleware	- router middleware for logging and debugging
plugin		- funcs that are usable in templates
shared		- packages for templates, MySQL, cryptography, sessions, and json
static		- location of statically serve files like CSS and JS
template	- HTML templates
~~~

There are a few external packages:

~~~
github.com/gorilla/context				- registry for global request variables
github.com/gorilla/sessions				- cookie and filesystem sessions
github.com/go-sql-driver/mysql 			- MySQL driver
github.com/jmoiron/sqlx 				- MySQL general purpose extensions
github.com/josephspurrier/csrfbanana 	- CSRF protection for gorilla sessions
github.com/julienschmidt/httprouter 	- high performance HTTP request router
golang.org/x/crypto/bcrypt 				- password hashing algorithm
~~~

The templates are:

~~~
about.tmpl		- quick info about the app
anon_home.tmpl	- public home page
auth_home.tmpl	- home page once you login
footer.tmpl		- footer
index.tmpl		- base template for all the pages
login.tmpl		- login page
menu.tmpl		- menu at the top of all the pages
register.tmpl	- register page
~~~

## Templates

There are a few template funcs that are available to make working with the templates 
and static files easier:

~~~ html
<!-- CSS files with timestamps -->
{{CSS "static/css/normalize3.0.0.min.css"}}
parses to
<link rel="stylesheet" type="text/css" href="/static/css/normalize3.0.0.min.css?1435528339" />

<!-- JS files with timestamps -->
{{JS "static/js/jquery1.11.0.min.js"}}
parses to
<script type="text/javascript" src="/static/js/jquery1.11.0.min.js?1435528404"></script>

<!-- Same page hyperlinks -->
{{LINK "register" "Create a new account."}}
parses to
<a href="/register">Create a new account.</a>
~~~

There are a few variables you can use in templates as well:

~~~ html
<!-- Use AuthLevel=auth to determine if a user is logged in -->
{{if eq .AuthLevel "auth"}}
You are logged in.
{{else}}
You are not logged in.
{{end}}

<!-- Use BaseURI to print the base URL of the web app -->
<li><a href="{{.BaseURI}}about">About</a></li>

<!-- Use token to output the CSRF token in a form -->
<input type="hidden" name="token" value="{{.token}}">
~~~

## Controllers

Access a gorilla session:

~~~ go
// Get the current session
sess := session.Instance(r)
...
// Close the session after you are finished making changes
sess.Save(r, w)
~~~

Trigger a flash message on the next page load:

~~~ go
sess.AddFlash(view.Flash{"Sorry, no brute force :-)", view.FlashNotice})
sess.Save(r, w) // Ensure you save the session after making a change to it
~~~

Validate form fields are not empty:

~~~ go
// Ensure a user submitted all the required form fields
if validate, missingField := view.Validate(r, []string{"email", "password"}); !validate {
	sess.AddFlash(view.Flash{"Field missing: " + missingField, view.FlashError})
	sess.Save(r, w)
	LoginGET(w, r)
	return
}
~~~

Render a template:

~~~ go
// Create a new view
v := view.New(r)

// Set the template name
v.Name = "login"

// Assign a variable that is accessible in the form
v.Vars["token"] = csrfbanana.Token(w, r, sess)

// Refill any form fields from a POST operation
view.Repopulate([]string{"email"}, r.Form, v.Vars)

// Render the template
v.Render(w)
~~~

## Configuration

To make the web app a little more flexible, you can make changes to different 
components in one place through the config.json file. If you want to add any 
of your own settings, you can add them to config.json and then to config.go so 
you can reference them in your code. This is config.json:

~~~ json
{
	"Server": {
		"Hostname": "",
		"Port": 80
	},
	"Session": {
		"SecretKey": "@r4B?EThaSEh_drudR7P_hub=s#s2Pah",
		"Name": "gosess",
		"Options": {
			"Path": "/",
			"Domain": "",
			"MaxAge": 28800,
			"Secure": false,
			"HttpOnly": true
		}
	},
	"View": {
		"BaseURI": "/",
		"Extension": "tmpl",
		"Folder": "template",
		"Name": "blank",
		"Caching": true
	},
	"MySQL": {
		"Username": "root",
		"Password": "",
		"Database": "webframework",
		"Hostname": "127.0.0.1",
		"Port": 3306,
		"Parameter": "?parseTime=true"
	},
	"Template": {
		"Root": "index",
		"Children": [
			"menu",
			"footer"
		]
	}
}
~~~