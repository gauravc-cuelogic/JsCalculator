Validation and Error framework
=======
This document describes validation and error framework used for "Entitlement Applicability" service.

We will be following the [WDPRO Error](https://wiki.wdpro.wdig.com/display/Tech/Content+Type+Registry+-+WDPRO+Error) data structure for rendering errors in Rest response body. The implementation for this Error model is already done in [WDPR errors](https://github.disney.com/wdpro-development/wdpr-svc-core-java7/tree/master/wdpr-errors/src/main/java/com/disney/wdpr/platform/errors) library and we are leveraging this and few other common wdpr libraries for our error handling.

ErrorableObject is container class that holds list of Error objects. [Error](https://github.disney.com/wdpro-development/wdpr-svc-core-java7/blob/master/wdpr-errors/src/main/java/com/disney/wdpr/platform/errors/Error.java) is the main interface for all errors, it's most commonly used implementation is UniversalError. ValidationError interface extends Error and it's implementation class is UniversalValidationError.



## Rest Layer Validation
Basic validation of required path parameters and fields in request body is done in the Rest layer. This includes checking whether the field values are in expected format. 

Here is an example of validation in Rest layer
```
        ErrorableObject errorableObject = new ErrorableObject();
        
        // Validate whether destination was provided
        if (StringUtils.isBlank(destination)) {
    
            UniversalValidationError error = ValidationErrorsUtil.buildValidationError("Destination is required", PARAM_DESTINATION, destination, ErrorsUtil.VALIDATION_TYPE_MISSING_REQUIRED_FIELD, Optional.empty());
            errorableObject.addError(error);
        }
        // Some more validations
        // Add other validation errors to ErrorableObject
        
       if(errorableObject.hasErrors()){
			
            return Response.status(HttpStatus.SC_BAD_REQUEST).entity(errorableObject).build();
       }
```

Here is an example of how to build map of additional properties and generate validation error for invalid format of a input field
```
      Map<String, Object> validationProperties = ValidationPropertyMapBuilder.init()
                .addValidationProperty(ValidationProperty.EXPECTED_FORMAT, "yyyy-MM-dd")                
                .build();

        UniversalValidationError error= ValidationErrorsUtil.buildValidationError("Invalid date format", "startDate", rejectedValue, ValidationErrorTypeEnum.INVALID_FORMAT.name(), Optional.of(validationProperties));
```

 
### LocalDate, LocalTime and LocalDateTime path / query parameters
Handling of validation of path / query parameters that are of the following types, is handled by deserializers that are registered with FasterJacksonMapper. In such cases, you don't have to write any code in the Rest layer, it is handled for you by the deserializers.
- LocalDate
- LocalTime
- LocalDateTime


## Validation at the Business service layer
Any business rule related validation is performed at the Business Service layer.

```
        // Validate Destination with the data in DB Destination table
        String destination = searchCriteria.getDestination();
        boolean isValidDestination = destinationDAO.getAllDestinationIds().contains(destination);

        if (!isValidDestination) {

            String errorMessage = String.format("Destination %s is not supported", destination);
            throw ValidationErrorsUtil.buildValidationException(errorMessage, "destination", destination, ValidationErrorTypeEnum.INVALID_VALUES.name(), Optional.empty(), Optional.empty());

        }

```

If you want to accumulate errors and send multiple errors in the Rest response , use MultiErrorException ( which holds a collection of errors) . 

#### NOTE : In case of multiple errors, the Http status code of the response will be based on the first error in the collection.

Here is an example of how to collect multiple errors and throw MultiErrorException

```
Set<Error> errors = Sets.newHashSet();

if(<some condition>){
  errors.add( ErrorType.MOCK_DINING_PLAN_CONFLICT_ERROR.buildError());
}

if(<sonme other condition>){
  errors.add( ErrorType.CONTENT_TYPE_NOT_SUPPORTED.buildError());
}

if(!errors.isEmpty()){

	throw MultiErrorException.of(errors);
}
```
 
The easiest way to build an error is to use our 'ErrorType' enum, which has utility method to build an error with the data held in the enumeration value. 

If the ErrorType you need is not already there, add appropriate entries in ErrorType enumeration. 

Here is an example entry in ErrorType
```
MOCK_DINING_PLAN_CONFLICT_ERROR(HttpStatus.CONFLICT, "Received Dining Plan Conflict error from backend")

//Here is how you can create an error object using the above mentioned ErrorType value
Error error1 = ErrorType.MOCK_DINING_PLAN_CONFLICT_ERROR.buildError();


```

When the above mentioned error is added to either to an ErrorableObject or to BackendException, it will produce the following response.

```
Response Http Status 409

Response body :


{
	"errors" : [
		{
			"typeId" : "MOCK_DINING_PLAN_CONFLICT_ERROR",
			"message" : "Received Dining Plan Conflict error from backend"
		}
	]
}
```
## Error handling in Backend Services
TBD
This would depend on the technology ( e.g. Retrofit ) we decide to use to make backend Rest / http calls.




