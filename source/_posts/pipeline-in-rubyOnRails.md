title: Pipeline in RubyOnRails
date: 2015-08-06 22:14:05
category: RubyOnRails
---
Here is a simple introduction about how to use pipeline in Rails Framework step by step.
<!--more-->

Last week I spent several days on studying on how Rails pipeline works. I鈥檇 like to share you with what I finally found. Maybe we can take its advantage in future release.

There are several steps of applying pipeline into our project:

## Prepare folders

Create folders under "app/assets" for each resource(here each controller is a resource) to store all related static files(js, css, image). For dashboards, we need to create two folders, under "app/assets/javascript" and "app/assets/stylesheet" respectively.
![Alt text](/img/pipeline_001.jpg)

After the creation of the folders is done, we need to move the related js resource and css resource to that folders.

## Create manifest

Create a manifest for each resource to describe its static files. For dashboards, we need to create two manifest files, dashboards.js and dashboards.css
![Alt text](/img/pipeline_002.jpg)

## Config manifest

In the manifest, we need to point out which static files should be compressed into a single file.

+ dashboard.js
![Alt text](/img/pipeline_003.jpg)

+ dashboard.css
![Alt text](/img/pipeline_004.jpg)

## Set the precompile list

All the static files that need to be compressed by pipeline should be specified explicitly in config/application.rb
![Alt text](/img/pipeline_005.jpg)

## Compress static files

Compress all the resource files with the command of *RAILS_ENV=production bin/rake assets:precompile*

After compression, the name of resource file will be changed:
![Alt text](/img/pipeline_006.jpg)

## Refer compressed resource

Add reference to the compressed resource file in application.html.erb. Here is the sample code:
![Alt text](/img/pipeline_007.jpg)

Each page will only load its corresponding resource file. For example, dashboards page will only load dashboards.js and dashboards.css.
	聽
Since the all the js files have been compressed into dashboard.js and all the css file into dashboards.css, it would be sufficient for dashboards page to load only dashboards.js and dashboards.css

## Deal with images

After applying pipeline, all images' name will be changed. So we need to use rails tag to display image.

Firstly, we can use css to present image instead of img tag.

Then, We need to use .css.erb so that we can take the advantage of Rails Helper to convert the image鈥檚 name to its real name.
![Alt text](/img/pipeline_008.jpg)

## Production Model

Switch the web server to production environment with the command of rails server 鈥揺 production

*Now we can access the compressed resource file in production model.*

