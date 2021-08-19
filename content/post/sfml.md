---
title: "Writing a simple cpp game with Sfml"
date: 2021-05-27T00:05:06-04:00
author: "ozan ocak"
tags: [
    "cpp",
    "sfml",
]
---
<h3>Setting up environment on mac</h3>

We going to build simple snake game by using sfml library on mac os environment. I prefer creating projects with Makefile rather than using build-in tools so We will not use Xcode.First We need to download  sfml library
```bash
sudo brew install libsfml-dev
```
then We will add easy c++ project extension in VSCode and after pressing Ctrl+Shift+P, search easy cpp create new project and finally We will create with clang++. Before executing makefile We need add dependencies in libraries in makefile.
```Makefile
LIBRARIES	:= -lsfml-graphics -lsfml-window -lsfml-system
```
In the SFML [page](https://www.sfml-dev.org/tutorials/2.5/start-linux.php ), We will borrow the main.cpp sample. As it works, We can start devoloping real project.Every executable starts from main function since kernel is interrupted for looking for another main(). return 0 indicates there is no error in the executable.

We need draw a window first. In game.cpp ,we create Game constructor,destructor and Run methods from game.hpp. We define m_context pointer
```cpp
std::shared_ptr<Context> m_context;
```



```cpp (StateMan.hpp)
#pragma once   
#include <SFML/System/Time.hpp>

namespace Engine
{
    class State
    {
    public:
        State(){};      // constructor
        virtual ~State(){};     //destructor

        virtual void Init() = 0;   //pure function or interface
        virtual void ProcessInput() = 0;
        virtual void Update(sf::Time deltaTime) = 0;
        virtual void Draw() = 0;

        virtual void Pause(){};
        virtual void Start(){};
    };
    
} 
```
First of all pragma once is a preprocessor letting compiler know the headers are supposed to be included no more than once. Virtual class can have implementation and it derived classes can override it but if virtual class is pure function via =0, it doesnt have have any defination and must be written defination by derived classes(it is interface). 

```cpp
#pragma once
#include <stack>
#include <memory>
#include <State.hpp>

namespace Engine
{
 class StateMan
 {
  private:
    std::stack<std::unique_ptr<State>> m_stateStack; // m_stateStack[][]
    std::unique_ptr<State> m_newState;

    bool m_add;
    bool m_replace;
    bool m_remove;

  public:
    StateMan();
    ~StateMan();

    void Add(std::unique_ptr<State> toAdd, bool replace = false);
    void PopCurrent();
    void ProcessStateChange();
    std::unique_ptr<State>& GetCurrent();
 };

} 
```


```cpp
#include "Game.hpp"

int main()
{
    Game game;
    game.Run();
    return 0;
}
```

Game object created and Run() method called. 

```cpp
#pragma once
#include <memory>
#include <SFML/Graphics/RenderWindow.hpp>
#include "AssetMan.hpp"
#include "StateMan.hpp"

enum AssetID
{
    MAIN_FONT = 0,GRASS,FOOD,WALL,SNAKE
};

struct Context
{
    std::unique_ptr<Engine::AssetMan> m_assets;
    std::unique_ptr<Engine::StateMan> m_states;
    std::unique_ptr<sf::RenderWindow> m_window;

    Context()
    {
        m_assets = std::make_unique<Engine::AssetMan>();
        m_states = std::make_unique<Engine::StateMan>();
        m_window = std::make_unique<sf::RenderWindow>();
    }
};

class Game
{
private:
    std::shared_ptr<Context> m_context;
    const sf::Time TIME_PER_FRAME = sf::seconds(1.f/60.f);

public:
    Game();
    ~Game();
    void Run();
};
```

 Memory is standart cpp library in order to manage memory, we will use unique_ptr(which is a smart pointer that indicates only single refernce can point to pointers, smart pointers doesnt need to be cared external memory management, once the program counter out of scope, the array will be deleted).
We create 3 array with unique_ptr of std library, their types are written in "< >".we need to add



