## A beginner's guide to implementing forms in a Rack-based Sinatra application

The following is a summary of information I gathered going through Launch School's RB175 Networked Applications course, which includes a few tutorials of building web applications using the Sinatra framework. The aim was to create beginner-friendly documentation on how to implement forms in a Sinatra app. To get the most use out of this summary, you should know the basics of HTTP request-response cycles and how to build a simple Sinatra app with view templates and routes.

These are the topics covered:
- [HTML elements and attributes needed to send a form](#html-elements-and-attributes-needed-to-send-a-form)
- [Sending a form using a GET request](#sending-a-form-using-a-get-request)
- [Different ways to format the user input field](#different-ways-to-format-the-user-input-field)
- [Simulating input data for testing](#simulating-input-data-for-testing)


### HTML elements and attributes needed to send a form

The minimal requirement for writing a form which will lead to a specific HTTP request being made, is to use a `<form>` element with `method` and `action` attributes. The value of the `method` attribute determines the type of HTTP request made when the form is sent, whereas the value of the `action` attribute determines where the request is sent (i.e., the path part of the route that will be invoked in response to the request). Capturing the input data and submitting the form can be accomplished in different ways. One of the simplest ways is by using an `<input>` element (opening tag only) with a `name` attribute. For example, if you have the following form in a template document:

```html
<form action="/new" method="post">
  <input name="title">
</form>
```

it will render a text box where the user can input information that upon pressing `Enter` will result in submitting the form data. This will be accomplished by sending a POST request which will lead to the following route being invoked from within the application code:

```ruby
post "/new" do
# ...
end
```
Within this route we can access the submitted form data using `params[:title]`.

**How does this work?**

- The `name` attribute of the `<input>` tag defines how the application will identify the input data.
- The submitted data will be captured in a `params` hash accessible inside our Sinatra route.
- The value of the `<input>` element's `name` attribute (`"title"` in the above example) becomes a key in the `params` hash and the input data becomes the value for that key.

### Sending a form using a GET request

Generally, when we think of sending a form within a web app, we associate it with making a POST request. However, there are some scenarios when a GET request is appropriate for a form. Some examples include:

#### 1. Search box

A typical example of this is a search box that takes you to the place in a text where the search item appears. In this example we implement a `<button>` element to submit the form.

```html
<form action="/search" method="get">
 <input name="query">
 <button type="submit">Search</button>
</form>
```

#### 2. Buttons

In this scenario, we have buttons instead of links to send GET requests for a specific resource. This can be useful when the input from user should only consist of a "Yes" or "No".

```html
<p>Ready to play? Click Yes to start the game, No to exit</p>
 <form action="/word" method="get">
   <button>Yes</button>
 </form>
 <form action="/finish" method="get">
   <button>No</button>
 </form>
```

### Different ways to format the user input field

#### 1. Using the `<input>` element
A single row input field can be achieved by using an `<input>` element (opening tag only). A `type` attribute can be added to an `<input>` element that will define what type of data it can accept and the type of input field that will be rendered.

Examples:
- `type=”text”` gives a text box.
- `type=”submit”` gives a submit button.
- `type="password"` gives a text box that renders each user input character as a `*` or a `.`.
- `type="number"` will only allow number-based input.
- `type="range"` will restrict the input to a specific range, can be used with `min`, `max` and `step` attributes.
- `type="checkbox"` will render a checkbox.
- `type="radio"` will render radio buttons.

To group many checkboxes or radio buttons together, assign the same value to the `name` attribute inside each `<input>` element.

To pair an `<input>` element with a `<label>`, add an `id` attribute to the `<input>` that has the same value as the `for` attribute within the `<label>`.

Example:
```html
<form action="/meal" method="post">
  <label for="dinner">What do you want to eat for dinner?</label>
  <br>
  <input type="text" name="food" id="dinner">
</form>
```

To display a default value in the input field before the user enters their input, use a `value` attribute.

Example:
```html
<form action="/new" method="post">
  <input name="title" value="already pre-filled">
</form>
```

#### 2. Using elements other than `<input>`
To be able to write in a longer text, use `<textarea>` instead of `<input>`. The `<textarea>` element can have `rows` and `cols` attributes that will dictate the size of the textbox.

To create a dropdown menu, add `<option>` elements (each with a `value` attribute) embedded inside of a `<select>` element.

Example:
```html
<form method="post" action="/new">
  <label for="activity">What should we do after dinner?</label>
  <select id="activity" name="activity">
    <option value="watch a movie">Watch a movie</option>
    <option value="play cards">Play Cards</option>
    <option value="play board games">Play Board Games</option>
    <option value="read">Read</option>
    <input type="submit">
  </select>
</form>
```

To be able to locate an option from a dropdown menu, use an `<input type="text">` element, followed by a `<datalist>` element with the `<option>` elements embedded inside.

Example:
```html
<form method="post" action="/new">
  <label for="city">Where would you like to live?</label>
  <input type="text" list="cities" id="city" name="city">

  <datalist id="cities">
    <option value="London"></option>
    <option value="Barcelona"></option>
    <option value="Quebec City"></option>
    <option value="Cape Town"></option>
  </datalist>
</form>
```

### Simulating input data for testing
To be able to access form data in a testing suite, build your test class to include Rack methods and define an `app` method that will start the application:

```ruby
require "minitest/autorun"
require "rack/test"

require_relative "path_to_app_file"

class AppTest < Minitest::Test
  include Rack::Test::Methods

  def app
    Sinatra::Application
  end

  # write tests here

end
```

Let's say we have a simple form in a view template:

```html
<form action="/question" method="post">
  <p>What is 1 + 2?</p>
  <input name="answer">
</form>
```

and the corresponding route in your application code:

```ruby
post "/question" do
  if params[:answer] == "3"
    "Correct answer"
  else
    "Wrong answer"
  end
end
```

We would like to test the `post "/question"` route and simulate user input to determine if the correct response is given. Add the following tests to your custom `AppTest` class:

```ruby
def test_wrong_answer
  post "/question", answer: "5"

  assert_equal "5", last_request.params["answer"]
  assert_equal 200, last_response.status
  assert_equal "text/html;charset=utf-8", last_response["Content-Type"]
  assert_includes last_response.body, "Wrong answer"
end

def test_right_answer
  post "/question", answer: "3"

  assert_equal "3", last_request.params["answer"]
  assert_equal 200, last_response.status
  assert_equal "text/html;charset=utf-8", last_response["Content-Type"]
  assert_includes last_response.body, "Correct answer"
end
```

**How does this work?**

Taking a closer look at the source code for [Rack::Test](https://github.com/rack/rack-test/blob/master/lib/rack/test.rb):
```ruby
# Issue a GET request for the given URI with the given params and Rack environment.
# Stores the issued request object in #last_request and the app's response in #last_response.
# Yield last_response to a block if given.
# ...

# Issue a POST request for the given URI. See #get

def post(uri, params = {}, env = {}, &block)
  custom_request('POST', uri, params, env, &block)
end
```

This tells us we can send a `post “/path”` request and pass in the parameters we want.

To access the form data from the request, we first need to access the request:

```ruby
puts last_request
# => #<Rack::Request:0x00007fb36aa83c00>
```
Now when we look inside the `Rack::Request` class, we see a method `#params` that will return exactly what we want:
```ruby
puts last_request.params
# => {"answer"=>"3"}
```

Note: This information is also stored inside the `env` hash in the `Rack::Request` object and we could access it this way:

```ruby
puts last_request.env["rack.request.form_hash"]
# => {"answer"=>"3"}
```

**How does this work?**

- `last_request.env` returns a hash containing information about the request.
- Within this `env` hash is an item `"rack.request.form_hash"=>{"answer"=>"3"}`, where the string `"rack.request.form_hash"` is a key with a value of a hash that contains the data submitted in the form.
