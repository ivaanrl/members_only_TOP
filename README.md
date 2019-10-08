# README


    Think about and spec out how to set up your data models for this application. You’ll need users with the usual simple identification attributes like name and email and password but also some sort of indicator of their member status. They’ll need to create posts as well. Given what you know about passwords, you’ll be using a :password_digest field instead of a :password field.
    Create your new members-only Rails app and Github repo. Update your README.
    Start by migrating and setting up your basic User model (no membership attributes yet).
    Include the bcrypt-ruby gem in your Gemfile. $ bundle install it. (note: This might just be bcrypt)
    Add the #has_secure_password method to your User file.
    Go into your Rails console and create a sample user to make sure it works properly. It probably looks something like: User.create(:name => "foobar", :email => "foo@bar.com", :password => "foobar", :password_confirmation => "foobar")

    Test the #authenticate command which is now available on your User model (thanks to #has_secure_password) on the command line – does it return the user if you give it the correct password?

    > user = User.create(:name => "foobar", :email => "foo@bar.com", :password => "foobar", :password_confirmation => "foobar")
    > user.authenticate("somethingelse")
    => false
    > user.authenticate("foobar")
    => #<User id: 1, name: "foobar", email: "foo@bar.com", password_digest: "$2a$10$9Lx...", created_at: "2016...", updated_at: "2016...">

Sessions and Sign In

Now let’s make sure our users can sign in.

    Create a sessions_controller.rb and the corresponding routes. Make “sign in” links in the layout as necessary.
    Fill in the #new action to create a blank session and send it to the view.
    Build a simple form with #form_for to sign in the user at app/views/sessions/new.html.erb. Verify that you can see the form.
    We want to remember that our user is signed in, so you’ll need to create a new string column for your User table called something like :remember_token which will store that user’s special token.
    When you create a new user, you’ll want to give that user a brand new token. Use a #before_create callback on your User model to:
        Create a remember token (use SecureRandom.urlsafe_base64 to generate a random string)
        Encrypt that token (with the Digest::SHA1.hexdigest method on the stringified (#to_s) version of your token)
        Save it for your user.
    Create a couple of users to populate your app with. We won’t be building a sign up form, so you’ll need to create new users via the command line. Your #before_create should now properly give each newly created user a special token.
    Now fill in the #create action of your SessionsController to actually create the user’s session. The first step is to find the user based on their email address and then compare the hash of the password they submitted in the params to the hashed password stored in the database (using #authenticate). See Chapter 8 with questions but try not to immediately copy verbatim – you’re doing this to learn.
    Once you’ve verified that the user has submitted the proper password, sign that user in.
    Create a new method in your ApplicationController which performs this sign in for you. Give the user a new remember token (so they don’t get stolen or stale). Store the remember token in the user’s browser using a cookie so whenever they visit a new page, we can check whether they are signed in or not. Use the cookies.permanent “hash” to do this.
    Create two other helpful methods in your ApplicationController – one to retrieve your current user (#current_user) and another to set it (#current_user=(user)). Retrieving your current user should use the ||= operator – if the current user is already set, you should return that user, otherwise you’ll need to pull out the remember token from the cookie and search for it in your database to pull up the corresponding user. If you can’t find a current_user, return nil.
    Set your current user whenever a user signs in.
    Build sign out functionality in your SessionsController#delete action which removes the current user and deletes the remember token from the cookie. It’s best if you make a call to a method (e.g. #sign_out) in your ApplicationController instead of just writing all the functionality inside the SessionsController.
    Create a link for signing out which calls the #delete method of your session controller. You’ll need to spoof the DELETE HTTP method, but that’s easily done by passing #link_to the option :method => :delete.

Authentication and Posts

Let’s build those secrets! We’ll need to make sure only signed in users can see the author of each post. We’re not going to worry about editing or deleting posts.

    Create a Post model and a Posts controller and a corresponding resource in your Routes file which allows the [:new, :create, :index] methods.
    Atop your Posts Controller, use a #before_action to restrict access to the #new and #create methods to only users who are signed in. Create the necessary helper methods in your ApplicationController.
    For your Posts Controller, prepare your #new action.
    Write a very simple form in the app/views/posts/new.html.erb view which will create a new Post.
    Make your corresponding #create action build a post where the foreign key for the author (e.g. user_id) is automatically populated based on whichever user is signed in. Redirect to the Index view if successful.
    Fill out the #index action of the PostsController and create the corresponding view. The view should show a list of every post.
    Now add logic in your Index view to display the author’s name, but only if a user is signed in.
    Sign in and create a few secret posts.
    Test it out – sign out and go to the index page. You should see a list of the posts but no author names. Sign in and the author names should appear. Your secrets are safe!

This is obviously a somewhat incomplete solution… We currently need to create new users from the Rails console. But it should give you some practice figuring out authentication. Feel free to improve it… maybe you’ll be the next SnapChat.

