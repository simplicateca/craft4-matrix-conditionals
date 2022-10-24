# Conditional Matrix Fields for Craft CMS 4.x

This tool is a stop-gap solution until Craft CMS core provides functionality for conditional fields inside matrix blocks.

This method uses a single twig file (`matrix-conditionals.twig`) that you can drop into your Craft CMS entry type field layouts as a `Template UI Element`.

Configuration is handled by editing a simple JSON-like configuration varible near the top of the twig file.

Developers can create conditional rules for when certain fields appear (or are hidden) based on the value of other fields within the same matrix block.

While this method is capable of conditionally hiding or displaying fields for a wide variety of use cases, extremely complicated or interconnected conditions might be difficult (or impossible) to manage with this method.

This tool does not prevent fields from loading and it does not remove them from the form. It simply hides them with CSS, allowing you create a more streamlined user experience for content editors (Hooray for better AX!).

This tool isn't perfect, and it doesn't do everything, but it's better than what we currently have :)


## Getting Started
 
1) Save the `matrix-conditionals.twig` file from this repository to a location in your `cms/templates` directory.
2) Configure the JavaScript at the top of the twig file to describe your conditional rules and actions based on the configuration information in these instructions.
3) Any time you add a matrix field to the field layout for an entry type (and you want to set condition for some of its fields), add a `Template UI Element` below the matrix field that references the `matrix-conditionals.twig` file you saved in step 1.

![Adding the matrix-conditionals.twig file as a Template UI Element in your Entry Type Field Layout](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/install.jpg)

**That's it.**

No plugins to install. No custom modules or assetbundles to configure. Just a twig file included in your field layout through the Craft admin UI.


### Quick Overview

Sample configuration:

    const matrixConditionalsConfig = {
        'contentBuilder.*.background': {
            ':empty:': {
                showOnUnequal: 'coverage',
                hideOnEqual: 'coverage'
            },
            'chooseCustom': {
                showOnEqual: 'backgroundColor',
                hideOnUnequal: 'backgroundColor'
            }
        },

        '*.video.videoSource': {
            'youtube': {
                showOnEqual: ['externalUrl']
            },
            'localAsset': {
                showOnEqual: ['mp4File'],
                hideOnEqual: ['externalUrl']
            }
        }
    }

The above config would react to changes in the values of two fields:

  1) Any field with the handle `background` inside any block type in the `contentBuilder` matrix field.

  2) Any field with the handle `videoSource` inside any `video` block type in any matrix field.

Based on the value of the `background` field, the fields `coverage` and `backgroundColor` would be revealed or hidden.

Based on the value of the `videoSource` field, the fields `externalUrl` and `mp4File` would be revealed or hidden.


Here are some screenshots of what this would look like for the `background` field:

> `background=':empty:'` 

Coverage field and Custom Color field are hidden

![A Background Field with no option selected](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/sample-1.jpg)

----

> `background=':notempty:'` 

Coverage field is visible, Custom Color field is still hidden

![A field labled "background" with the "White" option selected, and a field labled "Coverage" now visible beside it](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/sample-2.jpg)

----

> `background='chooseCustom'` 

Both the Coverage field and Custom Color field are visible

![A field labled "background" with the "Choose Custom" option selected, and fields labled "Coverage" and a Custom Color selector field now visible beside it](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/sample-3.jpg)

----

There are no limits to how many fields you can watch, or how many fields you can show/hide based on the value of fields you are watching.

**Conditional rules can only match against:**

 - Whether a field is `:empty:`
 - Whether a field is `:notempty:`
 - Whether a field has a specific value

There are no rules for matching against conditions like: value greater than, value less than, value contains X, etc.


## All Configuration Options

Conditional rules and actions are set in a simple JSON object at the top of the twig file organized like this:

    'matrixHandle.blockType.fieldHandle' : {
        'valueToWatchFor' : {
            showOnEqual   : ['fieldHandles', 'toMakeVisible', 'whenValueMatches'],
            showOnUnequal : ['fieldHandles', 'toMakeVisible', 'whenValueDoesNotMatch'],
            hideOnEqual   : ['fieldHandles', 'toHide', 'whenValueMatches'],
            hideOnUnequal : ['fieldHandles', 'toHide', 'whenValueDoesNotMatch']
        }
    }

You can have multiple versions of this file if you have multiple matrix fields, but if your rules aren't overly complicated this file can be configured to handle multiple matrix fields at once.

**Sample JavaScript config object:**

    const matrixConditionalsConfig = {
        // for any `background` field in any block type in the `contentBuilder` matrix field
        'contentBuilder.*.background': {
        
            // if the value is set to "color", show the `backgroundColor` field, otherwise hide it    
            'color': {
                showOnEqual: 'backgroundColor',
                hideOnUnequal: 'backgroundColor'
            }
        },
        
        // for the `videoSource` field in any `video` block type in and matrix field
        '*.video.videoSource': {
            // if the value is set to "youtube", show the `externalUrl` and `coverImage` fields
            'youtube': {
                showOnEqual: ['externalUrl', 'coverImage'],    
            },
                    
            // if the value is set to "vimeo", show the `externalUrl` and `coverImage` fields
            'vimeo': {
                showOnEqual: ['externalUrl', 'coverImage'],
            },
        
            // if the value is set to "localAsset", hide the `externalUrl` and `coverImage` 
            // fields, and show the `videoAsset` field, otherwise hide the `videoAsset` field.
            'localAsset': {
                hideOnEqual: ['externalUrl', 'coverImage'],
                showOnEqual: ['videoAsset'],
                hideOnUnequal: ['videoAsset']
            }
        },

        // when the image asset field has at least one option, show the caption field
        '*.*.imageAsset' : {
            ':notempty:': {
                showOnEqual: 'imageCaption',
                hideOnEqual: 'imageCaption'
            }
        },

        // when the table field has at least one row, show the notes field
        '*.*.table' : {
            ':notempty:': {
                showOnEqual: 'tableNotes',
                hideOnEqual: 'tableNotes'
            }
        },

        // when the lightswitch field is 'on' make the Light Color and Brightness fields visible
        '*.*.lightswitch' : {
            ':notempty:': {
                showOnEqual: ['lightColor', 'brightness'],
                hideOnUnequal: ['lightColor', 'brightness']
            }
        },

        // [NOTE] checkboxes are evaluated independantly as if they were individual lightswitch fields, not as part of a group
        // if the `customButtonText` checkbox in the `customizeButton` field is checked, show the `alternateButtonText` field
        // if the `customButtonColor` checkbox in the `customizeButton` field is checked, show the `alternateButtonColor` field
        '*.*.customizeButton' : {
            'customButtonText': {
                showOnEqual: 'alternateButtonText',
                hideOnUnequal: 'alternateButtonText'
            },
            'customButtonColor': {
                showOnEqual: 'alternateButtonColor',
                hideOnUnequal: 'alternateButtonColor'
            }
        },
    }


### Targetting Fields to Watch

Select the fields whose values you want to watch using their full matrix field handle, block type handle, and field handle, with a dot (.) separating each handle. i.e.

	`matrixHandle.blockType.fieldHandle`

You may use `*` as a wildcard like this: `*.*.fieldHandle`

**Examples**

`contentBuilder.imageBlock.imagePosition`

The `imagePosition` field inside any `imageBlock` block type in the `contentBuilder` matrix

--

`*.imageBlock.imagePosition`

The `imagePosition` field inside any `imageBlock` block type in any matrix

--

`contentBuilder.*.backgroundColor`

The `backgroundColor` field inside any block type in in the `contentBuilder` matrix

--

`*.*.backgroundColor`

All `backgroundColor` fields inside any block type in any matrix


### Matching Empty & Non-Empty Values

You can test whether or not a field is empty by using the special values `:empty:` and `:notempty:`.

This is the most widely supported type of matching across all field types available.

The following configurations will result in the same actions:

    ':empty:': {
        showOnUnequal: 'coverage',
        hideOnEqual: 'coverage'
    },
----
    ':notempty:': {
	    showOnEqual: 'coverage',
        hideOnUnequal: 'coverage'
	},


### Show / Hide Actions

Depending on how numerous and complicated your conditional rules are, you may need to list all possible values for a field, but maybe just the one you want to watch for.

There are four types of actions that we can perform on other fields based on the value of the field we are watching:

- `showOnEqual` - show one (or more) fields when the value of this field matches the condition value

- `hideOnEqual` - hide one (or more) fields when the value of this field matches the condition value

- `showOnUnequal` - show one (or more) fields when the value of this field *does not* matches the condition value

- `hideOnUnequal` - hide one (or more) fields when the value of this field *does not* matches the condition value


For each of the above action types, we can provide one or more fields to become visible (or hidden).

- Single fields can be passed as a string `'image'` or an array `['image']`

- Multiple fields must always be passed as an array `['image','imagePosition']`


### Supported Field Types

Most of the standard Craft CMS field types are supported. Not all fields support all conditions.

 **Dropdown / Select**

Matches `"optionValue"`  `:empty:`  `:notempty:`

---

**Plain Text**

Matches `"Exact Text Match"`  `:empty:`  `:notempty:`

---

**Radio Buttons**

Matches `"optionValue"`  `:empty:`  `:notempty:`

---

**Number**

Matches `"Exact Number"`  `:empty:`  `:notempty:`

---

**Checkboxes**

Matches `"optionValue"`only when checked.

_Additional Caveat -_ Checkboxes are evaluated individually, not as a group. They can not be `:empty:` or `:notempty:`

Sample checkbox config:

    '*.*.customizeButton' : {
	    'customButtonText': {
		    showOnEqual: 'buttonText',
		    hideOnUnequal: 'buttonText'
	    },
	    'customButtonColor': {
		    showOnEqual: 'buttonColor',
		    hideOnUnequal: 'buttonColor'
	    }
    }

With the above configuration, two checkboxes in the "Customize Button" field, independantly control the display of two other fields: `buttonText` and `buttonColor`. i.e.

![Two checkbox fields controlling the display of two other fields](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/checkboxes.jpg)

---

**Lightswitch**

Matches `:empty:` when the swtich is off, and `:notempty:`  when the switch is on.


---

**Entries**

Matches `:empty:` when there are no entries selected, and `:notempty:` when at least one entry is seleted. Can not test for specific entries.

---

**Assets**

Matches `:empty:` when no assets are selected, and `:notempty:` when at least one asset is seleted. Can not test for specific assets or types.

---

**Table**

Matches `:empty:` when the table has no rows, or `:notempty:` when at least one row exists. Can not test for specific columns or rows within the table.

### Currently Unsupported Field Types

The following fields have not been tested and/or are not currently supported to base your conditional rules on.

You may still show/hide fields of these types based on the value of a supported field type.

- Categories
- Color
- Date
- Email
- Money
- Multi-Select
- Redactor (or any other richtext/WYSIWYG field)
- Tags
- Time
- URL
- User

Of the above, Date, Email, Money, Time, and URL are the most likely to work out of the box, they just have not been tested yet. Users has a shot at working too, assuming it's structured similarly to Entries and Assets.

Tags and Categories might also be possible, but probably a long shot without additional work. Plus, these field types are either going away or changing drastically soon, so there's not much point in stressing about them.

Some custom fields *might* be supported out of the box, depending on their underlying form control structure, but I can't make any promises and have done minimal testing.


## Alternate Use - Protection Against 'PEBKAC' Errors

You can also use this tool to hide certain fields from content editors that don't have sufficient permissions (or knowledge) to change them.

To do this, you would use the "Current User Condition" conditional rule when adding the `UI Template Element` to your entry type, and setting a different twig file to load for a certain users / user types.

![Adding the matrix-conditionals.twig file as a Template UI Element in your Entry Type Field Layout with the "Current User Condition" field highlighted](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/install-user-conditions.jpg)

Within this alternate file you might configure your conditions like this:

    const matrixConditionalsConfig = {

        // hide the `complicatedApiQueryString` field any `externalApiCall` matrix blocks for all users
        // while still allowing anyone to change the 'numberOfResults' that appear
        '*.externalApiCall.numberOfResults': {
            ':empty:': {
                hideOnEqual: 'complicatedApiQueryString',
                hideOnUnequal: 'complicatedApiQueryString'
            }
        }
    }

You should **NOT** load two copies of this file into the same entry type field layout. Only attempt to do so if you are positive that your "Current User Condition" rules are set to load completely separate versions based on the level of access you want to grant to your users.

_Also note -_ This will not completely prevent unpriviledged users from accessing hidden fields. Technically they could still write content to them since the fields will still exist in the DOM. This will only help to reduce the chances that a user might break something important that they shouldn't have been touching.


## Caveats

**Hidden fields still save data**

If you enter a value in a field, trigger a condition that hides that field, and then save your matrix block, the value you entered in the field prior to hiding it will still be saved to the database.

Your twig templates should not assume that because a field was hidden, it will have no value or because a field has a value that it was not hidden.


**Required fields are still required even when hidden**

If you set a field to be **required** and allow it to be hidden by a conditional setting, you might not be able to save your entry. Be careful.


**Conditional rules do not interact with each other**

Show/hide toggling is very binary, and not very smart. If two rules are triggered at the same time, both rules will be applied in order. You are responsible for being smart.

It is possible to create conditional rules that might make your matrix blocks uneditable or prevent certain fields from ever being visible (if you're not careful)

A rule like the following would likely result in the `customColor` field never being visible:

    '*.*.background': { 
	   'custom': { showOnEqual: 'customColor' }
	   'white': { hideOnUnequal: 'customColor' }
	}


**Hidden fields maintain layout position**

Fields are hidden with a class that has them retain their position in the field layout but otherwise hides them. If you would like fields to not maintain their layout position, you could add `display:none;` to the `matrixConditionals--hidden` class in the `{% css %}` block.


**Removing conditional rules**

Since all this tool does is hide fields with CSS after the form has loaded, it is *unlikely* to cause any isses with your underlying entry/matrix data.

As mentioned earlier, it is possible to create a combination of conditional rules that prevents a field from ever being visible. You are responsible for making sure that doesn't happen.

Removing the `Template UI Element` that references the `matrix-conditionals.twig` file from your field layout, or emptying the JS config object inside that file will return your matrix to normal working order and all fields should be visible and editable.


## Debugging

Debug can be turned on or off. Debug is current set to default to `true` (for beta testing purposes).

Change `debug =  true` to `debug = false` where `matrixConditionals.init` is called.

When debug is enabled, matching conditions and actions will be logged to the console.


## Compatibility

This has been confirmed to work in the following versions of Craft CMS:

 - v4.2.7

It's likely that it works in other versions as well, but as some of the fields require knowledge of the underlying HTML & form control structure, it's possible that core changes to that structure may effect some field types.

More versions will be added to this list as they are tested. Please let me know if you have this working successfully on any versions not listed here.

If a new version breaks anything, we will release an updated version of the code tagged to that specific release.

This tool is also confirmed to function properly if you are using [MatrixMate](https://github.com/vaersaagod/matrixmate). However, while it is possible to hide all of the fields within a MatrixMate tab, the tab itself will remain.


## Roadmap

- Get additional field types working
- Test on additional versions of Craft
- Improve a11y support for fields that are hidden
- Consider providing the ability to show/hide entire MatrixMate tabs
- Deprecate entirely once conditional fields in matrix blocks becomes part of the Craft CMS core


## Credits

Brought to you by  [simplicate.ca](https://www.simplicate.ca). Written by [Steve Comrie](https://github.com/stevecomrie).

Thanks to [Mats Mikkel Rummelhoff](https://github.com/mmikkel) for suggestions and improvement ideas.

Bug reports, feedback, and pull requests welcome.