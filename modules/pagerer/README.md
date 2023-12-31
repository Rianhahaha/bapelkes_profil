Pagerer
=======

Pagerer is a module providing a collection of pager styles to enhance Drupal
and Views standard pagers.

Administrators or site builders can preset multiple pager configurations. Each
'preset' allows a pager to be made up of three 'panes': left, center, and
right. Each pane can contain (or not) a pager style. In this way there are
plenty of possibilities to combine different elements to satisfy complex
requirements.

Pagerer allows to override Drupal's core pager with any of the preset
configurations. Also, a pager plugin for Views allows to use any of the preset
pagers within any view.

Pagerer uses standard Drupal pager classes to render the pagers, so CSS styling
is preserved.

Module developers can also make direct calls to the styles, thus allowing
even more complex scenarios.

Features
--------
- multi-pane pager
- Views pager plugin
- control whether to display links to pages, to items, or to item ranges
- links to progressively more distant pages (like +10, + 20, +100, +200)
- adaptive logic links
- specify text to be used to render page separators (like a vertical bar)
  and page breakers (like an ellipsis)
- supports Views' AJAX enabled pager
- allow page numbers in URL querystrings to start counting from 1

Styles
------
- _Standard_    - alike standard Drupal pager theme
- _Progressive_ - provides links to pages progressively more distant from current
- _Adaptive_    - provides links to pages following an adaptive logic
- _Basic_       - similar to Views mini pager
- _Multipane_   - a multi-pane (left, center, and right) pager style, enabling
                  each pane to contain one of the styles above

Requirements
------------
- The module **must** be downloaded or updated using Composer, see [Download contributed modules and themes using Composer](https://www.drupal.org/node/2718229#adding-modules).
- Drupal 10.0 and higher

Instructions
------------
- Download the module running ```composer require drupal/pagerer:^3``` from the command line.
- Enable the module.
- Check the Configuration page to setup.
- Create and configure any number of 'preset' pagers.
- Select a preset to use as a general replacement of Drupal core pager, or
  use a preset as a pager in Views.
- to change URL querystrings to start page count from one, go to the _URL settings_
  tab, select the _URL querystring_ checkbox, and _One-based_ from _Page index base_.

URL querystring override
------------------------
Historically in Drupal, the _'page'_ URL query parameter is a comma-delimited
string of numeric values, where each value is the target content page for the
corresponding pager element. The numeric values are expressed in base zero. For
instance, if we have 3 pagers on a single page, the URL query will look
something like this: _?page=0,2,0_. Actually the commas get encoded in Drupal 8
and above, so the real string will be _?page=0%2C2%2C0_.  
Since Pagerer 8.x-2.0 it is possible to override Drupal core's querystring so to
use dots instead of commas to separate the values, and to start the page count
from one instead of zero, so that the URL query will look like this instead:
 _?pg=1.3.1_.
Note that the standard _?page=0_ format will keep working when passed in to a
HTTP request, but it will no longer be output in rendered links.

Views pager plugin
------------------
The pager plugin for Views introduces a new _'Paged output, Pagerer'_ option in
the list of possible pagers to be used for a view.
It behaves like a 'full pager', with the additional option to select the
Pagerer preset to be used for rendering the pager - so that every view could
use a different preset, but also many views could use the same preset.

-------------------------------------------------------------------------------

Styles
======

Standard
--------
This style is alike standard Drupal pager theme.

Provides links to the 'neigborhood' of current page, plus
first/previous/next/last page. Extended control on the pager is available
through Pagerer's specific configuration variables.

Progressive
-----------
This style provides links to pages progressively more distant from current.

Besides links to the 'neigborhood' of current page, creates a list of
links which are progressively more distant from current page, displaying
either a page number or an offset from current page.

This is controlled via the 'progr_links' style confuguration variable, which
can take a value either 'absolute' or 'relative'.

Examples:

```
page 9 out of 212, progr_links 'absolute', display 'pages':
-------------------------------------------------------------------
1 . 4 . 7 8 [9] 10 11 . 14 . 19 . 59 . 109 . 212
-------------------------------------------------------------------
```

```
page 9 out of 212, progr_links 'relative', display 'pages':
-------------------------------------------------------------------
1 . -5 . 7 8 [9] 10 11 . +5 . +10 . +50 . +100 . 212
-------------------------------------------------------------------
```

The 'factors' style configuration variable controls the quantity of
progressive links generated. Each value in the comma delimited string
will be used as a scale factor for a progressive series of pow(10, n).

Examples: 'factors' => '10' will generate links for page offsets

```
  ..., -1000, -100, -10, 10, 100, 1000, ....
```

'factors' => '5,10' will generate links for page offsets

```
  ..., -1000, -500, -100, -50, -10, -5, 5, 10, 50, 100, 500, 1000, ....
```

etc.

Adaptive
--------
This style provides links to pages following an adaptive logic.

Besides links to the 'neigborhood' of current page, creates page links
which are adaptively getting closer to a target page, through subsequent
calls to the links themselves. More or less, the principle is the same
as of the binary search in an ordered list.

On a first call, the style creates links to a list of pages in the
neighborood of first page, plus a link to last page, plus links to 2
pages in the space between first and last page:
- one to the middle,
- one to the middle of the space between the first page and the one above

Example - current page in square brackets:

```
page 1 out of 252:
-------------------------------------------------------------------
[1] 2 3 4 5 . +62 . +125 . 252
-------------------------------------------------------------------
```

On subsequent calls, if a link outside of the neighborhood (nicknamed
'adaptive link') is called, then we will assume that the 'target' page
looked for is comprised within the interval between the pages to the
left and to the right of the link invoked. So, the style will restrict
the range of the pages to be presented in the pager by setting these
pages as the min and max boundaries (plus first and last page, which are
always displayed to 'release' the restriction), and recalculating the
middle and middle-of-the-middle to present the new links.

Example (following on from above):

click on +62, go to page 63 and lock page 5 (represented as -58 from 63)
and 126 (represented as +63 from 63) as new boundaries:
```
-------------------------------------------------------------------
1 . -58 . -31 . -15 . 61 62 [63] 64 65 . +15 . +31 . +63 . 252
-------------------------------------------------------------------
```
note how also the space on the left is filled in with links having same
absolute offset as the ones to the right.

and so on, click on -15, go to page 48 and lock page 32 (represented as
-16 from 48) and 61 (represented as +13 from 48):
```
-------------------------------------------------------------------
1 . -16 . -8 . -4 . 46 47 [48] 49 50 . +4 . +8 . +13 . 252
-------------------------------------------------------------------
```
Like for the 'Progressive' style, links are displayed either as a page number
or as an offset from current page. This is controlled via the 'progr_links'
style configuration variable, which can take a value either 'absolute' or
'relative'.

Basic
-----
Similar to the Views mini pager, presents current page and links to previous
and next page (by default).

Examples:

```
page 9 out of 955, display 'pages':
-----------------
< Page 9 of 955 >
-----------------
```

```
page 9 out of 955, total items = 47731, limit = 50, display = 'item_ranges':
--------------------------
< Items 401-450 of 47731 >
--------------------------
```

-------------------------------------------------------------------------------

For developers: rendering pagers through Pagerer themes
=======================================================

Pagerer implements two themes to render its own styled pagers:

- _pagerer_base_   - for single style pagers
- _pagerer_        - for the multipane pager

Embed the theme and its variables in a 'pager' type render array, like e.g.

```
    return [
      '#type' => 'pager',
      '#theme' => 'pagerer_base',
      '#element' => 0,
      '#style' => 'standard',
      '#config' => [
        'display_restriction' => 0,
      ],
    ];
```

pagerer_base
------------
The variables defined for this theme are the following:
- ```style```: the machine name of the style plugin used for styling the pager.
  Pagerer provides the following plugins: 'standard', 'progressive', 'adaptive',
  'mini', 'basic', 'scrollpane', 'slider'.
- ```element```: same as Drupal, an optional integer to distinguish between
  multiple pagers on one page.
- ```parameters```: same as Drupal, an associative array of query string
  parameters to append to the pager links.
- ```config```: (optional) an associative array of configuration elements to be
  passed to the style plugin.

The content of the ```config``` variable depends on the style plugin requirements.
Each plugin manages its own default values, that get merged with the values
passed by the config variable upon rendering of the pager.

- ```quantity```: the number of pages in the list of the 'neighborhood' of the
  current page. Same as the Drupal core pager theme's 'quantity' variable.
  Applies to styles: _standard, progressive, adaptive_.
- ```display```: can take the values "pages", "items", "item_ranges". Determines
  whether to display pages, or items, or item ranges.
- ```display_restriction```: it allows to restrict showing the pager based on the
  actual number of pages in the result set. It takes the minimum number of
  pages that the result set need to have to have a pager rendered:
  - 2 (pager is shown if the result set is made of at least two pages),
  - 1 (pager is shown even if the result set is made of one page only),
  - 0 (a 'no pages to display' text is rendered even if the result set is
      empty).
- ```display_mode```: Determines how the pager is rendered.
  Options are:
  - "none" (not displayed),
  - "normal" (as a list of pages),
  - "widget" (an active input box from where users can enter directly a page
    to go to).
  Applies to styles: _standard, progressive, adaptive, mini, basic_.
- ```prefix_display```: Determines whether to render a text label (e.g. "Page:")
  before the actual pager.
- ```suffix_display```: Determines whether to render a text label (e.g. "of @total")
  after the actual pager.
- ```separator_display```: A flag to indicate if a text separator has to be
  included between contiguous pages.
  Applies to styles: _standard, progressive, adaptive, scrollpane_.
- ```breaker_display```: A flag to indicate if a text element (e.g. "...") has to
  be introduced when the pages sequence breaks.
  Applies to styles: _standard, progressive, adaptive_.
- ```first_link```: Determines when to render a link to the first page (e.g.
  "<< First"). Options are:
  - "never" (not displayed),
  - "not_on_first" (not displayed if current page is the first),
  - "always" (always displayed).
- ```previous_link```: Determines when to render a link to the previous page
  (e.g. "< Previous"). Options are:
  - "never" (not displayed),
  - "not_on_first" (not displayed if current page is the first),
  - "always" (always displayed).
- ```next_link```: Determines when to render a link to the next page (e.g.
  "Next >"). Options are:
  - "never" (not displayed),
  - "not_on_last" (not displayed if current page is the last),
  - "always" (always displayed).
- ```last_link```: Determines when to render a link to the last page (e.g.
  "Last >>"). Options are:
  - "never" (not displayed),
  - "not_on_last" (not displayed if current page is the last),
  - "always" (always displayed).
- ```progr_links```: Select whether to render page links to pages outside of the
  'neighborhood' as "absolute" page numbers (or items/item ranges) or as
  "relative" offsets from current (e.g. "+10 +100 +1000").
  Applies to styles: _progressive, adaptive_.
- ```factors```: Comma delimited string of factors to use to determine progressive
  links.
  Applies to styles: _progressive_.
- ```widget_resize```: Determines if the input box width should be calculated
  dynamically based on the width of the string of the last page/item number.
  Set to 'false' to keep a fixed width as set by CSS styling. Default: 'true'.
  Applies to styles: _mini_.
- ```widget_button```: Determines if a button has to be shown aside the input box,
  clicking which the page relocation will be triggered. Options are:
  - "no" (page relocation will only occur by pressing the 'return' key on the
    keyboard),
  - "yes" (button is shown, and button is styled via CSS),
  - "auto" (button height is automatically resized to match the input box
    height).
  Applies to styles: _mini_.
- ```slider_width```: The width of the slider bar. Expressed in 'em' for CSS styling.
  Leave blank to default to CSS settings.
  Applies to styles: _slider_.
- ```slider_action```: Determines how the page relocation should be triggered after
  it has been selected through the jQuery slider. Options are:
  - "tickmark" (page relocation only occurs after user clicks a tickmark on the
    slider handle),
  - "timeout" (page relocation occurs after a grace time has elapsed),
  - "auto" (the timeout method is automatically selected based on the
    accuracy of the slider, i.e. if there are at least 3 pixels between
    contiguous pages).
  Applies to styles: _slider_.
- ```slider_action_timeout```: The grace time (in milliseconds) to wait before the
  page is relocated, in case "timeout" slider_action method is selected for
  the jQuery slider. "0" will trigger relocation immediately.
  Applies to styles: _slider_.
- ```slider_navigation_icons```: Determines whether to display +/- navigation icons
  on the sides of the jQuery slider. Options are "yes", "no", "auto" (the icons
  are automatically displayed based on the accuracy of the slider).
  Applies to styles: _slider_.
- ```tags```: an associative array of textual elements to be used to render the
  pager, see details in section below.

  The ```tags``` variable in Pagerer style configuration is an associative array of
  tags to be used to render any of the textual elements of the pager.

  - ```page_breaker```: Text to render a break in the page sequence. Defaults to an
    ellipsis (...).
    Applies to styles: _standard, progressive, adaptive_.
  - ```page_separator```: Text to fill between contiguous pages, if
    'separator_display' is set on. Defaults to a vertical bar (|).
    separators are rendered.
    Applies to styles: _standard, progressive, adaptive, scrollpane_.
  - ```pages```: An associative array of text elements to be used if 'display' is
    set to "pages". See below.
  - ```items```: An associative array of text elements to be used if 'display' is
    set to "items". See below.
  - ```item_ranges```: An associative array of text elements to be used if 'display' is
    set to "item_ranges". See below.

  Each of ```tags.pages```, ```tags.items``` and ```tags.item_ranges``` indicate how the relative
  text should be rendered in the pager, based on the current 'display' mode
  selected. Regardless of the display mode, the following placeholders will be
  resolved at runtime with live data:

  - ```@number```       - the target page number.
  - ```@offset```       - the offset of the target page from the current one, in pages.
  - ```@total```        - total number of pages in the pager.
  - ```@item```         - the number of the first item in the target page.
  - ```@item_low```     - the number of the first item in the target page.
  - ```@item_high```    - the number of the last item in the target page.
  - ```@item_offset```  - the offset of the target page from the current one, in items.
  - ```@total_items```  - total number of items in the pager.

- ```prefix_label```: Text to use to render a label (e.g. "Page") in front of the
  pager.
- ```suffix_label```: Text to use to render a label (e.g. "of @number") after the
  pager.
- ```page_current```: Text to use to render the current page/item/item range.
- ```page_previous```: Text to use to render a page/item/item range that is
  before the current page.
- ```page_next```: Text to use to render a page/item/item range that is
  after the current page.
- ```page_previous_relative```: Text to use to render the link to a previous page
  outside of the neighborhood (e.g. "-100").
  Applies to styles: _progressive, adaptive_.
- ```page_next_relative```: Text to use to render the link to a next page outside
  of the neighborhood (e.g. "+100").
  Applies to styles: _progressive, adaptive_.
- ```first```: Text to use to render the link to the first page (e.g. "<<
  first").
- ```previous```: Text to use to render the link to the previous page
  (e.g. "< Previous").
- ```next```: Text to use to render the link to the next page (e.g. "Next >").
- ```last```: Text to use to render the link to the last page (e.g. "Last >>").
- ```pageset_empty```: Text to use to render the current page in the pager in case
  there are no items in the pageset (e.g. 'No pages to display').
- ```page_current_title```: Help text used when hovering the current page link (e.g.
  'Current page').
- ```page_title```: Help text used when hovering a page link (e.g. 'Go to page
  @number').
- ```first_title```: Help text used when hovering a first page link (e.g. 'Go to
  first page').
- ```previous_title```: Help text used when hovering a previous page link (e.g.
  'Go to previous page').
- ```next_title```: Help text used when hovering a next page link (e.g. 'Go to
  next page').
- ```last_title```: Help text used when hovering a last page link (e.g. 'Go to
  last page').
- ```page_current_reader```: Text to be used for the current page by automated
  readers.
- ```page_reader```: Text to be used for non current page link by automated
  readers.
- ```first_reader```: Text to be used for the first page link by automated
  readers.
- ```previous_reader```: Text to be used for the previous page link by automated
  readers.
- ```next_reader```: Text to be used for the next page link by automated
  readers.
- ```last_reader```: Text to be used for the last page link by automated
  readers.
- ```widget_title```: Help text used when hovering the direct input widget (e.g.
  'Enter page, then press Return.').
  Applies to styles: _mini_.
- ```slider_title```: Help text used when hovering the slider (e.g. 'Drag the handle
  to the page required.').
  Applies to styles: _slider_.
- ```slider_tickmark_title```: Help text appended to the slider help when user is
  expected to click on the tickmark to start page relocation. Defaults to:
  'Then, click on the tickmark.'.
  Applies to styles: _slider_.


pagerer
-------

The ```pagerer``` theme itself is more a container of individual ```pagerer_base```
themes. The theme to be used in each pane (left, center, and right) and its
variables are either retrieved from a defined PagererPreset configuration
entity through its 'data' property, or passed to a {position}_pane variable
of the pagerer theme.

- ```element```: same as Drupal, an optional integer to distinguish between
  multiple pagers on one page.
- ```parameters```: same as Drupal, an associative array of query string
  parameters to append to the pager links.
- ```config```: (optional) an associative array of configuration elements to be
  passed to the style plugin:
  - ```preset```: (optional) specifies the preset pager configuration, created
    through Pagerer's admin UI, to be used to render the pager. If not
    specified, {position}_pane variables are to be passed. Also, any
    {position}_pane variables will override the preset configuration, if
    specified along the preset variable.
  - ```panes```: an associative array of 'left'|'center'|'right', where each key
    is an associative array of:
    - ```style```: the style plugin to pass to _pagerer_base_ theme: 'standard' |
      'progressive' | 'scrollpane' | 'adaptive' | 'mini' | 'slider' | 'none'
    - ```config```: the configuration array to pass to _pagerer_base_ theme

Example:

```
    return [
      '#type' => 'pager',
      '#theme' => 'pagerer',
      '#element' => 1,
      '#config' => [
        'panes' => [
          'left' => [
            'style' => 'mini',
            'config' => [
              'display' => 'items',
            ],
          ],
          'center' => [
            'style' => 'none',
          ],
          'right' => [
            'style' => 'mini',
            'config' => [
              'display' => 'pages',
            ],
          ],
        ],
      ],
    ];
```
