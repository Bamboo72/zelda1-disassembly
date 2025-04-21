# CS 2810
## Final Project - Zelda 1 
## Jacob Schwartz

### Title and Project Track
For my final project, I chose Project Track 1: “Classic Game Restoration and Modification”. The classic game I chose is The Legend of Zelda for the Nintendo Entertainment System.

### Documentation
I worked on this project over the course of the three weeks we were given for the final project. Over the course of those three weeks, I put about 30 hours of work into this project (Note that time includes research and messing around). I recorded each time I worked in the `Project Documentation.md` file in the project. This ended up being quite long because I also recorded my goals and thought process in that file under the “Actual Timeline Log” section.
		
This project was a bit of a challenge for me due to the fact that I’d never done something like this before. I did, however, manage to make a handful of modifications to the game that I am happy with. It was very helpful to have the disassembled code and comments from Aldo Nunez, which was the owner of the [GitHub repo](https://github.com/aldonunez/zelda1-disassembly) I forked from.


### The changes I was able to make include:
1. The text in the cave where you get the wooden sword is changed.
2. The map in the overworld now has a blinking red indicator of where the next dungeon is.
3. Because I had to remove code to add my own, the pond secret doesn't work anymore. But, I added a new way to enter level 7 via a bombable wall.
4. I added a new cave 3 rooms east from Link's starting point. This cave's ID is generated randomly each time a room is loaded.
*Note, this game is playable, but there is some visual weirdness that goes on*


### How to run / view my changes
If you want to run the game, you will need a NES game emulator such as Mesen or FCEUX. I personally used Mesen when testing my changes. You can simply open the Z.nes file and play the game with my modifications.
If you want to view my code, you can do so at this GitHub repo.

### Challenges
Because this was my first time working on an NES game, I faced a good amount of challenges:
1. I had to learn and get used to the Assembly language that all NES games were written in. 6502 has the same basic concepts of most Assembly languages, but it only has three registers named A, X, and Y. There are many commands specific to each register, and I had to learn how they all work together. I am very grateful to Chibi Akuma’s [tutorial](https://www.chibiakumas.com/6502/#Lesson1) and [cheatsheet](https://www.chibiakumas.com/6502/CheatSheet.pdf) for 6502 Assembly basics.
2. The base code for The Legend of Zelda is a lot more complicated and detailed than I expected. A large challenge for me was navigating the code and finding the specific functionalities that I wanted to modify. The comments and section names from Aldo Nunez were helpful to a degree, but I still found the code difficult to navigate.
3. Understanding the structure of the game’s memory was another challenge I faced. I had to understand it for things like the cave ID system, as well as changing the layout of the rooms on the overworld. What helped me here was trial and error, as well as studying the [world map](https://pbs.twimg.com/media/DrVxoM6WkAEfLlv.jpg:large) online and the variables.inc file included in the code.
4. As I was working on this project, I felt like I was missing something. Eventually, I found what I was missing in the game’s dat files. These hold things like text strings and level info. Once I figured out where those were and how to edit them, it opened up the possibilities of making progress on the changes I wanted to make. I used a hex editor called HxD to edit the dat files. I also made a [spreadsheet](https://docs.google.com/spreadsheets/d/1MG5nnYAY3EUDUUIN2MeGmIRUrDPwgNK-1gyuo3_T_fs/edit?usp=sharing) to help me keep track of and make the calculation of the game’s room ID to level ID conversions.
5. The base game took advantage of the space available on the game cartridge. To bypass these types of errors: “ld65: Warning: src\Z.cfg:42: Segment 'BANK_07_ISR' start address is too low in 'ROM_07' by 3 bytes. ld65: Error: Cannot generate most of the files due to memory area overflow” , I ended up having to remove some code to put my own in. This was tricky to decide what to remove, and I decided on taking out the animation for the “pond secret” which serves as the entrance to Level 7.
In the end, I was able to make 2 out of the three changes I outlined in my proposal. I felt like I didn’t have time to change the music of the game, but I did learn enough about the world and cave code to add a hint to the map and to add a custom cave. It wasn’t exactly what I had intended, but I am happy with the result of the project. I had a lot of fun seeing the different unintended glitches caused by modifying the code.

### Course Concepts Applied
This project greatly increased my understanding of the following:
1. 6502 Assembly code
How it uses its three registers by loading and storing values
How it uses jumping, comparing, and branching to do conditional logic and loops
		Here’s an example of the 6502 code I wrote:
```
UpdateMapHintMarker:
    LDA CurLevel
    BNE Exit                    ; If in UW, then return.
    LDA #$05                    ; Load the hex value 5 into A
    JSR SwitchBank              ; Switching to another memory bank
    LDA #$01                    ; Load the hex value 1 into A
    STA $06                     ; A level test bit in [06] will be used to see whether we have a triforce piece.    
    JSR @FindNextLevel
    BEQ Exit                    ; If the next level room is 0, then return
    LDX #$04                    ; Load a hex value 4 into X (It's included after a comma when you STA. The purpose of this is: It adds an offset to the address being stored to. We'll keep it at 4 because the Triforce marker isn't used in the overworld)
    JMP UpdatePositionMarker    ; Update the marker

    ; This loops until it finds a level that hasn't been completed yet.
    ; Return:
    ; A: The roomID to show the hint
@FindNextLevel:
    LDA $06                      ; The level test bit
    BIT InvTriforce              ; If the level test bit is not in the triforce mask, then we don't have this piece. So, that's the next location
    BEQ @GetRoomIDForTriforceBit 
    
    ASL $06 ; Shift to the next bit to check
    BNE @FindNextLevel 
```
2. Memory layout and memory banks
How the game uses hex values to store memory
How the game performs binary operations on these values to perform more functionality
How the game switches between different banks of memory by exporting and importing code, and by using the SwitchBank section

