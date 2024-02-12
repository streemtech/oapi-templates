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




# Batching and caching
The following is potential code to use for batch and cache and dataloader generation.
It should be possible to use some form of mutation input and output for batched endpoint calls.
This should work for the appropriate endpoints, but not function for batch gorm requests.
Those requests will likely rely on `.Where("x in ?", []foo)` queries, which this form will likely not support easily.


```go

type inputMutationFunc[K comparable, V any] func([]K) V
type requestFunc[Input any, Output any] func(context.Context, Input) (Output, error)
type outputMutationFunc[Results any, Output any] func(Results) []*dataloader.Result[Output]

func Fake[SingleRequestData comparable, InputParams any, OutputParams any, SingleResultData any](
	inMut inputMutationFunc[SingleRequestData, InputParams],
	requestFunc requestFunc[InputParams, OutputParams],
	outMut outputMutationFunc[OutputParams, SingleResultData],
	cache dataloader.Cache[SingleRequestData, SingleResultData],
) *dataloader.Loader[SingleRequestData, SingleResultData] {
	return dataloader.NewBatchedLoader(func(ctx context.Context, in []SingleRequestData) []*dataloader.Result[SingleResultData] {

		resp, err := requestFunc(ctx, inMut(in))

		//if its an error, provide it as an error
		if err != nil {
			return libraries.Map[SingleRequestData](in, func(SingleRequestData) *dataloader.Result[SingleResultData] {
				return &dataloader.Result[SingleResultData]{
					Error: err,
				}
			})
		}

		return outMut(resp)

		// return libraries.Map[SingleResultData](resultArr, func(r SingleResultData) *dataloader.Result[SingleResultData] {
		// 	return &dataloader.Result[SingleResultData]{
		// 		Data: r,
		// 	}
		// })

	}, dataloader.WithCache(cache))
}

func makeTest(c ClientWithResponsesInterface, cache dataloader.Cache[string, StreemtechAccountInformation]) *dataloader.Loader[string, StreemtechAccountInformation] {
	return Fake(
		func(srd []string) *GetUserIDsByEmailParams {
			return &GetUserIDsByEmailParams{
				Email: srd,
			}
		},
		func(ctx context.Context, r *GetUserIDsByEmailParams) (*GetUserIDsByEmailResponse, error) {
			return c.GetUserIDsByEmailWithResponse(ctx, r)
		},
		func(dat *GetUserIDsByEmailResponse) []*dataloader.Result[StreemtechAccountInformation] {
			return nil
		},
		cache,
	)

}


```