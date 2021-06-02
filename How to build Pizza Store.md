# Building a Pizza store
## Create a project from command line
First go to a folder where you want to build the application, in my case my root folder was ~/source/repos. Then the following command was issued to create the project.
```sh
 dotnet new blazorwasm --hosted -o PizzaStore
 ```
 A new folder PizzaStore was created and a new Blazor solution was created for me.
 ```
 $ cd PizzaStore/

MINGW64 ~/source/repos/PizzaStore
$ ls
Client  PizzaStore.sln  Server  Shared

MINGW64 ~/source/repos/PizzaStore
$ cd Client

MINGW64 ~/source/repos/PizzaStore/Client
$ dotnet run
```

## Clean template files
Some of the files that are created by the wizard are not necessary for the application. Hence we delete the files associated with weather forecast and counter.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzY3NjI2Mzc1LDE5MTI3NjM3OTJdfQ==
-->