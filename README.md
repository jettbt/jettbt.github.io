<h2 class="code-line" data-line-start=0 data-line-end=1 ><a id="GridBased_Battle_System_Game_0"></a>Grid-Based Battle System Game</h2>
<p class="has-line-data" data-line-start="2" data-line-end="3">I started and completed this project in the summer of 2022 as I wanted to see if I could create a functioning grid based battle system. I did this and went a bit further, creating custom art for the game, characters, UI, and backgrounds. This demo is complete with difficult enemy AI, many battle phases, and little flourishes like tiny health bars popping up when units take damage, area of effect moves that span unique grid shapes, status bar to show unit information on highlight, and multiple different units that level up and learn different moves at certain levels.</p>
<h5 class="code-line" data-line-start=4 data-line-end=5 ><a id="Ingame_Screenshots_4"></a>In-game Screenshots</h5>
<p class="has-line-data" data-line-start="5" data-line-end="15"><img src="https://user-images.githubusercontent.com/110650172/196539815-b455a380-2068-4709-b1c4-5216675b737d.png" alt="Grid1"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539846-b64978d9-0fb2-4b4a-ad86-a6be12f68baf.png" alt="Grid2"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539854-9584c49e-a5f9-46e8-924b-8489d3e8ec42.png" alt="Grid3"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539860-172149d8-27e2-49d5-ac6a-5ca773105f40.png" alt="Grid4"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539872-98c64916-f957-42eb-a93b-b24911f0c97b.png" alt="Grid5"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539887-a1501ad7-51d1-47e4-afe2-2ffe09af2a3d.png" alt="Grid6"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539901-43d7b451-77a8-4119-862d-63b18504bab2.png" alt="Grid7"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539909-937c229c-e8ad-413c-adb7-3519e558abc5.png" alt="Grid8"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539918-e561746d-77d3-4eb8-93f1-f7cd53a4ab59.png" alt="Grid9"><br>
<img src="https://user-images.githubusercontent.com/110650172/196539930-83cce34e-7fff-40f4-9fc8-c6b0312d8207.png" alt="grid10"></p>
<h5 class="code-line" data-line-start=16 data-line-end=17 ><a id="Code_snippits_16"></a>Code snippits</h5>
<pre><code class="has-line-data" data-line-start="18" data-line-end="40">public void DrawGrid()
    {
        //fills gridspace array with gridspace instances, attaches their grid number to them and sets their position, if
        //there is a collider layer gameobject on its position, it will set the grid instance to be unwalkable, it then draws
        //the space, creating a gameobject to serve as the image for the gridspace instance, also names it based on its
        //gridnumber
        for(int i = 0; i &lt; width; i++)
        {
            for(int j = 0; j &lt; height; j++)
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
</code></pre>
<p class="has-line-data" data-line-start="40" data-line-end="41">I use this function in conjunction with a few others to draw a grid based on certain parameters set in the grid class when initially constructed like “width” and “height”. The grid system is really a list of GridSpace instances. Here is some insight into the GridSpace class:</p>
<pre><code class="has-line-data" data-line-start="43" data-line-end="70">public GridSpace(Vector3 position, Vector2 gridNumber, bool walkable = true)
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
</code></pre>
<p class="has-line-data" data-line-start="70" data-line-end="71">These are just two of the many functions in the GridSpace class alone, but they give an idea of what I decided to keep track of in each GridSpace instance. When initialized, each is given a position, gridNumber, and a bool to determine whether it is walkable. This “walkable” bool is important when using my floodfill algorithm that determines where a unit can move to!</p>
<pre><code class="has-line-data" data-line-start="74" data-line-end="146"> private void RunEnemyTurn(GridBattleUnit gridBattleUnit)
    {
        //list of potential turns for the AI to choose from\
        List&lt;PotentialTurn&gt; potentialTurns = new List&lt;PotentialTurn&gt;();

        //find list of moves that unit has mp to use
        List&lt;Move&gt; MovesUsableMp = new List&lt;Move&gt;();
        foreach(Move move in gridBattleUnit.Monster.Moves)
        {
            if(CheckIfManaToUseMove(gridBattleUnit, move))
            {
                MovesUsableMp.Add(move);
            }
        }


        HideCursor();
        state = GridBattleState.EnemyTurn;

        List&lt;GridSpace&gt; potentialMoves = new List&lt;GridSpace&gt;();
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
                    List&lt;GridSpace&gt; potentialMoveTargetSpaces = new List&lt;GridSpace&gt;();
                    potentialMoveTargetSpaces = grid.ReturnFlood(gridSpace.gridNumber,move.Base.Range,move.Base.WhatToFlood);
                    foreach(GridSpace gridSpace1 in potentialMoveTargetSpaces)
                    {
                        int value = SimulateMove(gridBattleUnit,gridSpace1.gridNumber,move);
                        if(value &gt; 0)
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

        potentialTurns = potentialTurns.OrderByDescending(x =&gt; x.value).ToList&lt;PotentialTurn&gt;();
        if(potentialTurns.Count == 0)
        {
            StartNextTurn();
            return;
        }

        List&lt;PotentialTurn&gt; bestTurns = potentialTurns.Where(x =&gt; x.value &gt;= potentialTurns[0].value).ToList&lt;PotentialTurn&gt;();

        int a = UnityEngine.Random.Range(0,bestTurns.Count);
        PotentialTurn turnToRun = bestTurns[a];
        
        gridBattleUnit.moveGameObject(turnToRun.moveToGridNumber);
        
        gridBattleUnit.FaceNumber(turnToRun.moveUseGridNumber);
        

        StartCoroutine(UseMove(turnToRun.moveToUse,GetFoesInMoveUse(gridBattleUnit,turnToRun.moveUseGridNumber,turnToRun.moveToUse)));
    }
</code></pre>
<p class="has-line-data" data-line-start="146" data-line-end="147">This is perhaps the system I am most proud of! This handles the enemy AI. The idea is that I process every possible turn that the enemy can make, and store each choice within that turn into a PotentialTurn class. I then call an algorithm to evaluate that turn and give it a value, currently based on how much damage is dealt to ally units. I love this system though because if I decided to fully flesh out this game I can easily modify the value determining algorithm to take account of other types of moves or the gamestate itself, for example valuing healing an enemy (ally from the enemies perspective) over attacking player units when an enemy units health drops below a certain threshold! Man, I love coding :).</p>
<p class="has-line-data" data-line-start="148" data-line-end="149">–More Information On the Way–</p>
<h2 class="code-line" data-line-start=150 data-line-end=151 ><a id="Pokemon_FireRed_ReverseEngineering_Project_150"></a>Pokemon FireRed Reverse-Engineering Project</h2>
<p class="has-line-data" data-line-start="152" data-line-end="153">I decided to see if I could reverse-engineer Pokemon FireRed edition on Unity! I accomplished most of the funcitonality of the battle system, overworld movement and interaction, and even added some features not present in the original game like your first party slot pokemon following you!</p>
<h5 class="code-line" data-line-start=154 data-line-end=155 ><a id="Ingame_Screenshots_154"></a>In-game Screenshots</h5>
<p class="has-line-data" data-line-start="156" data-line-end="165"><img src="https://user-images.githubusercontent.com/110650172/196537182-ae969771-6067-4fbb-8254-79dbaa6a6fb6.png" alt="FireRed Image1"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537194-dc4cf5fe-6632-4c00-b6ac-3f9cdb069ae8.png" alt="FireRed Image2"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537207-f2ef4a7b-a241-45f7-9748-1c940313a6ad.png" alt="FireRed Image3"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537236-82717e7f-7c30-4bac-bafc-2e4fdb029a35.png" alt="FireRed Image4"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537252-c80f0ebe-8a69-427c-80bb-8c926159ec81.png" alt="FireRed Image5"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537260-f57f7e30-e0ea-44ec-9c83-641b61051920.png" alt="FireRed Image6"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537278-e8c9c07e-a47f-47ea-9167-2a4ad70922e0.png" alt="FireRed Image7"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537284-faeee496-2500-4dcf-83c6-dfc08fbfa201.png" alt="FireRed Image8"><br>
<img src="https://user-images.githubusercontent.com/110650172/196537300-15935e2c-f517-450d-9ca1-369232879f33.png" alt="FireRed Image9"></p>
<p class="has-line-data" data-line-start="167" data-line-end="168">–More Information On the Way–</p>
