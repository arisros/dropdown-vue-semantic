# dropdown-vue-semantic

> Vue Dropdown with Semantic

## Build Setup

``` bash
# install dependencies
npm install dropdown-vue-semantic --save-dev

``` bash
# use
import dropdown from 'dropdown-vue-semantic'
{
  components: {..., dropdown}
}

``` bash
'name', // like v model
'fluid', // fluid style
'searchAble', // enablesearch
'toLowerCase', // to lower case all options
'placeholder', // placeholder default
'fieldId', // id key
'fieldName', // name key
'defaultSelected', // default selected from data
'touch', // event
'error', // style true error
'dependOn', // chain depend on v model
'dependOnName', // idk
'endPoint', // if options direct from endpoint
'endPointChild', // still not need
'options', // options array 'id and name'
'disabled' // disabled
