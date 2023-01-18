# IMPORT A CSV DATA TABLE INTO UNREAL

# 1- Create a struct
   - In VS Code, create a new file ConfigurationDataStruct
   ```
   - Structs can store different types of variables inside one object
   - They are like classes but, unlike classes, they don't have inheritance so each variable from the same type of struct will have its own value 
  ```
  
## 1.1- Define my struct
   - #include "Engine/DataTable.h" and "ConfigurationDataStruct.generated.h"
   - Mark it as USTRUCT type to be used in Unreal, assign it as BlueprintType and inherit from FTableRowBase
   - Expose each column of the table to the blueprint using UPROPERTY
   ```
   - Blueprint type allows our Unreal project to access this struct 
   - inherit from the FTableRowBase class because it is a struct that will read a table made of rows
   ```

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

# 3- Create a data actor through which the other actors will consume the data from our data table 
  - Add a new cpp class type actor to be our data actor so that the other actors can consume data directly from my data actor.
  - Header file:
    - Declare a FConfigurationDataStruct pointer to store each row of the file
    - Declare a UDataTable pointer to store the entire data table file we imported and expose it to the blueprint with UPROPERTY
    - Add a Getter to get the metrics in the table
```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "ConfigurationDataActor.generated.h"

UCLASS()
class DATATABLE_API AConfigurationDataActor : public AActor
{
	GENERATED_BODY()

private:
	//Declare a FConfigurationDataStruct pointer to store the rows of the file
	FConfigurationDataStruct* ConfigurationDataRow; 

	
public:	
	// Sets default values for this actor's properties
	AConfigurationDataActor();

	//Allows to use the blueprint editor to populate this UDataTable variable with values from the data table I imported 
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "My Data Table")
	UDataTable* ConfigurationDataTable; 

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	float GetTeddyBearMoveAmountPerSecond();

};
```

  - Implementation cpp file
    - Set tick to false
    - In BeginPlay() get the data from each row the data table and store in my UDataTable variable
      - Use my UDataTable variable to call FindRow() function passing in the type of row I'm trying to find and store the value into my FConfigurationDataStruct row variable
    - Implement the Getter for that metric
      - return the row variable accessing the column variable
   - Inside Unreal
     - Create a blueprint based on this data actor, BP_Configuration data actor
     - Inside this BP go to my table UPROPERTY and select my ConfigurationData. This give my data actor access to my DataTable with data imported from the CSV file
     - Add this actor to the scene
```cpp
#include "ConfigurationDataActor.h"

// Sets default values
AConfigurationDataActor::AConfigurationDataActor()
{
 	// Set tick to false
	PrimaryActorTick.bCanEverTick = false;

}

// Called when the game starts or when spawned
void AConfigurationDataActor::BeginPlay()
{
	Super::BeginPlay();


	FString ContextString;
	//Row variable = File variable->FindRow<Type of data in the row>("name of that row", context string)
	ConfigurationDataRow = ConfigurationDataTable->FindRow<FConfigurationDataStruct>("ConfigData", ContextString);

	
}

// Called every frame
void AConfigurationDataActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

float AConfigurationDataActor::GetTeddyBearMoveAmountPerSecond()
{
	return ConfigurationDataRow->TeddyBearMoveAmountPerSecond; 
}
```

