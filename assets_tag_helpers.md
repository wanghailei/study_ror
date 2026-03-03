### Asset Tag Helpers

Asset tag helpers provide methods for generating HTML that link views to feeds, JavaScript, stylesheets, images, videos, and audios. There are six asset tag helpers available in Rails:

- [`auto_discovery_link_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-auto_discovery_link_tag)
- [`javascript_include_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-javascript_include_tag)
- [`stylesheet_link_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-stylesheet_link_tag)
- [`image_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-image_tag)
- [`video_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-video_tag)
- [`audio_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-audio_tag)

You can use these tags in layouts or other views, although the `auto_discovery_link_tag`, `javascript_include_tag`, and `stylesheet_link_tag`, are most commonly used in the `<head>`section of a layout.

The asset tag helpers do *not* verify the existence of the assets at the specified locations; they simply assume that you know what you're doing and generate the link.

#### [3.1.1. Linking to Feeds with the `auto_discovery_link_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-feeds-with-the-auto-discovery-link-tag)

The [`auto_discovery_link_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-auto_discovery_link_tag) helper builds HTML that most browsers and feed readers can use to detect the presence of RSS, Atom, or JSON feeds. It takes the type of the link (`:rss`, `:atom`, or `:json`), a hash of options that are passed through to url_for, and a hash of options for the tag:

```
<%= auto_discovery_link_tag(:rss, {action: "feed"},
  {title: "RSS Feed"}) %>
```



There are three tag options available for the `auto_discovery_link_tag`:

- `:rel` specifies the `rel` value in the link. The default value is "alternate".
- `:type` specifies an explicit MIME type. Rails will generate an appropriate MIME type automatically.
- `:title` specifies the title of the link. The default value is the uppercase `:type` value, for example, "ATOM" or "RSS".

#### [3.1.2. Linking to JavaScript Files with the `javascript_include_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-javascript-files-with-the-javascript-include-tag)

The [`javascript_include_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-javascript_include_tag) helper returns an HTML `script` tag for each source provided.

If you are using Rails with the [Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html) enabled, this helper will generate a link to `/assets/javascripts/` rather than `public/javascripts` which was used in earlier versions of Rails. This link is then served by the asset pipeline.

A JavaScript file within a Rails application or Rails engine goes in one of three locations: `app/assets`, `lib/assets` or `vendor/assets`. These locations are explained in detail in the [Asset Organization section in the Asset Pipeline Guide](https://guides.rubyonrails.org/asset_pipeline.html#asset-organization).

You can specify a full path relative to the document root, or a URL, if you prefer. For example, to link to a JavaScript file `main.js` that is inside one of `app/assets/javascripts`, `lib/assets/javascripts` or `vendor/assets/javascripts`, you would do this:

```
<%= javascript_include_tag "main" %>
```



Rails will then output a `script` tag such as this:

```
<script src='/assets/main.js'></script>
```



The request to this asset is then served by the Sprockets gem.

To include multiple files such as `app/assets/javascripts/main.js` and `app/assets/javascripts/columns.js` at the same time:

```
<%= javascript_include_tag "main", "columns" %>
```



To include `app/assets/javascripts/main.js` and `app/assets/javascripts/photos/columns.js`:

```
<%= javascript_include_tag "main", "/photos/columns" %>
```



To include `http://example.com/main.js`:

```
<%= javascript_include_tag "http://example.com/main.js" %>
```



#### [3.1.3. Linking to CSS Files with the `stylesheet_link_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-css-files-with-the-stylesheet-link-tag)

The [`stylesheet_link_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-stylesheet_link_tag) helper returns an HTML `<link>` tag for each source provided.

If you are using Rails with the "Asset Pipeline" enabled, this helper will generate a link to `/assets/stylesheets/`. This link is then processed by the Sprockets gem. A stylesheet file can be stored in one of three locations: `app/assets`, `lib/assets`, or `vendor/assets`.

You can specify a full path relative to the document root, or a URL. For example, to link to a stylesheet file that is inside a directory called `stylesheets` inside of one of `app/assets`, `lib/assets`, or `vendor/assets`, you would do this:

```
<%= stylesheet_link_tag "main" %>
```



To include `app/assets/stylesheets/main.css` and `app/assets/stylesheets/columns.css`:

```
<%= stylesheet_link_tag "main", "columns" %>
```



To include `app/assets/stylesheets/main.css` and `app/assets/stylesheets/photos/columns.css`:

```
<%= stylesheet_link_tag "main", "photos/columns" %>
```



To include `http://example.com/main.css`:

```
<%= stylesheet_link_tag "http://example.com/main.css" %>
```



By default, the `stylesheet_link_tag` creates links with `rel="stylesheet"`. You can override this default by specifying an appropriate option (`:rel`):

```
<%= stylesheet_link_tag "main_print", media: "print" %>
```



#### [3.1.4. Linking to Images with the `image_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-images-with-the-image-tag)

The [`image_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-image_tag) helper builds an HTML `<img />` tag to the specified file. By default, files are loaded from `public/images`.

Note that you must specify the extension of the image.

```
<%= image_tag "header.png" %>
```



You can supply a path to the image if you like:

```
<%= image_tag "icons/delete.gif" %>
```



You can supply a hash of additional HTML options:

```
<%= image_tag "icons/delete.gif", {height: 45} %>
```



You can supply alternate text for the image which will be used if the user has images turned off in their browser. If you do not specify an alt text explicitly, it defaults to the file name of the file, capitalized and with no extension. For example, these two image tags would return the same code:

```
<%= image_tag "home.gif" %>
<%= image_tag "home.gif", alt: "Home" %>
```



You can also specify a special size tag, in the format "{width}x{height}":

```
<%= image_tag "home.gif", size: "50x20" %>
```



In addition to the above special tags, you can supply a final hash of standard HTML options, such as `:class`, `:id`, or `:name`:

```
<%= image_tag "home.gif", alt: "Go Home",
                          id: "HomeImage",
                          class: "nav_bar" %>
```



#### [3.1.5. Linking to Videos with the `video_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-videos-with-the-video-tag)

The [`video_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-video_tag) helper builds an HTML5 `<video>` tag to the specified file. By default, files are loaded from `public/videos`.

```
<%= video_tag "movie.ogg" %>
```



Produces

```
<video src="/videos/movie.ogg" />
```



Like an `image_tag` you can supply a path, either absolute, or relative to the `public/videos` directory. Additionally you can specify the `size: "#{width}x#{height}"` option just like an `image_tag`. Video tags can also have any of the HTML options specified at the end (`id`, `class` et al).

The video tag also supports all of the `<video>` HTML options through the HTML options hash, including:

- `poster: "image_name.png"`, provides an image to put in place of the video before it starts playing.
- `autoplay: true`, starts playing the video on page load.
- `loop: true`, loops the video once it gets to the end.
- `controls: true`, provides browser supplied controls for the user to interact with the video.
- `autobuffer: true`, the video will pre load the file for the user on page load.

You can also specify multiple videos to play by passing an array of videos to the `video_tag`:

```
<%= video_tag ["trailer.ogg", "movie.ogg"] %>
```



This will produce:

```
<video>
  <source src="/videos/trailer.ogg">
  <source src="/videos/movie.ogg">
</video>
```



#### [3.1.6. Linking to Audio Files with the `audio_tag`](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-audio-files-with-the-audio-tag)

The [`audio_tag`](https://api.rubyonrails.org/v8.1.1/classes/ActionView/Helpers/AssetTagHelper.html#method-i-audio_tag) helper builds an HTML5 `<audio>` tag to the specified file. By default, files are loaded from `public/audios`.

```
<%= audio_tag "music.mp3" %>
```



You can supply a path to the audio file if you like:

```
<%= audio_tag "music/first_song.mp3" %>
```



You can also supply a hash of additional options, such as `:id`, `:class`, etc.

Like the `video_tag`, the `audio_tag` has special options:

- `autoplay: true`, starts playing the audio on page load
- `controls: true`, provides browser supplied controls for the user to interact with the audio.
- `autobuffer: true`, the audio will pre load the file for the user on page load.