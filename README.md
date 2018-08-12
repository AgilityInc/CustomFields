# Agility Custom Fields

This repo serves as an example of how to build custom field types for Agility CMS.

- Build your own re-usable field types
- Compatible with any Agility instance, module or content input form
- Use HTML, CSS and JS
- Content Manager Development Mode for testing
- Integrate with CMS UI
- Code hosted within Agility or externally

## Examples
1. [Friendly URL field](friendly-url) - Auto-generates a friendly URL value based off another text field

2. [Color Picker field](colorpicker) - Allows editors to select a color from a color-picker and save the value as rgba/hex 

## How it Works
On load, the **Content Manager** will check for the existence of a a single **Input Form Customization JS file**. If one has been set, the JS will be executed and consequently register each Custom Field declaration you have within the file. This will tell the **Content Manager** which custom fields are available for use, and allow you add a custom field to any content/module definition.

When a custom field is rendered within a content/module input form in the browser, your custom code is executed. A custom field typically includes your registration JS function (including the code that is executed when the field is rendered) and a reference to an HTML file representing your presentation layer.

## Setup

### Register an Input Form Customization JS File
You **must** have an **Input Form Customization** JS file referenced in the Content Manager. This can be found in **Settings > Development Framework**.

![custom field input form customization](screenshots/custom-field-input-form-customization.png?raw=true "Input Form Customization File")

First thing you need to decide is whether your JS file will be stored and managed within Agility (via Inline Code) or hosted externally.

**Inline Code:**
- Built-in code editor within the browser
- Code stored in high availability Agility CDN over a dynamic URL
- Has version history

**Custom Script URL:**
- Can be deployed anywhere
- Easier to intergrate into source control and CI/D pipelines
- Use your own IDE

#### Input Form Customization File Format
This file is comprised of one or more custom field function declarations and its registration. 

The **Content Manager** uses [KnockoutJS](http://knockoutjs.com/) for declaritive bindings, automatic ui refresh, dependancy tracking, and templating. While you don't need to use KnockoutJS to build a custom field, you will still likely interface with **Content Manager** KnockoutJS observable objects such as *contentItem* and *fieldBinding*. It is recommended to have a basic understanding of how Knockout observables work.

**Boilerplate Custom Field Function Declaration:**
```javascript
var CustomFieldFunctionDeclaration = function () {

    this.Label = "Sample Custom Field"; //[Requried] the name of the custom field as will appear in content/module def form builder
    this.ReferenceName = "SampleCustomField"; //[Required] the reference name of the custom field for internal purposes

    this.Render = function (options) {
        /// <summary>[Optional] This function is called whenever the field is rendered in an input form - this includes after the item has been saved and the field is re-rendered.</summary>
        /// <param name="options" type="Object">
        ///     <field name="$elem" type="jQueryElem">The .field-row jQuery Dom Element.</field>
        ///     <field name="contentItem" type="ContentItem Object">The entire Content Item object including Values and their KO Observable properties of all other fields on the form.</field>
        ///     <field name="fieldBinding" type="KO Observable">The value binding of thie Custom Field Type. Get and set this field's value by using this property - i.e. fieldBinding('new val')</field>
        ///     <field name="fieldSetting" type="Object">Object representing the field's settings such as 'Required', 'Hidden', 'Label', and 'Description'</field>
        ///     <field name="readonly" type="boolean">Represents if this field should be readonly or not.</field>
        /// </param>
    }

    /// <field name="Template" type="String">[Optional] The absoulte URL to an HTML template that represents your custom field, or the referencename to an Inline Code file. Your ViewModel will be automatically bound to this template.</field>
    this.Template = 'https://domain.com/html/somefile.html';

    /// <field name="DepenenciesJS"> type="Array">[Optional] The Javscript dependencies that must be loaded before your ViewModel is bound. They will be loaded in the order you specify.</field>
    this.DependenciesJS = [
        { id: 'somejs-reference-name', src: 'https://domain.com/js/somefile.js' } //src is an absolute URL to a JS file, or the referencename to an Inline Code file.
    ];

    /// <field name="DepenenciesCSS" type="Array">[Optional] The CSS dependencies that must be loaded before your ViewModel is bound. They will be loaded in the order you specify.</field>
    this.DependenciesCSS = [
        { id: 'somecss-reference-name', src: 'https://domain.com/css/somefile.css' } //src is an absolute URL to a CSS file, or the referencename to an Inline Code file.
    ];


    /// <field name="ViewModel" type="KO ViewModel">The KO ViewModel that will be binded to your HTML template</field>
    this.ViewModel = function (options) {
        /// <summary>[Optional] The KO ViewModel that will be binded to your HTML template.</summary>
        /// <param name="options" type="Object">
        ///     <field name="$elem" type="jQueryElem">The .field-row jQuery Dom Element.</field>
        ///     <field name="contentItem" type="ContentItem Object">The entire Content Item object including Values and their KO Observable properties of all other fields on the form.</field>
        ///     <field name="fieldBinding" type="KO Observable">The value binding of thie Custom Field Type. Get and set this field's value by using this property.</field>
        ///     <field name="fieldSetting" type="Object">Object representing the field's settings such as 'Hidden', 'Label', and 'Description'</field>
        ///     <field name="readonly" type="boolean">Represents if this field should be readonly or not.</field>
        /// </param>

        this.value = options.fieldBinding; //reference the field KO observable value
        this.contentID = options.contentItem.ContentID; //set the contentID of the current loaded item (NewItem = -1)
        this.attrBinding = {}; //pass any custom attributes to the input field

        if (options.fieldSetting.Settings.Required === "True") {
            //if this field is marked as required, add a required parsley attribute
            this.attrBinding['data-parsley-required'] = true;
        }

    }
}
```

**Boilerplate HTML Template:**
```html
<div class="sample-field">
    <input type="text" class="form-control" data-bind="value: value, attr: attrBinding" />
</div>
```

**Register the Custom Field:**
```javascript
ContentManager.Global.CustomInputFormFields.push(new CustomFieldFunctionDeclaration());
```


### Verifying your Input Form Customization JS File
1. After you've set your **Input Form Customization JS file** and added a custom field declaration funciton and registration, you will need to refresh the Content Manager to refresh the loaded JS file. 

2. Verify your custom input field has been registered by going to **Settings > Content Definitions (or Module Definitions)** and create or update an existing definition. Add a new field, and select the **Custom Field** tab. If successful, you should see your custom field listed under **Field Type**.

![custom field options](screenshots/custom-field-options.png?raw=true "Custom Field Options")

## Testing
There are several ways to test new/changes to a custom field type.

1. For new custom fields, create a test environment for your field by creating a *test* content/module definition that you can use just for testing. If it works there, it will work in ANY definition.

2. For updates to existing custom fields, create a **new version** of the custom field type in your Input Form Custimization file (i.e. "Sample Field v2") and test in a *test* content/module definition.

3. If making updates to an existing custom field, and creating test definitions is not feasible then you can make use of a **Development Mode** within the **Content Manager** to test changes on the existing definition forms without affecting the forms for any editors.

### Advanced Testing Development Mode
Special care should be taken when making code changes to an existing custom field type that is being used across numerous types of content. Creating **new versions** of custom fields and *test* definitions can be cumbersome at times. 

To address this, the **Content Manager** has a **Development Mode** that can be turned on and off by a developer. Turning **Development Mode** *ON* will set *your browser session only* to pull-in a **Dev Mode Input Form Customization JS file** . This allows you to create a copy of your current **Input Form Customization JS file**, specify it as your **Development Mode** JS file and make any required changes and test within the browser. 

It is important to note that only your browser session will pull-in this **Development Mode** JS file. When development is complete, you can simply replace the live file with your dev file.

#### Detecting Development Mode in JS
In your custom field code, you may want to have conditional logic when running in **Development Mode**. An example of this could be changing some variable values such as API endpoints, or other references for testing purposes.

**Detect Development Mode:**
```javacript
if(ContentManager.Developer.DevMode()) {
    //do something 
}
```

#### Turning ON Development Mode
1. In the **Content Manager** go to **Settings > Development Framework**
2. Click on **Turn on Development Mode**
3. The application will refresh
4. An *pylon* icon will now appear in the status bar showing that you are in **Development Mdoe**

![cms development mode](screenshots/cms-development-mode.png?raw=true "Development Mode")

#### Turning OFF Development Mode
1. Click on the *pylon* in the status bar
2. Confirm and click *OK* on the dialogue window

OR

1. Go to **Settings > Development Framework**
2. Click *Turn off Development Mode*



