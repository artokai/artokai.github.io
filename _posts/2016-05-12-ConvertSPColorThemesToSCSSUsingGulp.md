---
title: "Convert SharePoint color themes to SCSS using gulp"
header:
  teaser: "posts/2016-05-12_teaser.jpg"
tags:
- SharePoint
- Gulp
---

Color themes provide an easy way to apply lightweight branding in SharePoint. 
And if you're using a custom CSS file, you can reuse the color definitions from by adding
the SharePoint specific ```ReplaceColor``` preprocessing instruction in your CSS file. 
This command will instruct SharePoint to replace the color defined in the CSS file 
with a value from the currently applied color theme.

```css
.myClass {
  /* [ReplaceColor(themeColor:"BodyText")] */ color:#262626;
  padding: 10px;
  font-family: arial, sans-serif; 
}
```

This approach works nicely if you need to use the defined colors inside SharePoint. 
But in some cases you might need to reuse the defined color theme in a CSS
file which is not preprocessed by SharePoint and where those preprocessing
commands therefore will not work.

For these cases I wrote a small gulp plugin which can be used to convert 
a SharePoint's spcolor-file to a SCSS partial which will contain all those 
color definitions as SCSS variables.

```sass
body {
    background-color: $spcolor_PageBackground;
    color: $spcolor_BodyText;
    font-family: arial, sans-serif;
}
```

The utility is licensed under the MIT license and it's available as an npm-package 
called [gulp-spcolor2scss](https://www.npmjs.com/package/gulp-spcolor2scss). 
The source code for the plugin can be found in 
[GitHub](https://github.com/artokai/gulp-spcolor2scss). If you find any problems 
with the plugin, please submit a new issue or even a pull request to the GitHub repository.


## Installation

Install package with NPM and add it to your development dependencies:


```shell
npm install --save-dev gulp-spcolor2scss
```

## Basic usage

### 1. Use gulp to convert your spcolor file to a scss partial

```javascript
'use strict';

var gulp = require('gulp');
var sass = require('gulp-sass');
var header = require('gulp-header');
var spcolor2scss = require('gulp-spcolor2scss');

gulp.task('spcolor', function() {
    return gulp.src('./themes/custom.spcolor', { buffer: true })
        .pipe(spcolor2scss())
        .pipe(header("//\n// This file is autogenerated. Do not edit!\n//\n"))        
        .pipe(gulp.dest('./sass'))        
});

gulp.task('css', ['spcolor'], function() {
    return gulp.src('./sass/**/*.scss')
        .pipe(sass().on('error', sass.logError))
        .pipe(gulp.dest('css'));    
});
```

### 2. Import the generated partial to you sass file

```sass
// Original spcolor file was named 'custom.spcolor'
// It has been renamed to "_custom_spcolor.scss" by the plugin
@import "custom_spcolor";
```

### 3. Use the generated sass variables in your styles

The plugin prefixes all spcolor variable names with 'spcolor_' in order 
to avoid collisions with other variables you might be using.

```sass
body {
    background-color: $spcolor_PageBackground;
    color: $spcolor_BodyText;
    font-family: arial, sans-serif;
}

h1 {
    color: $spcolor_HeaderText;
}
```