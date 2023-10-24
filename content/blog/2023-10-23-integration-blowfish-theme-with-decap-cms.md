---
title: Integrating DecapCMS with Hugo and Blowfish Theme, Part 1
date: 2023-10-24T16:30:37.705Z
description: The journey of making this blog ;)
draft: false
showHero: true
thumbnail: /img/blowfish.jpeg
tags:
  - tech
  - blogging
  - hugo
  - decapcms
---
This is the story of how I integrated Blowfish theme in Hugo with DecapCMS. 

I love how Hugo builds in nano seconds and serves static HTML without extra JavaScript or network calls to a database. It offers improved SEO and a better user experience. What I don't like is that to make changes, I would often need to open up VIM on the computer with my Git repo. I want to be able log into an admin site and make edits and posts from a web browser a la WordPress style. But what I *don't* want is to manage a MySQL instance and keep all my site data in MySQL - or any database for that matter (other than git, technically a database 😅). This is where DecapCMS comes in.

{{< github repo="decaporg/decap-cms" >}}

DecapCMS, formerly NetlifyCMS, is a git based CMS. It offers a web interface to commit markdown files to my git repo - and generate my site, using Git as the single source of truth. DecapCMS [has instructions for integrating with Hugo](https://decapcms.org/docs/hugo/), but with Hugo there's also a need to integrate with whatever hugo theme your using. I'm using Blowfish theme:

{{< github repo="nunocoracao/blowfish" >}}

### Choosing The Feature Image 

Many Hugo theme expect you to make a directory for each blog post, and then put the markdown in an `index.md` folder and put whatever featured image you're using for the blog post in that directory. Here is a link to[ how Blowfish says to make a thumbnail](https://blowfish.page/docs/thumbnails/). Basically, they say to make the blog post into a directory and name an image "featured.jpg" or whatever image extension. Like this:

```
content
└── awesome_article
    ├── index.md
    └── featured.png
```

But DecapCMS makes posts by committing a file. And I want to upload featured images via DecapCMS. I also prefer images that I can reuse based on my needs, I don't want them to be based on a specific location.

So step one is overriding Blowfish's way to find a featured image. To override a theme in Hugo, you find the related HTML template in the theme - and you copy it to your `layouts/_default` folder. For example, I discovered that the partial html folder in Blowfish responsible for the featured image in the blog post's hero section is `themes/blowfish/layouts_default/partials/hero/basic.html` so I make a `layouts_default/partials/hero/basic.html` file in my root directory. I copy whatever is in the theme's html - so I can make changes.

This is the contents of the original `basic.html` file:

```go
{{ $disableImageOptimization := .Page.Site.Params.disableImageOptimization | default false }}

{{- $images := .Resources.ByType "image" -}}
{{- $featured := $images.GetMatch "*feature*" -}}
{{- if not $featured }}
  {{ $featured = $images.GetMatch "{*cover*,*thumbnail*}" }}
{{ end -}}
{{- with $featured -}}
    {{ if $disableImageOptimization }}
        {{ with . }}
            <div 
              class="w-full h-36 md:h-56 lg:h-72 single_hero_basic nozoom" 
              style="background-image:url({{ .RelPermalink }});"></div>
        {{ end }}
    {{ else }}
        {{ with .Resize "1200x" }}
            <div 
              class="w-full h-36 md:h-56 lg:h-72 single_hero_basic nozoom" 
              style="background-image:url({{ .RelPermalink }});"></div>
        {{ end }}
    {{ end }}
{{- end -}}
```

The template does the following:

1. Checks for the images in the local directory relative to **current page.** See the [Hugo docs here](https://gohugo.io/content-management/page-resources/) to see how.
2. If there's an image file with the `*feature*` in the name it uses that as the image.
3. If not it looks for images with cover or thumbnail in the name.
4. It uses that image with image optimization, unless image optimization has been disabled.

Here are the changes to make it show an image from the asset folder of my choosing.

```go
{{ $disableImageOptimization := .Page.Site.Params.disableImageOptimization |
  default false
}}
{{ $thumbnail := .Params.thumbnail }}
{{ $featured := resources.Get $thumbnail }}
{{- if not $featured }}
  {{ with .Site.Params.defaultFeaturedImage }}
    {{ $featured = resources.Get . }}
  {{ end }}
{{ end -}}
{{- with $featured -}}
  {{ if $disableImageOptimization }}
    <div
      class="w-full h-36 md:h-56 lg:h-72 single_hero_basic nozoom"
      style="background-image:url({{ .RelPermalink }});"
    ></div>
  {{ end }}
{{ else }}
  {{ with .Resize "1200x" }}
    <div
      class="w-full h-36 md:h-56 lg:h-72 single_hero_basic nozoom"
      style="background-image:url({{ .RelPermalink }});"
    ></div>
  {{ end }}
{{- end -}}
```

I'm getting the string of whatever appears in my front matter under the key of `thumbnail` and using that to get the image from my `assets` directory. [See the Hugo docs here](https://gohugo.io/content-management/image-processing/#global-resource) to see how `resources.Get` does that. If nothing is set as `thumbnail`, I use the default featured image set for the site.

Here is the config of the DecapCMS admin config.yaml to allow editing the blog post from with a featured image from DecapCMS:

```yaml
backend:
  name: git-gateway
  branch: master
media_folder: assets/img
public_folder: /img
collections:
  - name: "blog"
    label: "Blog"
    folder: "content/blog"
    create: true
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    editor:
      preview: false
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Publish Date", name: "date", widget: "datetime" }
      - { label: "Description", name: "description", widget: "string" }
      - { label: "Draft", name: "draft", widget: "boolean", default: true }
      - label: "Show Hero"
        name: "showHero",
        widget: "boolean",
        default: true,
      - { label: "Body", name: "body", widget: "markdown" }
      - label: "Featured Image"
        name: "thumbnail"
        widget: "image"
        choose_url: true
        media_library:
          config:
            multiple: false
      - label: "Tags"
        name: "tags"
        widget: "list"
        default: ["news"]
```

Now the featured image appearing in the hero section of post will be whatever we set as the featured image in DecapCMS!!! Hurray!!

Now we just have to rinse and repeat wherever else the feature image is set in the theme!