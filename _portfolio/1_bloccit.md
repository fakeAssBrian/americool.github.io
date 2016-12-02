---
layout: post
title: Bloccit
feature-img: "img/sample_feature_img.png"
thumbnail-path: "img/bloccit_base_screenshot.png"
short-description: A Swell Reddit Clone
---

**Summary:** The goal of this project was to make a fully-functional clone of reddit called “bloccit” using the Ruby on Rails framework. This was my first serious foray into programming, as well as my first attempt at using the Rails framework, CRUD, MVC, Rspec and many other common elements of web-development. This is considered a large part of the foundation of Bloc’s Ruby on Rails curriculum.

**Explanation:** Bloccit works much like basic version of reddit - a massive bulletin board like site which contains numerous pages related to specific topics (subreddits). Each of these topic pages contain numerous user submitted posts about the topic, which are then each open for additional comments from all users. This sample application works in the same way, with the main distinctions that the topics are all listed on a singular page (rather than reddit which has far too many subreddits to list) and that the comments are slightly simpler here (reddit allows reply to comments, replies to replies, and so to infinity, whereas bloccit keeps comments on a single level below the posts.) Additionally, like  reddit, Bloccit allows for users to “up or down vote” each post and topic once, which either increases or decreases it’s overall rating (defined as the number of up-votes minus the number of down votes.)  Bloccit also allows for “favoriting” (which is similar, but not identical to any reddit feature)  on posts, which is more of a generic like, but is stored as visible user data, and allows users to see (on their user page) which content they favorited, acting as something of an indirect bookmark. Finally, not unlike many other similar message boards, Bloccit allows for posts and topics to have many descriptive “labels” which can match and associate it with other posts and topics of a similar subject matter.

**Problem:** Overall the initial creation of this project, (despite my inexperience with the Rails framework,) wasn’t exceptionally challenging. Obviously getting used to the amazing amount of built in functionality in Rails (which often led me to believe I simply wasn’t understanding how the code worked,) small hiccups involving getting my computer to properly handle Ruby, Rails, and all the various dependencies, as well as typos forcing me to constantly clean up and reset my database, all proved to be frustrations that were often time consuming learning experiences, but ultimately weren’t truly code or logic based challenges.  However, as  I moved beyond static pages and the basic layout of models, controllers, and views, certain things became substantially more difficult:


 1.) *Creating functional User classes with proper authentication and sessions: A first (albeit milder) difficulty was creating user classes that had proper authentication (user names, emails, password, and password confirmation for signups) that would properly handle errors in signing in and signing up, as well as keeping track of the current session (which user(s)) were logged in, and how that would effect the users view and what was or was not visible.*

 2.) *Creation of Labels and Polymorphic Many-to-Many relationships: Perhaps the hardest part of this assignment was setting up add understanding polymorphic relationships. This required not the fairly logical parent/child style relationship of Topic and Post (a topic has a post, a post belongs to a topic, they’re linked by a foreign id key, etc.), but rather linking  many labels to many topics and posts, and vice versa.*

 **Solution:** First, to deal with the issue of creating users classes with proper authentication let's look at the code below:

 {% highlight ruby %}

 before_create :generate_auth_token

 EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i

 validates :name, length: {minimum: 1, maximum: 100}, presence: true
 validates :password, presence: true, length: {minimum: 6}, if: "password_digest.nil?"
 validates :password, length: {minimum: 6}, allow_blank: true

 validates :email,
          presence: true,
          uniqueness: {case_sensitive: false},
          length: {minimum: 3, maximum: 100},
          format: {with: EMAIL_REGEX}
 {% endhighlight %}

 Rails offers a pretty simple, but nonetheless robust tool that allows to check (or validate) various class attributes upon the instantiation of a new instance of a class. The email regex is particularly noteworthy, (as regular expressions still make me feel a little nauseous), because it makes checking for validly formatted email addresses relatively simple.

Additionally, we can see a reference to "generate_auth_token" at the top (which is performed whenever a new instance of the user class is created.) This is important and the method itself can be seen down here:

{% highlight ruby %}
def generate_auth_token
    loop do
      self.auth_token = SecureRandom.base64(64)
      break unless User.find_by(auth_token: auth_token)
    end
  end
{% endhighlight %}

This part of code is crucial to creating and saving sessions, this line of code sets a new authentication token for the user, if this token is not found for any other users then the loop breaks, and everything is set. Otherwise, the loop continues and sets a new token.

To further elaborate on this here are three other snippets of code. The first is from the sessions controller:

{% highlight ruby %}
def create
  user = User.find_by(email: params[:session][:email].downcase)

  if user && user.authenticate(params[:session][:password])
    create_session(user)
    flash[:notice] = "Welcome, #{user.name}!"
    redirect_to root_path
  else
    flash.now[:alert] = 'Invalid email/password combination'
    render :new
  end
end
{% endhighlight %}

In the above, we can see that when the user logs in (i.e. creates a new session) we locate the user based on the email address, authenticate based on the password and create a new session.

Here we can see it called in the user controller when a new user is created:

{% highlight ruby %}
def create
    @user = User.new
    @user.name = params[:user][:name]
    @user.email = params[:user][:email]
    @user.password = params[:user][:password]
    @user.password_confirmation = params[:user][:password_confirmation]

    if @user.save
      flash[:notice] = "Welcome to Bloccit #{@user.name}!"
      create_session(@user)
      redirect_to root_path
    else
      flash.now[:alert] = "There was an error creating your account. Please try again."
      render :new
    end
  end
{% endhighlight %}

and lastly we can see where create_session originates in sessions_helper:
{% highlight ruby %}
def create_session(user)
    session[:user_id] = user.id
  end

  def destroy_session(user)
    session[:user_id] = nil
  end

  def current_user
    User.find_by(id: session[:user_id])
  end
{% endhighlight %}

What is not shown here is that because the helper module is included in the main ApplicationController, which all other controllers inherit from, most of the application has access to current_user. This (as you can imagine) keeps track of the current logged in user, and allows a number of different view restrictions. One of the most basic of which is shown below:

{% highlight ruby %}

<div class="pull-right user-info">
      <%if current_user %>
      <%= image_tag current_user.avatar_url(48), class: "gravatar" %>
      <div class="pull-right">
        <%= link_to current_user.name, user_path(current_user) %> <br/> <%= link_to "Sign Out", session_path(current_user), method: :delete %>
      </div>
      <%else%>
        <%= link_to "Sign In", new_session_path %> or
        <%= link_to "Sign Up", new_user_path %>
      <%end%>
    </div>

{% endhighlight %}

This is from the main application html, and this section acts as something of a navbar in each page. As we can see, if the current_user is present (meaning there is a user logged in) it pulls their name, avatar icon, and a link to sign out to the top right corner of the screen. Otherwise, it provides a Sign In/Sign Up set of links which are a path to either a new session or user respectively.  There are of course many other examples that change what is visible, (some which check if there is a current user *AND* weather that user is an admin, or possibly if the user is the author of the post,) but this example is given for simplicity and clarity. (A full link to the application's public github is provided at the bottom of the page.)

While the previous challenge was largely code-based and required little theoretical work, implementing the label feature proved to be the opposite. While rails includes tools that make many-to-many relationships simple to write, properly understanding how the system works with the database is substantially more complex.

The issue is that multiple labels have to be associated with posts and topics and that posts and topics have to be in turn associated with multiple labels. Another way of think of it is that each label may have many topics and/or posts (the label 'dogs' might be used on the topic such as "pets" and the post "Fido the worlds fastest retriever" but the topic might also include other labels such as 'cats' and the post might contain other labels like 'world records'), this is called a Many-to-Many or Polymorphic relationship.  The trick to make this work using the Rails Database Active record is by using what is called a "join-table" which exists exclusively to connect the other tables. Consider this example:

![many-to-many_example](/img/many-to-many.png){:class="img-responsive"}

Here we can imagine that an address can have multiple people (a couple or whole family) or even many businesses (picture a large shared office building), and at the same time, a business could have multiple address (chain locations, branches etc.) and a person certainly could as well (work, home, etc.) It's also fairly easy to see how some of these addresses could overlap. So in order to combine them we have our "join table." In this example, each table has it's descriptors (a string titling it basically,) and it's own unique id to keep track of it in the database. What the join table does is keep track of an associated address_id to link to an address, and has an associative_id which matches to the unique id for either business or person, and is told which it references using the "associative_type" string (business or person in this case.)

In the case of bloccit, Address is replaced with label, the address join table is called "Lableings" (due to a naming convention I believe it's attributes are "lable_id", "labable_id", "labeable_type" - respective to the above example), and Buisness and Person are replaced with Topic and Post (both which have a number of other attributes.) Though it's mostly prebuilt "Rails magic" at work below, I've included some snippets of code to show how it is implemented:


{% highlight ruby %}
class Label < ActiveRecord::Base
  has_many :labelings
  has_many :topics, through: :labelings, source: :labelable, source_type: :Topic
  has_many :posts, through: :labelings, source: :labelable, source_type: :Post

  def self.update_labels(label_string)
    return Label.none if label_string.blank?

    label_string.split(",").map do |label|
      Label.find_or_create_by(name: label.strip)
    end
  end
end
{% endhighlight %}

Above is the label class. As we can see Rails offers a lot of nice short hands. Labels has many labelings (the connector table), as well as many topics and posts, which are connected through labelings.

{% highlight ruby %}
class Labeling < ActiveRecord::Base
  belongs_to :labelable, polymorphic: true
  belongs_to :label
end
{% endhighlight %}

...and in pure rails magic we can see how simply one can create a class that acts as a join table to the database.


**Results:** This project (which is actually much larger in scope than all that is discussed here,) with a little work and help from my mentor and the curriculum ended up fully-functional, as can be seen in various screenshots below:

Here we can see the landing page with the sign in options representing parts of the view visible there is no session present.

![bloccit_screenshot_6](/img/bloccit_6.png){:class="img-responsive"}

Here's the sign in page.

![bloccit_screenshot_5](/img/bloccit_5.png){:class="img-responsive"}

Here's an example of the topic page, we can see that the sign-in option has now been replaced with an avatar, user name, and sign out option because we have a current session.

![bloccit_screenshot_4](/img/bloccit_4.png){:class="img-responsive"}

Don't mind the gibberish, posts were randomly generated by code in the database for testing. (Note the options for up/down votes.)

![bloccit_screenshot_3](/img/bloccit_3.png){:class="img-responsive"}

New post page with various labels added at the bottom

![bloccit_screenshot_1](/img/bloccit_1.png){:class="img-responsive"}

...and here we can see the labels represented as blue bubbles below the title of the post.

![bloccit_screenshot_2](/img/bloccit_2.png){:class="img-responsive"}

**Conclusion:** Overall, until the later sections, this project was not exceptionally difficult, though it was a crucial learning experience. It was an excellent example of how Rails works, and a particularly good example of the type of projects rails is incredibly efficient and creating. It definitely serves as good ground work to future attempts to work within the rails framework and serves as a good basic guide for TDD, working with databases, associating models, and working within the MVC architecture. For future projects, I think it would be valuable to work on something that is less redundant to existing web-based-software, and while perhaps less broad in scope, has a deeper and more valuable exploration of certain features and functionality.

Here is a link to the project's [github page] if you'd like to see the full code.


[github page]: https://github.com/americool/bloccit
