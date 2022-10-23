# Craft CMS matrix-conditionals.twig

A twig file that can be added to your Entry Types as a UI Element. Can be configured to with basic conditionals to show / hide fields within a matrix field blocks based on the values of other fields within the same block.

Field conditions and actions are configured within a relatively simple JS object at the top of the file.

Fields are hidden with a class that has them retain their position in the field layout but otherwise hides them.

Debug can be turned on or off. It is currently set to on for beta testing purposes

This is a beta. It *probably* won't screw up your data, but it is possible to create rules that will make your matrix blocks uneditable or fields that remain perpetually hidden if you're not careful.

Removing the UI element that references this field (or emptying the JS config object) will return your matrix to normal working order.

Rules and conditions are relatively simplistic. Most just show/hide based on a value. No greater than, less than, value contains, etc.

It's not perfect, but it's better than what we have now :)

## How to Use
 
1) Save the `matrix-conditionals.twig` file some where in your `cms/templates` directory. Maybe a folder like `_cp`.
2) Configure the JavaScript at the top of the file that describes your conditions and actions.
3) Any time you add a matrix field to an entry types' field layout, also include this file as a Template UI Element (not necessary if your matrix field doesn't contain any conditionals).

You can have multiple versions of this file if you have multiple matrix fields, but if your fields aren't overly complicated this file can be configured to handle multiple matrix fields at once.

## Configuration

Sample JavaScript config object:

    // for any `background` field in any block type in the `contentBuilder` matrix field
    'contentBuilder:*:background': {
    
        // if the value is set to "color", show the `backgroundColor` field, otherwise hide it    
        'color': {
            'showOnEqual': 'backgroundColor',
            'hideOnUnequal': 'backgroundColor'
        }
    },
    
    // for the `videoSource` field in any `video` block type in and matrix field
    '*:video:videoSource': {
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


### Identifying fields

Choose the fields whose values you want to watch using the matrix field handle, block type, and field handle:

	`matrixHandle:blockType:fieldHandle`

You may use `*` as a wildcard like this: `*:*:fieldHandle`

**Examples**

`contentBuilder:imageBlock:imagePosition`

The `imagePosition` field inside any `imageBlock` block type in the `contentBuilder` matrix

--

`*:imageBlock:imagePosition`

The `imagePosition` field inside any `imageBlock` block type in any matrix

--

`contentBuilder:*:backgroundColor`

The `backgroundColor` field inside any block type in in the `contentBuilder` matrix

--

`*:*:backgroundColor`

All `backgroundColor` fields inside any block type in any matrix




### Actions per value

Depending on how numerous and complicated your conditional rules are, you may need to list all possible values for a field, but maybe just the one you want to watch for.

There are four conditions that we can take action on: 

- showOnEqual

- hideOnEqual

- showOnUnequal

- hideOnUnequal

For each of the above conditions, we can provide:

- a single field: `'image'`

- or multiple fields: `['image','imagePosition']`
  
  
## Roadmap

- beter a11y consideration for fields that are hidden


## Credits

Brought to you by  [Simplicate](https://www.simplicate.ca)