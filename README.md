# PostCSS Xcom Styleguide

> Creates a styleguide from YAML-structured CSS-comments. Supports Twig, Jade/Pug, HTML and PHP templates.

## Background
Although there are plenty styleguide generators available, they all seem to only parse markup files. This creates an extra layer of abstraction when used in an environment that relies heavily on templating systems. We don't want to end up having to maintain multiple templates for the same output, so we thought it might be good to have the template function as input for the styleguide. So there it is: PostCSS Xcom Styleguide

## Installation

### PostCSS
This is a plugin for PostCSS so make sure to have PostCSS installed in your product before using this plugin. There are different usages for PostCSS [depending on your build tool or workflow](https://github.com/postcss/postcss#usage).

### PostCSS plugin
You can install the plugin directly from the [NPM registry](https://www.npmjs.com/package/postcss-xcom-styleguide)
```
npm install postcss-xcom-styleguide --save
```

## Getting started
The plugin extracts comments inside your stylesheets which have to be structured as valid YAML-documents. For instance:

```css
/*
---
section: Buttons
title: Primary buttons
demo: styles/modules/buttons/buttons.php styles/data.json
---
There is a description for primary buttons.
It can be cool and even cooler
*/

.button--primary {
    display: inline-block;
    padding: 10px;
    background: blue;
    color: #fff;
    transition: background 250ms;

    &:hover {
        background: red;
    }
}
```

A section is actually a single page inside the styleguide. Different modules from different css-files can still belong to the same sections; they are automatically being aggregated by the plugin. This allows for pages being created that display single modules but could potentially contain full page mockups.

### Parameters

#### section (required)
The name of the section the comment-block belongs to. Specify the same name (case insensitive) across blocks to group them under the same section.

#### title (required)
The title to be displayed for the block inside the section

#### demo (optional)
The demo attribute consists of two parts: <relative/path/to/demo/file> and <relative/path/to/data/object> (optional)

The first argument of the attribute is the path to te actual demo that should showup in the styleguide. Input formats supported are: Twig, Jade/Pug, HTML and PHP. Twig and PHP templates are actually generated by a PHP parser. This parser is mounted by the styleguide once it detects Twig or PHP-templates.

The second argument is the path to a JSON-object that contains mock-data which can feed template data. This is particular useful to recreate data that is not yet available in the styleguide but will be once the templates are used inside a CMS-like environment. The objects can also mock functions that are provided externally by, for example, Processwire. For instance:

```json
{
    "button": {
        "title": "go There"
    },
    "page": {
        "cropImages": [{
            "alt": "image 01",
            "getThumb()": "http://placehold.it/400x300.jpg"
        }, {
            "alt": "image 01",
            "getThumb()": "http://placehold.it/400x300.jpg"
        }]
    },
    "pages": [{
        "title": "page one"
        },{
        "title": "page two"
        }
    ],
    "tree": {
        "build()": "<ul><li>test</li></ul>"
    }
}
```

In the data-object above there are a few mocked function objects that can be used inside templates, e.g. `$page->cropImages->getThumb()` and `$tree->build()`.

Notice:
While the demo attribute is optional, the block will obviously not show up under the styleguides' section while it's empty.

#### description (optional)
Anything outside of the YAML-delimiters (---) is considered a description for the comment block. Github-flavoured markdown is supported inside these descriptions.

## Options
The following options are available inside the plugin

option | default | description | example
--- | --- | --- | ---
title | '' | The title for the styleguide | My Awwesome Styleguide
src | file processed by PostCSS | Optionally specify a file that should be processed (might be handy when compiling multiple files) | wireframes.css
dest | styleguide | Output path for the styleguide | styles
twig.extensions(*) | null | File path (relative to root) pointing to Twig-extensions | extensions.php
php.extensions(*) | null | File path (relative to root) pointing to PHP-extensions | functions.php
include | null | Array with either javascript files or stylesheets that will be included in the styleguide | ['css/master.css', 'js/main.js']

(*) Both PHP and Twig are instantiated without any plugins or function installed. To still make them available during styleguide compilation, they can be provided externally. 

Twig extensions example:
```php
<?php
function myTwigExtension(\Twig_Environment &$twig) {
	$twig->addFunction(new \Twig_SimpleFunction('__', function($data){
    	return $data;
  	}));

	$twig->addFilter(new \Twig_SimpleFilter('strftime',function($item1,$item2){
    	return strftime($item2,$item1);
  	}));
}
```

PHP extensions example:
```php
<?php
function __($text) {
    return $text;
}
```

## Roadmap
- Write functional and unit tests
- Only parse changed comment blocks to improve speed
- Accept array inside options src
