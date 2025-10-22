---
date: '2025-10-22'
lastmod: '2025-10-22'
draft: false
title: "How to Use Turbo Frames in Ruby on Rails: Modal Window Example"
slug: 'turbo-frames-modal-example-ruby-on-rails'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to use Turbo Frames in Ruby on Rails to create a modal window for creating posts. A practical example focusing on the essential Turbo Frame concepts."
keywords: ["Ruby on Rails", "Turbo Frames", "Hotwire", "Modal Window", "Ruby", "Rails 7", "Turbo"]
tags: ["Ruby on Rails", "Turbo", "Hotwire"]
categories: ['Ruby']
hiddenFromHomePage: false
featuredImage: 'logo.jpg'
---

# How to Use Turbo Frames in Ruby on Rails: Modal Window Example

Turbo Frames allow you to update specific parts of a page without reloading the entire page. This guide shows how to create a modal window for creating posts using Turbo Frames.

## What are Turbo Frames?

Turbo Frames let you divide your page into independent sections that can be updated separately. When you click a link or submit a form, only the content inside the matching Turbo Frame gets updated.

## Setup

Create a new Rails project and generate a Posts scaffold:

```bash
rails new turbo_frames_example
cd turbo_frames_example
rails generate scaffold Post title:string body:text
rails db:migrate
```

Rails 7+ includes `turbo-rails` by default.

## Step 1: Add Turbo Frame to Layout

Open `app/views/layouts/application.html.erb` and add the modal frame before `</body>`:

```erb
<!DOCTYPE html>
<html>
  <head>
    <title>TurboFramesExample</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>
  <body>
    <%= yield %>

    <%= turbo_frame_tag :modal %>
  </body>
</html>
```

**Key point**: `turbo_frame_tag :modal` creates a container where Turbo will load content.

## Step 2: Wrap new.html.erb in Turbo Frame

Update `app/views/posts/new.html.erb`:

```erb
<%= turbo_frame_tag :modal do %>
  <div>
    <h1>New Post</h1>
    <%= link_to "Close", posts_path, data: { turbo_frame: "_top" } %>
    <%= render "form", post: @post %>
  </div>
<% end %>
```

**Key points**:
- The `turbo_frame_tag :modal` matches the ID in the layout
- `data: { turbo_frame: "_top" }` on the close link navigates the entire page (exits the frame)

## Step 3: Update the Form

Modify `app/views/posts/_form.html.erb`:

```erb
<%= form_with(model: post, data: { turbo_frame: '_top' }) do |form| %>
  <% if post.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>
      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :body %>
    <%= form.text_area :body %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

**Key point**: `data: { turbo_frame: '_top' }` on the form ensures that after saving, the entire page redirects (not just the frame).

## Step 4: Add Link to Open Modal

Update `app/views/posts/index.html.erb`:

```erb
<h1>Posts</h1>

<%= link_to "New Post", new_post_path, data: { turbo_frame: 'modal' } %>

<div id="posts">
  <% @posts.each do |post| %>
    <div>
      <h3><%= post.title %></h3>
      <p><%= post.body %></p>
      <%= link_to "Show", post %>
      <%= link_to "Edit", edit_post_path(post) %>
      <%= button_to "Delete", post, method: :delete %>
    </div>
  <% end %>
</div>
```

**Key point**: `data: { turbo_frame: 'modal' }` tells Turbo to load the response into the `:modal` frame instead of replacing the entire page.

## How It Works

1. User clicks "New Post" link with `data: { turbo_frame: 'modal' }`
2. Turbo intercepts the click and makes an AJAX request to `/posts/new`
3. From the response, Turbo extracts content inside `<turbo-frame id="modal">...</turbo-frame>`
4. Turbo injects that content into the `turbo_frame_tag :modal` in the layout
5. When the form is submitted with `data: { turbo_frame: '_top' }`, the entire page redirects

## Key Turbo Frame Concepts

### Frame Matching
Turbo matches frames by their ID. The `turbo_frame_tag :modal` in `new.html.erb` must match the one in `application.html.erb`.

### Breaking Out of Frames
- `data: { turbo_frame: "_top" }` - navigates the entire page (breaks out of the frame)
- Without this, the response would only update content inside the current frame

### Targeting Frames
- `data: { turbo_frame: 'modal' }` - loads the response into the specified frame
- Without this, Turbo would replace the entire page

## Optional: Edit in Modal

To also edit posts in a modal, update `app/views/posts/edit.html.erb`:

```erb
<%= turbo_frame_tag :modal do %>
  <div>
    <h1>Edit Post</h1>
    <%= link_to "Close", posts_path, data: { turbo_frame: "_top" } %>
    <%= render "form", post: @post %>
  </div>
<% end %>
```

And add `data: { turbo_frame: 'modal' }` to the edit link in `index.html.erb`:

```erb
<%= link_to "Edit", edit_post_path(post), data: { turbo_frame: 'modal' } %>
```

## Summary

To use Turbo Frames for a modal window:

1. Add `<%= turbo_frame_tag :modal %>` to your layout
2. Wrap the content in your target view with `<%= turbo_frame_tag :modal do %>`
3. Use `data: { turbo_frame: 'modal' }` on links that should open the modal
4. Use `data: { turbo_frame: '_top' }` on forms and links that should exit the modal

That's it! No JavaScript required.

## Additional Resources

- [Turbo Frames Documentation](https://turbo.hotwired.dev/handbook/frames)
- [Hotwire Official Guide](https://hotwired.dev/)
