angular-dynamic-forms
=====================
Build Forms in AngularJS From Nothing But JSON

Uses the MIT License.  See `LICENSE` file for details.

Installation
------------
1. Clone the project from either [GitHub][] or [BitBucket][] - whichever you prefer.
2. Copy `dynamic-forms.js` into your project wherever your other assets reside.

Use
---
As with any other [AngularJS][] module:

* include the script into your page anywhere after [AngularJS][] itself.

```html
    <script src="assets/js/dynamic-forms.js"></script>
```

* list `dynform` as a dependency of your project.

```javascript
    appModule = angular.module('app', ['dynform']);
```

* create a [dynamic-form](#the-directive) element anywhere in your page.

```html
    <dynamic-form template="formTemplate"
        ng-model="formData"
        ng-submit="processForm()">
    </dynamic-form>
```

* populate your [template](#the-template) with a JSON object describing the form you want to create.

```javascript
    $scope.formTemplate = {
        "first": {
            "type": "text",
            "label": "First Name"
        },
        "last": {
            "type": "text",
            "label": "Last Name"
        },
        "submit": {
            "type": "submit"
        },
    };
```

And that's about it!

The TL;DR Version
-----------------
### The Directive ###
You invoke the `dynamic-form` directive using an element (`<dynamic-form></dynamic-form>`) - other
options (such as class, attribute, and comment) are unsupported (for now).  The directive requires
two attributes: an [`ng-model`][], and either a `template` or a `template-url`.  The [`ng-model`][]
will be used to generate valid [`ng-model`][] attributes for the various input controls in the
template.  In accordance with how [AngularJS][] handles this attribute elsewhere, your entire form's
data will be available in keys of whichever model you specify here (though nested forms are an
exception, unless you specify a key in the outer form's model as the [`ng-model`][] of the inner
form).

If you specify a `template-url`, the `dynamic-form` directive will retrieve the form template via
[`$http`][] and build out your form based on the response.  Currently, failure is silently ignored.
This may change in a later release.

You may not want to rely on the directive to retrieve your form directly - perhaps you want to do
some processing on the server response before passing it to the directive for building, or maybe
you need to specify a more complex [`$http`][] request with advanced authentication.  Or perhaps you
just want to proactively handle failure to retrieve the template.  Enter the `template` attribute.
When the directive sees `template`, it ignores any `template-url` and instead uses the object
identified by the `template` attribute.

Any other attributes you specify on the `dynamic-form` element are copied across to the `form` or
[`ng-form`][] element that the directive builds to replace itself with.  Similarly, any pre-existing
contents are copied across as well, to the top of the resulting form, with the
dynamically-specified controls below them.  This allows you to nest `dynamic-form`s inside each
other in the same way as [`ng-form`][] (which is one reason this directive implements this
pseudo-transclusion).

The `dynamic-form` directive makes every attempt to set up the forms it generates to be valid HTML
forms, complete with the ability to have their data submitted to the server by the browser's native
form submission mechanism and still have the data in the same structure that it takes on in your
[AngularJS][] models.  This makes it easy to implement a fallback mode in case there is a problem
with using the standard [Angular][AngularJS] methods to handle your form inputs.  You will, of
course, need to provide your own `action` and `method` attributes for this to work completely.

### The Template ###
Regardless of whether it arrives via `template` or `template-url`, the form template is a
fairly-straightforward JavaScript object.  Its keys correspond (by default) to input control
`name`s, which in turn (also by default) correspond to keys in the form's model.  Each of the values
of this object is another object describing a form input control.  A `type` key identifies what kind
of control to build, which in turn determines what [other keys](#common-options) are expected.  Any
`type` that isn't supported builds a `<span>` containing the value of the `label` key, if one
exists, and any other keys as attributes.  Following is a list of all currently-supported `type`s,
and then a more detailed specification of each.  (Note that not all of these are properly supported
in all browsers, yet; there are a number of references around the web for [which browsers support
what][formsupport])

* [button](#button)
* [checkbox](#checkbox)
* [checklist](#checklist)
* [color](#color)
* [date](#date)
* [datetime](#datetime)
* [datetime-local](#datetime-local)
* [email](#email)
* [fieldset](#fieldset)
* [file](#file)
* [hidden](#hidden)
* [image](#image)
* [legend](#legend)
* [month](#month)
* [number](#number)
* [password](#password)
* [radio](#radio)
* [range](#range)
* [reset](#reset)
* [search](#search)
* [select](#select)
* [submit](#submit)
* [tel](#tel)
* [text](#text)
* [textarea](#textarea)
* [time](#time)
* [url](#url)
* [week](#week)

#### Common Options ####
* `attributes`: key-value pairs for arbitrary attributes not otherwise supported here; it is strongly
    recommended that you use this option *only* if the attribute you need isn't already supported
* `class`: see [`ng-class`][]
* `callback`: see [`ng-change`][] (or [`ng-click`][] for button-like types)
* `disabled`: see [`ng-disabled`][]
* `label`: wraps the control in a `<label>` tag with the value as text content (but see specific
    types for exceptions to how this is handled)
* __The following options are only supported for types that have values:__
    * `model`: overrides the control's ID as the value of [`ng-model`][] and `name` attributes;
        allows multiple controls to be tied to a single model
    * `readonly`: see [`ng-readonly`][]
    * `required`: see [`ng-required`][]
    * `val`: an initial value for the model

#### button ####
* __Renders:__ `<button></button>`
* __Additional Options:__
    * None
* __Other Notes:__
    * The value of `label` is used as the content of the `<button>` itself; no additional elements
        are created

#### checkbox ####
* __Renders:__ `<input type="checkbox">`
* __Additional Options:__
    * `isOn`: see [`ng-true-value`][]
    * `isOff`: see [`ng-false-value`][]
    * `slaveTo`: see [`ng-checked`][]
* __Other Notes:__
    * See also the [checklist](#checklist) type, below

#### checklist ####
* __Renders:__ multiple `<input type="checkbox">` controls
* __Additional Options:__
    * `options`: an object containing a collection of child objects, each describing a checkbox
        * The key of each child object specifies the key to associate with the checkbox it describes
        * `class`: applies a specific [`ng-class`][] to the current checkbox, independently of the
            rest
        * `label`: operates identically to the standard `label` option, but applies to a specific
            checkbox in the list
        * See the [checkbox](#checkbox) type for other fields supported here
* __Other Notes:__
    * This is a convenience type, used to tie a group of [checkbox](#checkbox) controls together
        under a single model; the model holds an object, and each control sets a separate key
        within it
    * You can set a `val` on the entire `checklist` (it must, of course, be an object) in addition
        to any per-option `val`s; the per-option versions are set after the full `checklist`
        version, so they will override anything set to their key by the `checklist` itself

#### color ####
* __Renders:__ `<input type="color">`
* __Additional Options:__
    * None
* __Other Notes:__
    * May not be [supported][colorsupport] in all browsers

#### date ####
* __Renders:__ `<input type="date">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

#### datetime ####
* __Renders:__ `<input type="datetime">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

#### datetime-local ####
* __Renders:__ `<input type="datetime-local">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

#### email ####
* __Renders:__ `<input type="email">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * On devices that have on-screen keyboards, the browser may modify the keyboard layout to make
        entering email addresses in these controls easier.

#### fieldset ####
* __Renders:__ `<fieldset></fieldset>`
* __Additional Options:__
    * `fields`: the template for the fields which should appear in the `fieldset`
* __Other Notes:__
    * The value of `label` is used to create a `<legend>` tag as the first child of the `fieldset`

#### file ####
* __Renders:__ `<input type="file">`
* __Additional Options:__
    * `multiple`: whether or not the user can select more than one file at a time with this single
        control
* __Other Notes:__
    * [A directive][filedirective] is included with this module that allows `file` controls to
        properly bind to [AngularJS][] models - the control's FileList object is stored in the
        model, and updating the model's value with a valid FileList object will update the control
        accordingly
    * Also included is [an AngularJS service][fileservice] that wraps the browser's FileReader in a
        promise, so you can get the contents of the selected file for further manipulation, or even
        send it along in an AJAX request, and all without leaving [AngularJS][]
    * Both of these additions are modified versions of code by [K. Scott Allen][] and made
        available on the [OdeToCode][] website; the original versions are linked above

#### hidden ####
* __Renders:__ `<input type="hidden">`
* __Additional Options:__
    * None
* __Other Notes:__
    * Because the underlying HTML control has so little functionality, this control only supports
        `model` and `val` keys

#### image ####
* __Renders:__ `<input type="image">`
* __Additional Options:__
    * `source`: the URL of the image to display in this control
* __Other Notes:__
    * The value of `label` is used to set the `alt` attribute of this control

#### legend ####
* __Renders:__ `<legend></legend>`
* __Additional Options:__
    * None
* __Other Notes:__
    * As a display-only control, only `class`, `label`, `callback` (via [`ng-click`][]) and
        `disabled` are supported on this control
    * The value of `label` is used to set the contents of this control

#### month ####
* __Renders:__ `<input type="month">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

#### number ####
* __Renders:__ `<input type="number">`
* __Additional Options:__
    * `maxValue`: the largest allowed value for this control
    * `minValue`: the smallest allowed value for this control
    * Also see [text](#text) below
* __Other Notes:__
    * May not be [supported][numbersupport] in all browsers

#### password ####
* __Renders:__ `<input type="password">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * The only real difference between this control and a [text](#text) control is in the rendering,
        so they support exactly the same options (with the exception of `splitBy`, since it makes no
        sense to split obscured-input strings)

#### radio ####
* __Renders:__ multiple `<input type="radio">` controls
* __Additional Options:__
    * `values`: an object which acts as a simple list of radio options to include
        * The key of each property of this option specifies the value the model should be set to
            when the associated radio `input` is selected
        * The value of each property of this option specifies the label to use for the associated
            radio `input`
* __Other Notes:__
    * Because a single radio `input` by itself isn't particularly useful in most cases, this control
        type assumes users will want to define a list of `value:label` pairs tied to a single model;
        if this is incorrect, you can still create `radio` controls with just one `value:label`
        each, and then tie them together using the `model` key
    * [The directive](#the-directive) doesn't prevent you from applying a `label` to the entire
        collection of `input` controls created by this control type - the entire `div` containing
        them will be wrapped in a `<label>` tag; keep this in mind when building style sheets

#### range ####
* __Renders:__ `<input type="range">`
* __Additional Options:__
    * `step`: the amount by which the control can increase or decrease in value
    * Also see [number](#number) above
* __Other Notes:__
    * By default, this control seems to provide its values to [AngularJS][] as strings.  This might
        be due to [Angular][AngularJS] (as well as the browser) handling them as regular
        [text](#text) controls internally.  Among its other minor tweaks, this module contains a
        very simple directive to override the default [`$parsers`][] mechanism for `range` controls
        and convert these values back to numbers (floats, in case your `step` is not an integer).
    * May not be [supported][rangesupport] in all browsers

#### reset ####
* __Renders:__ `<button type="reset"></button>`
* __Additional Options:__
    * None
* __Other Notes:__
    * As with [button](#button), the value of `label` provides the control's contents
    * [AngularJS][] doesn't seem to monitor the `reset` event, so your models wouldn't normally be
        updated when the form is cleared in this way; while this control is strongly dis-recommended
        in most cases, this directive supports it, so code is included that monitors and properly
        handles these events (NOTE - this feature has not been widely tested; please report any
        issues on [GitHub][issues-github] or [Bitbucket][issues-bitbucket])

#### search ####
* __Renders:__ `<input type="search">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * All browsers support this because it works exactly like a [text](#text) control.  The idea is
        that search boxes will be styled differently, and on some devices might even support
        additional input methods, such as voice recognition.  You'll probably want to tie these
        controls to some kind of search mechanism in your app, since users whose browsers *do*
        render them differently will expect them to act accordingly.

#### select ####
* __Renders:__ `<select></select>`
* __Additional Options:__
    * `autoOptions`: see [`ng-options`][]
    * `empty`: if not `false` or undefined, specifies the display value (contents) of an empty
        `option`, and tells the directive to include it in its output
    * `multiple`: if not `false` or undefined, allows the user to select more than one `option` at
        one time
    * `options`: an object containing a collection of child objects, each describing an `option` to
        include in the `select` list
        * The key of each child object gives the value of the associated `option`
        * `disabled`: see [`ng-disabled`][]
        * `group`: adds the `option` to an `optgroup` whose label is the value of the `group` key
        * `label`: the display value (contents) of the `option`
        * `slaveTo`: see [`ng-selected`][]
* __Other Notes:__
    * An unreleased prototype version of this module (which used a combination of [`ng-repeat`][]
        and [`ng-switch`][] instead of a [directive][]) specified four different control types for
        the functionality provided by this one - one for normal lists, one for grouped lists, one
        for multi-select lists, and one that combined multi-select with group support; this version
        is much cleaner about its approach - multi-select is the 'flag' option `multiple`, and
        groups are enabled by simply defining them with their associated values
    * Note that only one of `options` and `autoOptions` will be honored by this module - if you
        specify `autoOptions`, `options` is completely ignored; keep this in mind when building your
        forms because the actual values will have to be specified in a separate object

#### submit ####
* __Renders:__ `<button type="submit"></button>`
* __Additional Options:__
    * None
* __Other Notes:__
    * As with [button](#button), the value of `label` provides the control's contents

#### tel ####
* __Renders:__ `<input type="tel">`
* __Additional Options:__
    * See [text](#text) below
* __Other Notes:__
    * There is currently no validation support for this control in nearly any browser, and
        [AngularJS][] doesn't provide any by default.  This is mostly because telephone numbers are
        tricky beasts, especially with the differences between how these numbers are allocated from
        one part of the world to another.  And that's before we start getting messy with things
        like VoIP "numbers".

#### text ####
* __Renders:__ `<input type="text">`
* __Additional Options:__
    * `maxLength`: see [`ng-maxlength`][]
    * `minLength`: see [`ng-minlength`][]
    * `placeholder`: a value to display in a `text`-like control when it is empty
    * `splitBy`: see [`ng-list`][] (this option is only supported by `text` and
        [textarea](#textarea) controls)
    * `validate`: see [`ng-pattern`][]
* __Other Notes:__
    * This control serves as the base template for nearly all the other form input controls defined
        by HTML - as such, most of the controls supported by this directive support these options as
        well, leading their entries to refer here

#### textarea ####
* __Renders:__ `<textarea></textarea>`
* __Additional Options:__
    * See [text](#text) above
* __Other Notes:__
    * While the syntax used to define them in raw HTML differs to some extent, the only practical
        difference between [text](#text) and `textarea` controls is the multi-line support offered
        by `textarea` - therefore, the options available to each are identical

#### time ####
* __Renders:__ `<input type="time">`
* __Additional Options:__
    * See [text](#text) above
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

#### url ####
* __Renders:__ `<input type="url">`
* __Additional Options:__
    * See [text](#text) above
* __Other Notes:__
    * Similar to the [email](#email) type, some browsers will alter on-screen keyboards when this
        control type is selected in such a way that URL entry is simplified; [AngularJS][],
        meanwhile, enforces the requirement that the value be a valid URL, even if the browser does
        not

#### week ####
* __Renders:__ `<input type="week">`
* __Additional Options:__
    * See [text](#text) above
* __Other Notes:__
    * May not be [supported][datesupport] in all browsers

Acknowledgements
----------------
* [Frank Linehan][] for leading me in [the right direction][directive].
* [K. Scott Allen][] for the [file input directive][filedirective] and the [FileReader
    service][fileservice] adapted for use here.
* [Joel Hooks][] for pointing out that the LGPL is too strong for this kind of project,
	and that a monolithic piece of code like version 0.0.0 makes very little sense in
	an MVVM environment.

Issues And Assistance
---------------------
If you notice a problem, let me know about it on [GitHub][issues-github] or
[Bitbucket][issues-bitbucket]!

Any and all help is welcome; just fork the project on either [GitHub][] or [BitBucket][] (whichever
you prefer), and submit a pull request with your contribution(s)!

[colorsupport]: http://caniuse.com/input-color
[datesupport]: http://caniuse.com/input-datetime
[filedirective]: http://odetocode.com/blogs/scott/archive/2013/07/05/a-file-input-directive-for-angularjs.aspx
[fileservice]: http://odetocode.com/blogs/scott/archive/2013/07/03/building-a-filereader-service-for-angularjs-the-service.aspx
[formsupport]: http://caniuse.com/forms
[numbersupport]: http://caniuse.com/input-number
[pholdsupport]: http://caniuse.com/input-placeholder
[rangesupport]: http://caniuse.com/input-range

[GitHub]: https://github.com/danhunsaker/angular-dynamic-forms
[BitBucket]: https://bitbucket.org/danhunsaker/angular-dynamic-forms
[issues-github]: https://github.com/danhunsaker/angular-dynamic-forms/issues
[issues-bitbucket]: https://bitbucket.org/danhunsaker/angular-dynamic-forms/issues

[`$http`]: http://docs.angularjs.org/api/ng.$http
[`$parsers`]: http://docs.angularjs.org/api/ng.directive:ngModel.NgModelController
[directive]: http://docs.angularjs.org/guide/directive
[`ng-change`]: http://docs.angularjs.org/api/ng.directive:ngChange
[`ng-checked`]: http://docs.angularjs.org/api/ng.directive:ngChecked
[`ng-class`]: http://docs.angularjs.org/api/ng.directive:ngClass
[`ng-click`]: http://docs.angularjs.org/api/ng.directive:ngClick
[`ng-disabled`]: http://docs.angularjs.org/api/ng.directive:ngDisabled
[`ng-false-value`]: http://docs.angularjs.org/api/ng.directive:input.checkbox
[`ng-form`]: http://docs.angularjs.org/api/ng.directive:form
[`ng-list`]: http://docs.angularjs.org/api/ng.directive:ngList
[`ng-maxlength`]: http://docs.angularjs.org/api/ng.directive:input
[`ng-minlength`]: http://docs.angularjs.org/api/ng.directive:input
[`ng-model`]: http://docs.angularjs.org/api/ng.directive:ngModel
[`ng-multiple`]: http://code.angularjs.org/1.0.8/docs/api/ng.directive:ngMultiple
[`ng-options`]: http://docs.angularjs.org/api/ng.directive:select
[`ng-pattern`]: http://docs.angularjs.org/api/ng.directive:input
[`ng-readonly`]: http://docs.angularjs.org/api/ng.directive:ngReadonly
[`ng-repeat`]: http://docs.angularjs.org/api/ng.directive:ngRepeat
[`ng-required`]: http://docs.angularjs.org/api/ng.directive:input
[`ng-selected`]: http://docs.angularjs.org/api/ng.directive:ngSelected
[`ng-switch`]: http://docs.angularjs.org/api/ng.directive:ngSwitch
[`ng-true-value`]: http://docs.angularjs.org/api/ng.directive:input.checkbox

[AngularJS]: http://angularjs.org
[Frank Linehan]: http://frank-code.com
[K. Scott Allen]: http://odetocode.com/about/scott-allen
[OdeToCode]: http://odetocode.com
[Joel Hooks]: http://joelhooks.com
