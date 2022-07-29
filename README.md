## Introduction
This is a [Craft CMS](https://craftcms.com/) plugin for importing content from WordPress to Craft CMS based on Craft's [Feed Me](https://plugins.craftcms.com/feed-me) plugin. 

This plugin is in the development phase, so double-check migrated data before using it on production.

## Requirement
- WordPress 6 with REST API support
  - This plugin is tested with WordPress 6.0. WordPress introduced REST API from 4.7, so technically, it is possible to provide migration functionality from that version
- Enabling basic authentication on the WordPress site
  - WordPress doesn't provide basic authentication, so you can use third parties plugins like [WordPress Rest API Authentication](https://wordpress.org/plugins/wp-rest-api-authentication) to connect to REST API
- WordPress user to read from REST API.
- The latest version of Craft 4

## How migration works
The plugin's home page provides options for migrating WordPress items like users, files, pages, posts, tags, categories, and menus.
 
  <p align="center">
    <img src="https://user-images.githubusercontent.com/55586085/180656687-02849ccf-98aa-4c96-a502-a2865e10b7c5.png" width="700px">
  </p>
  

The plugin fetches WordPress attributes and fields and generates a mapping table for each mentioned item, as shown in the image below.

  <p align="center">
    <img src="https://user-images.githubusercontent.com/55586085/181279010-ced3bc9c-a1aa-4004-a903-e73986090ab7.png" width="1000px">
    <em>Mapping table for WordPress Posts. you can specify Craft's field target -that field can be in table or Matrix-</em>
  </p>
  
After mapping WordPress to Craft fields, this plugin: 
- Creates requested [custom fields](https://craftcms.com/docs/4.x/table-fields.html#settings). 
  - It can be a simple field -Plain text, Dropdown, etc.-, a column in a [table field](https://craftcms.com/docs/4.x/table-fields.html#the-field), or even a field in a [Matrix field](https://craftcms.com/docs/4.x/matrix-fields.html#settings)- 
- Generates feeds based on the Feed-Me plugin.
  - You can run these feeds to migrate data from WordPress to Craft CMS.

### User
These data can be converted to [Craft's users](https://craftcms.com/docs/4.x/users.html):
- User Id
- User link on WordPress site
- User's description, avatar URL, website
- WordPress site URL
- Advanced Custom fields for users

Currently, the user's email is not migrated, so we generate a random email for each user. 
Also default status of migrated users is inactive.

### Media
To migrate media to [Craft assets](https://craftcms.com/docs/4.x/assets.html), follow these steps:
- First of all, create a volume and a file system with the base path set to {@webroot/wp-content/uploads}
- You can add the default Alternative Text element to the volume field layout to migrate the alt of pictures from WordPress to that Craft element later.
- Upload WordPress media folders and their files from the {wp-content/uploads} folder of the WordPress site to the volume created on the first step via FTP.
- Use [Asset index utility](https://docs.craftcms.com/feed-me/v4/guides/importing-assets.html) to index files for that volume
- Use the migrate files option of this plugin to migrate files meta from WordPress to Craft - currently, alt, title, and caption are supported-
  - media Id on the WordPress site, URL, and WordPress site Id are migrated too.
- Optionally, there is a setting you can specify how asset title can be generated.

### Taxonomy
Tags and Categories in WordPress can be converted to Craft's entries, [categories](https://craftcms.com/docs/4.x/categories.html), and [tags](https://craftcms.com/docs/4.x/tags.html).
- Taxonomy Id on WordPress site, name, description, taxonomy URL, and WordPress site Id are migrated.
- Tags in Craft have no hierarchy, so taxonomies with parent->child relationship should be migrated to Craft entries or categories.

### Post and Page
Each post and page can be migrated to [entries](https://craftcms.com/docs/4.x/entries.html). These data are migrated for posts and pages:
- Post status on WordPress (publish, private, draft, ....)
- if a post is password protected
- Post Id
- Post link on WordPress site
- WordPress site URL
- Post title, created time, author, post excerpt, and body -Gutenberg is supported-, tags, and categories
- Advanced Custom fields

Please remember only to delete not needed data after all item migration is completed. We use data like post Id, post link, and ... for the migration process.

#### Gutenberg support
This plugin can migrate posts and page's body in two ways to Craft:
- The whole body is migrated to one text plain/Redactor/CKEditor field
- Each block of Gutenberg is migrated to one block of a Matrix in Craft
  - Gutenberg blocks have no predefined structure, but Craft's Matrix has a structure, so we should map each Gutenberg block to one block type of Matrix.
  - Currently, group blocks are not supported, but you can map the entire content of the group block to one block type.

To migrate a Gutenberg body to Craft's Matrix, do this:
- on the default migration page, make sure 'Migrate Gutenberg blocks to Matrix blocks' is enabled and click on migrate posts or pages
- on the mapping table, enable migration for each Gutenberg block and set the container to {Matrix name}/{Guenberg block name}
  - when you migrate a gallery block/audio/video block, the plugin extract src of gallery items and matches them to converted media migrated on previous steps so you can migrate these blocks to Craft's asset
  - when you migrate a post terms/tag cloud, block the plugin extract link of that taxonomy items and match them to converted taxonomies migrated on previous steps.

  <p align="center">
    <img src="https://user-images.githubusercontent.com/55586085/180813278-a6b5cee8-c738-405e-aa13-0da133880e63.png" width="1000px">
    <em>how to map Gutenberg blocks to Matrix's block type</em>
  </p>

### Menu
This plugin can migrate menus to [Structure Sections](https://craftcms.com/docs/4.x/entries.html#structures) or [Navigation](https://plugins.craftcms.com/navigation) plugin.
- WordPress menu Id and link are migrated.

### Navigation
This plugin can migrate navigation to [Structure Sections](https://craftcms.com/docs/4.x/entries.html#structures) or [Navigation](https://plugins.craftcms.com/navigation) plugin.
- WordPress navigation Id and link are migrated.

When migrating menu and navigation, this plugin tries to convert an internal WordPress link address to Craft internal link. for example, if there is a WordPress link for a post, we try to find that post on Craft -as an entry- and point the link to that entry.

## Supported WordPress plugins:
### Advanced Custom Fields
This plugin supports migrating most [ACF fields](https://wordpress.org/plugins/advanced-custom-fields/) to the Craft core field.

- On the mapping table, we show a list of all Craft's fields for each ACF field. Make sure you select the appropriate Craft field for that ACF field. For example:
  - for the taxonomy field, use Craft's tag field or Craft's category field -depending on the taxonomy type specified in the ACF setting-
  - use Craft's asset field for image and file field
  - use Craft's user field for the ACF user field
- When migrating the Select/Radio button/Checkbox fields, we create all used values on those fields as an option for related Craft's field. 
When running a feed with those fields, run that feed two times. One to create that field with all possible options and then run that feed again for migrating their values.
- You can migrate the Group field this way:
  - for each sub-field of that group, map that sub-field to the same container like {Matrix1/blocktype1} and a different field name like {subfield name}
  <p align="center">
    <img src="https://user-images.githubusercontent.com/55586085/180802749-47355c5c-79ed-4164-b02a-9b7c2dc4ab69.jpg" width="1000px">
    <em>How to map ACF group fields to fields in Matrix's block type</em>
  </p>
- Currently, repeater fields are not tested.

## Supported Craft CMS plugins
supported plugins available in plugin stores are:
- [Redactor](https://plugins.craftcms.com/redactor) // on the mapping table, you can choose to migrate a WordPress attribute/field to Redactor
- [Ckeditor](https://plugins.craftcms.com/ckeditor) // on the mapping table, you can choose to migrate a WordPress attribute/field to Ckeditor

## Plugin Settings
you can add a file named migrate-from-wordpress.php
to config folder of your project and override some settings like this:
```
<?PHP
return [
    'allowHttpWordPressSite' => false,  // by default, the protocol of the WordPress site must be HTTPS. you can allow connecting to the HTTP WordPress site here. - we send sensitive data like your WordPress account, so do it only when the WordPress site is on a local or trusted network-
    'allowReusingToken' => false, // only on dev environment it can be set to true to test feed values quickly. When set to false, the feed URL's token is valid once and regenerates itself.
    'cacheFeedValuesSeconds' => 0, // how many seconds feed's values are cached. the default value is 0, so always cached data is returned. For development purposes, we use 1 to see the latest changes.
];
```
 
- On the reset tab of the plugin setting, you can quickly clear data on Craft CMS and other data related to the migration process to start the migration process again. Resetting data related to Craft CMS is only available on the Dev environment.

## Troubleshooting
when running a feed doesn't return the expected result, try to debug that feed on {admin/feed-me/feeds} page -by clicking on the setting icon and then debug button-. 
If debug returns 'No feed items to process.' copy that feed URL and see what that URL returns on the browser. If there is an error, report that Log on Github issue or send the full Log to vnali.dev@gmail.com.

## License & Pricing
This plugin is licensed under MIT and is free of charge. 

## Donation
If this plugin was useful to you, saved your time, and made the migration process easier for you, you can donate via [this link](https://github.com/vnali/migrate-from-wordpress-plugin-docs/blob/main/donate.md) soon.

## FAQ

> What if I use a WordPress plugin whose data is not migrated via the plugin?

Please let us know which WordPress plugin you are using. If its data is available in REST API, We will work to add support for it.

> Is a multi-language WordPress site supported?

Currently, no, but we write this plugin to support multi-language later. please let us know which plugin you are using for that and if data is
provided on REST API, we will try to add a migration process for it

## ToDo
- [ ] Make some videos about the migration process
- [ ] Test plugin with more real data
- [x] Add a plugin setting to migrate posts with any status like trash items
- [ ] Finish PHPStan level 5
- [ ] Support Super table on migration table
- [x] Add navigation item migration

## Contact
Feel free to contact me by email at vnali.dev@gmail.com or direct message me via 'vnali' on [Craft CMS Discord](https://craftcms.com/discord) channel.
Clear cache if you cannot see the latest changes made in WordPress files
