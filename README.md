# IMPORT A CSV DATA TABLE INTO UNREAL

# 1- Create a struct
   - In VS Code, right click on the source folder, click add new file
   - Create one file for the header file MyDataStruct
   ```
   - Structs can store different types of variables inside one object
   - They are like classes but, unlike classes, they don't have inheritance so each variable from the same type of struct will have its own value 
  ```
  
## 1.1- Define my struct
   - #include "Engine/DataTable.h" and "MyDataStruct.generated.h"
   - Mark it as USTRUCT type to be used in Unreal, assign it as BlueprintType and inherit from FTableRowBase
   - Expose each column of the table to the blueprint using UPROPERTY

```cpp
//Blueprint type allows our Unreal project to access this struct 
USTRUCT(BlueprintType) 
//inherit from the FTableRowBase class because it is a struct that will read a table made of rows
struct FMyDataStruct: public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

public:

	//Constructor for the struct 
		//Initialize its member variables (one for each column of the data table)
	FMyDataStruct() : MyMovement() {}

	//Expose each variable of the struct to the blueprint using UPROPERTY (except for "Name" which is the default meta data column)
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
    - Declare a FMyDataStruct pointer to store each row of the file
    - Declare a UDataTable pointer and expose it to the blueprint with UPROPERTY so that we can attach our data table file we imported to this pointer as a UDataTable type.
    - Add a Getter to get the metrics in the table
```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyDataActor.generated.h"

UCLASS()
class DATATABLE_API AMyDataActor : public AActor
{
	GENERATED_BODY()

private:
	//Declare a FMyDataStruct pointer to store the rows of the file
	FMyDataStruct* MyDataRow; 

	
public:	
	//A constructor that sets default values for this actor's properties
	AMyDataActor();

	//Allows to use the blueprint editor to populate this UDataTable variable with values from the data table I imported 
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "My Data Table")
	UDataTable* MyDataTable; 

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
    - In BeginPlay() access my data table, find each row and store the information of each row in a row variable
      - Use my UDataTable variable to call FindRow() function passing in the type of row I'm trying to find and store the value into my MyDataRow variable
    - Implement the Getter for that metric
      - return the the specific column on that row of which data we want to store
```cpp
#include "MyDataActor.h"

// Sets default values
AMyActor::AMyDataActor()
{
 	// Set tick to false
	PrimaryActorTick.bCanEverTick = false;

}

// Called when the game starts or when spawned
void AMyDataActor::BeginPlay()
{
	Super::BeginPlay();


	FString ContextString;
	//Row variable = File variable->FindRow<the struct type representing the data in the row>("name of that row", context string)
	MyDataRow = MyDataTable->FindRow<FMyDataStruct>("Data", ContextString);

	
}

// Called every frame
void AMyDataActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

float AMyDataActor::GetMyMovement()
{
	return MyDataRow->MyMovement; 
}
```
   - Inside Unreal
     - Create a blueprint based on this data actor, BP_DataActor
     - Inside this BP go to my table UPROPERTY and select my data table. This gives my data actor access to my DataTable with data imported from the CSV file
     - Add this actor to the scene

# Make other actors in our world consume data in the data actor

# 1- Tag the data actor
  - This allows the other actors to find the data actor by its tag
  - Edit > Project settings > game play tag > add new gameplay tag > add new tag "MyDataActor"
  - Open BP_MyActor and on the tag field, add a new element and name it with the same name of your tag

# 2- Create an actor to consume the data
  - Create a new c++ class type pawn "MyActor"
  
  - Header file
    - Declare a pointer of MyActor type, an FVector for CurrentActorLocation and another for the NewActorLocation
    - Declare a GetMyData() function
    - Declare a MoveActor() function 
```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include "MyDataActor.h"
#include "MyMovementActor.generated.h" 

UCLASS()
class DATATABLE_API AMyMovementActor : public APawn
{
	GENERATED_BODY()

public:
	// Sets default values for this pawn's properties
	AMyMovementActor();
	
	void GetMyData();
	void MoveActor(AMyDataActor* MyData, FVector CurrentActorLocation);

private:

	AMyDataActor* MyData; 
	FVector CurrentActorLocation; 
	FVector NewActorLocation;

};

```
  - Implementation file
    - Inside GetMyData()
      - Iterate through all the actors with that tag and save them in a TArray of Actor pointers and save the configuration data in the configuration data pointer variable
      - If the number of actors with this tag is greater than zero, get the first actor in the array, cast it to a AMyActor pointer type and save it in my configuration data pointer
    - Inside MoveActor()
      - Get the current location for this actor
      - Declare a FVector to store this actor's new location
      - Use the MyData pointer variable to call the Get function that fetches the value of the metric you want to use and pass it to a float variable Move
      - Assign the new location for vector Y as the current location plus the Move float
      - Set the new actor location
    - Get CurrentActorLocation
    - Call GetMyData() on beginplay
    - Call MoveActor() on beginplay passing the MyData pointer and CurrentActorLocation
```cpp

#include "Engine/World.h" 
#include "Kismet/GameplayStatics.h" 
#include "MyMovementActor.h"

void AMyMovementActor::BeginPlay()
{
	Super::BeginPlay();

	GetMyData();
	
	CurrentActorLocation = GetActorLocation();

	MoveActor(MyData, CurrentActorLocation);
	
}

void AMyMovementActor::GetMyData()
{
	TArray<AActor*> MyDataActors;

	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "MyDataActor", MyDataActors);

	if (MyDataActors.Num() > 0)
	{
		MyData = (AMyDataActor*)MyDataActors[0]; 
	}
}

void AMyMovementActor::MoveActor(AMyDataActor* MyData, FVector CurrentActorLocation)
{
	try
	{
		float Move = MyData->GetMyMovement(); 

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

#include "CoreMinimal.h"
#include "GameFramework/SaveGame.h"
#include "MySaveGame.generated.h"


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

#include "MySaveGame.h"

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

	GetMyData();

	LoadGame();

	MoveActor(MyData, CurrentActorLocation);

	SaveGame();
}
```
