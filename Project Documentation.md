# CS 2810
## Final Project - Zelda 1 
## Jacob Schwartz A02357871

### Project Projected Timeline
#### Week 1: Build the project and familiarize myself with the project structure.
- Download the cc65 cross-compiler suite and VS Code extensions
- Identify the places I’ll need to work on to make the changes
- Read the tutorials and project outline
#### Week 2: Write the code for the new features
- Experiment with the location code and cave entrances
- Make the new cave
- Put enemies in the cave
- Experiment with changing sound code
- Make a simple song
- Experiment with the world map
- Add hints to the world map
- I will take some time in the middle of this week to evaluate my progress on the project and decide what I can realistically accomplish. In the case that making changes is less feasible than I expected, I will spend more time researching, and consider focusing on a single change.
#### Week 3: Debug and test the new features.
- I predict this stage will take a while, so I feel like this will take a whole week


### Changes and Modifications:
1. The text in the cave where you get the wooden sword is changed.
2. The map in the overworld now has a blinking red indicator of where the next dungeon is.
3. Because I had to remove code to add my own, the pond secret doesn't work anymore. But, I added a new way to enter level 7.
4. I added a new cave East 3 rooms from Link's starting point. This cave's ID is generated randomly each time a room is loaded.
*Note, this game is playable, but there is some visual weirdness that goes on*


### Actual Timeline Log:

3/31/2025: 1 hour 10 min - trying to get the game to compile. I downloaded and set up ca65, but I found out I might need to get an Original.nes file, which   
4/1: 15 min - trying to bypass the Original.nes situation - I got it to start assembling, but it seemed to be missing many dat files
4/2: 30 min - I managed to get the Original.nes from my future father-in-law, but then I realized I needed an emulator.. Mesen was very easy to download! And I just had to run the Z.nes file that I got by running the `.\build.ps1 -NoVerify` command. I also realized that I can run the Original.nes file in the emulator too. But I think if I want to make changes, I will need to assemble it to be different from the Original.nex.
```
Q - Select
W - Start
Arrow Keys - D-pad
B - A key
A - S key
```
4/3: 10:05 - 10:35 Read about 6502 and the NES memory layout
4/4: 10:45 - 11:20 Looked through the code that I have here and found this info:

1. Add a new cave that allows the user to practice fighting different enemy types.
    -  There's lots of cave insides logic in Z_01.asm
    -  But it seems that the cave and stairs logic is in Z_05.asm
    - Line 1595: `CPY #$24    ; $24 is black entrance, and contrasts with $70 to $73 (stairs).`
    - Line 7247 `CheckWarps:`
    ```
    ; Play the "secret found" tune.
    LDA #$04
    STA Tune1Request
    ```

2. Add new music and/or sound effects
    - Probably in Z_00.asm
    - Descriptions of song headers on line 16
    - CustomEnvelopeSong on line 

3. Add quality of life features such as indicators on the world map of the next recommended destination and better hints for obscure secrets. (The game manual provided many hints, but it would be nice to have these hints in-game)
    - Possibly in Z_01.asm?
    - Line 4901 has `MapScreenPosToPpuAddr:` 
    - `ChangePlayMapSquareOW` might be worth looking into too.

4/5: 
4/6: 
4/7: 
4/8: 
4/9: 8:53- 10:08
    - I need to learn about the LDA or Load Accumulator operator and STA or Store Accumulator. It's used to display cave entrances:
    ``` Z_05.asm line 5875
        LDA #$0C
        STA $0D
    ```
    - Cave tiles have an index of $24 for the bottom two tiles
        
    - The first cave:
        - Attribute address: $2BC2
        - Attribute data $AA
    - A different cave:
        - Attribute address: $23D3
        - Attribute data $AA

    I found this online:
    Here are all the 6502 Assembler ways of representing values, and how they will be treated.
    ```
    Prefix	Example	Z80 equivalent  	Meaning
    #	#16384	16384	Decimal Number
    #%	#%00001111	%00001111	Binary Number
    #$	#$4000	&4000	Hexadecimal number
    #'	#'a	'a'	ascii value

    12345	(16384)	decimal memory address
    $	$4000	(&4000)	Hexadecimal memory address
    ```
    I was reminded that 6502 only has three registers: A, X, and Y


4/10: 11:30- 12:00
    Goal: find the lookup table for the cave data
        - I've got a hunch about `CaveSourceRoomId`. Turns out, it's a variable in the Variables.inc file, and it points to $526 in memory. FF is the overworld, and 37 is level one.
            - According to datacrystal: It's the overworld room to return to when exiting the underworld
        - `HandleWarpOW` is also promising.
            - UndergroundEntranceTile := $65
                - This is set to 24 when you go in a cave or dungeon, and it's 26 in the overworld
            - This is where the logic is that decides if it's a level or a cave.

    Goal: find out what the different modes are:
    2- Target mode is 2 to load a level.
    9- target mode 9 (as in it's a cave)
    According to datacrystal:
    ```
    Address 12          Game Mode       0=Title/transitory    1=Selection Screen
                                        5=Normal              6=Preparing Scroll
                                        7=Scrolling           4=Finishing Scroll;
                                        E=Registration        F=Elimination 
    ```
    Seems like they're for the game intro.

    Goal Reevaluate this project: I think it's feasable to do SOMETHING, but it'll take more time than I've given it. I also might need to pick one thing to make sure it works.

4/11: 3:20 - 4:15
    Goal: look at yesterday's goals and try to accomplish them
    - TargetMode is different from Game mode.
        - TargetMode is set to 0B when you walk in a cave, but it switches to 02 in Level 1
    - It turns out that there are a lot of modes in this game. 
    
    - Also `LevelInfo_CellarRoomIdArray`
    - I wonder if there's a difference between caves and cellars... I think cellars are dungeons.
    New Goal: try to find how to place a cave on the map. (maybe look into how the tilemaps are stored?)
    ```
    The 130 background tiles from 70-F1 are swapped in and out as the game progresses. The splash screen tiles are loaded from ROM bank 1 (96B4). The surface tiles are loaded from ROM bank 3 (893B). The dungeon tiles are loaded from ROM bank 3 (811B).
    The 256 sprite tiles in VRAM (0000-FFF) are divided into fixed tiles that never change and enemy images that change as the player moves to different areas. Zelda uses two-tile sprites (8x16 pixels). Multiple sprites are grouped together to make larger creatures.
    ```
4/12: 11:15 - 1:00 and 3:30 - 5:30
        Okay, so I'm running out of days to work on this project. I need to figure out TODAY how the caves are loaded. I think it could be based on the `CaveSourceRoomId` attribute, because the different room you enter from could define the cave. But where is the data that says what the cave is based on the CaveSourceRoomId?? That is my main goal today.

    It really seems like `CaveSourceRoomId` only changes when you enter a level, not a cave...
    Overworld - FF
    Level 1 - 37
    Level 2 - 3C
    Level 3 - 74
    The first digit is the row, the second digit is the column.

     - `CellarSourceRoomId` really seems to be when you're in a dungeon already, and you go down stairs to a 2D area. In Level 1, it switches to  22 when you go down.

    - Important: RoomId, which is what room you're on the map is saved at $EB
    - So, it seems like there's a lot of cave code in Z_01.asm
        - CaveFlags := $413
        - LevelBlockAttrsE := $6A7E is where the cave level info is stored. Item ID's are in the low 6 bits, cave flags are in the high 2 bits.
        - Caves are initialized in the InitCave section
    - Also `FetchAttrs` seems important!!!

    - Switching gears to find how to make a new cave object. Where are the tile maps for the game?
        - FetchTileMapAddr
        - So, I think all of this information is actually in the zelda1-disassembly > bin > dat files such as RoomLayoutsOW.dat.

    - I... kinda got stuck. I wasn't sure how to edit these .dat files. I don't know if it's feasible or not. I looked up solutions online and didn't find any.

    - I MADE A CHANGE TO THE DAT FILE!!! It took a lot of effort.
    1. I asked GitHub Copilot for advice, and it basically told me what I was already suspecting.
    2. I downloaded a Hex file editor so I could edit the dat files, and decided to start with changing text rather than the world map.
    3. I made the change, but it wouldn't keep it. I fiddled with the build script file, until I realized there was a command `-NoExtract` which is what was what I needed to make it stop overriding my changes. "HT'S DANGEROUS TO GO ALONE" for the win!

4/13: 
4/14: 2:45 - 4:30
    Goals: I changed a letter, now can I change the map?
    Okay....
    So ....
    I managed to change a row of Link's sprite by changing CommonSpritePatterns.dat
    But.. I couldn't figure out what changing RoomLayoutsOWModified did. I'm not sure what the structure is exactly...
    Then I looked it up, and you can only have 1 cave per room, so my plan I had for 9 dungeon caves is a no go.
    I think I should forgo my second feature because delving into music would be something new, but I've already become a little familiar with the map and caves.
    I probably can't add an additional cave, so my new goal is to convert a cave to have something cool/different inside it. Probably change the text/sprites because I figured out how to do that.
    I need to demonstrate my understanding of the code though, and changing the text and sprites is mostly visual stuff. I think .... I should add something new.
    1. Change the map flags
    2. Add something new instead. Can I have enemies in a cave?
    Maybe I should have the old man's cave just cycle through the dungeons? Or display a random dungeon once Link has the sword. Or give Link all the items, and then let him access all the dungeons from that one cave?
4/15: 10:40 - 12:30
    Okay, picking up where I left off last time.
    Map flags are stored in: 
        CaveFlags := $413
        CaveItemIds := $422
        Cave flags are in the high 2 bits from the Cave level block info.
        The Cave level block info is in LevelBlockAttrsE := $6A7E
        The cave's index is gotten using (object type - $6A (aka $106)) and stored in Y
    I wasn't quite able to figure it out. I did change the caves... kinda. 
    I tried making Link teleport to the starting room whenever he took damage, and that didn't work. I need to remember that if I add new code, I probably have to remove other stuff. 
    2:55 - 4:50

    I found `UpdatePlayerPositionMarker` which keeps track of the overworld map.
    I think this is a change that I can make. In dungeons, it already has indicators on the map of where you need to go.
    I could even potentially change the text that usually says what level you're on do say what the hint indicator is.
        - So, the level number is saved in the respective LevelInfoUW.dat file in address 33. The rest of the string is located in ... I'm not sure where...

    ``` Here's how the hint indicator works in the UW:
    UpdateTriforcePositionMarker:
        LDA CurLevel                ; Load the level id number in A
        BEQ Exit                    ; If in OW, then return.
        LDA #$05                    ; Load the hex value 5 into A
        JSR SwitchBank              
        JSR HasCompass              ; Check if the player has the compass
        BEQ Exit                    ; If player hasn't gotten the compass, then return.
        LDA LevelInfo_TriforceRoomId; Load the room id number for the triforce based on the level number
        LDX #$04                    ; Load the hex value 4 into X
        JMP UpdatePositionMarker    ; Update the marker
    ```    

    ```
    UpdatePlayerPositionMarker:
        LDA GameMode                ; Return if mode 9 or whirlwind teleporting.
        CMP #$09
        BEQ L6AAF_Exit
        LDX WhirlwindTeleportingState
        BNE L6AAF_Exit
        LDA RoomId                  ; Load the room id number
        LDX #$00                    ; Load the hex value 0 into X
    ```

    - What I gather is:
        - There are conditions we want it to display (probably for me, it's based on what items are collected)
        - A gets stored with the room id
        - X gets stored with ____
        - Jump to the subroutine which will actually display it.

    ``` So mine will be more like this:
    UpdateMapHintMarker:
        ; TODO: make it based on the items that have been collected so far
        LDA #$05                    ; Load the hex value 5 into A
        JSR SwitchBank              ; Switching to another memory bank because this code is in 7, and we want to do stuff in 1... I think.
        LDA HintMarkerRoomId; Load the room id number for the hint
        LDX #$04                    ; Load a hex value 4 into X (It's included after a comma when you STA. The purpose of this is: It adds an offset to the address being stored to. We'll keep it at 4 because the Triforce marker isn't used in the overworld)
        JMP UpdatePositionMarker    ; Update the marker
    ```  
    8:45 - 9:53

    Find out where it keeps track of what items and collectables we have so far:
        - Items := $657 ; I think this is mostly for sword type and accessing other items.
        - InvTriforce := $671 ; This can be used to track which dungeon should be done next... Wait, maybe not because dungeons can be done out of order.
        - It's probably recorded in the LevelInfo_WorldFlagsAddr which levels are completed. So... I think it would be cool if it tells you where the "next" level is located.

4/16: 1:47 - 4:16
    Goal for the next hour or so: Find the flags that say which dungeons have been complete, and then make the location of the map marker indicate the next dungeon room.

    Before dungeon 1 is beat, it says FF
    When dungeon 1 is beat, it says 7F
    When I was in dungeon 2 it said FF again, 
    and when I beat dungeon 2 it said 7F...

    Okay, so, When it's displaying the triforce pieces in the start menu, it displays them based on which dungeon has been cleared. They're the `triforce tile slot`s

    Okay! I've got it now! Here's how it work:
    1. The InvTriforce is stored at $671.
    2. When you get a triforce piece, it's stored here by changing the bit. 
    - Level 1 cleared = 0000 0001 = #$1
    - Level 2 cleared = 0000 0011 = #$4
    - All levels cleared = 1111 1111 = #$255
    3. It loops through each of the digits of InvTriforce and displays the triforce based on that.

    Here's the logic we'll use for the mark hint:
    ; A level test bit in [06] will be used to see whether
    ; we have a triforce piece.    
    LDA #$01
    STA $06                     ; [06] level test bit
 
    ; Returns:
    ; A: The roomID to show the hint
    ; 
    @FindNextLevel
    ; If the level test bit is not in the triforce mask, then we don't
    ; have this piece. So, that's the next location we want
    LDA $06                     ; [06] level test bit
    BIT InvTriforce
    BEQ @FindNextLevel
    ; 
    ; Somehow, based on the number, we need to get the roomID. There's probably something in the code that says where each level is?
    ; For now, I might hard code it...   Level 1 - 37
    Level 1 - 37
    Level 2 - 3C
    Level 3 - 74

    4:35 - 5:15
    I put the code in.
    I needed more space, so I commented out DrawStatusBarPotion and @LoopBoomerang
    It kinda works! There's definitely a few bugs.
    - The marker doesn't show up all of the time. It does if you press Start
    - The marker doesn't flash
    - The inventory item slot got messed up - likely because I commented out stuff.
    
    30 min + 10:00-
    - Made it only visible in the OW.
    - Commented out AnimatePond and RevealPondStairs, and uncommented the DrawStatusBarPotion and @LoopBoomerang
    ? How do we change the color? - It does it in UpdatePositionMarker, but I don't know why it won't do mine as well, because it seems to be based on X, and I made X the same.

    - Somehow, based on the number, we need to get the roomID. There's probably something in the code that says where each level is?
    
    ```
    ; Get the cave index attribute.
    LDY RoomId
    LDA LevelBlockAttrsB, Y
    AND #$FC                    ; Cave index

    ; If < $40, go deal with a level.
    ; $40 means cave index $10. Levels have cave indexes 1 to 9.
    CMP #$40
    BCC @LoadLevel ; If < $40 ; Whatever is in A gets logical shifted right twice inorder to get the level number
    ```
    So the RoomId to Level mapping is done in the addresses LevelBlockAttrsB + RoomId.
    That's $68FE + #$3C aka 60 = $693A which holds... 0A? Oh! Because it shifts right twice:
    0A = 01010 -shift-twice-> 0010 which is the value of 2.
    
    And we're looking for:
    - Level 1: $01 -> hex 09 = bin 01001 -shift-twice-> 00010 which is the value of 2
    - Level 2: $02 -> hex 0A = bin 01010 -shift-twice-> 00010 which is the value of 2
    - Level 3: $03 -> hex 0B = bin 10001 -shift-twice-> 00100 which is the value of 4
    , 0C, 0D, 0E, 0F
    hex 0F = bin 1111 -shift-twice-> 0011 which is the value of 3
    - Level 8: $08 -> hex 10
    - Level 9: $09 -> hex 11 = bin 10001 -shift-twice-> 00100 which is the value of 4

    $3A, $3B, $3C, $3D, $3E, $3F
    - Cave index 16: $10 -> $40
    - Cave index 20: $14 -> $50 meaning (shortcuts)

4/17: 
    10:45 - 12:10

    I made a spreadsheet to better help me recognize how the memory stores which cave to load.
    So now, I need to apply that knowledge to get the roomID based on which bit of the triforce is not yet gotten. For example:
    If level one and three have been beaten, but not two, the InvTriforce value will be 0000 0101. So we need to point to the second level.
    This might be a bit tricky..
    
    Hypothetically, the value we're looking at is getting bit shifted each time we loop through. And when we find a bit that is 0, we stop.
    So it would be stopped as 0000 0010??
    I need to understand how this bit shifting and checking works.

    3:30 - 5:00
    Goal: Get the marker to display in the right places.
    So bit checking uses a mask to check if two bits are the same:
    - Mask: 0000 0010 checks against 1001 1011 to see if the bit is 0. 

    So, I added the conditional logic to test which level is next, but to make it assemble and link, I had to comment out more of the code concerning the pond secret.
    The map marker seems to work! But, I'm getting some goofy bugs when enemies are killed.. XD
    I fixed the bugs, I shouldn't comment out anything from Jump Tables
    After testing it for the first few dungeons, it seems to work just fine!... Except the pond secret for level 7 is broken... Wait! Maybe I could just remove the need for the pond secret?

    I could just use this code without needing the secret part:
    ```
    RevealPondStairs:
        ; Set X and Y in this slot for the stairs in the pond.
        ; Go reveal the stairs as a secret.
        LDA #$60
        STA ObjX, X
        LDA #$90
        STA ObjY, X
        JMP RevealAndFlagSecretStairsObj
    ```
    7:10 - 10:45
    I've been trying to get the pond secret to work, but it keeps breaking.
    ObjType is in $34F
    Okay, I've been working on that for too long, and it's still broken. I'll come back to that..

    Now I'm going to fix the marker's visual behavior (blinking and start menu.)
    Fixed the blinking by adding
    ```
    LDA #$04
    JSR SwitchBank
    JSR UpdateMapHintMarker
    ```
    to ` @CheckOW:`

    I found out where the problem with the start menu scrolling is. It's in `MovePositionMarkers`. So the map marker now stays on the map. Buuut. When I try to add the extra position marker I'm getting some crazy map layout bugs. I'm not sure how to get the best of both worlds.

    I got the Old Man's intro text to work!! The problem was that aparently there are special values for character that are at the end of the line. 
    
    Idea: what if I could make the player position marker blink a certain color if there's a secret in that room? It could blink different colors based on the secret type?

    I found Link's tunic color value. Looking into NES color values, there's 64 options.
    The tunic color value is located at CPU address $6B92.
    Now, how to change that default...
    ```
    LDA #$2C
    STA $6B92
    ```
    After looking for a while, I'm still not sure how to change the value upon initialization of the PPU data.

    I want to try adding a cave to room 7A.
    It has to do with RoomLayoutsOWModified

    TODO:
    - Fix the marker's behavior
    - Figure out how to change the tunic color. (Look into the rings?)
    - Look into player position marker blinking a certain color

4/18: 
    10 min + 10 min (Today was my mom's 50th birthday party)
    Goal: work to understand `UpdatePositionMarker` so I could add blinking to the player position too, and then change the palette of the blink based on the roomID.

    Okay, maybe more important goal: Try to figure out how to add a new cave.
    I want to try adding a cave to room 7A.
    It has to do with RoomLayoutsOWModified

4/19: 
    11:40 - 12:30
    1:20 - 2:50
    3:45 - 5:55
    6:05 - 8:27

    Okay, so I've confirmed what I was suspecting. Each row of data in RoomLayoutsOW corresponds with a room. And each Hex value represents a row. So for example,
    "47 47 47 47 47 47 85 47 47 47 47 47 47 47 47 47" is room # 7D. It's all the same tiles, but there's a hidden cave on the 7th tile, which is the `85`.
    I think what was confusing me was I was expecting 128 rooms * 16 columns to be 2048, but there's only 121 rows of 16 in the RoomLayoutsOW file. I think this is because there's some duplicate rooms?

    I found the starting room, but it wasn't the row I was expecting it to be on:
    "01 A7 84 90 10 02 A5 08 08 A0 06 06 06 06 01 01"

    Room 7A is a room that is reused, so that might make it had to find it's layout... Nevermind, it was just at the first occurance of the room type (Room 07)
    "A7 A6 A7 F1 A9 A8 A9 A2 A3 A8 A6 A7 A6 A7 A6 01"

     A7 A6 A7 F1 A9 A8 A9 10 02 A8 A6 A7 A6 A7 A6 01

     It worked! When you go in the cave it just resets Link to the same room. Because this room is reused, it also makes a cave for the other occurances... But that's fine.
     Next step: make the cave load a cave.

    Set the address of $6978 to a cave ID value.
    Maybe I'll make it load a random cave value? I'm having fun just seeing different effects happen.
    I also need to fix the respawn location.
    How to get a random hex value? In Variables we have Random := $18, and it seems like the next 13 addresses have random variables.
    ```
    LDA Random
    STA $6978
    ```
    I need to figure out how to update memory when Link enters a specific room. Look into 
    - InitMode_EnterRoom - Seems like it's for caves and cellars
    - StepOutside
    - CalculateNextRoomOW 
    - I got it to kinda work by putting it in `CreateRoomObjects`

    Now, How to fix the location of exiting the cave...
    I figured out that the X position of the exit is stored in LevelBlockAttrsA, specifically at $68F8. And that the proper value it should be set to is #$73
    

    I think I found the fairy fountain layouts:
    "00	A9 02 77 02 53 54 D1 D1 54 56 02 77 02 A8 00"
    
    This one might be the entrance to level 7? Nope, the prior line is.
    "00 A9 10 53 54 B1 55 B2 54 54 54 56 02 B5 A8 00"
    This one is the room with the White Sword.

    I can add an entrance to level 7 so long as there's a cave entrance on that room.
    The row in the RoomLayoutsOW dat file is 896

    I got it to work! I added a tile column that adds a bombable wall just left of the pool. I used #$D2

```
    LDA #$3F   ; High byte of the PPU Address
    STA $2006   ; Use STA $2006 to write it to the PPUADDR register.
    LDA #$11   ; Low byte of the PPU Address
    STA $2006   ; Use STA $2006 to write it to the PPUADDR register.
    
    LDA #$FF
    STA $2007 ; Use STA $2007 to store the data in the PPU
```

4/20: Easter
4/21: Project showcase

    At this point I've worked ~ *approximately* ~ 1740 minutes, or 29 hours. This includes the research that had to be done at the beginning. I was also rather general in keeping track of my hours on most days. 

    During the presentation, the triforce pieces didn't show up... I think that might be because of the recent changes I've added...
    I think it was because I had these lines flipped.. whoops!
    ```
    LDA CurLevel
    BEQ @LoadRandomCave
    ```

4/22: Project is due!

