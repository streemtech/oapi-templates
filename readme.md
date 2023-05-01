# oapi-templates

This is the list of templates used by streemtech to generate our API code.
This loads data in using the OAPI codegen project, but also loads in several other helpfull things for primary development. 

## gorm
To support our gorm based database system, a series of extra generation templates have been added to the default generation. Those included are the following

### .schemas.{Name}.x-oapi-gorm
This contains all specalty config that applies globally to the functions.
#### updateIgnore
This is a list of keys that should be ignored in the update calls, and thus will not be able to be updated.
#### validation
This is the data used in the validation step to validate the inputs.
##### tag
This is the unique tag used by the validator to check against the input. (This is so that the default validation tag is not misused in these calls.)
##### (create|get|update|delete)Keys
This is an array of strings that represents the struct keys to validate