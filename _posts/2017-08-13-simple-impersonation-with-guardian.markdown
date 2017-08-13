---
layout: post
title: Simple impersonation with Guardian
modified:
share: true
categories: blog
excerpt: Manage multiple sessions and impersonate as other users with Guardian
comments: true
tags: [elixir, guardian, phoenix, impersonate, session]
date: 2017-08-13T18:00:00+01:00
---


Hi there! Today I would like to write about how to add the feature of impersonation in your Phoenix app. In this post we won't talk about how to add Guardian login. I'm going to assume that you already have Guardian in your app.

## What's Guardian:

> An authentication framework for use with Elixir applications.

You can read about it [here](https://github.com/ueberauth/guardian){:target="_blank"}

## The approach:

The main idea it's pretty simple: Guardian allow us to have multiple sessions. So we'll take in advantage this feature and we're going to create a couple of sessions, with different key, every time a user do login. Thereby when we perform an impersonation we'll change the current user, but we'll maintain the second session untouched, in order to know who we were before.

## The code:

### Main idea

The first of all, that cover the main idea, it's to add in your `create` method of your `sessions_controller.ex`, a new sign in with a custom key. In this example I'm going to use `impersonated_user`:

~~~ elixir
def create(conn, params = %{}) do
  conn
  |> put_flash(:info, "Logged in.")
  |> Guardian.Plug.sign_in(user)
  |> Guardian.Plug.sing_in(user, :access, key: :impersonated_user)
  |> redirect(to: user_path(conn, :index))
end
~~~

Also, we need to «activate» this new key in our pipelines. So you need to go to your `router.ex` file, and edit the `browser_session` pipeline. It should look like this:

~~~ elixir
pipeline :browser_session do
  plug Guardian.Plug.VerifySession
  plug Guardian.Plug.LoadResource
  plug Guardian.Plug.VerifySession, key: :impersonated_user
  plug Guardian.Plug.LoadResource, key: :impersonated_user
end
~~~

(Also you can create a new pipeline for `impersonated_user` and add it wherever you need)


With this, in our `Guardian.Plug` resource, we have a couple of JWT with the user info, but differentiated by its key.

You can access to the info with:

~~~ elixir
Guardian.Plug.current_resource(conn)
Guardian.Plug.current_resource(conn, :impersonated_user)
~~~

The next thing we need to do, it's to store this info into the connection assigns. In order to be able to access it by `conn.assigns.impersonated_user`. For this pourpose, we're going to create a `Plug`, in our plugs folder.

~~~ elixir
defmodule App.Plug.ImpersonatedUser do
  import Plug.Conn

  def init(opts), do: opts

  def call(conn, _opts) do
    impersonated_user = Guardian.Plug.current_resource(conn, :impersonated_user)
    assign(conn, :impersonated_user, impersonated_user)
  end
end
~~~

Now we need to add this `Plug` to our pipelines. For example, in our `authenticated_user`. That manage the user authentication. Once again, in our `router.ex`:

~~~ elixir
pipeline :authenticated_user do
  plug Guardian.Plug.EnsureAuthenticated, handler: App.SessionController
  plug Guardian.Plug.EnsureResource, handler: App.SessionController
  plug App.Plug.CurrentUser
  plug App.Plug.ImpersonatedUser
end
~~~

### Impersonation

Okay! Now we're prepared to make the impersonation. For doing this, we're creating a new controller that will change the current user by the one we want to supplant.

~~~ elixir
defmodule App.ImpersonateController do
  use App.Web, :controller

  alias App.User
  alias App.Repo

  def impersonate(conn, %{"impersonated" => %{"user_id" => user_id}}) do
    user = Repo.get(User, user_id)
    impersonated_user = conn.assigns.current_user

    conn
    |> Guardian.Plug.sign_out
    |> Guardian.Plug.sign_in(user)
    |> Guardian.Plug.sign_in(impersonated_user, :access, key: :impersonated_user)
    |> redirect(to: "/")
  end

  def stop_impersonation(conn, _params) do
    impersonated_user = conn.assigns.impersonated_user

    conn
    |> Guardian.Plug.sign_out
    |> Guardian.Plug.sign_in(impersonated_user)
    |> Guardian.Plug.sign_in(impersonated_user, :access, key: :impersonated_user)
    |> redirect(to: "/")
  end
end
~~~

Easy peasy, right? We are login as the requested user and storing the original/real user in the `impersonated_user`. Thus we're managing 2 sessions at the same time.

The next thing you need to add it's a form that sends the info to our `impersonate` method. Also, you need to add a link to `stop_impersonation`. But I don't want to spread out. I think that the main, and important, idea it's explained xP

### Test!

Yep, tests always matters. I'm going to assume that you already have a `conn_case.ex` or similar, that have a `guardian_login` method. So we need to add the new `:impersonated_user` sing_in in there. After that, our `guardian_login` method should look like this:

~~~ elixir
def guardian_login(%Plug.Conn{} = conn, user, token, opts) do
  conn
    |> bypass_through(App.Router, [:browser])
    |> get("/")
    |> Guardian.Plug.sign_in(user)
    |> Guardian.Plug.sign_in(user, :access, key: :impersonated_user)
    |> send_resp(200, "Flush the session yo")
    |> recycle()
end
~~~

Then, our tests:

~~~ elixir
defmodule App.ImpersonateControllerTest do
  use App.ConnCase
  alias App.Test.Helper

  setup do
    admin = insert(:admin)
    user = insert(:user)
    conn = guardian_login(conn, admin, :token, [])

    {:ok, %{conn: conn, admin: admin, user: user}}
  end

  describe "[POST] impersonate" do
    test "change current user when impersonating", %{conn: conn, admin: admin, user: user} do
      conn = post conn, impersonate_path(conn, :impersonate), impersonated: %{"user_id" => user.id}

      assert Guardian.Plug.current_resource(conn).id == user.id
      assert Guardian.Plug.current_resource(conn, :impersonated_user).id == admin.id
    end
  end

  describe "[GET] stop_impersonation" do
    test "stop impersonation when user is impersonated", %{conn: conn, admin: admin, user: user} do
      conn = post conn, impersonate_path(conn, :impersonate), impersonated: %{"user_id" => user.id}
      conn = get conn, stop_impersonation_path(conn, :stop_impersonation)

      assert Guardian.Plug.current_resource(conn).id == admin.id
      assert Guardian.Plug.current_resource(conn, :impersonated_user).id == admin.id
    end
  end
end
~~~

I wanted to talk about the test because as you can see, the `assert` is against `Guardian.Plug.current_resource` instead of `conn.assigns`. This is because the assigns are set in every request, passing through the pipelines. So, when the server returns the response, no pipeline are executed. And the assigns are not updated with the new info. So we need to assert against the info in the plug. That extracts the info directly from the JWT token, instead of the conn. (I hope I have explained this xD)

And that's all! I hope you find it useful. You can find me on twitter [@enoliglesias](https://twitter.com/enoliglesias){:target="_blank"} for whatever you want to ask x)
