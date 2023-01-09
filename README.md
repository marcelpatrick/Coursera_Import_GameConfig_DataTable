# IMPORT A CSV DATA TABLE INTO UNREAL

## 1- Create a struct
   - Structs can store different types of variables inside one object
   - They are like classes but, unlike classes, they don't have inheritance so each variable from the same type of struct will have its own value 
  
  
### 1.1- Define my struct
    - Mark it as USTRUCT type to be used in Unreal
    - Blueprint type allows our Unreal project to access this struct 
    - inherit from the FTableRowBase class because it is a struct that will read a table made of rows
