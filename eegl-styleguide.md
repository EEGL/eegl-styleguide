# EEGL Styleguide v1.0

## General:
- Use 2 spaces indentations (not 4 spaces) - it’s better for readability and structure

## SCSS:
- use `_file.scss` for imports (underscore prefix means it’s a dependency)
- structure rules - order them by type
    - geometry (display, width, margin, padding, ….)
    - styling (background, border, …)
    - font styling
    - animation
- use Photoshop’s -> copy CSS functions for styling fonts
- use SVGs, if possible, especially for logos & icons. Only vector art can be exported as SVG, don’t export/convert pixel images
- avoid shorthand properties, eg. don’t use `margin: 10px 0 2px;` - these could become chaotic when there are many media queries
- be cautious with floats - there are cross-browser techniques in CSS3 that can achieve the same results, without the annoyance of floating elements (even Bootstrap 4 is moving away from floats)
- never use inline style tags in html, unless there is a valid reason for that. Also use !important sparingly.
- don’t create rules like `margin-left-15`, `push-left-10`, `padding-20`, … and style elements with these classes that apply predefined margin/padding/positioning – this makes very hard to create proper responsive layouts. Create modular classes instead: `class="btn btn-main btn-hero"` and then apply css rules where it makes sense, eg. don’t add `background-color: red` to `.btn`, use .`btn-hero` for that, unless buttons on the site are generally red – so apply rules to short class names (eg. .btn) that are used everywhere on the site. If something is page specific create a secondary rule - `btn btn-about`. `.btn` should mean the same thing on all pages.
- Create a stylebook - `_content.scss` should have a site-wide collection of css rules, that defines the basic look. `h1, h2, h3, a, p` tags, forms, etc… – these should be the same on all pages, so defining them at one place and following the naming convention on all pages result a better & cleaner workflow
- `main.scss` should import all dependencies and hold resets, global modules, sass variables, mixins

## Javascript:
- the original styleguide was following Github’s guide, but they switched to coffeescript
- although we’re not using coffeescript, readability is still an important factor
- use single-quotes (`'string'`) - Douglas Crockford recommends this, I agree
- don’t use semicolons for line ending, in JS they’re totally optional. And when chaining promises, it’s much easier to add stuff
```
$http.promise()
  .then()
  .then()
```


## HTML / SEO:
- First title on the page must be always `h1`. All the other titles must be other then `h1`.
