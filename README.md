# Jett Badalament-Tirrell's Dev Blog

Welcome to my page! Here you can find information about me and my projects. If you have any questions, concerns, or interest regarding my projects, please feel free to email me at hwq8mr@virginia.edu !

## Grid-Based Battle System Game

![BattleHighlight on Character](https://user-images.githubusercontent.com/110650172/196505657-2fa609be-8320-4379-84d6-1188b2ccea79.png)

I started and completed this project in the summer of 2022 as I wanted to see if I could create a functioning grid based battle system. I did this and went a bit further, creating custom art for the game, for characters, UI, and backgrounds. This demo is complete with difficult enemy AI, many battle phases, and little flourishes like tiny health bars popping up when units take damage, area of effect moves that span unique grid shapes, status bar to show unit information on highlight, and multiple different units that level up and learn different moves at certain levels. I am very proud of this project, and I will explain some of the C# coding that went into this in the sections below. First, I would like to show some more screenshots to showcase the system and art.

![showing off movement of character to highlight](https://user-images.githubusercontent.com/110650172/196507329-eeb469ec-a19e-4459-9626-d0e0646e508b.png)
![showing off movement of character to highlight2](https://user-images.githubusercontent.com/110650172/196507341-2bd3aeb2-d3b5-4ce7-b655-7754d87584b4.png)

Here, I am showcasing that the character will look towards your cursors position. The character has four directions it can look in, so the grid is split up into sections with this method:
