# IMPORT A CSV DATA TABLE INTO UNREAL

# 1- Create a struct
   - Structs can store different types of variables inside one object
   - They are like classes but, unlike classes, they don't have inheritance so each variable from the same type of struct will have its own value 
  
## 1.1- Define my struct
   - Mark it as USTRUCT type to be used in Unreal
   - Blueprint type allows our Unreal project to access this struct 
   - inherit from the FTableRowBase class because it is a struct that will read a table made of rows

```cpp
USTRUCT(BlueprintType) 
struct FConfigurationDataStruct : public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

public:

	//Constructor for the struct
		//Assign default values to each variable inside the ()
		//ConfigurationDataStruct is the DataTable row type I'm trying to import
	FConfigurationDataStruct() : TeddyBearMoveAmountPerSecond(100) {}

	//Expose each column of the table to the blueprint using UPROPERTY (except for "Name" which is the default meta data column)
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Configuration Data Struct")
	float TeddyBearMoveAmountPerSecond;
};
```

# 2- Import the CSV file
   - Create a new folder in Unreal to store the file
   - Right click import and select the file
   - Select import as datatable and select ConfigurationDataStruct as the row type

# 3- Create an actor to give access to the csv datatable 
  - Add a new cpp class type actor. Inside this class:
  - Declare a FConfigurationDataStruct pointer to store the rows of the file
  - 
