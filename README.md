# Jett Badalament-Tirrell's Dev Blog

Welcome to my page! Here you can find information about me and my projects. If you have any questions, concerns, or interest regarding my projects, please feel free to email me at hwq8mr@virginia.edu !

## Grid-Based Battle System Game

I started and completed this project in the summer of 2022 as I wanted to see if I could create a functioning grid based battle system. I did this and went a bit further, creating custom art for the game, characters, UI, and backgrounds. This demo is complete with difficult enemy AI, many battle phases, and little flourishes like tiny health bars popping up when units take damage, area of effect moves that span unique grid shapes, status bar to show unit information on highlight, and multiple different units that level up and learn different moves at certain levels.

##### In-game Screenshots
![Grid1](https://user-images.githubusercontent.com/110650172/196539815-b455a380-2068-4709-b1c4-5216675b737d.png)
![Grid2](https://user-images.githubusercontent.com/110650172/196539846-b64978d9-0fb2-4b4a-ad86-a6be12f68baf.png)
![Grid3](https://user-images.githubusercontent.com/110650172/196539854-9584c49e-a5f9-46e8-924b-8489d3e8ec42.png)
![Grid4](https://user-images.githubusercontent.com/110650172/196539860-172149d8-27e2-49d5-ac6a-5ca773105f40.png)
![Grid5](https://user-images.githubusercontent.com/110650172/196539872-98c64916-f957-42eb-a93b-b24911f0c97b.png)
![Grid6](https://user-images.githubusercontent.com/110650172/196539887-a1501ad7-51d1-47e4-afe2-2ffe09af2a3d.png)
![Grid7](https://user-images.githubusercontent.com/110650172/196539901-43d7b451-77a8-4119-862d-63b18504bab2.png)
![Grid8](https://user-images.githubusercontent.com/110650172/196539909-937c229c-e8ad-413c-adb7-3519e558abc5.png)
![Grid9](https://user-images.githubusercontent.com/110650172/196539918-e561746d-77d3-4eb8-93f1-f7cd53a4ab59.png)
![grid10](https://user-images.githubusercontent.com/110650172/196539930-83cce34e-7fff-40f4-9fc8-c6b0312d8207.png)

##### Code snippits
```
public void DrawGrid()
    {
        //fills gridspace array with gridspace instances, attaches their grid number to them and sets their position, if
        //there is a collider layer gameobject on its position, it will set the grid instance to be unwalkable, it then draws
        //the space, creating a gameobject to serve as the image for the gridspace instance, also names it based on its
        //gridnumber
        for(int i = 0; i < width; i++)
        {
            for(int j = 0; j < height; j++)
            {
                Vector2 gridNumber = new Vector2(i,j);
                gridArray[i,j] = new GridSpace(GridNumberToVector3(gridNumber), gridNumber);
                if(Physics2D.OverlapCircle(gridArray[i,j].position, 0.03f,GameLayers.i.SolidLayer) != null)
                {
                    gridArray[i,j].walkable = false;
                }
                DrawSpace(gridArray[i,j]);
                gridArray[i,j].Initialize();
            }
        }
    }
```
I use this function in conjunction with a few others to draw a grid based on certain parameters set in the grid class when initially constructed like "width" and "height". The grid system is really a list of GridSpace instances. Here is some insight into the GridSpace class:

```
public GridSpace(Vector3 position, Vector2 gridNumber, bool walkable = true)
    {
        this.position = position;
        this.gridNumber = gridNumber;
        this.walkable = walkable;
    }

    public bool Floodable(FloodType whatToFlood = FloodType.Walkable)
    {
        switch(whatToFlood)
        {
            case FloodType.Walkable:
                if(battleUnit != null) return false;
                if(!walkable) return false;
                else return true;
            
            case FloodType.UnitsNEmpty:
                if(!walkable) return false;
                return true;
            


        }
        return true;
        
    }
```
These are just two of the many functions in the GridSpace class alone, but they give an idea of what I decided to keep track of in each GridSpace instance. When initialized, each is given a position, gridNumber, and a bool to determine whether it is walkable. This "walkable" bool is important when using my floodfill algorithm that determines where a unit can move to!


```
 private void RunEnemyTurn(GridBattleUnit gridBattleUnit)
    {
        //list of potential turns for the AI to choose from\
        List<PotentialTurn> potentialTurns = new List<PotentialTurn>();

        //find list of moves that unit has mp to use
        List<Move> MovesUsableMp = new List<Move>();
        foreach(Move move in gridBattleUnit.Monster.Moves)
        {
            if(CheckIfManaToUseMove(gridBattleUnit, move))
            {
                MovesUsableMp.Add(move);
            }
        }


        HideCursor();
        state = GridBattleState.EnemyTurn;

        List<GridSpace> potentialMoves = new List<GridSpace>();
        potentialMoves = grid.ReturnFlood(gridBattleUnit.GridNumber,gridBattleUnit.moveSpaces,FloodType.Walkable);

        //decide which movetarget to use
        MoveTarget moveTarget = MoveTarget.Foe;

        foreach(GridSpace gridSpace in potentialMoves)
        {
            foreach(Move move in MovesUsableMp)
            {
                if(move.Base.Target == moveTarget)
                {
                    //create list of all areas move can be used,, floodtype is ally because we want to target allies as an enemy
                    List<GridSpace> potentialMoveTargetSpaces = new List<GridSpace>();
                    potentialMoveTargetSpaces = grid.ReturnFlood(gridSpace.gridNumber,move.Base.Range,move.Base.WhatToFlood);
                    foreach(GridSpace gridSpace1 in potentialMoveTargetSpaces)
                    {
                        int value = SimulateMove(gridBattleUnit,gridSpace1.gridNumber,move);
                        if(value > 0)
                        {
                            //store space moved to, attack used, and value in potential turn list
                            PotentialTurn newPotentialTurn = new PotentialTurn();
                            newPotentialTurn.moveToGridNumber = gridSpace.gridNumber;
                            newPotentialTurn.moveToUse = move;
                            newPotentialTurn.value = value;
                            newPotentialTurn.moveUseGridNumber = gridSpace1.gridNumber;
                            potentialTurns.Add(newPotentialTurn);
                        }                       
                    }
                }
            }
        }

        potentialTurns = potentialTurns.OrderByDescending(x => x.value).ToList<PotentialTurn>();
        if(potentialTurns.Count == 0)
        {
            StartNextTurn();
            return;
        }

        List<PotentialTurn> bestTurns = potentialTurns.Where(x => x.value >= potentialTurns[0].value).ToList<PotentialTurn>();

        int a = UnityEngine.Random.Range(0,bestTurns.Count);
        PotentialTurn turnToRun = bestTurns[a];
        
        gridBattleUnit.moveGameObject(turnToRun.moveToGridNumber);
        
        gridBattleUnit.FaceNumber(turnToRun.moveUseGridNumber);
        

        StartCoroutine(UseMove(turnToRun.moveToUse,GetFoesInMoveUse(gridBattleUnit,turnToRun.moveUseGridNumber,turnToRun.moveToUse)));
    }
```
This is perhaps the system I am most proud of! This handles the enemy AI. The idea is that I process every possible turn that the enemy can make, and store each choice within that turn into a PotentialTurn class. I then call an algorithm to evaluate that turn and give it a value, currently based on how much damage is dealt to ally units. I love this system though because if I decided to fully flesh out this game I can easily modify the value determining algorithm to take account of other types of moves or the gamestate itself, for example valuing healing an enemy (ally from the enemies perspective) over attacking player units when an enemy units health drops below a certain threshold! Man, I love coding :).

--More Information On the Way--

## Pokemon FireRed Reverse-Engineering Project

I decided to see if I could reverse-engineer Pokemon FireRed edition on Unity! I accomplished most of the funcitonality of the battle system, overworld movement and interaction, and even added some features not present in the original game like your first party slot pokemon following you!

##### In-game Screenshots

![FireRed Image1](https://user-images.githubusercontent.com/110650172/196537182-ae969771-6067-4fbb-8254-79dbaa6a6fb6.png)
![FireRed Image2](https://user-images.githubusercontent.com/110650172/196537194-dc4cf5fe-6632-4c00-b6ac-3f9cdb069ae8.png)
![FireRed Image3](https://user-images.githubusercontent.com/110650172/196537207-f2ef4a7b-a241-45f7-9748-1c940313a6ad.png)
![FireRed Image4](https://user-images.githubusercontent.com/110650172/196537236-82717e7f-7c30-4bac-bafc-2e4fdb029a35.png)
![FireRed Image5](https://user-images.githubusercontent.com/110650172/196537252-c80f0ebe-8a69-427c-80bb-8c926159ec81.png)
![FireRed Image6](https://user-images.githubusercontent.com/110650172/196537260-f57f7e30-e0ea-44ec-9c83-641b61051920.png)
![FireRed Image7](https://user-images.githubusercontent.com/110650172/196537278-e8c9c07e-a47f-47ea-9167-2a4ad70922e0.png)
![FireRed Image8](https://user-images.githubusercontent.com/110650172/196537284-faeee496-2500-4dcf-83c6-dfc08fbfa201.png)
![FireRed Image9](https://user-images.githubusercontent.com/110650172/196537300-15935e2c-f517-450d-9ca1-369232879f33.png)


--More Information On the Way--
