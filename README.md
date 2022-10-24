# Craft CMS 4 - Conditional Matrix Fields

A twig file (`matrix-conditionals.twig`) that can be added to your Entry Types as a `Template UI Element` and can be configured to with basic conditional rules to allow the value of specific fields to control whether other fields within the same block are visible or hidden.

## What Does This Do?

Allows you to watch most matrix block field types, and immediately make other fields (within the same block) visible or hidden based on the value of the field you are watching.

**For example:**

This configuration: 

    'contentBuilder.*.background': {
        ':empty:': {
            'showOnUnequal': 'coverage',
            'hideOnEqual': 'coverage'
        },
        'chooseCustom': {
            'showOnEqual': 'backgroundColor',
            'hideOnUnequal': 'backgroundColor'
        }
    },

Would watch any fields with the handle `background` inside of any block types within the matrix field  with the handle `contentBuilder`

Conditional fields within the block would react based on the value of the `background` field like so:

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


## Storing Field Values & Required Fields

This does not remove fields from the layout or the form. It simply hides them.

If you enter a value in a field, trigger a condition that hides that field, and then save your matrix block, the value you entered in the field prior to hiding it will still be saved to the database.

Your twig templates should not assume that because a field was hidden, it will have no value.

Required fields are still considered required even if they are hidden. If you set a field to be **required** and allow it to be hidden by a conditional setting, you might not be able to save your entry. Be careful.




## Other Caveats

This is a beta. It is *unlikely* to mess with your data, but it is possible to create conditional rules that might make your matrix blocks uneditable or prevent certain fields from ever being visible (if you're not careful)

Show/hide toggling is very binary, and not very smart. If two rules are triggered at the same time, both rules will be applied in order. You are responsible for being smart.

A rule like the following would likely result in the `customColor` field never being visible:

    '*.*.background': { 
	   'custom': { showOnEqual: 'customColor' }
	   'white': { hideOnUnequal: 'customColor' }
	}

Fields are hidden with a class that has them retain their position in the field layout but otherwise hides them. If you would like fields to not maintain their layout position, you could add `display:none;` to the `matrixConditionals--hidden` class in the `{% css %}` block.

Removing the Template UI Element that references the `matrix-conditionals.twig` file (or emptying the JS config object inside that file) will return your matrix to normal working order and all fields should be visible and editable.

Conditional rules can only match against:

 - The exact value of a field (or it's Option Value).
 - Whether a field is `:empty:`
 - Whether a field is `:notempty:`

There are no condition rules for matching things like greater than, less than, contains, etc.

It's not perfect, but it's better than what we have now :)

## How to Get Started
 
1) Save the `matrix-conditionals.twig` file from this repository to a location in your `cms/templates` directory.
2) Configure the JavaScript at the top of the file to describe your conditional rules and actions based on the configuration information in these instructions.
3) Any time you add a matrix field to an entry types' field layout, also include this file as a `Template UI Element` (not necessary if your matrix field doesn't contain any conditionals).

![Adding the matrix-conditionals.twig file as a Template UI Element in your Entry Type Field Layout](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/github/twig-conditionals/install.jpg)

**That's it.**

No plugins to install. No custom modules to configure. Just a twig file included in your field layout.


## Configuration

Field conditionals and actions are configured via a simple JSON object at the top of the twig file.

You can have multiple versions of this file if you have multiple matrix fields, but if your fields aren't overly complicated this file can be configured to handle multiple matrix fields at once.

Sample JavaScript config object:

    // for any `background` field in any block type in the `contentBuilder` matrix field
    'contentBuilder.*.background': {
    
        // if the value is set to "color", show the `backgroundColor` field, otherwise hide it    
        'color': {
            'showOnEqual': 'backgroundColor',
            'hideOnUnequal': 'backgroundColor'
        }
    },
    
    // for the `videoSource` field in any `video` block type in and matrix field
    '*.video.videoSource': {
        // if the value is set to "youtube", show the `externalUrl` and `coverImage` fields
        'youtube': {
            'showOnEqual': ['externalUrl', 'coverImage'],    
        },
                
        // if the value is set to "vimeo", show the `externalUrl` and `coverImage` fields
        'vimeo': {
            'showOnEqual': ['externalUrl', 'coverImage'],
        },
    
        // if the value is set to "localAsset", hide the `externalUrl` and `coverImage` 
        // fields, and show the `videoAsset` field, otherwise hide the `videoAsset` field.
        'localAsset': {
            'hideOnEqual': ['externalUrl', 'coverImage'],
            'showOnEqual': ['videoAsset'],
            'hideOnUnequal': ['videoAsset']
        }
     }


### Identifying Fields

Choose the fields whose values you want to watch using the matrix field handle, block type, and field handle, with a dot separating each handle.

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


### Matching Empty & Non-Empty fields

You can test whether or not a field is empty by using the special values `:empty:` and `:notempty:`.

The following configurations will result in the same action:

    ':empty:': {
        'showOnUnequal': 'coverage',
        'hideOnEqual': 'coverage'
    },
----
    ':notempty:': {
	    'showOnEqual': 'coverage',
		'hideOnUnequal': 'coverage'
	},


### Actions On Value Match (or Not Match)

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
		    'showOnEqual': 'buttonText',
		    'hideOnUnequal': 'buttonText'
	    },
	    'customButtonColor': {
		    'showOnEqual': 'buttonColor',
		    'hideOnUnequal': 'buttonColor'
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
The following fields have not been tested and/or are not currently supported to base your conditional rules on. You may still show/hide fields of these types based on the value of a supported field type.

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


## Debugging
Debug can be turned on or off. Debug is current set to default to `true` (for beta testing purposes).

Change `debug =  true` to `debug = false` where `matrixConditionals.init` is called.

When debug is enabled, matching conditions and actions will be logged to the console.

## Confirmed Working

As of right now, this has been confirmed to work in the following versions of Craft CMS:

 - v4.2.7

More versions will be added as they are tested. If a new version breaks anything, we will release a version tagged to that specific release.
 
## Roadmap

- Get additional field types working
- improve a11y flags for fields that are hidden


## Credits

Brought to you by  [simplicate.ca](https://www.simplicate.ca). Written by Steve Comrie.