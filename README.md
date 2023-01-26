# IMPORT A CSV DATA TABLE INTO UNREAL

# 1- Create a struct
   - In Unreal, create a new C++ file MyDataStruct
   ```
   - Structs can store different types of variables inside one object
   - They are like classes but, unlike classes, they don't have inheritance so each variable from the same type of struct will have its own value 
  ```
  
## 1.1- Define my struct
   - #include "Engine/DataTable.h" and "MyDataStruct.generated.h"
   - Mark it as USTRUCT type to be used in Unreal, assign it as BlueprintType and inherit from FTableRowBase
   - Expose each column of the table to the blueprint using UPROPERTY
   ```
   - Blueprint type allows our Unreal project to access this struct 
   - inherit from the FTableRowBase class because it is a struct that will read a table made of rows
   ```

```cpp
USTRUCT(BlueprintType) 
struct FMyDataStruct: public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

public:

	//Constructor for the struct
		//Assign default values to each variable inside the ()
		//MyDataStructis the DataTable row type I'm trying to import
	FMyDataStruct() : MyMovement(100) {}

	//Expose each column of the table to the blueprint using UPROPERTY (except for "Name" which is the default meta data column)
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "My Data Struct")
	float MyMovement;
};
```

# 2- Import the CSV file
   - Create a new folder in Unreal to store the file
   - Right click import and select the file
   - Select import as datatable and select MyDataStructas the row type

# 3- Create a data actor
  - Add a new cpp class type actor to be our data actor so that the other actors can consume data directly from this data actor.
  - Header file:
    - Declare a FMyDataStructpointer to store each row of the file
    - Declare a UDataTable pointer to store the entire data table file we imported and expose it to the blueprint with UPROPERTY
    - Add a Getter to get the metrics in the table
```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

UCLASS()
class DATATABLE_API AMyActor : public AActor
{
	GENERATED_BODY()

private:
	//Declare a FMyDataStructpointer to store the rows of the file
	FMyDataStruct* ConfigurationDataRow; 

	
public:	
	// Sets default values for this actor's properties
	AMyActor();

	//Allows to use the blueprint editor to populate this UDataTable variable with values from the data table I imported 
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "My Data Table")
	UDataTable* ConfigurationDataTable; 

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	float GetMyMovement();

};
```

  - Implementation cpp file
    - Set tick to false
    - In BeginPlay() get the data from each row the data table and store in my UDataTable variable
      - Use my UDataTable variable to call FindRow() function passing in the type of row I'm trying to find and store the value into my FMyDataStructrow variable
    - Implement the Getter for that metric
      - return the row variable accessing the column variable
```cpp
#include "MyActor.h"

// Sets default values
AMyActor::AMyActor()
{
 	// Set tick to false
	PrimaryActorTick.bCanEverTick = false;

}

// Called when the game starts or when spawned
void AMyActor::BeginPlay()
{
	Super::BeginPlay();


	FString ContextString;
	//Row variable = File variable->FindRow<Type of data in the row>("name of that row", context string)
	ConfigurationDataRow = ConfigurationDataTable->FindRow<FMyDataStruct>("ConfigData", ContextString);

	
}

// Called every frame
void AMyActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

float AMyActor::GetMyMovement()
{
	return ConfigurationDataRow->MyMovement; 
}
```
   - Inside Unreal
     - Create a blueprint based on this data actor, BP_Configuration data actor
     - Inside this BP go to my table UPROPERTY and select my ConfigurationData. This give my data actor access to my DataTable with data imported from the CSV file
     - Add this actor to the scene

# Make other actors in our world consume data in the data actor

# 1- Tag the data actor
  - This allows the other actors to find the data actor by its tag
  - Edit > Project settings > game play tag > add new gameplay tag > add new tag "MyActor"
  - Open BP_MyActor and on the tag field, add a new element and name it with the same name of your tag

# 2- Create an actor to consume the data
  - Create a new c++ class type pawn "MyActor"
  
  - Header file
    - Declare a pointer of MyActor type, an FVector for CurrentActorLocation and another for the NewActorLocation
    - Define a GetConfigurationData() function
    - Declare a MoveActor() function 
```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "MyActor.h"
#include "MyActor.generated.h"

UCLASS()
class DATATABLE_API AMyActor : public APawn
{
	GENERATED_BODY()

public:
	// Sets default values for this pawn's properties
	AMyActor();
	
	void GetConfigurationData();
	void MoveActor(AMyActor* ConfigurationData, FVector CurrentActorLocation);

private:

	AMyActor* MyActor; 
	FVector CurrentActorLocation; 
	FVector NewActorLocation;

};

```
  - Implementation file
    - Inside GetConfigurationData()
      - Iterate through all the actors with that tag and save them in a TArray of Actor pointers and save the configuration data in the configuration data pointer variable
      - If the number of actors with this tag is greater than zero, get the first actor in the array, cast it to a AMyActor pointer type and save it in my configuration data pointer
    - Inside MoveActor()
      - Get the current location for this actor
      - Declare a FVector to store this actor's new location
      - Use the ConfigurationData pointer variable to call the Get function that fetches the value of the metric you want to use and pass it to a float variable Move
      - Assign the new location for vector Y as the current location plus the Move float
      - Set the new actor location
    - Get CurrentActorLocation
    - Call GetConfigurationData() on beginplay
    - Call MoveActor() on beginplay passing the ConfigurationData pointer and CurrentActorLocation
```cpp
void AMyActor::BeginPlay()
{
	Super::BeginPlay();

	GetConfigurationData();
	
	CurrentActorLocation = GetActorLocation();

	MoveActor(ConfigurationData, CurrentActorLocation);
	
}

void AMyActor::GetConfigurationData()
{
	TArray<AActor*> MyActors;

	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "MyActor", MyActors);

	if (MyActors.Num() > 0)
	{
		ConfigurationData = (AMyActor*)MyActors[0]; 
	}
}

void AMyActor::MoveActor(AMyActor* ConfigurationData)
{
	try
	{
		float Move = ConfigurationData->GetMyMovement(); 

		if (Move < 1)
		{
			throw Move;
		}
		
		NewActorLocation.Y = CurrentActorLocation.Y + Move;
	}
	catch(float Move)
	{
		UE_LOG(LogTemp, Warning, TEXT("Move can't be smaller than 1. Move is %f"), Move);
	}

	SetActorLocation(NewActorLocation);
}
```

  - Create a Blueprint based on this class and place it into the world
    - Add a static mesh to this blueprint

# Save Game

- In Unreal Engine add a save game class MySaveGame

- In its header file 
  - declare an float variable to store the position from the data table to be passed to the Y component of the actor's position vector and expose it with UPROPERTY
```cpp
UCLASS()
class DATATABLE_API UMySaveGame : public USaveGame
{
	GENERATED_BODY()
	
public:

	UPROPERTY(VisibleAnywhere, Category = "Saved Y")
	float SavedY = 0.0f;
};
```

- in MyActor header file
  - Declare a LoadGame() and SaveGame() functions
  - Declare a UMySaveGame pointer variable
```cpp
public:
	void LoadGame();
	void SaveGame();

private:
	UMySaveGame* MySaveGame; 
```

- In MyActor Implementation file
  - Inside LoadGame() 
    - try to load a previously saved game. 
    - If there is none, initialize the current actor location and create a new empty save game object
    - If there was already a saved game, load the SavedY location and pass it into the CurrentActorLocation
```cpp
void AMyActor::LoadGame()
{
	//Try to load a previously saved game
	MySaveGame = Cast<UMySaveGame>(UGameplayStatics::LoadGameFromSlot("MySaveSlot", 0));

	//If there is no previously saved game
	if (MySaveGame == nullptr)
	{
		//Initialize current actor location
		CurrentActorLocation = GetActorLocation();

		//Create an empty save game object to be filled later
		MySaveGame = Cast<UMySaveGame>(UGameplayStatics::CreateSaveGameObject(UMySaveGame::StaticClass()));
	}
	else //If there is already a saved game
	{
		//Define location Y = to that we had saved previously
		CurrentActorLocation.Y = MySaveGame->SavedY;
	}
	
}
```

  - Inside SaveGame()
    - Pass the NewActorLocation into the SavedY variable in the MySaveGame object
    - Save game to slot

```cpp
void AMyActor::SaveGame()
{
	MySaveGame->SavedY = NewActorLocation.Y; 

	UGameplayStatics::SaveGameToSlot(MySaveGame, "MySaveSlot", 0);
}
```

  - In BeginPlay()
    - Replace the previous CurrentActorLocation initialization by LoadGame()
    - Call SaveGame() after MoveActor()
```cpp
void AMyActor::BeginPlay()
{
	Super::BeginPlay();

	GetConfigurationData();

	LoadGame();

	MoveActor(ConfigurationData, CurrentActorLocation);

	SaveGame();
}
```
