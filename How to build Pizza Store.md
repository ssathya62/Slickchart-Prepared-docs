
# Building a Pizza store
## Create a project from the command line
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

## Start building the application
### Create domain objects
Create a Pizza class in Shared project that will hold its name, price and spiciness. We are going to have all properties *immutable* as our application is **not** about editing pizzas.

Next create a menu class that holds various types of pizzas and a method to retrive a pizza by its Id.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MDg3MzE5MzIsMTk5MTY5MjE0NywxOT
EyNzYzNzkyXX0=
-->