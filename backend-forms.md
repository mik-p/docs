# Backend Forms

- [Introduction](#introduction)
- [Configuring the form behavior](#configuring-form)
    - [Create page](#form-create-page)
    - [Update page](#form-update-page)
    - [Preview page](#form-preview-page)
- [Defining form fields](#form-fields)
    - [Tab options](#form-tab-options)
    - [Field options](#form-field-options)
- [Available field types](#field-types)
- [Form widgets](#form-widgets)
- [Form views](#form-views)
    - [Create view](#form-create-view)
    - [Update view](#form-update-view)
    - [Preview view](#form-preview-view)
- [Applying conditions to fields](#field-conditions)
    - [Input preset converter](#field-input-preset)
    - [Trigger events](#field-trigger-events)
    - [Field dependencies](#field-dependencies)
    - [Preventing a field from being submitted](#prevent-field-submission)
- [Extending form behavior](#extend-form-behavior)
    - [Form Model Events](#form-model-events)
    - [Overriding controller action](#overriding-action)
    - [Overriding controller redirect](#overriding-redirect)
    - [Extending form model query](#extend-model-query)
    - [Extending form fields](#extend-form-fields)
    - [Filtering form fields](#filter-form-fields)
- [Validating form fields](#validate-form-fields)

<a name="introduction"></a>
## Introduction

The **Form behavior** is a controller [behavior](../services/behaviors) used for easily adding form functionality to a backend page. The behavior provides three pages called Create, Update and Preview. The Preview page is a read-only version of the Update page. When you use the form behavior you don't need to define the `create`, `update` and `preview` actions in the controller - the behavior does it for you. However you should provide the corresponding view files.

The Form behavior depends on form [field definitions](#form-fields) and a [model class](../database/model). In order to use the form behavior you should add it to the `$implement` property of the controller class. Also, the `$formConfig` class property should be defined and its value should refer to the YAML file used for configuring the behavior options.

```php
namespace Acme\Blog\Controllers;

class Categories extends \Backend\Classes\Controller
{
    public $implement = ['Backend.Behaviors.FormController'];

    public $formConfig = 'config_form.yaml';
}
```

> **NOTE:** Very often the form and [list behavior](lists) are used together in a same controller.

<a name="configuring-form"></a>
## Configuring the form behavior

The configuration file referred in the `$formConfig` property is defined in YAML format. The file should be placed into the controller's [views directory](controllers-ajax/#introduction). Below is an example of a typical form behavior configuration file:

```yaml
# ===================================
#  Form Behavior Config
# ===================================

name: Blog Category
form: $/acme/blog/models/post/fields.yaml
modelClass: Acme\Blog\Post

create:
    title: New Blog Post

update:
    title: Edit Blog Post

preview:
    title: View Blog Post
```

The following fields are required in the form configuration file:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Field | Description
------------- | -------------
`name` | the name of the object being managed by this form.
`form` | a configuration array or reference to a form field definition file, see [form fields](#form-fields).
`modelClass` | a model class name, the form data is loaded and saved against this model.

The configuration options listed below are optional. Define them if you want the form behavior to support the [Create](#form-create-page), [Update](#form-update-page) or [Preview](#form-preview-page) pages.

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`defaultRedirect` | used as a fallback redirection page when no specific redirect page is defined.
`create` | a configuration array or reference to a config file for the Create page.
`update` | a configuration array or reference to a config file for the Update page.
`preview` | a configuration array or reference to a config file for the Preview page.

<a name="form-create-page"></a>
### Create page

To support the Create page add the following configuration to the YAML file:

```yaml
create:
    title: New Blog Post
    redirect: acme/blog/posts/update/:id
    redirectClose: acme/blog/posts
    flashSave: Post has been created!
```

The following configuration options are supported for the Create page:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`title` | a page title, can refer to a [localization string](../plugin/localization).
`redirect` | redirection page when record is saved.
`redirectClose` | redirection page when record is saved and the **close** post variable is sent with the request.
`flashSave` | flash message to display when record is saved, can refer to a [localization string](../plugin/localization).
`form` | overrides the default form fields definitions for the create page only.

<a name="form-update-page"></a>
### Update page

To support the Update page add the following configuration to the YAML file:

```yaml
update:
    title: Edit Blog Post
    redirect: acme/blog/posts
    flashSave: Post updated successfully!
    flashDelete: Post has been deleted.
```

The following configuration options are supported for the Update page:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`title` | a page title, can refer to a [localization string](../plugin/localization).
`redirect` | redirection page when record is saved.
`redirectClose` | redirection page when record is saved and **close** post variable is sent with the request.
`flashSave` | flash message to display when record is saved, can refer to a [localization string](../plugin/localization).
`flashDelete` | flash message to display when record is deleted, can refer to a [localization string](../plugin/localization).
`form` | overrides the default form fields definitions for the update page only.

<a name="form-preview-page"></a>
### Preview page

To support the Preview page add the following configuration to the YAML file:

```yaml
preview:
    title: View Blog Post
```

The following configuration options are supported for the Preview page:

Option  | Description
------------- | -------------
`title` | a page title, can refer to a [localization string](../plugin/localization).
`form` | overrides the default form fields definitions for the preview page only.

<a name="form-fields"></a>
## Defining form fields

Form fields are defined with the YAML file. The form fields configuration is used by the form behavior for creating the form controls and binding them to the model fields. The file is placed to a subdirectory of the **models** directory of a plugin. The subdirectory name matches the model class name written in lowercase. The file name doesn't matter, but **fields.yaml** and **form_fields.yaml** are common names. Example form fields file location:

```css
📂 plugins
 ┗ 📂 acme
   ┗ 📂 blog
     ┗ 📂 models             <=== Plugin models directory
       ┣ 📂 post             <=== Model configuration directory
       ┃ ┗ 📜 fields.yaml    <=== Model form fields config file
       ┗ 📜 Post.php         <=== model class
```

Fields can be placed in three areas, the **outside area**, **primary tabs** or **secondary tabs**. The next example shows the typical contents of a form fields definition file.

```yaml
# ===================================
#  Form Field Definitions
# ===================================

fields:
    blog_title:
        label: Blog Title
        description: The title for this blog

    published_at:
        label: Published date
        description: When this blog post was published
        type: datepicker

    [...]

tabs:
    fields:
        [...]

secondaryTabs:
    fields:
        [...]
```

Fields from related models can be rendered with the [Relation Widget](#widget-relation) or the [Relation Manager](relations#relationship-types). The exception is a OneToOne or morphOne related field, which must be defined as **relation[field]** and then can be specified as any other field of the model:

```yaml
    user_name:
        label: User Name
        description: The name of the user
    avatar[name]:
        label: Avatar
        description: will be saved in the Avatar table
    published_at:
        label: Published date
        description: When this blog post was published
        type: datepicker

    [...]
```

<a name="form-tab-options"></a>
### Tab options

For each tab definition, namely `tabs` and `secondaryTabs`, you can specify these options:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`stretch` | specifies if this tab stretches to fit the parent height.
`defaultTab` | the default tab to assign fields to. Default: Misc.
`icons` | assign icons to tabs using tab names as the key.
`lazy` | array of tabs to be loaded dynamically when clicked. Useful for tabs that contain large amounts of content.
`cssClass` | assigns a CSS class to the tab container.
`paneCssClass` | assigns a CSS class to an individual tab pane. Value is an array, key is tab index or label, value is the CSS class. It can also be specified as a string, in which case the value will be applied to all tabs.

> **NOTE:** It is not recommended to use lazy loading on tabs with fields that are affected by triggers.

```yaml
tabs:
    stretch: true
    defaultTab: User
    cssClass: text-blue
    lazy:
        - Groups
    paneCssClass:
        0: first-tab
        1: second-tab
    icons:
        User: icon-user
        Groups: icon-group

    fields:
        username:
            type: text
            label: Username
            tab: User

        groups:
            type: relation
            label: Groups
            tab: Groups
```

<a name="form-field-options"></a>
### Field options

For each field you can specify these options (where applicable):

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`label` | a name when displaying the form field to the user.
`type` | defines how this field should be rendered (see [Available fields types](#field-types) below). Default: `text`.
`span` | aligns the form field to one side. Options: `auto`, `left`, `right`, `storm`, `full`. Default: `full`. The parameter `storm` allows you to display the form as a Bootstrap grid, using the `cssClass` property, for example, `cssClass: col-xs-4`.
`size` | specifies a field size for fields that use it, for example, the textarea field. Options: `tiny`, `small`, `large`, `huge`, `giant`.
`placeholder` | if the field supports a placeholder value.
`comment` | places a descriptive comment below the field.
`commentAbove` | places a comment above the field.
`commentHtml` | allow HTML markup inside the comment. Options: `true`, `false`.
`default` | specify the default value for the field. For `dropdown`, `checkboxlist`, `radio` and `balloon-selector` widgets, you may specify an option key here to have it selected by default.
`defaultFrom` | takes the default value from the value of another field.
`tab` | assigns the field to a tab.
`cssClass` | assigns a CSS class to the field container.
`readOnly` | prevents the field from being modified. Options: `true`, `false`.
`disabled` | prevents the field from being modified and excludes it from the saved data. Options: `true`, `false`.
`hidden` | hides the field from the view and excludes it from the saved data. Options: `true`, `false`.
`stretch` | specifies if this field stretches to fit the parent height.
`context` | specifies what context should be used when displaying the field. Context can also be passed by using an `@` symbol in the field name, for example, `name@update`.
`dependsOn` | an array of other field names this field [depends on](#field-dependencies), when the other fields are modified, this field will update.
`trigger` | specify conditions for this field using [trigger events](#field-trigger-events).
`preset` | allows the field value to be initially set by the value of another field, converted using the [input preset converter](#field-input-preset).
`required` | places a red asterisk next to the field label to indicate it is required (make sure to setup validation on the model as this is not enforced by the form controller).
`attributes` | specify custom HTML attributes to add to the form field element.
`containerAttributes` | specify custom HTML attributes to add to the form field container.
`permissions` | the [permissions](users#users-and-permissions) that the current backend user must have in order for the field to be used. Supports either a string for a single permission or an array of permissions of which only one is needed to grant access.

<a name="field-types"></a>
## Available field types

There are various native field types that can be used for the **type** setting. For more advanced form fields, a [form widget](#form-widgets) can be used instead.

<style>
    .collection-method-list {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="content-list collection-method-list" markdown="1">
- [Text](#field-text)
- [Number](#field-number)
- [Password](#field-password)
- [Email](#field-email)
- [Textarea](#field-textarea)
- [Dropdown](#field-dropdown)
- [Radio List](#field-radio)
- [Balloon Selector](#field-balloon)
- [Checkbox](#field-checkbox)
- [Checkbox List](#field-checkboxlist)
- [Switch](#field-switch)
- [Section](#field-section)
- [Partial](#field-partial)
- [Hint](#field-hint)
- [Widget](#field-widget)
</div>

<a name="field-text"></a>
### Text

`text` - renders a single line text box. This is the default type used if none is specified.

```yaml
blog_title:
    label: Blog Title
    type: text
```

<a name="field-number"></a>
### Number

`number` - renders a single line text box that takes numbers only.

```yaml
your_age:
    label: Your Age
    type: number
    step: 1  # defaults to 'any'
    min: 1   # defaults to not present
    max: 100 # defaults to not present
```

If you would like to validate this field server-side on save to ensure that it is numeric, please use the `$rules` property on your model, like so:

```php
/**
 * @var array Validation rules
 */
public $rules = [
    'your_age' => 'numeric',
];
```

For more information on model validation, please visit [the documentation page](../services/validation#rule-numeric).

<a name="field-password"></a>
### Password

`password ` - renders a single line password field.

```yaml
user_password:
    label: Password
    type: password
```

<a name="field-email"></a>
### Email

`email` - renders a single line text box with the type of `email`, triggering an email-specialised keyboard in mobile browsers.

```yaml
user_email:
    label: Email Address
    type: email
```

If you would like to validate this field on save to ensure that it is a properly-formatted email address, please use the `$rules` property on your model, like so:

```php
/**
 * @var array Validation rules
 */
public $rules = [
    'user_email' => 'email',
];
```

For more information on model validation, please visit [the documentation page](../services/validation#rule-email).

<a name="field-textarea"></a>
### Textarea

`textarea` - renders a multiline text box. A size can also be specified with possible values: `tiny`, `small`, `large`, `huge`, `giant`.

```yaml
blog_contents:
    label: Contents
    type: textarea
    size: large
```

<a name="field-dropdown"></a>
### Dropdown

`dropdown` - renders a dropdown with specified options. There are 6 ways to provide the drop-down options.

The first method defines `options` directly in the YAML file(two variants):

(value only):

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
    default: published
    options:
        draft
        published
        archived
```

(key / value):

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
    default: published
    options:
        draft: Draft
        published: Published
        archived: Archived
```

The second method defines options with a method declared in the model class. If the options element is omitted, the framework expects a method with the name `get*FieldName*Options` to be defined in the model. Using the example above, the model should have the `getStatusTypeOptions` method. The first argument of this method is the current value of this field and the second is the current data object for the entire form. This method should return an array of options in the format **key => label**.

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
```

Supplying the dropdown options in the model class:

```php
public function getStatusTypeOptions($value, $formData)
{
    return ['all' => 'All', ...];
}
```

The third global method `getDropdownOptions` can also be defined in the model, this will be used for all dropdown field types for the model. The first argument of this method is the field name, the second is the current value of the field, and the third is the current data object for the entire form. It should return an array of options in the format **key => label**.

```php
public function getDropdownOptions($fieldName, $value, $formData)
{
    if ($fieldName == 'status') {
        return ['all' => 'All', ...];
    }
    else {
        return ['' => '-- none --'];
    }
}
```

The fourth method uses a specific method declared in the model class. In the next example the `listStatuses` method should be defined in the model class. This method receives all the same arguments as the `getDropdownOptions` method, and should return an array of options in the format **key => label**.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    options: listStatuses
```

Supplying the dropdown options to the model class:

```php
public function listStatuses($fieldName, $value, $formData)
{
    return ['published' => 'Published', ...];
}
```

The fifth method allows you to specify a static method on a class to return the options:

```yaml
status:
    label: Blog Post Status
    type: dropdown
    options: \MyAuthor\MyPlugin\Classes\FormHelper::staticMethodOptions
```

Supplying the dropdown options to the model class:

```php
public static function staticMethodOptions($formWidget, $formField)
{
    return ['published' => 'Published', ...];
}
```

The sixth method allows you to specify a callable object via an array definition. If using PHP, you're able to provide an array with the first element being the object and the second element being the method you want to call on that object. If you're using YAML, you're limited to a static method defined as the second element and the namespaced reference to a class as the first element:

```yaml
status:
    label: Blog Post Status
    type: dropdown
    options: [\MyAuthor\MyPlugin\Classes\FormHelper, staticMethodOptions]
```

Supplying the dropdown options to the model class:

```php
public static function staticMethodOptions($formWidget, $formField)
{
    return ['published' => 'Published', ...];
}
```

### Add icon to dropdown options

In order to add an icon or an image for every option which will be rendered in the dropdown field the options have to be provided as a multidimensional array with the following format `'key' => ['label-text', 'icon-class'],`.

```php
public function listStatuses($fieldName, $value, $formData)
{
    return [
        'published' => ['Published', 'icon-check-circle'],
        'unpublished' => ['Unpublished', 'icon-minus-circle'],
        'draft' => ['Draft', 'icon-clock-o']
    ];
}
```

To define the behavior when there is no selection, you may specify an `emptyOption` value to include an empty option that can be reselected.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    emptyOption: -- no status --
```

Alternatively you may use the `placeholder` option to use a "one-way" empty option that cannot be reselected.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    placeholder: -- select a status --
```

By default the dropdown has a searching feature, allowing quick selection of a value. This can be disabled by setting the `showSearch` option to `false`.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    showSearch: false
```

<a name="field-radio"></a>
### Radio List

`radio` - renders a list of radio options, where only one item can be selected at a time.

```yaml
security_level:
    label: Access Level
    type: radio
    default: guests
    options:
        all: All
        registered: Registered only
        guests: Guests only
```

Radio lists can also support a secondary description.

```yaml
security_level:
    label: Access Level
    type: radio
    options:
        all: [All, Guests and customers will be able to access this page.]
        registered: [Registered only, Only logged in member will be able to access this page.]
        guests: [Guests only, Only guest users will be able to access this page.]
```

Radio lists support the same methods for defining the options as the [dropdown field type](#field-dropdown). For radio lists the method could return either the simple array: **key => value** or an array of arrays for providing the descriptions: **key => [label, description]**. Options can be displayed inline with each other instead of in separate rows by specifying `cssClass: 'inline-options'` on the radio field config.

<a name="field-balloon"></a>
### Balloon Selector

`balloon-selector` - renders a list, where only one item can be selected at a time.

```yaml
gender:
    label: Gender
    type: balloon-selector
    default: female
    options:
        female: Female
        male: Male
```

Balloon selectors support the same methods for defining the options as the [dropdown field type](#field-dropdown).

<a name="field-checkbox"></a>
### Checkbox

`checkbox` - renders a single checkbox.

```yaml
show_content:
    label: Display content
    type: checkbox
    default: true
```

<a name="field-checkboxlist"></a>
### Checkbox List

`checkboxlist` - renders a list of checkboxes.

```yaml
permissions:
    label: Permissions
    type: checkboxlist
    # set to true to explicitly enable the "Select All", "Select None" options
    # on lists that have <=10 items (>10 automatically enables it)
    quickselect: true
    default: open_account
    options:
        open_account: Open account
        close_account: Close account
        modify_account: Modify account
```

Checkbox lists support the same methods for defining the options as the [dropdown field type](#field-dropdown) and also support secondary descriptions, found in the [radio field type](#field-radio). Options can be displayed inline with each other instead of in separate rows by specifying `cssClass: 'inline-options'` on the checkboxlist field config.

<a name="field-switch"></a>
### Switch

`switch` - renders a switchbox.

```yaml
show_content:
    label: Display content
    type: switch
    comment: Flick this switch to display content
    on: myauthor.myplugin::lang.models.mymodel.show_content.on
    off: myauthor.myplugin::lang.models.mymodel.show_content.off
```

<a name="field-section"></a>
### Section

`section` - renders a section heading and subheading. The `label` and `comment` values are optional and contain the content for the heading and subheading.

```yaml
user_details_section:
    label: User details
    type: section
    comment: This section contains details about the user.
```

<a name="field-partial"></a>
### Partial

`partial` - renders a partial, the `path` value can refer to a partial view file otherwise the field name is used as the partial name. Inside the partial these variables are available: `$value` is the default field value, `$model` is the model used for the field and `$field` is the configured class object `Backend\Classes\FormField`.

```yaml
content:
    type: partial
    path: $/acme/blog/models/comments/_content_field.htm
```

<a name="field-hint"></a>
### Hint

`hint` - identical to a `partial` field but renders inside a hint container that can be hidden by the user.

```yaml
content:
    type: hint
    path: content_field
```

<a name="field-widget"></a>
### Widget

`widget` - renders a custom form widget, the `type` field can refer directly to the class name of the widget or the registered alias name.

```yaml
blog_content:
    type: Backend\FormWidgets\RichEditor
    size: huge
```

<a name="form-widgets"></a>
## Form widgets

There are various form widgets included as standard, although it is common for plugins to provide their own custom form widgets. You can read more on the [Form Widgets](widgets#form-widgets) article.

<div class="content-list collection-method-list" markdown="1">
- [Code editor](#widget-codeeditor)
- [Color picker](#widget-colorpicker)
- [Data table](#widget-datatable)
- [Date picker](#widget-datepicker)
- [File upload](#widget-fileupload)
- [Markdown editor](#widget-markdowneditor)
- [Media finder](#widget-mediafinder)
- [Nested Form](#widget-nestedform)
- [Record finder](#widget-recordfinder)
- [Relation](#widget-relation)
- [Repeater](#widget-repeater)
- [Rich editor / WYSIWYG](#widget-richeditor)
- [Sensitive](#widget-sensitive)
- [Tag list](#widget-taglist)
</div>

<a name="widget-codeeditor"></a>
### Code editor

`codeeditor` - renders a plaintext editor for formatted code or markup. Note the options may be inherited by the code editor preferences defined for the Administrator in the backend.

```yaml
css_content:
    type: codeeditor
    size: huge
    language: html
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`language` | code language, for example, php, css, javascript, html. Default: `php`.
`showGutter` | shows a gutter with line numbers. Default: `true`.
`wrapWords` | breaks long lines on to a new line. Default `true`.
`fontSize` | the text font size. Default: 12.

<a name="widget-colorpicker"></a>
### Color picker
`colorpicker` - renders controls to select a color value.

```yaml
color:
    label: Background
    type: colorpicker
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`availableColors` |  list of available colors. If not provided, the widget will use the global available colors.
`allowCustom` | If `false`, only colors specified in `availableColors` will be available for selection. The color picker palette selector will be disabled. Default: `true`
`allowEmpty` | allows empty input value. Default: `false`
`formats` | Specifies the color format(s) to store. Can be a string or an array of values out of `cmyk`, `hex`, `hsl` and `rgb`. Specifying `all` as a string will allow all formats. Default: `hex`
`showAlpha` | If `true`, the opacity slider will be available. Default: `false`

There are two ways to provide the available colors for the colorpicker. The first method defines the `availableColors` directly as a list of color codes in the YAML file:

```yaml
color:
    label: Background
    type: colorpicker
    availableColors: ['#000000', '#111111', '#222222']
```

The second method uses a specific method declared in the model class.  This method should return an array of colors in the same format as in the example above. The first argument of this method is the field name, the second is the currect value of the field, and the third is the current data object for the entire form.

```yaml
color:
    label: Background
    type: colorpicker
    availableColors: myColorList
```

Supplying the available colors in the model class:

```php
public function myColorList($fieldName, $value, $formData)
{
    return ['#000000', '#111111', '#222222']
}
```

If the `availableColors` field in not defined in the YAML file, the colorpicker uses a set of 20 default colors. You can also define a custom set of default colors to be used in all color pickers that do not have the `availableColors` field specified. This can be managed in the **Customize back-end** area of the Settings.

<a name="widget-datatable"></a>
### Data table

`datatable` - renders an editable table of records, formatted as a grid. Cell content can be editable directly in the grid, allowing for the management of several rows and columns of information.

> **NOTE:** In order to use this with a model, the field should be defined as a `jsonable` attribute, or as another attribute that can handle storing arrayed data.

```yaml
data:
    type: datatable
    adding: true
    btnAddRowLabel: Add Row Above
    btnAddRowBelowLabel: Add Row Below
    btnDeleteRowLabel: Delete Row
    columns: []
    deleting: true
    dynamicHeight: true
    fieldName: null
    height: false
    keyFrom: id
    recordsPerPage: false
    searching: false
    toolbar: []
```

#### Table configuration

The following lists the configuration values of the data table widget itself.

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------ | -----------
`adding` | allow records to be added to the data table. Default: `true`.
`btnAddRowLabel` | defines a custom label for the "Add Row Above" button.
`btnAddRowBelowLabel` | defines a custom label for the "Add Row Below" button.
`btnDeleteRowLabel` | defines a custom label for the "Delete Row" button.
`columns` | an array representing the column configuration of the data table. See the *Column configuration* section below.
`deleting` | allow records to be deleted from the data table. Default: `false`.
`dynamicHeight` | if `true`, the data table's height will extend or shrink depending on the records added, up to the maximum size defined by the `height` configuration value. Default: `false`.
`fieldName` | defines a custom field name to use in the POST data sent from the data table. Leave blank to use the default field alias.
`height` | the data table's height, in pixels. If set to `false`, the data table will stretch to fit the field container.
`keyFrom` | the data attribute to use for keying each record. This should usually be set to `id`. Only supports integer values.
`postbackHandlerName` | specifies the AJAX handler name in which the data table content will be sent with. When set to `null` (default), the handler name will be auto-detected from the request name used by the form which contains the data table. It is recommended to keep this as `null`.
`recordsPerPage` | the number of records to show per page. If set to `false`, the pagination will be disabled.
`searching` | allow records to be searched via a search box. Default: `false`.
`toolbar` | an array representing the toolbar configuration of the data table.

#### Column configuration

The data table widget allows for the specification of columns as an array via the `columns` configuration variable. Each column should use the field name as a key, and the following configuration variables to set up the field.

Example:

```yaml
columns:
    id:
        type: string
        title: ID
        validation:
            integer:
                message: Please enter a number
    name:
        type: string
        title: Name
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------ | -----------
`type` | the input type for this column's cells. Must be one of the following: `string`, `checkbox`, `dropdown` or `autocomplete`.
`options` | for `dropdown` and `autocomplete` columns only - this specifies the AJAX handler that will return the available options, as an array. The array key is used as the value of the option, and the array value is used as the option label.
`readOnly` | whether this column is read-only. Default: `false`.
`title` | defines the column's title.
`validation` | an array specifying the validation for the content of the column's cells. See the *Column validation* section below.
`width` | defines the width of the column, in pixels.

#### Column validation

Column cells can be validated against the below types of validation. Validation should be specified as an array, with the type of validation used as a key, and an optional message specified as the `message` attrbute for that validation.

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Validation | Description
---------- | -----------
`float` | Validates the data as a float. An optional boolean `allowNegative` attribute can be provided, allowing for negative float numbers.
`integer` | Validates the data as an integer. An optional boolean `allowNegative` attribute can be provided, allowing for negative integers.
`length` | Validates the data to be of a certain length. An integer `min` and `max` attribute must be provided, representing the minimum and maximum number of characters that must be entered.
`regex` | Validates the data against a regular expression. A string `pattern` attribute must be provided, defining the regular expression to test the data against.
`required` | Validates that the data must be entered before saving.

<a name="widget-datepicker"></a>
### Date picker

`datepicker` - renders a text field used for selecting date and times.

```yaml
published_at:
    label: Published
    type: datepicker
    mode: date
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`mode` | the expected result, either date, datetime or time. Default: `datetime`.
`format` | provide an explicit date display format. Eg: `Y-m-d`
`minDate` | the minimum/earliest date that can be selected.
`maxDate` | the maximum/latest date that can be selected.
`firstDay` | the first day of the week. Default: 0 (Sunday).
`showWeekNumber` | show week numbers at head of row. Default: `false`
`ignoreTimezone` | store date and time exactly as it is displayed, ignoring the backend specified timezone preference.

<a name="widget-fileupload"></a>
### File upload

`fileupload` - renders a file uploader for images or regular files.

```yaml
avatar:
    label: Avatar
    type: fileupload
    mode: image
    imageHeight: 260
    imageWidth: 260
    thumbOptions:
        mode: crop
        offset:
            - 0
            - 0
        quality: 90
        sharpen: 0
        interlace: false
        extension: auto
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`mode` | the expected file type, either file or image. Default: image.
`imageWidth` | if using image type, the image will be resized to this width, optional.
`imageHeight` | if using image type, the image will be resized to this height, optional.
`fileTypes` | file extensions that are accepted by the uploader, optional. Eg: `zip,txt`
`mimeTypes` | MIME types that are accepted by the uploader, either as file extension or fully qualified name, optional. Eg: `bin,txt`
`maxFilesize` | file size in Mb that are accepted by the uploader, optional. Default: from "upload_max_filesize" param value
`useCaption` | allows a title and description to be set for the file. Default: true
`prompt` | text to display for the upload button, applies to files only, optional.
`thumbOptions` | options to pass to the thumbnail generating method for the file
`attachOnUpload` | Automatically attaches the uploaded file on upload if the parent record exists instead of using deferred binding to attach on save of the parent record. Defaults to false.

> **NOTE:** Unlike the [Media Finder FormWidget](#widget-mediafinder), the File Upload FormWidget uses [database file attachments](../database/attachments); so the field name must match a valid `attachOne` or `attachMany` relationship on the Model associated with the Form. **IMPORTANT:** Having a database column with the name used by this field type (i.e. a database column with the name of an existing `attachOne` or `attachMany` relationship) **will** cause this FormWidget to break. Use database columns with the Media Finder FormWidget and file attachment relationships with the File Upload FormWidget.

<a name="widget-markdowneditor"></a>
### Markdown editor

`markdown` - renders a basic editor for markdown formatted text.

```yaml
md_content:
    type: markdown
    size: huge
    mode: split
```

Option | Description
------------- | -------------
`mode` | the expected view mode, either tab or split. Default: `tab`.

<a name="widget-mediafinder"></a>
### Media finder

`mediafinder` - renders a field for selecting an item from the media manager library. Expanding the field displays the media manager to locate a file. The resulting selection is a string as the relative path to the file.

```yaml
background_image:
    label: Background image
    type: mediafinder
    mode: image
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`mode` | the expected file type, either file or image. Default: file.
`prompt` | text to display when there is no item selected. The `%s` character represents the media manager icon.
`imageWidth` | if using image type, the preview image will be displayed to this width, optional.
`imageHeight` | if using image type, the preview image will be displayed to this height, optional.

> **NOTE:** Unlike the [File Upload FormWidget](#widget-fileupload), the Media Finder FormWidget stores its data as a string representing the path to the image selected within the Media Library.

<a name="widget-nestedform"></a>
### Nested Form

`nestedform` - renders a nested form as the contents of this field, returns data as an array of the fields contained.

> **NOTE:** In order to use this with a model, the field should be defined as a `jsonable` attribute, or as another attribute that can handle storing arrayed data.

```yaml
content:
    type: nestedform
    usePanelStyles: false
    form:
        fields:
            added_at:
                label: Date added
                type: datepicker
            details:
                label: Details
                type: textarea
            title:
                label: This the title
                type: text
        tabs:
            meta_title:
                lable: Meta Title
                tab: SEO
            color:
                label: Color
                type: colorpicker
                tab: Design
        secondaryTabs:
            is_active:
                label: Active
                type: checkbox
            logo:
                label: Logo
                type: mediafinder
                mode: image
```

A nested form supports the same syntax as a form itself, including tabs and secondaryTabs. The jsonsable attribute, has the structure of your form definition. It's even possible to use nested forms inside a nested form.

Option | Description
------------- | -------------
`form`  | same as in [form definition](#form-fields)
`usePanelStyles` | defines if a panel like look is applied or not (defaults `true`)

<a name="widget-recordfinder"></a>
### Record finder

`recordfinder` - renders a field with details of a related record. Expanding the field displays a popup list to search large amounts of records. Supported by singular relationships only.

```yaml
user:
    label: User
    type: recordfinder
    list: ~/plugins/winter/user/models/user/columns.yaml
    recordsPerPage: 10
    title: Find Record
    prompt: Click the Find button to find a user
    keyFrom: id
    nameFrom: name
    descriptionFrom: email
    conditions: email = "bob@example.com"
    scope: whereActive
    searchMode: all
    searchScope: searchUsers
    useRelation: false
    modelClass: Winter\User\Models\User
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`keyFrom` | the name of column to use in the relation used for key. Default: `id`.
`nameFrom` | the column name to use in the relation used for displaying the name. Default: `name`.
`descriptionFrom` | the column name to use in the relation used for displaying a description. Default: `description`.
`title` | text to display in the title section of the popup.
`prompt` | text to display when there is no record selected. The `%s` character represents the search icon.
`list` | a configuration array or reference to a list column definition file, see [list columns](lists#list-columns).
`recordsPerPage` | records to display per page, use 0 for no pages. Default: 10
`conditions` | specifies a raw where query statement to apply to the list model query.
`scope` | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the list query always. The first argument will contain the model that the widget will be attaching its value to, i.e. the parent model.
`searchMode` | defines the search strategy to either contain all words, any word or exact phrase. Supported options: all, any, exact. Default: `all`.
`searchScope` | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the search query, the first argument will contain the search term.
`useRelation` | Flag for using the name of the field as a relation name to interact with directly on the parent model. Default: `true`. Set to `false` in order to bypass the relationship logic and only store and retrieve the selected record using its primary key. Best suited for use in [`jsonable` attributes](../database/model#property-jsonable) or where the relationship is unabled to be loaded. **NOTE:** When this is disabled the field name **MUST** be the actual name of the field where the value will be stored / retrieved, it cannot be the name of a relationship.
**modelClass** | Class of the model to use for listing records when useRelation = false

<a name="widget-relation"></a>
### Relation

`relation` - renders either a dropdown or checkbox list according to the field relation type. Singular relationships display a dropdown, multiple relationships display a checkbox list. The label used for displaying each relation is sourced by the `nameFrom` or `select` definition.

```yaml
categories:
    label: Categories
    type: relation
    nameFrom: title
```

Alternatively, you may populate the label using a custom `select` statement. Any valid SQL statement works here.

```yaml
user:
    label: User
    type: relation
    select: concat(first_name, ' ', last_name)
```

You can also provide a model scope to use to filter the results with the `scope` property.

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`nameFrom` | a model attribute name used for displaying the relation label. Default: `name`.
`select` | a custom SQL select statement to use for the name.
`order` | an order clause to sort options by. Example: `name desc`.
`emptyOption` | text to display when there is no available selections.
`scope` | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the list query always.

<a name="widget-repeater"></a>
### Repeater

`repeater` - renders a repeating set of form fields defined within.

```yaml
extra_information:
    type: repeater
    titleFrom: title_when_collapsed
    form:
        fields:
            added_at:
                label: Date added
                type: datepicker
            details:
                label: Details
                type: textarea
            title_when_collapsed:
                label: This field is the title when collapsed
                type: text
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`form` | a reference to form field definition file, see [backend form fields](#form-fields). Inline fields can also be used.
`prompt` | text to display for the create button. Default: `Add new item`.
`titleFrom` | name of field within items to use as the title for the collapsed item.
`minItems` | minimum items required. Pre-displays those items when not using groups. For example if you set **'minItems: 1'** the first row will be displayed and not hidden.
`maxItems` | maximum number of items to allow within the repeater.
`groups` | references a group of form fields placing the repeater in group mode (see below). An inline definition can also be used.
`style` | the behavior style to apply for repeater items. Can be one of the following: `default`, `collapsed` or `accordion`. See the **Repeater styles** section below for more information.

The repeater field supports a group mode which allows a custom set of fields to be chosen for each iteration.

```yaml
content:
    type: repeater
    prompt: Add content block
    groups: $/acme/blog/config/repeater_fields.yaml
```

This is an example of a group configuration file, which would be located in **/plugins/acme/blog/config/repeater_fields.yaml**. Alternatively these definitions could be specified inline with the repeater.

```yaml
textarea:
    name: Textarea
    description: Basic text field
    icon: icon-file-text-o
    fields:
        text_area:
            label: Text Content
            type: textarea
            size: large

quote:
    name: Quote
    description: Quote item
    icon: icon-quote-right
    fields:
        quote_position:
            span: auto
            label: Quote Position
            type: radio
            options:
                left: Left
                center: Center
                right: Right
        quote_content:
            span: auto
            label: Details
            type: textarea
```

Each group must specify a unique key and the definition supports the following options.

Option | Description
------------- | -------------
`name` | the name of the group.
`description` | a brief description of the group.
`icon` | defines an icon for the group, optional.
`fields` | form fields belonging to the group, see [backend form fields](#form-fields).

> **NOTE**: The group key is stored along with the saved data as the `_group` attribute.

#### Repeater style

The `style` attribute of the repeater widget controls the behaviour of repeater items. There are three different types of styles available for developers:

- `default:` Shows all the repeater items as expanded on page load. This is the default current behavior, and will be used if style is not defined in the repeater widget's configuration.
- `collapsed:` Shows all the repeater items as collapsed (minimised) on page load. The user can collapse or expand items as they wish.
- `accordion:` Shows only the first repeater item as expanded on load, with all others collapsed. When another item is exanded, any other expanded item is collapsed, effectively making it so that only one item is expanded at a time.

<a name="widget-richeditor"></a>
### Rich editor / WYSIWYG

`richeditor` - renders a visual editor for rich formatted text, also known as a WYSIWYG editor.

```yaml
html_content:
    type: richeditor
    toolbarButtons: bold|italic
    size: huge
```

Option | Description
------------- | -------------
`toolbarButtons` | which buttons to show on the editor toolbar.

The available toolbar buttons are:

```
fullscreen, bold, italic, underline, strikeThrough, subscript, superscript, fontFamily, fontSize, |, color, emoticons, inlineStyle, paragraphStyle, |, paragraphFormat, align, formatOL, formatUL, outdent, indent, quote, insertHR, -, insertLink, insertImage, insertVideo, insertAudio, insertFile, insertTable, undo, redo, clearFormatting, selectAll, html
```

> **NOTE**: `|` will insert a vertical separator line in the toolbar and `-` a horizontal one.

<a name="widget-sensitive"></a>
### Sensitive

`sensitive` - renders a revealable password field that can be used for sensitive information such as API keys or secrets, configuration values, etc. A sensitive field can be toggled visible and hidden at the user's request.

A sensitive field that contains a previously entered value will have the value replaced with a placeholder value on load, preventing the value from being guessed by length or copied. Upon revealing the value, the original value is retrieved by AJAX and populated into the field.

```yaml
api_secret:
    type: sensitive
    allowCopy: false
    hideOnTabChange: true
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`allowCopy` | adds a "copy" action to the sensitive field, allowing the user to copy the password without revealing it. Default: `false`
`hiddenPlaceholder` | sets the placeholder text that is used to simulate a hidden, unrevealed value. You can change this to a long or short string to emulate different length values. Default: `__hidden__`
`hideOnTabChange` | if true, the sensitive field will automatically be hidden if the user navigates to a different tab, or minimizes their browser. Default: `true`

<a name="widget-taglist"></a>
### Tag list

`taglist` - renders a field for inputting a list of tags.

```yaml
tags:
    type: taglist
    separator: space
```

A tag list support the same methods for defining the options as the [dropdown field type](#field-dropdown).

```yaml
tags:
    type: taglist
    options:
        - Red
        - Blue
        - Orange
```

You may use the `mode` called **relation** where the field name is a [many-to-many relationship](../database/relations#many-to-many). This will automatically source and assign tags via the relationship. If custom tags are supported, they will be created before assignment.

```yaml
tags:
    type: taglist
    mode: relation
```

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`mode` | controls how the value is returned, either string, array or relation. Default: `string`
`separator` | separate tags with the specified character, either comma or space. Default: `comma`
`customTags` | allows custom tags to be entered manually by the user. Default: true
`options` | specifies a method or array for predefined options. Set to true to use model `get*Field*Options` method. Optional.
`nameFrom` | if relation mode is used, a model attribute name for displaying the tag name. Default: `name`
`useKey` | use the key instead of value for saving and reading data. Default: `false`

<a name="form-views"></a>
## Form views

For each page your form supports [Create](#form-create-page), [Update](#form-update-page) and [Preview](#form-preview-page) you should provide a [view file](#introduction) with the corresponding name - `create.htm`, `update.htm` and `preview.htm`.

The form behavior adds two methods to the controller class: `formRender` and `formRenderPreview`. These methods render the form controls configured with the YAML file described above.

<a name="form-create-view"></a>
### Create view

The **create.htm** view represents the Create page that allows users to create new records. A typical Create page contains breadcrumbs, the form itself, and the form buttons. The **data-request** attribute should refer to the `onSave` AJAX handler provided by the form behavior. Below is a contents of the typical create.htm form.

```html
<?= Form::open(['class'=>'layout']) ?>

    <div class="layout-row">
        <?= $this->formRender() ?>
    </div>

    <div class="form-buttons">
        <div class="loading-indicator-container">
            <button
                type="button"
                data-request="onSave"
                data-request-data="close:true"
                data-hotkey="ctrl+enter, cmd+enter"
                data-load-indicator="Creating Category..."
                class="btn btn-default">
                Create and Close
            </button>
            <span class="btn-text">
                or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
            </span>
        </div>
    </div>

<?= Form::close() ?>
```

<a name="form-update-view"></a>
### Update view

The **update.htm** view represents the Update page that allows users to update or delete existing records. A typical Update page contains breadcrumbs, the form itself, and the form buttons. The Update page is very similar to the Create page, but usually has the Delete button. The `data-request` attribute should refer to the `onSave` AJAX handler provided by the form behavior. Below is a contents of the typical update.htm form.

```html
<?= Form::open(['class'=>'layout']) ?>

    <div class="layout-row">
        <?= $this->formRender() ?>
    </div>

    <div class="form-buttons">
        <div class="loading-indicator-container">
            <button
                type="button"
                data-request="onSave"
                data-request-data="close:true"
                data-hotkey="ctrl+enter, cmd+enter"
                data-load-indicator="Saving Category..."
                class="btn btn-default">
                Save and Close
            </button>
            <button
                type="button"
                class="wn-icon-trash-o btn-icon danger pull-right"
                data-request="onDelete"
                data-load-indicator="Deleting Category..."
                data-request-confirm="Do you really want to delete this category?">
            </button>
            <span class="btn-text">
                or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
            </span>
        </div>
    </div>

<?= Form::close() ?>
```

<a name="form-preview-view"></a>
### Preview view

The **preview.htm** view represents the Preview page that allows users to preview existing records in the read-only mode. A typical Preview page contains breadcrumbs and the form itself. Below is a contents of the typical preview.htm form.

```html
<div class="form-preview">
    <?= $this->formRenderPreview() ?>
</div>
```

<a name="field-conditions"></a>
## Applying conditions to fields

Sometimes you may want to manipulate the value or appearance of a form field under certain conditions, for example, you may want to hide an input if a checkbox is ticked. There are a few ways you can do this, either by using the trigger API or field dependencies. The input preset converter is primarily used to converting field values. These options are described in more detail below.

<a name="field-input-preset"></a>
### Input preset converter

The input preset converter is defined with the `preset` [form field option](#form-field-options) and allows you to convert text entered into an element to a URL, slug or file name value in another input element.

In this example we will automatically fill out the `url` field value when a user enters text in the `title` field. If the text **Hello world** is typed in for the Title, the URL will follow suit with the converted value of **/hello-world**. This behavior will only occur when the destination field (`url`) is empty and untouched.

```yaml
title:
    label: Title

url:
    label: URL
    preset:
        field: title
        type: url
```

Alternatively, the `preset` value can also be a string that refers to the **field** only, the `type` option will then default to **slug**.

```yaml
slug:
    label: Slug
    preset: title
```

The following options are available for the `preset` option:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`field` | defines the other field name to source the value from.
`type` | specifies the conversion type. See below for supported values.
`prefixInput` | optional, prefixes the converted value with the value found in the supplied input element using a CSS selector.

Following are the supported types:

Type | Description
------------- | -------------
`exact` | copies the exact value
`slug` | formats the copied value as a slug
`url` | same as slug but prefixed with a /
`camel` | formats the copied value with camelCase
`file` | formats the copied value as a file name with whitespace replaced with dashes

<a name="field-trigger-events"></a>
### Trigger events

Trigger events are defined with the `trigger` [form field option](#form-field-options) and is a simple browser based solution that uses JavaScript. It allows you to change elements attributes such as visibility or value, based on another elements' state. Here is a sample definition:

```yaml
is_delayed:
    label: Send later
    comment: Place a tick in this box if you want to send this message at a later time.
    type: checkbox

send_at:
    label: Send date
    type: datepicker
    cssClass: field-indent
    trigger:
        action: show
        field: is_delayed
        condition: checked
```

In the above example the `send_at` form field will only be shown if the `is_delayed` field is checked. In other words, the field will show (action) if the other form input (field) is checked (condition). The `trigger` definition specifies these options:

<style>
    .attributes-table-precessor + table td:first-child,
    .attributes-table-precessor + table td:first-child > * { white-space: nowrap; }
</style>
<div class="attributes-table-precessor"></div>

Option | Description
------------- | -------------
`action` | defines the action applied to this field when the condition is met. Supported values: show, hide, enable, disable, empty.
`field` | defines the other field name that will trigger the action. Normally the field name refers to a field in the same level form. For example, if this field is in a [repeater widget](#widget-repeater), only fields in that same [repeater widget](#widget-repeater) will be checked. However, if the field name is preceded by a caret symbol `^` like: `^parent_field`, it will refer to a [repeater widget](#widget-repeater) or form one level higher than the field itself. Additionally, if more than one caret `^` is used, it will refer that many levels higher: `^^grand_parent_field`, `^^^grand_grand_parent_field`, etc.
`condition` | determines the condition the specified field should satisfy for the condition to be considered "true". Supported values: checked, unchecked, value[somevalue].

<a name="field-dependencies"></a>
### Field dependencies

Form fields can declare dependencies on other fields by defining the `dependsOn` [form field option](#form-field-options) which provides a more robust server side solution for updating fields when their dependencies are modified. When the fields that are declared as dependencies change, the defining field will update using the AJAX framework. This provides an opportunity to interact with the field's properties using the `filterFields` methods or changing available options to be provided to the field. Examples below:

```yaml
country:
    label: Country
    type: dropdown

state:
    label: State
    type: dropdown
    dependsOn: country
```

In the above example the `state` form field will refresh when the `country` field has a changed value. When this occurs, the current form data will be filled in the model so the dropdown options can use it.

```php
public function getCountryOptions()
{
    return ['au' => 'Australia', 'ca' => 'Canada'];
}

public function getStateOptions()
{
    if ($this->country == 'au') {
        return ['act' => 'Capital Territory', 'qld' => 'Queensland', ...];
    }
    elseif ($this->country == 'ca') {
        return ['bc' => 'British Columbia', 'on' => 'Ontario', ...];
    }
}
```

This example is useful for manipulating the model values, but it does not have access to the form field definitions. You can filter the form fields by defining a `filterFields` method inside the model, described in the [Filtering form fields](#filter-form-fields) section. An example is provided below:

```yaml
dnsprovider:
    label: DNS Provider
    type: dropdown

registrar:
    label: Registrar
    type: dropdown

specificfields[for][provider1]:
    label: Provider 1 ID
    type: text
    hidden: true
    dependsOn:
        - dnsprovider
        - registrar

specificfields[for][provider2]:
    label: Provider 2 ID
    type: text
    hidden: true
    dependsOn:
        - dnsprovider
        - registrar
```

And the logic for the filterFields method would be as follows:

```php
public function filterFields($fields, $context = null)
{
    $displayedVendors = strtolower($this->dnsprovider->name . $this->registrar->name);
    if (str_contains($displayedVendors, 'provider1')) {
        $fields->{'specificfields[for][provider1]'}->hidden = false;
    }
    if (str_contains($displayedVendors, 'provider2')) {
        $fields->{'specificfields[for][provider2]'}->hidden = false;
    }
}
```

In the above example, both the `provider1` and `provider2` fields will automatically refresh whenever either the `dnsprovider` or `registrar` fields are modified. When this occurs, the full form cycle will be processed, which means that any logic defined in `filterFields` methods would be run again, allowing you to filter which fields get displayed dynamically.

<a name="prevent-field-submission"></a>
### Preventing a field from being submitted

Sometimes you may need to prevent a field from being submitted. In order to do that, just add an underscore (\_) before the name of the field in the form configuration file. Form fields beginning with an underscore are purged automatically and no longer saved to the model.

```yaml
address:
    label: Title
    type: text

_map:
    label: Point your address on the map
    type: mapviewer
```

<a name="extend-form-behavior"></a>
## Extending form behavior

Sometimes you may wish to modify the default form behavior and there are several ways you can do this.

<a name="form-model-events"></a>
### Form Model Events

Several controller methods can called at various points during the lifecycle of the `FormController` to provide injection points for custom logic. See the [API docs](/docs/api/develop/Backend/Behaviors/FormController.html#method_formBeforeSave) for a full reference of what they are. Generally speaking any method in the API docs prefixed with `form` can be overridden in your controller to change the default behaviour or act as an injection point for custom logic.

> **NOTE:** It may be more desirable to use [model events](/docs/events/event/model.beforeSave) to implement your logic instead as those are always run when applicable if the model is being affected, no matter where the interaction with the model is occuring.

<a name="overriding-action"></a>
### Overriding controller action

You can use your own logic for the `create`, `update` or `preview` action method in the controller, then optionally call the Form behavior parent method.

```php
public function update($recordId, $context = null)
{
    //
    // Do any custom code here
    //

    // Call the FormController behavior update() method
    return $this->asExtension('FormController')->update($recordId, $context);
}
```

<a name="overriding-redirect"></a>
### Overriding controller redirect

You can specify the URL to redirect to after the model is saved by overriding the `formGetRedirectUrl` method. This method returns the location to redirect to with relative URLs being treated as backend URLs.

```php
public function formGetRedirectUrl($context = null, $model = null)
{
    return 'https://example.com';
}
```

<a name="extend-model-query"></a>
### Extending model query

The lookup query for the form [database model](../database/model) can be extended by overriding the `formExtendQuery` method inside the controller class. This example will ensure that soft deleted records can still be found and updated, by applying the **withTrashed** scope to the query:

```php
public function formExtendQuery($query)
{
    $query->withTrashed();
}
```

<a name="extend-form-fields"></a>
### Extending form fields

You can extend the fields of another controller from outside by calling the `extendFormFields` static method on the controller class. This method can take three arguments, **$form** will represent the Form widget object, **$model** represents the model used by the form and **$context** is a string containing the form context. Take this controller for example:

```php
class Categories extends \Backend\Classes\Controller
{
    public $implement = ['Backend.Behaviors.FormController'];

    public $formConfig = 'config_form.yaml';
}
```

Using the `extendFormFields` method you can add extra fields to any form rendered by this controller. Since this has the potential to affect all forms used by this controller, it is a good idea to check the **$model** is of the correct type. Here is an example:

```php
Categories::extendFormFields(function($form, $model, $context)
{
    if (!$model instanceof MyModel) {
        return;
    }

    $form->addFields([
        'my_field' => [
            'label'   => 'My Field',
            'comment' => 'This is a custom field I have added.',
        ],
    ]);

});
```

You can also extend the form fields internally by overriding the `formExtendFields` method inside the controller class. This will only affect the form used by the `FormController` behavior.

```php
class Categories extends \Backend\Classes\Controller
{
    [...]

    public function formExtendFields($form)
    {
        $form->addFields([...]);
    }
}
```

The following methods are available on the $form object.

Method | Description
------------- | -------------
`addFields` | adds new fields to the outside area
`addTabFields` | adds new fields to the tabbed area
`addSecondaryTabFields` | adds new fields to the secondary tabbed area
`removeField` | remove a field from any areas

Each method takes an array of fields similar to the [form field configuration](#form-fields).

<a name="filter-form-fields"></a>
### Filtering form fields

You can filter the form field definitions by overriding the `filterFields` method inside the Model used. This allows you to manipulate visibility and other field properties based on the model data. The method takes two arguments **$fields** will represent an object of the fields already defined by the [field configuration](#form-fields) and **$context** represents the active form context.

```php
public function filterFields($fields, $context = null)
{
    if ($this->source_type == 'http') {
        $fields->source_url->hidden = false;
        $fields->git_branch->hidden = true;
    }
    elseif ($this->source_type == 'git') {
        $fields->source_url->hidden = false;
        $fields->git_branch->hidden = false;
    }
    else {
        $fields->source_url->hidden = true;
        $fields->git_branch->hidden = true;
    }
}
```

The above example will set the `hidden` flag on certain fields by checking the value of the Model attribute `source_type`. This logic will be applied when the form first loads and also when updated by a [defined field dependency](#field-dependencies).

<a name="validate-form-fields"></a>
## Validating form fields

To validate the fields of your form you can make use of the [Validation](../database/traits#validation) trait in your model.
