# Using Carbon in Rails 8

## 集成 Carbon Web Components

#### Step 1: 導入 web components .js 文件。

把製作好的 .js 文件複製到 Rails app 的 vender/javascript 文件夾中。

````bash
appname/vender/javascript/carbon/components.js # 製作好的合一的 Web Components .js 文件。
````

#### Step 2: 配置 config/importmap.rb

```ruby
pin "@carbon/components", to: "carbon/components.js"
```

#### Step 3: 配置 app/javascript/application.js

```javascript
import "@carbon/components";
```

#### Step 4: 配置 application.html.erb

```erb
<body class="cds-theme-zone-g90">
```

## 集成 Styles

```ruby
appname/app/assets/stylesheets/application.css # 
appname/app/assets/stylesheets/themes.css # 這是 Carbon 的所有 themes，min到一個.css文件裡。
appname/app/assets/stylesheets/styles.css # 這是 Carbon 的所有 styles，min到一個.css文件裡。
appname/app/assets/stylesheets/fonts.css # 設置 font 文件的路徑而已。
```

### Carbon Global Styles

雖然文檔說 web components 的 .js 文件中已經內嵌了 styles，但我發現，如果不引用 styles.css，各個component的字體就顯示地不對。

### Customise Carbon styles

As Shadow DOM (one of the Web Components specs that `@carbon/web-components` uses) promises, ==styles that `@carbon/web-components` defines does not affect styles in your application, or vice versa==.

However, in cases where your application or a Carbon-derived style guide wants to change the styles of our components, use CSS custom properties.

@TODO

### Themes

To make the whole Rails app looks in a Carbon theme, follow two steps:

**Step 1**: Download the latest `theme.css` file from the official Carbon CDN, and put it in `app/assets/stylesheets/`.

The following includes classes for creating Carbon theme zones (`white`, `g10`, `g90`, `g100`). Note that these classes take advantage of using CSS Custom Properties enabled in Carbon.

```html
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/version/[v2.x.y]/themes.css" />
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/tag/latest/themes.css" />
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/tag/next/themes.css" />
```

**Step 2**: In `app/views/layouts/application.html.erb`:

```html
<body class="cds-theme-zone-white">
	<%= yield %>
</body>
```

The other themes are: `cds-theme-zone-g100`, `cds-theme-zone-g90`, `cds-theme-zone-g10`.  

Done!

Of course, ==a theme can be used on an area or setcton of a page, or even just to a certain component==.

```html
<html>
<head>
	<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/tag/latest/themes.css" />
</head>
<body>
	<cds-dropdown value="bar" class="cds-theme-zone-white">
		<cds-dropdown-item value="foo">Foo</cds-dropdown-item>
		<cds-dropdown-item value="bar">Bar</cds-dropdown-item>
		<cds-dropdown-item value="baz">Baz</cds-dropdown-item>
	</cds-dropdown>
	<div class="cds-theme-zone-g100" style="background: var(--cds-background)">
		<cds-button kind="tertiary">button</cds-button>
		<cds-tag filter type="red" title="Clear selection">This is a tag</cds-tag>
    </div>
</body>
</html>
```

### Carbon Grid

The following includes Carbon grid (using flex grid) and all corresponding grid classes.

```html
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/version/[v2.x.y]/grid.css" />
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/tag/latest/grid.css" />
<link rel="stylesheet" href="https://1.www.s81c.com/common/carbon/web-components/tag/next/grid.css" />
```

[Learn more about the Carbon 2x Grid](https://carbondesignsystem.com/guidelines/2x-grid/overview)

## 集成 IBM Plex fonts and fonts.css



```ruby
appname/app/assets/fonts/IBMPlexMono-Regular.woff2 # 字體文件。
```

### fonts.css

字體文件，放在 app/assets/fonts/文件夾下。

不知為何，如果在`@font-face`中如下寫，經常出現404錯誤。% 在我新建的bos中，沒有再遇到這種情況。20250503 %

```css
@font-face {
	src: asset_path("fonts/IBMPlexMono-Regular.woff2") format("woff2");
}
```

所以，我使用 `asset_path()` 來規避這個路徑問題，實測有效。

```css
@font-face {
	font-family: "IBM Plex Mono";
	font-style: normal;
	font-weight: 400;
	src: asset_path("fonts/IBMPlexMono-Regular.woff2") format("woff2");
}

@font-face {
	font-family: "IBM Plex Sans";
	font-style: normal;
	font-weight: 400;
	src: asset_path("fonts/IBMPlexSans-Regular.woff2") format("woff2");
}

@font-face {
	font-family: "IBM Plex Sans";
	font-style: bold;
	font-weight: 700;
	src: asset_path("fonts/IBMPlexSans-Bold.woff2") format("woff2");
}
@font-face {
	font-family: "IBM Plex Sans SC";
	font-style: normal;
	font-weight: 400;
	src: asset_path("fonts/IBMPlexSansSC-Regular.woff2") format("woff2");
}
@font-face {
	font-family: "IBM Plex Sans SC";
	font-style: bold;
	font-weight: 600;
	src: asset_path("fonts/IBMPlexSansSC-Bold.woff2") format("woff2");
}
```

## 集成 Carbon icons



所有 Carbon Icons，大约2000多个。合并为一个文件icons.svg，大概有1MB大小。

### 加載和引用

先把 icons.svg 文件放在：

```ruby
appname/app/assets/images/icons.svg 
```

然後在 application.html.erb 中引用之：

```erb
<link rel="preload" as="image" href="<%= asset_path('icons.svg') %>" type="image/svg+xml">
```

### 使用一個icon

```html
<cds-button kind="primary">
	<svg slot="icon" aria-hidden="true"><use xlink:href=<%= asset_path('icons.svg#edit') %>></use></svg>Edit
</cds-button>
```

我把`<svg slot="icon”>`的多種屬性，都移動進了`application.css`中，這樣在ERB或HTML中可以少些代碼。 `aria-hidden="true"`只能放在HTML中。

```css
svg[slot="icon"] {
	width: 16px;
	height: 16px;
	fill: currentColor;
}
```

其中，`fill: currentColor;` 讓icon的顏色與文字的顏色保持一致。這一句非常重要。

TODO：我想寫一個 IconHelper，能方便地在ERB中使用一個 Carbon icon。例如
```erb
<%= icon "edit" size:16  color:red %>
```

## Mapping rails view helpers to Carbon web components

### Rails View Helpers and Their Mappings

Rails view helpers simplify HTML generation in views, and many have corresponding CDS web components for a consistent design. Below, we map these helpers to their standard HTML elements and CDS equivalents.

#### Button and Link Helpers

- `button_to`: Generates a form with a `<button>` for form submissions, mapping to `<form><cds-button></form>` from CDS, as `<cds-button>` is the component for buttons, and the form wrapper remains standard HTML. This is supported by the user’s mention of using `carbon_tag "button"`, generating `<cds-button>`, confirming the mapping.
- `link_to`: Generates an `<a>` tag for links, mapping to `<cds-link>` from CDS, as `<cds-link>` is the component for hyperlinks, aligning with standard HTML `<a>`.

#### Form Input Helpers

- `text_field_tag`: Generates `<input type="text">`, mapping to `<cds-text-input>` from CDS, as `<cds-text-input>` is designed for text inputs, with attributes like `value` and `placeholder` supported, corresponding to standard HTML `<input type="text">`.
- `password_field_tag`: Generates `<input type="password">`, mapping to `<cds-text-input kind="password">`, as Carbon’s `<cds-text-input>` supports a `kind` attribute for password fields, aligning with standard HTML `<input type="password">`.
- `check_box_tag`: Generates `<input type="checkbox">`, mapping to `<cds-checkbox>`, as `<cds-checkbox>` is the component for checkboxes, corresponding to standard HTML `<input type="checkbox">`.
- `radio_button_tag`: Generates `<input type="radio">`, mapping to `<cds-radio-button>`, as `<cds-radio-button>` is the component for radio buttons, aligning with standard HTML `<input type="radio">`.
- `select_tag`: Generates `<select>`, mapping to `<cds-select>`, as `<cds-select>` is the component for dropdowns, corresponding to standard HTML `<select>`.
- `text_area_tag`: Generates `<textarea>`, mapping to `<cds-textarea>`, as `<cds-textarea>` is the component for text areas, aligning with standard HTML `<textarea>`.
- `file_field_tag`: Generates `<input type="file">`, mapping to `<cds-file-uploader>`, as `<cds-file-uploader>` is the component for file uploads, corresponding to standard HTML `<input type="file">`.

#### Asset Helpers

- `image_tag`: Generates `<img>`, with no direct CDS web component, as Carbon likely relies on standard HTML `<img>` for images, possibly styled with utility classes. This is an unexpected detail, as most form elements have direct mappings, but images do not, requiring standard HTML with Carbon styling.

#### General Tag Helpers

- ==`content_tag`: Can generate any HTML tag==, so the mapping depends on the tag specified, but for specific tags like `<div>` or `<span>`, there might not be direct CDS components, remaining as standard HTML.

#### Table: Mapping Rails View Helpers to Carbon Web Components and HTML Elements

To organize the findings, here’s the table of mappings:

| Rails Helper       | Standard HTML Element   | Carbon Web Component             | WHL      |
| ------------------ | ----------------------- | -------------------------------- | -------- |
| button_to          | <form><button></form>   | <form><cds-button></form>        | /        |
| link_to            | <a>                     | <cds-link>                       | link     |
| text_field_tag     | <input type="text">     | <cds-text-input>                 | input    |
| password_field_tag | <input type="password"> | <cds-text-input kind="password"> | /        |
| check_box_tag      | <input type="checkbox"> | <cds-checkbox>                   | checkbox |
| radio_button_tag   | <input type="radio">    | <cds-radio-button>               | radio    |
| select_tag         | <select>                | <cds-select>                     | /        |
| text_area_tag      | <textarea>              | <cds-textarea>                   | textarea |
| file_field_tag     | <input type="file">     | <cds-file-uploader>              | file     |

This table includes the main helpers, with form helpers like `text_field_tag` used for tag-based forms, and notes the unexpected detail for `image_tag`, where there’s no direct CDS component, relying on standard HTML.

#### Considerations and Limitations

- Form Builder Helpers**: For model forms, helpers like `f.text_field` (on a form builder) would map similarly, but the table uses tag helpers for consistency, as they’re more direct. The user can extend mappings for form builders based on this.
- **Accessibility**: Carbon components include built-in accessibility features, like labels for `<cds-text-input>`, which might differ from standard Rails, requiring adjustments in helper implementation, as noted in earlier discussions.
- **Scalability**: The mappings cover common helpers, but for less frequent components, users can use `carbon_tag` directly, maintaining flexibility, as seen in the user’s previous approach.

## My Experience

寫一個 `CarbonTagHelper` module，放在`config/initializers/carbon_tag_helper.rb`。在這裡面， override Rails form tags。有的容易覆蓋，有的不容易。

form.submit，不容易修改，因為 shadow DOM 的關係。直接把ERB中的代碼替換成 form.button 就行了。因為 button 是容易用 cds-button 替代的。

input，在 form.text_field 中加 class: "cds-text-input" 就已經產生完全一樣的效果了。在 app/javascript/application.js 中給所有的 input 都應用 class="cds-text-input"，就全局有效。

form 本身不能用 cds-form 替代，因為生成的DOM內在結構會無法提交。

沒有必要完全保留 Rails tag 的實現方式。有些實現，也並不好，例如 form.submit 用inpu[type="submit"]就太老舊了。還好，可以用 `<button>`實現。



### collection_select and cds-select



Carbon doesn’t show a “prompt” by inserting a fake option when you’re using the Web Component <cds-select>. Instead, you should set the placeholder attribute on <cds-select> and make sure the parent has no value until a real choice is made. Carbon docs and examples show <cds-select placeholder="…">…</cds-select>, not a disabled item as a placeholder.

Why your prompt isn’t appearing? You’re trying to inject a <cds-select-item value=""> as a “prompt” and optionally mark it selected/disabled. But <cds-select> manages selection via the parent’s value attribute, so that injected item won’t be treated as a placeholder once value is present (or when the component is looking for a match).

Carbon’s intended pattern for “no selection yet” is placeholder on the select, not a selected/disabled item.

Minimal fix to your module

Make two small changes:
	1.	Set the placeholder when there’s no selected value and a Rails :prompt is given.
	2.	Don’t inject a “prompt item”. Let the placeholder do the job.

module CarbonSelectRenderer
	private

```ruby
def select_content_tag(option_tags, options, html_options)
	html_options = html_options.stringify_keys
	[:required, :multiple, :size].each do |prop|
		html_options[prop.to_s] = options.delete(prop) if options.key?(prop) && !html_options.key?(prop.to_s)
	end

	add_default_name_and_id(html_options)

	if placeholder_required?(html_options)
		raise ArgumentError, "include_blank cannot be false for a required field." if options[:include_blank] == false
		options[:include_blank] ||= true unless options[:prompt]
	end

	current_value = options.fetch(:selected) { value() }

	# If nothing is selected and a prompt exists, use Carbon's placeholder attribute.
	if current_value.blank? && options[:prompt]
		html_options["placeholder"] = prompt_text(options[:prompt])
	end

	# Important: only set `value` on the parent when we actually have one.
	html_options["value"] = current_value if current_value.present?

	select = content_tag("cds-select", add_options(option_tags, options, current_value), html_options)

	if html_options["multiple"] && options.fetch(:include_hidden, true)
		tag("input", disabled: html_options["disabled"], name: html_options["name"], type: "hidden", value: "", autocomplete: "off") + select
	else
		select
	end
end

def add_options(option_tags, options, value = nil)
	# Keep include_blank for a real empty choice the user can pick.
	if options[:include_blank]
		content = (options[:include_blank] if options[:include_blank].is_a?(String))
		label = (" " unless content)
		option_tags = tag_builder.content_tag_string("cds-select-item", content, value: "", label: label) + "\n" + option_tags
	end

	# REMOVE the “prompt item” insertion — placeholder handles it.
	# (No block here.)

	# Convert native <option> to <cds-select-item>
	ActiveSupport::SafeBuffer.new(
		option_tags.gsub(/<option/, '<cds-select-item').gsub(/<\/option>/, '</cds-select-item>')
	)
end
```
end

ActionView::Helpers::Tags::SelectRenderer.prepend CarbonSelectRenderer

How it behaves now
	•	No selection yet + prompt: provided → <cds-select placeholder="Your prompt">…</cds-select> renders the greyed prompt text.
	•	User selects an item → component sets a real value; prompt disappears.
	•	You still want a selectable blank → keep include_blank:; it stays as a real, selectable (empty) option, separate from the placeholder.

Alternative pattern (if you ever need it)

In React Carbon, an empty first item (value="") can produce a default empty selection (no placeholder). That’s a different pattern and not necessary when you have the Web Component placeholder.  ￼

⸻

Quick check list
	•	Don’t set html_options["value"] when you want to show the prompt.
	•	Do set html_options["placeholder"] = prompt_text(...) when value.blank?.
	•	Don’t inject a “prompt” <cds-select-item>; keep include_blank only if you want a real empty choice.
	•	Ensure every real option becomes <cds-select-item value="…">Label</cds-select-item>.

This matches Carbon’s Web Components usage where placeholder is supported on <cds-select>.  ￼
