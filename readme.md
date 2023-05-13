# oapi-templates

This is the list of templates used by streemtech to generate our API code.
This loads data in using the OAPI codegen project, but also loads in several other helpfull things for primary development. 

## gorm
To support our gorm based database system, a series of extra generation templates have been added to the default generation. Those included are the following


### .schemas.{Name}.x-oapi-gorm
This contains all specalty config that applies globally to the functions.

#### updateIgnore
This is an array of keys that should be ignored in the update calls, and thus will not be able to be updated.

#### validation
This is an object to validate the data used during create/update.
##### (create|update)
This is the root object used to look for validation of.
###### tag
This is the unique tag used by the validator to check against the input. (This is so that the default validation tag is not misused in these calls.)
###### Keys
This is an array of strings that represents the struct keys to validate

#### batch
This is an object that defines the method of generating a batch object. If no object is defined, batching will not be created for this object.
##### key
This is a string representing the Struct Field (Key) used to get the key for the dataloader.
##### type
This is a string used as the type for the key in the dataloader.